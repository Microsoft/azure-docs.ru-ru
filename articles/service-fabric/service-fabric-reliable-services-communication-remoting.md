---
title: Удаленное взаимодействие служб с помощью C# в Service Fabric
description: Удаленное взаимодействие Service Fabric позволяет осуществлять обмен данными между службами и клиентами и службами C# с помощью удаленного вызова процедур.
ms.topic: conceptual
ms.date: 09/20/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: a0486a27d76c978a65c4a3cfd81df52a12e4ea1d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98791583"
---
# <a name="service-remoting-in-c-with-reliable-services"></a>Удаленное взаимодействие в Reliable Services с помощью C#

> [!div class="op_single_selector"]
> * [C# в Windows](service-fabric-reliable-services-communication-remoting.md)
> * [Java в Linux](service-fabric-reliable-services-communication-remoting-java.md)
>
>

Для служб, которые не привязаны к определенному протоколу обмена данными или стеку, например веб-API, Windows Communication Foundation и другим, платформа Reliable Services предоставляет механизм удаленного взаимодействия для быстрой и простой настройки удаленного вызова процедур. В этой статье рассматривается, как настраивать удаленные вызовы процедур для служб на C#.

## <a name="set-up-remoting-on-a-service"></a>Настройка удаленного доступа в службе

Чтобы настроить для службы удаленный доступ, достаточно выполнить два простых шага:

1. Создание интерфейса для реализации в службе. Этот интерфейс определяет методы, которые будут доступны для удаленного вызова процедур в службе. Эти методы должны быть асинхронными методами, возвращающими задачи. Интерфейс должен реализовать `Microsoft.ServiceFabric.Services.Remoting.IService` , чтобы показать, что служба имеет интерфейс удаленного взаимодействия.
2. Используйте прослушиватель удаленного взаимодействия в службе. Прослушиватель удаленного взаимодействия — это реализация `ICommunicationListener`, которая предоставляет возможности удаленного взаимодействия. Пространство имен `Microsoft.ServiceFabric.Services.Remoting.Runtime` содержит метод расширения `CreateServiceRemotingInstanceListeners` для служб без отслеживания и с отслеживанием состояния, который позволяет создать прослушиватель удаленного взаимодействия с использованием соответствующего стандартного транспортного протокола.

>[!NOTE]
>Пространство имен `Remoting` доступно как отдельный пакет NuGet `Microsoft.ServiceFabric.Services.Remoting`.

Например, приведенная ниже служба без отслеживания состояния предоставляет один метод для получения "Hello World" посредством удаленного вызова процедуры.

```csharp
using Microsoft.ServiceFabric.Services.Communication.Runtime;
using Microsoft.ServiceFabric.Services.Remoting;
using Microsoft.ServiceFabric.Services.Remoting.Runtime;
using Microsoft.ServiceFabric.Services.Runtime;

public interface IMyService : IService
{
    Task<string> HelloWorldAsync();
}

class MyService : StatelessService, IMyService
{
    public MyService(StatelessServiceContext context)
        : base (context)
    {
    }

    public Task<string> HelloWorldAsync()
    {
        return Task.FromResult("Hello!");
    }

    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
     return this.CreateServiceRemotingInstanceListeners();
    }
}
```

> [!NOTE]
> Аргументы и возвращаемые данные в интерфейсе службы могут иметь простые, сложные или настраиваемые типы, но они должны поддерживать сериализацию с помощью .NET [DataContractSerializer](/dotnet/framework/wcf/feature-details/types-supported-by-the-data-contract-serializer).
>
>

## <a name="call-remote-service-methods"></a>Вызов удаленных методов службы

Вызов методов в службе с помощью стека удаленного взаимодействия осуществляется с помощью локального прокси-сервера для службы через класс `Microsoft.ServiceFabric.Services.Remoting.Client.ServiceProxy` . Метод `ServiceProxy` создает локальный прокси-сервер, используя тот же интерфейс, который реализует служба. С помощью этого прокси-сервера можно удаленно вызвать методы в интерфейсе.

```csharp

IMyService helloWorldClient = ServiceProxy.Create<IMyService>(new Uri("fabric:/MyApplication/MyHelloWorldService"));

string message = await helloWorldClient.HelloWorldAsync();

```

Платформа удаленного взаимодействия распространяет исключения, созданные службой, на клиент. Поэтому при использовании метода `ServiceProxy` клиент отвечает за обработку исключений, которые создает служба.

## <a name="service-proxy-lifetime"></a>Время существования прокси службы

Создание прокси службы не требует больших ресурсов, поэтому вы можете создавать такие объекты в любых количествах. Экземпляры прокси службы можно использовать повторно. Если удаленный вызов процедуры создает исключение, вы по-прежнему можете использовать тот же экземпляр прокси-сервера. Каждый объект прокси-сервера службы содержит клиент обмена данными для отправки сообщений по сети. При запуске удаленных вызовов проводятся внутренние проверки для определения работоспособности этого клиента. В зависимости от результата проверки клиент может быть создан повторно. Поэтому, если возникает исключение, вам не нужно заново создавать `ServiceProxy`.

### <a name="service-proxy-factory-lifetime"></a>Время существования фабрики прокси-сервера службы

[ServiceProxyFactory](/dotnet/api/microsoft.servicefabric.services.remoting.client.serviceproxyfactory) — это фабрика, которая создает экземпляры прокси-сервера для различных интерфейсов удаленного взаимодействия. Если для создания прокси-сервера вы используете API `ServiceProxyFactory.CreateServiceProxy`, платформа создает отдельный прокси-сервер службы.
При необходимости переопределить свойства [IServiceRemotingClientFactory](/dotnet/api/microsoft.servicefabric.services.remoting.v1.client.iserviceremotingclientfactory) имеет смысл создать фабрику вручную.
Создание фабрики — ресурсоемкая операция. Фабрика прокси службы поддерживает внутренний кэш для клиента обмена данными.
Мы рекомендуем кэшировать фабрику прокси-сервера службы на максимально возможный период времени.

## <a name="remoting-exception-handling"></a>Обработка исключений удаленного взаимодействия

Все исключения удаленного взаимодействия, порождаемые API службы, отправляются обратно в клиент как AggregateException. Исключения удаленного взаимодействия должны поддерживать сериализацию с помощью DataContract. В противном случае API прокси-сервера создает исключение [ServiceException](/dotnet/api/microsoft.servicefabric.services.communication.serviceexception) с ошибкой сериализации.

Прокси-сервер службы обрабатывает все исключения отработки отказа в той секции службы, для которой он создан. Он повторно разрешает конечные точки в случае исключений отработки отказа (повторяющихся исключений) и повторяет вызов к правильной конечной точке. Число повторных попыток для исключений отработки отказа не ограничено.
Если порождаются временные исключения, прокси-сервер повторяет вызов.

Параметры повтора по умолчанию определяются [OperationRetrySettings](/dotnet/api/microsoft.servicefabric.services.communication.client.operationretrysettings).

Пользователь может настроить эти значения, передав объект OperationRetrySettings в конструктор ServiceProxyFactory.

## <a name="use-the-remoting-v2-stack"></a>Использование стека удаленного взаимодействия версии 2

Пакет NuGet для удаленного взаимодействия версии 2.8 позволяет использовать стек удаленного взаимодействия версии 2. Стек удаленного взаимодействия версии 2 работает лучше. Также он предоставляет дополнительные функции, например пользовательскую сериализацию, и больше подключаемых API.
Код шаблона по-прежнему использует стек удаленного взаимодействия версии 1.
Удаленное взаимодействие версии 2 несовместим с версией 1 (предыдущий стек удаленного взаимодействия). Чтобы это не повлияло на доступность службы, соблюдайте инструкции из руководства по [обновлению с версии 1 до версии 2](#upgrade-from-remoting-v1-to-remoting-v2).

Можно использовать следующий подход для включения стека версии 2.

### <a name="use-an-assembly-attribute-to-use-the-v2-stack"></a>Использование атрибута сборки для перехода на стек версии 2

Этот шаг изменяет код шаблона, добавляя атрибут сборки для использования стека версии 2.

1. Измените ресурс конечной точки с `"ServiceEndpoint"` на `"ServiceEndpointV2"` в манифесте службы.

   ```xml
   <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpointV2" />
    </Endpoints>
   </Resources>
   ```

2. Используйте метод расширения `Microsoft.ServiceFabric.Services.Remoting.Runtime.CreateServiceRemotingInstanceListeners`, чтобы создать прослушивателей удаленного взаимодействия (одинаково для версии 1 и 2).

   ```csharp
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return this.CreateServiceRemotingInstanceListeners();
    }
   ```

3. Пометьте сборку с удаленными интерфейсами атрибутом `FabricTransportServiceRemotingProvider`.

   ```csharp
   [assembly: FabricTransportServiceRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2, RemotingClientVersion = RemotingClientVersion.V2)]
   ```

Вносить изменения в проект клиента не требуется.
Скомпилируйте сборку клиента со сборкой интерфейса, чтобы гарантировать использование упомянутого выше атрибута сборки.

### <a name="use-explicit-v2-classes-to-use-the-v2-stack"></a>Использования явных классов версии 2 для перехода на стек версии 2

Вместо атрибута сборки можно использовать явные классы версии 2, чтобы включить стек версии 2.

Этот шаг изменяет код шаблона, добавляя явные классы версии 2 для использования стека версии 2.

1. Измените ресурс конечной точки с `"ServiceEndpoint"` на `"ServiceEndpointV2"` в манифесте службы.

   ```xml
   <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpointV2" />
    </Endpoints>
   </Resources>
   ```

2. Используйте [FabricTransportServiceRemotingListener](/dotnet/api/microsoft.servicefabric.services.remoting.v2.fabrictransport.runtime.fabrictransportserviceremotinglistener) из пространства имен `Microsoft.ServiceFabric.Services.Remoting.V2.FabricTransport.Runtime`.

   ```csharp
   protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return new[]
        {
            new ServiceInstanceListener((c) =>
            {
                return new FabricTransportServiceRemotingListener(c, this);

            })
        };
    }
   ```

3. Используйте [FabricTransportServiceRemotingClientFactory](/dotnet/api/microsoft.servicefabric.services.remoting.v2.fabrictransport.client.fabrictransportserviceremotingclientfactory) из пространства имен `Microsoft.ServiceFabric.Services.Remoting.V2.FabricTransport.Client`, чтобы создать клиенты.

   ```csharp
   var proxyFactory = new ServiceProxyFactory((c) =>
          {
              return new FabricTransportServiceRemotingClientFactory();
          });
   ```

## <a name="upgrade-from-remoting-v1-to-remoting-v2"></a>Обновление службы удаленного взаимодействия с версии 1 до версии 2

Чтобы обновить версию 1 до версии 2, выполните двухэтапное обновление. Соблюдайте порядок действий в этой процедуре.

1. Обновите службу версии 1 до версии 2 с помощью указанного атрибута.
Такое изменение гарантирует, что служба будет ожидать передачу данных от прослушивателей версии 1 и 2.

    а. Добавьте в манифест службы ресурс конечной точки с именем ServiceEndpointV2.
      ```xml
      <Resources>
        <Endpoints>
          <Endpoint Name="ServiceEndpointV2" />  
        </Endpoints>
      </Resources>
      ```

    b. Используйте приведенный ниже метод расширения, чтобы создать прослушиватель удаленного взаимодействия.

    ```csharp
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return this.CreateServiceRemotingInstanceListeners();
    }
    ```

    c. Добавьте атрибут сборки в интерфейсы удаленного взаимодействия, чтобы использовать прослушиватели версий 1 и 2 и клиент версии 2.
    ```csharp
    [assembly: FabricTransportServiceRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2|RemotingListenerVersion.V1, RemotingClientVersion = RemotingClientVersion.V2)]

      ```
2. Обновите клиент версии 1 до версии 2 с помощью атрибута клиента версии 2.
Этот шаг гарантирует, что клиент будет использовать стек версии 2.
Никаких изменений в проект или службу клиента вносить не требуется. Достаточно выполнить сборку проектов клиента с обновленной сборкой интерфейса.

3. Этот шаг является необязательным. Используйте атрибут прослушивателя версии 2 и обновите службу версии 2.
Этот шаг гарантирует, что служба будет ожидать передачу данных только от прослушивателя версии 2.

    ```csharp
    [assembly: FabricTransportServiceRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2, RemotingClientVersion = RemotingClientVersion.V2)]
    ```


## <a name="use-the-remoting-v2-interface-compatible-stack"></a>Использование стека удаленного взаимодействия версии 2 (с совместимым интерфейсом)

 Стек удаленного взаимодействия версии 2 (с совместимым интерфейсом, известный как V2_1) обладает всеми функциями стека удаленного взаимодействия версии 2. Стек его интерфейса совместим со стеком удаленного взаимодействия версии 1, но не обладает обратной совместимостью с версиями 1 и 2. Чтобы выполнить обновление версии 1 до версии 2_1, не влияя на доступность службы, следуйте действиям, описанным в этой статье.


### <a name="use-an-assembly-attribute-to-use-the-remoting-v2-interface-compatible-stack"></a>Использование атрибута сборки для перехода на стек удаленного взаимодействия версии 2 (с совместимым интерфейсом)

Выполните следующие действия, чтобы перейти на стек V2_1.

1. Добавьте в манифест службы ресурс конечной точки с именем ServiceEndpointV2_1.

   ```xml
   <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpointV2_1" />  
    </Endpoints>
   </Resources>
   ```

2. Используйте метод расширения удаленного взаимодействия, чтобы создать прослушиватель удаленного взаимодействия.

   ```csharp
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return this.CreateServiceRemotingInstanceListeners();
    }
   ```

3. Добавьте [атрибут сборки](/dotnet/api/microsoft.servicefabric.services.remoting.fabrictransport.fabrictransportserviceremotingproviderattribute) в интерфейсы удаленного взаимодействия.

   ```csharp
    [assembly:  FabricTransportServiceRemotingProvider(RemotingListenerVersion=  RemotingListenerVersion.V2_1, RemotingClientVersion= RemotingClientVersion.V2_1)]

   ```

Вносить изменения в проект клиента не требуется.
Скомпилируйте сборку клиента со сборкой интерфейса, чтобы гарантировать использование упомянутого выше атрибута сборки.

### <a name="use-explicit-remoting-classes-to-create-a-listenerclient-factory-for-the-v2-interface-compatible-version"></a>Использование явных классов удаленного взаимодействия для создания прослушивателя и фабрики клиента для версии 2 (с совместимым интерфейсом)

Выполните следующие действия.

1. Добавьте в манифест службы ресурс конечной точки с именем ServiceEndpointV2_1.

   ```xml
   <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpointV2_1" />  
    </Endpoints>
   </Resources>
   ```

2. Используйте [прослушиватель удаленного взаимодействия версии 2](/dotnet/api/microsoft.servicefabric.services.remoting.v2.fabrictransport.runtime.fabrictransportserviceremotinglistener). По умолчанию используется ресурс конечной точки службы с именем ServiceEndpointV2. Он должен быть определен в манифесте службы.

   ```csharp
   protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return new[]
        {
            new ServiceInstanceListener((c) =>
            {
                var settings = new FabricTransportRemotingListenerSettings();
                settings.UseWrappedMessage = true;
                return new FabricTransportServiceRemotingListener(c, this,settings);

            })
        };
    }
   ```

3. Используйте [фабрику клиента](/dotnet/api/microsoft.servicefabric.services.remoting.v2.fabrictransport.client.fabrictransportserviceremotingclientfactory) версии 2.
   ```csharp
   var proxyFactory = new ServiceProxyFactory((c) =>
          {
            var settings = new FabricTransportRemotingSettings();
            settings.UseWrappedMessage = true;
            return new FabricTransportServiceRemotingClientFactory(settings);
          });
   ```

## <a name="upgrade-from-remoting-v1-to-remoting-v2-interface-compatible"></a>Обновление службы удаленного взаимодействия с версии 1 до версии 2 (с совместимым интерфейсом).

Чтобы обновить версию 1 до версии 2 (с совместимым интерфейсом, или версия 2_1), выполните двухэтапное обновление. Соблюдайте порядок действий в этой процедуре.

> [!NOTE]
> При обновлении с версии v1 на v2 убедитесь, что `Remoting` пространство имен Обновлено для использования версии 2. Пример: "Microsoft. ServiceFabric. Services. Remoting. v2. FabricTransport. Client"
>
>

1. Обновите службу версии 1 до версии 2_1 с помощью указанного ниже атрибута.
Это изменение гарантирует, что служба будет ожидать передачи данных от прослушивателей версии 1 и 2_1.

    а. Добавьте в манифест службы ресурс конечной точки с именем ServiceEndpointV2_1.
      ```xml
      <Resources>
        <Endpoints>
          <Endpoint Name="ServiceEndpointV2_1" />  
        </Endpoints>
      </Resources>
      ```

    b. Используйте приведенный ниже метод расширения, чтобы создать прослушиватель удаленного взаимодействия.

    ```csharp
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return this.CreateServiceRemotingInstanceListeners();
    }
    ```

    c. Добавьте атрибут сборки в интерфейсы удаленного взаимодействия, чтобы использовать прослушиватель версии 1 и 2_1 и клиент версии 2_1.
    ```csharp
   [assembly: FabricTransportServiceRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2_1 | RemotingListenerVersion.V1, RemotingClientVersion = RemotingClientVersion.V2_1)]

      ```
2. Обновите клиент версии 1 до версии 2_1 с помощью атрибута клиента версии 2_1.
Это гарантирует, что клиент будет использовать стек версии 2_1.
Никаких изменений в проект или службу клиента вносить не требуется. Достаточно выполнить сборку проектов клиента с обновленной сборкой интерфейса.

3. Этот шаг является необязательным. Удалите из атрибута прослушивателя версии 1 и обновите службу версии 2.
Этот шаг гарантирует, что служба будет ожидать передачу данных только от прослушивателя версии 2.

    ```csharp
    [assembly: FabricTransportServiceRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2_1, RemotingClientVersion = RemotingClientVersion.V2_1)]
    ```
  
### <a name="use-custom-serialization-with-a-remoting-wrapped-message"></a>Использование настраиваемой сериализации с упакованным сообщением удаленного взаимодействия

В упакованном сообщении удаленного взаимодействия мы создаем отдельный упакованный объект, в поле которого хранятся все параметры.
Выполните следующие действия.

1. Реализуйте интерфейс `IServiceRemotingMessageSerializationProvider`, чтобы обеспечить пользовательскую сериализацию.
    В этом фрагменте кода показан пример такой реализации.

      ```csharp
      public class ServiceRemotingJsonSerializationProvider : IServiceRemotingMessageSerializationProvider
      {
        public IServiceRemotingMessageBodyFactory CreateMessageBodyFactory()
        {
          return new JsonMessageFactory();
        }

        public IServiceRemotingRequestMessageBodySerializer CreateRequestMessageSerializer(Type serviceInterfaceType, IEnumerable<Type> requestWrappedType, IEnumerable<Type> requestBodyTypes = null)
        {
          return new ServiceRemotingRequestJsonMessageBodySerializer();
        }

        public IServiceRemotingResponseMessageBodySerializer CreateResponseMessageSerializer(Type serviceInterfaceType, IEnumerable<Type> responseWrappedType, IEnumerable<Type> responseBodyTypes = null)
        {
          return new ServiceRemotingResponseJsonMessageBodySerializer();
        }
      }
      ```
      ```csharp
        class JsonMessageFactory : IServiceRemotingMessageBodyFactory
            {

              public IServiceRemotingRequestMessageBody CreateRequest(string interfaceName, string methodName, int numberOfParameters, object wrappedRequestObject)
              {
                return new JsonBody(wrappedRequestObject);
              }

              public IServiceRemotingResponseMessageBody CreateResponse(string interfaceName, string methodName, object wrappedRequestObject)
              {
                return new JsonBody(wrappedRequestObject);
              }
            }
      ```
      ```csharp
      class ServiceRemotingRequestJsonMessageBodySerializer : IServiceRemotingRequestMessageBodySerializer
        {
            private JsonSerializer serializer;

            public ServiceRemotingRequestJsonMessageBodySerializer()
            {
              serializer = JsonSerializer.Create(new JsonSerializerSettings()
              {
                TypeNameHandling = TypeNameHandling.All
                });
              }

              public IOutgoingMessageBody Serialize(IServiceRemotingRequestMessageBody serviceRemotingRequestMessageBody)
             {
               if (serviceRemotingRequestMessageBody == null)
               {
                 return null;
               }          
               using (var writeStream = new MemoryStream())
               {
                 using (var jsonWriter = new JsonTextWriter(new StreamWriter(writeStream)))
                 {
                   serializer.Serialize(jsonWriter, serviceRemotingRequestMessageBody);
                   jsonWriter.Flush();
                   var bytes = writeStream.ToArray();
                   var segment = new ArraySegment<byte>(bytes);
                   var segments = new List<ArraySegment<byte>> { segment };
                   return new OutgoingMessageBody(segments);
                 }
               }
              }

              public IServiceRemotingRequestMessageBody Deserialize(IIncomingMessageBody messageBody)
             {
               using (var sr = new StreamReader(messageBody.GetReceivedBuffer()))
               {
                 using (JsonReader reader = new JsonTextReader(sr))
                 {
                   var ob = serializer.Deserialize<JsonBody>(reader);
                   return ob;
                 }
               }
             }
            }
      ```
      ```csharp
      class ServiceRemotingResponseJsonMessageBodySerializer : IServiceRemotingResponseMessageBodySerializer
       {
         private JsonSerializer serializer;

        public ServiceRemotingResponseJsonMessageBodySerializer()
        {
          serializer = JsonSerializer.Create(new JsonSerializerSettings()
          {
              TypeNameHandling = TypeNameHandling.All
            });
          }

          public IOutgoingMessageBody Serialize(IServiceRemotingResponseMessageBody responseMessageBody)
          {
            if (responseMessageBody == null)
            {
              return null;
            }

            using (var writeStream = new MemoryStream())
            {
              using (var jsonWriter = new JsonTextWriter(new StreamWriter(writeStream)))
              {
                serializer.Serialize(jsonWriter, responseMessageBody);
                jsonWriter.Flush();
                var bytes = writeStream.ToArray();
                var segment = new ArraySegment<byte>(bytes);
                var segments = new List<ArraySegment<byte>> { segment };
                return new OutgoingMessageBody(segments);
              }
            }
          }

          public IServiceRemotingResponseMessageBody Deserialize(IIncomingMessageBody messageBody)
          {

             using (var sr = new StreamReader(messageBody.GetReceivedBuffer()))
             {
               using (var reader = new JsonTextReader(sr))
               {
                 var obj = serializer.Deserialize<JsonBody>(reader);
                 return obj;
               }
             }
           }
       }
    ```
    ```csharp
    class JsonBody : WrappedMessage, IServiceRemotingRequestMessageBody, IServiceRemotingResponseMessageBody
    {
          public JsonBody(object wrapped)
          {
            this.Value = wrapped;
          }

          public void SetParameter(int position, string parameName, object parameter)
          {  //Not Needed if you are using WrappedMessage
            throw new NotImplementedException();
          }

          public object GetParameter(int position, string parameName, Type paramType)
          {
            //Not Needed if you are using WrappedMessage
            throw new NotImplementedException();
          }

          public void Set(object response)
          { //Not Needed if you are using WrappedMessage
            throw new NotImplementedException();
          }

          public object Get(Type paramType)
          {  //Not Needed if you are using WrappedMessage
            throw new NotImplementedException();
          }
    }
    ```

2. Переопределите поставщик сериализации по умолчанию, указав `JsonSerializationProvider` в качестве прослушивателя удаленного взаимодействия.

   ```csharp
   protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
   {
       return new[]
       {
           new ServiceInstanceListener((c) =>
           {
               return new FabricTransportServiceRemotingListener(c, this,
                   new ServiceRemotingJsonSerializationProvider());
           })
       };
   }
   ```

3. Переопределите поставщик сериализации по умолчанию, указав `JsonSerializationProvider` в качестве фабрики клиента удаленного взаимодействия.

    ```csharp
    var proxyFactory = new ServiceProxyFactory((c) =>
    {
        return new FabricTransportServiceRemotingClientFactory(
        serializationProvider: new ServiceRemotingJsonSerializationProvider());
      });
      ```

## <a name="next-steps"></a>Дальнейшие действия

* [Веб-API с OWIN в модели Reliable Services](./service-fabric-reliable-services-communication-aspnetcore.md)
* [WCF-based communication stack for Reliable Services](service-fabric-reliable-services-communication-wcf.md) (Стек взаимодействия для Reliable Services на основе WCF)
* [Secure service remoting communications in a C# service](service-fabric-reliable-services-secure-communication.md) (Защита удаленного взаимодействия в службе C#)
