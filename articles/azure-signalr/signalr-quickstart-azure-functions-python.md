---
title: Краткое руководство. Использование бессерверной службы Azure SignalR с Python
description: Краткое руководство по использованию служб "Функции Azure" и SignalR для создания чат-комнаты с помощью Python.
author: anthonychu
ms.service: signalr
ms.devlang: python
ms.topic: quickstart
ms.date: 12/14/2019
ms.author: antchu
ms.custom: devx-track-python
ms.openlocfilehash: 82a2e64d27fcbbb3801e6dab347343e086a0b70f
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105048227"
---
# <a name="quickstart-create-a-chat-room-with-azure-functions-and-signalr-service-using-python"></a>Краткое руководство. Создание комнаты чата с помощью Функций Azure и SignalR с помощью Python

Служба Azure SignalR позволяет легко добавлять в приложение функции реального времени. Функции Azure — бессерверная платформа, которая позволяет выполнять код без необходимости управлять какой-либо инфраструктурой. В этом кратком руководстве вы научитесь использовать службы SignalR и "Функции Azure" для построения бессерверного чат-приложения, работающего в режиме реального времени.

## <a name="prerequisites"></a>Предварительные требования

Это краткое руководство предназначено для macOS, Windows или Linux.

Скачайте и установите редактор кодов [Visual Studio Code](https://code.visualstudio.com/).

Установите [Azure Functions Core Tools](https://github.com/Azure/azure-functions-core-tools#installing) (версии 2.7.1505 и выше) для локального запуска приложений Функций Azure на Python.

Для работы с Функциями Azure требуется [Python 3.6 или более поздних версий](https://www.python.org/downloads/). Поддерживаемые версии Python приведены в [этой статье](/azure/azure-functions/functions-reference-python#python-version).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qspython).

## <a name="log-in-to-azure"></a>Вход в Azure

Войдите на портал Azure по адресу <https://portal.azure.com/> с помощью своей учетной записи Azure.

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qspython).

[!INCLUDE [Create instance](includes/signalr-quickstart-create-instance.md)]

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qspython).

[!INCLUDE [Clone application](includes/signalr-quickstart-clone-application.md)]

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qspython).

## <a name="configure-and-run-the-azure-function-app"></a>Настройка и запуск приложения-функции Azure

1. В браузере, где открыт портал Azure, выполнив поиск имени в верхней части портала, убедитесь, что ранее развернутый экземпляр службы SignalR успешно создан. Выберите экземпляр, чтобы открыть его.

    ![Поиск экземпляра службы SignalR](media/signalr-quickstart-azure-functions-csharp/signalr-quickstart-search-instance.png)

1. Выберите **ключи** для просмотра строк подключения экземпляра службы SignalR.

1. Выберите и скопируйте основную строку подключения.

    ![Выберите и скопируйте основную строку подключения.](media/signalr-quickstart-azure-functions-javascript/signalr-quickstart-keys.png)

1. В редакторе кода откройте в клонированном репозитории папку *src/chat/python*.

1. Чтобы локально разрабатывать и тестировать функции Python, необходимо работать в среде Python 3.6 или 3.7. Для создания и активации виртуальной среды с именем `.venv` выполните следующие команды.

    **Linux и macOS**:

    ```bash
    python3.7 -m venv .venv
    source .venv/bin/activate
    ```

    **Windows:**

    ```powershell
    py -3.7 -m venv .venv
    .venv\scripts\activate
    ```

1. Переименуйте *local.settings.sample.json* на *local.settings.json*.

1. В файл **local.settings.json** вставьте строку подключения как значение параметра **AzureSignalRConnectionString**. Сохраните файл.

1. Функции Python упорядочиваются в папки. В каждой папке содержатся два файла: *function.json*, который определяет используемые в функции привязки, и *\_\_init\_\_.py*, который представляет собой основную часть функции. В этом приложении-функции есть две функции триггеров HTTP.

    - **Согласование** использует входную привязку *SignalRConnectionInfo* для генерации и возврата сведений о допустимом соединении.
    - **Сообщения**. Эта функция получает мгновенное сообщение в тексте запроса и использует выходную привязку *SignalR* для рассылки сообщений ко всем подключенным клиентским приложениям.

1. В терминале с активированной виртуальной средой перейдите в папку *src/chat/python*. Установите необходимые пакеты Python с помощью PIP.

    ```bash
    python -m pip install -r requirements.txt
    ```

1. Запустите приложение-функцию.

    ```bash
    func start
    ```

    ![Запуск приложения-функции](media/signalr-quickstart-azure-functions-python/signalr-quickstart-run-application.png)
    
Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qspython).

[!INCLUDE [Run web application](includes/signalr-quickstart-run-web-application.md)]

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qspython).

[!INCLUDE [Cleanup](includes/signalr-quickstart-cleanup.md)]

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qspython).

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы создали и запустили бессерверное приложение в режиме реального времени в VS Code. Далее узнайте, как развертывать Функции Azure с помощью VS Code.

> [!div class="nextstepaction"]
> [Развертывание Функций Azure с помощью VS Code](/azure/developer/javascript/tutorial-vscode-serverless-node-01)

