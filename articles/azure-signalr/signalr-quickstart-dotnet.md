---
title: Разработка с ASP.NET в Службе Azure SignalR
description: Краткое руководство по использованию Службы Azure SignalR для создания комнаты чата с помощью платформы ASP.NET.
author: sffamily
ms.service: signalr
ms.devlang: dotnet
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.date: 09/28/2020
ms.author: zhshang
ms.openlocfilehash: c39ef505b0cea0ad0c03b81683db8441077cd0d2
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94874548"
---
# <a name="quickstart-create-a-chat-room-with-aspnet-and-signalr-service"></a>Краткое руководство. Создание комнаты чата с помощью ASP.NET и Службы SignalR

Служба Azure SignalR основана на экземпляре [SignalR для ASP.NET Core 2.1](/aspnet/core/signalr/introduction?preserve-view=true&view=aspnetcore-2.1), который совместим с ASP.NET SignalR **не** на 100 %. В Службе Azure SignalR повторно реализован протокол данных ASP.NET SignalR, основанный на последних технологиях ASP.NET Core. При использовании Службы Azure SignalR для ASP.NET SignalR некоторые функции ASP.NET SignalR больше не поддерживаются, например, Azure SignalR не отвечает на сообщения при повторном подключении клиента. Кроме того, транспортировка Forever Frame и JSONP не поддерживается. Чтобы приложение ASP.NET SignalR работало со Службой SignalR, нужно внести некоторые изменения в код и использовать подходящую версию зависимых библиотек.

Полный список сравнения функций ASP.NET SignalR и Службы ASP.NET Core см. в [документе об отличиях версий](/aspnet/core/signalr/version-differences?preserve-view=true&view=aspnetcore-3.1).

В этом кратком руководстве вы узнаете, как приступить к работе с ASP.NET и Службой Azure SignalR для аналогичного [приложения комнаты чата](./signalr-quickstart-dotnet-core.md).


[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note-dotnet.md)]

## <a name="prerequisites"></a>Предварительные требования

* [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/)
* [.NET 4.6.1](https://www.microsoft.com/net/download/windows)
* [ASP.NET SignalR 2.4.1](https://www.nuget.org/packages/Microsoft.AspNet.SignalR/)

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsnet).

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите на [портал Azure](https://portal.azure.com/) с помощью своей учетной записи Azure.

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsnet).

[!INCLUDE [Create instance](includes/signalr-quickstart-create-instance.md)]

Для приложений ASP.NET SignalR *бессерверный* режим не поддерживается. Всегда используйте режим *По умолчанию* или *Классический* для экземпляра Службы Azure SignalR.

Вы также можете создать ресурсы Azure, используемые в этом кратком руководстве, с помощью [сценария создания Службы SignalR](scripts/signalr-cli-create-service.md).

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsnet).

## <a name="clone-the-sample-application"></a>Клонирование примера приложения

Пока служба развертывается, давайте перейдем к коду. Клонируйте [пример приложения из GitHub](https://github.com/aspnet/AzureSignalR-samples/tree/master/aspnet-samples/ChatRoom), задайте строку подключения службы SignalR и запустите приложение в локальной среде.

1. Откройте окно терминала Git. Перейдите в папку, в которую вы хотите клонировать пример проекта.

1. Выполните команду ниже, чтобы клонировать репозиторий с примером. Эта команда создает копию примера приложения на локальном компьютере.

    ```bash
    git clone https://github.com/aspnet/AzureSignalR-samples.git
    ```

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsnet).

## <a name="configure-and-run-chat-room-web-app"></a>Настройка и запуск веб-приложения комнаты чата

1. Запустите Visual Studio и откройте решение в папке *aspnet-samples/ChatRoom/* клонированного репозитория.

1. В браузере, где открыт портал Azure, найдите и выберите созданный экземпляр.

1. Выберите **ключи** для просмотра строк подключения экземпляра службы SignalR.

1. Выберите и скопируйте основную строку подключения.

1. Теперь задайте строку подключения в файле *web.config*.

    ```xml
    <configuration>
    <connectionStrings>
        <add name="Azure:SignalR:ConnectionString" connectionString="<Replace By Your Connection String>"/>
    </connectionStrings>
    ...
    </configuration>
    ```

1. В файле *Startup.cs* вместо вызова метода `MapSignalR()` необходимо вызвать `MapAzureSignalR({YourApplicationName})` и передать строку подключения, чтобы приложение подключилось к службе, а не самостоятельно размещало SignalR. Замените `{YourApplicationName}` на имя приложения. Это уникальное имя. Оно нужно, чтобы отличать это приложение от других приложений. Вы можете использовать `this.GetType().FullName` как значение.

    ```cs
    public void Configuration(IAppBuilder app)
    {
        // Any connection or hub wire up and configuration should go here
        app.MapAzureSignalR(this.GetType().FullName);
    }
    ```

    Прежде чем использовать эти API, вам необходимо сослаться на пакет SDK для служб. Откройте **Инструменты | Диспетчер пакетов NuGet | Консоль диспетчера пакетов** и выполните команду:

    ```powershell
    Install-Package Microsoft.Azure.SignalR.AspNet
    ```

    Кроме этих изменений, все остальное остается неизменным. Вы по-прежнему можете использовать интерфейс центра, с которым вы уже знакомы, чтобы записывать бизнес-логику.

    > [!NOTE]
    > В реализации конечная точка `/signalr/negotiate` предоставляется для согласования с помощью пакета SDK для Службы Azure SignalR. Когда клиенты попытаются подключиться, будет возвращен специальный ответ о согласовании и клиенты будут перенаправлены в конечную точку службы, определенную в строке подключения.

1. Нажмите клавишу <kbd>F5</kbd>, чтобы запустить проект в режиме отладки. Вы увидите, что приложение запущено локально. Вместо размещения среды выполнения SignalR самим приложением, оно теперь подключается к Службе Azure SignalR.

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsnet).

[!INCLUDE [Cleanup](includes/signalr-quickstart-cleanup.md)]

> [!IMPORTANT]
> Удаление группы ресурсов — необратимая операция, и все соответствующие ресурсы удаляются окончательно. Будьте внимательны, чтобы случайно не удалить не ту группу ресурсов или не те ресурсы. Если ресурсы для размещения этого примера созданы в имеющейся группе ресурсов, содержащей ресурсы, которые следует сохранить, можно удалить каждый ресурс отдельно в соответствующих колонках вместо удаления группы ресурсов.

Войдите на [портал Azure](https://portal.azure.com) и щелкните **Группы ресурсов**.

Введите имя группы ресурсов в текстовое поле **Фильтровать по имени...** . В инструкциях к этому краткому руководству была использована группа ресурсов с именем *SignalRTestResources*. В своей группе ресурсов в списке результатов щелкните **...** , а затем **Удалить группу ресурсов**.

![DELETE](./media/signalr-quickstart-dotnet-core/signalr-delete-resource-group.png)

Через некоторое время группа ресурсов и все ее ресурсы будут удалены.

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsnet).

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы создали ресурс Службы Azure SignalR и использовали его с веб-приложением ASP.NET. Далее вы узнаете, как разработать приложения в режиме реального времени с помощью Службы Azure SignalR с использованием ASP.NET Core.

> [!div class="nextstepaction"]
> [Краткое руководство. Создание чата с помощью Службы SignalR](./signalr-quickstart-dotnet-core.md)
