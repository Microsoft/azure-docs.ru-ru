---
title: Интеграция нескольких ресурсов иммерсивного средства чтения
titleSuffix: Azure Cognitive Services
description: В этом учебнике описано, как создать приложение Node.js, которое запускает иммерсивное средство чтения, используя несколько ресурсов иммерсивного средства чтения.
author: skamal
manager: nitinme
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: how-to
ms.date: 01/14/2020
ms.author: skamal
ms.custom: devx-track-js
ms.openlocfilehash: dc6c7bd9e38aa21ab10f55fca2500b0c12194d43
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102620172"
---
# <a name="integrate-multiple-immersive-reader-resources"></a>Интеграция нескольких ресурсов иммерсивного средства чтения

В [обзорной статье](./overview.md) вы узнали о том, что представляет собой иммерсивное средство чтения и каким образом в нем реализованы проверенные методы, улучшающие понимание прочитанного для начинающих, тех, кто изучает язык, и учащихся с особыми потребностями. В [кратком руководстве](./quickstarts/client-libraries.md)вы узнали, как использовать иммерсивное средство чтения с одним ресурсом. В этом учебнике описано, как интегрировать несколько ресурсов иммерсивного средства чтения в одном приложении. В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создание нескольких ресурсов иммерсивного средства чтения в существующей группе ресурсов
> * Запуск иммерсивного средства чтения с помощью нескольких ресурсов

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

* Чтобы создать веб-приложение, запускающее иммерсивное средство чтения с помощью NodeJS, изучите [краткое руководство](./quickstarts/client-libraries.md?pivots=programming-language-nodejs). В таком кратком руководстве вы настроите одно иммерсивное средство чтения. В этом руководстве мы будем опираться на предыдущее руководство.

## <a name="create-the-immersive-reader-resources"></a>Создание ресурсов иммерсивного средства чтения

Выполните [эти инструкции](./how-to-create-immersive-reader.md) для создания каждого иммерсивного ресурса чтения. В скрипте **CREATE-иммерсивереадерресаурце** есть `ResourceName` `ResourceSubdomain` Параметры, и `ResourceLocation` . Эти параметры должны быть уникальными для каждого создаваемого ресурса. Остальные параметры должны совпадать с параметрами, которые использовались при настройке первого ресурса иммерсивного средства чтения. Таким образом, каждый ресурс может быть связан с одной и той же группой ресурсов Azure и приложением Azure AD.

В приведенном ниже примере показано, как создать два ресурса: один в WestUS, а другой — в EastUS. Обратите внимание на уникальные значения для `ResourceName`, `ResourceSubdomain`и `ResourceLocation`.

```azurepowershell-interactive
Create-ImmersiveReaderResource
  -SubscriptionName <SUBSCRIPTION_NAME> `
  -ResourceName Resource_name_wus `
  -ResourceSubdomain resource-subdomain-wus `
  -ResourceSKU <RESOURCE_SKU> `
  -ResourceLocation westus `
  -ResourceGroupName <RESOURCE_GROUP_NAME> `
  -ResourceGroupLocation <RESOURCE_GROUP_LOCATION> `
  -AADAppDisplayName <AAD_APP_DISPLAY_NAME> `
  -AADAppIdentifierUri <AAD_APP_IDENTIFIER_URI> `
  -AADAppClientSecret <AAD_APP_CLIENT_SECRET>

Create-ImmersiveReaderResource
  -SubscriptionName <SUBSCRIPTION_NAME> `
  -ResourceName Resource_name_eus `
  -ResourceSubdomain resource-subdomain-eus `
  -ResourceSKU <RESOURCE_SKU> `
  -ResourceLocation eastus `
  -ResourceGroupName <RESOURCE_GROUP_NAME> `
  -ResourceGroupLocation <RESOURCE_GROUP_LOCATION> `
  -AADAppDisplayName <AAD_APP_DISPLAY_NAME> `
  -AADAppIdentifierUri <AAD_APP_IDENTIFIER_URI> `
  -AADAppClientSecret <AAD_APP_CLIENT_SECRET>
```

## <a name="add-resources-to-environment-configuration"></a>Добавление ресурсов в конфигурацию среды

