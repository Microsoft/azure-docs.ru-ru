---
title: Примеры для Java
titleSuffix: Azure Cognitive Search
description: Найдите демонстрационные примеры кода Java для Azure Когнитивный поиск, которые используют пакет SDK для Azure .NET для Java.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 01/27/2021
ms.openlocfilehash: b5ae38a3dc4a9324a4141314106d67c96c06c8e6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98955043"
---
# <a name="java-code-samples-for-azure-cognitive-search"></a>Примеры кода Java для Azure Когнитивный поиск

Узнайте о примерах кода Java, демонстрирующих функциональность и рабочий процесс решения Azure Когнитивный поиск. В этих примерах используется [**Клиентская библиотека azure когнитивный Поиск**](/java/api/overview/azure/search-documents-readme) для [**пакета Azure SDK для Java**](/azure/developer/java/sdk), который можно просмотреть по следующим ссылкам.

| Целевой объект | Ссылка |
|--------|------|
| Скачивание пакета | [search.maven.org/artifact/com.azure/azure-search-documents](https://search.maven.org/artifact/com.azure/azure-search-documents) |
| Справочник по API | [com.azure.search.docументс](/java/api/com.azure.search.documents)  |
| Тестовые случаи API | [github.com/Azure/azure-sdk-for-java/tree/azure-search-documents_11.1.3/sdk/search/azure-search-documents/src/test](https://github.com/Azure/azure-sdk-for-java/tree/azure-search-documents_11.1.3/sdk/search/azure-search-documents/src/test) |
| Исходный код | [github.com/Azure/azure-sdk-for-java/tree/azure-search-documents_11.1.3/sdk/search/azure-search-documents/src](https://github.com/Azure/azure-sdk-for-java/tree/azure-search-documents_11.1.3/sdk/search/azure-search-documents/src)  |

## <a name="sdk-samples"></a>Примеры SDK

Примеры кода из группы разработки Azure SDK демонстрируют использование API. Эти примеры можно найти в [**Azure/Azure-SDK-for-Java/дерево/Master/SDK/Поиск/Azure-Search-Documents/src/Samples**](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/search/azure-search-documents/src/samples) на сайте GitHub.

| Примеры | Описание |
|---------|-------------|
| [Создание индекса поиска](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/search/azure-search-documents/src/samples/java/com/azure/search/documents/indexes/CreateIndexExample.java) | Демонстрирует создание [индексов поиска](search-what-is-an-index.md). |
| [Создание синонимов](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/search/azure-search-documents/src/samples/java/com/azure/search/documents/SynonymMapsCreateExample.java) | Демонстрирует создание [сопоставлений синонимов](search-synonyms.md).  |
| [Создание индексатора поиска](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/search/azure-search-documents/src/samples/java/com/azure/search/documents/indexes/CreateIndexerExample.java) | Демонстрирует создание [индексаторов](search-indexer-overview.md). |
| [Создание источника данных индексатора поиска](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/search/azure-search-documents/src/samples/java/com/azure/search/documents/indexes/DataSourceExample.java) | Демонстрирует создание источников данных индексатора, необходимых для индексирования [поддерживаемых источников данных Azure](search-indexer-overview.md#supported-data-sources)на основе индексатора. |
| [Создание набора навыков](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/search/azure-search-documents/src/samples/java/com/azure/search/documents/indexes/CreateSkillsetExample.java) |  Демонстрирует создание [навыков](cognitive-search-working-with-skillsets.md) , присоединенных индексаторов и выполняющих обогащение на основе искусственного интеллекта во время индексирования. |
| [Загрузка документов](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/search/azure-search-documents/src/samples/java/com/azure/search/documents/IndexContentManagementExample.java) | Демонстрирует, как отправлять или объединять документы в индекс в операции [импорта данных](search-what-is-data-import.md) . |
| [Синтаксис запросов](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/search/azure-search-documents/src/samples/java/com/azure/search/documents/SearchAsyncWithFullyTypedDocumentsExample.java) | Демонстрирует, как настроить [базовый запрос](search-query-overview.md). |

## <a name="doc-samples"></a>Примеры документов

Примеры кода из группы Когнитивный поиск демонстрируют функции и рабочие процессы. Многие из этих примеров приведены в руководствах, кратких руководствах и в статьях с инструкциями. Эти примеры можно найти в [**Azure-Samples/Azure-Search-Java-Samples**](https://github.com/Azure-Samples/azure-search-java-samples) на сайте GitHub.

| Примеры | Статья | 
|---------|-------------|
| [QuickStart](https://github.com/Azure-Samples/azure-search-java-samples/tree/java-rest-api/quickstart) | Исходный код для [краткого руководства: создание индекса поиска в Java и в других частях](search-get-started-java.md). Этот пример не был обновлен для пакета SDK для Java. Он вызывает интерфейсы API-интерфейсов RESTFUL. |

> [!Tip]
> Воспользуйтесь [браузером примеров](/samples/browse/?languages=java&products=azure-cognitive-search) для поиска примеров кода Майкрософт в GitHub, отфильтрованных по продуктам, службам и языкам.

## <a name="other-samples"></a>Другие примеры

Следующие примеры также публикуются командой Когнитивный поиск, но на них нет ссылок в документации. Связанные файлы сведений содержат инструкции по использованию.

| Примеры | Описание |
|---------|-------------|
| [Поиск-Java-Приступая к работе](https://github.com/Azure-Samples/azure-search-java-samples/tree/master/search-java-getting-started) | Использует клиентскую библиотеку пакета SDK для Java для создания, загрузки и запроса индекса поиска. Этот образец в настоящее время является автономным. |
| [Поиск — Java-индексатор — демонстрация](https://github.com/Azure-Samples/azure-search-java-samples/tree/java-rest-api/search-java-indexer-demo) | Демонстрирует Azure Cosmos DB индексатора в Java. Этот пример не был обновлен для пакета SDK для Java. Он вызывает интерфейсы API-интерфейсов RESTFUL.|