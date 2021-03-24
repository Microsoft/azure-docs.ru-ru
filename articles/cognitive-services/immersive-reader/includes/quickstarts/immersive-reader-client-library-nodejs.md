---
title: 'Клиентская библиотека Node.js иммерсивного средства чтения: краткое руководство'
titleSuffix: Azure Cognitive Services
description: В рамках этого краткого руководства вы создадите веб-приложения с нуля и добавите функции API "Иммерсивное средство чтения".
services: cognitive-services
author: nitinme
manager: nitinme
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: include
ms.date: 09/14/2020
ms.author: nitinme
ms.custom: devx-track-js
ms.openlocfilehash: ffef03457f9d3faf385bd0a924474dfa6efc4121
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102619274"
---
[Иммерсивное средство чтения](https://www.onenote.com/learningtools) — это инклюзивное решение, в котором реализованы проверенные методы, улучшающие понимание текста при чтении у людей, которые учатся читать или изучают язык, а также у людей, которые испытывают определенные трудности при обучении, например, из-за дислексии. Вы можете использовать иммерсивное средство чтения в своих приложениях, чтобы изолировать текст для фокусировки, отображать рисунки, связанные с часто используемыми словами, выделять части речи, читать вслух выделенный текст, переводить слова и фразы в реальном времени и многое другое.

В этом кратком руководстве показано, как создать веб-приложение с нуля и интегрировать иммерсивное средство чтения с помощью соответствующей клиентской библиотеки. Полностью рабочий пример этого краткого руководства доступен [на GitHub](https://github.com/microsoft/immersive-reader-sdk/tree/master/js/samples/quickstart-nodejs).

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services).
* Ресурс "Иммерсивное средство чтения", настроенный для проверки подлинности Azure Active Directory. Инструкции по настройке см. [здесь](../../how-to-create-immersive-reader.md). Вам потребуются некоторые значения, созданные здесь при настройке свойств среды. Сохраните результаты своего сеанса в текстовом файле для использования в будущем.
* [Node.js](https://nodejs.org/) и [Yarn](https://yarnpkg.com).
* Интегрированная среда разработки, такая как [Visual Studio Code](https://code.visualstudio.com/).

## <a name="create-a-nodejs-web-app-with-express"></a>Создание веб-приложения Node.js с помощью средства Express

Создайте веб-приложение Node.js с помощью средства `express-generator`.

```bash
npm install express-generator -g
express --view=pug quickstart-nodejs
cd quickstart-nodejs
```

Установите зависимости yarn и добавьте зависимости `request` и `dotenv`, которые будут использоваться в этом кратком руководстве позже.

```bash
yarn
yarn add request
yarn add dotenv
```

## <a name="set-up-authentication"></a>Настройка проверки подлинности

### <a name="configure-authentication-values"></a>Настройка значений проверки подлинности

Создайте новый файл с именем _.env_ в корневом каталоге проекта. Вставьте в него следующий код, указав значения, заданные при создании ресурса "Иммерсивное средство чтения".
Не включайте кавычки или символы "{" и "}".

```text
TENANT_ID={YOUR_TENANT_ID}
CLIENT_ID={YOUR_CLIENT_ID}
CLIENT_SECRET={YOUR_CLIENT_SECRET}
SUBDOMAIN={YOUR_SUBDOMAIN}
```

Не забудьте зафиксировать этот файл в системе управления версиями, так как он содержит секреты, для которых не следует предоставлять открытый доступ.

Затем откройте файл _app.js_ и добавьте следующее в его начало. В Node загрузятся свойства, указанные в файле .env в качестве переменных среды.

```javascript
require('dotenv').config();
```

### <a name="update-the-router-to-acquire-the-token"></a>Обновление маршрутизатора для получения токена
Откройте файл _routes\index.js_ и замените автоматически созданный код приведенным ниже кодом.

Этот код создает конечную точку API, которая получает токен проверки подлинности Azure AD, используя пароль субъект-службы. Кроме того, этот код возвращает дочерний домен, а также объект, содержащий токен и поддомен.

```javascript
var express = require('express');
var router = express.Router();
var request = require('request');

/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('index', { title: 'Express' });
});

router.get('/GetTokenAndSubdomain', function(req, res) {
    try {
        request.post({
            headers: {
                'content-type': 'application/x-www-form-urlencoded'
            },
            url: `https://login.windows.net/${process.env.TENANT_ID}/oauth2/token`,
            form: {
                grant_type: 'client_credentials',
                client_id: process.env.CLIENT_ID,
                client_secret: process.env.CLIENT_SECRET,
                resource: 'https://cognitiveservices.azure.com/'
            }
        },
        function(err, resp, tokenResult) {
            if (err) {
                console.log(err);
                return res.status(500).send('CogSvcs IssueToken error');
            }

            var tokenResultParsed = JSON.parse(tokenResult);

            if (tokenResultParsed.error) {
                console.log(tokenResult);
                return res.send({error :  "Unable to acquire Azure AD token. Check the debugger for more information."})
            }

            var token = tokenResultParsed.access_token;
            var subdomain = process.env.SUBDOMAIN;
            return res.send({token, subdomain});
        });
    } catch (err) {
        console.log(err);
        return res.status(500).send('CogSvcs IssueToken error');
    }
});

