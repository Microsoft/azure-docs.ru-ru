---
title: 'ML Studio (классическая модель): использование веб-служб в Azure'
description: После развертывания службы машинного обучения из Машинное обучение Azure Studio (классической) веб-службу RESTFUL можно использовать как службу запросов и ответов в режиме реального времени либо как службу пакетного выполнения.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio-classic
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18, devx-track-python, devx-track-js, devx-track-csharp
ms.date: 05/29/2020
ms.openlocfilehash: 0dc49265c0ea799e194e4ac7004b558d8a9d4dd8
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100519275"
---
# <a name="how-to-consume-a-machine-learning-studio-classic-web-service"></a>Как использовать веб-службу Машинное обучение Studio (классическая модель)

**ПРИМЕНИМО К:**  ![Знак "флажок", который означает, что статья применима к Студии машинного обучения (классической).](../../../includes/media/aml-applies-to-skus/yes.png) Студия машинного обучения (классическая)   ![Это знак "X", который означает, что статья применима к Машинному обучению Azure.](../../../includes/media/aml-applies-to-skus/no.png)[Машинное обучение Azure](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)


После развертывания прогнозной модели Машинное обучение Azure Studio (классической) в качестве веб-службы можно использовать REST API для отправки данных и получения прогнозов. Данные можно отправлять в режиме реального времени или пакетном режиме.

Дополнительные сведения о создании и развертывании Машинное обучение веб-службы с помощью Машинное обучение Studio (классическая модель) см. здесь:

