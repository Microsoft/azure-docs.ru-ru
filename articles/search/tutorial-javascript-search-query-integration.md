---
title: Учебник по JavaScript. Основные возможности интеграции Поиска
titleSuffix: Azure Cognitive Search
description: Общие сведения о поисковых запросах пакета SDK JavaScript, используемых на веб-сайте с поддержкой поиска
manager: nitinme
author: diberry
ms.author: diberry
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 03/09/2021
ms.custom: devx-track-js
ms.devlang: javascript
ms.openlocfilehash: cf4e1b1ecf209b587a45ca4c43607bfa95155aee
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "104723559"
---
# <a name="4---search-integration-highlights"></a>4\. Основные возможности интеграции с Поиском

На предыдущих занятиях вы научились добавлять поиск в статическое веб-приложение. В этом уроке описываются основные шаги, выполнив которые можно настроить интеграцию. Если вы ищете краткий справочник о том, как интегрировать Поиск в приложение JavaScript, необходимые сведения можно узнать из этой статьи.

## <a name="azure-sdk-azuresearch-documents"></a>Пакет SDK Azure @azure/search-documents 

Приложение-функция использует пакет Azure SDK для службы "Когнитивный поиск":

* NPM: [@azure/search-documents](https://www.npmjs.com/package/@azure/search-documents)
* Справочная документация: [клиентская библиотека](/javascript/api/overview/azure/search-documents-readme)

Приложение-функция выполняет проверку подлинности с помощью пакета SDK для облачного API Когнитивного поиска, используя имя ресурса, ключ ресурса и имя индекса. Секреты хранятся в параметрах статического веб-приложения и передаются в функцию в виде переменных среды. 

## <a name="configure-secrets-in-a-configuration-file"></a>Настройка секретов в файле конфигурации

:::code language="javascript" source="~/azure-search-javascript-samples/search-website/api/config.js" highlight="3,4" :::

## <a name="azure-function-search-the-catalog"></a>Функция Azure: Поиск в каталоге

`Search` [API](https://github.com/Azure-Samples/azure-search-javascript-samples/blob/master/search-website/api/Search/index.js) принимает условие поиска и выполняет поиск по документам в индексе поиска, возвращая список совпадений. 

Маршрутизация для Search API содержится в привязках [function.json](https://github.com/Azure-Samples/azure-search-javascript-samples/blob/master/search-website/api/Search/function.json).

Функция Azure извлекает сведения о конфигурации Поиска и выполняет запрос.

:::code language="javascript" source="~/azure-search-javascript-samples/search-website/api/Search/index.js" highlight="4-9, 75" :::

## <a name="client-search-from-the-catalog"></a>Клиент: Поиск из каталога

Вызовите Функцию Azure в клиенте React, используя следующий код: 

:::code language="javascript" source="~/azure-search-javascript-samples/search-website/src/pages/Search/Search.js" highlight="40-51" :::

## <a name="azure-function-suggestions-from-the-catalog"></a>Функция Azure: предложения из каталога

`Suggest`[API](https://github.com/Azure-Samples/azure-search-javascript-samples/blob/master/search-website/api/Suggest/index.js) принимает условие поиска, когда пользователь выполняет ввод, и предлагает условия поиска, такие как заголовки книг и авторы, по документам в индексе поиска, и возвращает небольшой список совпадений. 

Средство подбора для поиска, `sg`, определяется в [файле схемы](https://github.com/Azure-Samples/azure-search-javascript-samples/blob/master/search-website/bulk-insert/good-books-index.json), используемом при выполнении групповой отправки.

Маршрутизация для Suggest API содержится в привязках [function.json](https://github.com/Azure-Samples/azure-search-javascript-samples/blob/master/search-website/api/Suggest/function.json).

:::code language="javascript" source="~/azure-search-javascript-samples/search-website/api/Suggest/index.js" highlight="4-9, 21" :::

## <a name="client-suggestions-from-the-catalog"></a>Клиент: предложения из каталога

API функции Suggest вызывается в приложении React `\src\components\SearchBar\SearchBar.js` в рамках инициализации компонента:

:::code language="javascript" source="~/azure-search-javascript-samples/search-website/src/components/SearchBar/SearchBar.js" highlight="52-60" :::

## <a name="azure-function-get-specific-document"></a>Функция Azure: получение конкретного документа 

`Lookup` [API](https://github.com/Azure-Samples/azure-search-javascript-samples/blob/master/search-website/api/Lookup/index.js) принимает идентификатор и возвращает объект документа из индекса Search. 

Маршрутизация для Lookup API содержится в привязках [function.json](https://github.com/Azure-Samples/azure-search-javascript-samples/blob/master/search-website/api/Lookup/function.json).

:::code language="javascript" source="~/azure-search-javascript-samples/search-website/api/Lookup/index.js" highlight="4-9, 17" :::

## <a name="client-get-specific-document"></a>Клиент: получение конкретного документа 

Этот программный интерфейс функции вызывается в приложении React `\src\pages\Details\Detail.js` в рамках инициализации компонента:

:::code language="javascript" source="~/azure-search-javascript-samples/search-website/src/pages/Details/Details.js" highlight="19-29" :::

## <a name="next-steps"></a>Дальнейшие действия

* [Индексирование данных Azure SQL](search-indexer-tutorial.md)
