---
title: API сборщика данных HTTP в Azure Monitor | Документация Майкрософт
description: API сборщика данных HTTP в Azure Monitor можно использовать для добавления данных POST JSON в рабочую область Log Analytics из любого клиента, который может вызывать REST API. В этой статье описывается, как использовать API, и приводятся примеры публикации данных с использованием разных языков программирования.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/14/2020
ms.openlocfilehash: e32bf95ef52fdd081eeaa476f44bf5dab99657d6
ms.sourcegitcommit: 6386854467e74d0745c281cc53621af3bb201920
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2021
ms.locfileid: "102452124"
---
# <a name="send-log-data-to-azure-monitor-with-the-http-data-collector-api-public-preview"></a>Отправка данных журналов в Azure Monitor c помощью API сборщика данных HTTP (общедоступная предварительная версия)
В этой статье показано, как с помощью API сборщика данных HTTP отправлять данные журналов в Azure Monitor из клиента REST API.  Здесь также описано, как отформатировать данные, собранные скриптом или приложением, добавить их в запрос и авторизовать этот запрос в Azure Monitor.  В этой статье приведены примеры для PowerShell, C# и Python.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

> [!NOTE]
> API сборщика данных HTTP для Azure Monitor предоставляется в общедоступной предварительной версии.

## <a name="concepts"></a>Основные понятия
API сборщика данных HTTP можно использовать для отправки данных журналов в рабочую область Log Analytics, размещенную в Azure Monitor, из любого клиента с поддержкой REST API.  Это может быть runbook в службе автоматизации Azure, который собирает данные по управлению из Azure или другого облака, или любая другая система управления, использующая Azure Monitor для консолидации и анализа данных журналов.

Все данные в рабочей области Log Analytics хранятся как запись определенного типа.  Чтобы отправить данные в API сборщика данных HTTP, их необходимо отформатировать как несколько записей в формате JSON.  При отправке данных в репозитории для каждой записи в запрошенных полезных данных создается отдельная запись.


![Обзор сборщика данных HTTP](media/data-collector-api/overview.png)



## <a name="create-a-request"></a>Создание запроса
Чтобы использовать API сборщика данных HTTP, необходимо создать запрос POST, содержащий данные для отправки в нотацию объектов JavaScript (JSON).  В следующих трех таблицах перечислены обязательные атрибуты для каждого запроса. Более подробное описание каждого из атрибутов приводится далее в этой статье.

### <a name="request-uri"></a>Универсальный код ресурса (URI) запроса
| Атрибут | Свойство. |
|:--- |:--- |
| Метод |POST |
| URI |https://\<CustomerId\>.ods.opinsights.azure.com/api/logs?api-version=2016-04-01 |
| Тип содержимого |приложение/json |

### <a name="request-uri-parameters"></a>Параметры URI запроса
| Параметр | Описание |
|:--- |:--- |
| CustomerID |Уникальный идентификатор для рабочей области Log Analytics. |
| Ресурс |Имя ресурса API: /api/logs. |
| Версия API |Версия API для использования с этим запросом. В настоящее время это версия 2016-04-01. |

