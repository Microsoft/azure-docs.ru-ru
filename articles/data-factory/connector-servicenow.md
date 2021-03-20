---
title: Копирование данных из ServiceNow
description: Узнайте, как копировать данные из ServiceNow в поддерживаемые хранилища данных в качестве приемников с помощью действия копирования в конвейере фабрики данных Azure.
ms.author: jingwang
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 08/01/2019
ms.openlocfilehash: 4e7ebc422a9fd8503c5a3b004e1d06cb5ebfb987
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100378457"
---
# <a name="copy-data-from-servicenow-using-azure-data-factory"></a>Копирование данных из ServiceNow с помощью фабрики данных Azure
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описывается, как с помощью действия копирования в фабрике данных Azure копировать данные из ServiceNow. Это продолжение [статьи об обзоре действия копирования](copy-activity-overview.md), в которой представлены общие сведения о действии копирования.

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Этот соединитель ServiceNow поддерживается для следующих действий:

- [Действие копирования](copy-activity-overview.md) с использованием [матрицы поддерживаемых источников и приемников](copy-activity-overview.md)
- [Действие поиска](control-flow-lookup-activity.md)

Данные из ServiceNow можно скопировать в любое поддерживаемое хранилище данных, используемое в качестве приемника. Список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования, приведен в таблице [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats).

Фабрика данных Azure имеет встроенный драйвер для настройки подключения. Поэтому с использованием этого соединителя вам не нужно устанавливать драйверы вручную.

## <a name="getting-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Следующие разделы содержат сведения о свойствах, которые используются для определения сущностей фабрики данных, относящихся к соединителю ServiceNow.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы ServiceNow поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства type нужно задать значение **ServiceNow**. | Да |
| endpoint | Конечная точка сервера ServiceNow (`http://<instance>.service-now.com`).  | Да |
| authenticationType | Тип проверки подлинности. <br/>Допустимые значения: **Basic**, **OAuth2**. | Да |
| username | Имя пользователя, используемое для подключения к серверу ServiceNow для обычной проверки подлинности и OAuth2.  | Да |
| password | Пароль, соответствующий имени пользователя для обычной проверки подлинности и OAuth2. Пометьте это поле как SecureString, чтобы безопасно хранить его в фабрике данных, или [добавьте ссылку на секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md). | Да |
| clientid | Идентификатор клиента для проверки подлинности OAuth2.  | Нет |
| clientSecret | Секрет клиента для проверки подлинности OAuth2. Пометьте это поле как SecureString, чтобы безопасно хранить его в фабрике данных, или [добавьте ссылку на секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md). | нет |
| useEncryptedEndpoints | Указывает, шифруются ли конечные точки источника данных с помощью протокола HTTPS. Значение по умолчанию — true.  | Нет |
| useHostVerification | Указывает, должно ли имя узла в сертификате сервера совпадать с именем узла сервера при подключении по протоколу TLS. Значение по умолчанию — true.  | Нет |
| usePeerVerification | Указывает, следует ли проверять удостоверение сервера при подключении по протоколу TLS. Значение по умолчанию — true.  | Нет |

**Пример**.

```json
{
    "name": "ServiceNowLinkedService",
    "properties": {
        "type": "ServiceNow",
        "typeProperties": {
            "endpoint" : "http://<instance>.service-now.com",
            "authenticationType" : "Basic",
            "username" : "<username>",
            "password": {
                 "type": "SecureString",
                 "value": "<password>"
            }
        }
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных

Полный список разделов и свойств, доступных для определения наборов данных, см. в статье о [наборах данных](concepts-datasets-linked-services.md). Этот раздел содержит список свойств, поддерживаемых набором данных ServiceNow.

Чтобы скопировать данные из ServiceNow, установите свойство type набора данных **ServiceNowObject**. Поддерживаются следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type набора данных должно иметь значение **сервиценовобжект** . | Да |
| tableName | Имя таблицы. | Нет (если свойство query указано в источнике действия) |

**Пример**

```json
{
    "name": "ServiceNowDataset",
    "properties": {
        "type": "ServiceNowObject",
        "typeProperties": {},
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<ServiceNow linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). Этот раздел содержит список свойств, поддерживаемых источником ServiceNow.

### <a name="servicenow-as-source"></a>ServiceNow в качестве источника

Чтобы копировать данные из ServiceNow, установите тип источника **ServiceNowSource** в действии копирования. В разделе **source** действия копирования поддерживаются следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство type источника действия копирования должно иметь значение **ServiceNowSource**. | Да |
| query | Используйте пользовательский SQL-запрос для чтения данных. Например: `"SELECT * FROM Actual.alm_asset"`. | Нет (если для набора данных задано свойство tableName) |

Указывая в запросе схему и столбец для ServiceNow, обратите внимание на следующую информацию. Также просмотрите **советы по [улучшению производительности](#performance-tips) копирования**.

- **Схема.** В запросе к ServiceNow укажите схему как `Actual` или `Display`, что можно рассматривать как параметр `sysparm_display_value` со значением true или false при вызове [REST API ServiceNow](https://developer.servicenow.com/app.do#!/rest_api_doc?v=jakarta&id=r_AggregateAPI-GET). 
- **Столбец.** Имя столбца для фактического значения в схеме `Actual` — это `[column name]_value`, а отображаемое имя в `Display` — это `[column name]_display_value`. Обратите внимание, что имя столбца должно соответствовать имени в схеме, используемой в запросе.

**Образец запроса:** 
 `SELECT col_value FROM Actual.alm_asset` НИ`SELECT col_display_value FROM Display.alm_asset`

**Пример**.

```json
"activities":[
    {
        "name": "CopyFromServiceNow",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<ServiceNow input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "ServiceNowSource",
                "query": "SELECT * FROM Actual.alm_asset"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```
## <a name="performance-tips"></a>Советы по улучшению производительности

### <a name="schema-to-use"></a>Используемые схемы

В ServiceNow есть две разные схемы: **Actual** (возвращает фактические данные) и **Display** (возвращает отображаемые значения данных). 

При наличии фильтра в запросе используйте схему Actual, так как она обеспечивает лучшую производительность копирования. Если используется схема Actual, ServiceNow нативно поддерживает фильтр при выборке данных и возвращает только отфильтрованный набор результатов. Если же используется схема Display, ADF извлекает все данные, а затем фильтрует их.

### <a name="index"></a>Индекс

Индекс таблицы ServiceNow может помочь повысить производительность запросов. Подробности см. [здесь](https://docs.servicenow.com/bundle/geneva-servicenow-platform/page/administer/table_administration/task/t_CreateCustomIndex.html).

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Подробные сведения об этих свойствах см. в разделе [Действие поиска](control-flow-lookup-activity.md).


## <a name="next-steps"></a>Дальнейшие действия
В таблице [Поддерживаемые хранилища данных](copy-activity-overview.md#supported-data-stores-and-formats) приведен список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования в фабрике данных Azure.
