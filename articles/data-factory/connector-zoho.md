---
title: Копирование данных из Zoho с помощью фабрики данных Azure (предварительная версия)
description: Узнайте, как копировать данные из Zoho в поддерживаемые хранилища данных-приемники с помощью действия копирования в конвейере фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 08/03/2020
ms.author: jingwang
ms.openlocfilehash: e42638d484d2a71052c3a9410f73cbca9e038682
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100366897"
---
# <a name="copy-data-from-zoho-using-azure-data-factory-preview"></a>Копирование данных из Zoho с помощью фабрики данных Azure (предварительная версия)
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описывается, как с помощью действия копирования в фабрике данных Azure копировать данные из Zoho. Это продолжение [статьи об обзоре действия копирования](copy-activity-overview.md), в которой представлены общие сведения о действии копирования.

> [!IMPORTANT]
> Сейчас этот соединитель доступен в режиме предварительной версии. Попробуйте поработать с ним и оставьте свой отзыв. Если вы хотите использовать в своем решении зависимость от соединителей в предварительной версии, обратитесь в службу [поддержки Azure](https://azure.microsoft.com/support/).

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Этот соединитель Zoho поддерживается для следующих действий:

- [Действие копирования](copy-activity-overview.md) с использованием [матрицы поддерживаемых источников и приемников](copy-activity-overview.md)
- [Действие поиска](control-flow-lookup-activity.md)


Данные из Zoho можно скопировать в любое поддерживаемое хранилище данных, используемое в качестве приемника. Список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования, приведен в таблице [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats).

Этот соединитель поддерживает аутентификацию Xero доступа по маркеру и OAuth 2,0.

Фабрика данных Azure имеет встроенный драйвер для настройки подключения. Поэтому с использованием этого соединителя вам не нужно устанавливать драйверы вручную.

## <a name="getting-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Следующие разделы содержат сведения о свойствах, используемых для определения сущностей фабрики данных, относящихся к соединителю Zoho.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы Zoho поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства type необходимо задать значение **Zoho** | Да |
| connectionProperties | Группа свойств, определяющих способ подключения к Zoho. | Да |
| ***В разделе `connectionProperties` :*** | | |
| endpoint | Конечная точка сервера Zoho (`crm.zoho.com/crm/private`). | Да |
| authenticationType | Допустимые значения: `OAuth_2.0` и `Access Token` . | Да |
| clientid | Идентификатор клиента, связанный с приложением Zoho. | Да для проверки подлинности OAuth 2,0 | 
| клиентсекрект | Clientsecret, связанный с приложением Zoho. Пометьте это поле как SecureString, чтобы безопасно хранить его в фабрике данных, или [добавьте ссылку на секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md). | Да для проверки подлинности OAuth 2,0 | 
| refreshtoken | Маркер обновления OAuth 2,0, связанный с приложением Zoho, который используется для обновления маркера доступа по истечении срока его действия. Срок действия маркера обновления никогда не истечет. Чтобы получить маркер обновления, необходимо запросить `offline` access_type. Дополнительные сведения см. в [этой статье](https://www.zoho.com/crm/developer/docs/api/auth-request.html). <br>Пометьте это поле как SecureString, чтобы безопасно хранить его в фабрике данных, или [добавьте ссылку на секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md).| Да для проверки подлинности OAuth 2,0 |
| accessToken | Маркер доступа для аутентификации Zoho. Пометьте это поле как SecureString, чтобы безопасно хранить его в фабрике данных, или [добавьте ссылку на секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md). | Да |
| useEncryptedEndpoints | Указывает, шифруются ли конечные точки источника данных с помощью протокола HTTPS. Значение по умолчанию — true.  | Нет |
| useHostVerification | Указывает, должно ли имя узла в сертификате сервера совпадать с именем узла сервера при подключении по протоколу TLS. Значение по умолчанию — true.  | Нет |
| usePeerVerification | Указывает, следует ли проверять удостоверение сервера при подключении по протоколу TLS. Значение по умолчанию — true.  | Нет |

**Пример: проверка подлинности OAuth 2,0**

```json
{
    "name": "ZohoLinkedService",
    "properties": {
        "type": "Zoho",
        "typeProperties": {
            "connectionProperties": { 
                "authenticationType":"OAuth_2.0", 
                "endpoint": "crm.zoho.com/crm/private", 
                "clientId": "<client ID>", 
                "clientSecrect": {
                    "type": "SecureString",
                    "value": "<client secret>"
                },
                "accessToken": {
                    "type": "SecureString",
                    "value": "<access token>"
                }, 
                "refreshToken": {
                    "type": "SecureString",
                    "value": "<refresh token>"
                }, 
                "useEncryptedEndpoints": true,
                "useHostVerification": true, 
                "usePeerVerification": true
            }
        }
    }
}
```

**Пример: проверка подлинности маркера доступа**

```json
{
    "name": "ZohoLinkedService",
    "properties": {
        "type": "Zoho",
        "typeProperties": {
            "connectionProperties": { 
                "authenticationType":"Access Token", 
                "endpoint": "crm.zoho.com/crm/private", 
                "accessToken": {
                    "type": "SecureString",
                    "value": "<access token>"
                }, 
                "useEncryptedEndpoints": true, 
                "useHostVerification": true, 
                "usePeerVerification": true
            }
        }
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных

Полный список разделов и свойств, доступных для определения наборов данных, см. в статье о [наборах данных](concepts-datasets-linked-services.md). В этом разделе содержится список свойств, поддерживаемых набором данных Zoho.

Чтобы скопировать данные с Zoho, установите свойство type набора данных **ZohoObject**. Поддерживаются следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type набора данных должно иметь значение **зохубжект** . | Да |
| tableName | Имя таблицы. | Нет (если свойство query указано в источнике действия) |

**Пример**

```json
{
    "name": "ZohoDataset",
    "properties": {
        "type": "ZohoObject",
        "typeProperties": {},
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<Zoho linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). В этом разделе содержится список свойств, поддерживаемых источником Zoho.

### <a name="zoho-as-source"></a>Zoho в качестве источника

Чтобы копировать данные из Zoho, установите тип источника **ZohoSource** в действии копирования. В разделе **source** действия копирования поддерживаются следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство type источника действия копирования должно иметь значение **ZohoSource**. | Да |
| query | Используйте пользовательский SQL-запрос для чтения данных. Например: `"SELECT * FROM Accounts"`. | Нет (если для набора данных задано свойство tableName) |

**Пример**.

```json
"activities":[
    {
        "name": "CopyFromZoho",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Zoho input dataset name>",
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
                "type": "ZohoSource",
                "query": "SELECT * FROM Accounts"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Подробные сведения об этих свойствах см. в разделе [Действие поиска](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Дальнейшие действия
В таблице [Поддерживаемые хранилища данных](copy-activity-overview.md#supported-data-stores-and-formats) приведен список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования в фабрике данных Azure.
