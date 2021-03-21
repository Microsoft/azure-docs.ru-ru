---
title: Копирование данных из списка SharePoint Online с помощью Фабрики данных Azure
description: Сведения о том, как копировать данные из списка SharePoint Online в поддерживаемые хранилища данных в роли приемников с помощью действия копирования в конвейере Фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 05/19/2020
ms.author: jingwang
ms.openlocfilehash: f8074b69b97a6ef96837e73a1082d2deb67084d9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102177867"
---
# <a name="copy-data-from-sharepoint-online-list-by-using-azure-data-factory"></a>Копирование данных из списка SharePoint Online с помощью Фабрики данных Azure
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описывается, как с помощью действия копирования в Фабрике данных Azure копировать данные из списка SharePoint Online. Это продолжение статьи о [действии копирования в Фабрике данных Azure](copy-activity-overview.md), в которой представлены общие сведения о действии копирования.

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Этот соединитель списка SharePoint Online поддерживается для следующих действий.

- [Действие копирования](copy-activity-overview.md) с использованием матрицы [поддерживаемых источников и приемников](copy-activity-overview.md).
- [Действие поиска](control-flow-lookup-activity.md)

Данные из списка SharePoint Online можно скопировать в любое поддерживаемое в роли приемника хранилище данных. Список хранилищ данных, поддерживаемых действием копирования в качестве источников и приемников, приведен в разделе [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats).

В частности, этот соединитель списка SharePoint Online использует проверку подлинности по субъекту-службе и получает данные по протоколу OData.

