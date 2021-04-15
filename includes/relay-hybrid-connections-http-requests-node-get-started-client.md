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
ms.openlocfilehash: 3c18efa7eb520b765c9bb3c2aff00104f971f5a8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "67185525"
---
### <a name="create-a-nodejs-application"></a>Создание приложения Node.js

Если при создании ретранслятора вы отключили параметр"Требует авторизации клиента", запросы на URL-адрес гибридных подключений можно отправлять с помощью любого браузера. Для доступа к защищенным конечным точкам необходимо создать и передать маркер в заголовок `ServiceBusAuthorization`, который приведен ниже.

Для начала создайте новый файл JavaScript с именем `sender.js`.

### <a name="add-the-relay-npm-package"></a>Добавление пакета ретранслятора NPM

Запустите `npm install hyco-https` из командной строки NPM в папке проекта. Этот пакет также импортирует обычный пакет `https`. На клиентской стороне ключевое различие заключается в том, что пакет предоставляет функции для создания URI и маркеров ретранслятора.

### <a name="write-some-code-to-send-messages"></a>Написание кода для отправки сообщений

1. Добавьте `constants` в начало файла `sender.js`.
   
    ```js
    const https = require('hyco-https');
    ```

2. Добавьте следующие константы в файл `sender.js` для сведений о гибридном подключении. Замените заполнители в скобках значениями, которые получены при создании гибридного подключения.
   
   1. `const ns` — пространство имен ретранслятора. Необходимо использовать полное имя пространства имен, например `{namespace}.servicebus.windows.net`.
   2. `const path` — имя гибридного подключения.
   3. `const keyrule` — имя ключа SAS.
   4. `const key` — значение ключа SAS.

3. Добавьте в файл `sender.js` указанный ниже код. Обратите внимание, что этот код существенно не отличается от обычного использования клиента HTTPS Node.js. В нем просто добавлен заголовок авторизации.
   
    ```js
   https.get({
        hostname : ns,
        path : (!path || path.length == 0 || path[0] !== '/'?'/':'') + path,
        port : 443,
        headers : {
            'ServiceBusAuthorization' : 
                https.createRelayToken(https.createRelayHttpsUri(ns, path), keyrule, key)
        }
    }, (res) => {
        let error;
        if (res.statusCode !== 200) {
            console.error('Request Failed.\n Status Code: ${statusCode}');
            res.resume();
        } 
        else {
            res.setEncoding('utf8');
            res.on('data', (chunk) => {
                console.log(`BODY: ${chunk}`);
            });
            res.on('end', () => {
                console.log('No more data in response.');
            });
        };
    }).on('error', (e) => {
        console.error(`Got error: ${e.message}`);
    });
    ```
    Файл sender.js должен выглядеть следующим образом:
   
    ```js
    const https = require('hyco-https');
       
    const ns = "{RelayNamespace}";
    const path = "{HybridConnectionName}";
    const keyrule = "{SASKeyName}";
    const key = "{SASKeyValue}";
   
    https.get({
        hostname : ns,
        path : (!path || path.length == 0 || path[0] !== '/'?'/':'') + path,
        port : 443,
        headers : {
            'ServiceBusAuthorization' : 
                https.createRelayToken(https.createRelayHttpsUri(ns, path), keyrule, key)
        }
    }, (res) => {
        let error;
        if (res.statusCode !== 200) {
            console.error('Request Failed.\n Status Code: ${statusCode}');
            res.resume();
        } 
        else {
            res.setEncoding('utf8');
            res.on('data', (chunk) => {
                console.log(`BODY: ${chunk}`);
            });
            res.on('end', () => {
                console.log('No more data in response.');
            });
        };
    }).on('error', (e) => {
        console.error(`Got error: ${e.message}`);
    });
    ```

