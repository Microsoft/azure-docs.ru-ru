---
title: Исходящая аутентификация
description: Сведения о настройке или удалении исходящей проверки подлинности для планировщика Azure
services: scheduler
ms.service: scheduler
author: derek1ee
ms.author: deli
ms.reviewer: klam, estfan
ms.topic: article
ms.date: 08/15/2016
ms.openlocfilehash: 0a8d79af9f45731971cb1be1f39fc193f9d0f0d9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "80878975"
---
# <a name="outbound-authentication-for-azure-scheduler"></a>Исходящая проверка подлинности планировщика Azure

> [!IMPORTANT]
> Поддержка планировщика Azure [прекращается](../logic-apps/logic-apps-overview.md). Вместо него будет использоваться [Azure Logic Apps](../scheduler/migrate-from-scheduler-to-logic-apps.md#retire-date). Чтобы продолжить работу с настроенными в планировщике задачами, [перейдите на Azure Logic Apps](../scheduler/migrate-from-scheduler-to-logic-apps.md) как можно скорее. 
>
> Планировщик больше не доступен на портале Azure, но [REST API](/rest/api/scheduler) и [командлеты PowerShell Планировщика Azure](scheduler-powershell-reference.md) можно использовать и дальше для управления заданиями и коллекциями заданий.

Задания планировщика Azure могут вызывать службы, требующие проверки подлинности. В число таких служб входят другие службы Azure, Salesforce.com, Facebook и защищенные пользовательские веб-сайты. Вызванная служба определяет, может ли задание планировщика получить доступ к запрашиваемым ресурсам. 

Планировщик поддерживает следующие модели проверки подлинности: 

* Проверка с помощью *сертификата клиента* при использовании SSL- или TLS-сертификатов клиента
* *Обычная* проверка подлинности
* Проверка подлинности *Azure Active Directory (AD) OAuth*

## <a name="add-or-remove-authentication"></a>Добавление или удаление проверки подлинности

* Чтобы добавить проверку подлинности в задание планировщика, при создании или обновлении задания добавьте дочерний элемент `authentication` JavaScript Object Notation (JSON) в элемент `request`. 

  В ответах никогда не отображаются секретные данные, передаваемые службе планировщика запросами PUT, POST или PATCH в составе объекта `authentication`. 
  Ответы обнуляют секретную информацию или могут использовать открытый маркер, который представляет сущность, прошедшую проверку подлинности. 

* Чтобы удалить проверку подлинности из задания планировщика, явным образом выполните запрос PUT или PATCH к заданию и установите для объекта `authentication` нулевое значение. Никакие свойства проверки подлинности в ответе показаны не будут.

## <a name="client-certificate"></a>Сертификат клиента

### <a name="request-body---client-certificate"></a>Текст запроса для проверки подлинности с помощью сертификата клиента

При добавлении проверки подлинности с использованием модели `ClientCertificate` укажите в тексте запроса следующие дополнительные элементы.  

| Элемент | Обязательно | Описание |
|---------|----------|-------------|
| **authentication** (родительский элемент) | Объект проверки подлинности для использования сертификата SSL/TLS клиента |
| **type** | Да | Тип проверки подлинности. Для сертификатов SSL/TLS клиента это значение равно `ClientCertificate`. |
| **pfx** | Да | Содержимое PFX-файла в кодировке Base64 |
| **password** | Да | Пароль для доступа к PFX-файлу |
||| 

### <a name="response-body---client-certificate"></a>Текст ответа при проверке подлинности с помощью сертификата клиента 

При отправке запроса со сведениями о проверке подлинности в ответ включаются следующие элементы проверки подлинности.

| Элемент | Описание | 
|---------|-------------| 
| **authentication** (родительский элемент) | Объект проверки подлинности для использования сертификата SSL/TLS клиента |
| **type** | Тип проверки подлинности. Для сертификатов SSL/TLS клиента это значение равно `ClientCertificate`. |
| **certificateThumbprint** |Отпечаток сертификата |
| **certificateSubjectName** |Различающееся имя субъекта сертификата |
| **certificateExpiration** | Дата окончания срока действия сертификата |
||| 

### <a name="sample-rest-request---client-certificate"></a>Пример запроса REST для проверки подлинности с помощью сертификата клиента

```json
PUT https://management.azure.com/subscriptions/<Azure-subscription-ID>/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobcollections/southeastasiajc/jobs/httpjob?api-version=2016-01-01 HTTP/1.1
User-Agent: Fiddler
Host: management.azure.com
Authorization: Bearer sometoken
Content-Type: application/json; charset=utf-8

{
  "properties": {
    "startTime": "2015-05-14T14:10:00Z",
    "action": {
      "request": {
        "uri": "https://mywebserviceendpoint.com",
        "method": "GET",
        "headers": {
          "x-ms-version": "2013-03-01"
        },
        "authentication": {
          "type": "clientcertificate",
          "password": "password",
          "pfx": "pfx key"
        }
      },
      "type": "http"
    },
    "recurrence": {
      "frequency": "minute",
      "endTime": "2016-04-10T08:00:00Z",
      "interval": 1
    },
    "state": "enabled"
  }
}
```

### <a name="sample-rest-response---client-certificate"></a>Пример ответа REST при проверке подлинности с помощью сертификата клиента

```json
HTTP/1.1 200 OKCache-Control: no-cache
Pragma: no-cache
Content-Length: 858
Content-Type: application/json; charset=utf-8
Expires: -1
x-ms-request-id: 56c7b40e-721a-437e-88e6-f68562a73aa8
Server: Microsoft-IIS/8.5
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
x-ms-ratelimit-remaining-subscription-resource-requests: 599
x-ms-correlation-request-id: 1075219e-e879-4030-bc81-094e54fbabce
x-ms-routing-request-id: WESTUS:20160316T190424Z:1075219e-e879-4030-bc81-094e54fbabce
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 16 Mar 2016 19:04:23 GMT

{
  "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobCollections/southeastasiajc/jobs/httpjob",
  "type": "Microsoft.Scheduler/jobCollections/jobs",
  "name": "southeastasiajc/httpjob",
  "properties": {
    "startTime": "2015-05-14T14:10:00Z",
    "action": {
      "request": {
        "uri": "https://mywebserviceendpoint.com",
        "method": "GET",
        "headers": {
          "x-ms-version": "2013-03-01"
        },
        "authentication": {
          "certificateThumbprint": "88105CG9DF9ADE75B835711D899296CB217D7055",
          "certificateExpiration": "2021-01-01T07:00:00Z",
          "certificateSubjectName": "CN=Scheduler Mgmt",
          "type": "ClientCertificate"
        }
      },
      "type": "http"
    },
    "recurrence": {
      "frequency": "minute",
      "endTime": "2016-04-10T08:00:00Z",
      "interval": 1
    },
    "state": "enabled",
    "status": {
      "nextExecutionTime": "2016-03-16T19:05:00Z",
      "executionCount": 0,
      "failureCount": 0,
      "faultedCount": 0
    }
  }
}
```

## <a name="basic"></a>Basic

### <a name="request-body---basic"></a>Текст запроса для обычной проверки подлинности

При добавлении проверки подлинности с использованием модели `Basic` укажите в тексте запроса следующие дополнительные элементы.

| Элемент | Обязательно | Описание |
|---------|----------|-------------|
| **authentication** (родительский элемент) | Объект проверки подлинности для использования обычной проверки подлинности | 
| **type** | Да | Тип проверки подлинности. Для обычной проверки подлинности это значение равно `Basic`. | 
| **username** | Да | Имя пользователя для проверки подлинности | 
| **password** | Да | Пароль для проверки подлинности |
|||| 

### <a name="response-body---basic"></a>Текст ответа при обычной проверке подлинности

При отправке запроса со сведениями о проверке подлинности в ответ включаются следующие элементы проверки подлинности.

| Элемент | Описание | 
|---------|-------------|
| **authentication** (родительский элемент) | Объект проверки подлинности для использования обычной проверки подлинности |
| **type** | Тип проверки подлинности. Для обычной проверки подлинности это значение равно `Basic`. |
| **username** | Имя пользователя, прошедшего проверку подлинности |
||| 

### <a name="sample-rest-request---basic"></a>Пример запроса REST для обычной проверки подлинности

```json
PUT https://management.azure.com/subscriptions/<Azure-subscription-ID>/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobcollections/southeastasiajc/jobs/httpjob?api-version=2016-01-01 HTTP/1.1
User-Agent: Fiddler
Host: management.azure.com
Authorization: Bearer sometoken
Content-Length: 562
Content-Type: application/json; charset=utf-8

{
  "properties": {
    "startTime": "2015-05-14T14:10:00Z",
    "action": {
      "request": {
        "uri": "https://mywebserviceendpoint.com",
        "method": "GET",
        "headers": {
          "x-ms-version": "2013-03-01"
        },
        "authentication": {
          "type": "basic",
          "username": "user",
          "password": "password"
        }
      },
      "type": "http"
    },
    "recurrence": {
      "frequency": "minute",
      "endTime": "2016-04-10T08:00:00Z",
      "interval": 1
    },
    "state": "enabled"
  }
}
```

### <a name="sample-rest-response---basic"></a>Пример ответа REST для обычной проверки подлинности

```json
HTTP/1.1 200 OK
Cache-Control: no-cache
Pragma: no-cache
Content-Length: 701
Content-Type: application/json; charset=utf-8
Expires: -1
x-ms-request-id: a2dcb9cd-1aea-4887-8893-d81273a8cf04
Server: Microsoft-IIS/8.5
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
x-ms-ratelimit-remaining-subscription-resource-requests: 599
x-ms-correlation-request-id: 7816f222-6ea7-468d-b919-e6ddebbd7e95
x-ms-routing-request-id: WESTUS:20160316T190506Z:7816f222-6ea7-468d-b919-e6ddebbd7e95
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 16 Mar 2016 19:05:06 GMT

{  
   "id":"/subscriptions/<Azure-subscription-ID>/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobCollections/southeastasiajc/jobs/httpjob",
   "type":"Microsoft.Scheduler/jobCollections/jobs",
   "name":"southeastasiajc/httpjob",
   "properties":{  
      "startTime":"2015-05-14T14:10:00Z",
      "action":{  
         "request":{  
            "uri":"https://mywebserviceendpoint.com",
            "method":"GET",
            "headers":{  
               "x-ms-version":"2013-03-01"
            },
            "authentication":{  
               "username":"user1",
               "type":"Basic"
            }
         },
         "type":"Http"
      },
      "recurrence":{  
         "frequency":"Minute",
         "endTime":"2016-04-10T08:00:00Z",
         "interval":1
      },
      "state":"Enabled",
      "status":{  
         "nextExecutionTime":"2016-03-16T19:06:00Z",
         "executionCount":0,
         "failureCount":0,
         "faultedCount":0
      }
   }
}
```

## <a name="active-directory-oauth"></a>Active Directory OAuth

### <a name="request-body---active-directory-oauth"></a>Текст запроса для проверки подлинности Active Directory OAuth 

При добавлении проверки подлинности с использованием модели `ActiveDirectoryOAuth` укажите в тексте запроса следующие дополнительные элементы.

| Элемент | Обязательно | Описание |
|---------|----------|-------------|
| **authentication** (родительский элемент) | Да | Объект проверки подлинности для использования проверки подлинности ActiveDirectoryOAuth. |
| **type** | Да | Тип проверки подлинности. Для аутентификации ActiveDirectoryOAuth это значение равно `ActiveDirectoryOAuth`. |
| **tenant** | Да | Идентификатор клиента Azure AD. Идентификатор клиента Azure AD можно узнать, выполнив команду `Get-AzureAccount` в Azure PowerShell. |
| **audience** | Да | Этот параметр имеет значение `https://management.core.windows.net/`. | 
| **clientId** | Да | Идентификатор клиента для приложения Azure AD | 
| **secret** | Да | Секретные данные клиента, запрашивающего маркер. | 
|||| 

### <a name="response-body---active-directory-oauth"></a>Текст ответа при проверке подлинности Active Directory OAuth

При отправке запроса со сведениями о проверке подлинности в ответ включаются следующие элементы проверки подлинности.

| Элемент | Описание |
|---------|-------------|
| **authentication** (родительский элемент) | Объект проверки подлинности для использования проверки подлинности ActiveDirectoryOAuth. |
| **type** | Тип проверки подлинности. Для аутентификации ActiveDirectoryOAuth это значение равно `ActiveDirectoryOAuth`. | 
| **tenant** | Идентификатор клиента Azure AD. |
| **audience** | Этот параметр имеет значение `https://management.core.windows.net/`. |
| **clientId** | Идентификатор клиента для приложения Azure AD |
||| 

### <a name="sample-rest-request---active-directory-oauth"></a>Пример запроса REST для проверки подлинности Active Directory OAuth

```json
PUT https://management.azure.com/subscriptions/<Azure-subscription-ID>/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobcollections/southeastasiajc/jobs/httpjob?api-version=2016-01-01 HTTP/1.1
User-Agent: Fiddler
Host: management.azure.com
Authorization: Bearer sometoken
Content-Length: 757
Content-Type: application/json; charset=utf-8

{
  "properties": {
    "startTime": "2015-05-14T14:10:00Z",
    "action": {
      "request": {
        "uri": "https://mywebserviceendpoint.com",
        "method": "GET",
        "headers": {
          "x-ms-version": "2013-03-01"
        },
        "authentication": {
          "tenant":"microsoft.onmicrosoft.com",
          "audience":"https://management.core.windows.net/",
          "clientId":"dc23e764-9be6-4a33-9b9a-c46e36f0c137",
          "secret": "G6u071r8Gjw4V4KSibnb+VK4+tX399hkHaj7LOyHuj5=",
          "type":"ActiveDirectoryOAuth"
        }
      },
      "type": "Http"
    },
    "recurrence": {
      "frequency": "Minute",
      "endTime": "2016-04-10T08:00:00Z",
      "interval": 1
    },
    "state": "Enabled"
  }
}
```

### <a name="sample-rest-response---active-directory-oauth"></a>Пример ответа REST при проверке подлинности Active Directory OAuth

```json
HTTP/1.1 200 OK
Cache-Control: no-cache
Pragma: no-cache
Content-Length: 885
Content-Type: application/json; charset=utf-8
Expires: -1
x-ms-request-id: 86d8e9fd-ac0d-4bed-9420-9baba1af3251
Server: Microsoft-IIS/8.5
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
x-ms-ratelimit-remaining-subscription-resource-requests: 599
x-ms-correlation-request-id: 5183bbf4-9fa1-44bb-98c6-6872e3f2e7ce
x-ms-routing-request-id: WESTUS:20160316T191003Z:5183bbf4-9fa1-44bb-98c6-6872e3f2e7ce
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 16 Mar 2016 19:10:02 GMT

{
   "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobCollections/southeastasiajc/jobs/httpjob",
   "type": "Microsoft.Scheduler/jobCollections/jobs",
   "name": "southeastasiajc/httpjob",
   "properties": {
      "startTime": "2015-05-14T14:10:00Z",
      "action": {  
         "request": {
            "uri": "https://mywebserviceendpoint.com",
            "method": "GET",
            "headers": {  
               "x-ms-version": "2013-03-01"
            },
            "authentication": {  
               "tenant": "microsoft.onmicrosoft.com",
               "audience": "https://management.core.windows.net/",
               "clientId": "dc23e764-9be6-4a33-9b9a-c46e36f0c137",
               "type": "ActiveDirectoryOAuth"
            }
         },
         "type": "Http"
      },
      "recurrence": {  
         "frequency": "minute",
         "endTime": "2016-04-10T08:00:00Z",
         "interval": 1
      },
      "state": "Enabled",
      "status": {  
         "lastExecutionTime": "2016-03-16T19:10:00.3762123Z",
         "nextExecutionTime": "2016-03-16T19:11:00Z",
         "executionCount": 5,
         "failureCount": 5,
         "faultedCount": 1
      }
   }
}
```

## <a name="next-steps"></a>Дальнейшие действия

* [Основные понятия, терминология и иерархия сущностей планировщика Azure](scheduler-concepts-terms.md)
* [Ограничения, значения по умолчанию и коды ошибок планировщика Azure](scheduler-limits-defaults-errors.md)
* [Справочник по API REST планировщика Azure](/rest/api/scheduler)
* [Справочник по командлетам PowerShell планировщика Azure](scheduler-powershell-reference.md)