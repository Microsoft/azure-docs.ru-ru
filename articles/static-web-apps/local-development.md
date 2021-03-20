---
title: Настройка локальной разработки для Статических веб-приложений Azure
description: Сведения о настройке локальной среды разработки для Статических веб-приложений Azure
services: static-web-apps
author: burkeholland
ms.service: static-web-apps
ms.topic: how-to
ms.date: 05/08/2020
ms.author: buhollan
ms.custom: devx-track-js
ms.openlocfilehash: 4d6dae8a4f4ed83af3103e95e711bacdb62cf522
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91326173"
---
# <a name="set-up-local-development-for-azure-static-web-apps-preview"></a>Настройка локальной разработки для Статических веб-приложений Azure (предварительная версия)

Экземпляр Статических веб-приложений Azure состоит из двух различных типов приложений. Первый — это веб-приложение для статического содержимого. Веб-приложения часто создаются с помощью интерфейсных платформ и библиотек или генераторов статических сайтов. Второй аспект — это API, который представляет собой приложение "Функции Azure", которое предоставляет полнофункциональную серверную среду разработки.

При работе в облаке Статические веб-приложения Azure легко сопоставляют запросы к маршруту `api` из веб-приложения в приложение "Функции Azure" без необходимости конфигурации CORS. Вам необходимо локально настроить приложение, чтобы имитировать такое поведение.

Эта статья содержит рекомендации для локальной разработки, включая следующие концепции:

- Настройка веб-приложения для статического содержимого.
- Настройка приложения "Функции Azure" для API приложения.
- Отладка и запуск приложения.
- Рекомендации по структуре файлов и папок приложения.

## <a name="prerequisites"></a>Предварительные требования