### <a name="request-headers"></a>Заголовки запроса
| Header | Описание |
|:--- |:--- |
| Авторизация |Подпись авторизации. Далее в этой статье вы найдете сведения о том, как создать заголовок HMAC-SHA256. |
| Log-Type |Укажите тип записи для отправляемых данных. Может содержать только буквы, цифры и символы подчеркивания (_) и не может превышать 100 символов. |
| x-ms-date |Дата обработки запроса в формате RFC 1123. |
| x-MS-Азурересаурцеид | Идентификатор ресурса Azure, с которым должны быть связаны данные. Заполняет свойство [_ResourceId](./log-standard-columns.md#_resourceid) и позволяет включать данные в запросы [контекста ресурсов](./design-logs-deployment.md#access-mode) . Если это поле не указано, данные не будут включаться в запросы контекста ресурсов. |
| time-generated-field | Имя поля данных, содержащее метку времени элемента данных. Если вы укажете здесь поле, его содержимое будет использоваться как значение параметра **TimeGenerated**. Если это поле не указано, по умолчанию для **TimeGenerated** будет использоваться время приема сообщения. Содержимое поля сообщения должно соответствовать формату ISO 8601: YYYY-MM-DDThh:mm:ssZ. |

## <a name="authorization"></a>Авторизация
Любой запрос к API сборщика данных HTTP в Azure Monitor должен содержать заголовок авторизации. Чтобы проверить подлинность запроса, необходимо подписать запрос с помощью первичного или вторичного ключа для рабочей области, выполняющей запрос. Затем следует передать подпись как часть запроса.   

Вот формат заголовка авторизации:

```
Authorization: SharedKey <WorkspaceID>:<Signature>
```

*WorkspaceID* — это уникальный идентификатор для рабочей области Log Analytics. *Signature* — это [код проверки подлинности сообщения на основе хэша (HMAC)](/dotnet/api/system.security.cryptography.hmacsha256), созданный из запроса и вычисленный с помощью [алгоритма SHA256](/dotnet/api/system.security.cryptography.sha256). Затем его можно закодировать с помощью кодировки Base64.

Используйте этот формат для кодирования строки подписи **SharedKey**:

```
StringToSign = VERB + "\n" +
                  Content-Length + "\n" +
               Content-Type + "\n" +
                  x-ms-date + "\n" +
                  "/api/logs";
```

Вот пример строки подписи:

```
POST\n1024\napplication/json\nx-ms-date:Mon, 04 Apr 2016 08:00:00 GMT\n/api/logs
```

При наличии строки подписи закодируйте ее с помощью алгоритма HMAC-SHA256 в строке в кодировке UTF-8, а затем закодируйте результат в Base64. Используйте следующий формат:

```
Signature=Base64(HMAC-SHA256(UTF8(StringToSign)))
```

Примеры в следующих разделах содержат образец кода, с помощью которого вы сможете создать заголовок авторизации.

## <a name="request-body"></a>Текст запроса
Текст сообщения должен иметь формат JSON. Он должен содержать одну или несколько записей с парами имен и значений свойств в следующем формате. Имя свойства может содержать только буквы, цифры и символ подчеркивания (_).

```json
[
    {
        "property 1": "value1",
        "property 2": "value2",
        "property 3": "value3",
        "property 4": "value4"
    }
]
```

Вы можете сгруппировать в одном запросе несколько записей, используя следующий формат. Все записи должны принадлежать к одному типу.

```json
[
    {
        "property 1": "value1",
        "property 2": "value2",
        "property 3": "value3",
        "property 4": "value4"
    },
    {
        "property 1": "value1",
        "property 2": "value2",
        "property 3": "value3",
        "property 4": "value4"
    }
]
```

## <a name="record-type-and-properties"></a>Тип и свойства записи
При отправке данных через API сборщика данных HTTP в Azure Monitor определяется тип пользовательской записи. В настоящее время нельзя записывать данные в записи существующих типов, созданные с помощью других типов данных и решений. Azure Monitor считывает входящие данные, а затем создает свойства, которые соответствуют типам данных для введенных значений.

Каждый запрос к API сборщика данных должен содержать заголовок **Log-Type** с именем типа записей. Суффикс **_CL** автоматически добавляется к вводимому имени, чтобы пользовательский журнал отличался от журналов других типов. Например, если вы введете имя **MyNewRecordType**, Azure Monitor создаст запись с типом **MyNewRecordType_CL**. Это помогает избежать конфликтов между именами типов, создаваемыми пользователями, и готовыми именами существующих или будущих решений Microsoft.

Чтобы определить тип данных для свойства, Azure Monitor добавляет суффикс к имени свойства. Если свойство содержит значение NULL, свойство не включается в эту запись. В таблице ниже перечислены типы данных свойства и соответствующие суффиксы.

| Тип данных свойства | Суффикс |
|:--- |:--- |
| Строка |_s |
| Логическое |_b |
| Double |_d |
| Дата и время |_t |
| GUID (хранится в виде строки) |_g |

> [!NOTE]
> Строковые значения, которые выглядят как идентификаторы GUID, получают суффикс _g и форматируются как идентификатор GUID, даже если входящее значение не включает дефисы. Например, "8145d822-13a7-44ad-859c-36f31a84f6dd" и "8145d82213a744ad859c36f31a84f6dd" будут храниться как "8145d822-13a7-44ad-859c-36f31a84f6dd". Единственным различием между этой и другой строкой является _g в имени и вставка тире, если они не указаны во входных данных. 

Тип данных, который Azure Monitor использует для каждого свойства, зависит от того, существует ли тип записи для новой записи.

* Если тип записи не существует, Azure Monitor создает новый, используя определение типа JSON для выбора типа данных для каждого свойства новой записи.
* Если тип записи существует, Azure Monitor пытается создать новую запись на основе существующих свойств. Если тип данных для свойства в новой записи не соответствует существующему типу и не может быть преобразован в него или если запись включает свойство, которое не существует, Azure Monitor создает новое свойство с соответствующим суффиксом.

Например, при отправке следующих данных создается запись с тремя свойствами: **number_d**, **boolean_b** и **string_s**:

![Пример записи 1](media/data-collector-api/record-01.png)

Если отправить следующую запись со всеми значениями в строковом формате, свойства не изменятся. Эти значения можно преобразовать в существующие типы данных:

![Пример записи 2](media/data-collector-api/record-02.png)

Если после этого вы отправите следующую запись, Azure Monitor создаст новые свойства **boolean_d** и **string_d**. Преобразовать эти значения нельзя:

![Пример записи 3](media/data-collector-api/record-03.png)

Если вы теперь отправите следующую запись, перед созданием типа записи, Azure Monitor создаст запись с тремя свойствами: **number_s**, **boolean_s** и **string_s**. В этой записи каждое начальное значение форматируется как строка:

![Пример записи 4](media/data-collector-api/record-04.png)

## <a name="reserved-properties"></a>Зарезервированные свойства
Следующие свойства зарезервированы и не должны использоваться в пользовательском типе записи. Если полезные данные содержат любое из этих имен свойств, появится сообщение об ошибке.

- tenant

## <a name="data-limits"></a>Ограничения данных
Существует ряд ограничений на данные, публикуемые в API сбора данных Azure Monitor.

* Не более 30 МБ на одну запись в API сборщика данных Azure Monitor. Это ограничение размера для одной публикации. Если данные одной публикации превышают 30 МБ, необходимо разделить данные на меньшие фрагменты и отправить их параллельно.
* Не более 32 КБ для значений полей. Если значение поля превышает 32 КБ, данные будут усечены.
* Рекомендуемое максимальное количество полей для данного типа — 50. Это ограничение введено для удобства поиска и использования.  
* Каждая таблица в рабочей области Log Analytics поддерживает не более 500 столбцов (в этой статье они называются полями). 
* Имя столбца не может содержать более 500 символов.

## <a name="return-codes"></a>Коды возврата
Код состояния HTTP 200 означает, что запрос получен для обработки. Такой результат означает, что операция завершена успешно.

В этой таблице представлен полный набор кодов состояний, которые может возвращать служба:

| Код | Состояние | Код ошибки | Описание |
|:--- |:--- |:--- |:--- |
| 200 |ОК | |Запрос был успешно принят. |
| 400 |Недопустимый запрос |InactiveCustomer |Рабочая область закрыта. |
| 400 |Недопустимый запрос |InvalidApiVersion |Указанная версия API не распознана службой. |
| 400 |Недопустимый запрос |InvalidCustomerId |Указан недопустимый идентификатор рабочей области. |
| 400 |Недопустимый запрос |InvalidDataFormat |Отправлены недопустимые данные JSON. Текст ответа может содержать дополнительные сведения о том, как устранить ошибку. |
| 400 |Недопустимый запрос |InvalidLogType |Указанный тип журнала содержит специальные символы или цифры. |
| 400 |Недопустимый запрос |MissingApiVersion |Версия API не указана. |
| 400 |Недопустимый запрос |MissingContentType |Тип содержимого не указан. |
| 400 |Недопустимый запрос |MissingLogType |Обязательное значение типа журнала не указано. |
| 400 |Недопустимый запрос |UnsupportedContentType |Для типа содержимого не было задано значение **application/json**. |
| 403 |Запрещено |InvalidAuthorization |Службе не удалось проверить подлинность запроса. Проверьте правильность идентификатора и ключа подключения рабочей области. |
| 404 |Не найдено | | Указан неправильный URL-адрес либо запрос слишком большой. |
| 429 |Слишком много запросов | | В службу поступает слишком большой объем данных из вашей учетной записи. Повторите запрос позже. |
| 500 |Внутренняя ошибка сервера |UnspecifiedError |Служба обнаружила внутреннюю ошибку. Повторите запрос. |
| 503 |Служба недоступна |ServiceUnavailable |Служба сейчас недоступна для получения запросов. Повторите запрос. |

## <a name="query-data"></a>Данные запросов
Чтобы выполнить запрос данных, отправленных через API сборщика данных HTTP в Azure Monitor, найдите записи с **типом**, равным указанному вами значению **LogType** с суффиксом **_CL**. Например, если вы использовали значение **MyCustomLog**, будут возвращены все записи с типом `MyCustomLog_CL`.

## <a name="sample-requests"></a>Примеры запросов
В следующих разделах вы найдете примеры отправки данных в API сборщика данных HTTP в Azure Monitor на разных языках программирования.

Для каждого примера выполните следующие действия, чтобы задать переменные в заголовке авторизации:

1. На портале Azure перейдите в свою рабочую область Log Analytics.
2. Выберите **Управление агентами**.
2. Справа от **идентификатора рабочей области** щелкните значок копирования и вставьте идентификатор как значение переменной **CustomerID**.
3. Справа от **первичного ключа** щелкните значок копирования и вставьте идентификатор как значение переменной **SharedKey**.

Также можно изменить переменные для типа журнала и данных JSON.

### <a name="powershell-sample"></a>Пример для PowerShell
```powershell
# Replace with your Workspace ID
$CustomerId = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"  

# Replace with your Primary Key
$SharedKey = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# Specify the name of the record type that you'll be creating
$LogType = "MyRecordType"

# You can use an optional field to specify the timestamp from the data. If the time field is not specified, Azure Monitor assumes the time is the message ingestion time
$TimeStampField = ""


# Create two records with the same set of properties to create
$json = @"
[{  "StringValue": "MyString1",
    "NumberValue": 42,
    "BooleanValue": true,
    "DateValue": "2019-09-12T20:00:00.625Z",
    "GUIDValue": "9909ED01-A74C-4874-8ABF-D2678E3AE23D"
},
{   "StringValue": "MyString2",
    "NumberValue": 43,
    "BooleanValue": false,
    "DateValue": "2019-09-12T20:00:00.625Z",
    "GUIDValue": "8809ED01-A74C-4874-8ABF-D2678E3AE23D"
}]
"@

# Create the function to create the authorization signature
Function Build-Signature ($customerId, $sharedKey, $date, $contentLength, $method, $contentType, $resource)
{
    $xHeaders = "x-ms-date:" + $date
    $stringToHash = $method + "`n" + $contentLength + "`n" + $contentType + "`n" + $xHeaders + "`n" + $resource

    $bytesToHash = [Text.Encoding]::UTF8.GetBytes($stringToHash)
    $keyBytes = [Convert]::FromBase64String($sharedKey)

    $sha256 = New-Object System.Security.Cryptography.HMACSHA256
    $sha256.Key = $keyBytes
    $calculatedHash = $sha256.ComputeHash($bytesToHash)
    $encodedHash = [Convert]::ToBase64String($calculatedHash)
    $authorization = 'SharedKey {0}:{1}' -f $customerId,$encodedHash
    return $authorization
}


# Create the function to create and post the request
Function Post-LogAnalyticsData($customerId, $sharedKey, $body, $logType)
{
    $method = "POST"
    $contentType = "application/json"
    $resource = "/api/logs"
    $rfc1123date = [DateTime]::UtcNow.ToString("r")
    $contentLength = $body.Length
    $signature = Build-Signature `
        -customerId $customerId `
        -sharedKey $sharedKey `
        -date $rfc1123date `
        -contentLength $contentLength `
        -method $method `
        -contentType $contentType `
        -resource $resource
    $uri = "https://" + $customerId + ".ods.opinsights.azure.com" + $resource + "?api-version=2016-04-01"

    $headers = @{
        "Authorization" = $signature;
        "Log-Type" = $logType;
        "x-ms-date" = $rfc1123date;
        "time-generated-field" = $TimeStampField;
    }

    $response = Invoke-WebRequest -Uri $uri -Method $method -ContentType $contentType -Headers $headers -Body $body -UseBasicParsing
    return $response.StatusCode

}

