---
title: Копирование данных из Office 365 с помощью фабрики данных Azure
description: Узнайте, как копировать данные из Office 365 в поддерживаемые целевые хранилища данных (приемники) с помощью действия копирования в конвейере Фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 10/20/2019
ms.author: jingwang
ms.openlocfilehash: a7df69e7c5701074b40d6fa8340d8a0e247f00de
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100393009"
---
# <a name="copy-data-from-office-365-into-azure-using-azure-data-factory"></a>Копирование данных из Office 365 в Azure с помощью фабрики данных Azure
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Фабрика данных Azure интегрируется с [Microsoft Graph Data Connect](/graph/data-connect-concept-overview), позволяя вам использовать масштабируемые организационные данные в клиенте Office 365 в Azure в масштабируемом виде и выполнять сборку аналитических приложений и извлекать аналитические сведения на основе этих ценных ресурсов данных. Интеграция с Privileged Access Management обеспечивает безопасное управление доступом к ценным и проверенным данным в Office 365.  Обратитесь к [этой ссылке](/graph/data-connect-concept-overview) , чтобы получить общие сведения о Microsoft Graph Data Connect, и обратитесь к [этой](/graph/data-connect-policies#licensing) ссылке для получения сведений о лицензировании.

Из этой статьи вы узнаете, как с помощью действия копирования в Фабрике данных Azure скопировать данные из Office 365. Это продолжение [статьи об обзоре действия копирования](copy-activity-overview.md), в которой представлены общие сведения о действии копирования.

## <a name="supported-capabilities"></a>Поддерживаемые возможности
Соединитель Office 365 и Microsoft Graph Data Connect позволяют выполнять масштабирование различных типов наборов данных из почтовых ящиков с поддержкой электронной почты Exchange, включая контакты адресной книги, события календаря, сообщения электронной почты, сведения о пользователе, параметры почтового ящика и т. д.  Ознакомьтесь [с полным](/graph/data-connect-datasets) списком доступных наборов данных.

Сейчас в рамках одного действия копирования можно **Копировать данные из Office 365 в [хранилище BLOB-объектов Azure](connector-azure-blob-storage.md), [Azure Data Lake Storage 1-го поколения](connector-azure-data-lake-store.md)и [Azure Data Lake Storage 2-го поколения](connector-azure-data-lake-storage.md) в формате JSON** (тип setOfObjects). Если вы хотите загрузить данные Office 365 в хранилища других типов или в других форматах, добавьте к первому действию копирования дополнительные действия, которые передадут эти данные в любое [поддерживаемое ADF целевое хранилище](copy-activity-overview.md#supported-data-stores-and-formats) (см. столбец "Поддерживается в качестве приемника" в таблице "Поддерживаемые хранилища и форматы данных").

>[!IMPORTANT]
>- Подписка Azure, содержащая фабрику данных и целевое хранилище данных должны размещаться в том же клиенте Azure Active Directory (AAD), что и клиент Office 365.
>- Среда выполнения Azure Integration Runtime, используемая для действия копирования, а также целевой объект должны размещаться в том же регионе, где расположен почтовый ящик пользователей клиента Office 365. В [этой статье](concepts-integration-runtime.md#integration-runtime-location) описано, как определяется расположение Azure Integration Runtime. В [этой таблице](/graph/data-connect-datasets#regions) собран список поддерживаемых регионов Office и соответствующих им регионов Azure.
>- Проверка подлинности субъекта-службы является единственным механизмом проверки подлинности, поддерживаемым для хранилища больших двоичных объектов Azure, Azure Data Lake Storage 1-го поколения и Azure Data Lake Storage 2-го поколения в качестве целевых

## <a name="prerequisites"></a>Предварительные условия

Чтобы скопировать данные из Office 365 в Azure, выполните следующие предварительные требования:

- Администратору клиента Office 365 необходимо выполнить для подключения описанные [здесь](/graph/data-connect-get-started) действия.
- Создайте и настройте веб-приложение AAD в Azure Active Directory.  Инструкции см. в разделе [Создание приложения Azure AD](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal).
- Запишите следующие значения, которые вам потребуются для определения связанной службы для Office 365.
    - Идентификатор клиента. Инструкции см. в статье о [получении идентификатора клиента](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in).
    - Идентификатор приложения и ключ приложения.  Инструкции см. в статье о [получении идентификатора приложения и ключа проверки подлинности](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in).
- Добавьте удостоверение пользователя, который будет обращаться к данным в роли владельца веб-приложения AAD (в AAD откройте "Веб-приложения" > "Параметры" > "Владельцы" > "Добавить владельца"). 
    - Удостоверение пользователя должно находиться в организации Office 365, из которой вы получаете данные, и не должно быть удостоверением гостевого пользователя.

## <a name="approving-new-data-access-requests"></a>Утверждение новых запросов на доступ к данным

Если вы впервые запрашиваете данные для этого контекста (сочетание, к которому обращается таблица данных, в какой целевой учетной записи загружаются данные и какие удостоверения пользователя осуществляют запрос на доступ к данным, вы увидите состояние действия копирования "выполняется" и только при щелчке [ссылки "сведения" в разделе "действия](copy-activity-overview.md#monitoring) " отобразится состояние "рекуестингконсент".  Участник группы лиц, утверждающих доступ к данным, должен утвердить запрос в Privileged Access Management, прежде чем можно будет продолжить извлечение данных.

На [этой странице](/graph/data-connect-tips#approve-pam-requests-via-office-365-admin-portal) описано, как утверждающее лицо может утвердить запрос на доступ к данным, а [здесь](/graph/data-connect-pam) представлены общие сведения об интеграции с Privileged Access Management и настройке группы лиц, утверждающих доступ к данным.

## <a name="policy-validation"></a>Проверка политики

Если в рамках управляемого приложения создан файл определения приложения, и для ресурсов в группе ресурсов для управления назначены политики Azure, для каждого запуска действия копирования этот файл определения приложения проверяет применение назначенных политик. [Здесь](/graph/data-connect-policies#policies) собран список поддерживаемых политик.

## <a name="getting-started"></a>Начало работы

>[!TIP]
>Пошаговое руководство по использованию соединителя Office 365 см. в статье [Загрузка данных из Office 365 с помощью Фабрики данных Azure](load-office-365-data.md).

Можно создать конвейер с использованием действия копирования, воспользовавшись одним из приведенных ниже инструментов или пакетов SDK. Щелкните ссылку, чтобы перейти к учебнику, содержащему пошаговые инструкции по созданию конвейера с использованием действия копирования. 

- [Портал Azure](quickstart-create-data-factory-portal.md)
- [Пакет SDK для .NET](quickstart-create-data-factory-dot-net.md)
- [Пакет SDK для Python](quickstart-create-data-factory-python.md)
- [Azure PowerShell](quickstart-create-data-factory-powershell.md)
- [REST API](quickstart-create-data-factory-rest-api.md)
- [Шаблон Azure Resource Manager](quickstart-create-data-factory-resource-manager-template.md). 

Следующие разделы содержат сведения о свойствах JSON, которые используются для определения сущностей Фабрики данных, относящихся к соединителю Office 365.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы Office 365 поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства type необходимо задать значение **Office365**. | Да |
| office365TenantId | Идентификатор клиента Azure, которому принадлежит учетная запись Office 365. | Да |
| servicePrincipalTenantId | Укажите сведения о клиенте, в котором находится веб-приложение AAD. | Да |
| servicePrincipalId | Укажите идентификатора клиента приложения. | Да |
| servicePrincipalKey | Укажите ключ приложения. Пометьте это поле как SecureString, чтобы безопасно хранить его в Фабрике данных. | Да |
| connectVia | Среда Integration Runtime, используемая для подключения к хранилищу данных.  Если не указано другое, по умолчанию используется интегрированная среда выполнения Azure. | Нет |

>[!NOTE]
> Значения **office365TenantId** и **servicePrincipalTenantId** имеют следующие различия и принимают следующие значения.
>- Если вы корпоративный разработчик и создаете приложение для работы с данными Office 365, которое будет использоваться в пределах организации, в обоих свойствах следует указать один и тот же идентификатор клиента, а именно идентификатор AAD вашей организации.
>- Если вы разработчик независимого поставщика программного обеспечения разрабатывает приложение для своих клиентов, то office365TenantId будет ИДЕНТИФИКАТОРом клиента AAD, а СервицепринЦипалтенантид будет ИДЕНТИФИКАТОРом клиента AAD вашей компании.

**Пример**.

```json
{
    "name": "Office365LinkedService",
    "properties": {
        "type": "Office365",
        "typeProperties": {
            "office365TenantId": "<Office 365 tenant id>",
            "servicePrincipalTenantId": "<AAD app service principal tenant id>",
            "servicePrincipalId": "<AAD app service principal id>",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<AAD app service principal key>"
            }
        }
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных

Полный список разделов и свойств, доступных для определения наборов данных, см. в статье о [наборах данных](concepts-datasets-linked-services.md). В этом разделе содержится список свойств, поддерживаемых набором данных Office 365.

Для копирования данных из Office 365 поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство type для набора данных должно иметь значение **Office365Table**. | Да |
| tableName | Имя набора данных, который нужно извлечь из Office 365. [Здесь](/graph/data-connect-datasets#datasets) вы найдете список наборов данных, доступных для извлечения из Office 365. | Да |

Если вы настраиваете параметр `dateFilterColumn` , `startTime` , `endTime` и `userScopeFilterUri` в наборе данных, он по-прежнему поддерживается как есть, хотя в дальнейшем предлагается использовать новую модель в источнике действия.

**Пример**

```json
{
    "name": "DS_May2019_O365_Message",
    "properties": {
        "type": "Office365Table",
        "linkedServiceName": {
            "referenceName": "<Office 365 linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [],
        "typeProperties": {
            "tableName": "BasicDataSet_v0.Event_v1"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). В этом разделе содержится список свойств, поддерживаемых источником данных Office 365.

### <a name="office-365-as-source"></a>Office 365 в качестве источника

Чтобы скопировать данные из Office 365, в разделе **источник** действия копирования поддерживаются следующие свойства.

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type источника действия копирования должно иметь значение **Office365Source** . | Да |
| алловедграупс | Предикат выбора группы.  Используйте это свойство, чтобы выбрать до 10 групп пользователей, для которых будут извлечены данные.  Если группы не указаны, будут возвращены данные для всей Организации. | Нет |
| усерскопефилтерури | Если `allowedGroups` свойство не задано, можно использовать выражение предиката, которое применяется ко всему клиенту для фильтрации конкретных строк, извлекаемых из Office 365. Формат предиката должен соответствовать формату запроса Microsoft Graph API, например `https://graph.microsoft.com/v1.0/users?$filter=Department eq 'Finance'` . | Нет |
| датефилтерколумн | Имя столбца фильтра даты и времени. Это свойство используется для ограничения диапазона времени, в течение которого извлекаются данные Office 365. | Да, если набор данных имеет один или несколько столбцов DateTime. Список наборов данных, для которых требуется этот фильтр DateTime, см. [здесь](/graph/data-connect-filtering#filtering) . |
| startTime | Начальное значение даты и времени для фильтрации. | Да `dateFilterColumn` , если указан |
| endTime | Конечное значение даты и времени для фильтрации. | Да `dateFilterColumn` , если указан |
| аутпутколумнс | Массив столбцов для копирования в приемник. | Нет |

**Пример**.

```json
"activities": [
    {
        "name": "CopyFromO365ToBlob",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Office 365 input dataset name>",
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
                "type": "Office365Source",
                "dateFilterColumn": "CreatedDateTime",
                "startTime": "2019-04-28T16:00:00.000Z",
                "endTime": "2019-05-05T16:00:00.000Z",
                "userScopeFilterUri": "https://graph.microsoft.com/v1.0/users?$filter=Department eq 'Finance'",
                "outputColumns": [
                    {
                        "name": "Id"
                    },
                    {
                        "name": "CreatedDateTime"
                    },
                    {
                        "name": "LastModifiedDateTime"
                    },
                    {
                        "name": "ChangeKey"
                    },
                    {
                        "name": "Categories"
                    },
                    {
                        "name": "OriginalStartTimeZone"
                    },
                    {
                        "name": "OriginalEndTimeZone"
                    },
                    {
                        "name": "ResponseStatus"
                    },
                    {
                        "name": "iCalUId"
                    },
                    {
                        "name": "ReminderMinutesBeforeStart"
                    },
                    {
                        "name": "IsReminderOn"
                    },
                    {
                        "name": "HasAttachments"
                    },
                    {
                        "name": "Subject"
                    },
                    {
                        "name": "Body"
                    },
                    {
                        "name": "Importance"
                    },
                    {
                        "name": "Sensitivity"
                    },
                    {
                        "name": "Start"
                    },
                    {
                        "name": "End"
                    },
                    {
                        "name": "Location"
                    },
                    {
                        "name": "IsAllDay"
                    },
                    {
                        "name": "IsCancelled"
                    },
                    {
                        "name": "IsOrganizer"
                    },
                    {
                        "name": "Recurrence"
                    },
                    {
                        "name": "ResponseRequested"
                    },
                    {
                        "name": "ShowAs"
                    },
                    {
                        "name": "Type"
                    },
                    {
                        "name": "Attendees"
                    },
                    {
                        "name": "Organizer"
                    },
                    {
                        "name": "WebLink"
                    },
                    {
                        "name": "Attachments"
                    },
                    {
                        "name": "BodyPreview"
                    },
                    {
                        "name": "Locations"
                    },
                    {
                        "name": "OnlineMeetingUrl"
                    },
                    {
                        "name": "OriginalStart"
                    },
                    {
                        "name": "SeriesMasterId"
                    }
                ]
            },
            "sink": {
                "type": "BlobSink"
            }
        }
    }
]
```

## <a name="next-steps"></a>Дальнейшие действия
В таблице [Поддерживаемые хранилища данных](copy-activity-overview.md#supported-data-stores-and-formats) приведен список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования в фабрике данных Azure.