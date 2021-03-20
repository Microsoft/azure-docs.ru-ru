---
title: Веб-действие в фабрике данных Azure
description: Сведения об использовании веб-действия, одного из действий потока управления, поддерживаемых в фабрике данных, для вызова конечной точки REST из конвейера.
author: dcstwh
ms.author: weetok
ms.service: data-factory
ms.topic: conceptual
ms.date: 12/19/2018
ms.openlocfilehash: e4578b41e5cbb62c8a1bfa0c48d4fd60d042a506
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100361525"
---
# <a name="web-activity-in-azure-data-factory"></a>Веб-действие в фабрике данных Azure
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]


Веб-действие можно использовать для вызова из конвейера фабрики данных пользовательской конечной точки REST. Вы можете передать наборы данных и связанные службы, которые будет использовать это действие и к которым оно будет обращаться.

> [!NOTE]
> Веб-действия поддерживаются для вызова URL-адресов, размещенных в частной виртуальной сети, а также с помощью локальной среды выполнения интеграции. Среда выполнения интеграции должна содержать указание на конечную точку URL-адреса. 

> [!NOTE]
> Максимальный поддерживаемый размер полезных данных выходного ответа составляет 4 МБ.  

## <a name="syntax"></a>Синтаксис

```json
{
   "name":"MyWebActivity",
   "type":"WebActivity",
   "typeProperties":{
      "method":"Post",
      "url":"<URLEndpoint>",
      "connectVia": {
          "referenceName": "<integrationRuntimeName>",
          "type": "IntegrationRuntimeReference"
      }
      "headers":{
         "Content-Type":"application/json"
      },
      "authentication":{
         "type":"ClientCertificate",
         "pfx":"****",
         "password":"****"
      },
      "datasets":[
         {
            "referenceName":"<ConsumedDatasetName>",
            "type":"DatasetReference",
            "parameters":{
               ...
            }
         }
      ],
      "linkedServices":[
         {
            "referenceName":"<ConsumedLinkedServiceName>",
            "type":"LinkedServiceReference"
         }
      ]
   }
}

```

## <a name="type-properties"></a>Свойства типа

