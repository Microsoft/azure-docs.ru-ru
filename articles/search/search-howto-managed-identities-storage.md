---
title: Настройка подключения к учетной записи хранения с помощью управляемого удостоверения
titleSuffix: Azure Cognitive Search
description: Узнайте, как настроить подключение индексатора к учетной записи хранения Azure с помощью управляемого удостоверения.
manager: luisca
author: markheff
ms.author: maheff
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: f26ca04955dfa854a8ee17b7aa255a6ed991b8df
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94358377"
---
# <a name="set-up-a-connection-to-an-azure-storage-account-using-a-managed-identity"></a>Настройка подключения к учетной записи хранения Azure с использованием управляемого удостоверения

На этой странице описывается, как настроить подключение индексатора к учетной записи хранения Azure с использованием управляемого удостоверения вместо того, чтобы добавлять учетные данные в строку подключения к объекту источника данных.

Перед изучением этой функции желательно иметь представление о том, что такое индексатор и как настроить индексатор для источника данных. См. подробнее в следующих статьях:
* [Обзор индексатора](search-indexer-overview.md)
* [Индексатор BLOB-объектов](search-howto-indexing-azure-blob-storage.md)
* [Индексатор Azure Data Lake Storage 2-го поколения](search-howto-index-azure-data-lake-storage.md)
* [Индексатор таблиц Azure](search-howto-indexing-azure-tables.md)

## <a name="set-up-the-connection"></a>Настройка подключения

### <a name="1---turn-on-system-assigned-managed-identity"></a>1\. Включение управляемого удостоверения, назначаемого системой

Если включено управляемое удостоверение, назначаемое системой, Azure самостоятельно создаст удостоверение для службы поиска, которое можно использовать для проверки подлинности в других службах Azure для того же клиента и той же подписки. Затем это удостоверение можно использовать в назначениях управления доступом на основе ролей Azure (Azure RBAC), разрешающих доступ к данным во время индексирования.

![Включение управляемого удостоверения, назначаемого системой](./media/search-managed-identities/turn-on-system-assigned-identity.png "Включение управляемого удостоверения, назначаемого системой")

Нажав **Сохранить**, вы увидите идентификатор объекта, назначенный вашей службе поиска.

![Идентификатор объекта](./media/search-managed-identities/system-assigned-identity-object-id.png "Идентификатор объекта.")
 
### <a name="2---add-a-role-assignment"></a>2\. Добавление назначения роли

На этом шаге вы предоставите службе "Когнитивный поиск Azure" разрешение на чтение данных из учетной записи хранения.

1. На портале Azure перейдите к учетной записи хранения, которая содержит данные для индексирования.
2. Выберите **Управление доступом (IAM)**
3. Щелкните **Добавить**, а затем **Добавить назначение роли**.

    ![Добавление назначения роли](./media/search-managed-identities/add-role-assignment-storage.png "Добавление назначения роли")

4. Выберите нужные роли в зависимости от типа учетной записи хранения, которую вы намерены индексировать.
    1. Для хранилища BLOB-объектов Azure необходимо добавить службу поиска в роль **читателя данных BLOB-объекта хранилища** .
    1. Azure Data Lake Storage 2-го поколения требует, чтобы служба поиска была добавлена в роль **читателя данных BLOB-объекта хранилища** .
    1. Для хранилища таблиц Azure необходимо добавить службу поиска в роль **чтения и доступа к данным** .
5.  В поле **Назначение доступа к** сохраните выбранное значение **Пользователь, группа или субъект-служба Azure AD**.
6.  Найдите службу поиска, выберите ее и щелкните **Сохранить**.

    Пример для хранилища BLOB-объектов Azure и Azure Data Lake Storage 2-го поколения:

    ![Назначение роли для чтения данных BLOB-объектов хранилища](./media/search-managed-identities/add-role-assignment-storage-blob-data-reader.png "Назначение роли для чтения данных BLOB-объектов хранилища")

    Пример для хранилища таблиц Azure:

    ![Назначение роли для чтения и доступа к данным](./media/search-managed-identities/add-role-assignment-reader-and-data-access.png "Назначение роли для чтения и доступа к данным")

### <a name="3---create-the-data-source"></a>3\. Создание источника данных

[REST API](/rest/api/searchservice/create-data-source), портал Azure и [пакет SDK для .NET](/dotnet/api/azure.search.documents.indexes.models.searchindexerdatasourceconnection) поддерживают строку подключения управляемого удостоверения. Ниже приведен пример создания источника данных для индексации данных из учетной записи хранения с помощью [REST API](/rest/api/searchservice/create-data-source) и строки подключения управляемой идентификации. Формат строки подключения управляемого удостоверения одинаков для REST API, пакета SDK для .NET и портал Azure.

