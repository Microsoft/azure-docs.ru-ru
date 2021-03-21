---
title: Настройка индексатора SharePoint Online (Предварительная версия)
titleSuffix: Azure Cognitive Search
description: Настройка индексатора SharePoint Online для автоматизации индексирования содержимого библиотеки документов в Когнитивный поиск Azure.
manager: luisca
author: MarkHeff
ms.author: maheff
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 03/01/2021
ms.openlocfilehash: 5a44c40838b7f7fa9ca499ade49317ff9ce828fe
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102498903"
---
# <a name="how-to-configure-sharepoint-online-indexing-in-cognitive-search-preview"></a>Настройка индексирования SharePoint Online в Когнитивный поиск (Предварительная версия)

> [!IMPORTANT] 
> В настоящее время поддержка SharePoint Online находится в **общедоступной предварительной версии**. Вы можете запросить доступ к предварительной версии для предварительного просмотра, заполнив [эту форму](https://aka.ms/azure-cognitive-search/indexer-preview).
>
> Для предварительной версии функции соглашение об уровне обслуживания не предусмотрено. Мы не рекомендуем использовать ее в рабочей среде. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
> 
> Эта функция доступна в [REST API версии 2020-06-30-Preview](search-api-preview.md) . В настоящее время нет поддержки портала или пакета SDK.

> [!NOTE]
> SharePoint Online поддерживает детализированную модель авторизации, которая определяет доступ для каждого пользователя на уровне документа. Индексатор SharePoint Online не запрашивает эти разрешения в индексе поиска, а Когнитивный поиск не поддерживает авторизацию на уровне документа. Когда документ индексируется из SharePoint Online в службу поиска, содержимое становится доступным для всех, у кого есть доступ на чтение индекса. Если требуются разрешения уровня документа, следует изучить фильтры безопасности, чтобы обрезать результаты несанкционированного содержимого. Дополнительные сведения см. в статье [Фильтрация по безопасности с помощью Active Directoryных удостоверений](search-security-trimming-for-azure-search-with-aad.md).

В этой статье описывается, как использовать Когнитивный поиск Azure для индексирования документов (например, документов PDF, Microsoft Office документов и нескольких других распространенных форматов), хранящихся в библиотеках документов SharePoint Online, в индексе Azure Когнитивный поиск. Во первых, в нем объясняются основы настройки и настройки индексатора. Затем предлагается более углубленно изучить возможные сценарии и поведения.

## <a name="functionality"></a>функциональное назначение;
Индексатор в Когнитивный поиск Azure — это средство, которое извлекает данные и метаданные для поиска из источника данных. Индексатор SharePoint Online будет подключаться к сайту SharePoint Online и индексировать документы из одной или нескольких библиотек документов. Индексатор предоставляет следующие функциональные возможности:
+ Индексировать содержимое из одной или нескольких библиотек документов SharePoint Online.
+ Индексировать содержимое из библиотек документов SharePoint Online, которые находятся в том же клиенте, что и служба Когнитивный поиск Azure. Индексатор не будет работать с сайтами SharePoint, которые находятся в разных клиентах, чем служба Когнитивный поиск Azure. 
+ Индексатор будет поддерживать инкрементное индексирование, то есть он определит, какое содержимое в библиотеке документов изменилось и только индексирует обновленное содержимое в будущих запусках индексирования. Например, если в качестве индексатора изначально индексируется 5 документов PDF, то 1 обновляется, а затем индексатор выполняется снова, индексатор будет индексировать только 1 PDF-файл, который был обновлен.
+ Текстовые и нормализованные изображения будут извлечены по умолчанию из индексируемых документов. При необходимости можно добавить набор навыков в конвейер для дальнейшего уточнения содержимого. Дополнительные сведения о навыков см. в статье концепции набора [навыков в Azure когнитивный Поиск](cognitive-search-working-with-skillsets.md).

## <a name="supported-document-formats"></a>Поддерживаемые форматы документов

Индексатор Azure Когнитивный поиск SharePoint Online может извлекать текст из следующих форматов документов:

[!INCLUDE [search-document-data-sources](../../includes/search-blob-data-sources.md)]

## <a name="incremental-indexing-and-deletion-detection"></a>Добавочное индексирование и обнаружение удаления 
По умолчанию индексатор SharePoint Online поддерживает добавочное индексирование, то есть определяет, какое содержимое в библиотеке документов изменилось и индексирует только обновленное содержимое в будущих запусках индексирования. Например, если в качестве индексатора изначально индексируется 5 документов Word, то 1 обновляется, а индексатор снова запускается, индексатор будет повторно индексировать только один документ Word, который был обновлен.

Обнаружение удаления также поддерживается по умолчанию. Это означает, что при удалении документа из библиотеки документов SharePoint Online индексатор обнаружит удаление во время выполнения следующего индексатора и удалит документ из индекса.

## <a name="setting-up-sharepoint-online-indexing"></a>Настройка индексирования SharePoint Online
Чтобы настроить индексатор SharePoint Online, необходимо выполнить некоторые действия в портал Azure и некоторые действия с помощью REST API предварительной версии. Эта предварительная версия не поддерживается пакетом SDK.

### <a name="step-1-enable-system-assigned-managed-identity"></a>Шаг 1. Включение управляемого удостоверения, назначенного системой
Если управляемое системой удостоверение включено, Azure создает удостоверение для службы поиска, которое может использоваться индексатором.

![Включить управляемое удостоверение, назначенное системой](media/search-howto-index-sharepoint-online/enable-managed-identity.png "Включить управляемое удостоверение, назначенное системой")

Нажав **Сохранить**, вы увидите идентификатор объекта, назначенный вашей службе поиска.

![Управляемое удостоверение, назначенное системой](media/search-howto-index-sharepoint-online/system-assigned-managed-identity.png "Управляемое удостоверение, назначенное системой")

### <a name="step-2-create-an-aad-application"></a>Шаг 2. Создание приложения AAD
Индексатор SharePoint Online будет использовать это приложение AAD для проверки подлинности. 

1.  Перейдите на [портал Azure](https://portal.azure.com/).

1.  Откройте меню в левой части главной страницы и выберите **Azure Active Directory** затем выберите **Регистрация приложений**. Выберите **+ Создать регистрацию.**
    1.  Укажите имя приложения.
    2.  Выберите **один клиент**.
    3.  URI перенаправления не требуется.
    4.  Нажмите кнопку **Зарегистрировать**.

1.  В меню слева выберите **разрешения API** , а затем **добавьте разрешение**, а затем **Microsoft Graph** **делегированные разрешения**. Добавьте следующие разрешения API: 
    1.  **"Делегированные файлы. чтение. все"** 
    2.  **Делегирование — сайты. чтение. все** 
    3.  **Делегированный — пользователь. чтение**

    ![Делегированные разрешения API](media/search-howto-index-sharepoint-online/delegated-api-permissions.png "Делегированные разрешения API")

    Использование делегированных разрешений означает, что индексатор будет обращаться к сайту SharePoint в контексте пользователя. Поэтому при запуске индексатора он будет иметь доступ только к содержимому, к которому имеет доступ вошедший в систему пользователь. При создании индексатора или обновлении источника даты происходит вход пользователя. Шаг входа описан далее в этой статье.

1.  Перейдите на вкладку **Проверка подлинности** . Установите для параметра **Разрешить потокам общедоступного клиента** значение **Да** , а затем выберите **сохранить**.

1.  Выберите **+ Добавить платформу**, затем **мобильные и классические приложения**, затем установите флажок `https://login.microsoftonline.com/common/oauth2/nativeclient` , а затем **Настройте**.

    ![Конфигурация проверки подлинности приложения AAD](media/search-howto-index-sharepoint-online/aad-app-authentication-configuration.png "Конфигурация проверки подлинности приложения AAD")

1.  Предоставьте согласие администратора (требуется только для определенных клиентов).

    Некоторые клиенты заблокированы таким образом, что для этих делегированных разрешений API требуется согласие администратора. В этом случае перед созданием индексатора необходимо предоставить администратору согласие администратора для этого приложения AAD. 

    Так как не все клиенты имеют это требование, рекомендуется пропускать этот шаг и продолжить работу с инструкциями. Если при создании индексатора вам потребуется согласие администратора, проверка подлинности не будет выполнена, так как для подтверждения подлинности требуется администратор. В этом случае предоставьте согласие администратора клиента с помощью кнопки ниже.

    ![Предоставление согласия администратора для приложения AAD](media/search-howto-index-sharepoint-online/aad-app-grant-admin-consent.png "Предоставление согласия администратора для приложения AAD")

<a name="create-data-source"></a>

### <a name="step-3-create-data-source"></a>Шаг 3. Создание источника данных
> [!IMPORTANT] 
> Начиная с этого раздела, для выполнения оставшихся действий необходимо использовать предварительную версию REST API. Если вы не знакомы с REST API Azure Когнитивный поиск, рекомендуем ознакомиться с этим [кратким](search-get-started-rest.md)руководством.

Источник данных определяет, какие данные следует индексировать, какие учетные данные требуются для доступа к данным и какие политики нужны, чтобы эффективно выявлять изменения в данных (новые, измененные или удаленные строки). Источник данных могут использовать несколько индексаторов в одной службе поиска.

Для индексирования SharePoint источник данных должен иметь следующие обязательные свойства:
+ Свойство **name** — уникальное имя источника данных в службе поиска.
+ **тип** должен быть "SharePoint". Это зависит от регистра.
+ **учетные данные** предоставляют конечную точку SharePoint Online и идентификатор приложения AAD (клиент). Примером конечной точки SharePoint Online является `https://microsoft.sharepoint.com/teams/MySharePointSite` . Конечную точку SharePoint Online можно получить, перейдя на домашнюю страницу сайта SharePoint и скопировав URL-адрес из браузера.
+ **контейнер** указывает, какую библиотеку документов индексировать. Дополнительные сведения о создании контейнера можно найти в разделе [Управление документами, индексируемыми](#controlling-which-documents-are-indexed) в этом документе.

Создание источника данных

```http
POST https://[service name].search.windows.net/datasources?api-version=2020-06-30-Preview
Content-Type: application/json
api-key: [admin key]

{
    "name" : "sharepoint-datasource",
    "type" : "sharepoint",
    "credentials" : { "connectionString" : "SharePointOnlineEndpoint=[SharePoint Online site url];ApplicationId=[AAD App ID]" },
    "container" : { "name" : "defaultSiteLibrary", "query" : null }
}
```

### <a name="step-4-create-an-index"></a>Шаг 4. Создание индекса
Индекс задает поля в документе, атрибуты, и другие компоненты, которые определяют процедуру поиска.

Вот как создать индекс с полем содержимого, поддерживающим Поиск, для хранения текста, извлеченного из документов в библиотеке документов:

```http
POST https://[service name].search.windows.net/indexes?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
    "name" : "sharepoint-index",
    "fields": [
        { "name": "id", "type": "Edm.String", "key": true, "searchable": false },
        { "name": "metadata_spo_item_name", "type": "Edm.String", "key": false, "searchable": true, "filterable": false, "sortable": false, "facetable": false },
        { "name": "metadata_spo_item_path", "type": "Edm.String", "key": false, "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "metadata_spo_item_content_type", "type": "Edm.String", "key": false, "searchable": false, "filterable": true, "sortable": false, "facetable": true },
        { "name": "metadata_spo_item_last_modified", "type": "Edm.DateTimeOffset", "key": false, "searchable": false, "filterable": false, "sortable": true, "facetable": false },
        { "name": "metadata_spo_item_size", "type": "Edm.Int64", "key": false, "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "content", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false }
    ]
}

```

Дополнительные сведения см. в разделе [Создание индекса (REST API)](https://docs.microsoft.com/rest/api/searchservice/create-index).

### <a name="step-5-create-an-indexer"></a>Шаг 5. Создание индексатора
Индексатор соединяет источник данных с целевым индексом поиска и предоставляет расписание для автоматизации обновления данных. После создания индекса и источника данных можно приступать к созданию индексатора!

В этом разделе вам будет предложено войти в систему с учетными данными Организации, имеющими доступ к сайту SharePoint. По возможности рекомендуется создать новую учетную запись пользователя организации и дать новому пользователю точные разрешения, которые должен иметь индексатор.

Создание индексатора состоит из нескольких этапов.

1.  Отправьте запрос на попытку создания индексатора.

    ```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30-Preview
    Content-Type: application/json
    api-key: [admin key]
    
        {
          "name" : "sharepoint-indexer",
          "dataSourceName" : "sharepoint-datasource",
          "targetIndexName" : "sharepoint-index",
          "fieldMappings" : [
            { 
              "sourceFieldName" : "metadata_spo_site_library_item_id", 
              "targetFieldName" : "id", 
              "mappingFunction" : { 
                "name" : "base64Encode" 
              } 
            }
          ]
        }
    
    ```

1.  При первом создании индексатора произойдет сбой, и появится следующая ошибка. Перейдите по ссылке в сообщении об ошибке. Если вы не переходите по ссылке в течение 10 минут, срок действия кода истечет, и необходимо будет повторно создать [источник данных](#create-data-source).

    ```http
    {
        "error": {
            "code": "",
            "message": "Error with data source: To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code <CODE> to authenticate.  Please adjust your data source definition in order to proceed."
        }
    }
    ```

1.  Укажите код, указанный в сообщении об ошибке.

    ![Введите код устройства](media/search-howto-index-sharepoint-online/enter-device-code.png "Введите код устройства")

1.  Индексатор SharePoint будет получать доступ к содержимому SharePoint в качестве пользователя, выполнившего вход. Пользователь, который входит в систему на этом шаге, будет иметь пользователя, выполнившего вход. Таким образом, при входе с учетной записью пользователя, не имеющей доступа к документу в библиотеке документов, которую необходимо индексировать, индексатор не будет иметь доступ к этому документу.

    По возможности рекомендуется создать новую учетную запись пользователя и дать новому пользователю точные разрешения, которые должен иметь индексатор.

1.  Утвердите запрошенные разрешения.

    ![Утверждение разрешений API](media/search-howto-index-sharepoint-online/aad-app-approve-api-permissions.png "Утверждение разрешений API")

1.  Повторная отправка запроса на создание индексатора. На этот раз запрос должен быть выполнен. 

    ```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30-Preview
    Content-Type: application/json
    api-key: [admin key]
    
    {
        "name" : "sharepoint-indexer",
        "dataSourceName" : "sharepoint-datasource",
        "targetIndexName" : "sharepoint-index"
    }
    ```

### <a name="step-6-check-the-indexer-status"></a>Шаг 6. Проверка состояния индексатора
После создания индексатора можно проверить состояние индексатора, выполнив следующий запрос.

```http
GET https://[service name].search.windows.net/indexers/sharepoint-indexer/status?api-version=2020-06-30-Preview
Content-Type: application/json
api-key: [admin key]
```

Дополнительные сведения о состоянии индексатора можно найти здесь: [Получение состояния индексатора](https://docs.microsoft.com/rest/api/searchservice/get-indexer-status).

## <a name="updating-the-data-source"></a>Обновление источника данных
Если объект источника данных не обновляется, индексатор может выполняться по расписанию без взаимодействия с пользователем. Тем не менее при каждом обновлении объекта источника данных Azure Когнитивный поиск потребуется выполнить вход, чтобы индексатор выполнялся. Например, при изменении запроса к источнику данных потребуется выполнить вход еще раз с помощью `https://microsoft.com/devicelogin` и нового кода.

После обновления источника данных выполните следующие действия.

1.  Запуск индексатора вручную.

    ```http
    POST https://[service name].search.windows.net/indexers/sharepoint-indexer/run?api-version=2020-06-30-Preview  
    Content-Type: application/json
    api-key: [admin key]
    ```

    Дополнительные сведения о запросе на выполнение индексатора можно найти здесь: [Run индексатор](https://docs.microsoft.com/rest/api/searchservice/run-indexer).

1.  Проверьте состояние индексатора. Если последний запуск индексатора содержит ошибку, сообщающую о переходе на `https://microsoft.com/devicelogin` эту страницу, и укажите новый код. 

    ```http
    GET https://[service name].search.windows.net/indexers/sharepoint-indexer/status?api-version=2020-06-30-Preview
    Content-Type: application/json
    api-key: [admin key]
    ```

    Дополнительные сведения о состоянии индексатора можно найти здесь: [Получение состояния индексатора](https://docs.microsoft.com/rest/api/searchservice/get-indexer-status).

1.  Имя входа

1.  Вручную запустите индексатор и проверьте состояние индексатора. На этот раз запуск индексатора должен успешно запуститься.

## <a name="indexing-document-metadata"></a>Индексирование метаданных документа
Если вы настроили индексатор для индексирования метаданных документа, для индекса будут доступны следующие метаданные.

> [!NOTE]
> Пользовательские метаданные не включаются в текущую версию предварительной версии.

| Идентификатор | Type | Описание | 
| ------------- | -------------- | ----------- |
| metadata_spo_site_library_item_id | Edm.String | Сочетание ключа идентификатора сайта, идентификатора библиотеки и идентификатора элемента, уникально идентифицирующего элемент в библиотеке документов для сайта. |
| metadata_spo_site_id | Edm.String | Идентификатор сайта SharePoint Online. |
| metadata_spo_library_id | Edm.String | Идентификатор библиотеки документов. |
| metadata_spo_item_id | Edm.String | Идентификатор элемента (документа) в библиотеке. |
| metadata_spo_item_last_modified | Edm.DateTimeOffset | Дата и время последнего изменения элемента (в формате UTC). |
| metadata_spo_item_name | Edm.String | Имя элемента. |
| metadata_spo_item_size | Edm.Int64 | Размер элемента (в байтах). | 
| metadata_spo_item_content_type | Edm.String | Тип содержимого элемента. | 
| metadata_spo_item_extension | Edm.String | Расширение элемента. |
| metadata_spo_item_weburi | Edm.String | Универсальный код ресурса (URI) элемента. |
| metadata_spo_item_path | Edm.String | Сочетание родительского пути и имени элемента. | 

Индексатор SharePoint Online также поддерживает метаданные, относящиеся к каждому типу документа. Дополнительные сведения можно найти в [свойствах метаданных содержимого, используемых в когнитивный Поиск Azure](search-blob-metadata-properties.md).

<a name="controlling-which-documents-are-indexed"></a>

## <a name="controlling-which-documents-are-indexed"></a>Управление индексацией документов
Один индексатор SharePoint Online может индексировать содержимое из одной или нескольких библиотек документов. Используйте параметр *Container* при создании источника данных, чтобы указать библиотеки документов, которые необходимо индексировать. У *контейнера* источника данных есть два свойства: *имя* и *запрос*. 

### <a name="name"></a>Имя
Свойство *Name* является обязательным и должно иметь одно из трех значений:
+ *дефаултсителибрари*
    + Индексировать все содержимое из библиотеки документов по умолчанию для сайтов.
+   *аллсителибрариес*
    + Индексировать все содержимое из всех библиотек документов на сайте. При этом не выполняется индексация библиотек документов с дочернего сайта. В этом случае они могут быть указаны в *запросе* .
+   *усекуери*
    + Только содержимое индекса, определенное в *запросе*.

### <a name="query"></a>Запрос
Свойство *Query* состоит из пар "ключевое слово-значение". Ниже приведены ключевые слова, которые можно использовать. Это могут быть URL-адреса сайтов или URL-адреса библиотек документов.

> [!NOTE]
> Чтобы получить значение для определенного ключевого слова, мы рекомендуем открыть SharePoint Online в браузере, перейдя к библиотеке документов, которую вы пытаетесь включить или исключить, и скопировать URI из браузера. Это самый простой способ получить значение для использования с ключевым словом в запросе.

| Ключевое слово | Описание запроса | Пример |
| ------------- | -------------- | ----------- |
| null | Если значение равно null или пусто, индексируется либо библиотека документов по умолчанию, либо все библиотеки документов в зависимости от имени контейнера. | Индексировать все содержимое из библиотеки сайта по умолчанию: <br><br>  ``` "container" : { "name" : "defaultSiteLibrary", "query" : null } ``` |
| инклуделибрариесинсите | Индексировать содержимое из всех библиотек в определенном сайте в строке подключения. Они ограничены дочерними сайтами сайта <br><br> Значение *запроса* для этого ключевого слова должно быть универсальным кодом ресурса (URI) сайта или дочернего сайта. | Индексировать все содержимое из всех библиотек документов на личного сайта. <br><br> ``` "container" : { "name" : "useQuery", "query" : "includeLibrariesInSite=https://mycompany.sharepoint.com/mysite" } ``` |
| инклуделибрари | Индексировать содержимое из этой библиотеки. <br><br> Значение *запроса* для этого ключевого слова должно быть в одном из следующих форматов: <br><br> Пример 1: <br><br> *Инклуделибрари = [сайт или дочерний сайт]/[Библиотека документов]* <br><br> Пример 2. <br><br> URI, скопированный из браузера. | Индексировать все содержимое из Мидокументлибрари: <br><br> Пример 1: <br><br> ``` "container" : { "name" : "useQuery", "query" : "includeLibrary=https://mycompany.sharepoint.com/mysite/MyDocumentLibrary" } ``` <br><br> Пример 2. <br><br> ``` "container" : { "name" : "useQuery", "query" : "includeLibrary=https://mycompany.sharepoint.com/teams/mysite/MyDocumentLibrary/Forms/AllItems.aspx" } ``` |
| ексклуделибрари |  Не индексировать содержимое из этой библиотеки. <br><br> Значение *запроса* для этого ключевого слова должно быть в одном из следующих форматов: <br><br> Пример 1: <br><br> *Ексклуделибрари = [URI сайта или дочернего сайта]/[Библиотека документов]* <br><br> Пример 2. <br><br> URI, скопированный из браузера. | Индексировать все содержимое из всех моих библиотек, кроме Мидокументлибрари: <br><br> Пример 1: <br><br> ``` "container" : { "name" : "useQuery", "query" : "includeLibrariesInSite=https://mysite.sharepoint.com/subsite1; excludeLibrary=https://mysite.sharepoint.com/subsite1/MyDocumentLibrary" } ``` <br><br> Пример 2. <br><br> ``` "container" : { "name" : "useQuery", "query" : "includeLibrariesInSite=https://mycompany.sharepoint.com/teams/mysite; excludeLibrary=https://mycompany.sharepoint.com/teams/mysite/MyDocumentLibrary/Forms/AllItems.aspx" } ``` |

## <a name="index-by-file-type"></a>Индекс по типу файла
Вы можете контролировать, какие документы индексируются и что они пропускаются.

### <a name="include-documents-having-specific-file-extensions"></a>Включить документы, имеющие определенные расширения файлов
Индексировать можно только документы с указанными расширениями имен файлов с помощью `indexedFileNameExtensions` параметра конфигурации индексатора. Значение представлено строкой, содержащей разделенный запятыми список расширений файлов (с предшествующей точкой). Например, для индексации только. PDF и. Документы DOCX:

```http
PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2020-06-30-Preview
Content-Type: application/json
api-key: [admin key]

{
    ... other parts of indexer definition
    "parameters" : { "configuration" : { "indexedFileNameExtensions" : ".pdf,.docx" } }
}
```

### <a name="exclude-documents-having-specific-file-extensions"></a>Исключить документы с определенными расширениями файлов
С помощью параметра конфигурации можно исключить документы с определенными расширениями имен файлов из индексирования `excludedFileNameExtensions` . Значение представлено строкой, содержащей разделенный запятыми список расширений файлов (с предшествующей точкой). Например, чтобы индексировать все содержимое, за исключением тех, которые имеют значение. PNG и. Расширения JPEG:

```http
PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2020-06-30-Preview
Content-Type: application/json
api-key: [admin key]

{
    ... other parts of indexer definition
    "parameters" : { "configuration" : { "excludedFileNameExtensions" : ".png,.jpeg" } }
}
```

Если `indexedFileNameExtensions` существуют оба `excludedFileNameExtensions` параметра и, Azure когнитивный поиск сначала просматривает `indexedFileNameExtensions` , а затем по адресу `excludedFileNameExtensions` . Это означает, что если одно расширение файла присутствует в обоих списках, объект будет исключен из индексации.

## <a name="handling-errors"></a>Обработка ошибок
По умолчанию индексатор SharePoint Online останавливается сразу после обнаружения документа с неподдерживаемым типом содержимого (например, изображением). Чтобы пропустить определенные типы содержимого, можно воспользоваться параметром `excludedFileNameExtensions`. Однако может потребоваться индексировать документы, не зная все возможные типы содержимого заранее. Чтобы продолжить индексирование при обнаружении неподдерживаемого типа содержимого, задайте `failOnUnsupportedContentType` для параметра конфигурации значение false:

```http
PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2020-06-30-Preview
Content-Type: application/json
api-key: [admin key]

{
    ... other parts of indexer definition
    "parameters" : { "configuration" : { "failOnUnsupportedContentType" : false } }
}
```

Для некоторых документов Azure Когнитивный поиск не может определить тип содержимого или обработать документ, который является неподдерживаемым типом содержимого. Чтобы пропустить этот сбой, задайте для `failOnUnprocessableDocument` параметра конфигурации значение False:

```http
"parameters" : { "configuration" : { "failOnUnprocessableDocument" : false } }
```

Azure Когнитивный поиск ограничивает размер индексируемых документов. Эти ограничения описаны в статье [ограничения службы в Azure когнитивный Поиск](https://docs.microsoft.com/azure/search/search-limits-quotas-capacity). Документы с слишком размером по умолчанию обрабатываются как ошибки. Однако вы по-прежнему можете индексировать метаданные хранилища для документов с длинными размерами, если `indexStorageMetadataOnlyForOversizedDocuments` для параметра конфигурации задано значение true:

```http
"parameters" : { "configuration" : { "indexStorageMetadataOnlyForOversizedDocuments" : true } }
```

Можно также продолжить индексирование при возникновении ошибок в любой точке обработки, во время синтаксического анализа документов или при добавлении документов в индекс. Чтобы пропустить определенное количество ошибок, задайте для параметров конфигурации `maxFailedItems` и `maxFailedItemsPerBatch` нужные значения. Пример:

```http
{
    ... other parts of indexer definition
    "parameters" : { "maxFailedItems" : 10, "maxFailedItemsPerBatch" : 10 }
}
```

## <a name="see-also"></a>См. также раздел
+ [Indexers in Azure Cognitive Search](search-indexer-overview.md) (Индексаторы в службе "Когнитивный поиск Azure")
+ [Свойства метаданных содержимого, используемые в Когнитивный поиск Azure](search-blob-metadata-properties.md)