module.exports = router;
```

Конечная точка API **GetTokenAndSubdomain** должна быть защищена какой-либо формой аутентификации (например, [OAuth](https://oauth.net/2/)), чтобы предотвратить получение неавторизованными пользователями токенов для использования в службе Иммерсивного средства чтения и выставлении счетов. Эти сведения выходят за рамки этого руководства.

## <a name="add-sample-content"></a>Добавление примеров содержимого

Теперь мы добавим пример содержимого для этого веб-приложения. Откройте файл _views\index.pug_ и замените автоматически созданный код приведенным ниже примером.

```pug
doctype html
html
   head
      title Immersive Reader Quickstart Node.js

      link(rel='stylesheet', href='https://stackpath.bootstrapcdn.com/bootstrap/3.4.1/css/bootstrap.min.css')

      // A polyfill for Promise is needed for IE11 support.
      script(src='https://cdn.jsdelivr.net/npm/promise-polyfill@8/dist/polyfill.min.js')

      script(src='https://contentstorage.onenote.office.net/onenoteltir/immersivereadersdk/immersive-reader-sdk.1.0.0.js')
      script(src='https://code.jquery.com/jquery-3.3.1.min.js')

      style(type="text/css").
        .immersive-reader-button {
          background-color: white;
          margin-top: 5px;
          border: 1px solid black;
          float: right;
        }
   body
      div(class="container")
        button(class="immersive-reader-button" data-button-style="iconAndText" data-locale="en")

        h1(id="ir-title") About Immersive Reader
        div(id="ir-content" lang="en-us")
          p Immersive Reader is a tool that implements proven techniques to improve reading comprehension for emerging readers, language learners, and people with learning differences. The Immersive Reader is designed to make reading more accessible for everyone. The Immersive Reader

            ul
                li Shows content in a minimal reading view
                li Displays pictures of commonly used words
                li Highlights nouns, verbs, adjectives, and adverbs
                li Reads your content out loud to you
                li Translates your content into another language
                li Breaks down words into syllables

          h3 The Immersive Reader is available in many languages.

          p(lang="es-es") El Lector inmersivo está disponible en varios idiomas.
          p(lang="zh-cn") 沉浸式阅读器支持许多语言
          p(lang="de-de") Der plastische Reader ist in vielen Sprachen verfügbar.
          p(lang="ar-eg" dir="rtl" style="text-align:right") يتوفر \"القارئ الشامل\" في العديد من اللغات.

script(type="text/javascript").
  function getTokenAndSubdomainAsync() {
        return new Promise(function (resolve, reject) {
            $.ajax({
                url: "/GetTokenAndSubdomain",
                type: "GET",
                success: function (data) {
                    if (data.error) {
                        reject(data.error);
                    } else {
                        resolve(data);
                    }
                },
                error: function (err) {
                    reject(err);
                }
            });
        });
    }

    $(".immersive-reader-button").click(function () {
        handleLaunchImmersiveReader();
    });

    function handleLaunchImmersiveReader() {
        getTokenAndSubdomainAsync()
            .then(function (response) {
                const token = response["token"];
                const subdomain = response["subdomain"];
                // Learn more about chunk usage and supported MIME types https://docs.microsoft.com/azure/cognitive-services/immersive-reader/reference#chunk
                const data = {
                    title: $("#ir-title").text(),
                    chunks: [{
                        content: $("#ir-content").html(),
                        mimeType: "text/html"
                    }]
                };
                // Learn more about options https://docs.microsoft.com/azure/cognitive-services/immersive-reader/reference#options
                const options = {
                    "onExit": exitCallback,
                    "uiZIndex": 2000
                };
                ImmersiveReader.launchAsync(token, subdomain, data, options)
                    .catch(function (error) {
                        alert("Error in launching the Immersive Reader. Check the console.");
                        console.log(error);
                    });
            })
            .catch(function (error) {
                alert("Error in getting the Immersive Reader token and subdomain. Check the console.");
                console.log(error);
            });
    }

    function exitCallback() {
        console.log("This is the callback function. It is executed when the Immersive Reader closes.");
    }
```


Обратите внимание, что весь текст имеет атрибут **lang**, который описывает языки текста. Этот атрибут помогает Иммерсивному средству чтения предоставить соответствующие функции языка и грамматики.

## <a name="build-and-run-the-app"></a>Создание и запуск приложения

Теперь наше веб-приложение готово. Запустите приложение, выполнив следующую команду:

```bash
npm start
```

Откройте веб-браузер и перейдите по адресу _http://localhost:3000_ . Вы увидите следующее:

![Пример приложения (Node.js)](../../media/quickstart-nodejs/1-buildapp.png)

## <a name="launch-the-immersive-reader"></a>Запуск Иммерсивного средства чтения

При нажатии кнопки "иммерсивное средство чтения" появится запущенное иммерсивное средство чтения с содержимым на странице.

![Иммерсивное средство чтения (Node.js)](../../media/quickstart-nodejs/2-viewimmersivereader.png)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Создание ресурса и настройка AAD](../../how-to-create-immersive-reader.md)