> [!TIP]
> Этот соединитель поддерживает копирование данных из **списка** SharePoint Online, но не из файла. Процедура копирования файла из SharePoint Online описана [здесь](#copy-file-from-sharepoint-online).

## <a name="prerequisites"></a>Предварительные требования

Этот соединитель списка SharePoint Online использует проверку подлинности по субъекту-службе для подключения к SharePoint. Чтобы настроить этот механизм, выполните следующие действия:

1. Чтобы зарегистрировать сущность приложения в Azure Active Directory (Azure AD), необходимо следовать указаниям из раздела [Регистрация приложения в клиенте Azure AD](../storage/common/storage-auth-aad-app.md#register-your-application-with-an-azure-ad-tenant). Запишите следующие значения, которые используются для определения связанной службы:

    - Идентификатор приложения
    - Ключ приложения
    - Tenant ID

2. Предоставьте сайту SharePoint Online разрешения на доступ к зарегистрированному приложению: 

    > [!NOTE]
    > Для этой операции требуется разрешение владельца сайта SharePoint Online. Чтобы найти владельца, перейдите на главную страницу сайта, щелкните элемент "Участников: X" в правом углу и проверьте, для кого указано роль "Владелец".

    1. Откройте ссылку на сайт SharePoint Online, например `https://[your_site_url]/_layouts/15/appinv.aspx` (заменив URL-адрес сайта).
    2. Найдите зарегистрированный идентификатор приложения, заполните пустые поля и щелкните "Создать".

        - Домен приложения: `localhost.com`
        - URL-адрес перенаправления: `https://www.localhost.com`
        - XML-код запроса разрешения:

        ```xml
        <AppPermissionRequests AllowAppOnlyPolicy="true">
            <AppPermissionRequest Scope="http://sharepoint/content/sitecollection/web" Right="Read"/>
        </AppPermissionRequests>
        ```

        ![Предоставление разрешения в SharePoint](media/connector-sharepoint-online-list/sharepoint-online-grant-permission.png)

    3. Щелкните "Сделать доверенным" для этого приложения.

## <a name="get-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

В следующих разделах приведены сведения о свойствах, которые можно использовать при определении сущностей Фабрики данных для соединителя списка SharePoint Online.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы списка SharePoint Online поддерживаются следующие свойства.

| **Свойство**        | **Описание**                                              | **Обязательно** |
| ------------------- | ------------------------------------------------------------ | ------------ |
| type                | Для свойства type необходимо задать значение  **SharePointOnlineList**.  | Да          |
| siteUrl             | URL-адрес сайта SharePoint Online, например `https://contoso.sharepoint.com/sites/siteName`. | Да          |
| servicePrincipalId  | Идентификатор приложения (клиента), зарегистрированного в Azure Active Directory. | Да          |
| servicePrincipalKey | Ключ приложения. Пометьте это поле как **SecureString**, чтобы безопасно хранить его в фабрике данных, или [добавьте ссылку на секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md). | Да          |
| tenantId            | Идентификатор клиента, установленный для приложения.          | Да          |
| connectVia          | [Среда выполнения интеграции](concepts-integration-runtime.md), используемая для подключения к хранилищу данных. Дополнительные сведения см. в разделе [Предварительные условия](#prerequisites) ранее в этой статье. Если не указано другое, по умолчанию используется интегрированная Azure Integration Runtime. | Нет           |

**Пример**.

```json
{
    "name": "SharePointOnlineList",
    "properties": {
        "type": "SharePointOnlineList",
        "typeProperties": {
            "siteUrl": "<site URL>",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<service principal key>"
            },
            "tenantId": "<tenant ID>"
        }
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных

Полный список разделов и свойств, используемых для определения наборов данных, приведен в статье [Наборы данных и связанные службы в фабрике данных Azure](concepts-datasets-linked-services.md). Следующий раздел содержит список свойств, поддерживаемых для набора данных таблицы SAP.

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства **type** набора данных необходимо задать значение **SharePointOnlineLResource**. | Да |
| listName | Имя списка SharePoint Online. | Да |

**Пример**

```json
{
    "name": "SharePointOnlineListDataset",
    "properties":
    {
        "type": "SharePointOnlineListResource",
        "linkedServiceName": {
            "referenceName": "<SharePoint Online List linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties":
        {
            "listName": "<name of the list>"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, доступных для определения действий, см. в статье, посвященной [конвейерам и действиям в Фабрике данных Azure](concepts-pipelines-activities.md).  Следующий раздел содержит список свойств, поддерживаемых для списка SharePoint Online в роли источника.

### <a name="sharepoint-online-list-as-source"></a>Список SharePoint Online в роли источника

Для копирования данных из списка SharePoint Online действие копирования поддерживает следующие свойства в разделе **Источник**.

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство **type** источника действия копирования должно иметь значение **SharePointOnlineListSource**. | Да |
| query | Настраиваемые параметры запроса OData для фильтрации данных. Например, `"$top=10&$select=Title,Number"`. | нет |
| httpRequestTimeout | Время ожидания ответа для HTTP-запроса (в секундах). По умолчанию используется значение 300 (5 минут). | нет |

**Пример**

```json
"activities":[
    {
        "name": "CopyFromSharePointOnlineList",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SharePoint Online List input dataset name>",
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
            "source": {
                "type": "SharePointOnlineListSource",
                "query": "<OData query e.g. $top=10&$select=Title,Number>"
            }, 
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

> [!NOTE]
> В фабрике данных Azure нельзя выбрать более одного типа данных *выборки* для источника списка SharePoint Online.

## <a name="data-type-mapping-for-sharepoint-online-list"></a>Сопоставление типов данных для списка SharePoint Online

При копировании данных из списка SharePoint Online используются следующие сопоставления типов данных списка SharePoint Online с промежуточными типами данных Фабрики данных Azure. 

| **Тип данных SharePoint Online**                 | **Тип данных OData**                                  | **Промежуточный тип данных Фабрики данных Azure** |
| ----------------------------------------------- | ---------------------------------------------------- | ---------------------------------------- |
| Одна строка текста                             | Edm.String                                           | Строка                                   |
| Несколько строк текста                          | Edm.String                                           | Строка                                   |
| Выбор (меню для выбора)                    | Edm.String                                           | Строка                                   |
| Число (1, 1,0, 100)                            | Edm.Double                                           | Double                                   |
| Currency ($, ¥, &euro; )                              | Edm.Double                                           | Double                                   |
| Дата и время                                   | Edm.DateTime                                         | Дата и время                                 |
| Поиск (информация уже имеется на сайте)       | Edm.Int32                                            | Int32                                    |
| Да/Нет (флажок)                              | Edm.Boolean                                          | Логическое                                  |
| Пользователь или группа                                 | Edm.Int32                                            | Int32                                    |
| Гиперссылка или рисунок                            | Edm.String                                           | Строка                                   |
| Вычисляемый (вычисление на основе других столбцов) | Edm.String / Edm.Double / Edm.DateTime / Edm.Boolean | String / Double / DateTime / Boolean     |
| Вложение                                      | Не поддерживается                                        |                                          |
| Результат задачи                                    | Не поддерживается                                        |                                          |
| Внешние данные                                   | Не поддерживается                                        |                                          |
| Управляемые метаданные                                | Не поддерживается                                        |                                          |

## <a name="copy-file-from-sharepoint-online"></a>Копирование файлов из SharePoint Online

Чтобы скопировать файл из SharePoint Online, вы можете применить **веб-действие** для проверки подлинности и получения маркера доступа из SharePoint Online, а затем передать его в **действие копирования**, указав **соединитель HTTP в качестве источника**.

![Поток копирования файлов SharePoint](media/connector-sharepoint-online-list/sharepoint-online-copy-file-flow.png)

1. Чтобы создать приложение AAD и предоставить разрешение на доступ к SharePoint Online, выполните инструкции из раздела [Предварительные требования](#prerequisites). 

2. Создайте **веб-действие**, чтобы получить маркер доступа из SharePoint Online:

    - **URL-адрес**: `https://accounts.accesscontrol.windows.net/[Tenant-ID]/tokens/OAuth/2`. Замените идентификатор клиента.
    - **Метод:** POST
    - **Заголовки**:
        - Content-Type: application/x-www-form-urlencoded
    - **Текст**: `grant_type=client_credentials&client_id=[Client-ID]@[Tenant-ID]&client_secret=[Client-Secret]&resource=00000003-0000-0ff1-ce00-000000000000/[Tenant-Name].sharepoint.com@[Tenant-ID]`. Замените значение идентификатора клиента, секрета клиента, идентификатора арендатора и имени арендатора.

    > [!CAUTION]
    > В веб-действии установите для параметра "Защищенный вывод" значение true, чтобы значение маркера не сохранялось в формате обычного текста. Для всех дальнейших действий, которые используют это значение, параметр "Защищенный ввод" должен иметь значение true.

3. Создайте цепочку с **действием копирования**, где в качестве источника указан соединитель HTTP, для копирования содержимого файлов SharePoint Online:

    - Связанная служба HTTP:
        - **Базовый URL-адрес**: `https://[site-url]/_api/web/GetFileByServerRelativeUrl('[relative-path-to-file]')/$value`. Замените URL-адрес сайта и относительный путь к файлу. Относительный путь к файлу выглядит примерно так: `/sites/site2/Shared Documents/TestBook.xlsx`.
        - **Тип проверки подлинности**: Анонимный *(чтобы использовать токен носителя, который настраивается позднее в источнике действия копирования)*
    - Набор данных: выберите необходимый формат. Чтобы скопировать файл "как есть", выберите тип "Двоичный".
    - Источник действия копирования:
        - **Метод запроса**: GET
        - **Дополнительный заголовок**: используйте выражение `@{concat('Authorization: Bearer ', activity('<Web-activity-name>').output.access_token)}`, в котором в качестве заголовка авторизации используется токен носителя, созданный вышестоящим веб-действием. Замените имя веб-действия.
    - Настройте приемник действия копирования обычным образом.

> [!NOTE]
> Даже если приложение Azure AD имеет `FullControl` разрешения на доступ к SharePoint Online, нельзя копировать файлы из библиотек документов с включенным IRM.

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Подробные сведения об этих свойствах см. в разделе [Действие поиска](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Дальнейшие действия

В разделе [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats) приведен список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования в Фабрике данных Azure.
