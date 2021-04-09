---
title: Создание комнаты чата с помощью служб "Функции Azure" и SignalR с использованием Java
description: Краткое руководство по использованию служб "Функции Azure" и SignalR для создания чат-комнаты с помощью Java.
author: sffamily
ms.service: signalr
ms.devlang: java
ms.topic: quickstart
ms.date: 03/04/2019
ms.author: zhshang
ms.custom: devx-track-java
ms.openlocfilehash: 62f31d8426e86cc20e957f3a47b30109dee0a634
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96185680"
---
# <a name="quickstart-use-java-to-create-a-chat-room-with-azure-functions-and-signalr-service"></a>Краткое руководство. Создание комнаты чата с помощью служб "Функции Azure" и SignalR с использованием Java

Служба Azure SignalR позволяет без труда добавлять в приложение функции для работы в реальном времени. Функции Azure — это бессерверная платформа, которая позволяет выполнять код без управления инфраструктурой. В рамках этого краткого руководства вы создадите бессерверное приложение чата на языке Java, работающее в реальном времени, с помощью службы SignalR и Функций.

## <a name="prerequisites"></a>Предварительные требования

- Редактор кода, например [Visual Studio Code](https://code.visualstudio.com/).
- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.
- [Azure Functions Core Tools](https://github.com/Azure/azure-functions-core-tools#installing). Используется для локального запуска приложений-функций Azure.

   > [!NOTE]
   > Требуемые привязки службы SignalR в Java поддерживаются только в Azure Functions Core Tools версии 2.4.419 (версия хоста 2.0.12332) или выше.

   > [!NOTE]
   > Чтобы установить расширения, для Azure Functions Core Tools необходимо, чтобы был установлен [пакет SDK для .NET Core](https://www.microsoft.com/net/download). Тем не менее для создания приложения "Функции Azure" для JavaScript не требуются знания платформы .NET.

- [Java Developer Kit (JDK)](https://www.azul.com/downloads/zulu/) версии 8.
- [Apache Maven](https://maven.apache.org) 3.0 или более поздней версии.

> [!NOTE]
> Это краткое руководство предназначено для macOS, Windows или Linux.

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsjava).

## <a name="log-in-to-azure"></a>Вход в Azure

Войдите на портал Azure по адресу <https://portal.azure.com/> с помощью своей учетной записи Azure.

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsjava).

[!INCLUDE [Create instance](includes/signalr-quickstart-create-instance.md)]

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsjava).

[!INCLUDE [Clone application](includes/signalr-quickstart-clone-application.md)]

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsjava).

## <a name="configure-and-run-the-azure-function-app"></a>Настройка и запуск приложения-функции Azure

1. В браузере, где открыт портал Azure, выполнив поиск имени в верхней части портала, убедитесь, что ранее развернутый экземпляр службы SignalR успешно создан. Выберите экземпляр, чтобы открыть его.

    ![Поиск экземпляра службы SignalR](media/signalr-quickstart-azure-functions-csharp/signalr-quickstart-search-instance.png)

1. Выберите **ключи** для просмотра строк подключения экземпляра службы SignalR.

1. Выберите и скопируйте основную строку подключения.

    ![Создание службы SignalR](media/signalr-quickstart-azure-functions-javascript/signalr-quickstart-keys.png)

1. В редакторе кода откройте папку *src/chat/java* из клонированного репозитория.

1. Переименуйте *local.settings.sample.json* на *local.settings.json*.

1. В файл **local.settings.json** вставьте строку подключения как значение параметра **AzureSignalRConnectionString**. Сохраните файл.

1. Основной файл, содержащий функции, находится по такому пути: *src/chat/java/src/main/java/com/function/Functions.java*.

    - **Согласование** использует входную привязку *SignalRConnectionInfo* для генерации и возврата сведений о допустимом соединении.
    - **sendMessage** получает мгновенное сообщение в тексте запроса и использует выходную привязку *SignalR*, чтобы разослать сообщение всем подключенным клиентским приложениям.

1. В терминале убедитесь, что вы находитесь в папке *src/chat/java*. Создайте приложение-функцию.

    ```bash
    mvn clean package
    ```

1. Выполните локальное тестирование приложения-функции.

    ```bash
    mvn azure-functions:run
    ```
    
Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsjava).

[!INCLUDE [Run web application](includes/signalr-quickstart-run-web-application.md)]

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsjava).

[!INCLUDE [Cleanup](includes/signalr-quickstart-cleanup.md)]

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsjava).

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы создали и запустили бессерверное приложение в режиме реального времени с помощью Maven. Узнайте, как создавать функции Azure на языке Java с нуля.

> [!div class="nextstepaction"]
> [Создание первой функции с помощью Java и Maven](../azure-functions/create-first-function-cli-csharp.md?pivots=programming-language-java%2cprogramming-language-java)