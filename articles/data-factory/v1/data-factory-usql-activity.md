---
title: Преобразование данных с помощью скрипта U-SQL — Azure
description: Узнайте, как обрабатывать или преобразовывать данные, выполняя скрипты U-SQL в Azure Data Lake Analytics службы вычислений версии 1.
ms.service: data-factory
ms.topic: conceptual
ms.date: 10/01/2017
author: nabhishek
ms.author: abnarain
ms.custom: devx-track-csharp
robots: noindex
ms.openlocfilehash: 5931cb28721e8658a771ceea1cd94624a0c09f7c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100392924"
---
# <a name="transform-data-by-running-u-sql-scripts-on-azure-data-lake-analytics"></a>Преобразование данных с помощью сценариев U-SQL в Azure Data Lake Analytics 
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](data-factory-usql-activity.md)
> * [Версия 2 (текущая)](../transform-data-using-data-lake-analytics.md)

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. руководство по [применению действия U-SQL в службе "Фабрика данных Azure" версии 2](../transform-data-using-data-lake-analytics.md).

Конвейер в фабрике данных Azure обрабатывает данные в связанной службе хранилища с помощью связанных вычислительных служб. В нем содержится последовательность действий, каждое из которых выполняет определенную операцию обработки. В этой статье описывается **действие U-SQL в Data Lake Analytics**, которое запускает сценарий **U-SQL** в связанной службе вычислений **Azure Data Lake Analytics**. 

Перед созданием конвейера с действием U-SQL в Data Lake Analytics следует создать учетную запись Data Lake Analytics. Дополнительные сведения об Azure Data Lake Analytics см. в статье [Начало работы с аналитикой озера данных Azure](../../data-lake-analytics/data-lake-analytics-get-started-portal.md).

В руководстве по [созданию первого конвейера](data-factory-build-your-first-pipeline.md) подробно описаны процедуры создания фабрики данных, связанных служб, наборов данных и конвейера. Для создания сущностей фабрики данных запустите предложенные фрагменты кода JSON в редакторе фабрики данных, Visual Studio или Azure PowerShell.

## <a name="supported-authentication-types"></a>Поддерживаемые типы аутентификации
Действие U-SQL поддерживает указанные ниже типы проверки подлинности Data Lake Analytics.
* Проверка подлинности субъекта-службы
* Проверка подлинности учетных данных пользователя (OAuth) 

