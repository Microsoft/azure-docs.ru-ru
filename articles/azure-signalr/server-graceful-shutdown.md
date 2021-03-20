---
title: Аккуратно завершите работу сервера приложений.
description: В этой статье содержатся сведения о корректном завершении работы сервера приложений SignalR.
author: terencefan
ms.author: tefa
ms.date: 11/12/2020
ms.service: signalr
ms.topic: conceptual
ms.openlocfilehash: d9dd7ce9cf321628598a7bb866c5d1b1a6fb0e1e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98201677"
---
# <a name="server-graceful-shutdown"></a>Корректное завершение работы сервера
Служба Microsoft Azure SignalR предоставляет два режима для корректного завершения работы сервера. 

Ключевое преимущество использования этой функции заключается в предотвращении невозможности неожиданного подключения клиента. 

Вместо этого можно либо подождать клиентские подключения, чтобы закрыть себя в соответствии с бизнес-логикой, либо даже перенести клиентское подключение на другой сервер без потери данных. 

## <a name="how-it-works"></a>Принцип работы

Как правило, в процессе корректного завершения работы будет четыре этапа:

1. **Установка сервера в автономном режиме**

    Это означает, что на этот сервер не будут направляться дополнительные клиентские подключения.

2. **`OnShutdown`Обработчики триггеров**

    Вы можете зарегистрировать обработчики завершения работы для каждого концентратора, принадлежащего серверу.
    Они будут вызываться в отношении зарегистрированного заказа прямо после получения ответа **финакк** от службы Azure SignalR. Это означает, что этот сервер был настроен в автономном режиме в службе Azure SignalR.

    Вы можете транслировать сообщения или выполнить некоторые задания очистки на этом этапе, когда все ловушки завершения работы будут выполнены, мы переходим к следующему этапу.

3. **Дождитесь завершения всех клиентских подключений**, в зависимости от выбранного режима может быть:

    **Режим установлен в Ваитфорклиентстоклосе**

    Служба Azure SignalR будет содержать существующие клиенты.

    Может потребоваться разработать такой способ, как широковещательное сообщение для всех клиентов, а затем позволить своим клиентам принять решение о том, когда нужно закрыть или повторно подключиться.

    Прочитайте [чатсампле](https://github.com/Azure/azure-signalr/tree/dev/samples/ChatSample) для примера использования, который пересылает сообщение "Exit", чтобы активировать закрытие клиента в ловушке завершения работы.

    **Режим установлен в Мигратеклиентс**

    Служба Azure SignalR попытается перенаправить клиентское подключение на этом сервере на другой допустимый сервер. 
    
    В этом сценарии `OnConnectedAsync` и `OnDisconnectedAsync` будет запускаться на новом сервере и старом сервере соответственно с `IConnectionMigrationFeature` набором в `HttpContext` , который можно использовать для указания того, было ли переносится клиентское подключение. Это может быть полезно, особенно для сценариев с отслеживанием состояния.

    Соединение с клиентом будет немедленно перенесено после доставки текущего сообщения. Это означает, что следующее сообщение будет направляться на новый сервер.

4. **Прерывать соединения с сервером**

    После того как все клиентские соединения были закрыты или перенесены, или превышено время ожидания (30 по умолчанию),

    Пакет SDK сервера SignalR продолжит процесс завершения работы на этом этапе и завершит все соединения с сервером.

    Клиентские подключения будут по-прежнему отброшены, если его не удалось закрыть или перенести. Например, не завершено ни одного подходящего сообщения целевого сервера или текущего клиента на сервер.

## <a name="sample-codes"></a>Примеры кодов.

Добавьте следующие параметры, когда `AddAzureSignalR` :

```csharp
services.AddSignalR().AddAzureSignalR(option =>
{
    option.GracefulShutdown.Mode = GracefulShutdownMode.WaitForClientsClose;
    // option.GracefulShutdown.Mode = GracefulShutdownMode.MigrateClients;
    option.GracefulShutdown.Timeout = TimeSpan.FromSeconds(30);

    option.GracefulShutdown.Add<Chat>(async (c) =>
    {
        await c.Clients.All.SendAsync("exit");
    });
});
```

### <a name="configure-onconnected-and-ondisconnected-while-setting-graceful-shutdown-mode-to-migrateclients"></a>Настройте `OnConnected` и, `OnDisconnected` установив режим корректного завершения работы, в значение `MigrateClients` .

Мы предоставили "Иконнектионмигратионфеатуре", чтобы указать, было ли соединение перенесено.

```csharp
public class Chat : Hub {

    public override async Task OnConnectedAsync()
    {
        Console.WriteLine($"{Context.ConnectionId} connected.");

        var feature = Context.GetHttpContext().Features.Get<IConnectionMigrationFeature>();
        if (feature != null)
        {
            Console.WriteLine($"[{feature.MigrateTo}] {Context.ConnectionId} is migrated from {feature.MigrateFrom}.");
            // Your business logic.
        }

        await base.OnConnectedAsync();
    }

    public override async Task OnDisconnectedAsync(Exception e)
    {
        Console.WriteLine($"{Context.ConnectionId} disconnected.");

        var feature = Context.GetHttpContext().Features.Get<IConnectionMigrationFeature>();
        if (feature != null)
        {
            Console.WriteLine($"[{feature.MigrateFrom}] {Context.ConnectionId} will be migrated to {feature.MigrateTo}.");
            // Your business logic.
        }

        await base.OnDisconnectedAsync(e);
    }
}
```