# Submit the data to the API endpoint
Post-LogAnalyticsData -customerId $customerId -sharedKey $sharedKey -body ([System.Text.Encoding]::UTF8.GetBytes($json)) -logType $logType  
```

### <a name="c-sample"></a>Пример на языке C#
```csharp
using System;
using System.Net;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Security.Cryptography;
using System.Text;
using System.Threading.Tasks;

namespace OIAPIExample
{
    class ApiExample
    {
        // An example JSON object, with key/value pairs
        static string json = @"[{""DemoField1"":""DemoValue1"",""DemoField2"":""DemoValue2""},{""DemoField3"":""DemoValue3"",""DemoField4"":""DemoValue4""}]";

        // Update customerId to your Log Analytics workspace ID
        static string customerId = "xxxxxxxx-xxx-xxx-xxx-xxxxxxxxxxxx";

        // For sharedKey, use either the primary or the secondary Connected Sources client authentication key   
        static string sharedKey = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";

        // LogName is name of the event type that is being submitted to Azure Monitor
        static string LogName = "DemoExample";

        // You can use an optional field to specify the timestamp from the data. If the time field is not specified, Azure Monitor assumes the time is the message ingestion time
        static string TimeStampField = "";

        static void Main()
        {
            // Create a hash for the API signature
            var datestring = DateTime.UtcNow.ToString("r");
            var jsonBytes = Encoding.UTF8.GetBytes(json);
            string stringToHash = "POST\n" + jsonBytes.Length + "\napplication/json\n" + "x-ms-date:" + datestring + "\n/api/logs";
            string hashedString = BuildSignature(stringToHash, sharedKey);
            string signature = "SharedKey " + customerId + ":" + hashedString;

            PostData(signature, datestring, json);
        }