Мы рекомендуем использовать проверку подлинности субъекта-службы, особенно для выполнения U-SQL по расписанию. При проверке подлинности учетных данных пользователя может истечь срок действия маркера. Сведения о настройке см. в разделе [Свойства связанной службы](#azure-data-lake-analytics-linked-service).

## <a name="azure-data-lake-analytics-linked-service"></a>Связанная служба аналитики озера данных Azure
Можно создать связанную службу **Azure Data Lake Analytics** , чтобы связать службу вычислений Azure Data Lake Analytics с фабрикой данных Azure. Действие U-SQL Data Lake Analytics в конвейере ссылается на эту связанную службу. 

В следующей таблице приведены описания универсальных свойств из определения JSON. Вы можете выбирать между проверкой подлинности на основе субъекта-службы и учетных данных пользователя.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| **type** |Свойству type необходимо присвоить значение **AzureDataLakeAnalytics**. |Да |
| **accountName** |Имя учетной записи аналитики озера данных Azure. |Да |
| **dataLakeAnalyticsUri** |Универсальный код ресурса (URI) аналитики озера данных Azure. |Нет |
| **subscriptionId** |Идентификатор подписки Azure |Нет (если не указан, используется подписка фабрики данных). |
| **resourceGroupName** |Имя группы ресурсов Azure |Нет (если не указано, используется группа ресурсов фабрики данных). |

### <a name="service-principal-authentication-recommended"></a>Проверка подлинности на основе субъекта-службы (рекомендуется)
При использовании проверки подлинности на основе субъекта-службы необходимо зарегистрировать сущность приложения в Azure Active Directory (Azure AD) и предоставить ей доступ к Data Lake Store. Подробные инструкции см. в статье [Аутентификация между службами в Data Lake Store с помощью Azure Active Directory](../../data-lake-store/data-lake-store-service-to-service-authenticate-using-active-directory.md). Запишите следующие значения, которые используются для определения связанной службы:
* Идентификатор приложения
* Ключ приложения 
* Tenant ID

Используйте проверку подлинности на основе субъекта-службы, указав следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| **servicePrincipalId** | Укажите идентификатора клиента приложения. | Да |
| **servicePrincipalKey** | Укажите ключ приложения. | Да |
| **tenant** | Укажите сведения о клиенте (доменное имя или идентификатор клиента), в котором находится приложение. Эти сведения можно получить, наведя указатель мыши на правый верхний угол страницы портала Azure. | Да |

**Пример. Проверка подлинности субъекта-службы**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "adftestaccount",
            "dataLakeAnalyticsUri": "azuredatalakeanalytics.net",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        }
    }
}
```

### <a name="user-credential-authentication"></a>Использование проверки подлинности на основе учетных данных пользователя
Кроме того, для Data Lake Analytics можно использовать проверку подлинности на основе учетных данных пользователя, указав приведенные ниже свойства.

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| **проверки** | Нажмите кнопку **Авторизовать** в редакторе фабрики данных и введите учетные данные. URL-адрес авторизации будет создан автоматически и присвоен этому свойству. | Да |
| **sessionId** | Идентификатор сеанса OAuth из сеанса авторизации OAuth. Каждый идентификатор сеанса является уникальным и используется только один раз. Этот параметр создается автоматически при использовании редактора фабрики данных. | Да |

**Пример. Использование проверки подлинности на основе учетных данных пользователя**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "adftestaccount",
            "dataLakeAnalyticsUri": "azuredatalakeanalytics.net",
            "authorization": "<authcode>",
            "sessionId": "<session ID>", 
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        }
    }
}
```

#### <a name="token-expiration"></a>Срок действия маркера
Срок действия кода авторизации, созданного с помощью кнопки **Авторизовать**, через некоторое время истекает. Сроки действия для различных типов учетных записей пользователей см. в следующей таблице. По истечении **срока действия маркера** проверки подлинности может появиться следующее сообщение об ошибке: "Произошла ошибка при операции с учетными данными: invalid_grant — AADSTS70002: ошибка при проверке учетных данных". AADSTS70008: срок действия предоставленных прав доступа истек или они были отозваны. Идентификатор отслеживания: d18629e8-af88-43c5-88e3-d8419eb1fca1 Идентификатор корреляции: fac30a0c-6be6-4e02-8d69-a776d2ffefd7 Временная отметка: 2015-12-15 21:09:31Z".

| Тип пользователя | Срок действия |
|:--- |:--- |
| Учетные записи пользователей, которые не управляются с помощью Azure Active Directory (@hotmail.com, @live.com и т. д.) |12 часов |
| Учетные записи пользователей, которые управляются Azure Active Directory (AAD) |14 дней после последнего запуска среза. <br/><br/>90 дней, если срез, основанный на связанной службе на основе OAuth, выполняется по крайней мере раз в 14 дней. |

Чтобы избежать этой ошибки или исправить ее, вам потребуется повторно авторизоваться с помощью кнопки **Авторизовать** и повторно развернуть связанную службу, когда **срок действия маркера истечет**. Значения свойств **sessionId** и **authorization** также можно задавать программно с помощью кода:

```csharp
if (linkedService.Properties.TypeProperties is AzureDataLakeStoreLinkedService ||
    linkedService.Properties.TypeProperties is AzureDataLakeAnalyticsLinkedService)
{
    AuthorizationSessionGetResponse authorizationSession = this.Client.OAuth.Get(this.ResourceGroupName, this.DataFactoryName, linkedService.Properties.Type);

    WindowsFormsWebAuthenticationDialog authenticationDialog = new WindowsFormsWebAuthenticationDialog(null);
    string authorization = authenticationDialog.AuthenticateAAD(authorizationSession.AuthorizationSession.Endpoint, new Uri("urn:ietf:wg:oauth:2.0:oob"));

    AzureDataLakeStoreLinkedService azureDataLakeStoreProperties = linkedService.Properties.TypeProperties as AzureDataLakeStoreLinkedService;
    if (azureDataLakeStoreProperties != null)
    {
        azureDataLakeStoreProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeStoreProperties.Authorization = authorization;
    }

    AzureDataLakeAnalyticsLinkedService azureDataLakeAnalyticsProperties = linkedService.Properties.TypeProperties as AzureDataLakeAnalyticsLinkedService;
    if (azureDataLakeAnalyticsProperties != null)
    {
        azureDataLakeAnalyticsProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeAnalyticsProperties.Authorization = authorization;
    }
}
```

