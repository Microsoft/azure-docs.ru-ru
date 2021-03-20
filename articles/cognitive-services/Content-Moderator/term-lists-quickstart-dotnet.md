---
title: Проверка текста с помощью списка пользовательских терминов на C# в Content Moderator
titleSuffix: Azure Cognitive Services
description: Узнайте, как модерировать текст с помощью настраиваемых списков терминов, используя пакет SDK Azure Content Moderator для C#.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 10/24/2019
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: 93d90232fb530a6c14c40558fc6a9974a1da42de
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92900918"
---
# <a name="check-text-against-a-custom-term-list-in-c"></a>Проверка текста с помощью списка пользовательских терминов на C#

Стандартный глобальный список терминов в Azure Content Moderator достаточен для большинства задач модерации контента. Тем не менее может потребоваться проверка специальных терминов, относящихся к вашей организации. Например, может потребоваться добавить теги с названиями конкурентов для более тщательной проверки. 

С помощью [пакета SDK Content Moderator для .NET](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) можно создавать настраиваемые списки терминов для использования с API-интерфейсом контроля текста.

В этой статье содержатся сведения и примеры кода, которые помогут приступить к работе с пакетом SDK Content Moderator для .NET в следующих целях:
- Создание списка
- добавление терминов в список;
- проверка терминов по списку;
- удаление терминов из списка;
- удаление списка;
- изменение сведений о списке;
- обновление индекса для того, чтобы изменения в списке были учтены при новой проверке.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/), прежде чем начинать работу. 

## <a name="sign-up-for-content-moderator-services"></a>Регистрация в службах Content Moderator

Прежде чем использовать службы Content Moderator с помощью REST API или пакета SDK, необходимо получить ключ подписки. Подпишитесь на службу Content Moderator на [портале Azure](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesContentModerator), чтобы получить его.

## <a name="create-your-visual-studio-project"></a>Создание проекта Visual Studio

1. Добавьте в свое решение новый проект **Консольное приложение (.NET Framework)**.

1. Присвойте проекту имя **TermLists**. Выберите этот проект в качестве единственного запускаемого проекта для решения.

### <a name="install-required-packages"></a>Установка необходимых пакетов

Установите следующие пакеты NuGet для проекта TermLists:

- Microsoft.Azure.CognitiveServices.ContentModerator;
- Microsoft.Rest.ClientRuntime
- Microsoft.Rest.ClientRuntime.Azure
- Newtonsoft.Json.

### <a name="update-the-programs-using-statements"></a>Обновление инструкций using в программе

Добавьте следующие операторы `using`.

```csharp
using Microsoft.Azure.CognitiveServices.ContentModerator;
using Microsoft.Azure.CognitiveServices.ContentModerator.Models;
using Newtonsoft.Json;
using System;
using System.Collections.Generic;
using System.IO;
using System.Threading;
```

### <a name="create-the-content-moderator-client"></a>Создание клиента Content Moderator

Добавьте следующий фрагмент кода, чтобы создать клиент Content Moderator для своей подписки. Обновите `AzureEndpoint` поля и, `CMSubscriptionKey` указав значения URL-адреса конечной точки и ключа подписки. Их можно найти на вкладке **Быстрый запуск** ресурса в портал Azure.

```csharp
/// <summary>
/// Wraps the creation and configuration of a Content Moderator client.
/// </summary>
/// <remarks>This class library contains insecure code. If you adapt this 
/// code for use in production, use a secure method of storing and using
/// your Content Moderator subscription key.</remarks>
public static class Clients
{
    /// <summary>
    /// The base URL fragment for Content Moderator calls.
    /// </summary>
    private static readonly string AzureEndpoint = "YOUR ENDPOINT URL";

    /// <summary>
    /// Your Content Moderator subscription key.
    /// </summary>
    private static readonly string CMSubscriptionKey = "YOUR API KEY";

    /// <summary>
    /// Returns a new Content Moderator client for your subscription.
    /// </summary>
    /// <returns>The new client.</returns>
    /// <remarks>The <see cref="ContentModeratorClient"/> is disposable.
    /// When you have finished using the client,
    /// you should dispose of it either directly or indirectly. </remarks>
    public static ContentModeratorClient NewClient()
    {
        // Create and initialize an instance of the Content Moderator API wrapper.
        ContentModeratorClient client = new ContentModeratorClient(new ApiKeyServiceClientCredentials(CMSubscriptionKey));

        client.Endpoint = AzureEndpoint;
        return client;
    }
}
```

