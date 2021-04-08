---
title: включить файл
description: включить файл
author: anthonychu
ms.service: signalr
ms.topic: include
ms.date: 03/04/2019
ms.author: antchu
ms.custom: include file
ms.openlocfilehash: 2af9c1dfd13d3aeafa7cf1ac76537117ecc15aff
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "84317672"
---
## <a name="run-the-web-application"></a>Запуск веб-приложения

1. Чтобы упростить тестирование клиента, откройте в браузере страницу с примером нашего одностраничного веб-приложения: [https://azure-samples.github.io/signalr-service-quickstart-serverless-chat/demo/chat-v2/](https://azure-samples.github.io/signalr-service-quickstart-serverless-chat/demo/chat-v2/). 

    > [!NOTE]
    > Путь к исходному HTML-файлу: [/docs/demo/chat-v2/index.html](https://github.com/Azure-Samples/signalr-service-quickstart-serverless-chat/blob/master/docs/demo/chat-v2/index.html). Если вы хотите разместить HTML-файл у себя, запустите локальный HTTP-сервер (например, [http-server](https://www.npmjs.com/package/http-server)) в каталоге */docs/demo/chat-v2*. Обязательно добавьте источник в параметр `CORS` в файле *local.settings.json*, как в примере.
    > 
    > ```javascript
    > "Host": {
    >  "LocalHttpPort": 7071,
    >  "CORS": "http://localhost:8080,https://azure-samples.github.io",
    >  "CORSCredentials": true
    > }
    >
    > ```

1. Когда появится запрос на ввод базового URL-адреса приложения-функции, введите `http://localhost:7071`.

1. Введите имя пользователя при появлении запроса.

1. Веб-приложение вызывает функцию *GetSignalRInfo* в приложении-функции для получения сведений о подключении, чтобы подключиться к службе Azure SignalR. После установки подключения появится поле ввода сообщений чата.

1. Введите сообщение и нажмите клавишу ВВОД. Приложение отправляет сообщение в функцию *SendMessage* приложения-функции Azure, которое использует привязку для вывода SignalR, чтобы транслировать сообщение всем подключенным клиентам. Если все работает правильно, сообщение должно появиться в приложении.

    ![Выполнение приложения](../media/signalr-quickstart-azure-functions-csharp/signalr-quickstart-run-application.png)

1. Откройте другой экземпляр веб-приложения в новом окне браузера. Вы увидите, что все отправляемые сообщения отображаются во всех экземплярах приложения.
