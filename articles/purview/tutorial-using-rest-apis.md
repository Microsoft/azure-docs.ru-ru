---
title: Руководство по использованию REST API
description: Сведения о том, как с помощью REST API для Azure Purview обращаться к содержимому каталога.
author: hophanms
ms.author: hophan
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: tutorial
ms.date: 12/03/2020
ms.openlocfilehash: bfb808c634ba946e1a4825d7828db6df8963352c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98951248"
---
# <a name="tutorial-use-the-rest-apis"></a>Руководство по использованию REST API

Из этого руководства вы узнаете, как использовать REST API для Azure Purview. Любой пользователь, желающий отправить данные в каталог Azure Purview, включить этот каталог в состав автоматизированного процесса или создать собственный пользовательский интерфейс взаимодействия с каталогом, может применить для этого существующие REST API.

В этом руководстве описано следующее:

> [!div class="checklist"]
>
> * создание субъекта-службы (приложения);
> * настройка в каталоге доверия к субъекту-службе (приложению);
> * просмотр документации по REST API;
> * сбор необходимых значений для использования REST API;
> * вызов REST API с помощью клиента Postman;
> * создание клиента .NET для вызова REST API.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

* Чтобы приступить к работе, вам нужна учетная запись Azure Purview. Если у вас еще нет каталога, воспользуйтесь [кратким руководством по созданию учетной записи Azure Purview](create-catalog-portal.md).

