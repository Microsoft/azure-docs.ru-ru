---
title: Фильтры безопасности для усечения результатов с помощью Active Directory
titleSuffix: Azure Cognitive Search
description: Узнайте, как реализовать привилегии безопасности на уровне документа для результатов поиска Когнитивный поиск Azure с помощью фильтров безопасности и удостоверений Azure Active Directory (AD).
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 12/16/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 5788585b2365b12a90a508e5a972b61f73e48c15
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97629516"
---
# <a name="security-filters-for-trimming-azure-cognitive-search-results-using-active-directory-identities"></a>Фильтры безопасности для усечения результатов Когнитивный поиск Azure с использованием удостоверений Active Directory

В этой статье показано, как использовать удостоверения безопасности Azure Active Directory (AD) вместе с фильтрами в Когнитивный поиск Azure, чтобы обрезать результаты поиска на основе членства в группе пользователей.

В этой статье рассматриваются следующие задачи:
> [!div class="checklist"]
> - Создание групп и пользователей Azure AD
> - связывание пользователя с созданной группой;
> - кэширование новых групп;
> - индексация документов со связанными группами;
> - отправка запроса на поиск с помощью фильтра идентификаторов групп.
> 
> [!NOTE]
> Примеры фрагментов когда в этой статье написаны языке C#. Полный исходный код можно найти на сайте [GitHub](https://github.com/Azure-Samples/search-dotnet-getting-started). 

## <a name="prerequisites"></a>Предварительные условия

Индекс в Когнитивный поиске Azure должен иметь [поле безопасности](search-security-trimming-for-azure-search.md) для хранения списка удостоверений групп, имеющих доступ на чтение документа. Этот вариант использования предполагает прямое соответствие между защищаемым элементом (например, заявление на вступление в колледж конкретного человека) и полем безопасности, в котором указывается, кто имеет доступ к этому элементу (персонал приемной комиссии).

Необходимо иметь разрешения администратора Azure AD, необходимые в этом пошаговом руководстве для создания пользователей, групп и ассоциаций. 

Приложение также должно быть зарегистрировано в Azure AD как приложение с несколькими клиентами, как описано в следующей процедуре.

### <a name="register-your-application-with-azure-active-directory"></a>Регистрация приложения в Azure Active Directory

Этот шаг интегрирует приложение с Azure AD в целях принятия входов в учетные записи пользователей и групп. Если вы не являетесь администратором клиента в вашей организации, вам может потребоваться [создать новый клиент](../active-directory/develop/quickstart-create-new-tenant.md) , чтобы выполнить следующие действия.

