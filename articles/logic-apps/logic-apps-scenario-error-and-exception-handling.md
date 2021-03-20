---
title: Обработка исключений & сценарий регистрации ошибок
description: Реальный вариант использования и сценарий для расширенной обработки исключений и ведения журнала ошибок в Azure Logic Apps
services: logic-apps
ms.suite: integration
author: hedidin
ms.reviewer: klam, estfan, logicappspm
ms.topic: article
ms.date: 07/29/2016
ms.openlocfilehash: fdf5f25ae6f89ccc06c95ee1be021691dab0047a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96000357"
---
# <a name="scenario-exception-handling-and-error-logging-for-logic-apps"></a>Сценарий обработки исключений и ведения журнала ошибок для приложений логики

В этом сценарии описано, как улучшить поддержку для обработки исключений в приложении логики. Мы использовали пример из реальной жизни, который можно считать ответом на вопрос "Поддерживает ли Azure Logic Apps обработку исключений и ошибок?".

> [!NOTE]
> В текущей схеме Azure Logic Apps предоставляется стандартный шаблон для ответов на действия. В этот шаблон входят внутренние проверки и обработка сообщений об ошибках, возвращаемых из приложения API.

## <a name="scenario-and-use-case-overview"></a>Обзор сценария и варианта использование

Ниже приводится история, которая лежит в основе варианта использования для данного сценария. 