В кратком руководстве вы научились создавать файл конфигурации среды, содержащий параметры `TenantId`, `ClientId`, `ClientSecret` и `Subdomain`. Так как все ресурсы используют одно и то же приложение Azure AD, для `TenantId`, `ClientId`, `ClientSecret`можно использовать те же значения. Единственное изменение, которое необходимо сделать, это указать поддомен для каждого ресурса.

Теперь новый файл __. env__ должен выглядеть следующим образом:

```text
TENANT_ID={YOUR_TENANT_ID}
CLIENT_ID={YOUR_CLIENT_ID}
CLIENT_SECRET={YOUR_CLIENT_SECRET}
SUBDOMAIN_WUS={YOUR_WESTUS_SUBDOMAIN}
SUBDOMAIN_EUS={YOUR_EASTUS_SUBDOMAIN}
```

Не забудьте зафиксировать этот файл в системе управления версиями, так как он содержит секреты, для которых не следует предоставлять открытый доступ.

Далее следует изменить файл _routes\index.js_, созданный для поддержки нескольких ресурсов. Замените содержимое приведенным ниже кодом.

Как и ранее, этот код создает конечную точку API, которая получает токен проверки подлинности Azure AD, используя пароль субъект-службы. На этот раз пользователь может указать расположение ресурса и передать его в качестве параметра запроса. Затем он возвращает объект, содержащий маркер и соответствующий поддомен.

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

            var subdomain = "";
            var region = req.query && req.query.region;
            switch (region) {
                case "eus":
                    subdomain = process.env.SUBDOMAIN_EUS
                    break;
                case "wus":
                default:
                    subdomain = process.env.SUBDOMAIN_WUS
            }

            return res.send({token, subdomain});
        });
    } catch (err) {
        console.log(err);
        return res.status(500).send('CogSvcs IssueToken error');
    }
});

module.exports = router;
```

Конечная точка API **getimmersivereaderlaunchparams** должна быть защищена с помощью формы проверки подлинности (например, [OAuth](https://oauth.net/2/)), чтобы предотвратить получение неавторизованными пользователями токенов для использования относительно службы "Иммерсивное средство чтения" и выставления счетов. Эти сведения выходят за рамки этого руководства.

## <a name="launch-the-immersive-reader-with-sample-content"></a>Запуск иммерсивного средства чтения с примером содержимого

1. Откройте файл _views\index.pug_ и замените его содержимое следующим кодом. Этот код заполнит страницу примером содержимого и добавит две кнопки, которые запустят иммерсивное средство чтения. Один для запуска иммерсивного средства чтения для ресурса EastUS, а другой — для ресурса WestUS.

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
                button(class="immersive-reader-button" data-button-style="icon" data-locale="en" onclick='handleLaunchImmersiveReader("wus")') WestUS Immersive Reader
                button(class="immersive-reader-button" data-button-style="icon" data-locale="en" onclick='handleLaunchImmersiveReader("eus")') EastUS Immersive Reader

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
    function getTokenAndSubdomainAsync(region) {
            return new Promise(function (resolve, reject) {
                $.ajax({
                    url: "/GetTokenAndSubdomain",
                    type: "GET",
                    data: {
                        region: region
                    },
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

        function handleLaunchImmersiveReader(region) {
            getTokenAndSubdomainAsync(region)
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

3. Теперь наше веб-приложение готово. Запустите приложение, выполнив следующую команду:

    ```bash
    npm start
    ```

4. Откройте веб-браузер и перейдите по адресу `http://localhost:3000`. Вы увидите упомянутое выше содержимое на странице. Нажмите кнопку **Иммерсивное средство чтения EastUS** или кнопку **Иммерсивное средство чтения WestUS**, чтобы запустить иммерсивное средство чтения с помощью соответствующих ресурсов.

## <a name="next-steps"></a>Дальнейшие действия

* Ознакомьтесь с разделом о [пакете SDK для иммерсивного средства чтения](https://github.com/microsoft/immersive-reader-sdk) и [справочнике по этому пакету](./reference.md).
* Просмотрите примеры кода на сайте [GitHub](https://github.com/microsoft/immersive-reader-sdk/tree/master/js/samples/advanced-csharp).