---
title: Перемещение данных из SalesForce с помощью фабрики данных
description: Узнайте, как перемещать данные из Salesforce с использованием фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 07/18/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: ccc20f415d13356de755af5d1d3afc5b29de72f2
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100387059"
---
# <a name="move-data-from-salesforce-by-using-azure-data-factory"></a>Перемещение данных из Salesforce с помощью фабрики данных Azure
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](data-factory-salesforce-connector.md)
> * [Версия 2 (текущая)](../connector-salesforce.md)

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. статью о [соединителе Salesforce в службе "Фабрика данных Azure" версии 2](../connector-salesforce.md).

В этой статье описано, как переместить данные из Salesforce в любое хранилище данных, указанное в таблице [поддерживаемых хранилищ данных](data-factory-data-movement-activities.md#supported-data-stores-and-formats) (столбец "Приемник"), с использованием действия копирования в фабрике данных Azure. Эта статья основана на статье о [действиях перемещения данных](data-factory-data-movement-activities.md) , которая представляет собой общий обзор перемещения данных с помощью действий копирования и поддерживаемых сочетаний хранилищ данных.

Сейчас фабрика данных Azure поддерживает только перемещение данных из Salesforce в другие [поддерживаемые хранилища-приемники](data-factory-data-movement-activities.md#supported-data-stores-and-formats), но не наоборот.

## <a name="supported-versions"></a>Поддерживаемые версии
Этот соединитель поддерживает следующие выпуски Salesforce: Developer Edition, Professional Edition, Enterprise Edition или Unlimited Edition. Он также поддерживает копирование из рабочей среды Salesforce, песочницы и пользовательского домена.

## <a name="prerequisites"></a>Предварительные условия
* Требуется включить разрешения API. Дополнительные сведения о включении доступа к API в Salesforce с помощью набора разрешений см. [здесь](https://www.data2crm.com/migration/faqs/enable-api-access-salesforce-permission-set/).
* Чтобы скопировать данные из Salesforce в локальное хранилище данных, в локальной среде необходимо установить шлюз управления данными версии не ниже 2.0.

## <a name="salesforce-request-limits"></a>Ограничения запросов Salesforce
Для Salesforce установлены ограничения на общее число запросов API и одновременных запросов API. Обратите внимание на следующие моменты.

- Если количество одновременных запросов превышает ограничение, выполняется регулирование и возникают случайные ошибки.
- В случае превышения ограничения на общее число запросов учетная запись Salesforce блокируется на 24 часа.

В обоих сценариях может также появиться ошибка "REQUEST_LIMIT_EXCEEDED". Дополнительные сведения см. в разделе "API Request Limits" (Ограничения запросов API) статьи об [ограничениях для разработчика Salesforce](https://resources.docs.salesforce.com/200/20/en-us/sfdc/pdf/salesforce_app_limits_cheatsheet.pdf).

## <a name="getting-started"></a>Начало работы
Вы можете создать конвейер с действием копирования, который перемещает данные из Salesforce, с помощью разных инструментов и интерфейсов API.

Самый простой способ создать конвейер — использовать **Мастер копирования**. В статье [Руководство. Создание конвейера с действием копирования с помощью мастера копирования фабрики данных](data-factory-copy-data-wizard-tutorial.md) приведены краткие пошаговые указания по созданию конвейера с помощью мастера копирования данных.

Для создания конвейера можно также использовать следующие средства: **Visual Studio**, **Azure PowerShell**, **Azure Resource Manager шаблон**, **API .NET** и **REST API**. Пошаговые инструкции по созданию конвейера с действием копирования см. в разделе [учебник по действиям копирования](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) .

Независимо от используемого средства или API-интерфейса, для создания конвейера, который перемещает данные из источника данных в приемник, выполняются следующие шаги:

1. Создайте **связанные службы** , чтобы связать входные и выходные данные хранилища с фабрикой данных.
2. Создайте **наборы данных**, которые представляют входные и выходные данные для операции копирования.
3. Создайте **конвейер** с действием копирования, который принимает входной набор данных и возвращает выходной набор данных.

Если вы используете мастер, то он автоматически создает определения JSON для сущностей фабрики данных (связанных служб, наборов данных и конвейера). При использовании инструментов и интерфейсов API (за исключением API .NET) вы самостоятельно определяете эти сущности фабрики данных в формате JSON. Пример определений JSON для сущностей фабрики данных, которые используются для копирования данных из Salesforce, доступен в разделе [Пример JSON. Копирование данных из Salesforce в большой двоичный объект Azure](#json-example-copy-data-from-salesforce-to-azure-blob) этой статьи.

Следующие разделы содержат сведения о свойствах JSON, которые используются для определения сущностей фабрики данных, относящихся к Salesforce.

## <a name="linked-service-properties"></a>Свойства связанной службы
В таблице ниже приведены описания элементов JSON, которые относятся к связанной службе Salesforce.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| type |Свойство Type должно иметь значение: **Salesforce**. |Да |
| environmentUrl | Укажите URL-адрес экземпляра Salesforce. <br><br> -По умолчанию — "https: \/ /Login.Salesforce.com". <br> Чтобы скопировать данные из песочницы, укажите https://test.salesforce.com. <br> — Чтобы скопировать данные из пользовательского домена, укажите URL-адрес, например https://[домен].my.salesforce.com. |Нет |
| username |Укажите имя пользователя для учетной записи пользователя. |Да |
| password |Укажите пароль для учетной записи пользователя. |Да |
| securityToken |Укажите маркер безопасности для учетной записи пользователя. Инструкции по получению и сбросу маркера безопасности см. в статье [Get security token](https://help.salesforce.com/apex/HTViewHelpDoc?id=user_security_token.htm) (Получение маркера безопасности). Общие сведения о маркере безопасности см. в статье [Security and the API](https://developer.salesforce.com/docs/atlas.en-us.api.meta/api/sforce_api_concepts_security.htm) (Безопасность и API). |Да |

## <a name="dataset-properties"></a>Свойства набора данных
Полный список разделов и свойств, используемых для определения наборов данных, см. в статье [Наборы данных в фабрике данных Azure](data-factory-create-datasets.md). Разделы structure, availability и policy JSON набора данных одинаковы для всех типов наборов данных (SQL Azure, большие двоичные объекты Azure, таблицы Azure и т. д.).

Раздел **typeProperties** отличается для каждого типа набора данных и предоставляет сведения о расположении данных в хранилище данных. Раздел typeProperties для набора данных типа **RelationalTable** содержит следующие свойства.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| tableName |Имя таблицы в Salesforce |Нет (если для свойства **RelationalSource** задано значение **query**). |

> [!IMPORTANT]
> Имя API для любых настраиваемых объектов должно содержать приставку __c.

![На снимке экрана показаны подробные сведения об определении пользовательского объекта, где можно увидеть имена пользовательских объектов в виде P I.](media/data-factory-salesforce-connector/data-factory-salesforce-api-name.png)

## <a name="copy-activity-properties"></a>Свойства действия копирования
Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure: создание конвейеров, цепочки действий и расписаний для них](data-factory-create-pipelines.md). Такие свойства, как имя, описание, входные и выходные таблицы, различные политики, доступны для всех типов действий.

С другой стороны, свойства, доступные в разделе typeProperties действия, зависят от конкретного типа действия. Для действия копирования они различаются в зависимости от типов источников и приемников.

В случае действия копирования, если источник относится к типу **RelationalSource** (который содержит Salesforce), в разделе typeProperties доступны следующие свойства.

| Свойство. | Описание | Допустимые значения | Обязательно |
| --- | --- | --- | --- |
| query |Используйте пользовательский запрос для чтения данных. |Запрос SQL-92 или запрос, написанный на [объектно-ориентированном языке запросов Salesforce (SOQL)](https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta/soql_sosl/sforce_api_calls_soql.htm) . Например, `select * from MyTable__c`. |Нет (если для свойства **tableName** задано значение **dataset**). |

> [!IMPORTANT]
> Имя API для любых настраиваемых объектов должно содержать приставку __c.

![На снимке экрана показаны настраиваемые поля & связей, в которых можно видеть имена пользовательских объектов в виде P.](media/data-factory-salesforce-connector/data-factory-salesforce-api-name-2.png)

## <a name="query-tips"></a>Советы по запросам
### <a name="retrieving-data-using-where-clause-on-datetime-column"></a>Получение данных с использованием предложения WHERE в столбце даты и времени
При указании запроса SOQL или SQL обратите внимание на различие в формате даты и времени. Пример:

* **Пример SOQL**: `$$Text.Format('SELECT Id, Name, BillingCity FROM Account WHERE LastModifiedDate >= {0:yyyy-MM-ddTHH:mm:ssZ} AND LastModifiedDate < {1:yyyy-MM-ddTHH:mm:ssZ}', WindowStart, WindowEnd)`
* **Пример SQL**:
    * **Использование мастера копирования для указания запроса:** `$$Text.Format('SELECT * FROM Account WHERE LastModifiedDate >= {{ts\'{0:yyyy-MM-dd HH:mm:ss}\'}} AND LastModifiedDate < {{ts\'{1:yyyy-MM-dd HH:mm:ss}\'}}', WindowStart, WindowEnd)`
    * **Использование редактирования JSON для указания запроса (соответствующие escape-символы):** `$$Text.Format('SELECT * FROM Account WHERE LastModifiedDate >= {{ts\\'{0:yyyy-MM-dd HH:mm:ss}\\'}} AND LastModifiedDate < {{ts\\'{1:yyyy-MM-dd HH:mm:ss}\\'}}', WindowStart, WindowEnd)`

### <a name="retrieving-data-from-salesforce-report"></a>Извлечение данных из отчета Salesforce
Из отчетов Salesforce можно извлекать данные, указывая запросы в формате `{call "<report name>"}`, например: `"query": "{call \"TestReport\"}"`.

### <a name="retrieving-deleted-records-from-salesforce-recycle-bin"></a>Восстановление удаленных записей из корзины Salesforce
Чтобы запросить из корзины Salesforce обратимо удаленные записи, укажите в своем запросе **"IsDeleted = 1"**. Например,

* Чтобы запросить только удаленные записи, укажите "SELECT * FROM MyTable__c, **где IsDeleted = 1**"
* Чтобы запросить все записи, включая существующие и удаленные, укажите "SELECT * FROM MyTable__c, **где IsDeleted = 0 или IsDeleted = 1**"

## <a name="json-example-copy-data-from-salesforce-to-azure-blob"></a>Пример JSON. Копирование данных из Salesforce в большой двоичный объект Azure
В следующем примере приводятся образцы определений JSON, которые можно использовать для создания конвейера с помощью [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) или [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Вы узнаете, как копировать данные из Salesforce в хранилище BLOB-объектов Azure. Тем не менее данные можно копировать в любой из указанных [здесь](data-factory-data-movement-activities.md#supported-data-stores-and-formats) приемников. Это делается с помощью действия копирования в фабрике данных Azure.

Ниже приведены артефакты фабрики данных, которые необходимо создать для реализации сценария. В разделах после списка приведены подробные сведения об этих действиях.

* Связанная служба типа [Salesforce](#linked-service-properties)
* Связанная служба типа [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)
* Входной [набор данных](data-factory-create-datasets.md) типа [RelationalTable](#dataset-properties).
* Выходной [набор данных](data-factory-create-datasets.md) типа [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
* [Конвейер](data-factory-create-pipelines.md) с действием копирования, в котором используются [RelationalSource](#copy-activity-properties) и [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties)

**Связанная служба SalesForce**

В этом примере используется связанная служба **Salesforce**. Список свойств, поддерживаемых этой связанной службой, приведен в разделе [Свойства связанной службы Salesforce](#linked-service-properties). Инструкции по получению и сбросу маркера безопасности см. в статье [Get security token](https://help.salesforce.com/apex/HTViewHelpDoc?id=user_security_token.htm) (Получение маркера безопасности).

```json
{
    "name": "SalesforceLinkedService",
    "properties":
    {
        "type": "Salesforce",
        "typeProperties":
        {
            "username": "<user name>",
            "password": "<password>",
            "securityToken": "<security token>"
        }
    }
}
```
**Связанная служба хранения Azure**

```json
{
    "name": "AzureStorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
    }
}
```
**Входной набор данных Salesforce**

```json
{
    "name": "SalesforceInput",
    "properties": {
        "linkedServiceName": "SalesforceLinkedService",
        "type": "RelationalTable",
        "typeProperties": {
            "tableName": "AllDataType__c"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

Если для параметра **external** задать значение **true**, то фабрика данных воспримет этот набор данных как внешний, который создан не в результате какого-либо действия в фабрике данных.

> [!IMPORTANT]
> Имя API для любых настраиваемых объектов должно содержать приставку __c.

![На снимке экрана показаны сведения о пользовательском определении объекта, где можно увидеть метку единственного числа, множественное число, имя объекта и имя P I.](media/data-factory-salesforce-connector/data-factory-salesforce-api-name.png)

**Выходной набор данных большого двоичного объекта Azure**

Данные записываются в новый BLOB-объект каждый час (frequency: hour, interval: 1).

```json
{
    "name": "AzureBlobOutput",
    "properties":
    {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties":
        {
            "folderPath": "adfgetstarted/alltypes_c"
        },
        "availability":
        {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

**Конвейер с действием копирования**

Конвейер содержит действие копирования, которое использует входной и выходной наборы данных и выполняется каждый час. В определении JSON конвейера для типа **source** установлено значение **RelationalSource**, а для типа **sink** — значение **BlobSink**.

Список свойств, поддерживаемых RelationalSource, см. в разделе [Свойства типа RelationalSource](#copy-activity-properties).

```json
{
    "name":"SamplePipeline",
    "properties":{
        "start":"2016-06-01T18:00:00",
        "end":"2016-06-01T19:00:00",
        "description":"pipeline with copy activity",
        "activities":[
        {
            "name": "SalesforceToAzureBlob",
            "description": "Copy from Salesforce to an Azure blob",
            "type": "Copy",
            "inputs": [
            {
                "name": "SalesforceInput"
            }
            ],
            "outputs": [
            {
                "name": "AzureBlobOutput"
            }
            ],
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "SELECT Id, Col_AutoNumber__c, Col_Checkbox__c, Col_Currency__c, Col_Date__c, Col_DateTime__c, Col_Email__c, Col_Number__c, Col_Percent__c, Col_Phone__c, Col_Picklist__c, Col_Picklist_MultiSelect__c, Col_Text__c, Col_Text_Area__c, Col_Text_AreaLong__c, Col_Text_AreaRich__c, Col_URL__c, Col_Text_Encrypt__c, Col_Lookup__c FROM AllDataType__c"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }
        ]
    }
}
```
> [!IMPORTANT]
> Имя API для любых настраиваемых объектов должно содержать приставку __c.

![На снимке экрана показаны настраиваемые поля, & связи с вызываемыми именами A P.](media/data-factory-salesforce-connector/data-factory-salesforce-api-name-2.png)


### <a name="type-mapping-for-salesforce"></a>Сопоставление типов для Salesforce

| Тип данных Salesforce | Тип на основе .NET |
| --- | --- |
| Автонумерация |Строка |
| Флажок |Логическое |
| Валюта |Decimal |
| Дата |Дата и время |
| Дата и время |Дата/время |
| Email |Строка |
| Идентификатор |Строка |
| Связь для подстановки |Строка |
| Список множественного выбора |Строка |
| Число |Decimal |
| Процент |Decimal |
| Телефон |Строка |
| Список выбора |Строка |
| Текстовый |Строковый |
| Текстовое поле |Строка |
| Текстовое поле (длинное) |Строка |
| Текстовое поле (расширенное) |Строка |
| Текст (зашифрованный) |Строка |
| URL-адрес |Строка |

> [!NOTE]
> Сведения о сопоставлении столбцов в наборе данных, используемом в качестве источника, со столбцами в приемнике см. в [этой статье](data-factory-map-columns.md).

[!INCLUDE [data-factory-structure-for-rectangualr-datasets](../../../includes/data-factory-structure-for-rectangualr-datasets.md)]

## <a name="performance-and-tuning"></a>Производительность и настройка
Ознакомьтесь с [руководством по настройке производительности действия копирования](data-factory-copy-activity-performance.md), в котором описываются ключевые факторы, влияющие на производительность перемещения данных (действие копирования) в фабрике данных Azure, и различные способы оптимизации этого процесса.
