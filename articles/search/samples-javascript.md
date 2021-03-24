---
title: Примеры для JavaScript
titleSuffix: Azure Cognitive Search
description: Найдите примеры кода JavaScript для Azure Когнитивный поиск, которые используют пакет SDK для Azure .NET для JavaScript.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 01/27/2021
ms.openlocfilehash: 85a4d6390087100d8d9521f6ac20dbace3a711eb
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104955947"
---
# <a name="javascript-code-samples-for-azure-cognitive-search"></a>Примеры кода JavaScript для Azure Когнитивный поиск

Узнайте о примерах кода JavaScript, демонстрирующих функциональность и рабочий процесс решения Azure Когнитивный поиск. В этих примерах используется [**Клиентская библиотека azure когнитивный Поиск**](/javascript/api/overview/azure/search-documents-readme) для [**пакета Azure SDK для JavaScript**](/azure/developer/javascript/), который можно просмотреть по следующим ссылкам.

| Целевой объект | Ссылка |
|--------|------|
| Скачивание пакета | [www.npmjs.com/package/@azure/search-documents](https://www.npmjs.com/package/@azure/search-documents) |
| Справочник по API | [@azure/search-documents](/javascript/api/@azure/search-documents/)  |
| Тестовые случаи API | [github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/test](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/test) |
| Исходный код | [github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents)  |

## <a name="sdk-samples"></a>Примеры SDK

Примеры кода из группы разработки Azure SDK демонстрируют использование API. Эти примеры можно найти в [**Azure-SDK-for-JS/дерево/Master/SDK/Поиск/Поиск-документы/примеры**](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples) на сайте GitHub.

### <a name="javascript-sdk-samples"></a>Примеры SDK для JavaScript

| Примеры | Описание |
|---------|-------------|
| [индексирует](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/javascript/src/indexes) | Демонстрирует создание, обновление, получение, перечисление и удаление [индексов поиска](search-what-is-an-index.md). В этот пример категории также включен пример статистики службы. |
| [Датасаурцеконнектионс (для индексаторов)](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/javascript/src/dataSourceConnections) | Демонстрирует создание, обновление, получение, перечисление и удаление источников данных индексатора, необходимых для индексирования [поддерживаемых источников данных Azure](search-indexer-overview.md#supported-data-sources)на основе индексатора. |
| [индексаторов](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/javascript/src/indexers) |  Демонстрирует создание, обновление, получение, перечисление, сброс и удаление [индексаторов](search-indexer-overview.md).|
| [Набор навыков](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/javascript/src/skillSets) |   Демонстрирует создание, обновление, получение, перечисление и удаление [навыков](cognitive-search-working-with-skillsets.md) , присоединенных индексаторов, и выполнение обогащения на основе искусственного интеллекта во время индексирования. |
| [synonymMaps](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/javascript/src/synonymMaps) | Демонстрирует создание, обновление, получение, перечисление и удаление [сопоставлений синонимов](search-synonyms.md).  |

### <a name="typescript-samples"></a>Примеры TypeScript

| Примеры | Описание |
|---------|-------------|
| [индексирует](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/typescript/src/indexes) | Демонстрирует создание, обновление, получение, перечисление и удаление [индексов поиска](search-what-is-an-index.md). В этот пример категории также включен пример статистики службы. |
| [Датасаурцеконнектионс (для индексаторов)](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/typescript/src/dataSourceConnections) | Демонстрирует создание, обновление, получение, перечисление и удаление источников данных индексатора, необходимых для индексирования [поддерживаемых источников данных Azure](search-indexer-overview.md#supported-data-sources)на основе индексатора. |
| [индексаторов](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/typescript/src/indexers) |  Демонстрирует создание, обновление, получение, перечисление, сброс и удаление [индексаторов](search-indexer-overview.md).|
| [Набор навыков](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/typescript/src/skillSets) |   Демонстрирует создание, обновление, получение, перечисление и удаление [навыков](cognitive-search-working-with-skillsets.md) , присоединенных индексаторов, и выполнение обогащения на основе искусственного интеллекта во время индексирования. |
| [synonymMaps](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/search/search-documents/samples/typescript/src/synonymMaps) | Демонстрирует создание, обновление, получение, перечисление и удаление [сопоставлений синонимов](search-synonyms.md).  |

## <a name="doc-samples"></a>Примеры документов

Примеры кода из группы Когнитивный поиск демонстрируют функции и рабочие процессы. Многие из этих примеров приведены в руководствах, кратких руководствах и в статьях с инструкциями. Эти примеры можно найти в примерах [**Azure-Samples/Azure-Search-JavaScript-Samples**](https://github.com/Azure-Samples/azure-search-javascript-samples) на сайте GitHub.

| Примеры | Статья |
|---------|---------|
| [QuickStart](https://github.com/Azure-Samples/azure-search-javascript-samples/tree/master/quickstart/v11) | Исходный код для [краткого руководства: создание индекса поиска в JavaScript](search-get-started-javascript.md). В этой статье рассматривается базовый рабочий процесс создания, загрузки и запроса индекса поиска с помощью демонстрационных данных. |

> [!Tip]
> Воспользуйтесь [браузером примеров](/samples/browse/?languages=javascript&products=azure-cognitive-search) для поиска примеров кода Майкрософт в GitHub, отфильтрованных по продуктам, службам и языкам.

## <a name="other-samples"></a>Другие примеры

Следующие примеры также публикуются командой Когнитивный поиск, но на них нет ссылок в документации. Связанные файлы сведений содержат инструкции по использованию.

| Примеры | Описание |
|---------|-------------|
| [Azure-Search-реагирующий-шаблон](https://github.com/dereklegenzoff/azure-search-react-template) | Шаблон отклика для Когнитивный поиск Azure (github.com) |