### <a name="add-private-properties"></a>Добавление частных свойств

Добавьте следующие частные свойства в пространство имен TermLists в классе Program.

```csharp
/// <summary>
/// The language of the terms in the term lists.
/// </summary>
private const string lang = "eng";

/// <summary>
/// The minimum amount of time, in milliseconds, to wait between calls
/// to the Content Moderator APIs.
/// </summary>
private const int throttleRate = 3000;

/// <summary>
/// The number of minutes to delay after updating the search index before
/// performing image match operations against the list.
/// </summary>
private const double latencyDelay = 0.5;
```

## <a name="create-a-term-list"></a>Создание списка терминов

Создайте список терминов с помощью **ContentModeratorClient.ListManagementTermLists.Create**. Первый **создаваемый** параметр — это строка, обозначающая тип MIME, которая должна иметь значение "application/json". Дополнительные сведения см. в [справочнике по API](https://westus2.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f67f). Второй параметр — объект **Body**, содержащий имя и описание нового списка терминов.

> [!NOTE]
> Существует максимальное ограничение в **5 списков терминов**, каждый из которых может содержать **не более 10 000 терминов**.

Добавьте следующее определение метода в пространство имен TermLists в классе Program.

> [!NOTE]
> Для ключа службы Content Moderator установлен лимит количества запросов в секунду. При его превышении пакет SDK создаст исключение с кодом ошибки 429. Для ключа уровня "Бесплатный" предусмотрен лимит в один запрос в секунду.

```csharp
/// <summary>
/// Creates a new term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <returns>The term list ID.</returns>
static string CreateTermList (ContentModeratorClient client)
{
    Console.WriteLine("Creating term list.");

    Body body = new Body("Term list name", "Term list description");
    TermList list = client.ListManagementTermLists.Create("application/json", body);
    if (false == list.Id.HasValue)
    {
        throw new Exception("TermList.Id value missing.");
    }
    else
    {
        string list_id = list.Id.Value.ToString();
        Console.WriteLine("Term list created. ID: {0}.", list_id);
        Thread.Sleep(throttleRate);
        return list_id;
    }
}
```

## <a name="update-term-list-name-and-description"></a>Обновление имени и описания списка терминов

Обновить сведения о списке терминов можно с помощью **ContentModeratorClient.ListManagementTermLists.Update**. Первый **обновляемый** параметр — идентификатор списка терминов. Второй параметр — это тип MIME, который должен иметь значение "application/json". Дополнительные сведения см. в [справочнике по API](https://westus2.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f685). Третий параметр — объект **Body**, который содержит новые имя и описание.

Добавьте следующее определение метода в пространство имен TermLists в классе Program.

```csharp
/// <summary>
/// Update the information for the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list to update.</param>
/// <param name="name">The new name for the term list.</param>
/// <param name="description">The new description for the term list.</param>
static void UpdateTermList (ContentModeratorClient client, string list_id, string name = null, string description = null)
{
    Console.WriteLine("Updating information for term list with ID {0}.", list_id);
    Body body = new Body(name, description);
    client.ListManagementTermLists.Update(list_id, "application/json", body);
    Thread.Sleep(throttleRate);
}
```

## <a name="add-a-term-to-a-term-list"></a>Добавление термина в список терминов

Добавьте следующее определение метода в пространство имен TermLists в классе Program.

```csharp
/// <summary>
/// Add a term to the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list to update.</param>
/// <param name="term">The term to add to the term list.</param>
static void AddTerm (ContentModeratorClient client, string list_id, string term)
{
    Console.WriteLine("Adding term \"{0}\" to term list with ID {1}.", term, list_id);
    client.ListManagementTerm.AddTerm(list_id, term, lang);
    Thread.Sleep(throttleRate);
}
```

## <a name="get-all-terms-in-a-term-list"></a>Получение всех терминов в списке терминов

Добавьте следующее определение метода в пространство имен TermLists в классе Program.

```csharp
/// <summary>
/// Get all terms in the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list from which to get all terms.</param>
static void GetAllTerms(ContentModeratorClient client, string list_id)
{
    Console.WriteLine("Getting terms in term list with ID {0}.", list_id);
    Terms terms = client.ListManagementTerm.GetAllTerms(list_id, lang);
    TermsData data = terms.Data;
    foreach (TermsInList term in data.Terms)
    {
        Console.WriteLine(term.Term);
    }
    Thread.Sleep(throttleRate);
}
```

## <a name="add-code-to-refresh-the-search-index"></a>Добавление кода для обновления индекса поиска

После внесения изменений в список терминов следует обновить его индекс поиска, чтобы эти изменения учитывались при следующем использовании списка терминов для проверки текста. Точно так же поисковая система на компьютере (если она включена) или поисковая система в Интернете постоянно обновляет свой индекс, чтобы добавлять новые файлы или страницы.

Обновить индекс поиска списка терминов можно с помощью **ContentModeratorClient.ListManagementTermLists.RefreshIndexMethod**.

Добавьте следующее определение метода в пространство имен TermLists в классе Program.

```csharp
/// <summary>
/// Refresh the search index for the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list to refresh.</param>
static void RefreshSearchIndex (ContentModeratorClient client, string list_id)
{
    Console.WriteLine("Refreshing search index for term list with ID {0}.", list_id);
    client.ListManagementTermLists.RefreshIndexMethod(list_id, lang);
    Thread.Sleep((int)(latencyDelay * 60 * 1000));
}
```

## <a name="screen-text-using-a-term-list"></a>Проверка текста с помощью списка терминов

Для проверки текста с помощью списка терминов используется метод **ContentModeratorClient.TextModeration.ScreenText**, который принимает следующие параметры.

- Язык терминов в список терминов.
- Тип MIME: "text/html", "text/xml", "text/markdown" или "text/plain".
- Текст для проверки.
- Логическое значение. Задайте для этого поля значение **true**, чтобы выполнить автозамену в тексте перед проверкой.
- Логическое значение. Задайте для этого поля **значение true** , чтобы определить персональные данные в тексте.
- Идентификатор списка терминов.

Дополнительные сведения см. в [справочнике по API](https://westus2.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66f).

**ScreenText** возвращает объект **Screen**, содержащий свойство **Terms**, которое содержит все термины, обнаруженные Content Moderator при проверке. Обратите внимание на то, что если Content Moderator не обнаружил термины при проверке, то свойство **Terms** имеет значение **NULL**.

Добавьте следующее определение метода в пространство имен TermLists в классе Program.

```csharp
/// <summary>
/// Screen the indicated text for terms in the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list to use to screen the text.</param>
/// <param name="text">The text to screen.</param>
static void ScreenText (ContentModeratorClient client, string list_id, string text)
{
    Console.WriteLine("Screening text: \"{0}\" using term list with ID {1}.", text, list_id);
    Screen screen = client.TextModeration.ScreenText(lang, "text/plain", text, false, false, list_id);
    if (null == screen.Terms)
    {
        Console.WriteLine("No terms from the term list were detected in the text.");
    }
    else
    {
        foreach (DetectedTerms term in screen.Terms)
        {
            Console.WriteLine(String.Format("Found term: \"{0}\" from list ID {1} at index {2}.", term.Term, term.ListId, term.Index));
        }
    }
    Thread.Sleep(throttleRate);
}
```

## <a name="delete-terms-and-lists"></a>Удаление терминов и списков

Удалить термин или список достаточно просто. С помощью пакета SDK можно выполнить следующие задачи:

- удаление термина (**ContentModeratorClient.ListManagementTerm.DeleteTerm**);
- удаление всех терминов в списке без удаления самого списка (**ContentModeratorClient.ListManagementTerm.DeleteAllTerms**);
- удаление списка и всего его содержимого (**ContentModeratorClient.ListManagementTermLists.Delete**).

### <a name="delete-a-term"></a>Удаление термина

Добавьте следующее определение метода в пространство имен TermLists в классе Program.

```csharp
/// <summary>
/// Delete a term from the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list from which to delete the term.</param>
/// <param name="term">The term to delete.</param>
static void DeleteTerm (ContentModeratorClient client, string list_id, string term)
{
    Console.WriteLine("Removed term \"{0}\" from term list with ID {1}.", term, list_id);
    client.ListManagementTerm.DeleteTerm(list_id, term, lang);
    Thread.Sleep(throttleRate);
}
```

### <a name="delete-all-terms-in-a-term-list"></a>Удаление всех терминов в списке терминов

Добавьте следующее определение метода в пространство имен TermLists в классе Program.

```csharp
/// <summary>
/// Delete all terms from the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list from which to delete all terms.</param>
static void DeleteAllTerms (ContentModeratorClient client, string list_id)
{
    Console.WriteLine("Removing all terms from term list with ID {0}.", list_id);
    client.ListManagementTerm.DeleteAllTerms(list_id, lang);
    Thread.Sleep(throttleRate);
}
```

### <a name="delete-a-term-list"></a>Удаление списка терминов

Добавьте следующее определение метода в пространство имен TermLists в классе Program.

```csharp
/// <summary>
/// Delete the indicated term list.
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="list_id">The ID of the term list to delete.</param>
static void DeleteTermList (ContentModeratorClient client, string list_id)
{
    Console.WriteLine("Deleting term list with ID {0}.", list_id);
    client.ListManagementTermLists.Delete(list_id);
    Thread.Sleep(throttleRate);
}
```

## <a name="compose-the-main-method"></a>Создание метода Main

Добавьте определение метода **Main** в пространство имен **TermLists** в классе **Program**. Наконец, закройте класс **Program** и пространство имен **TermLists**.

```csharp
static void Main(string[] args)
{
    using (var client = Clients.NewClient())
    {
        string list_id = CreateTermList(client);

        UpdateTermList(client, list_id, "name", "description");
        AddTerm(client, list_id, "term1");
        AddTerm(client, list_id, "term2");

        GetAllTerms(client, list_id);

        // Always remember to refresh the search index of your list
        RefreshSearchIndex(client, list_id);

        string text = "This text contains the terms \"term1\" and \"term2\".";
        ScreenText(client, list_id, text);

        DeleteTerm(client, list_id, "term1");

        // Always remember to refresh the search index of your list
        RefreshSearchIndex(client, list_id);

        text = "This text contains the terms \"term1\" and \"term2\".";
        ScreenText(client, list_id, text);

        DeleteAllTerms(client, list_id);
        DeleteTermList(client, list_id);

        Console.WriteLine("Press ENTER to close the application.");
        Console.ReadLine();
    }
}
```

## <a name="run-the-application-to-see-the-output"></a>Запустите приложение, чтобы просмотреть выходные данные.

Выходные данные консоли будут выглядеть следующим образом:

```console
Creating term list.
Term list created. ID: 252.
Updating information for term list with ID 252.

Adding term "term1" to term list with ID 252.
Adding term "term2" to term list with ID 252.

Getting terms in term list with ID 252.
term1
term2

Refreshing search index for term list with ID 252.

Screening text: "This text contains the terms "term1" and "term2"." using term list with ID 252.
Found term: "term1" from list ID 252 at index 32.
Found term: "term2" from list ID 252 at index 46.

Removed term "term1" from term list with ID 252.

Refreshing search index for term list with ID 252.

Screening text: "This text contains the terms "term1" and "term2"." using term list with ID 252.
Found term: "term2" from list ID 252 at index 46.

Removing all terms from term list with ID 252.
Deleting term list with ID 252.
Press ENTER to close the application.
```

## <a name="next-steps"></a>Дальнейшие действия

Получите [пакет SDK Content Moderator для .NET](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) и [решение Visual Studio](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/ContentModerator) для выполнения инструкций из этого и других кратких руководств по Content Moderator для .NET и приступите к интеграции.