Свойство | Описание | Допустимые значения | Обязательно
-------- | ----------- | -------------- | --------
name | Имя веб-действия | Строка | Да
type | Необходимо задать значение **WebActivity**. | Строка | Да
method | Метод REST API для целевой конечной точки. | Строка. <br/><br/>Поддерживаемые типы: GET, POST и PUT | Да
url | Целевая конечная точка и путь | Строка (или выражение с типом результата "строка"). Действие завершится ошибкой времени ожидания через 1 минуту, если не получит ответ от конечной точки. | Да
Заголовки | Заголовки, которые отправляются в запрос. Например, чтобы задать язык и тип запроса: `"headers" : { "Accept-Language": "en-us", "Content-Type": "application/json" }`. | Строка (или выражение с типом результата "строка") | Да, требуется заголовок Content-type. `"headers":{ "Content-Type":"application/json"}`
текст | Представляет полезные данные, отправляемые конечной точке.  | Строка (или выражение с типом результата "строка"). <br/><br/>Просмотрите схему полезных данных запроса в разделе [Схема полезных данных запроса](#request-payload-schema). | Требуется для методов POST или PUT.
проверка подлинности | Метод проверки подлинности, используемый для вызова конечной точки. Поддерживаемые типы данных: Basic или ClientCertificate. Дополнительные сведения см. в разделе [Проверка подлинности](#authentication). Если проверка подлинности не требуется, исключите это свойство. | Строка (или выражение с типом результата "строка") | Нет
наборы данных | Список наборов данных, передаваемых в конечную точку. | Массив ссылок на наборы данных. Может быть пустым массивом. | Да
linkedServices | Список связанных служб, переданных в конечную точку. | Массив ссылок на связанные службы. Может быть пустым массивом. | Да
connectVia | [Среда выполнения интеграции](./concepts-integration-runtime.md), используемая для подключения к хранилищу данных. Вы можете использовать среду выполнения интеграции Azure или локальную среду выполнения интеграции (если хранилище данных находится в частной сети). Если это свойство не указано, служба использует среду выполнения интеграции Azure по умолчанию. | Справочник по среде выполнения интеграции. | Нет 

> [!NOTE]
> Конечные точки REST, которые вызывает веб-действие, должны возвращать ответ типа JSON. Действие завершится ошибкой времени ожидания через 1 минуту, если не получит ответ от конечной точки.

Ниже приведены требования к содержимому JSON:

| Тип значения | Текст запроса | Текст ответа |
|---|---|---|
|Объект JSON | Поддерживается | Поддерживается |
|Массив JSON | Поддерживается <br/>(В настоящее время массивы JSON не работают из-за ошибки. Исправление уже разрабатывается.) | Не поддерживается |
| Значение JSON | Поддерживается | Не поддерживается |
| Тип, отличный от JSON | Не поддерживается | Не поддерживается |
||||

## <a name="authentication"></a>Аутентификация

Ниже приведены поддерживаемые типы проверки подлинности в веб-действии.

### <a name="none"></a>Нет

Если проверка подлинности не требуется, не включайте свойство authentication.

### <a name="basic"></a>Basic

Укажите имя пользователя и пароль для использования с обычной проверкой подлинности.

```json
"authentication":{
   "type":"Basic",
   "username":"****",
   "password":"****"
}
```

### <a name="client-certificate"></a>Сертификат клиента

Укажите содержимое в кодировке base64 PFX-файла и пароль.

```json
"authentication":{
   "type":"ClientCertificate",
   "pfx":"****",
   "password":"****"
}
```

### <a name="managed-identity"></a>Управляемое удостоверение

Укажите URI ресурса, для которого маркер доступа будет запрошен с помощью управляемого удостоверения для фабрики данных. Для вызова API управления ресурсами Azure используйте `https://management.azure.com/`. Дополнительные сведения об управляемых удостоверениях для ресурсов Azure см. в статье [Что такое управляемые удостоверения для ресурсов Azure?](../active-directory/managed-identities-azure-resources/overview.md)

```json
"authentication": {
    "type": "MSI",
    "resource": "https://management.azure.com/"
}
```

> [!NOTE]
> Если для фабрики данных настроен репозиторий Git, необходимо сохранить учетные данные в Azure Key Vault, чтобы использовать проверку подлинности "базовый" или "сертификат клиента". В службе "Фабрика данных Azure" не хранятся пароли в Git.

## <a name="request-payload-schema"></a>Схема полезных данных запроса
При использовании метода POST или PUT свойство body представляет полезные данные, отправляемые конечной точке. Связанные службы и наборы данных можно передать в качестве части полезных данных. Ниже приведена схема полезных данных.

```json
{
    "body": {
        "myMessage": "Sample",
        "datasets": [{
            "name": "MyDataset1",
            "properties": {
                ...
            }
        }],
        "linkedServices": [{
            "name": "MyStorageLinkedService1",
            "properties": {
                ...
            }
        }]
    }
}
```

## <a name="example"></a>Пример
В этом примере веб-действие в конвейере вызывает конечную точку REST. Оно передает связанную службу SQL Azure и набор данных SQL Azure в конечную точку. Конечная точка остальное использует строку подключения SQL Azure для подключения к логическому серверу SQL Server и возвращает имя экземпляра SQL Server.

### <a name="pipeline-definition"></a>Определение конвейера

```json
{
    "name": "<MyWebActivityPipeline>",
    "properties": {
        "activities": [
            {
                "name": "<MyWebActivity>",
                "type": "WebActivity",
                "typeProperties": {
                    "method": "Post",
                    "url": "@pipeline().parameters.url",
                    "headers": {
                        "Content-Type": "application/json"
                    },
                    "authentication": {
                        "type": "ClientCertificate",
                        "pfx": "*****",
                        "password": "*****"
                    },
                    "datasets": [
                        {
                            "referenceName": "MySQLDataset",
                            "type": "DatasetReference",
                            "parameters": {
                                "SqlTableName": "@pipeline().parameters.sqlTableName"
                            }
                        }
                    ],
                    "linkedServices": [
                        {
                            "referenceName": "SqlLinkedService",
                            "type": "LinkedServiceReference"
                        }
                    ]
                }
            }
        ],
        "parameters": {
            "sqlTableName": {
                "type": "String"
            },
            "url": {
                "type": "String"
            }
        }
    }
}

```

### <a name="pipeline-parameter-values"></a>Значения параметров конвейера

```json
{
    "sqlTableName": "department",
    "url": "https://adftes.azurewebsites.net/api/execute/running"
}

```

### <a name="web-service-endpoint-code"></a>Код конечной точки веб-службы

```csharp

[HttpPost]
public HttpResponseMessage Execute(JObject payload)
{
    Trace.TraceInformation("Start Execute");

    JObject result = new JObject();
    result.Add("status", "complete");

    JArray datasets = payload.GetValue("datasets") as JArray;
    result.Add("sinktable", datasets[0]["properties"]["typeProperties"]["tableName"].ToString());

    JArray linkedServices = payload.GetValue("linkedServices") as JArray;
    string connString = linkedServices[0]["properties"]["typeProperties"]["connectionString"].ToString();

    System.Data.SqlClient.SqlConnection sqlConn = new System.Data.SqlClient.SqlConnection(connString);

    result.Add("sinkServer", sqlConn.DataSource);

    Trace.TraceInformation("Stop Execute");

    return this.Request.CreateResponse(HttpStatusCode.OK, result);
}

```

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь с другими действиями потока управления, которые поддерживаются фабрикой данных:

- [Действие выполнения конвейера](control-flow-execute-pipeline-activity.md)
- [Действие For Each](control-flow-for-each-activity.md)
- [Действие получения метаданных](control-flow-get-metadata-activity.md)
- [Действие поиска](control-flow-lookup-activity.md)
