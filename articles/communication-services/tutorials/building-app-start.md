---
title: Руководство. Подготовка веб-приложения для Служб коммуникации Azure (Node.js)
titleSuffix: An Azure Communication Services tutorial
description: Узнайте, как создать базовое веб-приложение, которое поддерживает Службы коммуникации Azure.
author: nmurav
services: azure-communication-services
ms.author: nmurav
ms.date: 03/10/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: a5a23d6a06c8cdff4deabac5251597b7ffe0c833
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105728051"
---
# <a name="tutorial-prepare-a-web-app-for-azure-communication-services-nodejs"></a>Руководство. Подготовка веб-приложения для Служб коммуникации Azure (Node.js)

Службы коммуникации Azure можно использовать, добавлять в приложения средства коммуникации в реальном времени. В этом руководстве показано, как настроить веб-приложение, которое поддерживает Службы коммуникации Azure. Это вводное руководство для разработчиков, желающих ознакомиться с возможностями коммуникаций в реальном времени.

Завершив работу с этим руководством, вы получите базовое веб-приложение, настроенное с использованием пакетов SDK Служб коммуникации Azure. Затем такое приложение можно будет использовать для создания решения для коммуникации в режиме реального времени.

Вы можете посетить [страницу Служб коммуникации Azure на сайте GitHub](https://github.com/Azure/communication) и оставить там свой отзыв.

В этом руководстве описано следующее:
> [!div class="checklist"]
> * настройка среды разработки;
> * настройка локального веб-сервера;
> * добавление пакетов Служб коммуникации Azure на веб-сайт;
> * публикация веб-сайта на статических веб-сайтах Azure.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. Дополнительные сведения см. на странице [Создайте бесплатную учетную запись Azure уже сегодня](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). Бесплатная учетная запись предусматривает предоставление 200 долл. США в качестве денег на счете в Azure, что позволяет испытать в работе любое сочетание служб.
- [Visual Studio Code](https://code.visualstudio.com/) для редактирования кода в локальной среде разработки.
- [webpack](https://webpack.js.org/) для создания пакета кода и его размещения в локальной среде.
- [Node.js](https://nodejs.org/en/) для установки зависимостей и управления ими, например для пакетов SDK и webpack Служб коммуникации Azure.
- [nvm и npm](/windows/nodejs/setup-on-windows) для управления версиями.
- [Расширение службы хранилища Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurestorage) для Visual Studio Code. Это расширение требуется для публикации приложения в службе хранилища Azure. [Подробнее о размещении статических веб-сайтов в службе хранилища Azure](../../storage/blobs/storage-blob-static-website.md).
- [Расширение Службы приложений Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice). Это расширение позволяет развертывать веб-сайты с дополнительной возможностью настройки полностью управляемой среды непрерывной поставки и непрерывной интеграции (CI/CD).
- [Расширение Функций Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) для создания собственных бессерверных приложений. Например, вы можете разместить в Функциях Azure приложение проверки подлинности.
- Активный ресурс Служб коммуникации и строка подключения. [Практическое руководство по созданию ресурса Служб коммуникации.](../quickstarts/create-communication-resource.md)
- Маркер доступа пользователя. Инструкции см. в [кратком руководстве по созданию маркеров доступа и управлению ими](../quickstarts/access-tokens.md?pivots=programming-language-javascript), а также в [руководстве по созданию доверенной службы проверки подлинности](./trusted-service-tutorial.md).


## <a name="configure-your-development-environment"></a>Настройка среды разработки

Локальная среда разработки будет иметь следующую конфигурацию:

:::image type="content" source="./media/step-one-pic-one.png" alt-text="Экран &quot;Схема, иллюстрирующая архитектуру среды разработки&quot;.":::


### <a name="install-nodejs-nvm-and-npm"></a>Установка Node.js, а также nvm и npm

Мы будем использовать Node.js, чтобы скачивать и устанавливать зависимости, требуемые для работы приложения на стороне клиента. Мы применим это решение для создания статических файлов, которые затем разместим в Azure, поэтому вам можно не беспокоиться о настройке на стороне сервера.

Разработчики Windows могут воспользоваться [этим руководством по Node.js](/windows/nodejs/setup-on-windows), чтобы настроить Node, nvm и npm. 

Это руководство основано на версии LTS 12.20.0. После установки nvm выполните следующую команду PowerShell, чтобы развернуть нужную версию:

```PowerShell
nvm list available
nvm install 12.20.0
nvm use 12.20.0
```

:::image type="content" source="./media/step-one-pic-two.png" alt-text="Экран &quot;Снимок экрана, на котором показаны команды для развертывания версии Node&quot;.":::

### <a name="configure-visual-studio-code"></a>Настройка Visual Studio Code

Вы можете скачать [Visual Studio Code](https://code.visualstudio.com/) для любой из [поддерживаемых платформ](https://code.visualstudio.com/docs/supporting/requirements#_platforms).

### <a name="create-a-workspace-for-your-azure-communication-services-projects"></a>Создание рабочей области для проектов Служб коммуникации Azure

Создайте папку для хранения файлов проекта, например: `C:\Users\Documents\ACS\CallingApp`. В Visual Studio Code выберите **Файл** > **Add Folder to Workspace** (Добавить папку в рабочую область), а затем добавьте созданную папку в рабочую область.

:::image type="content" source="./media/step-one-pic-three.png" alt-text="Экран &quot;Снимок экрана, на котором показаны варианты добавления файла в рабочую область&quot;.":::

На панели слева откройте **обозреватель**. Здесь вы увидите созданную папку `CallingApp` в рабочей области **БЕЗ ИМЕНИ**.

:::image type="content" source="./media/step-one-pic-four.png" alt-text="Экран &quot;Снимок экрана: обозреватель и рабочая область &quot;Без имени&quot;.":::

Вы можете указать для рабочей области любое имя. Чтобы проверить версию Node.js, щелкните правой кнопкой мыши папку `CallingApp` и выберите **Open in Integrated Terminal** (Открыть во встроенном терминале).

:::image type="content" source="./media/step-one-pic-five.png" alt-text="Экран &quot;Снимок экрана: выбор пункта для открытия папки во встроенном терминале&quot;.":::

В окне терминала введите следующую команду, чтобы проверить версию Node.js, которую мы установили на предыдущем шаге:

```JavaScript
node --version
```

:::image type="content" source="./media/step-one-pic-six.png" alt-text="Экран &quot;Снимок экрана: проверка версии узла&quot;.":::

### <a name="install-azure-extensions-for-visual-studio-code"></a>Установка расширений Azure для Visual Studio Code

Установите [расширение службы хранилища Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurestorage) из Visual Studio Marketplace или прямо в Visual Studio Code (**Представление** > **Расширения** > **Служба хранилища Azure**).

:::image type="content" source="./media/step-one-pic-seven.png" alt-text="Экран &quot;Снимок экрана: кнопка для установки расширения службы хранилища Azure&quot;.":::

Выполните эти же действия для расширений [Функций Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) и [Службы приложений Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice).


## <a name="set-up-a-local-web-server"></a>Настройка локального веб-сервера

### <a name="create-an-npm-package"></a>Создание пакета npm

В окне терминала перейдите в папку рабочей области и введите следующую команду:

``` console
npm init -y
```

Эта команда инициализирует новый пакет npm и добавляет `package.json` в корневую папку проекта.

:::image type="content" source="./media/step-one-pic-eight.png" alt-text="Экран &quot;Снимок экрана: пакет JSON&quot;.":::

Дополнительную документацию по `npm init`см. на [странице документации по npm для этой команды](https://docs.npmjs.com/cli/v6/commands/npm-init).

### <a name="install-webpack"></a>Установка webpack

Вы можете использовать [webpack](https://webpack.js.org/) для упаковывания кода в статические файлы, которые можно развернуть в Azure. Это решение включает в себя собственный сервер разработки, который можно настроить для работы с примером вызова.

В терминале введите следующую команду, чтобы установить webpack:

``` Console
npm install webpack@4.42.0 webpack-cli@3.3.11 webpack-dev-server@3.10.3 --save-dev
```

Это руководство протестировано с версиями, указанными в предыдущей команде. Указание `-dev` сообщает диспетчеру пакетов, что эта зависимость предназначена для разработки и ее не нужно добавлять в код, развертываемый в Azure.

Вы увидите, что в файл `package.json` в раздел `devDependencies` добавлены два новых пакета. Эти пакеты будут установлены в каталог `./CallingApp/node_modules/`.

:::image type="content" source="./media/step-one-pic-ten.png" alt-text="Экран &quot;Снимок экрана: конфигурация webpack&quot;.":::

### <a name="configure-the-development-server"></a>Настройка сервера разработки

При запуске в браузере статического приложения (например, файла `index.html`) используется протокол `file://`. Чтобы модули npm работали правильно, нужно использовать протокол HTTP, настроив webpack в качестве локального сервера разработки.

Вы создадите две конфигурации: одну для разработки и одну для рабочей среды. При подготовке для рабочей среды файлы уменьшаются, то есть из них удаляются все неиспользуемые пробелы и символы. Эта конфигурация полезна для рабочих сценариев, в которых нужно обеспечить минимальную задержку или замаскировать код.

Средство `webpack-merge` следует применить для работы с [разными файлами конфигурации для webpack](https://webpack.js.org/guides/production/).

Сначала давайте настроим среду разработки. Сначала необходимо установить `webpack merge`. В терминале выполните приведенную ниже команду.

```Console
npm install --save-dev webpack-merge
```

Вы увидите, что в файл `package.json` в раздел `devDependencies` добавлена еще одна зависимость.

Затем создайте новый файл `webpack.common.js` и добавьте в него следующий код:

```JavaScript
const path = require('path');
module.exports ={
    entry: './app.js',
    output: {
        filename:'app.js',
        path: path.resolve(__dirname, 'dist'),
    }
}
```

Затем следует добавить еще два файла, по одному для каждой конфигурации:

* `webpack.dev.js`
* `webpack.prod.js`

Теперь измените файл `webpack.dev.js`, добавив к нему следующий код:

```JavaScript
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
    mode: 'development',
    devtool: 'inline-source-map',
});
```
В этой конфигурации общие параметры импортируются из `webpack.common.js`, затем два файла объединяются, устанавливается режим `development`и sourcemap определяется как `inline-source-map`.

Режим разработки означает, что webpack не будет минифицировать файлы и создавать оптимизированные рабочие файлы. Подробную документацию по режимам webpack можно найти на [веб-странице webpack "Режим"](https://webpack.js.org/configuration/mode/).

Параметры sourcemap перечислены на [веб-странице webpack Devtool](https://webpack.js.org/configuration/devtool/#root). Настройка SourceMap упрощает отладку в браузере.

:::image type="content" source="./media/step-one-pic-11.png" alt-text="Экран &quot;Снимок экрана: код для настройки webpack&quot;.":::

Чтобы запустить сервер разработки, перейдите к `package.json` и добавьте следующий код в раздел `scripts`:

```JavaScript
    "build:dev": "webpack-dev-server --config webpack.dev.js"
```

Теперь этот файл будет выглядеть так:

```JavaScript
{
  "name": "CallingApp",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build:dev": "webpack-dev-server --config webpack.dev.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^4.42.0",
    "webpack-cli": "^3.3.11",
    "webpack-dev-server": "^3.10.3"
  }
}
```

Вы добавили команду, которую можно использовать из npm.

:::image type="content" source="./media/step-one-pic-12.png" alt-text="Экран &quot;Снимок экрана: изменение package.js&quot;.":::

### <a name="test-the-development-server"></a>Тестирование сервера разработки

 Создайте в Visual Studio Code три файла в папке проекта:

* `index.html`
* `app.js`
* `app.css` (необязательно, для задания стиля приложения)

Вставьте этот код в `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My first ACS application</title>
    <link rel="stylesheet" href="./app.css"/>
    <script src="./app.js" defer></script>
</head>
<body>
    <h1>Hello from ACS!</h1>
</body>
</html>
```
:::image type="content" source="./media/step-one-pic-13.png" alt-text="Экран &quot;Снимок экрана: HTML-файл&quot;.":::

Добавьте в `app.js` следующий код:

```JavaScript
alert('Hello world alert!');
console.log('Hello world console!');
```
Добавьте в `app.css` следующий код:

```CSS
html {
    font-family: sans-serif;
  }
```
Не забудьте сохранить! Несохраненный файл будет обозначен белой точкой рядом с именем файла в Explorer.

:::image type="content" source="./media/step-one-pic-14.png" alt-text="Экран &quot;Снимок экрана: файл App.js с кодом JavaScript&quot;.":::

Открыв эту страницу, вы увидите сообщение с оповещением в консоли браузера.

:::image type="content" source="./media/step-one-pic-15.png" alt-text="Экран &quot;Снимок экрана: файл App.css&quot;.":::

Выполните в терминале следующую команду, чтобы протестировать конфигурацию разработки:

```Console
npm run build:dev
```

Консоль отобразит сообщение о том, какой у работающего сервера адрес. По умолчанию это `http://localhost:8080`. `build:dev` — это команда, которую вы ранее добавили в `package.json`.

:::image type="content" source="./media/step-one-pic-16.png" alt-text="Экран &quot;Снимок экрана: запуск сервера разработки&quot;.":::
 
Перейдите в браузере по адресу сервера и убедитесь, что открываются ранее настроенные страница и оповещение.
 
:::image type="content" source="./media/step-one-pic-17.png" alt-text="Экран &quot;Скриншот HTML-страницы&quot;.":::
  
 
Пока сервер работает, вы можете внести любые изменения в код и сервер. HTML-страница автоматически перезагрузится. 

Затем перейдите к файлу `app.js` в Visual Studio Code и удалите `alert('Hello world alert!');`. Сохраните этот файл и убедитесь, что оповещение исчезнет из браузера.

Чтобы остановить сервер, выполните `Ctrl+C` в терминале. Чтобы запустить сервер в любое время, введите `npm run build:dev`.

## <a name="add-the-azure-communication-services-packages"></a>Добавление пакетов Служб коммуникации Azure

Используйте команду `npm install`, чтобы установить пакет SDK Служб коммуникации Azure для реализации вызовов на JavaScript.

```Console
npm install @azure/communication-common --save
npm install @azure/communication-calling --save
```

Это действие добавляет общие пакеты и пакеты вызовов Служб коммуникации Azure в качестве зависимостей пакета. Вы увидите,что в файл `package.json` добавлены два новых пакета. Дополнительные сведения о `npm install` см. на [странице документации по npm для этой команды](https://docs.npmjs.com/cli/v6/commands/npm-install)

:::image type="content" source="./media/step-one-pic-nine.png" alt-text="Экран &quot;Снимок экрана: код для установки пакетов Служб коммуникации Azure&quot;.":::

Эти пакеты предоставляются командой разработчиков Служб коммуникации Azure и содержат библиотеки проверки подлинности и вызова. Команда `--save` сообщает, что приложение зависит от этих пакетов в рабочей среде, а значит их нужно включить в `devDependencies` файла `package.json`. Теперь при сборке приложения для рабочей среды указанные пакеты будут добавлены в рабочий код.


## <a name="publish-your-website-to-azure-static-websites"></a>Публикация веб-сайта на статических веб-сайтах Azure

### <a name="create-a-configuration-for-production-deployment"></a>Создание конфигурации для развертывания в рабочей среде

Добавьте в `webpack.prod.js` следующий код:

```JavaScript
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
  mode: 'production',
});
```

Эта конфигурация будет объединена с `webpack.common.js` (где вы указали входной файл и где сохраняются результаты). В конфигурации также будет задан режим `production`.
 
В классе `package.json` добавьте следующий код.

```JavaScript
"build:prod": "webpack --config webpack.prod.js"
```

Файл должен выглядеть следующим образом.

```JavaScript
{
  "name": "CallingApp",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build:dev": "webpack-dev-server --config webpack.dev.js",
    "build:prod": "webpack --config webpack.prod.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "@azure/communication-calling": "^1.0.0-beta.6",
    "@azure/communication-common": "^1.0.0"
  },
  "devDependencies": {
    "webpack": "^4.42.0",
    "webpack-cli": "^3.3.11",
    "webpack-dev-server": "^3.10.3",
    "webpack-merge": "^5.7.3"
  }
}
```

:::image type="content" source="./media/step-one-pic-20.png" alt-text="Экран &quot;Снимок экрана: настроенные файлы&quot;.":::


В окне терминала выполните следующую команду:

```Console
npm run build:prod
```

Эта команда создает папку `dist` с готовым к работе статическим файлом `app.js`. 

:::image type="content" source="./media/step-one-pic-21.png" alt-text="Экран &quot;Снимок экрана: производственная сборка&quot;.":::
 
 
### <a name="deploy-your-app-to-azure-storage"></a>Развертывание приложения в службе хранилища Azure

Скопируйте `index.html` и `app.css` в папку `dist`.

В папке `dist` создайте файл с именем `404.html`. Скопируйте в этот файл следующий код разметки:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="./app.css"/>
    <title>Document</title>
</head>
<body>
    <h1>The page does not exists.</h1>
</body>
</html>
```

Сохраните файл (CONTROL + S).

Щелкните папку `dist` правой кнопкой мыши и выберите вариант **Deploy to Static Website via Azure Storage** (Развертывание на статическом веб-сайте через службу хранилища Azure).

:::image type="content" source="./media/step-one-pic-22.png" alt-text="Экран &quot;Снимок экрана: выбранные параметры для начала развертывания в Azure&quot;.":::
 
В разделе **Выбор подписки** выберите **Войти в Azure** (или **Создать бесплатную учетную запись Azure**, если вы еще не создали подписку).
 
:::image type="content" source="./media/step-one-pic-23.png" alt-text="Экран &quot;Снимок экран: выбранные параметры для входа в Azure&quot;.":::
 
Выберите **Создать новую учетную запись хранения** > **Дополнительно**.

:::image type="content" source="./media/step-one-pic-24.png" alt-text="Экран &quot;Снимок экрана: выбранные параметры для создания группы учетных записей хранения&quot;.":::
 
Укажите имя группы для хранилища:
 
:::image type="content" source="./media/step-one-pic-25.png" alt-text="Экран &quot;Снимок экрана: указание имени для учетной записи&quot;.":::
 
Если нужно, создайте группу ресурсов:
 
:::image type="content" source="./media/step-one-pic-26.png" alt-text="Экран &quot;Снимок экрана: выбор для создания группы ресурсов&quot;.":::
  
Для параметра **Would you like to enable static website hosting?** (Включить размещение статических веб-сайтов) выберите вариант **Да**.

:::image type="content" source="./media/step-one-pic-27.png" alt-text="Экран &quot;Снимок экрана: выбор параметра для включения размещения статических веб-сайтов&quot;.":::
  
В поле **Enter the index document name** (Введите имя документа индекса) примите имя файла по умолчанию. Вы уже создали файл `index.html`.

В поле **Enter the 404 error document path** (Введите путь к документу с сообщением об ошибке 404) введите **404.html**.  
  
Выберите расположение приложения. Это расположение определяет, какой обработчик мультимедиа будет в будущем использоваться вызывающим приложением при групповых вызовах. 

Службы коммуникации Azure выбирают обработчик мультимедиа в зависимости от расположения приложения.

:::image type="content" source="./media/step-one-pic-28.png" alt-text="Экран &quot;Снимок экрана: список расположений&quot;.":::
  
Дождитесь, пока завершится создание ресурса и веб-сайта. 
 
Выберите **Browse to website** (Перейти к веб-сайту).

:::image type="content" source="./media/step-one-pic-29.png" alt-text="Экран &quot;Снимок экрана: сообщение о завершении развертывания с кнопкой перехода на веб-сайт&quot;.":::
 
С помощью средств разработки в браузере вы можете проверить исходный код и просмотреть файл, подготовленный для рабочей среды.
 
:::image type="content" source="./media/step-one-pic-30.png" alt-text="Экран &quot;Снимок экрана: источник веб-сайта с файлом&quot;.":::

Перейдите на [портал Azure](https://portal.azure.com/#home), выберите созданные группу ресурсов и приложение. Затем выберите **Параметры** > **Статический веб-сайт**. Вы увидите, что статические веб-сайты включены. Обратите внимание на основную конечную точку, имя документа индекса, а также путь к документу с сообщением об ошибке.

:::image type="content" source="./media/step-one-pic-31.png" alt-text="Экран &quot;Снимок экрана: выбор статического веб-сайта&quot;.":::

Выберите элемент **Контейнеры** в разделе **Служба BLOB-объектов**. В списке показаны два созданных контейнера: один для журналов (`$logs`) и один для содержимого вашего веб-сайта (`$web`).

:::image type="content" source="./media/step-one-pic-32.png" alt-text="Экран &quot;Снимок экрана: конфигурация контейнера&quot;.":::

Если вы откроете контейнер `$web`, то увидите файлы, которые были ранее созданы в Visual Studio и развернуты в Azure. 

:::image type="content" source="./media/step-one-pic-33.png" alt-text="Экран &quot;Снимок экрана: файлы, развернутые в Azure&quot;.":::

Приложение можно в любой момент повторно развернуть из Visual Studio Code.

Теперь вы готовы к созданию первого веб-приложения Служб коммуникации Azure.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Добавление функции голосового вызова в приложение](../quickstarts/voice-video-calling/getting-started-with-calling.md)

Кроме того, вам может понадобиться следующее:

- [Добавление чата в приложение](../quickstarts/chat/get-started.md)
- [Создание маркеров доступа пользователей](../quickstarts/access-tokens.md)
- [Сведения об архитектуре клиент-сервер](../concepts/client-and-server-architecture.md)
- [Сведения о проверке подлинности](../concepts/authentication.md)
