---
title: Краткое руководство. REST API Службы Azure SignalR
description: Узнайте, как использовать REST API со Службой Azure SignalR в следующих примерах. Дополнительные сведения о спецификации REST API.
author: sffamily
ms.service: signalr
ms.topic: quickstart
ms.date: 11/13/2019
ms.author: zhshang
ms.openlocfilehash: 2613f91e8c7f1ad3a05792a9a165f4560c09a637
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94874531"
---
# <a name="quickstart-broadcast-real-time-messages-from-console-app"></a>Краткое руководство. Широковещательные сообщения в режиме реального времени из консольного приложения

Служба Azure SignalR предоставляет [REST API](https://github.com/Azure/azure-signalr/blob/dev/docs/rest-api.md) для поддержки сценариев связи сервера с клиентом, таких как широковещательная передача. Вы можете выбрать любой язык программирования, который может сделать вызов REST API. Вы можете отправлять сообщения всем подключенным клиентам, конкретному клиенту по имени или группе клиентов.

В этом кратком руководстве вы узнаете, как отправлять сообщения из приложения командной строки в подключенные клиентские приложения на языке C#.

## <a name="prerequisites"></a>Предварительные требования

Это краткое руководство предназначено для macOS, Windows или Linux.

* [Базовый пакет SDK для .NET](https://www.microsoft.com/net/download/core)
* Текстовый редактор или редактор кода по вашему выбору.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsapi).

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите на портал Azure по адресу <https://portal.azure.com/> с помощью своей учетной записи Azure.

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsapi).

[!INCLUDE [Create instance](includes/signalr-quickstart-create-instance.md)]

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsapi).

## <a name="clone-the-sample-application"></a>Клонирование примера приложения

Пока служба развертывается, давайте перейдем к подготовке кода. Клонируйте [пример приложения из GitHub](https://github.com/aspnet/AzureSignalR-samples.git), задайте строку подключения службы SignalR и запустите приложение в локальной среде.

1. Откройте окно терминала Git. Перейдите в папку, в которую вы хотите клонировать пример проекта.

1. Выполните команду ниже, чтобы клонировать репозиторий с примером. Эта команда создает копию примера приложения на локальном компьютере.

    ```bash
    git clone https://github.com/aspnet/AzureSignalR-samples.git
    ```
Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsapi).

## <a name="build-and-run-the-sample"></a>Сборка и запуск примера

Этот пример представляет собой консольное приложение, показывающее использование службы Azure SignalR. Он предоставляет два режима.

- Режим сервера — используйте простые команды для вызова REST API службы Azure SignalR.
- Режим клиента — подключитесь к службе Azure SignalR и получайте сообщения с сервера.

Также вы можете узнать, как создать маркер доступа для проверки подлинности с помощью службы Azure SignalR.

### <a name="build-the-executable-file"></a>Сборка исполняемого файла

В качестве примера мы используем macOS osx.10.13-x64. Можно найти [рекомендации](/dotnet/core/rid-catalog) по сборке на других платформах.

```bash
cd AzureSignalR-samples/samples/Serverless/

dotnet publish -c Release -r osx.10.13-x64
```

### <a name="start-a-client"></a>Запуск клиента

```bash
cd bin/Release/netcoreapp2.1/osx.10.13-x64/

Serverless client <ClientName> -c "<ConnectionString>" -h <HubName>
```

### <a name="start-a-server"></a>Запуск сервера

```bash
cd bin/Release/netcoreapp2.1/osx.10.13-x64/

Serverless server -c "<ConnectionString>" -h <HubName>
```

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsapi).

## <a name="run-the-sample-without-publishing"></a>Запуск примера без публикации

Можно также выполнить следующую команду, чтобы запустить сервер или клиент

```bash
# Start a server
dotnet run -- server -c "<ConnectionString>" -h <HubName>

# Start a client
dotnet run -- client <ClientName> -c "<ConnectionString>" -h <HubName>
```

### <a name="use-user-secrets-to-specify-connection-string"></a>Использование функции user-secrets для указания строки подключения

