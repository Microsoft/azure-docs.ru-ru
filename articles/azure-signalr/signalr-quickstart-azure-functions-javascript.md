---
title: Использование JavaScript для создания чат-комнаты с помощью служб "Функции Azure" и SignalR
description: Краткое руководство по использованию служб "Функции Azure" и SignalR для создания чат-комнаты с помощью JavaScript.
author: sffamily
ms.author: zhshang
ms.date: 12/14/2019
ms.topic: quickstart
ms.service: signalr
ms.devlang: javascript
ms.custom:
- devx-track-js
- mode-api
ms.openlocfilehash: 5e2c9dcd1efc85e1ea3fe6381cbfbff0c5e62fc5
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107533228"
---
# <a name="quickstart-use-javascript-to-create-a-chat-room-with-azure-functions-and-signalr-service"></a>Краткое руководство. Использование JavaScript для создания чат-комнаты с помощью служб "Функции Azure" и SignalR

Служба Azure SignalR позволяет без труда добавлять в приложение функции для работы в режиме реального времени. Функции Azure — это бессерверная платформа, которая позволяет выполнять код без управления инфраструктурой. В рамках этого краткого руководства вы создадите бессерверное чат-приложение на JavaScript, работающее в режиме реального времени, с помощью Службы SignalR и Функций.

## <a name="prerequisites"></a>Предварительные требования

- Редактор кода, например [Visual Studio Code](https://code.visualstudio.com/).
- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.
- [Azure Functions Core Tools](https://github.com/Azure/azure-functions-core-tools#installing) 2 или более поздней версии. Используется для локального запуска приложений-функций Azure.
- [Node.js](https://nodejs.org/en/download/) (версия 10.x).

   > [!NOTE]
   > Примеры должны работать и с другими версиями Node.js. Дополнительные сведения см. в [документации по версиям среды выполнения Функций Azure](../azure-functions/functions-versions.md#languages).

> [!NOTE]
> Это краткое руководство предназначено для macOS, Windows или Linux.

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsjs).

## <a name="log-in-to-azure"></a>Вход в Azure

Войдите на портал Azure по адресу <https://portal.azure.com/> с помощью своей учетной записи Azure.

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsjs).

[!INCLUDE [Create instance](includes/signalr-quickstart-create-instance.md)]

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsjs).

[!INCLUDE [Clone application](includes/signalr-quickstart-clone-application.md)]

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsjs).

## <a name="configure-and-run-the-azure-function-app"></a>Настройка и запуск приложения-функции Azure

1. В браузере, где открыт портал Azure, выполнив поиск имени в верхней части портала, убедитесь, что ранее развернутый экземпляр службы SignalR успешно создан. Выберите экземпляр, чтобы открыть его.

    ![Поиск экземпляра службы SignalR](media/signalr-quickstart-azure-functions-csharp/signalr-quickstart-search-instance.png)

1. Выберите **ключи** для просмотра строк подключения экземпляра службы SignalR.

1. Выберите и скопируйте основную строку подключения.

    ![Снимок экрана, на котором выделена основная строка подключения.](media/signalr-quickstart-azure-functions-javascript/signalr-quickstart-keys.png)

1. В редакторе кода откройте папку *src/chat/javascript* из клонированного репозитория.

1. Переименуйте *local.settings.sample.json* на *local.settings.json*.

1. В файл **local.settings.json** вставьте строку подключения как значение параметра **AzureSignalRConnectionString**. Сохраните файл.

1. Функции JavaScript упорядочиваются в папки. В каждой папке содержатся два файла: *function.json* определяет привязки, используемые в функции, а *index.js* представляет собой основную часть функции. В этом приложении-функции есть две функции триггеров HTTP.

    - **Согласование** использует входную привязку *SignalRConnectionInfo* для генерации и возврата сведений о допустимом соединении.
    - **Сообщения**. Эта функция получает мгновенное сообщение в тексте запроса и использует выходную привязку *SignalR* для рассылки сообщений ко всем подключенным клиентским приложениям.

1. В терминале убедитесь, что вы находитесь в папке *src/chat/javascript*. Запустите приложение-функцию.

    ```bash
    func start
    ```

    ![Создание службы SignalR](media/signalr-quickstart-azure-functions-javascript/signalr-quickstart-run-application.png)
    
Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsjs).

[!INCLUDE [Run web application](includes/signalr-quickstart-run-web-application.md)]

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsjs).

[!INCLUDE [Cleanup](includes/signalr-quickstart-cleanup.md)]

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsjs).

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы создали и запустили бессерверное приложение в режиме реального времени в VS Code. Далее узнайте, как развертывать Функции Azure с помощью VS Code.

> [!div class="nextstepaction"]
> [Развертывание Функций Azure с помощью VS Code](/azure/developer/javascript/tutorial-vscode-serverless-node-01)