        // Build the API signature
        public static string BuildSignature(string message, string secret)
        {
            var encoding = new System.Text.ASCIIEncoding();
            byte[] keyByte = Convert.FromBase64String(secret);
            byte[] messageBytes = encoding.GetBytes(message);
            using (var hmacsha256 = new HMACSHA256(keyByte))
            {
                byte[] hash = hmacsha256.ComputeHash(messageBytes);
                return Convert.ToBase64String(hash);
            }
        }

        // Send a request to the POST API endpoint
        public static void PostData(string signature, string date, string json)
        {
            try
            {
                string url = "https://" + customerId + ".ods.opinsights.azure.com/api/logs?api-version=2016-04-01";

                System.Net.Http.HttpClient client = new System.Net.Http.HttpClient();
                client.DefaultRequestHeaders.Add("Accept", "application/json");
                client.DefaultRequestHeaders.Add("Log-Type", LogName);
                client.DefaultRequestHeaders.Add("Authorization", signature);
                client.DefaultRequestHeaders.Add("x-ms-date", date);
                client.DefaultRequestHeaders.Add("time-generated-field", TimeStampField);

                System.Net.Http.HttpContent httpContent = new StringContent(json, Encoding.UTF8);
                httpContent.Headers.ContentType = new MediaTypeHeaderValue("application/json");
                Task<System.Net.Http.HttpResponseMessage> response = client.PostAsync(new Uri(url), httpContent);

                System.Net.Http.HttpContent responseContent = response.Result.Content;
                string result = responseContent.ReadAsStringAsync().Result;
                Console.WriteLine("Return Result: " + result);
            }
            catch (Exception excep)
            {
                Console.WriteLine("API Post Exception: " + excep.Message);
            }
        }
    }
}

