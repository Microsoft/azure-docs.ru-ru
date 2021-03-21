---
title: Копирование данных из Xero с помощью Фабрики данных Azure
description: Узнайте, как копировать данные из Xero в поддерживаемые хранилища данных-приемники с помощью действия копирования в конвейере фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/26/2021
ms.author: jingwang
ms.openlocfilehash: d795f8355943032751b911423b8aaa93b2df3206
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100366914"
---
# <a name="copy-data-from-xero-using-azure-data-factory"></a>Копирование данных из Xero с помощью Фабрики данных Azure

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описывается, как с помощью действия копирования в фабрике данных Azure копировать данные из Xero. Это продолжение [статьи об обзоре действия копирования](copy-activity-overview.md), в которой представлены общие сведения о действии копирования.

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Этот соединитель Xero поддерживается для следующих действий:

- [Действие копирования](copy-activity-overview.md) с использованием [матрицы поддерживаемых источников и приемников](copy-activity-overview.md)
- [Действие поиска](control-flow-lookup-activity.md)

Данные из Xero можно скопировать в любое поддерживаемое хранилище данных, используемое в качестве приемника. Список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования, приведен в таблице [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats).

В частности, этот соединитель Xero поддерживает:

- Проверка подлинности OAuth 2,0 и OAuth 1,0. Для OAuth 1,0 соединитель поддерживает [закрытое приложение](https://developer.xero.com/documentation/getting-started/getting-started-guide) Xero, но не общедоступное приложение.
- все таблицы Xero (конечные точки API), кроме таблицы "Отчеты".

## <a name="getting-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Следующие разделы содержат сведения о свойствах, которые используются для определения сущностей фабрики данных, относящихся к соединителю Xero.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы Xero поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства type необходимо задать значение **Xero**. | Да |
| connectionProperties | Группа свойств, определяющих способ подключения к Xero. | Да |
| ***В разделе `connectionProperties` :*** | | |
| узел | Конечная точка сервера Xero (`api.xero.com`).  | Да |
| authenticationType | Допустимые значения: `OAuth_2.0` и `OAuth_1.0` . | Да |
| consumerKey | Для OAuth 2,0 укажите **идентификатор клиента** для приложения Xero.<br>Для OAuth 1,0 укажите ключ потребителя, связанный с приложением Xero.<br>Пометьте это поле как SecureString, чтобы безопасно хранить его в фабрике данных, или [добавьте ссылку на секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md). | Да |
| privateKey | Для OAuth 2,0 укажите **секрет клиента** для приложения Xero.<br>Для OAuth 1,0 укажите закрытый ключ из PEM-файла, созданного для частного приложения Xero, в разделе [Создание пары открытого и закрытого ключей](https://developer.xero.com/documentation/auth-and-limits/create-publicprivate-key). Примечание для **создания privateKey. PEM с нумбитс 512** с помощью `openssl genrsa -out privatekey.pem 512` , 1024 не поддерживается. Включает весь текст из PEM-файла, в том числе окончания строк Unix (\n). Пример см. ниже.<br/><br>Пометьте это поле как SecureString, чтобы безопасно хранить его в фабрике данных, или [добавьте ссылку на секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md). | Да |
| tenantId | Идентификатор клиента, связанный с приложением Xero. Применимо к проверке подлинности OAuth 2,0.<br>Сведения о том, как получить идентификатор клиента, см. в [разделе Проверка клиентов, которым вы имеете право доступа](https://developer.xero.com/documentation/oauth2/auth-flow). | Да для проверки подлинности OAuth 2,0 |
| refreshtoken | Применимо к проверке подлинности OAuth 2,0.<br/>Маркер обновления OAuth 2,0 связан с приложением Xero и используется для обновления маркера доступа. срок действия маркера доступа истекает через 30 минут. Узнайте, как работает поток авторизации Xero и как получить маркер обновления из [этой статьи](https://developer.xero.com/documentation/oauth2/auth-flow). Чтобы получить маркер обновления, необходимо запросить [область offline_access](https://developer.xero.com/documentation/oauth2/scopes). <br/>**Знание ограничения**: Примечание. Xero сбрасывает маркер обновления после его использования для обновления маркера доступа. Для рабочей нагрузки перед выполнением каждого действия копирования необходимо задать допустимый маркер обновления для использования ADF.<br/>Пометьте это поле как SecureString, чтобы безопасно хранить его в фабрике данных, или [добавьте ссылку на секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md). | Да для проверки подлинности OAuth 2,0 |
| useEncryptedEndpoints | Указывает, шифруются ли конечные точки источника данных с помощью протокола HTTPS. Значение по умолчанию — true.  | Нет |
| useHostVerification | Указывает, требуется ли имя узла в сертификате сервера для соответствия имени узла сервера при подключении по протоколу TLS. Значение по умолчанию — true.  | Нет |
| usePeerVerification | Указывает, следует ли проверять удостоверение сервера при подключении по протоколу TLS. Значение по умолчанию — true.  | Нет |

**Пример: проверка подлинности OAuth 2,0**

```json
{
    "name": "XeroLinkedService",
    "properties": {
        "type": "Xero",
        "typeProperties": {
            "connectionProperties": { 
                "host": "api.xero.com",
                "authenticationType":"OAuth_2.0", 
                "consumerKey": {
                    "type": "SecureString",
                    "value": "<client ID>"
                },
                "privateKey": {
                    "type": "SecureString",
                    "value": "<client secret>"
                },
                "tenantId": "<tenant ID>", 
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

**Пример: проверка подлинности OAuth 1,0**

```json
{
    "name": "XeroLinkedService",
    "properties": {
        "type": "Xero",
        "typeProperties": {
            "connectionProperties": {
                "host": "api.xero.com", 
                "authenticationType":"OAuth_1.0", 
                "consumerKey": {
                    "type": "SecureString",
                    "value": "<consumer key>"
                },
                "privateKey": {
                    "type": "SecureString",
                    "value": "<private key>"
                }, 
                "useEncryptedEndpoints": true,
                "useHostVerification": true,
                "usePeerVerification": true
            }
        }
    }
}
```

**Пример значения закрытого ключа**

Включает весь текст из PEM-файла, в том числе окончания строк Unix (\n).

```
"-----BEGIN RSA PRIVATE KEY-----\nMII***************************************************P\nbu****************************************************s\nU/****************************************************B\nA*****************************************************W\njH****************************************************e\nsx*****************************************************l\nq******************************************************X\nh*****************************************************i\nd*****************************************************s\nA*****************************************************dsfb\nN*****************************************************M\np*****************************************************Ly\nK*****************************************************Y=\n-----END RSA PRIVATE KEY-----"
```

## <a name="dataset-properties"></a>Свойства набора данных

Полный список разделов и свойств, доступных для определения наборов данных, см. в статье о [наборах данных](concepts-datasets-linked-services.md). В этом разделе содержится список свойств, поддерживаемых набором данных Xero.

Чтобы копировать данные из Xero, установите свойство type набора данных **XeroObject**. Поддерживаются следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type набора данных должно иметь значение **ксерубжект** . | Да |
| tableName | Имя таблицы. | Нет (если свойство query указано в источнике действия) |

**Пример**

```json
{
    "name": "XeroDataset",
    "properties": {
        "type": "XeroObject",
        "typeProperties": {},
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<Xero linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). В этом разделе содержится список свойств, поддерживаемых источником Xero.

### <a name="xero-as-source"></a>Xero в качестве источника

Чтобы копировать данные из HTTP, установите тип источника **XeroSource** в действии копирования. В разделе **source** действия копирования поддерживаются следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство type источника действия копирования должно иметь значение **XeroSource**. | Да |
| query | Используйте пользовательский SQL-запрос для чтения данных. Например: `"SELECT * FROM Contacts"`. | Нет (если для набора данных задано свойство tableName) |

**Пример**.

```json
"activities":[
    {
        "name": "CopyFromXero",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Xero input dataset name>",
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
                "type": "XeroSource",
                "query": "SELECT * FROM Contacts"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

При указании запроса Xero обратите внимание на следующее:

- Таблицы со сложными элементами необходимо разделять на несколько. Например, банковские переводы имеют сложную структуру данных LineItems. Поэтому данные перевода сопоставляются с таблицами `Bank_Transaction` и `Bank_Transaction_Line_Items`, где `Bank_Transaction_ID` — это внешний ключ, который связывает их.

- Данные Xero имеют две схемы: `Minimal` (по умолчанию) и `Complete`. Полная схема содержит необходимые таблицы вызовов, которые требуют дополнительные данные (например, столбец идентификатора) перед выполнением запроса.

Ниже приведены таблицы с одинаковой информацией в полной и минимальной схеме. Чтобы уменьшить число вызовов API, используйте минимальную схему (по умолчанию).

- Bank_Transactions
- Contact_Groups 
- Контакты 
- Contacts_Sales_Tracking_Categories 
- Contacts_Phones 
- Contacts_Addresses 
- Contacts_Purchases_Tracking_Categories 
- Credit_Notes 
- Credit_Notes_Allocations 
- Expense_Claims 
- Expense_Claim_Validation_Errors
- Счета 
- Invoices_Credit_Notes
- Invoices_ Prepayments 
- Invoices_Overpayments 
- Manual_Journals 
- Overpayments 
- Overpayments_Allocations 
- Prepayments 
- Prepayments_Allocations 
- Receipts 
- Receipt_Validation_Errors 
- Tracking_Categories

Следующие таблицы можно запрашивать только с использованием полной схемы:

- Complete.Bank_Transaction_Line_Items 
- Complete.Bank_Transaction_Line_Item_Tracking 
- Complete.Contact_Group_Contacts 
- Complete.Contacts_Contact_ Persons 
- Complete.Credit_Note_Line_Items 
- Complete.Credit_Notes_Line_Items_Tracking 
- Complete.Expense_Claim_ Payments 
- Complete.Expense_Claim_Receipts 
- Complete.Invoice_Line_Items 
- Complete.Invoices_Line_Items_Tracking
- Complete.Manual_Journal_Lines 
- Complete.Manual_Journal_Line_Tracking 
- Complete.Overpayment_Line_Items 
- Complete.Overpayment_Line_Items_Tracking 
- Complete.Prepayment_Line_Items 
- Complete.Prepayment_Line_Item_Tracking 
- Complete.Receipt_Line_Items 
- Complete.Receipt_Line_Item_Tracking 
- Complete.Tracking_Category_Options

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Подробные сведения об этих свойствах см. в разделе [Действие поиска](control-flow-lookup-activity.md).


## <a name="next-steps"></a>Дальнейшие действия
Список источников данных, которые поддерживает действие копирования, приведен в таблице [поддерживаемых хранилищ данных и форматов](copy-activity-overview.md#supported-data-stores-and-formats).