Подробные сведения о классах фабрики данных, используемых в коде, см. в статьях [AzureDataLakeStoreLinkedService — класс](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakestorelinkedservice), [AzureDataLakeAnalyticsLinkedService — класс](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakeanalyticslinkedservice) и [AuthorizationSessionGetResponse — класс](/dotnet/api/microsoft.azure.management.datafactories.models.authorizationsessiongetresponse). Для использования класса WindowsFormsWebAuthenticationDialog следует добавить ссылку на Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms.dll. 

## <a name="data-lake-analytics-u-sql-activity"></a>Действие U-SQL в Data Lake Analytics
В следующем фрагменте кода JSON определяется конвейер с действием U-SQL в аналитике озера данных. Определение действия содержит ссылку на созданную ранее связанную службу аналитики озера данных Azure.   

```json
{
    "name": "ComputeEventsByRegionPipeline",
    "properties": {
        "description": "This is a pipeline to compute events for en-gb locale and date less than 2012/02/19.",
        "activities": 
        [
            {
                "type": "DataLakeAnalyticsU-SQL",
                "typeProperties": {
                    "scriptPath": "scripts\\kona\\SearchLogProcessing.txt",
                    "scriptLinkedService": "StorageLinkedService",
                    "degreeOfParallelism": 3,
                    "priority": 100,
                    "parameters": {
                        "in": "/datalake/input/SearchLog.tsv",
                        "out": "/datalake/output/Result.tsv"
                    }
                },
                "inputs": [
                    {
                        "name": "DataLakeTable"
                    }
                ],
                "outputs": 
                [
                    {
                        "name": "EventsByRegionTable"
                    }
                ],
                "policy": {
                    "timeout": "06:00:00",
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "retry": 1
                },
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1
                },
                "name": "EventsByRegion",
                "linkedServiceName": "AzureDataLakeAnalyticsLinkedService"
            }
        ],
        "start": "2015-08-08T00:00:00Z",
        "end": "2015-08-08T01:00:00Z",
        "isPaused": false
    }
}
```

В следующей таблице описаны имена и описания свойств, относящихся к этому действию. 

| Свойство            | Описание                              | Обязательно                                 |
| :------------------ | :--------------------------------------- | :--------------------------------------- |
| type                | Для свойства type нужно задать значение **DataLakeAnalyticsU-SQL**. | Да                                      |
| linkedServiceName   | Ссылка на службу Azure Data Lake Analytics, зарегистрированную в качестве связанной службы в фабрике данных | Да                                      |
| scriptPath          | Путь к папке, содержащей скрипт U-SQL В имени файла учитывается регистр. | Нет (если используется скрипт)                   |
| scriptLinkedService | Связанная служба, которая связывает хранилище, содержащее скрипт, с фабрикой данных | Нет (если используется скрипт)                   |
| скрипт              | Указание сценария непосредственно в строке вместо использования scriptPath и scriptLinkedService. Например, `"script": "CREATE DATABASE test"`. | Нет (при использовании scriptPath и scriptLinkedService) |
| degreeOfParallelism | Максимальное количество узлов, используемых одновременно для выполнения задания. | Нет                                       |
| priority            | Определяет, какие задания из всех в очереди должны запускаться в первую очередь. Чем меньше число, тем выше приоритет. | Нет                                       |
| параметры          | Параметры скрипта U-SQL          | Нет                                       |
| runtimeVersion      | Версия среды выполнения обработчика U-SQL, которую нужно использовать. | Нет                                       |
| compilationMode     | <p>Режим компиляции U-SQL. Может иметь одно из следующих значений.</p> <ul><li>**Semantic**: выполнение только семантических проверок и необходимых проверок работоспособности.</li><li>**Full:** выполнение полной компиляции, включая проверку синтаксиса, оптимизацию, создание кода и т. д.</li><li>**SingleBox:** выполнение полной компиляции с параметром TargetType, заданным для SingleBox.</li></ul><p>Если не указать значение для этого свойства, сервер определит оптимальный режим компиляции. </p> | Нет                                       |

