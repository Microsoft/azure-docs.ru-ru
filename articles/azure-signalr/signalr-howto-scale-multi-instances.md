---
title: Масштабирование с помощью нескольких экземпляров — служба Azure SignalR
description: Во многих сценариях масштабирования клиенту часто требуется подготавливать несколько экземпляров и настроить их совместное использование для создания крупномасштабного развертывания. Например, для сегментирования требуется поддержка нескольких экземпляров.
author: sffamily
ms.service: signalr
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 03/27/2019
ms.author: zhshang
ms.openlocfilehash: fd6ac8c4d4fc4c3fec4f549f8ef4f955e2b1c637
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89439220"
---
# <a name="how-to-scale-signalr-service-with-multiple-instances"></a>Как масштабировать службу SignalR с несколькими экземплярами?
Последний пакет SDK для службы SignalR поддерживает несколько конечных точек для экземпляров службы SignalR. С помощью этой функции можно масштабировать параллельные подключения или использовать их для обмена сообщениями между регионами.

## <a name="for-aspnet-core"></a>Для ASP.NET Core

### <a name="how-to-add-multiple-endpoints-from-config"></a>Как добавить несколько конечных точек из конфигурации?

Config с ключом `Azure:SignalR:ConnectionString` или `Azure:SignalR:ConnectionString:` строкой подключения службы SignalR.

Если ключ начинается с `Azure:SignalR:ConnectionString:` , он должен быть в формате `Azure:SignalR:ConnectionString:{Name}:{EndpointType}` , где `Name` и `EndpointType` являются свойствами `ServiceEndpoint` объекта и доступны из кода.

Можно добавить несколько строк подключения экземпляра с помощью следующих `dotnet` команд:

```cmd
dotnet user-secrets set Azure:SignalR:ConnectionString:east-region-a <ConnectionString1>
dotnet user-secrets set Azure:SignalR:ConnectionString:east-region-b:primary <ConnectionString2>
dotnet user-secrets set Azure:SignalR:ConnectionString:backup:secondary <ConnectionString3>
```

### <a name="how-to-add-multiple-endpoints-from-code"></a>Как добавить несколько конечных точек из кода?

`ServicEndpoint`Класс вводится для описания свойств конечной точки службы SignalR Azure.
При использовании пакета SDK службы SignalR для Azure можно настроить несколько конечных точек экземпляра с помощью:
```cs
services.AddSignalR()
        .AddAzureSignalR(options => 
        {
            options.Endpoints = new ServiceEndpoint[]
            {
                // Note: this is just a demonstration of how to set options.Endpoints
                // Having ConnectionStrings explicitly set inside the code is not encouraged
                // You can fetch it from a safe place such as Azure KeyVault
                new ServiceEndpoint("<ConnectionString0>"),
                new ServiceEndpoint("<ConnectionString1>", type: EndpointType.Primary, name: "east-region-a"),
                new ServiceEndpoint("<ConnectionString2>", type: EndpointType.Primary, name: "east-region-b"),
                new ServiceEndpoint("<ConnectionString3>", type: EndpointType.Secondary, name: "backup"),
            };
        });
```

### <a name="how-to-customize-endpoint-router"></a>Как настроить маршрутизатор конечной точки?

