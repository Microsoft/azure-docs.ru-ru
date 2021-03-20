---
title: Обновление до пакета SDK .NET для поиска Azure версии 3
titleSuffix: Azure Cognitive Search
description: Перенос кода в пакет SDK .NET для службы "Поиск Azure" версии 3 с предыдущих версий. Узнайте о новых возможностях и необходимых изменениях кода.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 11/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 0e9f4602582a8c24f74d6603b732e7b91d97fa8d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89002697"
---
# <a name="upgrade-to-azure-search-net-sdk-version-3"></a>Обновление до пакета SDK .NET для поиска Azure версии 3

<!--- DETAILS in the word doc
cosmosdb
NER v1 skill 
Indexer execution result errors no longer have status
the data source API will no longer return in the response of any REST operation, the connection string specified by the user.
--->

Если вы используете версию 2.0-preview [пакета SDK службы поиска Azure для .NET](/dotnet/api/overview/azure/search) или более раннюю, то эта статья поможет вам обновить приложение для использования версии 3.

Более общее пошаговое руководство по пакету SDK, включая примеры, см. в разделе [Использование службы поиска Azure в приложении .NET](search-howto-dotnet-sdk.md).

Версия 3 пакета SDK службы поиска Azure для .NET содержит некоторые отличия от более ранних версий. Эти отличия в основном незначительные, поэтому изменение кода потребует минимальных усилий. В разделе [Действия по обновлению](#UpgradeSteps) вы найдете инструкции о том, как изменить код для использования новой версии пакета SDK.

> [!NOTE]
> Если вы используете версию 1.0.2-preview или более раннюю, то сначала следует установить версию 1.1, и только затем обновить ее до версии 3. Инструкции приведены в разделе [Обновление пакета SDK службы поиска Azure для .NET до версии 1.1](search-dotnet-sdk-migration-version-1.md).
>
> Экземпляр службы поиска Azure поддерживает несколько версий REST API, включая последнюю. Можно продолжать использовать версию, которая больше не является последней, но рекомендуется выполнить перенос кода, чтобы использовать последнюю версию. При использовании REST API необходимо указывать версию API в каждом запросе с помощью параметра api-version. При использовании пакета SDK для .NET версия пакета SDK, которую вы используете, определяет соответствующую версию REST API. Если вы используете устаревшую версию пакета SDK, можно продолжать выполнять этот код без изменений, даже если служба обновлена для поддержки более новой версии API.

<a name="WhatsNew"></a>

## <a name="whats-new-in-version-3"></a>Новые возможности в версии 3
Версия 3 пакета SDK службы поиска Azure для .NET предназначена для последней общедоступной версии интерфейса REST API службы поиска Azure, в частности версии 2016-09-01. Это дает возможность использовать различные новые функции службы поиска Azure из приложений для .NET, включая следующие:

* [пользовательские анализаторы;](index-add-custom-analyzers.md)
* поддержка индексатора [хранилища BLOB-объектов Azure](search-howto-indexing-azure-blob-storage.md) и [хранилища таблиц Azure](search-howto-indexing-azure-tables.md);
* настройка индексатора посредством [сопоставления полей](search-indexer-field-mappings.md)
* поддержка тегов eTag, обеспечивающих безопасное одновременное обновление определений индекса, индексаторов и источников данных;
* поддержка декларативного создания определений полей индекса через дополнение класса модели и с помощью нового класса `FieldBuilder`;
* поддержка .NET Core и .NET Portable Profile 111.

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>Действия по обновлению
Прежде всего обновите справочник NuGet для `Microsoft.Azure.Search` , воспользовавшись консолью диспетчера пакетов NuGet или щелкнув правой кнопкой мыши ссылки проекта и выбрав "Управление пакетами NuGet" в Visual Studio.

После того как NuGet загрузит новые пакеты и их зависимости, перестройте проект. Сборка может быть выполнена успешно — в зависимости от того, как структурирован ваш код. Если она выполнена, то теперь все готово.

В противном случае отобразится сообщение об ошибке сборки следующего вида.

```output
Program.cs(31,45,31,86): error CS0266: Cannot implicitly convert type 'Microsoft.Azure.Search.ISearchIndexClient' to 'Microsoft.Azure.Search.SearchIndexClient'. An explicit conversion exists (are you missing a cast?)
```

Далее необходимо устранить эту ошибку сборки. Сведения о причинах ошибки и способах ее устранения см. в разделе [Критические изменения в версии 3](#ListOfChanges).

Могут появиться дополнительные предупреждения о сборке, связанные с устаревшими методами или свойствами. В предупреждениях будет указано, что следует использовать вместо устаревшей функции. Например, если приложение использует `IndexingParameters.Base64EncodeKeys`, то отобразится предупреждение `"This property is obsolete. Please create a field mapping using 'FieldMapping.Base64Encode' instead."`.

После устранения ошибки сборки при необходимости можно внести изменения в приложение, чтобы воспользоваться преимуществами новых функциональных возможностей. Новые возможности в пакете SDK описаны в разделе [Новые возможности в версии 3](#WhatsNew).

<a name="ListOfChanges"></a>

## <a name="breaking-changes-in-version-3"></a>Критические изменения в версии 3
В версии 3 содержится небольшое количество критических изменений, для которых может потребоваться изменение кода и выполнение повторной сборки приложения.

### <a name="indexesgetclient-return-type"></a>Тип возвращаемого значения Indexes.GetClient
Метод `Indexes.GetClient` имеет новый тип возвращаемого значения. Раньше он возвращал `SearchIndexClient`, но в версии 2.0-preview значение было изменено на `ISearchIndexClient`, и это изменение переносится в версию 3. Это необходимо для поддержки клиентов, которые желают макетировать метод `GetClient` для модульных тестов, возвращая реализацию макета `ISearchIndexClient`.

#### <a name="example"></a>Пример
Если код выглядит следующим образом:

```csharp
SearchIndexClient indexClient = serviceClient.Indexes.GetClient("hotels");
```

Можно изменить его на следующий для устранения ошибок сборки:

```csharp
ISearchIndexClient indexClient = serviceClient.Indexes.GetClient("hotels");
```

### <a name="analyzername-datatype-and-others-are-no-longer-implicitly-convertible-to-strings"></a>Типы AnalyzerName, DataType и другие больше не преобразуются в строки неявным образом
В пакете SDK службы поиска Azure для .NET существует много типов, которые являются производными от `ExtensibleEnum`. Ранее все они неявно преобразовывались в тип `string`. Однако в реализации `Object.Equals` была обнаружена ошибка для этих классов, и для исправления данной ошибки потребовалось отключить неявное преобразование. При этом явное преобразование в тип `string` по-прежнему доступно.

#### <a name="example"></a>Пример
Если код выглядит следующим образом:

```csharp
var customTokenizerName = TokenizerName.Create("my_tokenizer"); 
var customTokenFilterName = TokenFilterName.Create("my_tokenfilter"); 
var customCharFilterName = CharFilterName.Create("my_charfilter"); 
 
var index = new Index();
index.Analyzers = new Analyzer[] 
{ 
    new CustomAnalyzer( 
        "my_analyzer",  
        customTokenizerName,  
        new[] { customTokenFilterName },  
        new[] { customCharFilterName }), 
}; 
```

Можно изменить его на следующий для устранения ошибок сборки:

```csharp
const string CustomTokenizerName = "my_tokenizer"; 
const string CustomTokenFilterName = "my_tokenfilter"; 
const string CustomCharFilterName = "my_charfilter"; 
 
var index = new Index();
index.Analyzers = new Analyzer[] 
{ 
    new CustomAnalyzer( 
        "my_analyzer",  
        CustomTokenizerName,  
        new TokenFilterName[] { CustomTokenFilterName },  
        new CharFilterName[] { CustomCharFilterName })
}; 
```

### <a name="removed-obsolete-members"></a>Удалены устаревшие члены

Вы можете столкнуться с ошибками сборки, связанными с методами или свойствами, которые были отмечены в версии 2.0-preview как устаревшие, а в версии 3 были удалены. При возникновении таких ошибок воспользуйтесь описанными ниже способами их устранения.

- Если вы использовали конструктор `ScoringParameter(string name, string value)`, то вместо него воспользуйтесь этим: `ScoringParameter(string name, IEnumerable<string> values)`.
- Если вы использовали свойство `ScoringParameter.Value`, то вместо него воспользуйтесь свойством `ScoringParameter.Values` или методом `ToString`.
- Если вы использовали свойство `SearchRequestOptions.RequestId`, то вместо него воспользуйтесь свойством `ClientRequestId`.

### <a name="removed-preview-features"></a>Удалены функции предварительной версии

При обновлении с версии 2.0-preview до версии 3 учитывайте, что поддержка анализа JSON и CSV для индексаторов больших двоичных объектов была удалена, так как эта функция все еще находится на стадии предварительной версии. В частности, удалены следующие методы класса `IndexingParametersExtensions`:

- `ParseJson`
- `ParseJsonArrays`
- `ParseDelimitedTextFiles`

Если ваше приложение имеет жесткую зависимость от этих функций, то вы не сможете обновить пакет SDK службы поиска Azure для .NET до версии 3. Вы можете и далее использовать версию 2.0-preview. Однако необходимо учитывать, что **использовать предварительные версии пакетов SDK в рабочих приложениях не рекомендуется**. Предварительные версии функций предназначены исключительно для оценки и могут изменяться.

## <a name="conclusion"></a>Заключение
Если вам нужны дополнительные сведения об использовании пакета SDK .NET для службы поиска Azure, то ознакомьтесь с [практическим руководством по .NET](search-howto-dotnet-sdk.md).

Будем рады вашим отзывам о пакете SDK. При возникновении проблем вы можете попросить нас получить помощь по [Stack overflow](https://stackoverflow.com/questions/tagged/azure-search). При обнаружении ошибки можно зарегистрировать проблему в [репозитории GitHub пакета SDK .NET для Azure](https://github.com/Azure/azure-sdk-for-net/issues). Добавьте в заголовок вашей проблемы префикс "[Azure Search]".

Благодарим вас за использование поиска Azure!