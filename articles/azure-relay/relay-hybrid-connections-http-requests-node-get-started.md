---
title: Azure Relay гибридные подключения-запросов HTTP в узле
description: Создание консольного приложения Node.js для HTTP-запросов гибридных подключений Azure Relay в Node.
ms.topic: conceptual
ms.date: 06/23/2020
ms.custom: devx-track-js
ms.openlocfilehash: 249b4fa231cd54a1a8054b32985ed0e48fcc16f1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91263765"
---
# <a name="get-started-with-relay-hybrid-connections-http-requests-in-node"></a>Начало работы с HTTP-запросами гибридных подключений Azure Relay в Node

[!INCLUDE [relay-selector-hybrid-connections](../../includes/relay-selector-hybrid-connections.md)]

В этом кратком руководстве описано, как создать приложения Node.js, которые отправляют и получают сообщения по протоколу HTTP. Приложения используют функцию гибридных подключений службы Azure Relay. См. дополнительные сведения о [службе Azure Relay](relay-what-is-it.md). 

При работе с этим кратким руководством вы выполните следующие задачи:

1. Создайте пространство имен ретранслятора с помощью портала Azure.
2. Создайте гибридное подключение в этом пространстве имен с помощью портала Azure.
3. Создайте серверное консольное приложение (прослушиватель) для получения сообщений.
4. Создайте клиентское консольное приложение (отправитель) для отправки сообщений.
5. Запустите приложения.

## <a name="prerequisites"></a>Предварительные требования
- [Node.js](https://nodejs.org/en/).
- Подписка Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начать работу.

## <a name="create-a-namespace-using-the-azure-portal"></a>Создание пространства имен с помощью портала Azure
[!INCLUDE [relay-create-namespace-portal](../../includes/relay-create-namespace-portal.md)]

## <a name="create-a-hybrid-connection-using-the-azure-portal"></a>Создание гибридного подключения с помощью портала Azure
[!INCLUDE [relay-create-hybrid-connection-portal](../../includes/relay-create-hybrid-connection-portal.md)]

## <a name="create-a-server-application-listener"></a>Создание серверного приложения (прослушивателя)
Для ожидания передачи данных и получения сообщений, отправленных ретранслятором, создайте консольное приложение Node.js.

[!INCLUDE [relay-hybrid-connections-node-get-started-server](../../includes/relay-hybrid-connections-http-requests-node-get-started-server.md)]

## <a name="create-a-client-application-sender"></a>Создание клиентского приложения (отправителя)

Для отправки сообщений в ретранслятор используйте любой клиент HTTP или создайте консольное приложение Node.js.

[!INCLUDE [relay-hybrid-connections-node-get-started-client](../../includes/relay-hybrid-connections-http-requests-node-get-started-client.md)]

## <a name="run-the-applications"></a>Запуск приложений

1. Запустите серверное приложение. Для этого в командной строке Node.js введите `node listener.js`.
2. Запустите клиентское приложение. Для этого в командной строке Node.js введите `node sender.js`, а затем любой текст.
3. Убедитесь, что серверное консольное приложение выводит текст, введенный в клиентском приложении.

Поздравляем, вы создали приложение для гибридных подключений с помощью Node.js!

## <a name="next-steps"></a>Дальнейшие действия
В этом кратком руководстве описано, как создать клиентские и серверные приложения Node.js, которые отправляют и получают сообщения по протоколу HTTP. Функция гибридных подключений службы Azure Relay также поддерживает использование протокола WebSocket для отправки и получения сообщений. См. дополнительные сведения об [использовании протокола WebSocket с гибридными подключениями Azure Relay](relay-hybrid-connections-node-get-started.md).

В этом кратком руководстве описано, как использовать Node.js для создания клиентских и серверных приложений. Сведения о том, как создавать клиентские и серверные приложения с помощью .NET Framework, см. в руководствах по использованию [.NET и WebSocket](relay-hybrid-connections-dotnet-get-started.md) или [.NET и HTTP](relay-hybrid-connections-http-requests-dotnet-get-started.md).
