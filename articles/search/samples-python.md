---
title: Примеры для Python
titleSuffix: Azure Cognitive Search
description: Ознакомьтесь с примерами кода Python для Azure Когнитивный поиск, которые используют пакет SDK для Azure .NET для Python или других компонентов.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 01/27/2021
ms.openlocfilehash: 0d09851cf8e68cead4a67615aaa792512482f351
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98955128"
---
# <a name="python-code-samples-for-azure-cognitive-search"></a>Примеры кода Python для Azure Когнитивный поиск

Узнайте о примерах кода Python, демонстрирующих функциональность и рабочий процесс решения Azure Когнитивный поиск. В этих примерах используется [**Клиентская библиотека azure когнитивный Поиск**](/python/api/overview/azure/search-documents-readme) для [**пакета Azure SDK для Python**](/azure/developer/python/), который можно просмотреть по следующим ссылкам.

| Целевой объект | Ссылка |
|--------|------|
| Скачивание пакета | [pypi.org/project/azure-search-documents/](https://pypi.org/project/azure-search-documents/) |
| Справочник по API | [azure-search-documents](/python/api/azure-search-documents)  |
| Тестовые случаи API | [github.com/Azure/azure-sdk-for-python/tree/master/sdk/search/azure-search-documents/tests](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/search/azure-search-documents/tests) |
| Исходный код | [github.com/Azure/azure-sdk-for-python/tree/master/sdk/search/azure-search-documents](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/search/azure-search-documents)  |

## <a name="sdk-samples"></a>Примеры SDK

Примеры кода из группы разработки Azure SDK демонстрируют использование API. Эти примеры можно найти в [**Azure-SDK-for-Python/дерево/Master/SDK/Поиск/Azure-Search-Documents/Samples**](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/search/azure-search-documents/samples) на сайте GitHub.

| Примеры | Описание |
|---------|-------------|
| [Проверка подлинности](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_authentication.py) | Демонстрирует настройку клиента и проверку подлинности для службы. | 
| [Операции создания индекса — чтение и обновление — удаление](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_index_crud_operations.py) | Демонстрирует создание, обновление, получение, перечисление и удаление [индексов поиска](search-what-is-an-index.md). |
| [Операции создания индексатора — чтение и обновление — удаление](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_indexers_operations.py) | Демонстрирует создание, обновление, получение, перечисление, сброс и удаление [индексаторов](search-indexer-overview.md). |
| [Поиск источников данных индексатора](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_indexer_datasource_skillset.py) | Демонстрирует создание, обновление, получение, перечисление и удаление источников данных индексатора, необходимых для индексирования [поддерживаемых источников данных Azure](search-indexer-overview.md#supported-data-sources)на основе индексатора. |
| [Синонимы](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_synonym_map_operations.py) | Демонстрирует создание, обновление, получение, перечисление и удаление [сопоставлений синонимов](search-synonyms.md).  |
| [Загрузка документов](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_crud_operations.py) | Демонстрирует, как отправлять или объединять документы в индекс в операции [импорта данных](search-what-is-data-import.md) . |
| [Простой запрос](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_simple_query.py) | Демонстрирует, как настроить [базовый запрос](search-query-overview.md). |
| [Фильтрация запроса](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_filter_query.py) | Демонстрируется настройка [критерия фильтра](search-filters.md). |
| [Запрос аспекта](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/search/azure-search-documents/samples/sample_facet_query.py) | Демонстрирует работу с [аспектами](search-filters-facets.md). |

## <a name="doc-samples"></a>Примеры документов

Примеры кода из группы Когнитивный поиск демонстрируют функции и рабочие процессы. Многие из этих примеров приведены в руководствах, кратких руководствах и в статьях с инструкциями. Эти примеры можно найти в примерах [**Azure-Samples/Azure-Search-Python-Samples**](https://github.com/Azure-Samples/azure-search-python-samples) на сайте GitHub.

| Примеры | Статья |
|---------|---------|
| [QuickStart](https://github.com/Azure-Samples/azure-search-python-samples/tree/master/Quickstart) | Исходный код для [краткого руководства: создание индекса поиска в Python](search-get-started-python.md). В этой статье рассматривается базовый рабочий процесс создания, загрузки и запроса индекса поиска с помощью демонстрационных данных. |
| [Руководство по AI — обогащение](https://github.com/Azure-Samples/azure-search-python-samples/tree/master/Tutorial-AI-Enrichment)  | Исходный код для [учебника. Использование Python и AI для создания содержимого с возможностью поиска из больших двоичных объектов Azure](cognitive-search-tutorial-blob-python.md). В этой статье показано, как создать индексатор больших двоичных объектов с помощью набора квалификационных навыков, где набор навыков создает и преобразует необработанное содержимое, чтобы его можно было использовать для поиска или использования. |
| [AzureML-Custom-навык](https://github.com/Azure-Samples/azure-search-python-samples/tree/master/AzureML-Custom-Skill)  | Исходный код [. пример. Создание пользовательского навыка с помощью Python](cognitive-search-custom-skill-python.md). В этой статье демонстрируется интеграция индексатора и набора навыков с моделями глубокого обучения в Машинное обучение Azure. |

> [!Tip]
> Воспользуйтесь [браузером примеров](/samples/browse/?languages=python&products=azure-cognitive-search) для поиска примеров кода Майкрософт в GitHub, отфильтрованных по продуктам, службам и языкам.