- [Visual Studio Code](https://code.visualstudio.com/)
- [Расширение "Функции Azure"](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) для Visual Studio Code.
- [Расширение Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) для Visual Studio Code.
  - Требуется только в том случае, если вы не используете интерфейсную платформу JavaScript или CLI генератора статических сайтов.

## <a name="run-projects-locally"></a>Локальный запуск проектов

Локальный запуск статического веб-приложения Azure состоит из трех процессов, в зависимости от того, содержит ли проект API.

- Запуск локального веб-сервера.
- Запуск API.
- Подключение веб-проекта к API.

В зависимости от того, как создается веб-сайт, для запуска приложения в браузере может потребоваться (или не потребоваться) локальный веб-сервер. При использовании интерфейсных платформ JavaScript и генераторов статических сайтов такая функциональность встроена в соответствующие CLI (интерфейсы командной строки). Следующие ссылки указывают на справочник по CLI для выбора платформ, библиотек и генераторов.

### <a name="javascript-frameworks-and-libraries"></a>Платформы и библиотеки JavaScript

- [Angular CLI](https://angular.io/cli);
- [Vue CLI](https://cli.vuejs.org/guide/creating-a-project.html);
- [React CLI](https://create-react-app.dev/).

### <a name="static-site-generators"></a>Генераторы статических сайтов

- [Gatsby CLI](https://www.gatsbyjs.org/docs/gatsby-cli/);
- [Hugo](https://gohugo.io/getting-started/quick-start/);
- [Jekyll](https://jekyllrb.com/docs/usage/).

Если вы используете средство CLI для обслуживания сайта, можете перейти к разделу [Локальный запуск API](#run-api-locally).

### <a name="running-a-local-web-server-with-live-server"></a>Запуск локального веб-сервера с помощью Live Server

Расширение Live Server для Visual Studio Code предоставляет локальный веб-сервер разработки, который обслуживает статическое содержимое.

#### <a name="create-a-repository"></a>Создание репозитория

1. Убедитесь, что вы вошли в GitHub и, перейдите к нему [https://github.com/staticwebdev/vanilla-api/generate](https://github.com/staticwebdev/vanilla-api/generate) и создайте новый проект GitHub с именем **обычный-API**, используя этот шаблон.

    :::image type="content" source="media/local-development/vanilla-api.png" alt-text="Окно нового репозитория GitHub":::

1. Откройте Visual Studio Code.

1. Нажмите клавишу **F1**, чтобы отобразить палитру команд.

1. Введите **clone** (клон) в поле поиска и затем выберите вариант **Git: Clone** (Git: клон).

    :::image type="content" source="media/local-development/command-palette-git-clone.png" alt-text="параметр Git: Clone (Git: клон) в Visual Studio Code":::

1. Введите следующее значение для **URL-адреса репозитория**.

   ```http
   git@github.com:<YOUR_GITHUB_ACCOUNT>/vanilla-api.git
   ```

1. Выберите расположение папки для нового проекта.

1. При появлении запроса на открытие клонированного репозитория выберите **Открыть**.

    :::image type="content" source="media/local-development/open-new-window.png" alt-text="Открытие в новом окне":::

Visual Studio Code открывает клонированный проект в редакторе.

### <a name="run-the-website-locally-with-live-server"></a>Локальный запуск веб-сайта с помощью Live Server

1. Нажмите клавишу **F1**, чтобы отобразить палитру команд.

1. Введите **Live Server** в поле поиска, а затем выберите вариант **Live Server: Open with Live Server** (Live Server: открыть с помощью Live Server).

    На вкладке браузера откроется приложение.

    :::image type="content" source="media/local-development/vanilla-api-site.png" alt-text="Простой статический сайт, выполняющийся в браузере":::

    Это приложение выполняет HTTP-запрос к конечной точке `api/message`. Сейчас этот запрос завершается сбоем, так как необходимо запустить API компонент этого приложения.

### <a name="run-api-locally"></a>Локальный запуск API

Интерфейсы API Статических веб-приложений Azure работают на основе Функций Azure. Дополнительные сведения о добавлении API в проект Статических веб-приложений Azure см. в [этой статье](add-api.md).

В рамках процесса создания API создается конфигурация запуска для Visual Studio Code. Эта конфигурация хранится в папке _.vscode_. Эта папка содержит все необходимые параметры для локального построения и запуска API.

1. В Visual Studio Code нажмите клавишу **F5**, чтобы запустить API.

1. Откроется новый экземпляр терминала, в котором отобразятся выходные данные процесса сборки API.

    :::image type="content" source="media/local-development/terminal-api-debug.png" alt-text="API, запущенный в терминале Visual Studio Code":::

   Строка состояния Visual Studio Code теперь оранжевая. Этот цвет указывает, что API выполняется и отладчик присоединен.

1. Затем нажмите клавишу **Ctrl/Cmd** и щелкните URL-адрес в окне терминала, чтобы открыть окно браузера, вызывающее API.

    :::image type="content" source="media/local-development/hello-from-api-endpoint.png" alt-text="Отображение результата вызова API в браузере":::

### <a name="debugging-the-api"></a>Отладка API

1. Откройте файл _api/GetMessage/index.js_ в Visual Studio Code.

1. Щелкните в левом поле строку 2, чтобы задать точку останова. Появится красная точка, указывающая, что точка останова задана.

    :::image type="content" source="media/local-development/breakpoint-set.png" alt-text="Точка останова в Visual Studio Code":::

1. В браузере обновите страницу, запущенную по адресу <http://127.0.0.1:7071/api/message>.

1. При достижении точки останова в Visual Studio Code выполнение программы приостанавливается.

   :::image type="content" source="media/local-development/breakpoint-hit.png" alt-text="Достижение точки останова в Visual Studio Code":::

   Полный [процесс отладки доступен в Visual Studio Code](https://code.visualstudio.com/Docs/editor/debugging) для API.

1. Нажмите кнопку **Продолжить** на панели отладки, чтобы продолжить выполнение.

    :::image type="content" source="media/local-development/continue-button.png" alt-text="Кнопка &quot;Продолжить&quot; в Visual Studio Code":::

### <a name="calling-the-api-from-the-application"></a>Вызов API из приложения

При развертывании Статические веб-приложения Azure автоматически сопоставляют запросы с конечными точками в папке _api_. Это сопоставление гарантирует, что запросы от приложения к API должны выглядеть так, как показано в следующем примере.

```javascript
let response = await fetch("/api/message");
```

В зависимости от того, создано ли приложение с помощью интерфейса командной строки платформы JavaScript или нет, существуют два способа для настройки пути маршрута `api` при локальном выполнении приложения.

- Файлы конфигурации среды (рекомендуется для платформ и библиотек JavaScript).
- Локальный прокси-сервер

### <a name="environment-configuration-files"></a>Файлы конфигурации среды

Если вы создаете приложение с помощью интерфейсных платформ с CLI, используйте файлы конфигурации среды. Каждая платформа или библиотека имеет другой способ обработки этих файлов конфигурации среды. Обычно существует файл конфигурации для разработки, который используется, если приложение выполняется локально, и файл для рабочей среды, который используется при выполнении приложения в рабочей среде. CLI для платформы JavaScript или генератора статических сайтов, который вы используете, автоматически узнает об использовании файла разработки локально и рабочего файла при создании приложения с помощью Статических веб-приложений Azure.

В файле конфигурации разработки можно указать путь к API, который указывает на локальное расположение `http:127.0.0.1:7071`, где API для вашего сайта запускается локально.

```
API=http:127.0.0.1:7071/api
```

В рабочем файле конфигурации укажите путь к API в виде `api`. Таким образом, приложение будет вызывать API через yoursite.com/api при запуске в рабочей среде.

```
API=api
```

На эти значения конфигурации можно ссылаться как на переменные среды Node в JavaScript веб-приложения.

```js
let response = await fetch(`${process.env.API}/message`);
```

Если интерфейс командной строки используется, чтобы запустить сайт в режиме разработки или создать сайт для рабочей среды, значение `process.env.API` заменяется значением из соответствующего файла конфигурации.

Дополнительные сведения о настройке файлов среды для интерфейсных платформ и библиотек JavaScript см. в следующих статьях:

- [Создание и обслуживание приложений Angular](https://angular.io/guide/build#configuring-application-environments)
- [Добавление пользовательских переменных сред в React](https://create-react-app.dev/docs/adding-custom-environment-variables/)
- [Режимы и переменные среды в Vue](https://cli.vuejs.org/guide/mode-and-env.html)

[!INCLUDE [static-web-apps-local-proxy](../../includes/static-web-apps-local-proxy.md)]

##### <a name="restart-live-server"></a>Перезапуск Live Server

1. В Visual Studio Code нажмите клавишу **F1**, чтобы открыть палитру команд.

1. Введите **Live Server** и выберите вариант **Live Server: Stop Live Server** (Live Server: остановить Live Server).

    :::image type="content" source="media/local-development/stop-live-server.png" alt-text="Команда остановки Live Server в командной палитре Visual Studio":::

1. Нажмите клавишу **F1**, чтобы отобразить палитру команд.

1. Введите **Live Server** и выберите вариант **Live Server: Open with Live Server** (Live Server: открыть с помощью Live Server).

1. Обновите приложение, выполняемое по адресу `http://locahost:3000`. Теперь браузер отображает сообщение, возвращенное из API.

    :::image type="content" source="media/local-development/hello-from-api.png" alt-text="Отображаемое сообщение в браузере: Hello from API":::

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Настройка параметров приложения](application-settings.md)
