---
title: Удаленное взаимодействие служб с помощью Java в Azure Service Fabric
description: Удаленное взаимодействие Service Fabric позволяет осуществлять обмен данными между службами и клиентами и службами Java с помощью удаленного вызова процедур.
author: PavanKunapareddyMSFT
ms.topic: conceptual
ms.date: 06/30/2017
ms.custom: devx-track-java
ms.author: pakunapa
ms.openlocfilehash: d53d20510db70d81aab796efab48de40c880bb3a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87316130"
---
# <a name="service-remoting-in-java-with-reliable-services"></a>Удаленное взаимодействие службы в Java с Reliable Services
> [!div class="op_single_selector"]
> * [C# в Windows](service-fabric-reliable-services-communication-remoting.md)
> * [Java в Linux](service-fabric-reliable-services-communication-remoting-java.md)
>
>

Для служб, которые не привязаны к определенному протоколу обмена данными или стеку, например веб-API, Windows Communication Foundation (WCF) и другим, платформа Reliable Services предоставляет механизм удаленного взаимодействия для быстрой и простой настройки удаленного вызова процедур.  В этой статье рассматривается, как настраивать удаленные вызовы процедур для служб на Java.

## <a name="set-up-remoting-on-a-service"></a>Настройка удаленного доступа в службе
Процесс настройки удаленного доступа для службы состоит из двух простых этапов.

1. Создание интерфейса для реализации в службе. Этот интерфейс определяет методы, которые будут доступны для удаленного вызова процедур в службе. Эти методы должны быть асинхронными методами, возвращающими задачи. Интерфейс должен реализовать `microsoft.serviceFabric.services.remoting.Service` , чтобы показать, что служба имеет интерфейс удаленного взаимодействия.
2. Используйте прослушиватель удаленного взаимодействия в службе. Это реализация `CommunicationListener` , которая предоставляет возможности удаленного взаимодействия. `FabricTransportServiceRemotingListener` можно использовать, чтобы создать прослушиватель удаленного взаимодействия с использованием транспортного протокола удаленного взаимодействия по умолчанию.

Например, приведенная ниже служба без отслеживания состояния предоставляет один метод для получения "Hello World" посредством удаленного вызова процедуры.

```java
import java.util.ArrayList;
import java.util.concurrent.CompletableFuture;
import java.util.List;
import microsoft.servicefabric.services.communication.runtime.ServiceInstanceListener;
import microsoft.servicefabric.services.remoting.Service;
import microsoft.servicefabric.services.runtime.StatelessService;

public interface MyService extends Service {
    CompletableFuture<String> helloWorldAsync();
}

class MyServiceImpl extends StatelessService implements MyService {
    public MyServiceImpl(StatelessServiceContext context) {
       super(context);
    }

    public CompletableFuture<String> helloWorldAsync() {
        return CompletableFuture.completedFuture("Hello!");
    }

    @Override
    protected List<ServiceInstanceListener> createServiceInstanceListeners() {
        ArrayList<ServiceInstanceListener> listeners = new ArrayList<>();
        listeners.add(new ServiceInstanceListener((context) -> {
            return new FabricTransportServiceRemotingListener(context,this);
        }));
        return listeners;
    }
}
```

> [!NOTE]
> Аргументы и возвращаемые данные в интерфейсе службы могут иметь простые, сложные или настраиваемые типы, но они должны быть сериализуемыми.
>
>

## <a name="call-remote-service-methods"></a>Вызов удаленных методов службы
Вызов методов в службе с помощью стека удаленного взаимодействия осуществляется с помощью локального прокси-сервера для службы через класс `microsoft.serviceFabric.services.remoting.client.ServiceProxyBase` . Метод `ServiceProxyBase` создает локальный прокси-сервер, используя тот же интерфейс, который реализует служба. С помощью этого прокси можно без труда удаленно вызвать методы в интерфейсе.

```java

MyService helloWorldClient = ServiceProxyBase.create(MyService.class, new URI("fabric:/MyApplication/MyHelloWorldService"));

CompletableFuture<String> message = helloWorldClient.helloWorldAsync();

```

Платформа удаленного взаимодействия распространяет исключения, созданные в службе, на клиент. Поэтому логика обработки исключений на стороне клиента с использованием `ServiceProxyBase` может напрямую обрабатывать порождаемые службой исключения.

## <a name="service-proxy-lifetime"></a>Время существования ServiceProxy
Создание ServiceProxy не требует больших ресурсов, поэтому вы можете создавать такие объекты в любых количествах. Экземпляры ServiceProxy можно использовать повторно. Если удаленный вызов процедуры создает исключение, вы по-прежнему можете использовать тот же экземпляр прокси-сервера. Каждый объект ServiceProxy содержит клиент обмена данными, используемый для отправки сообщений по сети. При запуске удаленных вызовов проводятся внутренние проверки для определения работоспособности этого клиента. В зависимости от результата проверки клиент может быть создан повторно. Поэтому, если возникает исключение, вам не нужно заново создавать `ServiceProxy`.

### <a name="serviceproxyfactory-lifetime"></a>Время существования ServiceProxyFactory
[FabricServiceProxyFactory](/java/api/microsoft.servicefabric.services.remoting.client.fabricserviceproxyfactory) — это фабрика, которая создает прокси-сервер для различных интерфейсов удаленного взаимодействия. Если вы используете API `ServiceProxyBase.create` для создания прокси-сервера, то платформа создает `FabricServiceProxyFactory`.
При необходимости переопределить свойства [ServiceRemotingClientFactory](/java/api/microsoft.servicefabric.services.remoting.client.serviceremotingclientfactory) имеет смысл создать фабрику вручную.
Создание фабрики — ресурсоемкая операция. `FabricServiceProxyFactory` хранит кэш клиента обмена данными.
Рекомендуется кэшировать `FabricServiceProxyFactory` на как можно больший период времени.

## <a name="remoting-exception-handling"></a>Обработка исключений удаленного взаимодействия
Все исключения удаленного взаимодействия, порождаемые API службы, отправляются обратно в клиент как RuntimeException или FabricException.

ServiceProxy обрабатывает все исключения отработки отказа для секции службы, для которого он создан. Он повторно разрешает конечные точки в случае исключений отработки отказа (повторяющихся исключений) и повторяет вызов к правильной конечной точке. Число повторных попыток для исключения отработки отказа не ограничено.
В случае исключений TransientException только повторяется попытка вызова.

Параметры повтора по умолчанию — предоставленного by [OperationRetrySettings](/java/api/microsoft.servicefabric.services.communication.client.operationretrysettings).
Вы можете настроить эти значения, передав объект OperationRetrySettings в конструктор ServiceProxyFactory.

## <a name="next-steps"></a>Дальнейшие действия
* [Защита обмена данными для Reliable Services](service-fabric-reliable-services-secure-communication-java.md)
