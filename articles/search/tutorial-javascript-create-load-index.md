---
title: Учебник по JavaScript. Добавление поиска в веб-приложения
titleSuffix: Azure Cognitive Search
description: Создайте индекс и импортируйте данные CSV-файла в индекс службы "Поиск" с помощью JavaScript, используя пакет SDK для npm @azure/search-documents.
manager: nitinme
author: diberry
ms.author: diberry
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 03/18/2021
ms.custom: devx-track-js
ms.devlang: javascript
ms.openlocfilehash: 0fd28262f4a4b852386fa354037e69c5097109c5
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "104723588"
---
# <a name="2---create-and-load-search-index-with-javascript"></a>2\. Создание и загрузка индекса службы "Поиск"с помощью JavaScript

Продолжайте создавать веб-сайты с поддержкой Поиска, выполнив следующие действия:
* Создание ресурса Поиска с помощью расширения VS Code
* Создание нового индекса и импорт данных с помощью JavaScript, используя пример скрипта и пакет SDK Azure [@azure/search-documents](https://www.npmjs.com/package/@azure/search-documents).

## <a name="create-an-azure-search-resource"></a>Создание индекса службы "Поиск Azure" 

Создайте новый ресурс Поиска с помощью расширения [Когнитивный поиск Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurecognitivesearch) для Visual Studio Code.

1. В Visual Studio Code откройте [панель действий](https://code.visualstudio.com/docs/getstarted/userinterface) и выберите значок Azure. 

1. В боковой панели щелкните **правой кнопкой мыши подписку Azure** в области `Azure: Cognitive Search` и выберите **Create new search service** (Создать службу поиска).

    :::image type="content" source="./media/tutorial-javascript-create-load-index/visual-studio-code-create-search-resource.png" alt-text="Экран &quot;Выбор правой кнопкой мыши подписки Azure в области &quot;Когнитивный поиск Azure&quot; и пункта Create new search service (Создать новую службу поиска) на боковой панели&quot;.":::

1. При появлении запросов введите представленные ниже сведения:

    |prompt|ВВОД|
    |--|--|
    |Enter a globally unique name for the new Search Service (Введите глобально уникальное имя для новой службы "Поиск").|**Запомните это имя**. Это имя ресурса станет частью конечной точки ресурса.|
    |Select a resource group for new resources (Выберите группу ресурсов для новых ресурсов)|Выберите группу ресурсов, созданную для работы с этим учебником.|
    |Select the SKU for your Search service (Выберите номер SKU для службы "Поиск").|Для работы с этим учебником нужно выбрать SKU уровня **Бесплатный**. Изменить ценовую категорию SKU после создания службы невозможно.|
    |Select a location for new resources (Выберите расположение для новых ресурсов).|Выберите ближайший регион.|

1. После того, как вы укажете все данные при появлении запросов, будет создан ресурс службы "Поиск". 

## <a name="get-your-search-resource-admin-key"></a>Получение ключа администратора ресурсов службы "Поиск"

Получите ключ администратора ресурса службы "Поиск" с помощью расширения Visual Studio Code. 

1. В Visual Studio Code на боковой панели щелкните правой кнопкой мыши ресурс службы "Поиск" и выберите **Copy Admin Key** (Копировать ключ администратора).

    :::image type="content" source="./media/tutorial-javascript-create-load-index/visual-studio-code-copy-admin-key.png" alt-text="Выбор правой кнопкой мыши ресурса службы &quot;Поиск&quot; и пункта Copy Admin Key (Копировать ключ администратора).":::

1. Сохраните ключ администратора, он понадобится в [следующем разделе](#prepare-the-bulk-import-script-for-search). 

## <a name="prepare-the-bulk-import-script-for-search"></a>Подготовка скрипта массового импорта для Поиска

В скрипте используется пакет Azure SDK для Когнитивного поиска:

* [пакет npm @azure/search-documents](https://www.npmjs.com/package/@azure/search-documents)
* [Справочная документация](/javascript/api/overview/azure/search-documents-readme)

1. В Visual Studio Code откройте файл `bulk_insert_books.js` в подкаталоге, `search-web/bulk-insert`, замените следующие переменные собственными значениями, чтобы выполнить проверку подлинности с помощью пакета SDK службы "Поиск Azure":

    * YOUR-SEARCH-RESOURCE-NAME
    * YOUR-SEARCH-ADMIN-KEY

    :::code language="javascript" source="~/azure-search-javascript-samples/search-website/bulk-insert/bulk_insert_books.js" highlight="16,17" :::

1. Откройте встроенный терминал в Visual Studio для подкаталога каталога проекта, `search-web/bulk-insert`, и выполните следующую команду, чтобы установить зависимости. 

    ```bash
    npm install 
    ```

## <a name="run-the-bulk-import-script-for-search"></a>Запуск скрипта массового импорта для службы "Поиск"

1. Продолжайте использовать встроенный терминал в Visual Studio для подкаталога каталога проекта, `search-web/bulk-insert`, чтобы выполнить следующую команду bash для запуска скрипта `bulk_insert_books.js`:

    ```javascript
    npm start
    ```

1. По мере выполнения кода в консоли будет отображатся ход выполнения. 
1. После завершения отправки последней выводимой на консоль инструкцией будет "завершено".

## <a name="review-the-new-search-index"></a>Просмотр нового индекса службы "Поиск"

Индекс службы "Поиск" можно использовать сразу после завершения отправки. Просмотрите новый индекс.

1. В Visual Studio Code откройте расширение "Когнитивный поиск Azure" и выберите свой ресурс службы "Поиск".  

    :::image type="content" source="media/tutorial-javascript-create-load-index/visual-studio-code-search-extension-view-resource.png" alt-text="Экран &quot;Открытие в Visual Studio Code расширения &quot;Когнитивный поиск Azure&quot; и выбор ресурса службы &quot;Поиск&quot;.":::

1. Разверните пункты "Индексы", "Документы", затем `good-books`, и выберите документ, чтобы просмотреть все данные, относящиеся к документу.
 
    :::image type="content" source="media/tutorial-javascript-create-load-index/visual-studio-code-search-extension-view-docs.png" lightbox="media/tutorial-javascript-create-load-index/visual-studio-code-search-extension-view-docs.png" alt-text="Экран &quot;Развертывание пунктов &quot;Индексы&quot;, good-books и выбор документа&quot;.":::

## <a name="copy-your-search-resource-name"></a>Копирование имени ресурса службы "Поиск"

Запишите **имя ресурса службы "Поиск"** . Оно понадобится для подключения приложения-функции Azure к ресурсу службы "Поиск". 

> [!CAUTION]
> Хотя вам может быть предложено использовать в Функции Azure свой ключ администратора службы "Поиск", это не будет отвечать принципу минимальных привилегий. Функция Azure будет использовать ключ запроса для обеспечения соответствия минимальным привилегиям. 

## <a name="next-steps"></a>Дальнейшие действия

[Развертывание статического веб-приложения](tutorial-javascript-deploy-static-web-app.md)