Определение сценария см. в разделе [Определение сценария SearchLogProcessing.txt](#sample-u-sql-script). 

## <a name="sample-input-and-output-datasets"></a>Примеры входных и выходных наборов данных
### <a name="input-dataset"></a>Входной набор данных
В этом примере входной набор данных находится в хранилище озера данных Azure (файл SearchLog.tsv в папке datalake/input). 

```json
{
    "name": "DataLakeTable",
    "properties": {
        "type": "AzureDataLakeStore",
        "linkedServiceName": "AzureDataLakeStoreLinkedService",
        "typeProperties": {
            "folderPath": "datalake/input/",
            "fileName": "SearchLog.tsv",
            "format": {
                "type": "TextFormat",
                "rowDelimiter": "\n",
                "columnDelimiter": "\t"
            }
        },
        "availability": {
            "frequency": "Day",
            "interval": 1
        }
    }
}    
```

### <a name="output-dataset"></a>Выходной набор данных
В этом примере выходные данные, порождаемые скриптом U-SQL, сохраняются в хранилище озера данных Azure (папка datalake/output). 

```json
{
    "name": "EventsByRegionTable",
    "properties": {
        "type": "AzureDataLakeStore",
        "linkedServiceName": "AzureDataLakeStoreLinkedService",
        "typeProperties": {
            "folderPath": "datalake/output/"
        },
        "availability": {
            "frequency": "Day",
            "interval": 1
        }
    }
}
```

### <a name="sample-data-lake-store-linked-service"></a>Пример связанной службы Data Lake Store
Вот определение примера связанной службы Azure Data Lake Store, используемой наборами входных и выходных данных. 

```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
        }
    }
}
```

Описания свойств JSON см. в статье [Перемещение данных в озеро данных Azure и обратно с помощью фабрики данных Azure](data-factory-azure-datalake-connector.md). 

## <a name="sample-u-sql-script"></a>Пример сценария U-SQL

```
@searchlog =
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string
    FROM @in
    USING Extractors.Tsv(nullEscape:"#NULL#");

@rs1 =
    SELECT Start, Region, Duration
    FROM @searchlog
WHERE Region == "en-gb";

@rs1 =
    SELECT Start, Region, Duration
    FROM @rs1
    WHERE Start <= DateTime.Parse("2012/02/19");

OUTPUT @rs1   
    TO @out
      USING Outputters.Tsv(quoting:false, dateTimeFormat:null);
```

Значения параметров **\@ in** и **\@ out** в скрипте U-SQL динамически передаются модулем ADF с помощью раздела Parameters. См. раздел "Parameters" в определении конвейера.

В определении конвейера для заданий, которые выполняются в службе Azure Data Lake Analytics, можно определить другие свойства, например degreeOfParallelism и priority.

## <a name="dynamic-parameters"></a>Динамические параметры
В примере определения конвейера параметрам in и out присвоено жестко заданные значения. 

```json
"parameters": {
    "in": "/datalake/input/SearchLog.tsv",
    "out": "/datalake/output/Result.tsv"
}
```

Вместо этого можно использовать динамические параметры. Пример: 

```json
"parameters": {
    "in": "$$Text.Format('/datalake/input/{0:yyyy-MM-dd HH:mm:ss}.tsv', SliceStart)",
    "out": "$$Text.Format('/datalake/output/{0:yyyy-MM-dd HH:mm:ss}.tsv', SliceStart)"
}
```

В этом случае входные файлы по-прежнему берутся из папки /datalake/input, а выходные файлы создаются в папке /datalake/output. Имена файлов присваиваются динамически, на основе времени начала среза.