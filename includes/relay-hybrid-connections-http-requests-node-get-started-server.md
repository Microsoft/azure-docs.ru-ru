---
title: Включить файл
description: Включить файл
services: service-bus-relay
author: clemensv
ms.service: service-bus-relay
ms.topic: include
ms.date: 05/02/2018
ms.author: clemensv
ms.custom: include file
ms.openlocfilehash: fdd4e26839661ab9765b7d496e7f60c3686ba637
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "67185524"
---
### <a name="create-a-nodejs-application"></a>Создание приложения Node.js

Создайте новый файл JavaScript с именем `listener.js`.

### <a name="add-the-relay-npm-package"></a>Добавление пакета ретранслятора NPM

Запустите `npm install hyco-https` из командной строки NPM в папке проекта.

### <a name="write-some-code-to-handle-requests"></a>Написание кода для обработки запросов

1. Добавьте следующую константу в верхнюю часть файла `listener.js`.

    ```js
    const https = require('hyco-https');
    ```
2. Добавьте следующие константы в файл `listener.js` для сведений о гибридном подключении. Замените заполнители в скобках значениями, которые получены при создании гибридного подключения.

   1. `const ns` — пространство имен ретранслятора. Необходимо использовать полное имя пространства имен, например `{namespace}.servicebus.windows.net`.
   2. `const path` — имя гибридного подключения.
   3. `const keyrule` — имя ключа SAS.
   4. `const key` — значение ключа SAS.

3. Добавьте в файл `listener.js` указанный ниже код. :

    Как видите, код не слишком отличается от любого простого примера для сервера HTTP, который можно найти в руководствах для начинающих разработчиков Node.js. Единственное отличие в том, что вместо стандартной функции `createServer` используется `createRelayedServer`.

    ```js
    var uri = https.createRelayListenUri(ns, path);
    var server = https.createRelayedServer(
        {
            server : uri,
            token : () => https.createRelayToken(uri, keyrule, key)
        },
        (req, res) => {
            console.log('request accepted: ' + req.method + ' on ' + req.url);
            res.setHeader('Content-Type', 'text/html');
            res.end('<html><head><title>Hey!</title></head><body>Relayed Node.js Server!</body></html>');
        });

    server.listen( (err) => {
            if (err) {
              return console.log('something bad happened', err)
            }
            console.log(`server is listening on ${port}`)
          });

    server.on('error', (err) => {
        console.log('error: ' + err);
    });
    ```
    Вот как будет выглядеть файл listener.js:
   
    ```js
    const https = require('hyco-https');
   
    const ns = "{RelayNamespace}";
    const path = "{HybridConnectionName}";
    const keyrule = "{SASKeyName}";
    const key = "{SASKeyValue}";
   
    var uri = https.createRelayListenUri(ns, path);
    var server = https.createRelayedServer(
        {
            server : uri,
            token : () => https.createRelayToken(uri, keyrule, key)
        },
        (req, res) => {
            console.log('request accepted: ' + req.method + ' on ' + req.url);
            res.setHeader('Content-Type', 'text/html');
            res.end('<html><head><title>Hey!</title></head><body>Relayed Node.js Server!</body></html>');
        });

    server.listen( (err) => {
            if (err) {
              return console.log('something bad happened', err)
            }
            console.log(`server is listening on ${port}`)
          });

    server.on('error', (err) => {
        console.log('error: ' + err);
    });
    ```

