---
title: Примеры REST
titleSuffix: Azure Cognitive Search
description: Найдите демонстрационные примеры кода ОСТАВШЕЙся версии Azure Когнитивный поиск, которые используют API-интерфейсы поиска или управления.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 01/27/2021
ms.openlocfilehash: a7ab48759ac775c1195dedb4143d895bdcdec937
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98956587"
---
# <a name="rest-code-samples-for-azure-cognitive-search"></a>Примеры кода для Azure Когнитивный поиск

Узнайте о примерах REST API, демонстрирующих функциональность и рабочий процесс решения Когнитивный поиск Azure. В этих примерах используются [**API-интерфейсы RESTful поиска**](/rest/api/searchservice).

"Остальное" — это самый полный интерфейс программирования для Azure Когнитивный поиск, и все операции, которые могут быть вызваны программно, доступны сначала в остальное, а затем в пакетах SDK. По этой причине большинство примеров в документации используют интерфейсы API для демонстрации и объяснения важных концепций.

Примеры RESTFUL обычно разрабатываются и тестируются на завершающем компьютере, но можно использовать любой клиент, поддерживающий HTTP-вызовы:

+ Начните с [краткого руководства: создание индекса когнитивный Поиск Azure с помощью интерфейсов API](search-get-started-rest.md) для справки в составления HTTP-вызовах.
+ Попробуйте [Visual Studio Code расширение для когнитивный Поиск Azure](search-get-started-vs-code.md), которое в настоящее время находится на этапе предварительной версии, если вы работаете в Visual Studio Code.

## <a name="doc-samples"></a>Примеры документов

Примеры кода из группы Когнитивный поиск демонстрируют функции и рабочие процессы. Многие из этих примеров приведены в руководствах, кратких руководствах и в статьях с инструкциями. Эти примеры можно найти в примерах [**Azure-Samples/Azure-Search-POST-Samples**](https://github.com/Azure-Samples/azure-search-postman-samples) на сайте GitHub.

| Примеры | Статья |
|---------|---------|
| [Краткое руководство](https://github.com/Azure-Samples/azure-search-postman-samples/tree/master/Quickstart) | Исходный код для [краткого руководства: создание индекса поиска с помощью API-интерфейсов RESTful](search-get-started-rest.md). В этой статье рассматривается базовый рабочий процесс создания, загрузки и запроса индекса поиска с помощью демонстрационных данных. |
| [Руководство](https://github.com/Azure-Samples/azure-search-postman-samples/tree/master/Tutorial) | Исходный код для [учебника. Использование функций RESTful и AI для создания содержимого с возможностью поиска из больших двоичных объектов Azure](cognitive-search-tutorial-blob.md). В этой статье показано, как создать набор навыков, который перебирает большие двоичные объекты Azure, чтобы извлечь информацию и вывести структуру.|
| [Отладка — сеансы](https://github.com/Azure-Samples/azure-search-postman-samples/tree/master/Debug-sessions) | Исходный код для [учебника: диагностика, восстановление и фиксация изменений в](cognitive-search-tutorial-debug-sessions.md)наборе навыков. В этой статье показано, как использовать сеанс отладки набора навыков в портал Azure. Для создания объектов, используемых во время отладки, используется интерфейс RESTFUL.|
| [Пользовательские анализаторы](https://github.com/Azure-Samples/azure-search-postman-samples/tree/master/custom-analyzers) | Исходный код для [учебника: создание пользовательского анализатора для номеров телефонов](tutorial-create-custom-analyzer.md). В этой статье объясняется, как использовать анализаторы для сохранения шаблонов и специальных символов в содержимом с возможностью поиска.|
| [хранилище знаний](https://github.com/Azure-Samples/azure-search-postman-samples/tree/master/knowledge-store) | Исходный код для [создания хранилища знаний с помощью функции RESTful и POST](knowledge-store-create-rest.md). В этой статье описываются шаги, необходимые для заполнения хранилища знаний, используемого для рабочих процессов интеллектуального анализа знаний. |
| [проекции](https://github.com/Azure-Samples/azure-search-postman-samples/tree/master/projections) | Исходный код для [формирования и экспорта](knowledge-store-projections-examples.md)дополнений. В этой статье объясняется, как указать физические структуры данных в хранилище знаний.|
| [BLOB-объекты, зашифрованные с индексом](https://github.com/Azure-Samples/azure-search-postman-samples/commit/f5ebb141f1ff98f571ab84ac59dcd6fd06a46718) | Исходный код для [индексации зашифрованных больших двоичных объектов с помощью индексаторов больших двоичных объектов и навыков](search-howto-index-encrypted-blobs.md). В этой статье показано, как индексировать документы в хранилище BLOB-объектов Azure, которые ранее были зашифрованы с помощью Azure Key Vault. |

> [!Tip]
> Воспользуйтесь [браузером примеров](/samples/browse/?expanded=azure&languages=http&products=azure-cognitive-search) для поиска примеров кода Майкрософт в GitHub, отфильтрованных по продуктам, службам и языкам.

## <a name="other-samples"></a>Другие примеры

Следующие примеры также публикуются командой Когнитивный поиск, но на них нет ссылок в документации. Связанные файлы сведений содержат инструкции по использованию.

| Примеры | Описание |
|---------|-------------|
| [Примеры запросов](https://github.com/Azure-Samples/azure-search-postman-samples/tree/master/Query-examples) | Публиковать коллекции, демонстрирующие различные методы запросов, включая поиск нечетких символов, регулярное выражение и поиск с подстановочными знаками, автозаполнение и т. д. |