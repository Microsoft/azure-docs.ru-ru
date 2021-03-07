---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/27/2020
ms.author: trbye
ms.custom: devx-track-js
ms.openlocfilehash: 99be483f67bf5e3f9b27c63a2318df8761c16eff
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102434533"
---
:::row:::
    :::column span="3":::
        Пакет SDK для распознавания речи для JavaScript доступен в виде пакета NPM. см. раздел <a href="https://www.npmjs.com/package/microsoft-cognitiveservices-speech-sdk" target="_blank">Microsoft-cognitiveservices-Speech-SDK </a> и его сопутствующий репозиторий GitHub <a href="https://github.com/Microsoft/cognitive-services-speech-sdk-js" target="_blank">-Services-Speech-SDK-JS </a>.
    :::column-end:::
    :::column:::
        <br>
        <div class="icon is-large">
            <img alt="Node.js" src="https://docs.microsoft.com/media/logos/logo_nodejs.svg" width="60px">
        </div>
    :::column-end:::
:::row-end:::

> [!TIP]
> Несмотря на то, что пакет SDK для распознавания речи для JavaScript доступен в виде пакета NPM, так как Node.js и клиентские веб-браузеры могут использовать его, рассмотрите различные архитектурные аспекты каждой среды. Например, <a href="https://en.wikipedia.org/wiki/Document_Object_Model" target="_blank">объектная модель документов (DOM) </a> недоступна для приложений на стороне сервера, так как <a href="https://nodejs.org/api/fs.html" target="_blank">Файловая система </a> недоступна для клиентских приложений.

### <a name="nodejs-package-manager-npm"></a>Диспетчер пакетов Node.js (NPM)

Чтобы установить пакет SDK для распознавания речи для JavaScript, выполните следующую `npm install` команду ниже.

```nodejs
npm install microsoft-cognitiveservices-speech-sdk
```

Дополнительные сведения см. в <a href="https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/javascript/node" target="_blank"> кратком руководстве поNode.js Speech SDK </a>.