```

### <a name="python-2-sample"></a>Пример на языке Python версии 2
```python
import json
import requests
import datetime
import hashlib
import hmac
import base64

# Update the customer ID to your Log Analytics workspace ID
customer_id = 'xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'

# For the shared key, use either the primary or the secondary Connected Sources client authentication key   
shared_key = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# The log type is the name of the event that is being submitted
log_type = 'WebMonitorTest'

# An example JSON web monitor object
json_data = [{
   "slot_ID": 12345,
    "ID": "5cdad72f-c848-4df0-8aaa-ffe033e75d57",
    "availability_Value": 100,
    "performance_Value": 6.954,
    "measurement_Name": "last_one_hour",
    "duration": 3600,
    "warning_Threshold": 0,
    "critical_Threshold": 0,
    "IsActive": "true"
},
{   
    "slot_ID": 67890,
    "ID": "b6bee458-fb65-492e-996d-61c4d7fbb942",
    "availability_Value": 100,
    "performance_Value": 3.379,
    "measurement_Name": "last_one_hour",
    "duration": 3600,
    "warning_Threshold": 0,
    "critical_Threshold": 0,
    "IsActive": "false"
}]
body = json.dumps(json_data)

#####################
######Functions######  
#####################

# Build the API signature
def build_signature(customer_id, shared_key, date, content_length, method, content_type, resource):
    x_headers = 'x-ms-date:' + date
    string_to_hash = method + "\n" + str(content_length) + "\n" + content_type + "\n" + x_headers + "\n" + resource
    bytes_to_hash = bytes(string_to_hash).encode('utf-8')  
    decoded_key = base64.b64decode(shared_key)
    encoded_hash = base64.b64encode(hmac.new(decoded_key, bytes_to_hash, digestmod=hashlib.sha256).digest())
    authorization = "SharedKey {}:{}".format(customer_id,encoded_hash)
    return authorization