Известная медицинская организация поручила нам разработать решение на базе Azure, которое позволит создать портал для пациентов с использованием Microsoft Dynamics CRM Online. Также им нужно передавать информацию о приемах между порталом пациентов на базе Dynamics CRM Online и системой Salesforce. Для передачи информации о пациентах необходимо использовать стандарт [HL7 FHIR](https://www.hl7.org/implement/standards/fhir/) .

Проект включает два основных требования:  

* метод ведения журнала записей, отправленных с онлайн-портала Dynamics CRM Online;
* механизм для просмотра всех ошибок, возникающих в рамках рабочего процесса.

> [!TIP]
> Подробное видео об этом проекте можно посмотреть на сайте [Integration User Group](http://www.integrationusergroup.com/logic-apps-support-error-handling/ "Integration User Group").

## <a name="how-we-solved-the-problem"></a>Решение проблемы

Мы выбрали [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/ "Azure Cosmos DB") в качестве репозитория для записей журнала и ошибок (Cosmos DB ссылается на записи в виде документов). В Azure Logic Apps используется стандартный шаблон для всех ответов, поэтому нам не пришлось создавать настраиваемую схему. Чтобы **добавлять** и **запрашивать** записи журнала и записи об ошибках, можно было создать приложение API. В нем мы могли определить схему выполнения каждой из этих операций.  

Но существовало еще одно требование: автоматическое удаление записей через определенное время. Cosmos DB имеет свойство с названием срок [жизни](https://azure.microsoft.com/blog/documentdb-now-supports-time-to-live-ttl/ "Срок жизни") (TTL), которое позволило нам задать значение **времени** жизни для каждой записи или коллекции. Эта возможность избавляет от необходимости вручную удалять записи из Cosmos DB.

> [!IMPORTANT]
> Для работы с этим руководством необходимо создать базу данных Cosmos DB и две коллекции ("Ведение журнала" и "Ошибки").

## <a name="create-the-logic-app"></a>Создание приложения логики

Прежде всего необходимо создать приложение логики и открыть его в конструкторе приложений логики. В этом примере мы используем родительское и дочерние приложения логики. Предположим, мы уже создали родительское приложение, а теперь создаем одно дочернее приложение логики.

Так как нам нужен журнал записей, отправляемых из Dynamics CRM Online, мы начнем с верхнего уровня. Родительское приложение логики активирует дочернее, поэтому нам нужен триггер **запроса**.

### <a name="logic-app-trigger"></a>Триггер приложения логики

Мы используем триггер **запроса**, как показано в следующем примере:

``` json
"triggers": {
        "request": {
          "type": "request",
          "kind": "http",
          "inputs": {
            "schema": {
              "properties": {
                "CRMid": {
                  "type": "string"
                },
                "recordType": {
                  "type": "string"
                },
                "salesforceID": {
                  "type": "string"
                },
                "update": {
                  "type": "boolean"
                }
              },
              "required": [
                "CRMid",
                "recordType",
                "salesforceID",
                "update"
              ],
              "type": "object"
            }
          }
        }
      },

```


## <a name="steps"></a>Шаги

Необходимо записать в журнал источник (запрос) записи пациента, полученный с портала Dynamics CRM Online.

1. Необходимо получить новую запись о приеме с портала Dynamics CRM Online.

   Триггер, полученный из CRM, содержит следующие сведения: **идентификатор пациента CRM**, **тип записи**, **состояние записи — новая или обновленная** (логическое значение) и **идентификатор Salesforce**. Параметр **SalesforceId** (идентификатор Salesforce) может иметь значение NULL, так как он используется только для обновлений.
   Для получения записи CRM используются параметры **PatientID** (идентификатор пациента) и **Тип записи** службы CRM.

2. Далее необходимо добавить операцию **InsertLogEntry** для приложения API SQL Azure Cosmos DB, как показано ниже в конструкторе приложений логики.

   **Добавление записи журнала**

   ![Снимок экрана: конструктор приложений логики, отображающий параметры конфигурации для Insertlogentry для.](media/logic-apps-scenario-error-and-exception-handling/lognewpatient.png)

   **Добавление записи об ошибке**

   ![Снимок экрана: конструктор приложений логики, отображающий параметры конфигурации для Креатирроррекорд.](media/logic-apps-scenario-error-and-exception-handling/insertlogentry.png)

   **Проверка на ошибку создания записи**

   ![Снимок экрана: экран Креатирроррекорд в конструкторе приложений логики, отображающий поля для создания записи об ошибке.](media/logic-apps-scenario-error-and-exception-handling/condition.png)

## <a name="logic-app-source-code"></a>Исходный код приложения логики

> [!NOTE]
> Приведенные ниже фрагменты кода используется только в качестве примера. Так как в этом руководстве используется пример из реальной жизни, который уже реализован в рабочей среде, значения свойств **исходного узла**, связанные с планированием приемов, могут не отображаться. 

### <a name="logging"></a>Ведение журнала

В следующем примере кода приложения логики показано, как обрабатывать ведение журналов.

#### <a name="log-entry"></a>Запись журнала

Вот исходный код приложения логики для добавления записи в журнал:

``` json
"InsertLogEntry": {
        "metadata": {
        "apiDefinitionUrl": "https://.../swagger/docs/v1",
        "swaggerSource": "website"
        },
        "type": "Http",
        "inputs": {
        "body": {
            "date": "@{outputs('Gets_NewPatientRecord')['headers']['Date']}",
            "operation": "New Patient",
            "patientId": "@{triggerBody()['CRMid']}",
            "providerId": "@{triggerBody()['providerID']}",
            "source": "@{outputs('Gets_NewPatientRecord')['headers']}"
        },
        "method": "post",
        "uri": "https://.../api/Log"
        },
        "runAfter":    {
            "Gets_NewPatientecord": ["Succeeded"]
        }
}
```

#### <a name="log-request"></a>Запрос к журналу

Вот сообщение с запросом к журналу, переданное в приложение API:

``` json
    {
    "uri": "https://.../api/Log",
    "method": "post",
    "body": {
        "date": "Fri, 10 Jun 2016 22:31:56 GMT",
        "operation": "New Patient",
        "patientId": "6b115f6d-a7ee-e511-80f5-3863bb2eb2d0",
        "providerId": "",
        "source": "{/"Pragma/":/"no-cache/",/"x-ms-request-id/":/"e750c9a9-bd48-44c4-bbba-1688b6f8a132/",/"OData-Version/":/"4.0/",/"Cache-Control/":/"no-cache/",/"Date/":/"Fri, 10 Jun 2016 22:31:56 GMT/",/"Set-Cookie/":/"ARRAffinity=785f4334b5e64d2db0b84edcc1b84f1bf37319679aefce206b51510e56fd9770;Path=/;Domain=127.0.0.1/",/"Server/":/"Microsoft-IIS/8.0,Microsoft-HTTPAPI/2.0/",/"X-AspNet-Version/":/"4.0.30319/",/"X-Powered-By/":/"ASP.NET/",/"Content-Length/":/"1935/",/"Content-Type/":/"application/json; odata.metadata=minimal; odata.streaming=true/",/"Expires/":/"-1/"}"
        }
    }

```


#### <a name="log-response"></a>Ответ журнала

Вот сообщение с ответом из журнала, полученное из приложения API:

``` json
{
    "statusCode": 200,
    "headers": {
        "Pragma": "no-cache",
        "Cache-Control": "no-cache",
        "Date": "Fri, 10 Jun 2016 22:32:17 GMT",
        "Server": "Microsoft-IIS/8.0",
        "X-AspNet-Version": "4.0.30319",
        "X-Powered-By": "ASP.NET",
        "Content-Length": "964",
        "Content-Type": "application/json; charset=utf-8",
        "Expires": "-1"
    },
    "body": {
        "ttl": 2592000,
        "id": "6b115f6d-a7ee-e511-80f5-3863bb2eb2d0_1465597937",
        "_rid": "XngRAOT6IQEHAAAAAAAAAA==",
        "_self": "dbs/XngRAA==/colls/XngRAOT6IQE=/docs/XngRAOT6IQEHAAAAAAAAAA==/",
        "_ts": 1465597936,
        "_etag": "/"0400fc2f-0000-0000-0000-575b3ff00000/"",
        "patientID": "6b115f6d-a7ee-e511-80f5-3863bb2eb2d0",
        "timestamp": "2016-06-10T22:31:56Z",
        "source": "{/"Pragma/":/"no-cache/",/"x-ms-request-id/":/"e750c9a9-bd48-44c4-bbba-1688b6f8a132/",/"OData-Version/":/"4.0/",/"Cache-Control/":/"no-cache/",/"Date/":/"Fri, 10 Jun 2016 22:31:56 GMT/",/"Set-Cookie/":/"ARRAffinity=785f4334b5e64d2db0b84edcc1b84f1bf37319679aefce206b51510e56fd9770;Path=/;Domain=127.0.0.1/",/"Server/":/"Microsoft-IIS/8.0,Microsoft-HTTPAPI/2.0/",/"X-AspNet-Version/":/"4.0.30319/",/"X-Powered-By/":/"ASP.NET/",/"Content-Length/":/"1935/",/"Content-Type/":/"application/json; odata.metadata=minimal; odata.streaming=true/",/"Expires/":/"-1/"}",
        "operation": "New Patient",
        "salesforceId": "",
        "expired": false
    }
}

```

Теперь давайте рассмотрим действия по обработке ошибок.

### <a name="error-handling"></a>Обработка ошибок

В следующем примере кода приложения логики показано, как можно реализовать обработку ошибок.

#### <a name="create-error-record"></a>Создание записи об ошибке

Вот исходный код приложения логики, который используется для создания записи об ошибке:

``` json
"actions": {
    "CreateErrorRecord": {
        "metadata": {
        "apiDefinitionUrl": "https://.../swagger/docs/v1",
        "swaggerSource": "website"
        },
        "type": "Http",
        "inputs": {
        "body": {
            "action": "New_Patient",
            "isError": true,
            "crmId": "@{triggerBody()['CRMid']}",
            "patientID": "@{triggerBody()['CRMid']}",
            "message": "@{body('Create_NewPatientRecord')['message']}",
            "providerId": "@{triggerBody()['providerId']}",
            "severity": 4,
            "source": "@{actions('Create_NewPatientRecord')['inputs']['body']}",
            "statusCode": "@{int(outputs('Create_NewPatientRecord')['statusCode'])}",
            "salesforceId": "",
            "update": false
        },
        "method": "post",
        "uri": "https://.../api/CrMtoSfError"
        },
        "runAfter":
        {
            "Create_NewPatientRecord": ["Failed" ]
        }
    }
}             
```

#### <a name="insert-error-into-cosmos-db--request"></a>Добавление записи об ошибке в Cosmos DB — запрос

``` json

{
    "uri": "https://.../api/CrMtoSfError",
    "method": "post",
    "body": {
        "action": "New_Patient",
        "isError": true,
        "crmId": "6b115f6d-a7ee-e511-80f5-3863bb2eb2d0",
        "patientId": "6b115f6d-a7ee-e511-80f5-3863bb2eb2d0",
        "message": "Salesforce failed to complete task: Message: duplicate value found: Account_ID_MED__c duplicates value on record with id: 001U000001c83gK",
        "providerId": "",
        "severity": 4,
        "salesforceId": "",
        "update": false,
        "source": "{/"Account_Class_vod__c/":/"PRAC/",/"Account_Status_MED__c/":/"I/",/"CRM_HUB_ID__c/":/"6b115f6d-a7ee-e511-80f5-3863bb2eb2d0/",/"Credentials_vod__c/",/"DTC_ID_MED__c/":/"/",/"Fax/":/"/",/"FirstName/":/"A/",/"Gender_vod__c/":/"/",/"IMS_ID__c/":/"/",/"LastName/":/"BAILEY/",/"MasterID_mp__c/":/"/",/"C_ID_MED__c/":/"851588/",/"Middle_vod__c/":/"/",/"NPI_vod__c/":/"/",/"PDRP_MED__c/":false,/"PersonDoNotCall/":false,/"PersonEmail/":/"/",/"PersonHasOptedOutOfEmail/":false,/"PersonHasOptedOutOfFax/":false,/"PersonMobilePhone/":/"/",/"Phone/":/"/",/"Practicing_Specialty__c/":/"FM - FAMILY MEDICINE/",/"Primary_City__c/":/"/",/"Primary_State__c/":/"/",/"Primary_Street_Line2__c/":/"/",/"Primary_Street__c/":/"/",/"Primary_Zip__c/":/"/",/"RecordTypeId/":/"012U0000000JaPWIA0/",/"Request_Date__c/":/"2016-06-10T22:31:55.9647467Z/",/"ONY_ID__c/":/"/",/"Specialty_1_vod__c/":/"/",/"Suffix_vod__c/":/"/",/"Website/":/"/"}",
        "statusCode": "400"
    }
}
```

#### <a name="insert-error-into-cosmos-db--response"></a>Добавление записи об ошибке в Cosmos DB — ответ

``` json
{
    "statusCode": 200,
    "headers": {
        "Pragma": "no-cache",
        "Cache-Control": "no-cache",
        "Date": "Fri, 10 Jun 2016 22:31:57 GMT",
        "Server": "Microsoft-IIS/8.0",
        "X-AspNet-Version": "4.0.30319",
        "X-Powered-By": "ASP.NET",
        "Content-Length": "1561",
        "Content-Type": "application/json; charset=utf-8",
        "Expires": "-1"
    },
    "body": {
        "id": "6b115f6d-a7ee-e511-80f5-3863bb2eb2d0-1465597917",
        "_rid": "sQx2APhVzAA8AAAAAAAAAA==",
        "_self": "dbs/sQx2AA==/colls/sQx2APhVzAA=/docs/sQx2APhVzAA8AAAAAAAAAA==/",
        "_ts": 1465597912,
        "_etag": "/"0c00eaac-0000-0000-0000-575b3fdc0000/"",
        "prescriberId": "6b115f6d-a7ee-e511-80f5-3863bb2eb2d0",
        "timestamp": "2016-06-10T22:31:57.3651027Z",
        "action": "New_Patient",
        "salesforceId": "",
        "update": false,
        "body": "CRM failed to complete task: Message: duplicate value found: CRM_HUB_ID__c duplicates value on record with id: 001U000001c83gK",
        "source": "{/"Account_Class_vod__c/":/"PRAC/",/"Account_Status_MED__c/":/"I/",/"CRM_HUB_ID__c/":/"6b115f6d-a7ee-e511-80f5-3863bb2eb2d0/",/"Credentials_vod__c/":/"DO - Degree level is DO/",/"DTC_ID_MED__c/":/"/",/"Fax/":/"/",/"FirstName/":/"A/",/"Gender_vod__c/":/"/",/"IMS_ID__c/":/"/",/"LastName/":/"BAILEY/",/"MterID_mp__c/":/"/",/"Medicis_ID_MED__c/":/"851588/",/"Middle_vod__c/":/"/",/"NPI_vod__c/":/"/",/"PDRP_MED__c/":false,/"PersonDoNotCall/":false,/"PersonEmail/":/"/",/"PersonHasOptedOutOfEmail/":false,/"PersonHasOptedOutOfFax/":false,/"PersonMobilePhone/":/"/",/"Phone/":/"/",/"Practicing_Specialty__c/":/"FM - FAMILY MEDICINE/",/"Primary_City__c/":/"/",/"Primary_State__c/":/"/",/"Primary_Street_Line2__c/":/"/",/"Primary_Street__c/":/"/",/"Primary_Zip__c/":/"/",/"RecordTypeId/":/"012U0000000JaPWIA0/",/"Request_Date__c/":/"2016-06-10T22:31:55.9647467Z/",/"XXXXXXX/":/"/",/"Specialty_1_vod__c/":/"/",/"Suffix_vod__c/":/"/",/"Website/":/"/"}",
        "code": 400,
        "errors": null,
        "isError": true,
        "severity": 4,
        "notes": null,
        "resolved": 0
        }
}
```

#### <a name="salesforce-error-response"></a>Ответ на ошибку Salesforce

``` json
{
    "statusCode": 400,
    "headers": {
        "Pragma": "no-cache",
        "x-ms-request-id": "3e8e4884-288e-4633-972c-8271b2cc912c",
        "X-Content-Type-Options": "nosniff",
        "Cache-Control": "no-cache",
        "Date": "Fri, 10 Jun 2016 22:31:56 GMT",
        "Set-Cookie": "ARRAffinity=785f4334b5e64d2db0b84edcc1b84f1bf37319679aefce206b51510e56fd9770;Path=/;Domain=127.0.0.1",
        "Server": "Microsoft-IIS/8.0,Microsoft-HTTPAPI/2.0",
        "X-AspNet-Version": "4.0.30319",
        "X-Powered-By": "ASP.NET",
        "Content-Length": "205",
        "Content-Type": "application/json; charset=utf-8",
        "Expires": "-1"
    },
    "body": {
        "status": 400,
        "message": "Salesforce failed to complete task: Message: duplicate value found: Account_ID_MED__c duplicates value on record with id: 001U000001c83gK",
        "source": "Salesforce.Common",
        "errors": []
    }
}

```

### <a name="return-the-response-back-to-parent-logic-app"></a>Возвращение ответа в родительское приложение логики

После получения ответа его можно вернуть в родительское приложение логики.

#### <a name="return-success-response-to-parent-logic-app"></a>Возвращение успешного ответа в родительское приложение логики

``` json
"SuccessResponse": {
    "runAfter":
        {
            "UpdateNew_CRMPatientResponse": ["Succeeded"]
        },
    "inputs": {
        "body": {
            "status": "Success"
    },
    "headers": {
    "    Content-type": "application/json",
        "x-ms-date": "@utcnow()"
    },
    "statusCode": 200
    },
    "type": "Response"
}
```

#### <a name="return-error-response-to-parent-logic-app"></a>Возвращение ответа на ошибку в родительское приложение логики

``` json
"ErrorResponse": {
    "runAfter":
        {
            "Create_NewPatientRecord": ["Failed"]
        },
    "inputs": {
        "body": {
            "status": "BadRequest"
        },
        "headers": {
            "Content-type": "application/json",
            "x-ms-date": "@utcnow()"
        },
        "statusCode": 400
    },
    "type": "Response"
}

```


## <a name="cosmos-db-repository-and-portal"></a>Репозиторий и портал Cosmos DB

Благодаря использованию [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db) наше решение предоставляет больше возможностей.

### <a name="error-management-portal"></a>Портал управления ошибками

Чтобы просмотреть ошибки, полученные из Cosmos DB, можно создать веб-приложение MVC, в котором будут отображаться соответствующие записи об ошибках. В текущую версию приложения входят операции **List**, **Details**, **Edit** и **Delete**.

> [!NOTE]
> Операция Edit: Cosmos DB заменяет весь документ. Записи в представлении списка (**List**) и в подробном представлении (**Detail**) приведены в качестве примера. Это не фактические записи о приемах пациентов.

Ниже приведены примеры сведений о приложении MVC, созданные с помощью описанного выше метода.

#### <a name="error-management-list"></a>Список для управления ошибками
![Список ошибок](media/logic-apps-scenario-error-and-exception-handling/errorlist.png)

#### <a name="error-management-detail-view"></a>Просмотр подробных сведений об ошибке
![Сведения об ошибке](media/logic-apps-scenario-error-and-exception-handling/errordetails.png)

### <a name="log-management-portal"></a>Портал управления журналами

Для просмотра журналов мы также создали веб-приложение MVC. Ниже приведены примеры сведений о приложении MVC, созданные с помощью описанного выше метода.

#### <a name="sample-log-detail-view"></a>Просмотр подробных сведений об ошибке из журнала
![Просмотр сведений о записи журнала](media/logic-apps-scenario-error-and-exception-handling/samplelogdetail.png)

### <a name="api-app-details"></a>Сведения о приложении API

#### <a name="logic-apps-exception-management-api"></a>API управления исключениями приложений логики

Приложение API на основе Azure Logic Apps с открытым исходным кодом, которое мы создали для управления исключениями, поддерживает следующие возможности, включая два контроллера:

* **ErrorController** добавляет запись об ошибке (документ) в коллекцию Azure Cosmos DB;
* **LogController** добавляет запись журнала (документ) в коллекцию Azure Cosmos DB.

> [!TIP]
> Оба контроллера используют операции `async Task<dynamic>`, что позволяет выполнять операции в среде выполнения, а значит, мы сможем создать схему Azure Cosmos DB в тексте операции. 
> 

Каждый документ в Azure Cosmos DB должен иметь уникальный идентификатор. Мы используем идентификатор `PatientId` , к которому добавляем метку времени, преобразованную в значение в формате Unix (значение с двойной точностью). Чтобы удалить дробное значение, мы обрежем метку времени.

Исходный код API контроллера ошибок можно просмотреть на сайте [GitHub](https://github.com/HEDIDIN/LogicAppsExceptionManagementApi/blob/master/LogicAppsExceptionManagementApi/Controllers/LogController.cs).

Для вызова API из приложения логики используется следующий синтаксис:

``` json
 "actions": {
        "CreateErrorRecord": {
          "metadata": {
            "apiDefinitionUrl": "https://.../swagger/docs/v1",
            "swaggerSource": "website"
          },
          "type": "Http",
          "inputs": {
            "body": {
              "action": "New_Patient",
              "isError": true,
              "crmId": "@{triggerBody()['CRMid']}",
              "prescriberId": "@{triggerBody()['CRMid']}",
              "message": "@{body('Create_NewPatientRecord')['message']}",
              "salesforceId": "@{triggerBody()['salesforceID']}",
              "severity": 4,
              "source": "@{actions('Create_NewPatientRecord')['inputs']['body']}",
              "statusCode": "@{int(outputs('Create_NewPatientRecord')['statusCode'])}",
              "update": false
            },
            "method": "post",
            "uri": "https://.../api/CrMtoSfError"
          },
          "runAfter": {
              "Create_NewPatientRecord": ["Failed"]
            }
        }
 }
```

Выражение в примере кода выше проверяет, не имеет ли параметр *Create_NewPatientRecord* значение **Failed**.

## <a name="summary"></a>Итоги

* В приложении логики можно легко реализовать ведение журнала и обработку ошибок.
* Azure Cosmos DB можно использовать в качестве репозитория для хранения записей журнала и записей об ошибках (документов).
* С помощью веб-приложения MVC можно создать портал, на котором будут отображаться записи журналов и записи об ошибках.

### <a name="source-code"></a>Исходный код

Исходный код приложения API управления исключениями Logic Apps доступен в этом [репозитории GitHub](https://github.com/HEDIDIN/LogicAppsExceptionManagementApi "API управления исключениями приложений логики").

## <a name="next-steps"></a>Дальнейшие действия

* [Примеры приложений логики и распространенные сценарии](../logic-apps/logic-apps-examples-and-scenarios.md)
* [Мониторинг приложений логики](../logic-apps/monitor-logic-apps.md)
* [Автоматизация развертывания приложений логики](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md)