1. В [портал Azure](https://portal.azure.com)найдите ресурс Azure Active Directory для своей подписки.

1. Слева в разделе **Управление** выберите **Регистрация приложений**, а затем щелкните **Новая регистрация**.

1. Присвойте регистрационное имя, возможно, имя, похожее на имя приложения поиска. Выберите **Зарегистрировать**.

1. После создания регистрации приложения скопируйте идентификатор приложения. Необходимо предоставить эту строку приложению.

   При пошаговом выполнении [дотнесовтосекурититримминг](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToEncryptionUsingCMK)вставьте это значение в файл **app.config** .

   Повторите процедуру для идентификатора клиента.

   :::image type="content" source="media/search-manage-encryption-keys/cmk-application-id.png" alt-text="Идентификатор приложения в разделе &quot;основные компоненты&quot;":::

1. В левой части экрана выберите **разрешения API** и щелкните **Добавить разрешение**. 

1. Выберите **Microsoft Graph** а затем **делегированные разрешения**.

1. Найдите и добавьте следующие делегированные разрешения:

   - **Directory.ReadWrite.All**;
   - **Group.ReadWrite.All**
   - **User.ReadWrite.All**.

Microsoft Graph предоставляет API, который обеспечивает программный доступ к Azure AD через REST API. Пример кода в этом пошаговом руководстве использует разрешения для вызова API Microsoft Graph, чтобы создавать группы, пользователей и ассоциации. API также используются для кэширования идентификаторов групп для более быстрой фильтрации.

## <a name="create-users-and-groups"></a>Создание пользователей и групп

Если вы добавляете Поиск в установленное приложение, у вас могут быть существующие идентификаторы пользователей и групп в Azure AD. В этом случае можно пропустить следующие три шага. 

Однако, если у вас нет имеющихся пользователей, можно использовать API Microsoft Graph для создания субъектов безопасности. В следующих фрагментах кода показано, как создавать идентификаторы, которые становятся значениями данных для поля безопасности в индексе Azure Когнитивный поиск. В нашем гипотетическом приложении для приема в колледж это будут идентификаторы безопасности для сотрудников приемной комиссии.

Назначение пользователей и членство в группе может быть очень гибким, особенно в крупных организациях. Код, который создает удостоверения пользователей и групп, должен выполняться достаточно часто для учета изменений в членстве организации. Аналогично, для индекса Когнитивный поиск Azure требуется аналогичное расписание обновления, отражающее текущее состояние разрешенных пользователей и ресурсов.

### <a name="step-1-create-group"></a>Шаг 1. [Создание группы](/graph/api/group-post-groups) 

```csharp
private static Dictionary<Group, List<User>> CreateGroupsWithUsers(string tenant)
{
    Group group = new Group()
    {
        DisplayName = "My First Prog Group",
        SecurityEnabled = true,
        MailEnabled = false,
        MailNickname = "group1"
    };
```

### <a name="step-2-create-user"></a>Шаг 2. [Создание пользователя](/graph/api/user-post-users)

```csharp
User user1 = new User()
{
    GivenName = "First User",
    Surname = "User1",
    MailNickname = "User1",
    DisplayName = "First User",
    UserPrincipalName = String.Format("user1@{0}", tenant),
    PasswordProfile = new PasswordProfile() { Password = "********" },
    AccountEnabled = true
};
```

### <a name="step-3-associate-user-and-group"></a>Шаг 3. Связывание пользователя и групп

```csharp
List<User> users = new List<User>() { user1, user2 };
Dictionary<Group, List<User>> groups = new Dictionary<Group, List<User>>() { { group, users } };
```

### <a name="step-4-cache-the-groups-identifiers"></a>Шаг 4. Кэширование идентификаторов групп

Кроме того, чтобы сократить задержку в сети, можно кэшировать сопоставления групп пользователей, чтобы при выполнении запроса поиска группы возвращались из кэша, сохранив при этом циклическое обращение к Azure AD. Вы можете использовать [API пакетной службы Azure AD](/graph/json-batching) для отправки одного HTTP-запроса с несколькими пользователями и создания кэша.

Microsoft Graph может обрабатывать большое число запросов. Если возникает огромное количество запросов, Microsoft Graph отклоняет запрос с кодом состояния HTTP 429. Дополнительные сведения см. в статье [Руководство по регулированию Microsoft Graph](/graph/throttling).

## <a name="index-document-with-their-permitted-groups"></a>Индексирование документа с разрешенными группами

Операции запроса в Azure Когнитивный поиск выполняются через индекс Когнитивный поиск Azure. На этом шаге операция индексирования импортирует данные с возможностью поиска в индекс, включая идентификаторы, используемые в качестве фильтров безопасности. 

Когнитивный поиск Azure не проверяет подлинность удостоверений пользователей или предоставляет логику для определения содержимого, которое у пользователя есть разрешение на просмотр. Вариант использования фильтров безопасности предполагает обеспечение связи между конфиденциальным документом и идентификатором группы, имеющим доступ к этому документу, импортированным без изменений в индекс поиска. 

В гипотетическом примере тело запроса размещения в индексе Azure Когнитивный поиск будет включать в себя очерк или запись о соискательх, а также идентификатор группы, имеющий разрешение на просмотр этого содержимого. 

В универсальном примере, используемом в примере кода для этого пошагового руководства, действие индекса может выглядеть следующим образом:

```csharp
private static void IndexDocuments(string indexName, List<string> groups)
{
    IndexDocumentsBatch<SecuredFiles> batch = IndexDocumentsBatch.Create(
        IndexDocumentsAction.Upload(
            new SecuredFiles()
            {
                FileId = "1",
                Name = "secured_file_a",
                GroupIds = new[] { groups[0] }
            }),
              ...
            };

IndexDocumentsResult result = searchClient.IndexDocuments(batch);
```

## <a name="issue-a-search-request"></a>Отправка поискового запроса

В целях применения фильтров безопасности значения в поле безопасности в индексе являются статическими значениями, используемыми для включения или исключения документов в результатах поиска. Например, если идентификатором группы для приемов является "A11B22C33D44-E55F66G77-H88I99JKK", все документы в индексе Azure Когнитивный поиск, в которых в списке безопасности будут включены (или исключены), включаются в результаты поиска, отправляемые запрашивающей стороне.

Чтобы фильтровать документы, возвращаемые в результатах поиска, по группам для пользователя, выполняющего запрос, сделайте следующее.

### <a name="step-1-retrieve-users-group-identifiers"></a>Шаг 1. Получение идентификаторов группы пользователя

Если группы пользователя еще не кэшированы или истек срок действия кэша, выдайте запрос [группам](/graph/api/directoryobject-getmembergroups) .

```csharp
private static async void RefreshCache(IEnumerable<User> users)
{
    HttpClient client = new HttpClient();
    var userGroups = await _microsoftGraphHelper.GetGroupsForUsers(client, users);
    _groupsCache = new ConcurrentDictionary<string, List<string>>(userGroups);
}
```

### <a name="step-2-compose-the-search-request"></a>Шаг 2. Составление поискового запроса

Предположим, что у вас есть членство в группах пользователей. Это позволяет отправить поисковой запрос с соответствующими значениями фильтра.

```csharp
private static void SearchQueryWithFilter(string user)
{
    // Using the filter below, the search result will contain all documents that their GroupIds field   
    // contain any one of the Ids in the groups list
    string filter = String.Format("groupIds/any(p:search.in(p, '{0}'))", string.Join(",", String.Join(",", _groupsCache[user])));
    SearchOptions searchOptions =
        new SearchOptions()
        {
            Filter = filter
        };
    searchOptions.Select.Add("name");

    SearchResults<SecuredFiles> results = searchClient.Search<SecuredFiles>("*", searchOptions);

    Console.WriteLine("Results for groups '{0}' : {1}", _groupsCache[user], results.GetResults().Select(r => r.Document.Name));
}
```

### <a name="step-3-handle-the-results"></a>Шаг 3. Обработка результатов

Ответ включает отфильтрованный список, состоящий из тех документов, для которых у пользователя есть разрешение на просмотр. В зависимости от структуры страницы результатов поиска можно включить визуальные подсказки для отражения отфильтрованного набора результатов.

## <a name="next-steps"></a>Дальнейшие действия

В этом пошаговом руководстве вы узнали, как использовать входы в Azure AD для фильтрации документов в Azure Когнитивный поиск результаты, обрезая результаты документов, которые не соответствуют фильтру, указанному в запросе. Для альтернативного шаблона, который может быть более простым, или для повторного посещения других функций безопасности, см. следующие ссылки.

- [Фильтры безопасности для усечения результатов](search-security-trimming-for-azure-search.md)
- [Безопасность в Azure Когнитивный поиск](search-security-overview.md)