# Build and send a request to the POST API
def post_data(customer_id, shared_key, body, log_type):
    method = 'POST'
    content_type = 'application/json'
    resource = '/api/logs'
    rfc1123date = datetime.datetime.utcnow().strftime('%a, %d %b %Y %H:%M:%S GMT')
    content_length = len(body)
    signature = build_signature(customer_id, shared_key, rfc1123date, content_length, method, content_type, resource)
    uri = 'https://' + customer_id + '.ods.opinsights.azure.com' + resource + '?api-version=2016-04-01'

    headers = {
        'content-type': content_type,
        'Authorization': signature,
        'Log-Type': log_type,
        'x-ms-date': rfc1123date
    }

    response = requests.post(uri,data=body, headers=headers)
    if (response.status_code >= 200 and response.status_code <= 299):
        print 'Accepted'
    else:
        print "Response code: {}".format(response.status_code)

post_data(customer_id, shared_key, body, log_type)
```

### <a name="python-3-sample"></a>Пример Python 3
```python
import json
import requests
import datetime
import hashlib
import hmac
import base64

# Update the customer ID to your Log Analytics workspace ID
customer_id = 'xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'

# For the shared key, use either the primary or the secondary Connected Sources client authentication key   
shared_key = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# The log type is the name of the event that is being submitted
log_type = 'WebMonitorTest'

# An example JSON web monitor object
json_data = [{
   "slot_ID": 12345,
    "ID": "5cdad72f-c848-4df0-8aaa-ffe033e75d57",
    "availability_Value": 100,
    "performance_Value": 6.954,
    "measurement_Name": "last_one_hour",
    "duration": 3600,
    "warning_Threshold": 0,
    "critical_Threshold": 0,
    "IsActive": "true"
},
{   
    "slot_ID": 67890,
    "ID": "b6bee458-fb65-492e-996d-61c4d7fbb942",
    "availability_Value": 100,
    "performance_Value": 3.379,
    "measurement_Name": "last_one_hour",
    "duration": 3600,
    "warning_Threshold": 0,
    "critical_Threshold": 0,
    "IsActive": "false"
}]
body = json.dumps(json_data)

#####################
######Functions######  
#####################

