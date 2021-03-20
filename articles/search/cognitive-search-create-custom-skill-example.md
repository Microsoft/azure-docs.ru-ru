---
title: Пример пользовательской квалификации с использованием API Bing для поиска сущностей
titleSuffix: Azure Cognitive Search
description: Демонстрируется использование службы Поиск сущностей Bing в пользовательском навыке, сопоставленном с ИСКУССТВЕНным конвейером индексирования в Azure Когнитивный поиск.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 5755e14e53d359fd8b322939bf1325d21536d593
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89020190"
---
# <a name="example-create-a-custom-skill-using-the-bing-entity-search-api"></a>Пример. Создание пользовательского навыка с помощью API Bing для поиска сущностей

В этом примере вы узнаете, как создать пользовательский навык веб-API. Этот навык будет принимать местоположения, общедоступные рисунки и организации и возвращать их описания. В примере используется [функция Azure](https://azure.microsoft.com/services/functions/) для заключения [API Bing для поиска сущностей](https://azure.microsoft.com/services/cognitive-services/bing-entity-search-api/) , чтобы она реализовала пользовательский интерфейс квалификации.

## <a name="prerequisites"></a>Предварительные требования

+ Ознакомьтесь со статьей о [пользовательском интерфейсе навыков](cognitive-search-custom-skill-interface.md) , если вы не знакомы с интерфейсом ввода-вывода, который должен быть реализован в пользовательском навыке.

+ [!INCLUDE [cognitive-services-bing-entity-search-signup-requirements](../../includes/cognitive-services-bing-entity-search-signup-requirements.md)]

+ Установите [Visual Studio 2019](https://www.visualstudio.com/vs/) или более поздней версии, включая рабочую нагрузку разработки Azure.

## <a name="create-an-azure-function"></a>Создание функции Azure

Хотя в этом примере для размещения веб-API используется функция Azure, она не требуется.  При условии соблюдения [требований к интерфейсу для когнитивного навыка](cognitive-search-custom-skill-interface.md) выбранный вами подход не имеет значения. Однако Функции Azure упрощают создание пользовательских навыков.

### <a name="create-a-function-app"></a>Создание приложения-функции

1. В Visual Studio в меню Файл выберите **Создать** > **Проект**.

1. В диалоговом окне "Новый проект" выберите **Установленные**, разверните узел **Visual C#** > **Облако**, выберите **Функции Azure**, укажите имя проекта и нажмите кнопку **ОК**. Имя приложения-функции должно быть допустимым в качестве пространства имен C#, поэтому не используйте знаки подчеркивания, дефисы и другие символы, отличные от буквенно-цифровых.

1. Выберите **функции Azure v2 (.NET Core)**. Это также можно сделать с помощью версии 1, но код, написанный ниже, основан на шаблоне версии 2.

1. Выберите тип **Триггер HTTP**.

1. Для учетной записи хранения можно выбрать **Нет**, так как для этой функции никакое хранилище не требуется.

1. Нажмите кнопку **ОК**, чтобы создать проект функции и функцию, активируемую с помощью HTTP.

### <a name="modify-the-code-to-call-the-bing-entity-search-service"></a>Изменение кода для вызова службы Поиск сущностей Bing

Visual Studio создает проект и класс в нем, содержащий стандартный код для выбранного типа функции. Атрибут метода *FunctionName* задает имя функции. Атрибут *HttpTrigger* указывает, что функция вызывается HTTP-запросом.

Замените содержимое файла *Function1.cs* следующим кодом:

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Net.Http;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;

namespace SampleSkills
{
    /// <summary>
    /// Sample custom skill that wraps the Bing entity search API to connect it with a 
    /// AI enrichment pipeline.
    /// </summary>
    public static class BingEntitySearch
    {
        #region Credentials
        // IMPORTANT: Make sure to enter your credential and to verify the API endpoint matches yours.
        static readonly string bingApiEndpoint = "https://api.cognitive.microsoft.com/bing/v7.0/entities/";
        static readonly string key = "<enter your api key here>";  
        #endregion

        #region Class used to deserialize the request
        private class InputRecord
        {
            public class InputRecordData
            {
                public string Name { get; set; }
            }

            public string RecordId { get; set; }
            public InputRecordData Data { get; set; }
        }

        private class WebApiRequest
        {
            public List<InputRecord> Values { get; set; }
        }
        #endregion

        #region Classes used to serialize the response

        private class OutputRecord
        {
            public class OutputRecordData
            {
                public string Name { get; set; } = "";
                public string Description { get; set; } = "";
                public string Source { get; set; } = "";
                public string SourceUrl { get; set; } = "";
                public string LicenseAttribution { get; set; } = "";
                public string LicenseUrl { get; set; } = "";
            }

            public class OutputRecordMessage
            {
                public string Message { get; set; }
            }

            public string RecordId { get; set; }
            public OutputRecordData Data { get; set; }
            public List<OutputRecordMessage> Errors { get; set; }
            public List<OutputRecordMessage> Warnings { get; set; }
        }

        private class WebApiResponse
        {
            public List<OutputRecord> Values { get; set; }
        }
        #endregion

        #region Classes used to interact with the Bing API
        private class BingResponse
        {
            public BingEntities Entities { get; set; }
        }
        private class BingEntities
        {
            public BingEntity[] Value { get; set; }
        }

        private class BingEntity
        {
            public class EntityPresentationinfo
            {
                public string[] EntityTypeHints { get; set; }
            }

            public class License
            {
                public string Url { get; set; }
            }

            public class ContractualRule
            {
                public string _type { get; set; }
                public License License { get; set; }
                public string LicenseNotice { get; set; }
                public string Text { get; set; }
                public string Url { get; set; }
            }

            public ContractualRule[] ContractualRules { get; set; }
            public string Description { get; set; }
            public string Name { get; set; }
            public EntityPresentationinfo EntityPresentationInfo { get; set; }
        }
        #endregion

        #region The Azure Function definition

        [FunctionName("EntitySearch")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation("Entity Search function: C# HTTP trigger function processed a request.");

            var response = new WebApiResponse
            {
                Values = new List<OutputRecord>()
            };

            string requestBody = new StreamReader(req.Body).ReadToEnd();
            var data = JsonConvert.DeserializeObject<WebApiRequest>(requestBody);

            // Do some schema validation
            if (data == null)
            {
                return new BadRequestObjectResult("The request schema does not match expected schema.");
            }
            if (data.Values == null)
            {
                return new BadRequestObjectResult("The request schema does not match expected schema. Could not find values array.");
            }

            // Calculate the response for each value.
            foreach (var record in data.Values)
            {
                if (record == null || record.RecordId == null) continue;

                OutputRecord responseRecord = new OutputRecord
                {
                    RecordId = record.RecordId
                };

                try
                {
                    responseRecord.Data = GetEntityMetadata(record.Data.Name).Result;
                }
                catch (Exception e)
                {
                    // Something bad happened, log the issue.
                    var error = new OutputRecord.OutputRecordMessage
                    {
                        Message = e.Message
                    };

                    responseRecord.Errors = new List<OutputRecord.OutputRecordMessage>
                    {
                        error
                    };
                }
                finally
                {
                    response.Values.Add(responseRecord);
                }
            }

            return (ActionResult)new OkObjectResult(response);
        }

        #endregion

        #region Methods to call the Bing API
        /// <summary>
        /// Gets metadata for a particular entity based on its name using Bing Entity Search
        /// </summary>
        /// <param name="entityName">The name of the entity to extract data for.</param>
        /// <returns>Asynchronous task that returns entity data. </returns>
        private async static Task<OutputRecord.OutputRecordData> GetEntityMetadata(string entityName)
        {
            var uri = bingApiEndpoint + "?q=" + entityName + "&mkt=en-us&count=10&offset=0&safesearch=Moderate";
            var result = new OutputRecord.OutputRecordData();

            using (var client = new HttpClient())
            using (var request = new HttpRequestMessage {
                Method = HttpMethod.Get,
                RequestUri = new Uri(uri)
            })
            {
                request.Headers.Add("Ocp-Apim-Subscription-Key", key);

                HttpResponseMessage response = await client.SendAsync(request);
                string responseBody = await response?.Content?.ReadAsStringAsync();

                BingResponse bingResult = JsonConvert.DeserializeObject<BingResponse>(responseBody);
                if (bingResult != null)
                {
                    // In addition to the list of entities that could match the name, for simplicity let's return information
                    // for the top match as additional metadata at the root object.
                    return AddTopEntityMetadata(bingResult.Entities?.Value);
                }
            }

            return result;
        }

        private static OutputRecord.OutputRecordData AddTopEntityMetadata(BingEntity[] entities)
        {
            if (entities != null)
            {
                foreach (BingEntity entity in entities.Where(
                    entity => entity?.EntityPresentationInfo?.EntityTypeHints != null
                        && (entity.EntityPresentationInfo.EntityTypeHints[0] == "Person"
                            || entity.EntityPresentationInfo.EntityTypeHints[0] == "Organization"
                            || entity.EntityPresentationInfo.EntityTypeHints[0] == "Location")
                        && !String.IsNullOrEmpty(entity.Description)))
                {
                    var rootObject = new OutputRecord.OutputRecordData
                    {
                        Description = entity.Description,
                        Name = entity.Name
                    };

                    if (entity.ContractualRules != null)
                    {
                        foreach (var rule in entity.ContractualRules)
                        {
                            switch (rule._type)
                            {
                                case "ContractualRules/LicenseAttribution":
                                    rootObject.LicenseAttribution = rule.LicenseNotice;
                                    rootObject.LicenseUrl = rule.License.Url;
                                    break;
                                case "ContractualRules/LinkAttribution":
                                    rootObject.Source = rule.Text;
                                    rootObject.SourceUrl = rule.Url;
                                    break;
                            }
                        }
                    }

                    return rootObject;
                }
            }

            return new OutputRecord.OutputRecordData();
        }
        #endregion
    }
}
```

Обязательно введите собственное значение *ключа* в `key` константе на основе ключа, полученного при регистрации в API Bing для поиска сущностей.

Этот пример включает в себя весь необходимый код в одном файле для удобства. В [репозитории Power Skills](https://github.com/Azure-Samples/azure-search-power-skills/tree/master/Text/BingEntitySearch)можно найти немного более структурированную версию этого же навыка.

Конечно, файл можно переименовать с `Function1.cs` на `BingEntitySearch.cs` .

## <a name="test-the-function-from-visual-studio"></a>Тестирование функции из Visual Studio

Нажмите клавишу **F5** для запуска программы и тестирования поведения функции. В этом случае мы будем использовать функцию ниже для поиска двух сущностей. Используйте Postman или Fiddler для выполнения вызова, как показано ниже:

```http
POST https://localhost:7071/api/EntitySearch
```

### <a name="request-body"></a>Тело запроса
```json
{
    "values": [
        {
            "recordId": "e1",
            "data":
            {
                "name":  "Pablo Picasso"
            }
        },
        {
            "recordId": "e2",
            "data":
            {
                "name":  "Microsoft"
            }
        }
    ]
}
```

### <a name="response"></a>Ответ
Вы должны увидеть отклик, аналогичный следующему примеру:

```json
{
    "values": [
        {
            "recordId": "e1",
            "data": {
                "name": "Pablo Picasso",
                "description": "Pablo Ruiz Picasso was a Spanish painter [...]",
                "source": "Wikipedia",
                "sourceUrl": "http://en.wikipedia.org/wiki/Pablo_Picasso",
                "licenseAttribution": "Text under CC-BY-SA license",
                "licenseUrl": "http://creativecommons.org/licenses/by-sa/3.0/"
            },
            "errors": null,
            "warnings": null
        },
        "..."
    ]
}
```

## <a name="publish-the-function-to-azure"></a>Публикация функции в Azure

Когда поведение функции будет удовлетворено, его можно опубликовать.

1. Щелкните правой кнопкой мыши проект в **обозревателе решений** и выберите пункт **Опубликовать**. Выберите **создать**  >  **опубликовать**.

1. Если вы еще не подключились к учетной записи Azure с помощью Visual Studio, выберите **Добавить учетную запись....**

1. Следуйте инструкциям на экране. Вам будет предложено указать уникальное имя для службы приложений, подписку Azure, группу ресурсов, план размещения и учетную запись хранения, которую вы хотите использовать. При отсутствии группы ресурсов, плана размещения и учетной записи хранения их можно создать. По завершении нажмите кнопку **создать** .

1. После завершения развертывания обратите внимание на URL-адрес сайта. Это адрес приложения-функции в Azure. 

1. В [портал Azure](https://portal.azure.com)перейдите к группе ресурсов и найдите `EntitySearch` опубликованную функцию. В разделе **Управление** должны отображаться ключи узла. Выберите значок **копирования** для ключа узла *по умолчанию*.  

## <a name="test-the-function-in-azure"></a>Тестирование функции в Azure

Теперь, когда у вас есть ключ узла по умолчанию, протестируйте функцию следующим образом:

```http
POST https://[your-entity-search-app-name].azurewebsites.net/api/EntitySearch?code=[enter default host key here]
```

### <a name="request-body"></a>Текст запроса
```json
{
    "values": [
        {
            "recordId": "e1",
            "data":
            {
                "name":  "Pablo Picasso"
            }
        },
        {
            "recordId": "e2",
            "data":
            {
                "name":  "Microsoft"
            }
        }
    ]
}
```

Этот пример должен иметь тот же результат, который вы видели ранее при выполнении функции в локальной среде.

## <a name="connect-to-your-pipeline"></a>Подключение к конвейеру
Теперь, когда у вас есть новый пользовательский навык, можно добавить его в ваш набор навыков. В приведенном ниже примере показано, как вызвать навык для добавления описаний в Организации в документе (это можно расширить для работы с расположениями и людьми). Замените `[your-entity-search-app-name]` именем своего приложения.

```json
{
    "skills": [
      "[... your existing skills remain here]",  
      {
        "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
        "description": "Our new Bing entity search custom skill",
        "uri": "https://[your-entity-search-app-name].azurewebsites.net/api/EntitySearch?code=[enter default host key here]",
          "context": "/document/merged_content/organizations/*",
          "inputs": [
            {
              "name": "name",
              "source": "/document/merged_content/organizations/*"
            }
          ],
          "outputs": [
            {
              "name": "description",
              "targetName": "description"
            }
          ]
      }
  ]
}
```

Здесь мы подсчитани встроенный [навык распознавания сущностей](cognitive-search-skill-entity-recognition.md) , который должен присутствовать в наборе навыков, а также для создания документа со списком организаций. Для справки ниже приведена конфигурация навыков извлечения сущностей, которая может быть достаточной для создания необходимых данных:

```json
{
    "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
    "name": "#1",
    "description": "Organization name extraction",
    "context": "/document/merged_content",
    "categories": [ "Organization" ],
    "defaultLanguageCode": "en",
    "inputs": [
        {
            "name": "text",
            "source": "/document/merged_content"
        },
        {
            "name": "languageCode",
            "source": "/document/language"
        }
    ],
    "outputs": [
        {
            "name": "organizations",
            "targetName": "organizations"
        }
    ]
},
```

## <a name="next-steps"></a>Дальнейшие действия
Поздравляем! Вы создали свой первый пользовательский навык. Теперь вы можете использовать ту же схему для добавления пользовательских функций. Чтобы получить дополнительные сведения, перейдите по следующим ссылкам.

+ [Power Skills: репозиторий пользовательских навыков](https://github.com/Azure-Samples/azure-search-power-skills)
+ [Добавление пользовательского навыка в конвейер обогащения искусственного интеллекта](cognitive-search-custom-skill-interface.md)
+ [Определение набора навыков](cognitive-search-defining-skillset.md)
+ [Создание набора навыков (REST)](/rest/api/searchservice/create-skillset)
+ [Сопоставление обогащенных полей](cognitive-search-output-field-mapping.md)