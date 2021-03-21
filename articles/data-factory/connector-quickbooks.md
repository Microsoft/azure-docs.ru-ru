---
title: Копирование данных из QuickBooks Online с помощью Фабрики данных Azure (предварительная версия)
description: Узнайте, как копировать данные из QuickBooks Online в поддерживаемые хранилища данных в качестве приемников с помощью действия копирования в конвейере Фабрики данных Azure.
author: linda33wj
ms.author: jingwang
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 01/15/2021
ms.openlocfilehash: eecbcb817ad31480f8f6c3c7272328d06b17c081
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100384067"
---
# <a name="copy-data-from-quickbooks-online-using-azure-data-factory-preview"></a>Копирование данных из QuickBooks Online с помощью Фабрики данных Azure (предварительная версия)
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описывается, как с помощью действия копирования в фабрике данных Azure копировать данные из QuickBooks Online. Это продолжение [статьи об обзоре действия копирования](copy-activity-overview.md), в которой представлены общие сведения о действии копирования.

> [!IMPORTANT]
> Сейчас этот соединитель доступен в режиме предварительной версии. Попробуйте поработать с ним и оставьте свой отзыв. Если вы хотите использовать в своем решении зависимость от соединителей в предварительной версии, обратитесь в службу [поддержки Azure](https://azure.microsoft.com/support/).

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Этот соединитель QuickBooks поддерживается для следующих действий:

- [Действие копирования](copy-activity-overview.md) с использованием [матрицы поддерживаемых источников и приемников](copy-activity-overview.md)
- [Действие поиска](control-flow-lookup-activity.md)

Данные из QuickBooks Online можно скопировать в любое поддерживаемое хранилище данных, используемое в качестве приемника. Список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования, приведен в таблице [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats).

Этот соединитель поддерживает проверку подлинности QuickBooks OAuth 2,0.

## <a name="getting-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Следующие разделы содержат сведения о свойствах, которые используются для определения сущностей фабрики данных, относящихся к соединителю QuickBooks.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы QuickBooks поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства type необходимо задать значение **QuickBooks**. | Да |
| connectionProperties | Группа свойств, определяющих способ подключения к QuickBooks. | Да |
| ***В разделе `connectionProperties` :*** | | |
| endpoint | Конечная точка сервера QuickBooks Online. (это quickbooks.api.intuit.com).  | Да |
| companyId | Идентификатор компании QuickBooks для авторизации. Сведения о том, как найти идентификатор компании, см. в разделе [разделы справки найти свой идентификатор компании](https://quickbooks.intuit.com/community/Getting-Started/How-do-I-find-my-Company-ID/m-p/185551). | Да |
| consumerKey | Идентификатор клиента приложения QuickBooks Online для проверки подлинности OAuth 2,0. Дополнительные сведения см. [здесь](https://developer.intuit.com/app/developer/qbo/docs/develop/authentication-and-authorization/oauth-2.0#obtain-oauth2-credentials-for-your-app). | Да |
| consumerSecret | Секрет клиента приложения QuickBooks Online для проверки подлинности OAuth 2,0. Пометьте это поле как SecureString, чтобы безопасно хранить его в фабрике данных, или [добавьте ссылку на секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md). | Да |
| refreshtoken | Маркер обновления OAuth 2,0, связанный с приложением QuickBooks. Дополнительные сведения см. [здесь](https://developer.intuit.com/app/developer/qbo/docs/develop/authentication-and-authorization/oauth-2.0#obtain-oauth2-credentials-for-your-app). Примечание. срок действия маркера обновления истечет через 180 дней. Клиент должен регулярно обновлять маркер обновления. <br/>Пометьте это поле как SecureString, чтобы безопасно хранить его в фабрике данных, или [добавьте ссылку на секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md).| Да |
| useEncryptedEndpoints | Указывает, шифруются ли конечные точки источника данных с помощью протокола HTTPS. Значение по умолчанию — true.  | Нет |

**Пример**.

```json
{
    "name": "QuickBooksLinkedService",
    "properties": {
        "type": "QuickBooks",
        "typeProperties": {
            "connectionProperties": {
                "endpoint": "quickbooks.api.intuit.com",
                "companyId": "<company id>",
                "consumerKey": "<consumer key>", 
                "consumerSecret": {
                     "type": "SecureString",
                     "value": "<clientSecret>"
                },
                "refreshToken": {
                     "type": "SecureString",
                     "value": "<refresh token>"
                },
                "useEncryptedEndpoints": true
            }
        }
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных

Полный список разделов и свойств, доступных для определения наборов данных, см. в статье о [наборах данных](concepts-datasets-linked-services.md). Этот раздел содержит список свойств, поддерживаемых набором данных QuickBooks.

Чтобы скопировать данные из QuickBooks Online, установите для набора данных тип **QuickBooksObject**. Поддерживаются следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type набора данных должно иметь значение **куиккбуксобжект** . | Да |
| tableName | Имя таблицы. | Нет (если свойство query указано в источнике действия) |

**Пример**

```json
{
    "name": "QuickBooksDataset",
    "properties": {
        "type": "QuickBooksObject",
        "typeProperties": {},
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<QuickBooks linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). Этот раздел содержит список свойств, поддерживаемых источником QuickBooks.

### <a name="quickbooks-as-source"></a>QuickBooks в качестве источника

Чтобы копировать данные из QuickBooks Online, установите тип источника **QuickBooksSource** в действии копирования. В разделе **source** действия копирования поддерживаются следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство type источника действия копирования должно иметь значение **QuickBooksSource**. | Да |
| query | Используйте пользовательский SQL-запрос для чтения данных. Например: `"SELECT * FROM "Bill" WHERE Id = '123'"`. | Нет (если для набора данных задано свойство tableName) |

**Пример**.

```json
"activities":[
    {
        "name": "CopyFromQuickBooks",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<QuickBooks input dataset name>",
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
                "type": "QuickBooksSource",
                "query": "SELECT * FROM \"Bill\" WHERE Id = '123' "
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```
## <a name="copy-data-from-quickbooks-desktop"></a>Копирование данных из QuickBooks Desktop

Действие копирования в Фабрике данных Azure не может копировать данные напрямую из Quickbooks Desktop. Чтобы скопировать данные из Quickbooks Desktop, экспортируйте данные Quickbooks в файл с разделителями запятыми (CSV) и отправьте этот файл в хранилище BLOB-объектов Azure. Оттуда вы сможете скопировать данные в любой выбранный приемник с помощью Фабрики данных.

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Подробные сведения об этих свойствах см. в разделе [Действие поиска](control-flow-lookup-activity.md).


## <a name="next-steps"></a>Дальнейшие действия
В таблице [Поддерживаемые хранилища данных](copy-activity-overview.md#supported-data-stores-and-formats) приведен список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования в фабрике данных Azure.