* Руководство по созданию эксперимента в Машинное обучение Studio (классическая модель) см. в статье [Создание первого эксперимента](create-experiment.md).
* Дополнительные сведения о развертывании веб-службы см. в [этой статье](deploy-a-machine-learning-web-service.md).
* Дополнительные сведения о машинном обучении см. в [центре документации по Машинному обучению Azure](https://azure.microsoft.com/documentation/services/machine-learning/).



## <a name="overview"></a>Обзор
С помощью веб-службы машинного обучения Azure внешнее приложение взаимодействует с рабочим процессом машинного обучения, оценивая модель в режиме реального времени. Вызов веб-службы машинного обучения возвращает результаты прогноза внешнему приложению. Чтобы вызвать веб-службу машинного обучения, следует передать ключ API, создаваемый при развертывании прогноза. Веб-служба машинного обучения работает на основе архитектуры REST, используемой в основном в проектах с веб-программированием.

Машинное обучение Azure Studio (классическая модель) имеет два типа служб:

* Служба Request-Response (RR) — низкая задержка, Высокомасштабируемая служба, предоставляющая интерфейс для моделей без отслеживания состояния, созданных и развернутых из среды Машинное обучение Studio (классическая модель).
* Служба пакетного выполнения (BES) — асинхронная служба, оценивающая пакет записей данных.

Дополнительные сведения о Машинное обучение веб-служб см. [в разделе Развертывание веб-службы машинное обучение](deploy-a-machine-learning-web-service.md).

## <a name="get-an-authorization-key"></a>Получение ключа авторизации
При развертывании эксперимента создаются ключи API для веб-службы. Ключи можно извлечь из нескольких расположений.

### <a name="from-the-microsoft-azure-machine-learning-web-services-portal"></a>Из портала веб-службы Машинного обучения Microsoft Azure
Войдите на портал [веб-служб машинное обучение Microsoft Azure](https://services.azureml.net) .

Чтобы получить ключ API для новой веб-службы машинного обучения, сделайте следующее:

1. На портале веб-служб Машинного обучения Azure в верхнем меню щелкните **Веб-службы**.
2. Щелкните веб-службу, для которой требуется получить ключ.
3. В верхнем меню щелкните **Consume**(Использование).
4. Скопируйте и сохраните **Primary Key**(Первичный ключ).

Чтобы получить ключ API для классической веб-службы машинного обучения, сделайте следующее:

1. На портале веб-служб Машинного обучения Azure в верхнем меню щелкните **Classic Web Services** (Классические веб-службы).
2. Выберите веб-службу, с которой вы работаете.
3. Щелкните конечную точку, для которой требуется получить ключ.
4. В верхнем меню щелкните **Consume**(Использование).
5. Скопируйте и сохраните **Primary Key**(Первичный ключ).

### <a name="classic-web-service"></a>Классическая веб-служба
 Ключ для классической веб-службы также можно получить из Машинное обучение Studio (классическая модель).

#### <a name="machine-learning-studio-classic"></a>Студия машинного обучения (классическая версия)
1. В Машинное обучение Studio (классическая модель) щелкните **веб-службы** слева.
2. Щелкните веб-службу. **Ключ API** указан на вкладке **Панель мониторинга**.

## <a name="connect-to-a-machine-learning-web-service"></a><a id="connect"></a>Подключение к веб-службе Машинное обучение
К веб-службе машинного обучения можно подключиться, используя любой язык программирования, поддерживающий HTTP-запрос и HTTP-ответ. Примеры на языках C#, Python и R можно просмотреть на странице справки веб-службы машинного обучения.

**Справка по API машинного обучения.** Страница справки по API машинного обучения Azure создается при развертывании веб-службы. См. [учебник 3. Развертывание модели кредитных рисков](tutorial-part3-credit-risk-deploy.md).
Справка по API машинного обучения содержит сведения о веб-службе прогнозирования.

1. Выберите веб-службу, с которой вы работаете.
2. Щелкните конечную точку, для которой требуется просмотреть страницу справки по API.
3. В верхнем меню щелкните **Consume**(Использование).
4. Щелкните **API help page** (Страница справки API) под конечными точками "Запрос — ответ" или "Выполнение пакета".

**Просмотр справки по API машинного обучения для новой веб-службы**

На [портале веб-служб машинное обучение Azure](https://services.azureml.net/):

1. В верхнем меню щелкните **WEB SERVICES** (ВЕБ-СЛУЖБЫ).
2. Щелкните веб-службу, для которой требуется получить ключ.

Щелкните **Use Web Service** (Использовать веб-службу), чтобы получить значения URI для службы "запрос и ответ" и службы пакетного выполнения, а также для примеров кода на C#, R и Python.

Щелкните **Swagger API**, чтобы получить документацию на основе Swagger для интерфейсов API, вызываемых из передаваемых URI.

### <a name="c-sample"></a>Пример на языке C#
Чтобы подключиться к веб-службе Машинное обучение, используйте **HttpClient** , передающий ScoreData. Данные набора содержат Вектор свойств, n-мерный вектор числовых параметров, представляющий Данные набора. Службу машинного обучения можно аутентифицировать с помощью ключа API.

Чтобы подключиться к веб-службе Машинное обучение, необходимо установить пакет NuGet **Microsoft. AspNet. WebApi. Client** .

**Установка Microsoft. AspNet. WebApi. Client NuGet в Visual Studio**

1. Опубликуйте набор данных Загрузки из набора UCI: Adult 2 веб-службы.
2. Щелкните **Сервис** диспетчер  >  **пакетов NuGet**  >  **консоль диспетчера пакетов**.
3. Выберите **Установить пакет Microsoft.AspNet.WebApi.Client**.

**Для запуска примера выполните следующие действия:**

1. Опубликуйте эксперимент "Пример 1. Загрузка набора данных из UCI: набор данных для класса Adult 2", входящий в набор примеров машинного обучения Azure.
2. Назначьте apiKey ключ из веб-службы. См. **раздел Получение ключа авторизации** выше.
3. Назначьте serviceUri универсальный код ресурса запроса.

**Вот как будет выглядеть полный запрос:**
```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Net.Http;
using System.Net.Http.Formatting;
using System.Net.Http.Headers;
using System.Text;
using System.Threading.Tasks;

namespace CallRequestResponseService
{
    class Program
    {
        static void Main(string[] args)
        {
            InvokeRequestResponseService().Wait();
        }

        static async Task InvokeRequestResponseService()
        {
            using (var client = new HttpClient())
            {
                var scoreRequest = new
                {
                    Inputs = new Dictionary<string, List<Dictionary<string, string>>> () {
                        {
                            "input1",
                            // Replace columns labels with those used in your dataset
                            new List<Dictionary<string, string>>(){new Dictionary<string, string>(){
                                    {
                                        "column1", "value1"
                                    },
                                    {
                                        "column2", "value2"
                                    },
                                    {
                                        "column3", "value3"
                                    }
                                }
                            }
                        },
                    },
                    GlobalParameters = new Dictionary<string, string>() {}
                };

                // Replace these values with your API key and URI found on https://services.azureml.net/
                const string apiKey = "<your-api-key>"; 
                const string apiUri = "<your-api-uri>";
                
                client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue( "Bearer", apiKey);
                client.BaseAddress = new Uri(apiUri);

                // WARNING: The 'await' statement below can result in a deadlock
                // if you are calling this code from the UI thread of an ASP.NET application.
                // One way to address this would be to call ConfigureAwait(false)
                // so that the execution does not attempt to resume on the original context.
                // For instance, replace code such as:
                //      result = await DoSomeTask()
                // with the following:
                //      result = await DoSomeTask().ConfigureAwait(false)

                HttpResponseMessage response = await client.PostAsJsonAsync("", scoreRequest);

                if (response.IsSuccessStatusCode)
                {
                    string result = await response.Content.ReadAsStringAsync();
                    Console.WriteLine("Result: {0}", result);
                }
                else
                {
                    Console.WriteLine(string.Format("The request failed with status code: {0}", response.StatusCode));

                    // Print the headers - they include the request ID and the timestamp,
                    // which are useful for debugging the failure
                    Console.WriteLine(response.Headers.ToString());

                    string responseContent = await response.Content.ReadAsStringAsync();
                    Console.WriteLine(responseContent);
                }
            }
        }
    }
}
```

### <a name="python-sample"></a>Пример на Python
Чтобы подключиться к веб-службе машинного обучения, используйте библиотеку **urllib2** для Python 2.X и библиотеку **urllib.request** для Python 3.X. Вы передадите данные ScoreData, содержащие FeatureVector — n-мерный вектор числовых параметров, представляющий ScoreData. Службу машинного обучения можно аутентифицировать с помощью ключа API.

**Для запуска примера выполните следующие действия:**

1. Разверните эксперимент "Sample 1: Download dataset from UCI: Adult 2 class dataset" (Пример 1. Скачивание набора данных из UCI: набор данных для класса Adult 2), входящий в набор примеров машинного обучения.
2. Назначьте apiKey ключ из веб-службы. Ознакомьтесь с разделом **Получение ключа авторизации** в начале этой статьи.
3. Назначьте serviceUri универсальный код ресурса запроса.

**Вот как будет выглядеть полный запрос:**
```python
import urllib2 # urllib.request and urllib.error for Python 3.X
import json

data = {
    "Inputs": {
        "input1":
        [
            {
                'column1': "value1",   
                'column2': "value2",   
                'column3': "value3"
            }
        ],
    },
    "GlobalParameters":  {}
}

body = str.encode(json.dumps(data))

# Replace this with the URI and API Key for your web service
url = '<your-api-uri>'
api_key = '<your-api-key>'
headers = {'Content-Type':'application/json', 'Authorization':('Bearer '+ api_key)}

# "urllib.request.Request(url, body, headers)" for Python 3.X
req = urllib2.Request(url, body, headers)

try:
    # "urllib.request.urlopen(req)" for Python 3.X
    response = urllib2.urlopen(req)

    result = response.read()
    print(result)
# "urllib.error.HTTPError as error" for Python 3.X
except urllib2.HTTPError, error: 
    print("The request failed with status code: " + str(error.code))

    # Print the headers - they include the request ID and the timestamp, which are useful for debugging the failure
    print(error.info())
    print(json.loads(error.read())) 
```

### <a name="r-sample"></a>Пример на языке R

Чтобы подключиться к веб-службе машинного обучения, используйте для выполнения запроса и обработки ответа в формате JSON библиотеки **RCurl** и **rjson**. Вы передадите данные ScoreData, содержащие FeatureVector — n-мерный вектор числовых параметров, представляющий ScoreData. Службу машинного обучения можно аутентифицировать с помощью ключа API.

**Вот как будет выглядеть полный запрос:**
```r
library("curl")
library("httr")
library("rjson")

requestFailed = function(response) {
    return (response$status_code >= 400)
}

printHttpResult = function(response, result) {
    if (requestFailed(response)) {
        print(paste("The request failed with status code:", response$status_code, sep=" "))
    
        # Print the headers - they include the request ID and the timestamp, which are useful for debugging the failure
        print(response$headers)
    }
    
    print("Result:") 
    print(fromJSON(result))  
}

req = list(
        Inputs = list( 
            "input1" = list(
                "ColumnNames" = list("Col1", "Col2", "Col3"),
                "Values" = list( list( "0", "value", "0" ),  list( "0", "value", "0" )  )
            )                ),
        GlobalParameters = setNames(fromJSON('{}'), character(0))
)

body = enc2utf8(toJSON(req))
api_key = "abc123" # Replace this with the API key for the web service
authz_hdr = paste('Bearer', api_key, sep=' ')

response = POST(url= "<your-api-uri>",
        add_headers("Content-Type" = "application/json", "Authorization" = authz_hdr),
        body = body)

result = content(response, type="text", encoding="UTF-8")

printHttpResult(response, result)
```

### <a name="javascript-sample"></a>Пример на языке JavaScript

Чтобы подключиться к веб-службе машинного обучения, используйте в своем проекте пакет npm **request**. Кроме того, для форматирования входных данных и анализа результата вы будете использовать объект `JSON`. Выполните установку с помощью `npm install request --save` или добавьте `"request": "*"` в файл package.json в разделе `dependencies` и запустите `npm install`.

**Вот как будет выглядеть полный запрос:**
```js
let req = require("request");

const uri = "<your-api-uri>";
const apiKey = "<your-api-key>";

let data = {
    "Inputs": {
        "input1":
        [
            {
                'column1': "value1",
                'column2': "value2",
                'column3': "value3"
            }
        ],
    },
    "GlobalParameters": {}
}

const options = {
    uri: uri,
    method: "POST",
    headers: {
        "Content-Type": "application/json",
        "Authorization": "Bearer " + apiKey,
    },
    body: JSON.stringify(data)
}

req(options, (err, res, body) => {
    if (!err && res.statusCode == 200) {
        console.log(body);
    } else {
        console.log("The request failed with status code: " + res.statusCode);
    }
});
```