По умолчанию пакет SDK использует [дефаултендпоинтраутер](https://github.com/Azure/azure-signalr/blob/dev/src/Microsoft.Azure.SignalR/EndpointRouters/DefaultEndpointRouter.cs) для выбора конечных точек.

#### <a name="default-behavior"></a>Поведение по умолчанию 
1. Маршрутизация запросов клиентов

    Клиент `/negotiate` с сервером приложений. По умолчанию пакет SDK **случайным образом выбирает** одну конечную точку из набора доступных конечных точек службы.

2. Маршрутизация сообщений сервера

    Если * отправка сообщения определенному подключению * * * и целевое соединение направляется на текущий сервер, сообщение переходит непосредственно к этой подключенной конечной точке. В противном случае сообщения будут пересылаться в каждую конечную точку Azure SignalR.

#### <a name="customize-routing-algorithm"></a>Настройка алгоритма маршрутизации
Вы можете создать собственный маршрутизатор, если у вас есть специальные знания для указания конечных точек, на которые должны отправляться сообщения.

Пользовательский маршрутизатор определяется ниже в качестве примера, когда группы, начинающиеся с, `east-` всегда переходят в конечную точку с именем `east` :

```cs
private class CustomRouter : EndpointRouterDecorator
{
    public override IEnumerable<ServiceEndpoint> GetEndpointsForGroup(string groupName, IEnumerable<ServiceEndpoint> endpoints)
    {
        // Override the group broadcast behavior, if the group name starts with "east-", only send messages to endpoints inside east
        if (groupName.StartsWith("east-"))
        {
            return endpoints.Where(e => e.Name.StartsWith("east-"));
        }

        return base.GetEndpointsForGroup(groupName, endpoints);
    }
}
```

Другой пример, приведенный ниже, переопределяет поведение согласования по умолчанию, чтобы выбрать конечные точки в зависимости от расположения сервера приложений.

```cs
private class CustomRouter : EndpointRouterDecorator
{
    public override ServiceEndpoint GetNegotiateEndpoint(HttpContext context, IEnumerable<ServiceEndpoint> endpoints)
    {
        // Override the negotiate behavior to get the endpoint from query string
        var endpointName = context.Request.Query["endpoint"];
        if (endpointName.Count == 0)
        {
            context.Response.StatusCode = 400;
            var response = Encoding.UTF8.GetBytes("Invalid request");
            context.Response.Body.Write(response, 0, response.Length);
            return null;
        }

        return endpoints.FirstOrDefault(s => s.Name == endpointName && s.Online) // Get the endpoint with name matching the incoming request
               ?? base.GetNegotiateEndpoint(context, endpoints); // Or fallback to the default behavior to randomly select one from primary endpoints, or fallback to secondary when no primary ones are online
    }
}
```

Не забудьте зарегистрировать маршрутизатор для внедрения контейнера с помощью:

```cs
services.AddSingleton(typeof(IEndpointRouter), typeof(CustomRouter));
services.AddSignalR()
        .AddAzureSignalR(
            options => 
            {
                options.Endpoints = new ServiceEndpoint[]
                {
                    new ServiceEndpoint(name: "east", connectionString: "<connectionString1>"),
                    new ServiceEndpoint(name: "west", connectionString: "<connectionString2>"),
                    new ServiceEndpoint("<connectionString3>")
                };
            });
```

## <a name="for-aspnet"></a>Для ASP.NET

### <a name="how-to-add-multiple-endpoints-from-config"></a>Как добавить несколько конечных точек из конфигурации?

Config с ключом `Azure:SignalR:ConnectionString` или `Azure:SignalR:ConnectionString:` строкой подключения службы SignalR.

Если ключ начинается с `Azure:SignalR:ConnectionString:` , он должен быть в формате `Azure:SignalR:ConnectionString:{Name}:{EndpointType}` , где `Name` и `EndpointType` являются свойствами `ServiceEndpoint` объекта и доступны из кода.

В можно добавить несколько строк подключения к экземпляру `web.config` .

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <connectionStrings>
    <add name="Azure:SignalR:ConnectionString" connectionString="<ConnectionString1>"/>
    <add name="Azure:SignalR:ConnectionString:en-us" connectionString="<ConnectionString2>"/>
    <add name="Azure:SignalR:ConnectionString:zh-cn:secondary" connectionString="<ConnectionString3>"/>
    <add name="Azure:SignalR:ConnectionString:Backup:secondary" connectionString="<ConnectionString4>"/>
  </connectionStrings>
  ...
</configuration>
```

### <a name="how-to-add-multiple-endpoints-from-code"></a>Как добавить несколько конечных точек из кода?

`ServicEndpoint`Класс вводится для описания свойств конечной точки службы SignalR Azure.
При использовании пакета SDK службы SignalR для Azure можно настроить несколько конечных точек экземпляра с помощью:

```cs
app.MapAzureSignalR(
    this.GetType().FullName, 
    options => {
            options.Endpoints = new ServiceEndpoint[]
            {
                // Note: this is just a demonstration of how to set options.Endpoints
                // Having ConnectionStrings explicitly set inside the code is not encouraged
                // You can fetch it from a safe place such as Azure KeyVault
                new ServiceEndpoint("<ConnectionString1>"),
                new ServiceEndpoint("<ConnectionString2>"),
                new ServiceEndpoint("<ConnectionString3>"),
            }
        });
```

### <a name="how-to-customize-router"></a>Как настроить маршрутизатор?

Единственное различие между ASP.NET SignalR и ASP.NET Core SignalR — это тип контекста HTTP для `GetNegotiateEndpoint` . Для ASP.NET SignalR он имеет тип [иовинконтекст](https://github.com/Azure/azure-signalr/blob/dev/src/Microsoft.Azure.SignalR.AspNet/EndpointRouters/DefaultEndpointRouter.cs#L19) .

Ниже приведен пользовательский пример согласования для ASP.NET SignalR:

```cs
private class CustomRouter : EndpointRouterDecorator
{
    public override ServiceEndpoint GetNegotiateEndpoint(IOwinContext context, IEnumerable<ServiceEndpoint> endpoints)
    {
        // Override the negotiate behavior to get the endpoint from query string
        var endpointName = context.Request.Query["endpoint"];
        if (string.IsNullOrEmpty(endpointName))
        {
            context.Response.StatusCode = 400;
            context.Response.Write("Invalid request.");
            return null;
        }

        return endpoints.FirstOrDefault(s => s.Name == endpointName && s.Online) // Get the endpoint with name matching the incoming request
               ?? base.GetNegotiateEndpoint(context, endpoints); // Or fallback to the default behavior to randomly select one from primary endpoints, or fallback to secondary when no primary ones are online
    }
}
```

Не забудьте зарегистрировать маршрутизатор для внедрения контейнера с помощью:

```cs
var hub = new HubConfiguration();
var router = new CustomRouter();
hub.Resolver.Register(typeof(IEndpointRouter), () => router);
app.MapAzureSignalR(GetType().FullName, hub, options => {
    options.Endpoints = new ServiceEndpoint[]
                {
                    new ServiceEndpoint(name: "east", connectionString: "<connectionString1>"),
                    new ServiceEndpoint(name: "west", connectionString: "<connectionString2>"),
                    new ServiceEndpoint("<connectionString3>")
                };
});
```

## <a name="configuration-in-cross-region-scenarios"></a>Настройка в сценариях с разными регионами

`ServiceEndpoint`Объект имеет `EndpointType` свойство со значением `primary` или `secondary` .

`primary` конечные точки являются предпочтительными конечными точками для получения клиентского трафика и считаются более надежными сетевыми подключениями. `secondary` конечные точки считаются менее надежными сетевыми подключениями и используются только для получения трафика от сервера к клиенту, например для широковещательной передачи сообщений, а не для обмена данными между клиентом и сервером.

В вариантах между регионами сеть может быть нестабильной. Для одного сервера приложений, расположенного в *восточной части США*, конечная точка службы SignalR, расположенная в том же регионе *восточной части США* , может быть настроена как `primary` конечные точки в других регионах, помеченные как `secondary` . В этой конфигурации конечные точки служб в других регионах могут **получать** сообщения от этого сервера приложений *восточной части США* , но на этот сервер приложений не будут направляться клиенты, **перекрестные по регионам** . Архитектура показана на схеме ниже.

![Кросс-географические инфраструктуры](./media/signalr-howto-scale-multi-instances/cross_geo_infra.png)

Когда клиент пытается подключиться к `/negotiate` серверу приложений с помощью маршрутизатора по умолчанию, пакет SDK **случайным образом выбирает** одну конечную точку из набора доступных `primary` конечных точек. Если первичная конечная точка недоступна, пакет SDK **случайным образом выбирает** из всех доступных `secondary` конечных точек. Конечная точка помечается как **доступная** , если подключение между сервером и конечной точкой службы активно.

В сценарии с разными регионами, когда клиент пытается подключиться к `/negotiate` серверу приложений, размещенному в *восточной части США*, по умолчанию он всегда возвращает `primary` конечную точку, расположенную в том же регионе. Если все точки *восточной части США* недоступны, клиент перенаправляется на конечные точки в других регионах. В разделе, посвященном отработку отказа ниже, подробно описывается сценарий.

![Нормальное согласование](./media/signalr-howto-scale-multi-instances/normal_negotiate.png)

## <a name="fail-over"></a>Отработка отказа

Если все `primary` конечные точки недоступны, выбор клиента `/negotiate` из доступных `secondary` конечных точек. Для этого механизма отработки отказа требуется, чтобы каждая конечная точка была в качестве `primary` конечной точки хотя бы для одного сервера приложений.

![Отработка отказа](./media/signalr-howto-scale-multi-instances/failover_negotiate.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом руководство вы узнали, как настроить несколько экземпляров в одном приложении для масштабирования, сегментирования и использования в разных регионах.

Можно также использовать несколько конечных точек в сценариях высокого уровня доступности и аварийного восстановления.

> [!div class="nextstepaction"]
> [Настройка службы SignalR для аварийного восстановления и высокой доступности](./signalr-concept-disaster-recovery.md)
