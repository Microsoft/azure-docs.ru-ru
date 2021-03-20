---
title: Примеры для .NET
titleSuffix: Azure Cognitive Search
description: Поиск демонстрационных примеров кода C# для Azure Когнитивный поиск, использующих клиентские библиотеки .NET.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 01/27/2021
ms.openlocfilehash: cbb84a4934eed4d258cf07772753315785f98019
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99218168"
---
# <a name="net-c-code-samples-for-azure-cognitive-search"></a>Примеры кода .NET (C#) для Azure Когнитивный поиск

Узнайте о примерах кода C#, демонстрирующих функциональность и рабочий процесс решения Azure Когнитивный поиск. В этих примерах используется [**Клиентская библиотека azure когнитивный Поиск**](/dotnet/api/overview/azure/search) для [**пакета Azure SDK для .NET**](/dotnet/azure/), которую можно просмотреть по следующим ссылкам.

| Целевой объект | Ссылка |
|--------|------|
| Скачивание пакета | [www.nuget.org/packages/Azure.Search.Docументс/](https://www.nuget.org/packages/Azure.Search.Documents/) |
| Справочник по API | [azure.search.docументс](/dotnet/api/azure.search.documents)  |
| Тестовые случаи API | [github.com/Azure/azure-sdk-for-net/tree/master/sdk/search/Azure.Search.Docументс/тесты](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/search/Azure.Search.Documents/tests) |
| Исходный код | [github.com/Azure/azure-sdk-for-net/tree/master/sdk/search/Azure.Search.Docументс/src](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/search/Azure.Search.Documents/src)  |

## <a name="sdk-samples"></a>Примеры SDK

Примеры кода из группы разработки Azure SDK демонстрируют использование API. Эти примеры можно найти в [**Azure/Azure-SDK-for-NET/дерево/Master/SDK/Search/Azure.Search.Docументс/Samples**](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/samples/) на сайте GitHub.

| Примеры | Описание |
|---------|-------------|
| ["Hello World", синхронно](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/samples/Sample01a_HelloWorld.md) | Демонстрирует создание клиента, проверку подлинности и обработку ошибок с помощью синхронных методов.|
| ["Hello World" — асинхронно](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/samples/Sample01b_HelloWorldAsync.md) | Демонстрирует создание клиента, проверку подлинности и обработку ошибок с помощью асинхронных методов.  |
| [Операции уровня службы](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/samples/Sample02_Service.md) | Демонстрирует создание индексов, индексаторов, источников данных, навыков и сопоставлений синонимов. В этом примере также показано, как получить статистику службы и как запросить индекс.  |
| [Операции с индексами](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/samples/Sample03_Index.md) | Демонстрирует выполнение действия с существующим индексом, в данном случае получение числа документов, хранящихся в индексе.  |
| [фиелдбуилдеригноре](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/samples/Sample04_FieldBuilderIgnore.md) | Демонстрирует методику работы с неподдерживаемыми типами данных.  |
| [Индексирование документов (модель push-уведомлений)](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/samples/Sample05_IndexingDocuments.md) | Индексирование модели push, куда вы отправляете полезные данные JSON в индекс службы.   |
| [Пример ключа шифрования](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/samples/Sample06_EncryptedIndex.md) | Демонстрирует использование управляемого клиентом ключа шифрования для добавления дополнительного уровня защиты от конфиденциального содержимого.  |

## <a name="doc-samples"></a>Примеры документов

Примеры кода из группы Когнитивный поиск демонстрируют функции и рабочие процессы. Многие из этих примеров приведены в руководствах, кратких руководствах и в статьях с инструкциями. Эти примеры можно найти в статье [**примеры Azure-Samples/Azure-Search-DotNet-Samples**](https://github.com/Azure-Samples/azure-search-dotnet-samples) и в [**Azure-Samples/Search-DotNet-Приступая к работе**](https://github.com/Azure-Samples/search-dotnet-getting-started/) на GitHub.

| Примеры | Статья  |
|---------|-------------|
| [QuickStart](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/quickstart) | Исходный код для [краткого руководства: создание индекса поиска ](search-get-started-dotnet.md). В этой статье рассматривается базовый рабочий процесс создания, загрузки и запроса индекса поиска с помощью демонстрационных данных. |
| [дотнесовто](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo)  | Исходный код для [использования клиентской библиотеки .NET](search-howto-dotnet-sdk.md). В этой статье описывается базовый рабочий процесс, но более подробно рассматриваются сведения об использовании API.  |
| [дотнесовтосинонимс](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToSynonyms)  | Исходный код, [например: Добавление синонимов в C#](search-synonyms-tutorial-sdk.md). Списки синонимов используются для расширения запросов, предоставляя сопоставляемые термины, которые являются внешними по отношению к индексу. |
| [дотнеттоиндексерс](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToIndexers) | Исходный код для [учебника: индексирование данных SQL Azure с помощью пакета SDK для .NET](search-indexer-tutorial.md). В этой статье показано, как настроить индексатор SQL Azure с расписанием, сопоставлениями полей и параметрами.  |
| [дотнесовтоенкриптионусингкмк](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToEncryptionUsingCMK)  | Исходный код для [настройки ключей, управляемых клиентом, для шифрования данных](search-security-manage-encryption-keys.md). |
| [Создание первого приложения на языке C #](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v11) |  Исходный код для [учебника: создание первого приложения поиска](tutorial-csharp-create-first-app.md). Хотя большинство примеров являются консольными приложениями, в этом примере MVC используется веб-страница для переднего плана с примером индекса гостиниц, демонстрирующий базовый поиск, разбивку на страницы, автозаполнение и предлагаемые запросы, аспекты и фильтры. |
| [множественные источники данных](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/multiple-data-sources)  | Исходный код для [учебника: индекс из нескольких источников данных](tutorial-multiple-data-sources.md). |
|  [Оптимизация — индексирование данных](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/optimize-data-indexing) | Исходный код для [учебника. Оптимизация индексирования с помощью API push-уведомлений](tutorial-optimize-indexing-push-api.md).  |
| [Руководство по AI — обогащение](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/tutorial-ai-enrichment)  | Исходный код для [руководства. созданное AI содержимое из больших двоичных объектов Azure с помощью пакета SDK для .NET](cognitive-search-tutorial-blob-dotnet.md).  |

> [!Tip]
> Воспользуйтесь [браузером примеров](/samples/browse/?languages=csharp&products=azure-cognitive-search) для поиска примеров кода Майкрософт в GitHub, отфильтрованных по продуктам, службам и языкам.

## <a name="other-samples"></a>Другие примеры

Следующие примеры также публикуются командой Когнитивный поиск, но на них нет ссылок в документации. Связанные файлы сведений содержат инструкции по использованию.

| Примеры | Описание |
|---------|-------------|
| [Azure-Search-Power-Skills](https://github.com/Azure-Samples/azure-search-power-skills)  | Исходный код для использования настраиваемых навыков, которые можно включить в выигранные решения.  |
| [Акселератор решений для интеллектуального анализа знаний](/samples/azure-samples/azure-search-knowledge-mining/azure-search-knowledge-mining/) | Включает шаблоны, файлы поддержки и аналитические отчеты для создания прототипа комплексного решения по интеллектуальному анализу знаний.  |
| [Репозиторий приложений для поиска ковид-19](https://github.com/liamca/covid19search) | Репозиторий исходного кода для [приложения поиска ковид-19](https://covid19search.azurewebsites.net/) на основе когнитивный Поиск |
| [JFK](https://github.com/Microsoft/AzureSearch_JFK_Files) | Дополнительные сведения о [решении ЖФК](https://www.microsoft.com/ai/ai-lab-jfk-files). |
| [Поиск + QnA Maker Accelerator](https://github.com/Azure-Samples/search-qna-maker-accelerator) | [Решение](https://techcommunity.microsoft.com/t5/azure-ai/qna-with-azure-cognitive-search/ba-p/2081381) , объединяющее возможности поиска и QnA Maker. См. [веб-сайт демонстрации](https://aka.ms/qnaWithAzureSearchDemo)в реальном времени. |