# Build the API signature
def build_signature(customer_id, shared_key, date, content_length, method, content_type, resource):
    x_headers = 'x-ms-date:' + date
    string_to_hash = method + "\n" + str(content_length) + "\n" + content_type + "\n" + x_headers + "\n" + resource
    bytes_to_hash = bytes(string_to_hash, encoding="utf-8")  
    decoded_key = base64.b64decode(shared_key)
    encoded_hash = base64.b64encode(hmac.new(decoded_key, bytes_to_hash, digestmod=hashlib.sha256).digest()).decode()
    authorization = "SharedKey {}:{}".format(customer_id,encoded_hash)
    return authorization

# Build and send a request to the POST API
def post_data(customer_id, shared_key, body, log_type):
    method = 'POST'
    content_type = 'application/json'
    resource = '/api/logs'
    rfc1123date = datetime.datetime.utcnow().strftime('%a, %d %b %Y %H:%M:%S GMT')
    content_length = len(body)
    signature = build_signature(customer_id, shared_key, rfc1123date, content_length, method, content_type, resource)
    uri = 'https://' + customer_id + '.ods.opinsights.azure.com' + resource + '?api-version=2016-04-01'

    headers = {
        'content-type': content_type,
        'Authorization': signature,
        'Log-Type': log_type,
        'x-ms-date': rfc1123date
    }

    response = requests.post(uri,data=body, headers=headers)
    if (response.status_code >= 200 and response.status_code <= 299):
        print('Accepted')
    else:
        print("Response code: {}".format(response.status_code))

post_data(customer_id, shared_key, body, log_type)
```


### <a name="java-sample"></a>Пример Java

```java

import org.apache.http.HttpResponse;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.springframework.http.MediaType;

import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.text.SimpleDateFormat;
import java.util.Base64;
import java.util.Calendar;
import java.util.TimeZone;

import static org.springframework.http.HttpHeaders.CONTENT_TYPE;

public class ApiExample {

  private static final String workspaceId = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";
  private static final String sharedKey = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";
  private static final String logName = "DemoExample";
  /*
  You can use an optional field to specify the timestamp from the data. If the time field is not specified,
  Azure Monitor assumes the time is the message ingestion time
   */
  private static final String timestamp = "";
  private static final String json = "{\"name\": \"test\",\n" + "  \"id\": 1\n" + "}";
  private static final String RFC_1123_DATE = "EEE, dd MMM yyyy HH:mm:ss z";

  public static void main(String[] args) throws IOException, NoSuchAlgorithmException, InvalidKeyException {
    String dateString = getServerTime();
    String httpMethod = "POST";
    String contentType = "application/json";
    String xmsDate = "x-ms-date:" + dateString;
    String resource = "/api/logs";
    String stringToHash = String
        .join("\n", httpMethod, String.valueOf(json.getBytes(StandardCharsets.UTF_8).length), contentType,
            xmsDate , resource);
    String hashedString = getHMAC254(stringToHash, sharedKey);
    String signature = "SharedKey " + workspaceId + ":" + hashedString;

    postData(signature, dateString, json);
  }

  private static String getServerTime() {
    Calendar calendar = Calendar.getInstance();
    SimpleDateFormat dateFormat = new SimpleDateFormat(RFC_1123_DATE);
    dateFormat.setTimeZone(TimeZone.getTimeZone("GMT"));
    return dateFormat.format(calendar.getTime());
  }

  private static void postData(String signature, String dateString, String json) throws IOException {
    String url = "https://" + workspaceId + ".ods.opinsights.azure.com/api/logs?api-version=2016-04-01";
    HttpPost httpPost = new HttpPost(url);
    httpPost.setHeader("Authorization", signature);
    httpPost.setHeader(CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE);
    httpPost.setHeader("Log-Type", logName);
    httpPost.setHeader("x-ms-date", dateString);
    httpPost.setHeader("time-generated-field", timestamp);
    httpPost.setEntity(new StringEntity(json));
    try(CloseableHttpClient httpClient = HttpClients.createDefault()){
      HttpResponse response = httpClient.execute(httpPost);
      int statusCode = response.getStatusLine().getStatusCode();
      System.out.println("Status code: " + statusCode);
    }
  }