* Если вам нужно добавить в каталог данные, изучите [руководство по запуску начального набора и сканированию данных](tutorial-scan-data.md).

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите на [портал Azure](https://portal.azure.com).

## <a name="create-a-service-principal-application"></a>Создание субъекта-службы (приложения)

Чтобы клиент REST API мог обращаться к каталогу, этому клиенту необходимо предоставить субъект-службу (приложение) и удостоверение, которое распознается этим каталогом и которому доверяет этот каталог. При выполнении вызовов REST API к каталогу используется удостоверение субъекта-службы.

Клиенты, которые использовали уже существующие субъекты-службы (идентификаторы приложений), испытывали высокий уровень сбоев. Поэтому мы рекомендуем создавать для обращения к API новый субъект-службу.

Чтобы создать субъект-службу, сделайте следующее:

1. На [портале Azure](https://portal.azure.com) найдите и выберите элемент **Azure Active Directory**.
1. На странице **Azure Active Directory** щелкните на панели слева элемент **Регистрация приложений**.
1. Выберите **Новая регистрация**.
1. На странице **Регистрация приложения** сделайте следующее:
    1. Введите **имя** приложения (имя субъекта-службы).
    1. Выберите элемент **Учетные записи только в этом каталоге организации (только _&lt;имя_вашего_арендатора&gt;_  — один клиент)** .
    1. В разделе **URI перенаправления (необязательно)** выберите элемент **Интернет** и введите произвольное значение. Здесь необязательно использовать допустимый URI.
    1. Выберите **Зарегистрировать**.
1. На странице нового субъекта-службы скопируйте значения параметров **Отображаемое имя** и **Идентификатор приложения (клиента)** , которые нам потребуются далее.

   Идентификатор приложения соответствует значению `client_id` в нашем примере кода.

Чтобы использовать субъект-службу (приложение), нужно получить его пароль. Это делается так.

1. Войдите на портал Azure и выберите элемент **Azure Active Directory**, а затем на панели слева выберите элемент **Регистрация приложений**.
1. Выберите из списка субъект-службу (приложение).
1. На панели слева щелкните элемент **Сертификаты и секреты**.
1. Выберите **Создать секрет клиента**.
1. На странице **Добавление секрета клиента** введите **описание**, выберите срок действия в поле **Истекает**, а затем щелкните элемент **Добавить**.

   На странице **Секреты клиента** строка в столбце **Значение** содержит пароль от нового секрета. Сохраните его.

   :::image type="content" source="./media/tutorial-using-rest-apis/client-secret.png" alt-text="Снимок экрана: секрет клиента.":::

## <a name="configure-your-catalog-to-trust-the-service-principal-application"></a>Настройка в каталоге доверия к субъекту-службе (приложению)

Чтобы настроить в Azure Purview доверие к новому субъекту-службе, сделайте следующее:

1. Перейдите к учетной записи Purview

1. На странице **Учетная запись Purview** выберите вкладку **Управление доступом (IAM)** .

1. Щелкните **+ Добавить**.

1. **Добавить назначение ролей**

1. В поле "Роль" введите **Куратор данных Purview**.

    > [!Note]
    > Дополнительные сведения о разрешениях каталога данных Azure Purview см. в разделе [Разрешения каталога](catalog-permissions.md). Например, если вам нужно разрешение на запуск сканирования, требуется тип роли **Администратор источника данных Purview**.

1. В поле **Назначить доступ для** сохраните вариант по умолчанию **Пользователь, группа или субъект-служба**.

1. В разделе **Выбор** введите имя ранее созданного субъекта-службы, которому вы хотите назначить роль, а затем щелкните нужное имя в области результатов.

1. В меню «Параметры» щелкните пункт **Сохранить**

Итак, вы настроили субъект-службу с ролью администратора приложения, которая позволяет отправлять содержимое в каталог.

## <a name="view-the-rest-apis-documentation"></a>Просмотр документации по REST API

Чтобы просмотреть документацию Swagger по API, скачайте файл [PurviewCatalogAPISwagger.zip](https://github.com/Azure/Purview-Samples/raw/master/rest-api/PurviewCatalogAPISwagger.zip), извлеките его содержимое и откройте файл index.html.

Дополнительные сведения об API расширенного поиска и предложений, предоставляемых для Azure Purview, вы найдете в документации по фильтрации поиска в REST API. Автоматически создаваемый клиент сейчас не поддерживает настраиваемые параметры поиска. Чтобы обойти эту проблему, с помощью документации по фильтрации поиска определите в коде классы фильтров как параметры вызова API. Примеры таких API можно найти в документе index.html.

## <a name="collect-the-necessary-values-to-use-the-rest-apis"></a>Сбор необходимых значений для использования REST API

Найдите и сохраните следующие значения:

* Идентификатор клиента:
  * На [портале Azure](https://portal.azure.com) найдите и выберите элемент **Azure Active Directory**.
  * В разделе **Управление** на панели слева выберите элемент **Свойства**, найдите **идентификатор клиента** и выберите значок **Копировать в буфер обмена**, чтобы сохранить его значение.
* Конечная точка Atlas:
  * На странице [Учетные записи Azure Purview](https://aka.ms/purviewportal) на портале Azure найдите и выберите из списка учетную запись Azure Purview.
  * Щелкните элемент **Свойства**, найдите **конечную точку Atlas** и щелкните значок **Копировать в буфер обмена**, чтобы сохранить ее значение. Удалите часть строки *https://* , когда позже будете ее использовать.
* Имя учетной записи.
  * Извлеките имя каталога из адреса конечной точки Atlas. Например, если конечная точка Atlas имеет адрес `https://ThisIsMyCatalog.catalog.purview.azure.com`, значит, учетная запись имеет имя `ThisIsMyCatalog`.

## <a name="use-the-postman-client-to-call-the-rest-apis"></a>Вызов REST API с помощью клиента Postman

1. Установите [клиент Postman](https://www.getpostman.com/).
1. В этом клиенте выберите действие **Import** (Импорт) и укажите файл [Test.postman_collection.json](https://raw.githubusercontent.com/Azure/Purview-Samples/master/rest-api/Test.postman_collection.json).
1. Выберите элемент **Collections** (Коллекции), а затем — элемент **Test** (Тест).
1. Щелкните элемент **Get Token** (Получить токен):
    1. В поле URL-адреса рядом с маркером POST замените заполнитель *&lt;your-tenant-id&gt;* реальным идентификатором клиента, который вы скопировали в предыдущем разделе.
    1. Выберите вкладку **Body** (Текст) и замените заполнители в пути и тексте значениями из предыдущего шага.

       Когда вы щелкнете **Send** (Отправить), в тексте ответа будет получена структура JSON с именем *access_token* и строковым значением в кавычках. 
    1. Скопируйте это значение токена носителя (без кавычек), чтобы использовать его на следующем этапе.

1. Выберите элемент **/v2/types/typedefs**:
    1. Замените заполнитель в пути реальным значением адреса конечной точки Atlas. Чтобы получить это значение, перейдите к экземпляру каталога на портале Ibiza и щелкните элемент Overview (Обзор). 

       Здесь задается токен носителя из первого шага (его также можно вручную скопировать на вкладку "Авторизация").

    1. Щелкните элемент **Send** (Отправить), чтобы получить ответ.

## <a name="generate-a-net-client-to-call-the-rest-apis"></a>Создание клиента .NET для вызова REST API

### <a name="install-autorest"></a>Установка AutoRest



1. [Установите Node.js](https://github.com/Azure/autorest/blob/v2/docs/installing-autorest.md).
1. Откройте PowerShell и выполните следующую команду:

   ```powershell
   npm install -g autorest@V2
   ```

### <a name="download-rest-api-specszip-and-create-the-client"></a>Скачивание rest-api-specs.zip и создание клиента

1. Скачайте файл [rest-api-specs.zip](https://github.com/Azure/Purview-Samples/raw/master/rest-api/rest-api-specs.zip) и извлеките его содержимое.
1. Выполните следующую команду в PowerShell из извлеченной папки rest-api-specs:

   ```powershell
   autorest --input-file=data-plane/preview/purviewcatalog.json --csharp --output-folder=PurviewCatalogClient/CSharp --namespace=PurviewCatalog --add-credentials
   ```

   В папке rest-api-specs будут созданы папка PurviewCatalogClient и подпапка CSharp.

### <a name="create-a-sample-net-console-application"></a>Создание примера консольного приложения .NET

1. Откройте Visual Studio 2019. Эти инструкции были протестированы с бесплатным выпуском сообщества.
1. На странице **Create a new project** (Создание нового проекта) создайте проект **Console App (.NET Core)** (Консольное приложение для .NET Core) на C#.
1. Скопируйте и вставьте предоставленный [пример кода](#sample-code-for-the-console-application).
1. Замените заполнители *accountName*, *servicePrincipalId*, *servicePrincipalKey* и *tenantId* собранными ранее реальными значениями.
1. С помощью **Обозревателя решений** добавьте в проект Visual Studio папку с именем **Generated**.
1. Скопируйте созданную ранее папку rest-api-specs\PurviewCatalogClient\CSharp в новую папку \Generated. С помощью проводника или окна командной строки выполните операцию копирования, которая активирует в Visual Studio автоматическое добавление файлов в проект.
1. В **Обозревателе решений** щелкните проект правой кнопкой мыши и выберите пункт **Управление пакетами NuGet**.
1. Выберите вкладку **обзора**. Найдите и выберите элемент **Microsoft.Rest.ClientRuntime**.
1. Убедитесь, что используется версия не ниже 2.3.21, а затем щелкните элемент **Install** (Установить).
1. Создайте и запустите приложение.

Этот пример кода возвращает количество определений типов в каталоге и демонстрирует управление назначениями ролей. Дополнительные сведения см. в функции `DoRoleAssignmentOperations()` в этом примере кода. Дополнительные сведения о проекте см. в статье [Настройка проекта](https://github.com/Azure/autorest/blob/v2/docs/client/proj-setup.md).

### <a name="sample-code-for-the-console-application"></a>Пример кода для консольного приложения

```csharp
using Microsoft.Rest;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net.Http;

namespace PurviewCatalogSdkTest
{
    class Program
    {
        private static string accountName = "{account-name}";
        private static string servicePrincipalId = "{service-principal-id}";
        private static string servicePrincipalKey = "{service-principal-key}";
        private static string tenantId = "{tenant-id}";

        static void Main(string[] args)
        {
            Console.WriteLine("Azure Purview client");

            // You need to change the api path below (e.g. /api) based on what you're trying to call
            string baseUri = string.Format("https://{0}.catalog.purview.azure.com/api", accountName);

            // Get token and set auth
            var svcClientCreds = new TokenCredentials(getToken(), "Bearer");
            var client = new PurviewCatalog.PurviewCatalogServiceRESTAPIDocument(svcClientCreds);
            client.BaseUri = new System.Uri(baseUri);

            // /v2/types/typedefs
            var task = client.TypesREST.GetAllTypeDefsWithHttpMessagesAsync();
            Console.WriteLine("\nURI:\n" + task.Result.Request.RequestUri);
            Console.WriteLine("\nStatus Code:\n" + task.Result.Response.StatusCode);
            Console.WriteLine("\nResult:\n" + JsonConvert.SerializeObject(task.Result.Body, Formatting.Indented));
        }

        // Replace client_id and client_secret with application ID and password value from service principal
        private static string getToken()
        {
            var values = new Dictionary<string, string>

            // The "resource" could be "https://purview.azure.net" or "73c2949e-da2d-457a-9607-fcc665198967"
            {
                { "grant_type", "client_credentials" },
                { "client_id", servicePrincipalId },
                { "client_secret", servicePrincipalKey },
                { "resource", "73c2949e-da2d-457a-9607-fcc665198967" }
            };

            string authUrl = string.Format("https://login.windows.net/{0}/oauth2/token", tenantId);
            var content = new FormUrlEncodedContent(values);

            HttpClient authClient = new HttpClient();
            var bearerResult = authClient.PostAsync(authUrl, content);
            bearerResult.Wait();
            var resultContent = bearerResult.Result.Content.ReadAsStringAsync();
            resultContent.Wait();
            var bearerToken = JObject.Parse(resultContent.Result)["access_token"].ToString();
            var svcClientCreds = new TokenCredentials(bearerToken, "Bearer");

            return bearerToken;
        }
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Управление источниками данных](manage-data-sources.md)