Вы можете выполнить `dotnet user-secrets set Azure:SignalR:ConnectionString "<ConnectionString>"` в корневом каталоге примера. После этого вам больше не нужен параметр `-c "<ConnectionString>"`.

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsapi).

## <a name="usage"></a>Использование

После запуска сервера используйте такую команду для отправки сообщения.

```
send user <User Id>
send users <User List>
send group <Group Name>
send groups <Group List>
broadcast
```

Можно запустить несколько клиентов с разными именами.

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsapi).

## <a name="integration-with-third-party-services"></a><a name="usage"> </a> Интеграция со сторонними службами

Служба Azure SignalR позволяет интегрировать сторонние службы с системой.

### <a name="definition-of-technical-specifications"></a>Определение технических спецификаций

В следующей таблице показаны все версии поддерживаемых REST API. Также можно найти файл с определением для каждой версии.

Версия | Состояние API | Порт | Specific
--- | --- | --- | ---
`1.0-preview` | Доступно | 5002 | [Swagger](https://github.com/Azure/azure-signalr/tree/dev/docs/swagger/v1-preview.json)
`1.0` | Доступно | Standard | [Swagger](https://github.com/Azure/azure-signalr/tree/dev/docs/swagger/v1.json)

В следующе таблице показан список доступных API для каждой версии.

API | 1.0-preview | 1.0
--- | --- | ---
[Отправка всем](#broadcast) | **&#x2713;** | **&#x2713;**
[Отправка группе](#broadcast-group) | **&#x2713;** | **&#x2713;**
Широковещательная передача для некоторых групп | **&#x2713;** (Устарел) | `N / A`
[Отправка пользователю](#send-user) | **&#x2713;** | **&#x2713;**
Отправка некоторым пользователям | **&#x2713;** (Устарел) | `N / A`
[Добавление пользователя в группу](#add-user-to-group) | `N / A` | **&#x2713;**
[Удаление пользователя из группы](#remove-user-from-group) | `N / A` | **&#x2713;**
[Проверка существования пользователя](#check-user-existence) | `N / A` | **&#x2713;**
[Удаление пользователя из всех групп](#remove-user-from-all-groups) | `N / A` | **&#x2713;**
[Отправка сообщения в ответ на подключение](#send-connection) | `N / A` | **&#x2713;**
[Добавление подключения в группу](#add-connection-to-group) | `N / A` | **&#x2713;**
[Удаление подключения из группы](#remove-connection-from-group) | `N / A` | **&#x2713;**
[Закрытие подключения клиента](#close-connection) | `N / A` | **&#x2713;**
[Работоспособность служб](#service-health) | `N / A` | **&#x2713;**

<a name="broadcast"> </a>
### <a name="broadcast-to-everyone"></a>Широковещательная передача для всех

Версия | Метод HTTP для API | Request URL (URL-адрес запроса) | Тело запроса
--- | --- | --- | ---
`1.0-preview` | `POST` | `https://<instance-name>.service.signalr.net:5002/api/v1-preview/hub/<hub-name>` | `{"target": "<method-name>", "arguments": [...]}`
`1.0` | `POST` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>` | То же, что и выше

<a name="broadcast-group"> </a>
### <a name="broadcast-to-a-group"></a>Широковещательная передача для группы

Версия | Метод HTTP для API | Request URL (URL-адрес запроса) | Тело запроса
--- | --- | --- | ---
`1.0-preview` | `POST` | `https://<instance-name>.service.signalr.net:5002/api/v1-preview/hub/<hub-name>/group/<group-name>` | `{"target": "<method-name>", "arguments": [...]}`
`1.0` | `POST` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/groups/<group-name>` | То же, что и выше

<a name="send-user"> </a>
### <a name="sending-to-a-user"></a>Отправка пользователю

Версия | Метод HTTP для API | Request URL (URL-адрес запроса) | Тело запроса
--- | --- | --- | ---
`1.0-preview` | `POST` | `https://<instance-name>.service.signalr.net:5002/api/v1-preview/hub/<hub-name>/user/<user-id>` | `{"target": "<method-name>", "arguments": [...]}`
`1.0` | `POST` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/users/<user-id>` | То же, что и выше

<a name="add-user-to-group"> </a>
### <a name="adding-a-user-to-a-group"></a>Добавление пользователя в группу

Версия | Метод HTTP для API | Request URL (URL-адрес запроса)
--- | --- | ---
`1.0` | `PUT` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/groups/<group-name>/users/<user-id>`

<a name="remove-user-from-group"> </a>
### <a name="removing-a-user-from-a-group"></a>Удаление пользователя из группы

Версия | Метод HTTP для API | Request URL (URL-адрес запроса)
--- | --- | ---
`1.0` | `DELETE` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/groups/<group-name>/users/<user-id>`

<a name="check-user-existence"> </a>
### <a name="check-user-existence-in-a-group"></a>Проверка существования пользователя в группе

Версия API | Метод HTTP для API | Request URL (URL-адрес запроса)
---|---|---
`1.0` | `GET` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/users/<user-id>/groups/<group-name>`
`1.0` | `GET` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/groups/<group-name>/users/<user-id>` 

Код состояния отклика | Описание
---|---
`200` | Пользователь существует
`404` | Пользователь не существует

<a name="remove-user-from-all-groups"> </a>
### <a name="remove-a-user-from-all-groups"></a>Удаление пользователя из всех групп

Версия API | Метод HTTP для API | Request URL (URL-адрес запроса)
---|---|---
`1.0` | `DELETE` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/users/<user-id>/groups`

<a name="send-connection"> </a>
### <a name="send-message-to-a-connection"></a>Отправка сообщения в ответ на подключение

Версия API | Метод HTTP для API | Request URL (URL-адрес запроса) | Текст запроса
---|---|---|---
`1.0` | `POST` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/connections/<connection-id>` | `{ "target":"<method-name>", "arguments":[ ... ] }`

<a name="add-connection-to-group"> </a>
### <a name="add-a-connection-to-a-group"></a>Добавление подключения в группу

Версия API | Метод HTTP для API | Request URL (URL-адрес запроса)
---|---|---
`1.0` | `PUT` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/groups/<group-name>/connections/<connection-id>`
`1.0` | `PUT` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/connections/<connection-id>/groups/<group-name>`

<a name="remove-connection-from-group"> </a>
### <a name="remove-a-connection-from-a-group"></a>Удаление подключения из группы

Версия API | Метод HTTP для API | Request URL (URL-адрес запроса)
---|---|---
`1.0` | `DELETE` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/groups/<group-name>/connections/<connection-id>`
`1.0` | `DELETE` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/connections/<connection-id>/groups/<group-name>`

<a name="close-connection"> </a>
### <a name="close-a-client-connection"></a>Закрытие подключения клиента

Версия API | Метод HTTP для API | Request URL (URL-адрес запроса)
---|---|---
`1.0` | `DELETE` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/connections/<connection-id>`
`1.0` | `DELETE` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/connections/<connection-id>?reason=<close-reason>`

<a name="service-health"> </a>
### <a name="service-health"></a>Работоспособность служб

Версия API | Метод HTTP для API | Request URL (URL-адрес запроса)
---|---|---                             
`1.0` | `GET` | `https://<instance-name>.service.signalr.net/api/v1/health`

Код состояния отклика | Описание
---|---
`200` | Служба работает
`5xx` | Ошибка службы

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsapi).

[!INCLUDE [Cleanup](includes/signalr-quickstart-cleanup.md)]

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsapi).

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы узнали, как использовать REST API для трансляции сообщений от Службы SignalR к клиентам в режиме реального времени. Дополнительные сведения о том, как разработать и развернуть Функции Azure с помощью привязки Службы SignalR, которая построена на основе REST API.

> [!div class="nextstepaction"]
> [Краткое руководство. Создание комнаты чата с помощью служб "Функции Azure" и SignalR с помощью C#](signalr-quickstart-azure-functions-csharp.md)