  private static String getHMAC254(String input, String key) throws InvalidKeyException, NoSuchAlgorithmException {
    String hash;
    Mac sha254HMAC = Mac.getInstance("HmacSHA256");
    Base64.Decoder decoder = Base64.getDecoder();
    SecretKeySpec secretKey = new SecretKeySpec(decoder.decode(key.getBytes(StandardCharsets.UTF_8)), "HmacSHA256");
    sha254HMAC.init(secretKey);
    Base64.Encoder encoder = Base64.getEncoder();
    hash = new String(encoder.encode(sha254HMAC.doFinal(input.getBytes(StandardCharsets.UTF_8))));
    return hash;
  }

}


```


## <a name="alternatives-and-considerations"></a>Альтернативы и рекомендации
Хотя API сборщика данных должен охватывать большую часть ваших потребностей для сбора данных из свободной формы в журналы Azure, существуют экземпляры, где может потребоваться альтернатива для преодоления некоторых ограничений API. Доступны следующие параметры:

| Альтернатива | Описание | Лучше всего подходит для |
|---|---|---|
| [Пользовательские события](../app/api-custom-events-metrics.md?toc=%2Fazure%2Fazure-monitor%2Ftoc.json#properties): получение машинного кода на основе пакета SDK в Application Insights | Application Insights, обычно инструментированный с помощью пакета SDK в приложении, предлагает возможность отправки пользовательских данных через пользовательские события. | <ul><li> Данные, создаваемые в приложении, но не подобранные пакетом SDK с помощью одного из типов данных по умолчанию (запросы, зависимости, исключения и т. д.).</li><li> Данные, наиболее часто связанные с другими данными приложений в Application Insights </li></ul> |
| API сборщика данных в журналах Azure Monitor | API сборщика данных в журналах Azure Monitor — это полностью открытый способ приема данных. Все данные, отформатированные в объекте JSON, можно отправить здесь. После отправки он будет обработан и доступен в журналах для сопоставления с другими данными в журналах или с другими Application Insights данными. <br/><br/> Вы довольно просто отправляете данные в виде файлов в большой двоичный объект BLOB-объектов Azure, откуда эти файлы будут обрабатываться и отправлены в Log Analytics. Пример реализации такого конвейера см. в [этой](./create-pipeline-datacollector-api.md) статье. | <ul><li> Данные, которые не обязательно создаются в приложении, оснащенном в Application Insights.</li><li> Примеры включают таблицы подстановок и фактов, ссылочные данные, предварительно агрегированные статистические данные и т. д. </li><li> Предназначен для данных, на которые будут пересекаться другие Azure Monitor данные (Application Insights, другие типы данных журналов, центр безопасности, контейнеры и другие виртуальные машины и т. д.). </li></ul> |
| [Azure Data Explorer](/azure/data-explorer/ingest-data-overview) | Azure обозреватель данных (ADX) — это платформа данных, которая обеспечивает работу Application Insights аналитики и журналов Azure Monitor. Теперь в общедоступной версии ("GA") использование платформы данных в ее необработанной форме обеспечивает полную гибкость (но требует дополнительных затрат на управление) в кластере (Kubernetes RBAC, скорость хранения, схема и т. д.). ADX предоставляет много [вариантов приема](/azure/data-explorer/ingest-data-overview#ingestion-methods) , включая файлы [CSV, TSV и JSON](/azure/kusto/management/mappings) . | <ul><li> Данные, которые не будут сопоставляться с другими данными в разделе Application Insights или журналы. </li><li> В журналах Azure Monitor данные, для которых требуются расширенные возможности приема или обработки, недоступны в настоящее время. </li></ul> |


## <a name="next-steps"></a>Дальнейшие действия
- Чтобы получить данные из рабочей области Log Analytics, используйте [API поиска по журналам](./log-query-overview.md).

- Дополнительные сведения о [создании конвейера данных с помощью API сборщика данных](create-pipeline-datacollector-api.md) и рабочего процесса Logic Apps в Azure Monitor.