---
title: Начало работы с функцией перевода документов
description: Создание службы преобразования документов с помощью языков программирования C#, Go, Java, Node.js или Python
ms.topic: how-to
manager: nitinme
ms.author: lajanuar
author: laujan
ms.date: 03/05/2021
ms.openlocfilehash: 21df853d9b1c7250e9a6eea37a68835a180f610d
ms.sourcegitcommit: 2c1b93301174fccea00798df08e08872f53f669c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104773051"
---
# <a name="get-started-with-document-translation-preview"></a>Начало работы с переводом документа (Предварительная версия)

 В этой статье вы узнаете, как использовать преобразование документов с помощью методов HTTP REST API. Преобразование документов — это облачная функция службы [Azure Translator](../translator-info-overview.md) .  API перевода документов позволяет переводить целые документы, сохраняя структуру исходного документа и форматирование текста.

## <a name="prerequisites"></a>Предварительные требования

> [!NOTE]
>
> 1. Как правило, при создании ресурса "неработающий службы" в портал Azure можно создать ключ подписки с несколькими службами или ключ подписки с одной службой. Однако преобразование документов в настоящее время поддерживается только в ресурсе Translator (только одна служба) и **не** включено в ресурс Cognitive Services (несколько служб).
> 2. В настоящее время преобразование документов доступно в **стандартном плане обслуживания S1**. _См. раздел_ [цены на Cognitive Services — переводчик](https://azure.microsoft.com/pricing/details/cognitive-services/translator/).
>

Чтобы приступить к работе, вам потребуется:

* Активная [**учетная запись Azure**](https://azure.microsoft.com/free/cognitive-services/).  Если у вас ее нет, можно [**создать бесплатную учетную запись**](https://azure.microsoft.com/free/).

* Ресурс службы [**переводчика**](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextTranslation) (**не** ресурс Cognitive Services).

* [**Учетная запись хранилища BLOB-объектов Azure**](https://ms.portal.azure.com/#create/Microsoft.StorageAccount-ARM). Вы создадите контейнеры для хранения и организации данных больших двоичных объектов в вашей учетной записи хранения.

## <a name="get-your-custom-domain-name-and-subscription-key"></a>Получение имени пользовательского домена и ключа подписки

> [!IMPORTANT]
>
> * Вы не будете использовать конечную точку, найденную в портал Azure _ключей ресурсов и конечной_ точке глобального преобразователя, — `api.cognitive.microsofttranslator.com` для выполнения HTTP-запросов к преобразованию документов.
> * **Для всех запросов API к службе перевода документов требуется Пользовательская конечная точка домена**.

### <a name="what-is-the-custom-domain-endpoint"></a>Что такое конечная точка пользовательского домена?

Конечная точка пользовательского домена — это URL-адрес, отформатированный с учетом имени ресурса, имени узла и подкаталогов переводчика:

```http
https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1
```

### <a name="find-your-custom-domain-name"></a>Поиск имени пользовательского домена

Параметр **имя ресурса** (также называемый *пользовательским именем домена*) — это значение, введенное в поле **имя** при создании ресурса переводчика.

:::image type="content" source="../media/instance-details.png" alt-text="Изображение портал Azure, создание ресурса, мгновенная информация, поле имени.":::

### <a name="get-your-subscription-key"></a>Получение ключа подписки

Для запросов к службе переводчика требуется ключ только для чтения для проверки подлинности доступа.

1. Если вы создали новый ресурс, после развертывания выберите **Переход к ресурсу**. Если у вас есть ресурс перевода документа, перейдите непосредственно на страницу ресурсов.
1. На левой границе в разделе *Управление ресурсами* выберите **ключи и конечная точка**.
1. Скопируйте и вставьте ключ подписки в удобном месте, например в *Microsoft Notepad*.
1. Вставьте его в приведенный ниже код, чтобы проверить подлинность запроса к службе перевода документов.

:::image type="content" source="../media/translator-keys.png" alt-text="Изображение поля получить ключ подписки в портал Azure.":::

## <a name="create-your-azure-blob-storage-containers"></a>Создание контейнеров хранилища BLOB-объектов Azure

Вам потребуется  [**создать контейнеры**](../../../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container) в [**учетной записи хранилища BLOB-объектов Azure**](https://ms.portal.azure.com/#create/Microsoft.StorageAccount-ARM) для исходных, целевых и необязательных файлов глоссария.

* **Исходный контейнер**. В этом контейнере вы отправляете файлы для перевода (обязательно).
* **Целевой контейнер**. В этом контейнере будут храниться переведенные файлы (обязательно).  
* **Контейнер глоссария**. В этом контейнере вы отправляете файлы глоссария (необязательно).  

### <a name="create-sas-access-tokens-for-document-translation"></a>**Создание маркеров доступа SAS для перевода документов**

Параметр `sourceUrl` , `targetUrl` и необязательный `glossaryUrl`  должен включать маркер подписанного URL-адрес (SAS), добавленный в виде строки запроса. Маркер может быть назначен контейнеру или конкретным BLOB-объектам. *См. раздел* [**Создание маркеров SAS для процесса перевода документов**](create-sas-tokens.md).

* **Исходный** контейнер или большой двоичный объект должен иметь назначенный доступ для **чтения** и получения **списка** .
* **Целевой** контейнер или большой двоичный объект должен иметь назначенный доступ на **запись** и **список** .
* Контейнер **глоссария** или большой двоичный объект должен иметь назначенный доступ для  **чтения** и просмотра **списка** .

> [!TIP]
>
> * При переводе **нескольких** файлов (больших двоичных объектов) в операции **делегируйте доступ SAS на уровне контейнера**.  
> * При переводе **одного** файла (большого двоичного объекта) в операцию **делегируйте доступ SAS на уровне BLOB-объектов**.  
>

## <a name="set-up-your-coding-platform"></a>Настройка платформы кодирования

### <a name="c"></a>[C#](#tab/csharp)

* Создайте новый проект.
* Замените файл Program.cs кодом C#, указанным ниже.
* Задайте конечную точку. ключ подписки и значения URL-адресов контейнеров в Program. cs.
* Для обработки данных JSON добавьте [Newtonsoft.Jsв пакет с помощью интерфейса командной строки .NET](https://www.nuget.org/packages/Newtonsoft.Json/).
* Запустите программу из каталога проекта.

### <a name="nodejs"></a>[Node.js](#tab/javascript)

* Создайте новый проект Node.js.
* Установите библиотеку Аксиос с помощью `npm i axios` .
* Скопируйте приведенный ниже код в проект.
* Задайте значения конечная точка, ключ подписки и URL-адрес контейнера.
* Запустите программу.

### <a name="python"></a>[Python](#tab/python)  

* Создайте новый проект.
* Скопируйте и вставьте код из одного из примеров в проект.
* Задайте значения конечная точка, ключ подписки и URL-адрес контейнера.
* Запустите программу. Например: `python translate.py`.

### <a name="java"></a>[Java](#tab/java)

* Создайте рабочий каталог для своего проекта. Пример:

```powershell
mkdir sample-project
```

* В каталоге проекта создайте следующую структуру подкаталогов:  

  src</br>
&emsp; └ Main</br>
&emsp;&emsp;&emsp;└ Java

```powershell
mkdir -p src/main/java/
```

**Примечание**. исходные файлы Java (например, _Sample. Java_) в режиме «src/Main/**Java**».

* В корневом каталоге (например, *Sample-Project*) инициализируйте проект с помощью Gradle:

```powershell
gradle init --type basic
```

* Когда появится запрос на выбор **предметно-ориентированного языка**, выберите **Kotlin**.

* Обновите `build.gradle.kts`  файл. Помните, что вам потребуется обновить в `mainClassName` зависимости от примера:

  ```java
  plugins {
    java
    application
  }
  application {
    mainClassName = "{NAME OF YOUR CLASS}"
  }
  repositories {
    mavenCentral()
  }
  dependencies {
    compile("com.squareup.okhttp:okhttp:2.5.0")
  }
  ```

* Создайте файл Java в каталоге **Java** и скопируйте и вставьте код из предоставленного примера. Не забудьте добавить ключ и конечную точку подписки.

* **Выполните сборку и запустите пример из корневого каталога**:

```powershell
gradle build
gradle run
```

### <a name="go"></a>[GO](#tab/go)  

* Создайте новый проект go.
* Добавьте указанный ниже код.
* Задайте значения конечная точка, ключ подписки и URL-адрес контейнера.
* Сохраните файл с расширением .go.
* Откройте командную строку на компьютере, на котором установлен язык Go.
* Скомпилируйте файл, например "go build example-code.go".
* Запустите файл, например "example-code".

 ---

## <a name="make-document-translation-requests"></a>Создание запросов на перевод документов

Запрос на перевод пакетного документа отправляется в конечную точку службы переводчиков через запрос POST. В случае успеха метод POST возвращает `202 Accepted`  код ответа, и пакетный запрос создается службой.

### <a name="http-headers"></a>Заголовки HTTP

В каждый запрос API-интерфейса транслятора документов включены следующие заголовки:

|Заголовок HTTP|Описание|
|---|--|
|Ocp-Apim-Subscription-Key|**Обязательно**. значение — это ключ подписки Azure для вашего транслятора или ресурса Cognitive Services.|
|Content-Type|**Обязательный**: указывает тип содержимого полезных данных. Допустимые значения: Application/JSON или charset = UTF-8.|
|Content-Length|**Обязательный**: длина текста запроса.|

### <a name="post-request-body-properties"></a>Свойства текста запроса POST

* Текст запроса POST представляет собой объект JSON с именем `inputs` .
* `inputs`Объект содержит как `sourceURL` , так и адрес `targetURL` контейнера для пар исходного и целевого языка, а также может содержать `glossaryURL` адрес контейнера.
* `prefix`Поля и `suffix` (необязательно) используются для фильтрации документов в контейнере, включая папки.
* Значение  `glossaries`  поля (необязательно) применяется при переводе документа.
* `targetUrl`Для каждого целевого языка должно быть уникальное значение.

>[!NOTE]
> Если файл с таким именем уже существует в назначении, он будет перезаписан.

## <a name="post-a-translation-request"></a>Публикация запроса на перевод

<!-- markdownlint-disable MD024 -->
### <a name="post-request-body-without-optional-glossaryurl"></a>Тело запроса POST без необязательных Глоссарюрл

```json
{
    "inputs": [
        {
            "source": {
                "sourceUrl": "<https://YOUR-SOURCE-URL-WITH-READ-LIST-ACCESS-SAS>",
                "storageSource": "AzureBlob",
                "filter": {
                    "prefix": "News",
                    "suffix": ".txt"
                },
                "language": "en"
            },
            "targets": [
                {
                    "targetUrl": "<https://YOUR-SOURCE-URL-WITH-WRITE-LIST-ACCESS-SAS>",
                    "storageSource": "AzureBlob",
                    "category": "general",
                    "language": "de"
                }
            ]
        }
    ]
}
```

### <a name="post-request-body-with-optional-glossaryurl"></a>Тело запроса POST с необязательным Глоссарюрл

```json
{
  "inputs":[
    {
      "source":{
        "sourceUrl":"<https://YOUR-SOURCE-URL-WITH-READ-LIST-ACCESS-SAS>",
        "storageSource":"AzureBlob",
        "filter":{
          "prefix":"News",
          "suffix":".txt"
        },
        "language":"en"
      },
      "targets":[
        {
          "targetUrl":"<https://YOUR-SOURCE-URL-WITH-WRITE-LIST-ACCESS-SAS>",
          "storageSource":"AzureBlob",
          "category":"general",
          "language":"de",
          "glossaries":[
            {
              "glossaryUrl":"<https://YOUR-GLOSSARY-URL-WITH-READ-LIST-ACCESS-SAS>",
              "format":"xliff",
              "version":"1.2"
            }
          ]
        }
      ]
    }
  ]
}
```

> [!IMPORTANT]
>
> В приведенных ниже примерах кода вы жестко задаете ключ и конечную точку, где указано; не забудьте удалить ключ из кода по завершении и никогда не публиковать его в общедоступном виде.  Способы безопасного хранения и доступа к учетным данным см. в статье [Безопасность Azure Cognitive Services](/azure/cognitive-services/cognitive-services-security?tabs=command-line%2Ccsharp) .
>
> Может потребоваться обновить следующие поля в зависимости от операции:
>>>
>> * `endpoint`
>> * `subscriptionKey`
>> * `sourceURL`
>> * `targetURL`
>> * `glossaryURL`
>> * `id`  (ИД задания)
>>

#### <a name="locating--the-id-value"></a>Поиск `id` значения

* Задание можно найти `id`  в `Operation-Location`  значении URL-адреса заголовка ответа метода POST. Последний параметр URL-адреса — это задание операции **`id`** :

|**Заголовок ответа**|**URL-адрес результата**|
|-----------------------|----------------|
Operation-Location   | https://<<span>имя вашего ресурса>. cognitiveservices.Azure.com/Translator/Text/Batch/v1.0-Preview.1/BATCHES/9dce0aa9-78dc-41ba-8cae-2e2f3c2ff8ec</span>

* Для получения задания перевода документа можно также использовать запрос на **Получение заданий** `id` .

>

## <a name="_post-document-translation_-request"></a>_Отправить запрос на перевод документа_

Отправьте запрос на перевод пакетного документа в службу перевода.

### <a name="c"></a>[C#](#tab/csharp)

```csharp

    using System;
    using System.Net.Http;
    using System.Threading.Tasks;
    using System.Text;
    

    class Program
    {

        static readonly string route = "/batches";

        private static readonly string endpoint = "https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1";

        private static readonly string subscriptionKey = "<YOUR-SUBSCRIPTION-KEY>";

        static readonly string json = ("{\"inputs\": [{\"source\": {\"sourceUrl\": \"https://YOUR-SOURCE-URL-WITH-READ-LIST-ACCESS-SAS",\"storageSource\": \"AzureBlob\",\"language\": \"en\",\"filter\":{\"prefix\": \"Demo_1/\"} }, \"targets\": [{\"targetUrl\": \"https://YOUR-TARGET-URL-WITH-WRITE-LIST-ACCESS-SAS\",\"storageSource\": \"AzureBlob\",\"category\": \"general\",\"language\": \"es\"}]}]}");
        
        static async Task Main(string[] args)
        {
            using HttpClient client = new HttpClient();
            using HttpRequestMessage request = new HttpRequestMessage();
            {
            
                StringContent content = new StringContent(json, Encoding.UTF8, "application/json");

                request.Method = HttpMethod.Post;
                request.RequestUri = new Uri(endpoint + route);
                request.Headers.Add("Ocp-Apim-Subscription-Key", subscriptionKey);
                request.Content = content;
                
                HttpResponseMessage  response = await client.SendAsync(request);
                string result = response.Content.ReadAsStringAsync().Result;
                if (response.IsSuccessStatusCode)
                {
                    Console.WriteLine($"Status code: {response.StatusCode}");
                    Console.WriteLine();
                    Console.WriteLine($"Response Headers:"); 
                    Console.WriteLine(response.Headers);
                }
                else
                    Console.Write("Error");

            }

        }

    }
}
```

### <a name="nodejs"></a>[Node.js](#tab/javascript)

```javascript

const axios = require('axios').default;

let endpoint = 'https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1';
let route = '/batches';
let subscriptionKey = '<YOUR-SUBSCRIPTION-KEY>';

let data = JSON.stringify({"inputs": [
  {
      "source": {
          "sourceUrl": "https://YOUR-SOURCE-URL-WITH-READ-LIST-ACCESS-SAS",
          "storageSource": "AzureBlob",
          "language": "en",
          "filter":{
              "prefix": "Demo_1/"
          }
      },
      "targets": [
          {
              "targetUrl": "https://YOUR-TARGET-URL-WITH-WRITE-LIST-ACCESS-SAS",
              "storageSource": "AzureBlob",
              "category": "general",
              "language": "es"}]}]});

let config = {
  method: 'post',
  baseURL: endpoint,
  url: route,
  headers: {
    'Ocp-Apim-Subscription-Key': subscriptionKey,
    'Content-Type': 'application/json'
  },
  data: data
};

axios(config)
.then(function (response) {
  let result = { statusText: response.statusText, statusCode: response.status, headers: response.headers };
  console.log()
  console.log(JSON.stringify(result));
})
.catch(function (error) {
  console.log(error);
});
```

### <a name="python"></a>[Python](#tab/python)

```python

import requests

endpoint = "https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1"
subscriptionKey =  '<YOUR-SUBSCRIPTION-KEY>'
path = '/batches'
constructed_url = endpoint + path

payload= {
    "inputs": [
        {
            "source": {
                "sourceUrl": "https://YOUR-SOURCE-URL-WITH-READ-LIST-ACCESS-SAS",
                "storageSource": "AzureBlob",
                "language": "en",
                "filter":{
                    "prefix": "Demo_1/"
                }
            },
            "targets": [
                {
                    "targetUrl": "https://YOUR-TARGET-URL-WITH-WRITE-LIST-ACCESS-SAS",
                    "storageSource": "AzureBlob",
                    "category": "general",
                    "language": "es"
                }
            ]
        }
    ]
}
headers = {
  'Ocp-Apim-Subscription-Key': subscriptionKey,
  'Content-Type': 'application/json'
}

response = requests.post(constructed_url, headers=headers, json=payload)

print(f'response status code: {response.status_code}\nresponse status: {response.reason}\nresponse headers: {response.headers}')
```

### <a name="java"></a>[Java](#tab/java)

```java

import java.io.*;
import java.net.*;
import java.util.*;
import com.squareup.okhttp.*;

public class DocumentTranslation {
    String subscriptionKey = "'<YOUR-SUBSCRIPTION-KEY>'";
    String endpoint = "https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1";
    String path = endpoint + "/batches";

    OkHttpClient client = new OkHttpClient();

    public void post() throws IOException {
        MediaType mediaType = MediaType.parse("application/json");
        RequestBody body = RequestBody.create(mediaType,  "{\n \"inputs\": [\n {\n \"source\": {\n \"sourceUrl\": \"https://YOUR-SOURCE-URL-WITH-READ-LIST-ACCESS-SAS\",\n \"filter\": {\n  \"prefix\": \"Demo_1\"\n  },\n  \"language\": \"en\",\n \"storageSource\": \"AzureBlob\"\n  },\n \"targets\": [\n {\n \"targetUrl\": \"https://YOUR-TARGET-URL-WITH-WRITE-LIST-ACCESS-SAS\",\n \"category\": \"general\",\n\"language\": \"fr\",\n\"storageSource\": \"AzureBlob\"\n }\n ],\n \"storageType\": \"Folder\"\n }\n  ]\n}");
        Request request = new Request.Builder()
                .url(path).post(body)
                .addHeader("Ocp-Apim-Subscription-Key", subscriptionKey)
                .addHeader("Content-type", "application/json")
                .build();
        Response response = client.newCall(request).execute();
        System.out.println(response.code());
        System.out.println(response.headers());
    }

  public static void main(String[] args) {
        try {
            DocumentTranslation sampleRequest = new DocumentTranslation();
            sampleRequest.post();
        } catch (Exception e) {
            System.out.println(e);
        }
    }
}
```

### <a name="go"></a>[GO](#tab/go)

```go

package main

import (
 "bytes"
 "fmt"
"net/http"
)

func main() {
endpoint := "https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1"
subscriptionKey := "<YOUR-SUBSCRIPTION-KEY>"
uri := endpoint + "/batches"
method := "POST"

var jsonStr = []byte(`{"inputs":[{"source":{"sourceUrl":"https://YOUR-SOURCE-URL-WITH-READ-LIST-ACCESS-SAS","storageSource":"AzureBlob","language":"en","filter":{"prefix":"Demo_1/"}},"targets":[{"targetUrl":"https://YOUR-TARGET-URL-WITH-WRITE-LIST-ACCESS-SAS","storageSource":"AzureBlob","category":"general","language":"es"}]}]}`)

req, err := http.NewRequest(method, endpoint, bytes.NewBuffer(jsonStr))
req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)
req.Header.Add("Content-Type", "application/json")

client := &http.Client{}

if err != nil {
    fmt.Println(err)
    return
  }
  res, err := client.Do(req)
  if err != nil {
    fmt.Println(err)
    return
  }
  defer res.Body.Close()
  fmt.Println("response status:",  res.Status)
  fmt.Println("response headers",  res.Header)
}
```

---

## <a name="_get-file-formats_"></a>_ПОЛУЧЕНИЕ форматов файлов_ 

Получение списка поддерживаемых форматов файлов. В случае успеха этот метод возвращает `200 OK` код ответа.

### <a name="c"></a>[C#](#tab/csharp)

```csharp
   
using System;
using System.Net.Http;
using System.Threading.Tasks;


class Program
{


    private static readonly string endpoint = "https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1";

    static readonly string route = "/documents/formats";

    private static readonly string subscriptionKey = "<YOUR-SUBSCRIPTION-KEY>";

    static async Task Main(string[] args)
    {

        HttpClient client = new HttpClient();
            using HttpRequestMessage request = new HttpRequestMessage();
            {
                request.Method = HttpMethod.Get;
                request.RequestUri = new Uri(endpoint + route);
                request.Headers.Add("Ocp-Apim-Subscription-Key", subscriptionKey);


                HttpResponseMessage response = await client.SendAsync(request);
                string result = response.Content.ReadAsStringAsync().Result;

                Console.WriteLine($"Status code: {response.StatusCode}");
                Console.WriteLine($"Response Headers: {response.Headers}");
                Console.WriteLine();
                Console.WriteLine(result);
            }
}
```

### <a name="nodejs"></a>[Node.js](#tab/javascript)

```javascript

const axios = require('axios');

let endpoint = 'https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1';
let subscriptionKey = '<YOUR-SUBSCRIPTION-KEY>';
let route = '/documents/formats';

let config = {
  method: 'get',
  url: endpoint + route,
  headers: { 
    'Ocp-Apim-Subscription-Key': subscriptionKey
  }
};

axios(config)
.then(function (response) {
  console.log(JSON.stringify(response.data));
})
.catch(function (error) {
  console.log(error);
});

```

### <a name="java"></a>[Java](#tab/java)

```java
import java.io.*;
import java.net.*;
import java.util.*;
import com.squareup.okhttp.*;

public class GetFileFormats {

    String subscriptionKey = "<YOUR-SUBSCRIPTION-KEY>";
    String endpoint = "https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1";
    String url = endpoint + "/documents/formats";
    OkHttpClient client = new OkHttpClient();

    public void get() throws IOException {
        Request request = new Request.Builder().url(
                url).method("GET", null).addHeader("Ocp-Apim-Subscription-Key", subscriptionKey).build();
        Response response = client.newCall(request).execute(); 
            System.out.println(response.body().string());
        }

    public static void main(String[] args) throws IOException {
        try{
            GetJobs jobs = new GetJobs();
            jobs.get();
        } catch (Exception e) {
            System.out.println(e);
        }
    }
}

```

### <a name="python"></a>[Python](#tab/python)

```python

import http.client

host = '<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com'
parameters = '//translator/text/batch/v1.0-preview.1/documents/formats'
subscriptionKey =  '<YOUR-SUBSCRIPTION-KEY>'
conn = http.client.HTTPSConnection(host)
payload = ''
headers = {
  'Ocp-Apim-Subscription-Key': subscriptionKey
}
conn.request("GET", parameters , payload, headers)
res = conn.getresponse()
data = res.read()
print(res.status)
print()
print(data.decode("utf-8"))
```

### <a name="go"></a>[GO](#tab/go)

```go

package main

import (
  "fmt"
  "net/http"
  "io/ioutil"
)

func main() {

  endpoint := "https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1"
  subscriptionKey := "<YOUR-SUBSCRIPTION-KEY>"
  uri := endpoint + "/documents/formats"
  method := "GET"

  client := &http.Client {
  }
  req, err := http.NewRequest(method, uri, nil)

  if err != nil {
    fmt.Println(err)
    return
  }
  req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)

  res, err := client.Do(req)
  if err != nil {
    fmt.Println(err)
    return
  }
  defer res.Body.Close()

  body, err := ioutil.ReadAll(res.Body)
  if err != nil {
    fmt.Println(err)
    return
  }
  fmt.Println(res.StatusCode)
  fmt.Println(string(body))
}
```

---

## <a name="_get-job-status_"></a>_ПОЛУЧЕНИЕ состояния задания_ 

Получение текущего состояния для одного задания и сводка по всем заданиям в запросе на перевод документа. В случае успеха этот метод возвращает `200 OK` код ответа.
<!-- markdownlint-disable MD024 -->

### <a name="c"></a>[C#](#tab/csharp)

```csharp
   
using System;
using System.Net.Http;
using System.Threading.Tasks;


class Program
{


    private static readonly string endpoint = "https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1";

    static readonly string route = "/batches/{id}";

    private static readonly string subscriptionKey = "<YOUR-SUBSCRIPTION-KEY>";

    static async Task Main(string[] args)
    {

        HttpClient client = new HttpClient();
            using HttpRequestMessage request = new HttpRequestMessage();
            {
                request.Method = HttpMethod.Get;
                request.RequestUri = new Uri(endpoint + route);
                request.Headers.Add("Ocp-Apim-Subscription-Key", subscriptionKey);


                HttpResponseMessage response = await client.SendAsync(request);
                string result = response.Content.ReadAsStringAsync().Result;

                Console.WriteLine($"Status code: {response.StatusCode}");
                Console.WriteLine($"Response Headers: {response.Headers}");
                Console.WriteLine();
                Console.WriteLine(result);
            }
}
```

### <a name="nodejs"></a>[Node.js](#tab/javascript)

```javascript

const axios = require('axios');

let endpoint = 'https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1';
let subscriptionKey = '<YOUR-SUBSCRIPTION-KEY>';
let route = '/batches/{id}';

let config = {
  method: 'get',
  url: endpoint + route,
  headers: { 
    'Ocp-Apim-Subscription-Key': subscriptionKey
  }
};

axios(config)
.then(function (response) {
  console.log(JSON.stringify(response.data));
})
.catch(function (error) {
  console.log(error);
});

```

### <a name="java"></a>[Java](#tab/java)

```java

import java.io.*;
import java.net.*;
import java.util.*;
import com.squareup.okhttp.*;

public class GetJobStatus {

    String subscriptionKey = "<YOUR-SUBSCRIPTION-KEY>";
    String endpoint = "https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1";
    String url = endpoint + "/batches/{id}";
    OkHttpClient client = new OkHttpClient();

    public void get() throws IOException {
        Request request = new Request.Builder().url(
                url).method("GET", null).addHeader("Ocp-Apim-Subscription-Key", subscriptionKey).build();
        Response response = client.newCall(request).execute(); 
            System.out.println(response.body().string());
        }

    public static void main(String[] args) throws IOException {
        try{
            GetJobs jobs = new GetJobs();
            jobs.get();
        } catch (Exception e) {
            System.out.println(e);
        }
    }
}

```

### <a name="python"></a>[Python](#tab/python)

```python

import http.client

host = '<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com'
parameters = '//translator/text/batch/v1.0-preview.1/batches/{id}'
subscriptionKey =  '<YOUR-SUBSCRIPTION-KEY>'
conn = http.client.HTTPSConnection(host)
payload = ''
headers = {
  'Ocp-Apim-Subscription-Key': subscriptionKey
}
conn.request("GET", parameters , payload, headers)
res = conn.getresponse()
data = res.read()
print(res.status)
print()
print(data.decode("utf-8"))
```

### <a name="go"></a>[GO](#tab/go)

```go

package main

import (
  "fmt"
  "net/http"
  "io/ioutil"
)

func main() {

  endpoint := "https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1"
  subscriptionKey := "<YOUR-SUBSCRIPTION-KEY>"
  uri := endpoint + "/batches/{id}"
  method := "GET"

  client := &http.Client {
  }
  req, err := http.NewRequest(method, uri, nil)

  if err != nil {
    fmt.Println(err)
    return
  }
  req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)

  res, err := client.Do(req)
  if err != nil {
    fmt.Println(err)
    return
  }
  defer res.Body.Close()

  body, err := ioutil.ReadAll(res.Body)
  if err != nil {
    fmt.Println(err)
    return
  }
  fmt.Println(res.StatusCode)
  fmt.Println(string(body))
}
```

---

## <a name="_get-document-status_"></a>_ПОЛУЧЕНИЕ состояния документа_

### <a name="brief-overview"></a>Краткий обзор

Получение состояния определенного документа в запросе на перевод документа. В случае успеха этот метод возвращает `200 OK` код ответа.

### <a name="c"></a>[C#](#tab/csharp)

```csharp
   
using System;
using System.Net.Http;
using System.Threading.Tasks;


class Program
{


    private static readonly string endpoint = "https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1";

    static readonly string route = "/{id}/document/{documentId}";

    private static readonly string subscriptionKey = "<YOUR-SUBSCRIPTION-KEY>";

    static async Task Main(string[] args)
    {

        HttpClient client = new HttpClient();
            using HttpRequestMessage request = new HttpRequestMessage();
            {
                request.Method = HttpMethod.Get;
                request.RequestUri = new Uri(endpoint + route);
                request.Headers.Add("Ocp-Apim-Subscription-Key", subscriptionKey);


                HttpResponseMessage response = await client.SendAsync(request);
                string result = response.Content.ReadAsStringAsync().Result;

                Console.WriteLine($"Status code: {response.StatusCode}");
                Console.WriteLine($"Response Headers: {response.Headers}");
                Console.WriteLine();
                Console.WriteLine(result);
            }
}
```

### <a name="nodejs"></a>[Node.js](#tab/javascript)

```javascript

const axios = require('axios');

let endpoint = 'https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1';
let subscriptionKey = '<YOUR-SUBSCRIPTION-KEY>';
let route = '/{id}/document/{documentId}';

let config = {
  method: 'get',
  url: endpoint + route,
  headers: { 
    'Ocp-Apim-Subscription-Key': subscriptionKey
  }
};

axios(config)
.then(function (response) {
  console.log(JSON.stringify(response.data));
})
.catch(function (error) {
  console.log(error);
});

```

### <a name="java"></a>[Java](#tab/java)

```java

import java.io.*;
import java.net.*;
import java.util.*;
import com.squareup.okhttp.*;

public class GetDocumentStatus {

    String subscriptionKey = "<YOUR-SUBSCRIPTION-KEY>";
    String endpoint = "https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1";
    String url = endpoint + "/{id}/document/{documentId}";
    OkHttpClient client = new OkHttpClient();

    public void get() throws IOException {
        Request request = new Request.Builder().url(
                url).method("GET", null).addHeader("Ocp-Apim-Subscription-Key", subscriptionKey).build();
        Response response = client.newCall(request).execute(); 
            System.out.println(response.body().string());
        }

    public static void main(String[] args) throws IOException {
        try{
            GetJobs jobs = new GetJobs();
            jobs.get();
        } catch (Exception e) {
            System.out.println(e);
        }
    }
}

```

### <a name="python"></a>[Python](#tab/python)

```python

import http.client

host = '<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com'
parameters = '//translator/text/batch/v1.0-preview.1/{id}/document/{documentId}'
subscriptionKey =  '<YOUR-SUBSCRIPTION-KEY>'
conn = http.client.HTTPSConnection(host)
payload = ''
headers = {
  'Ocp-Apim-Subscription-Key': subscriptionKey
}
conn.request("GET", parameters , payload, headers)
res = conn.getresponse()
data = res.read()
print(res.status)
print()
print(data.decode("utf-8"))
```

### <a name="go"></a>[GO](#tab/go)

```go

package main

import (
  "fmt"
  "net/http"
  "io/ioutil"
)

func main() {

  endpoint := "https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1"
  subscriptionKey := "<YOUR-SUBSCRIPTION-KEY>"
  uri := endpoint + "/{id}/document/{documentId}"
  method := "GET"

  client := &http.Client {
  }
  req, err := http.NewRequest(method, uri, nil)

  if err != nil {
    fmt.Println(err)
    return
  }
  req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)

  res, err := client.Do(req)
  if err != nil {
    fmt.Println(err)
    return
  }
  defer res.Body.Close()

  body, err := ioutil.ReadAll(res.Body)
  if err != nil {
    fmt.Println(err)
    return
  }
  fmt.Println(res.StatusCode)
  fmt.Println(string(body))
}
```

---

## <a name="_delete-job_"></a>_УДАЛИТЬ задание_ 

### <a name="brief-overview"></a>Краткий обзор

Отмена обработки или задания, поставленного в очередь. Будут отменены только документы, для которых перевод не начался.

### <a name="c"></a>[C#](#tab/csharp)

```csharp
   
using System;
using System.Net.Http;
using System.Threading.Tasks;


class Program
{


    private static readonly string endpoint = "https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1";

    static readonly string route = "/batches/{id}";

    private static readonly string subscriptionKey = "<YOUR-SUBSCRIPTION-KEY>";

    static async Task Main(string[] args)
    {

        HttpClient client = new HttpClient();
            using HttpRequestMessage request = new HttpRequestMessage();
            {
                request.Method = HttpMethod.Delete;
                request.RequestUri = new Uri(endpoint + route);
                request.Headers.Add("Ocp-Apim-Subscription-Key", subscriptionKey);


                HttpResponseMessage response = await client.SendAsync(request);
                string result = response.Content.ReadAsStringAsync().Result;

                Console.WriteLine($"Status code: {response.StatusCode}");
                Console.WriteLine($"Response Headers: {response.Headers}");
                Console.WriteLine();
                Console.WriteLine(result);
            }
}
```

### <a name="nodejs"></a>[Node.js](#tab/javascript)

```javascript

const axios = require('axios');

let endpoint = 'https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1';
let subscriptionKey = '<YOUR-SUBSCRIPTION-KEY>';
let route = '/batches/{id}';

let config = {
  method: 'delete',
  url: endpoint + route,
  headers: { 
    'Ocp-Apim-Subscription-Key': subscriptionKey
  }
};

axios(config)
.then(function (response) {
  console.log(JSON.stringify(response.data));
})
.catch(function (error) {
  console.log(error);
});

```

### <a name="java"></a>[Java](#tab/java)

```java

import java.io.*;
import java.net.*;
import java.util.*;
import com.squareup.okhttp.*;

public class DeleteJob {

    String subscriptionKey = "<YOUR-SUBSCRIPTION-KEY>";
    String endpoint = "https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1";
    String url = endpoint + "/batches/{id}";
    OkHttpClient client = new OkHttpClient();

    public void get() throws IOException {
        Request request = new Request.Builder().url(
                url).method("DELETE", null).addHeader("Ocp-Apim-Subscription-Key", subscriptionKey).build();
        Response response = client.newCall(request).execute(); 
            System.out.println(response.body().string());
        }

    public static void main(String[] args) throws IOException {
        try{
            GetJobs jobs = new GetJobs();
            jobs.get();
        } catch (Exception e) {
            System.out.println(e);
        }
    }
}

```

### <a name="python"></a>[Python](#tab/python)

```python

import http.client

host = '<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com'
parameters = '//translator/text/batch/v1.0-preview.1/batches/{id}'
subscriptionKey =  '<YOUR-SUBSCRIPTION-KEY>'
conn = http.client.HTTPSConnection(host)
payload = ''
headers = {
  'Ocp-Apim-Subscription-Key': subscriptionKey
}
conn.request("DELETE", parameters , payload, headers)
res = conn.getresponse()
data = res.read()
print(res.status)
print()
print(data.decode("utf-8"))
```

### <a name="go"></a>[GO](#tab/go)

```go

package main

import (
  "fmt"
  "net/http"
  "io/ioutil"
)

func main() {

  endpoint := "https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1"
  subscriptionKey := "<YOUR-SUBSCRIPTION-KEY>"
  uri := endpoint + "/batches/{id}"
  method := "DELETE"

  client := &http.Client {
  }
  req, err := http.NewRequest(method, uri, nil)

  if err != nil {
    fmt.Println(err)
    return
  }
  req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)

  res, err := client.Do(req)
  if err != nil {
    fmt.Println(err)
    return
  }
  defer res.Body.Close()

  body, err := ioutil.ReadAll(res.Body)
  if err != nil {
    fmt.Println(err)
    return
  }
  fmt.Println(res.StatusCode)
  fmt.Println(string(body))
}
```

---

## <a name="content-limits"></a>Ограничения содержимого

В таблице ниже перечислены ограничения для данных, отправляемых в перевод документа.

|attribute | Ограничение|
|---|---|
|Размер документа| ≤ 40 МБ |
|Общее число файлов.|≤ 1000 |
|Общий размер содержимого в пакете | ≤ 250 МБ|
|Число целевых языков в пакете| ≤ 10 |
|Размер файла преобразования памяти| ≤ 10 МБ|

> [!NOTE]
> Приведенные выше ограничения содержимого могут быть изменены до выхода общедоступной версии.

## <a name="learn-more"></a>Материалы для дальнейшего изучения

* [Справочник по API Переводчика версии 3](../reference/v3-0-reference.md)
* [Поддержка языков](../language-support.md)
* [Подписки в службе управления API Azure](../../../api-management/api-management-subscriptions.md).

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Создание настраиваемой языковой системы с помощью настраиваемого транслятора](../custom-translator/overview.md)
>
>