Если индексация выполняется по учетной записи хранения, источник данных должен иметь следующие обязательные свойства:

* Свойство **name** — уникальное имя источника данных в службе поиска.
* **type**
    * Хранилище BLOB-объектов Azure: `azureblob`.
    * Хранилище таблиц Azure: `azuretable`.
    * Azure Data Lake Storage 2-го поколения: **тип** предоставляется после регистрации на использование предварительной версии через [эту форму](https://aka.ms/azure-cognitive-search/mi-preview-request).
* **credentials**
    * Если для проверки подлинности используется управляемое удостоверение, формат **учетных данных** отличается от того, который используется без управляемого удостоверения. Здесь вы предоставите ResourceId без ключа учетной записи и пароля. ResourceId должен содержать идентификатор подписки для учетной записи хранения, группу ресурсов для учетной записи хранения и имя учетной записи хранения.
    * Формат управляемого удостоверения: 
        * *ResourceId=/subscriptions/**идентификатор_подписки**/resourceGroups/**имя_группы_ресурсов**/providers/Microsoft.Storage/storageAccounts/**имя_учетной_записи_хранения**/;* .
* Свойство **container** определяет контейнер или таблицу в учетной записи хранения. По умолчанию все большие двоичные объекты в контейнере доступны для извлечения. Если требуется индексирование больших двоичных объектов из определенного виртуального каталога, можно указать этот каталог с помощью дополнительного параметра **query**.

Пример для создания большого двоичного объекта в качестве источника данных с помощью [REST API](/rest/api/searchservice/create-data-source):

```
POST https://[service name].search.windows.net/datasources?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
    "name" : "blob-datasource",
    "type" : "azureblob",
    "credentials" : { "connectionString" : "ResourceId=/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/resource-group-name/providers/Microsoft.Storage/storageAccounts/storage-account-name/;" },
    "container" : { "name" : "my-container", "query" : "<optional-virtual-directory-name>" }
}   
```

### <a name="4---create-the-index"></a>4\. Создание индекса

Индекс задает поля в документе, атрибуты, и другие компоненты, которые определяют процедуру поиска.

Ниже описан процесс создания индекса с доступным для поиска полем `content` для хранения текста, извлеченного из больших двоичных объектов:   

```http
    POST https://[service name].search.windows.net/indexes?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
          "name" : "my-target-index",
          "fields": [
            { "name": "id", "type": "Edm.String", "key": true, "searchable": false },
            { "name": "content", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false }
          ]
    }
```

Дополнительные сведения о создании индексов см. в статье [Create Index](/rest/api/searchservice/create-index) (Создание индекса).

### <a name="5---create-the-indexer"></a>5\. Создание индексатора

Индексатор соединяет источник данных с целевым индексом поиска и предоставляет расписание для автоматизации обновления данных.

Когда индекс и источник данных будут созданы, можно создать индексатор.

Пример определения индексатора для BLOB-объектов:

```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "blob-indexer",
      "dataSourceName" : "blob-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" }
    }
```

Этот индексатор будет выполняться каждые два часа (интервал расписания имеет значение "PT2H"). Чтобы запускать индексатор каждые 30 минут, задайте интервал "PT30M". Самый короткий интервал, который можно задать, составляет 5 минут. Расписание является необязательным. Если оно не указано, то индексатор выполняется только один раз при его создании. Однако индексатор можно запустить по запросу в любое время.   

Дополнительные сведения об API создания индексатора см. в разделе [Создание индексатора](/rest/api/searchservice/create-indexer).

Дополнительные сведения об определении расписаний индексаторов для Когнитивного поиска Azure см. [здесь](search-howto-schedule-indexers.md).

## <a name="accessing-secure-data-in-storage-accounts"></a>Доступ к защищенным данным в учетных записях хранения

Учетные записи хранения Azure можно дополнительно защитить с помощью брандмауэров и виртуальных сетей. Если вы хотите индексировать содержимое из учетной записи хранения BLOB-объектов или Data Lake учетной записи хранения Gen2, защищенной с помощью брандмауэра или виртуальной сети, следуйте инструкциям по [безопасному доступу к данным в учетных записях хранения через исключение доверенной службы](search-indexer-howto-access-trusted-service-exception.md).

## <a name="see-also"></a>См. также раздел

Дополнительные сведения об индексаторах службы хранилища Azure:

* [Индексатор BLOB-объектов](search-howto-indexing-azure-blob-storage.md)
* [Индексатор Azure Data Lake Storage 2-го поколения](search-howto-index-azure-data-lake-storage.md)
* [Индексатор таблиц Azure](search-howto-indexing-azure-tables.md)