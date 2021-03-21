---
title: Копирование данных из SAP ECC
description: Узнайте, как копировать данные из SAP ECC в поддерживаемые хранилища данных, используемые в качестве приемника, с помощью действия копирования в конвейере фабрики данных Azure.
ms.author: jingwang
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 10/28/2020
ms.openlocfilehash: a3e701f3d433b5b52d8992035ac4ad75b78cb795
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100386702"
---
# <a name="copy-data-from-sap-ecc-by-using-azure-data-factory"></a>Копирование данных из SAP ECC с помощью Фабрики данных Azure
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описывается, как с помощью действия копирования в Фабрике данных Azure копировать данные из SAP Enterprise Central Component (SAP ECC). Дополнительные сведения см. в статье [Общие сведения о действии копирования](copy-activity-overview.md).

>[!TIP]
>Сведения о общей поддержке ADF в сценарии интеграции данных SAP см. в статье [Интеграция данных SAP с помощью фабрики данных Azure](https://github.com/Azure/Azure-DataFactory/blob/master/whitepaper/SAP%20Data%20Integration%20using%20Azure%20Data%20Factory.pdf) с подробным введением в каждый соединитель SAP, компарсион и рекомендации.

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Этот соединитель SAP ECC поддерживается для следующих действий.

- [Действие копирования](copy-activity-overview.md) с использованием [матрицы поддерживаемых источников и приемников](copy-activity-overview.md)
- [Действие поиска](control-flow-lookup-activity.md)

Данные из SAP ECC можно скопировать в любое поддерживаемое хранилище данных, используемое в качестве приемника. Список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования, приведен в таблице [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats).

В частности, этот соединитель SAP ECC поддерживает:

- копирование данных из SAP ECC в SAP NetWeaver 7.0 и более поздних версий;
- копирование данных из любых объектов, предоставляемых службами SAP ECC OData, например:

  - таблицы или представления SAP;
  - объекты программного интерфейса бизнес-приложений [BAPI];
  - средства извлечения данных;
  - данные или промежуточные документы (IDOC), отправляемые в службу SAP Process Integration (PI), которые могут быть получены в виде OData через соответствующие адаптеры;

- копирование данных с помощью базовой проверки подлинности.

Версия 7,0 или более поздняя относится к версии SAP NetWeaver вместо SAP ECC. Например, SAP ECC 6,0 EHP 7 в целом имеет версию NetWeaver >= 7,4. Если вы не уверены в своей среде, выполните следующие действия для подтверждения версии из системы SAP:

1. Используйте графический интерфейс SAP для подключения к системе SAP. 
2. Перейдите к   ->  **состоянию** системы. 
3. Проверьте выпуск SAP_BASIS, убедитесь, что он больше или равен 701.  
      ![Проверка SAP_BASIS](./media/connector-sap-table/sap-basis.png)

>[!TIP]
>Чтобы скопировать данные из SAP ECC посредством таблицы или представления SAP, используйте соединитель [Таблица SAP](connector-sap-table.md), который быстрее и обеспечивает масштабирование.

## <a name="prerequisites"></a>Предварительные требования

Чтобы использовать этот соединитель SAP ECC, необходимо предоставить сущности SAP ECC через службы OData через шлюз SAP. В частности:

- **Настройте шлюз SAP**. Для серверов с SAP NetWeaver версии выше 7.4 шлюз SAP уже установлен. Для более ранних версий необходимо установить встроенный шлюз SAP или систему концентратора шлюза SAP, прежде чем предоставлять данные SAP ECC через службы OData. Сведения о том, как настроить шлюз SAP, см. в [руководстве по установке](https://help.sap.com/saphelp_gateway20sp12/helpdata/en/c3/424a2657aa4cf58df949578a56ba80/frameset.htm).

- **Активируйте и настройте службу SAP OData**. Активировать службы OData можно через TCODE SICF за считаные секунды. Также можно указать, какие объекты требуется предоставлять. Дополнительные сведения см. в [пошаговом руководстве](https://blogs.sap.com/2012/10/26/step-by-step-guide-to-build-an-odata-service-based-on-rfcs-part-1/).

[!INCLUDE [data-factory-v2-integration-runtime-requirements](../../includes/data-factory-v2-integration-runtime-requirements.md)]

## <a name="get-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

В следующих разделах содержатся сведения о свойствах, которые используются для определения сущностей фабрики данных, относящихся к соединителю SAP ECC.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы SAP ECC поддерживаются следующие свойства.

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| `type` | Для свойства `type` должно быть установлено значение `SapEcc`. | Да |
| `url` | URL-адрес службы SAP ECC OData. | Да |
| `username` | Имя пользователя, применяемое для подключения к SAP ECC. | Нет |
| `password` | Пароль (открытым текстом), применяемый для подключения к SAP ECC. | Нет |
| `connectVia` | [Среда выполнения интеграции](concepts-integration-runtime.md), используемая для подключения к хранилищу данных. Дополнительные сведения см. в разделе [Предварительные требования](#prerequisites). Если не указать среду выполнения, используется среда выполнения интеграции Azure по умолчанию. | Нет |

### <a name="example"></a>Пример

```json
{
    "name": "SapECCLinkedService",
    "properties": {
        "type": "SapEcc",
        "typeProperties": {
            "url": "<SAP ECC OData URL, e.g., http://eccsvrname:8000/sap/opu/odata/sap/zgw100_dd02l_so_srv/>",
            "username": "<username>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        }
    },
    "connectVia": {
        "referenceName": "<name of integration runtime>",
        "type": "IntegrationRuntimeReference"
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных

Полный список разделов и свойств, доступных для определения наборов данных, см. в разделе [Наборы данных в Фабрике данных Azure](concepts-datasets-linked-services.md). Следующий раздел содержит список свойств, поддерживаемых для набора данных SAP ECC.

Чтобы скопировать данные из SAP ECC, задайте для свойства `type` набора данных значение `SapEccResource`.

Поддерживаются следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| `path` | Путь к сущности SAP ECC OData. | Да |

### <a name="example"></a>Пример

```json
{
    "name": "SapEccDataset",
    "properties": {
        "type": "SapEccResource",
        "typeProperties": {
            "path": "<entity path, e.g., dd04tentitySet>"
        },
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<SAP ECC linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, доступных для определения действий, см. в разделе [Конвейеры и действия в Фабрике данных Azure](concepts-pipelines-activities.md). Следующий раздел содержит список свойств, поддерживаемых для источника SAP ECC.

### <a name="sap-ecc-as-a-source"></a>SAP ECC в качестве источника

Чтобы скопировать данные из SAP ECC, задайте для свойства `type` в разделе `source` действия копирования значение `SapEccSource`.

В разделе `source` для действия копирования поддерживаются следующие свойства.

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| `type` | Свойство `type` в разделе `source` действия копирования должно иметь значение `SapEccSource`. | Да |
| `query` | Параметры запроса OData для фильтрации данных. Пример:<br/><br/>`"$select=Name,Description&$top=10"`<br/><br/>Соединитель SAP ECC копирует данные из объединенного URL-адреса:<br/><br/>`<URL specified in the linked service>/<path specified in the dataset>?<query specified in the copy activity's source section>`<br/><br/>Дополнительные сведения см. в статье о [компонентах URL-адреса OData](https://www.odata.org/documentation/odata-version-3-0/url-conventions/). | Нет |
| `sapDataColumnDelimiter` | Единственный символ, используемый в качестве разделителя, передаваемый в SAP RFC для разделения выходных данных. | Нет |
| `httpRequestTimeout` | Время ожидания (значение **Временной диапазон**) ответа для HTTP-запроса. Это значение является интервалом времени для получения ответа, а не считывания данных ответа. Если не указано, значение по умолчанию — **00:30:00** (30 минут). | Нет |

### <a name="example"></a>Пример

```json
"activities":[
    {
        "name": "CopyFromSAPECC",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SAP ECC input dataset name>",
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
                "type": "SapEccSource",
                "query": "$top=10"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="data-type-mappings-for-sap-ecc"></a>Сопоставление типов данных для SAP ECC

При копировании данных из SAP ECC используются следующие сопоставления типов данных OData для данных SAP ECC с промежуточными типами данных фабрики данных Azure. Дополнительные сведения о том, как действие копирования сопоставляет исходную схему и типы данных для приемника, см. в статье [Сопоставление схем в действии копирования](copy-activity-schema-and-type-mapping.md).

| Тип данных OData | Тип промежуточных данных фабрики данных |
|:--- |:--- |
| `Edm.Binary` | `String` |
| `Edm.Boolean` | `Bool` |
| `Edm.Byte` | `String` |
| `Edm.DateTime` | `DateTime` |
| `Edm.Decimal` | `Decimal` |
| `Edm.Double` | `Double` |
| `Edm.Single` | `Single` |
| `Edm.Guid` | `String` |
| `Edm.Int16` | `Int16` |
| `Edm.Int32` | `Int32` |
| `Edm.Int64` | `Int64` |
| `Edm.SByte` | `Int16` |
| `Edm.String` | `String` |
| `Edm.Time` | `TimeSpan` |
| `Edm.DateTimeOffset` | `DateTimeOffset` |

> [!NOTE]
> Сложные типы данных в настоящее время не поддерживаются.

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Подробные сведения об этих свойствах см. в разделе [Действие поиска](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Дальнейшие действия

Список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования в Фабрике данных Azure, см. в [этой таблице](copy-activity-overview.md#supported-data-stores-and-formats).
