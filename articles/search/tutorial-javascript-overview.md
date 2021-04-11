---
title: Учебник по JavaScript. Обзор интеграции с поиском
titleSuffix: Azure Cognitive Search
description: Технический обзор и настройка для добавления поиска на веб-сайт и развертывания в статическом веб-приложении Azure.
manager: nitinme
author: diberry
ms.author: diberry
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 03/18/2021
ms.custom: devx-track-js
ms.devlang: javascript
ms.openlocfilehash: 03192b8a84b78682b53bf3d47e7de7b65eb8bceb
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "104723563"
---
# <a name="1---overview-of-adding-search-to-a-website"></a>1\. Обзор добавления поиска на веб-сайт

В этом учебнике описано создание веб-сайта для поиска по каталогу книг, а затем развертывание веб-сайта в статическом веб-приложении Azure. 

Приложение доступно в следующем виде: 
* [Образец](https://github.com/Azure-Samples/azure-search-javascript-samples/tree/master/search-website)
* [Демонстрационная версия веб-сайта — aka.ms/azs-good-books](https://aka.ms/azs-good-books)

## <a name="what-does-the-sample-do"></a>Как работает пример? 

Этот пример веб-сайта предоставляет доступ к каталогу, содержащему 10 000 книг. Пользователь может выполнять поиск в каталоге, вводя текст на панели поиска. Пока пользователь вводит текст, веб-сайт использует функцию предложений индекса поиска для заполнения текста в поле поиска. После завершения запроса отображается список книг с частью сведений. Пользователь может выбрать книгу, чтобы просмотреть все сведения, хранящиеся в индексе поиска книги. 

:::image type="content" source="./media/tutorial-javascript-overview/cognitive-search-enabled-book-website.png" alt-text="Этот пример веб-сайта предоставляет доступ к каталогу, содержащему 10 000 книг. Пользователь может выполнять поиск в каталоге, вводя текст на панели поиска. Пока пользователь вводит текст, веб-сайт использует функцию предложений индекса поиска для заполнения текста в поле поиска. После завершения поиска отображается список книг с частью сведений. Пользователь может выбрать книгу, чтобы просмотреть все сведения, хранящиеся в индексе поиска книги.":::

Возможности поиска включают в себя: 

* Поиск — предоставляет функции поиска для приложения.
* Предложение — предоставляет предложения по мере ввода текста пользователем в строке поиска.
* Поиск документов — выполняет поиск документа по идентификатору, чтобы получить все его содержимое на странице сведений.

## <a name="how-is-the-sample-organized"></a>Какова структура примера?

[Пример](https://github.com/Azure-Samples/azure-search-javascript-samples/tree/master/search-website) включает в себя следующее:

|Приложение|Цель|GitHub<br>Хранилище<br>Расположение|
|--|--|--|
|Клиент|Приложение React (уровень представления) для отображения книг с помощью поиска. Оно вызывает приложение-функцию Azure. |[/search-website/src](https://github.com/Azure-Samples/azure-search-javascript-samples/tree/master/search-website/src)|
|Сервер|Приложение-функция Azure (на бизнес-уровне) — вызывает API Когнитивного поиска Azure с помощью пакета SDK для JavaScript |[/search-website/api](https://github.com/Azure-Samples/azure-search-javascript-samples/tree/master/search-website/src)|
|BULK INSERT (массовая вставка)|Файл JavaScript для создания индекса и добавления в него документов.|[/search-website/bulk-insert](https://github.com/Azure-Samples/azure-search-javascript-samples/tree/master/search-website/bulk-insert)|

## <a name="set-up-your-development-environment"></a>Настройка среды разработки

Установите следующие элементы среды разработки: 

- [Node.js 12 или 14](https://nodejs.org/en/download)
    - Если на локальном компьютере установлена другая версия Node.js, используйте [диспетчер версий узла](https://github.com/nvm-sh/nvm) (nvm) или контейнер Docker.  
- [Git](https://git-scm.com/downloads);
- [Visual Studio Code](https://code.visualstudio.com/) со следующими расширениями:
    - [Ресурсы Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureresourcegroups)
    - [Когнитивный поиск Azure 0.2.0 или более поздней версии](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurecognitivesearch)
    - [Статическое веб-приложение Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurestaticwebapps) 
- Необязательное действие:
    - В рамках этого учебника API Функций Azure не запускался локально, но если вы планируете запустить его локально, необходимо глобально установить [azure-functions-core-tools](/azure/azure-functions/functions-run-local?tabs=linux%2Ccsharp%2Cbash) с помощью следующей команды Bash: 
    
    ```bash
    npm install -g azure-functions-core-tools
    ```

## <a name="fork-and-clone-the-search-sample-with-git"></a>Разветвление и клонирование примера поиска с помощью Git

Разветвление примера репозитория очень важно для развертывания статического веб-приложения. Веб-приложения определяют действия сборки и содержимое развертывания на основе вашего расположения разветвления GitHub. Выполнение кода в статическом веб-приложении осуществляется удаленно. При этом статические веб-приложения Azure считываются из кода в разветвленном примере.

1. В GitHub создайте разветвление [примера репозитория](https://github.com/Azure-Samples/azure-search-javascript-samples). 

    Завершите процесс разветвления в веб-браузере с помощью учетной записи GitHub. В этом учебнике используется равилка как часть развертывания в статическом веб-приложении Azure. 

1. В терминале Bash скачайте пример приложения на локальный компьютер. 

    Замените `YOUR-GITHUB-ALIAS` псевдонимом записи GitHub. 

    ```bash
    git clone https://github.com/YOUR-GITHUB-ALIAS/azure-search-javascript-samples
    ```

1. В Visual Studio Code откройте локальную папку клонированного репозитория. Оставшиеся задачи выполняются в Visual Studio Code, если не указано иное.

## <a name="create-a-resource-group-for-your-azure-resources"></a>Создайте группу ресурсов для ресурсов Azure.

1. В Visual Studio Code откройте [Панель действий](https://code.visualstudio.com/docs/getstarted/userinterface) и выберите значок Azure. 
1. На боковой панели **щелкните правой кнопкой мыши подписку Azure** в области `Resource Groups` и выберите **Создать группу ресурсов**.

    :::image type="content" source="./media/tutorial-javascript-overview/visual-studio-code-create-resource-group.png" alt-text="На боковой панели щелкните **правой кнопкой мыши подписку Azure** в области &quot;Группа ресурсов&quot; и выберите **Создать группу ресурсов**.":::
1. Введите имя группы ресурсов, например `cognitive-search-website-tutorial`. 
1. Выберите расположение рядом с вами.
1. При создании ресурсов Когнитивного поиска и статического веб-приложения далее в рамках данного учебника используется эта группа ресурсов. 

    Создание группы ресурсов предоставляет вам логическую единицу для управления ресурсами, включая их удаление по завершении их использования.

## <a name="next-steps"></a>Дальнейшие действия

* [Создание индекса поиска и загрузка документов](tutorial-javascript-create-load-index.md)
* [Развертывание статического веб-приложения](tutorial-javascript-deploy-static-web-app.md)
