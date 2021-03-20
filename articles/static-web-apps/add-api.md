---
title: Добавление API в Статические веб-приложения Azure с помощью Функций Azure
description: Руководство по началу работы со Статическими веб-приложениями Azure с добавлением бессерверного API в Статическое веб-приложение с помощью Функций Azure.
services: static-web-apps
author: manekinekko
ms.service: static-web-apps
ms.topic: how-to
ms.date: 05/29/2020
ms.author: wachegha
ms.custom: devx-track-js
ms.openlocfilehash: 8a42ca4cf69515c99bb2e0bd28bb62dfed0e18f8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100650695"
---
# <a name="add-an-api-to-azure-static-web-apps-preview-with-azure-functions"></a>Добавление API в предварительную версию Статистического веб-приложения Azure с помощью Функций Azure

Вы можете добавить бессерверные API в Статические веб-приложения Azure с помощью интеграции с Функциями Azure. В этой статье показано, как добавить и развернуть API на сайте Статических веб-приложений Azure.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской.
  - Если у вас нет учетной записи, вы можете [создать ее бесплатно](https://azure.microsoft.com/free).
- [Visual Studio Code](https://code.visualstudio.com/)
- [Расширение "Функции Azure"](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) для Visual Studio Code.
- Расширение [Live Server Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer).
- [Node.js](https://nodejs.org/download/) для локального запуска приложения API

## <a name="create-a-git-repository"></a>Создание репозитория Git

В следующих шагах показано, как создать репозиторий и клонировать файлы на компьютер.

1. Убедитесь, что вы вошли в GitHub и, перейдите к, чтобы [https://github.com/staticwebdev/vanilla-basic/generate](https://github.com/login?return_to=/staticwebdev/vanilla-basic/generate) создать новый репозиторий.
1. В поле _Имя репозитория_ введите **my-vanilla-api**.
1. Щелкните **Create repository from template** (Создание репозитория из шаблона).

   :::image type="content" source="media/add-api/create-repository.png" alt-text="Создание репозитория из vanilla-basic":::

После создания проекта скопируйте URL-адрес в браузере для нового репозитория. Этот URL-адрес используется в Visual Studio Code для клонирования репозитория Git.

1. Нажмите клавишу **F1**, чтобы открыть команду в палитре команд.
1. Вставьте URL-адрес в запрос _Git: Clone_ и нажмите клавишу **ВВОД**.

   :::image type="content" source="media/add-api/vscode-git-0.png" alt-text="Клонирование проекта GitHub с помощью Visual Studio Code":::

    Следуйте инструкциям на экране, чтобы выбрать расположение репозитория для клонирования проекта.

## <a name="create-the-api"></a>Создание API

Далее вы создадите проект функций Azure в качестве API приложения. 

1. В проекте _my-vanilla-api_ создайте вложенную папку с именем **api**.
1. Нажмите клавишу **F1**, чтобы отобразить палитру команд.
1. Введите **Azure Functions: Create New Project…** (Функции Azure: Создать новый проект…).
1. Нажмите клавишу **ВВОД**.
1. Щелкните **Обзор**.
1. Выберите папку **api** в качестве каталога для рабочей области проекта.
1. Щелкните **Выбрать**.

   :::image type="content" source="media/add-api/create-azure-functions-vscode-1.png" alt-text="На снимке экрана показана папка «P I» и кнопка «выбрать».":::

1. Введите следующие сведения по соответствующим запросам:

    - _Выбор языка_: выбор **JavaScript**
    - _Выберите шаблон для первой функции вашего проекта_. Выберите **триггер HTTP**.
    - _Укажите имя функции_: введите- **сообщение**
    - _Уровень авторизации_: выберите **Анонимный**, что позволит любому пользователю вызывать конечную точку функции.
        - Дополнительные сведения об уровнях авторизации см. в разделе [Ключи авторизации](../azure-functions/functions-bindings-http-webhook-trigger.md#authorization-keys).

Visual Studio Code создает проект функций Azure с функцией, активируемой HTTP.

Теперь приложение имеет структуру проекта, аналогичную приведенной ниже.

```files
├── api
│   ├── GetMessage
│   │   ├── function.json
│   │   ├── index.js
│   │   └── sample.dat
│   ├── host.json
│   ├── local.settings.json
│   ├── package.json
│   └── proxies.json
├── index.html
├── readme.md
└── styles.css
```

Далее вы измените `GetMessage` функцию, чтобы она возвращала сообщение на внешний интерфейс.

1. Обновите `GetMessage` функцию в разделе _API/-message/index.js_ следующим кодом.

    ```javascript
    module.exports = async function (context, req) {
      context.res = {
        body: {
          text: "Hello from the API"
        }
      };
    };
    ```

1. Обновите конфигурацию `GetMessage` в разделе `api/GetMessage/function.json` с помощью следующих параметров.

    ```json
    {
      "bindings": [
        {
          "authLevel": "anonymous",
          "type": "httpTrigger",
          "direction": "in",
          "name": "req",
          "methods": [
            "get"
          ],
          "route": "message"
        },
        {
          "type": "http",
          "direction": "out",
          "name": "res"
        }
      ]
    }
    ```

В указанных выше параметрах конечная точка API:

- Активируется при выполнении HTTP-запроса к функции
- доступна для всех запросов независимо от состояния проверки подлинности;
- предоставляется через маршрут _/api/message_.

## <a name="run-the-api-locally"></a>Запуск API в локальной среде

Visual Studio Code интегрируется с [Azure Functions Core Tools](../azure-functions/functions-run-local.md), чтобы перед публикацией в Azure можно было запустить этот проект на локальном компьютере разработки.

> [!TIP]
> Прежде чем продолжать, убедитесь, что все ресурсы, перечисленные в разделе [Предварительные требования](#prerequisites) , установлены.

1. Запустите функцию, нажав клавишу **F5** , чтобы запустить приложение "функции".

1. Если вы еще не установили Azure Functions Core Tools, выберите в запросе **Установить**.

    Основные инструменты показывают выходные данные работающего приложения на панели _терминала_ . В качестве части выходных данных можно увидеть конечную точку URL-адреса функции, активированной с помощью HTTP, которая выполняется локально.

    :::image type="content" source="media/add-api/create-azure-functions-vscode-2.png" alt-text="На снимке экрана показана вкладка терминал, на которой можно увидеть U R L.":::

1. При запуске основных средств перейдите по следующему URL-адресу, чтобы убедиться, что API работает правильно: `http://localhost:7071/api/message` .

   Ответ в браузере должен выглядеть следующим образом:

   :::image type="content" source="media/add-api/create-azure-functions-vscode-3.png" alt-text="Alt Текстскриншот показывает текстовый ответ в браузере.":::

1. Нажмите клавиши **SHIFT + F5** , чтобы прерывать сеанс отладки.

### <a name="call-the-api-from-the-application"></a>Вызов API из приложения

При развертывании в Azure запросы к API автоматически направляются в приложение функций для запросов, отправленных в `api` маршрут. Работая локально, необходимо настроить параметры приложения для запросов прокси-сервера к локальному API.

[!INCLUDE [static-web-apps-local-proxy](../../includes/static-web-apps-local-proxy.md)]

#### <a name="update-html-files-to-access-the-api"></a>Обновление HTML-файлов для доступа к API

1. Теперь обновите содержимое файла _index.html_ следующим кодом, чтобы получить текст из функции API и отобразить его на экране:

   ```html
   <!DOCTYPE html>
   <html lang="en">

   <head>
     <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
     <link rel="stylesheet" href="styles.css">
     <title>Vanilla JavaScript App</title>
   </head>

   <body>
     <main>
       <h1>Vanilla JavaScript App</h1>
       <p>Loading content from the API: <b id="name">...</b></p>
     </main>

     <script>
       (async function() {
         let { text } = await( await fetch(`/api/message`)).json();
         document.querySelector('#name').textContent = text;
       }())
     </script>
   </body>

   </html>
   ```

1. Нажмите клавишу **F5** , чтобы запустить проект API.

1. Нажмите клавишу **F1** и выберите **Live Server: Open with Live Server** (Live Server: открыть с помощью Live Server).

    Теперь сообщение API должно отобразиться на веб-странице.

   :::image type="content" source="media/add-api/create-azure-functions-vscode-4.png" alt-text="На снимке экрана показано сообщение A P I в браузере.":::

   > [!NOTE]
   > Для обслуживания файла `index.html` можно использовать другие серверы HTTP или прокси-серверы. Доступ к `index.html` из `file:///` не будет работать.

1. Нажмите клавиши **SHIFT + F5** , чтобы закрыть проект API.

### <a name="commit-and-push-your-changes-to-github"></a>Фиксация и отправка изменений в GitHub

С помощью Visual Studio Code зафиксируйте и отправьте изменения в удаленный репозиторий Git.

1. Нажмите клавишу **F1**, чтобы отобразить палитру команд.
1. Введите **Git: Commit All** (Git: зафиксировать все).
1. Добавление сообщения о фиксации и нажатие клавиши **Ввод**
1. Нажмите клавишу **F1**
1. Введите в **Git: Push** и нажмите клавишу **Ввод**

## <a name="create-a-static-web-app"></a>Создание статического веб-приложения

1. Перейдите на [портал Azure](https://portal.azure.com).
1. Щелкните **Создать ресурс**.
1. Поиск **статического веб-приложения**
1. Щелкните **статическое веб-приложение (Предварительная версия)** .
1. Нажмите кнопку **Создать**.

Затем добавьте параметры для конкретного приложения.

1. Выберите _подписку Azure_.
1. Выберите или создайте _группу ресурсов_.
1. Присвойте приложению имя **my-vanilla-api**.
1. Выберите ближайший к вам _регион_.
1. Выберите **Бесплатный** в поле _Номер SKU_.
1. Нажмите кнопку **Войти по учетным данным GitHub** и выполните проверку подлинности с помощью GitHub.
1. Выберите предпочитаемую _Организацию_.
1. Выберите **my-vanilla-api** из раскрывающегося списка _Репозиторий_.
1. Выберите **main** в раскрывающемся списке _Ветвь_.
1. Выберите выбранную платформу для конфигурации сборки из раскрывающегося списка " _предустановки сборки_ ".

 > Эти поля будут отражать структуру проекта по умолчанию для типа приложения. Измените значения в соответствии с вашими приложениями.

Затем добавьте следующие сведения о сборке.

1. Введите **/** для _расположения приложения_.
1. Введите **api** в поле _Расположение API_.
1. Удалите значение по умолчанию в _расположении артефакта приложения_, оставив поле пустым.
1. Щелкните **Review + create** (Просмотреть и создать).
1. Нажмите кнопку **Создать**.

    После нажатия кнопки _создать_ Azure выполняет две задачи. Сначала создаются базовые облачные службы для поддержки приложения. Затем начнется создание и развертывание приложения в фоновом процессе.

1. Нажмите кнопку **Перейти к ресурсу** , чтобы перейти на страницу _обзора_ веб-приложения.

    Так как приложение строится в фоновом режиме, можно щелкнуть баннер, содержащий ссылку, чтобы просмотреть состояние сборки.

    :::image type="content" source="media/add-api/github-action-flag.png" alt-text="Рабочий процесс GitHub":::

1. После завершения развертывания подразделение может переходить к веб-приложению, щелкнув ссылку _URL-адрес_ , показанную на странице _Обзор_ .

    :::image type="content" source="media/add-api/static-app-url-from-portal.png" alt-text="Доступ к URL-адресу статического приложения на портале Azure":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не хотите, чтобы это приложение можно было использовать, выполните следующие действия, чтобы удалить Статическое веб-приложение Azure и связанные с ним ресурсы.

1. Перейдите на [портал Azure](https://portal.azure.com).
1. В верхней строке поиска введите **Группы ресурсов**.
1. Щелкните **Группы ресурсов**.
1. Выберите **MyResourceGroup**.
1. На странице _myResourceGroup_ убедитесь, что перечислены те ресурсы, которые нужно удалить.
1. Выберите **Удалить**.
1. Введите **myResourceGroup** в текстовое поле.
1. Выберите команду **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Настройка параметров приложения](./application-settings.md)
