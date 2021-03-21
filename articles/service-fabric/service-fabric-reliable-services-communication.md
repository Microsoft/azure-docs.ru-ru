---
title: Общие сведения о Reliable Services связи
description: Общие сведения о модели обмена данными с Reliable Services, включая открытие прослушивателей для служб, разрешение конечных точек и обмен данными между службами.
ms.topic: conceptual
ms.date: 11/01/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: 3436d29446e963faea9bda47f5a5247b7de7d859
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97912620"
---
# <a name="how-to-use-the-reliable-services-communication-apis"></a>Использование API связи служб Reliable Services
Обмен данными между службами совершенно не влияет на работу платформы Azure Service Fabric. Допускается использование любых протоколов и стеков, от UDP до HTTP. Способ взаимодействия служб зависит только от выбора разработчика. Платформа приложений служб Reliable Services предоставляет несколько встроенных стеков связи, а также API-интерфейсы для создания пользовательских компонентов связи.

## <a name="set-up-service-communication"></a>Настройка обмена данными между службами
API Reliable Services использует простой интерфейс для взаимодействия служб. Чтобы открыть конечную точку для службы, просто реализуйте этот интерфейс:

```csharp

public interface ICommunicationListener
{
    Task<string> OpenAsync(CancellationToken cancellationToken);

    Task CloseAsync(CancellationToken cancellationToken);

    void Abort();
}

```

```java
public interface CommunicationListener {
    CompletableFuture<String> openAsync(CancellationToken cancellationToken);

    CompletableFuture<?> closeAsync(CancellationToken cancellationToken);

    void abort();
}
```

Затем добавьте реализацию прослушивателя связи, вернув его в переопределении метода класса на основе службы.

Для служб без отслеживания состояния:

```csharp
public class MyStatelessService : StatelessService
{
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        ...
    }
    ...
}
```
```java
public class MyStatelessService extends StatelessService {

    @Override
    protected List<ServiceInstanceListener> createServiceInstanceListeners() {
        ...
    }
    ...
}
```

Для служб с отслеживанием состояния:

```java
    @Override
    protected List<ServiceReplicaListener> createServiceReplicaListeners() {
        ...
    }
    ...
```

```csharp
public class MyStatefulService : StatefulService
{
    protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
    {
        ...
    }
    ...
}
```

В обоих случаях возвращается коллекция прослушивателей. Это позволяет службе прослушивать несколько конечных точек, потенциально с помощью различных протоколов, используя несколько прослушивателей. Например, можно использовать прослушиватель HTTP и отдельный прослушиватель WebSocket. Каждый прослушиватель получает имя, а получившаяся коллекция пар *имя:адрес* представляется как объект JSON, когда клиент запрашивает адреса ожидания для секции или экземпляра службы.

Для службы без отслеживания состояния переопределение возвращает коллекцию объектов ServiceInstanceListener. Объект `ServiceInstanceListener` может создавать интерфейс `ICommunicationListener(C#) / CommunicationListener(Java)` и присваивать ему имя. Для служб с отслеживанием состояния переопределение возвращает коллекцию объектов ServiceReplicaListener. Здесь есть небольшое отличие от служб без отслеживания состояния, так как `ServiceReplicaListener` может открывать интерфейс `ICommunicationListener` для вторичных реплик. Это позволяет не только использовать несколько прослушивателей связи в одной службе, но также указывать, какие прослушиватели принимают запросы для вторичных реплик, а какие — прослушивают только первичные реплики.

Например, у вас есть прослушиватель ServiceRemotingListener, который принимает вызовы RPC только для первичных реплик, и настраиваемый прослушиватель, который принимает запросы на чтение для вторичных реплик по протоколу HTTP:

```csharp
protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
{
    return new[]
    {
        new ServiceReplicaListener(context =>
            new MyCustomHttpListener(context),
            "HTTPReadonlyEndpoint",
            true),

        new ServiceReplicaListener(context =>
            this.CreateServiceRemotingListener(context),
            "rpcPrimaryEndpoint",
            false)
    };
}
```

> [!NOTE]
> При создании нескольких прослушивателей для службы каждому прослушивателю **необходимо** присвоить уникальное имя.
>
>

Наконец, вы можете описать конечные точки, которые требуются для службы, в [манифесте служб](service-fabric-application-and-service-manifests.md) в разделе конечных точек.

```xml
<Resources>
    <Endpoints>
      <Endpoint Name="WebServiceEndpoint" Protocol="http" Port="80" />
      <Endpoint Name="OtherServiceEndpoint" Protocol="tcp" Port="8505" />
    <Endpoints>
</Resources>

```

Прослушиватель связи может обращаться к ресурсам конечной точки, выделенным ему из `CodePackageActivationContext` в `ServiceContext`, и начинать прослушивать запросы при открытии.

```csharp
var codePackageActivationContext = serviceContext.CodePackageActivationContext;
var port = codePackageActivationContext.GetEndpoint("ServiceEndpoint").Port;

```
```java
CodePackageActivationContext codePackageActivationContext = serviceContext.getCodePackageActivationContext();
int port = codePackageActivationContext.getEndpoint("ServiceEndpoint").getPort();

```

> [!NOTE]
> Ресурсы конечных точек являются общими для всего пакета служб. Они выделяются платформой Service Fabric при активации пакета служб. Множество реплик, размещенных на одном и том же объекте ServiceHost, используют один и тот же порт. Это означает, что прослушиватель связи должен поддерживать совместное использование портов. Для этого при создании адреса прослушивания мы рекомендуем использовать для прослушивателя связи идентификатор раздела и идентификатор реплики или экземпляра.
>
>

### <a name="service-address-registration"></a>Регистрация адреса службы
Системная *служба именования* работает на кластерах Service Fabric. Служба именования — это регистратор для служб и их адресов, которые прослушивает каждый экземпляр или реплика службы. Когда метод `OpenAsync(C#) / openAsync(Java)``ICommunicationListener(C#) / CommunicationListener(Java)` завершает выполнение, возвращаемое значение регистрируется в службе именования. Значение, которое публикуется в службе именования, — это строка, значением которой может что угодно. Это строковое значение определяет, что клиенты могут видеть при запросе адреса службы у службы именования.

```csharp
public Task<string> OpenAsync(CancellationToken cancellationToken)
{
    EndpointResourceDescription serviceEndpoint = serviceContext.CodePackageActivationContext.GetEndpoint("ServiceEndpoint");
    int port = serviceEndpoint.Port;

    this.listeningAddress = string.Format(
                CultureInfo.InvariantCulture,
                "http://+:{0}/",
                port);

    this.publishAddress = this.listeningAddress.Replace("+", FabricRuntime.GetNodeContext().IPAddressOrFQDN);

    this.webApp = WebApp.Start(this.listeningAddress, appBuilder => this.startup.Invoke(appBuilder));

    // the string returned here will be published in the Naming Service.
    return Task.FromResult(this.publishAddress);
}
```
```java
public CompletableFuture<String> openAsync(CancellationToken cancellationToken)
{
    EndpointResourceDescription serviceEndpoint = serviceContext.getCodePackageActivationContext.getEndpoint("ServiceEndpoint");
    int port = serviceEndpoint.getPort();

    this.publishAddress = String.format("http://%s:%d/", FabricRuntime.getNodeContext().getIpAddressOrFQDN(), port);

    this.webApp = new WebApp(port);
    this.webApp.start();

    /* the string returned here will be published in the Naming Service.
     */
    return CompletableFuture.completedFuture(this.publishAddress);
}
```

Платформа Service Fabric предоставляет API-интерфейсы, которые позволяют клиентам и другим службам в дальнейшем запрашивать этот адрес по имени службы. Это важно, так как у службы нестатический адрес. Перемещение служб в кластере осуществляется для балансировки ресурсов и обеспечения их доступности. Это механизм, позволяющий клиентам распознавать адрес прослушивания для службы.

> [!NOTE]
> Полное пошаговое руководство по созданию прослушивателя каналов связи см. в статье [ASP.NET Core в Service Fabric Reliable Services](./service-fabric-reliable-services-communication-aspnetcore.md) (для C#). Для Java вы можете написать собственную реализацию HTTP-сервера. Пример приложения EchoServer см. на странице https://github.com/Azure-Samples/service-fabric-java-getting-started.
>
>

## <a name="communicating-with-a-service"></a>Взаимодействие со службой
API-интерфейс служб Reliable Services предоставляет следующие библиотеки для написания клиентов, взаимодействующих со службами.

### <a name="service-endpoint-resolution"></a>Разрешение конечной точки службы
Первый шаг взаимодействия со службой — разрешение адреса конечной точки раздела или экземпляра службы, с которой вы хотите связаться. Служебный класс `ServicePartitionResolver(C#) / FabricServicePartitionResolver(Java)` — это базовый примитивный тип, который помогает клиентам определить конечную точку во время выполнения. При использовании Service Fabric процесс определения конечной точки службы называется *разрешением конечной точки службы*.

Чтобы подключиться к службам в кластере, можно создать ServicePartitionResolver, используя параметры по умолчанию. Это рекомендуется для большинства сценариев:

```csharp
ServicePartitionResolver resolver = ServicePartitionResolver.GetDefault();
```
```java
FabricServicePartitionResolver resolver = FabricServicePartitionResolver.getDefault();
```

Чтобы подключиться к службам в другом кластере, ServicePartitionResolver можно создать с набором конечных точек шлюза кластера. Обратите внимание, что конечные точки шлюза — это просто другие конечные точки для подключения к тому же кластеру. Пример:

```csharp
ServicePartitionResolver resolver = new  ServicePartitionResolver("mycluster.cloudapp.azure.com:19000", "mycluster.cloudapp.azure.com:19001");
```
```java
FabricServicePartitionResolver resolver = new  FabricServicePartitionResolver("mycluster.cloudapp.azure.com:19000", "mycluster.cloudapp.azure.com:19001");
```

Кроме того, `ServicePartitionResolver` можно передать функцию, чтобы создать `FabricClient` для внутреннего использования.

```csharp
public delegate FabricClient CreateFabricClientDelegate();
```
```java
public FabricServicePartitionResolver(CreateFabricClient createFabricClient) {
...
}

public interface CreateFabricClient {
    public FabricClient getFabricClient();
}
```

`FabricClient` — это объект, который используется для взаимодействия с кластером Service Fabric в рамках различных операций управления в кластере. Это полезно, когда требуется больший контроль над тем, как класс ServicePartitionResolver взаимодействует с кластером. `FabricClient` выполняет внутреннее кэширование и обычно требует много ресурсов для создания, поэтому важно повторно использовать как можно больше экземпляров `FabricClient`.

```csharp
ServicePartitionResolver resolver = new  ServicePartitionResolver(() => CreateMyFabricClient());
```
```java
FabricServicePartitionResolver resolver = new  FabricServicePartitionResolver(() -> new CreateFabricClientImpl());
```

Метод resolve затем используется для получения адреса службы или раздела секционированных служб.

```csharp
ServicePartitionResolver resolver = ServicePartitionResolver.GetDefault();

ResolvedServicePartition partition =
    await resolver.ResolveAsync(new Uri("fabric:/MyApp/MyService"), new ServicePartitionKey(), cancellationToken);
```
```java
FabricServicePartitionResolver resolver = FabricServicePartitionResolver.getDefault();

CompletableFuture<ResolvedServicePartition> partition =
    resolver.resolveAsync(new URI("fabric:/MyApp/MyService"), new ServicePartitionKey());
```

Адрес службы можно разрешить с помощью ServicePartitionResolver, но потребуется дополнительная работа, чтобы разрешенный адрес можно было использовать правильно. Клиенту необходимо определить, произошел ли сбой из-за временной ошибки и можно ли его повторить (например, служба была перемещена или временно недоступна) или из-за постоянной ошибки (например, служба была удалена или запрошенный ресурс больше не существует). Экземпляры или реплики службы можно перемещать между узлами в любое время по различным причинам. Адрес службы, который разрешается с помощью ServicePartitionResolver, может устареть к тому времени, когда клиентский код попытается к ней подключиться. В этом случае клиент должен повторно разрешить адрес. Предыдущий `ResolvedServicePartition` указывает, что сопоставителю необходимо повторить попытку, вместо того чтобы просто получить кэшированный адрес.

Обычно в коде клиента не предусмотрена непосредственная работа с классом ServicePartitionResolver. Он создается и передается фабрикам клиента в API служб Reliable Services. Фабрики используют сопоставитель для создания клиентского объекта, который может применяться для взаимодействия со службами.

### <a name="communication-clients-and-factories"></a>Клиенты и фабрики связи
Библиотека фабрики связи реализует типичный шаблон повторных попыток обработки ошибок, упрощающий повтор подключений к конечным точкам разрешенной службы. Библиотека фабрики предоставляет механизм повтора, а вы — обработчики ошибок.

Класс `ICommunicationClientFactory(C#) / CommunicationClientFactory(Java)` определяет базовый интерфейс, реализуемый фабрикой клиента связи, которая создает клиенты, способные обмениваться данными со службой Service Fabric. Реализация CommunicationClientFactory зависит от стека связи, используемого службой Service Fabric, с которой клиенту требуется связаться. API Reliable Services предоставляет `CommunicationClientFactoryBase<TCommunicationClient>`. Это обеспечивает базовую реализацию интерфейса CommunicationClientFactory и выполнение задач, которые являются общими для всех стеков связи. (Эти задачи включают в себя использование класса ServicePartitionResolver для определения конечной точки службы.) Клиенты обычно реализуют абстрактный класс CommunicationClientFactoryBase для обработки специфической логики стека связи.

Клиент связи просто получает адрес и использует его для подключения к службе. Клиент может использовать любой протокол.

```csharp
public class MyCommunicationClient : ICommunicationClient
{
    public ResolvedServiceEndpoint Endpoint { get; set; }

    public string ListenerName { get; set; }

    public ResolvedServicePartition ResolvedServicePartition { get; set; }
}
```
```java
public class MyCommunicationClient implements CommunicationClient {

    private ResolvedServicePartition resolvedServicePartition;
    private String listenerName;
    private ResolvedServiceEndpoint endPoint;

    /*
     * Getters and Setters
     */
}
```

Фабрика клиента в основном отвечает за создание клиентов связи. Для клиентов, которые не поддерживают постоянное подключение, например из клиента HTTP, фабрике требуется только создать и вернуть клиент. Другие протоколы, поддерживающие постоянное подключение, например некоторые двоичные протоколы, также должны проверяться `ValidateClient(string endpoint, MyCommunicationClient client)` с помощью фабрики, чтобы определить, нужно ли повторно создать соединение.  

```csharp
public class MyCommunicationClientFactory : CommunicationClientFactoryBase<MyCommunicationClient>
{
    protected override void AbortClient(MyCommunicationClient client)
    {
    }

    protected override Task<MyCommunicationClient> CreateClientAsync(string endpoint, CancellationToken cancellationToken)
    {
    }

    protected override bool ValidateClient(MyCommunicationClient clientChannel)
    {
    }

    protected override bool ValidateClient(string endpoint, MyCommunicationClient client)
    {
    }
}
```
```java
public class MyCommunicationClientFactory extends CommunicationClientFactoryBase<MyCommunicationClient> {

    @Override
    protected boolean validateClient(MyCommunicationClient clientChannel) {
    }

    @Override
    protected boolean validateClient(String endpoint, MyCommunicationClient client) {
    }

    @Override
    protected CompletableFuture<MyCommunicationClient> createClientAsync(String endpoint) {
    }

    @Override
    protected void abortClient(MyCommunicationClient client) {
    }
}
```

Наконец, обработчик исключений определяет, какое действие следует предпринять при возникновении исключения. Исключения делятся на **повторяемые** и **неповторяемые**.

* **Неповторяемые** исключения просто возвращаются вызывающей стороне.
* **Повторяемые** исключения далее классифицируются как **временные** и **невременные**.
  * **Временные** исключения могут быть повторены без повторного разрешения адреса конечной точки службы. К ним относятся временные проблемы с сетью или сообщения об ошибках службы, отличные от тех, которые указывают, что адрес конечной точки службы не существует.
  * **Невременные** исключения требуют повторного разрешения адреса конечной точки службы. К ним относятся исключения, указывающие, что не удалось подключиться к конечной точке службы, потому что служба была перемещена на другой узел.

`TryHandleException` принимает решение о данном исключении. Если **неизвестно**, какое решение следует принять об исключении, то возвращается значение **false**. Если решение **известно**, то задается соответствующий результат и возвращается значение **true**.

```csharp
class MyExceptionHandler : IExceptionHandler
{
    public bool TryHandleException(ExceptionInformation exceptionInformation, OperationRetrySettings retrySettings, out ExceptionHandlingResult result)
    {
        // if exceptionInformation.Exception is known and is transient (can be retried without re-resolving)
        result = new ExceptionHandlingRetryResult(exceptionInformation.Exception, true, retrySettings, retrySettings.DefaultMaxRetryCount);
        return true;


        // if exceptionInformation.Exception is known and is not transient (indicates a new service endpoint address must be resolved)
        result = new ExceptionHandlingRetryResult(exceptionInformation.Exception, false, retrySettings, retrySettings.DefaultMaxRetryCount);
        return true;

        // if exceptionInformation.Exception is unknown (let the next IExceptionHandler attempt to handle it)
        result = null;
        return false;
    }
}
```
```java
public class MyExceptionHandler implements ExceptionHandler {

    @Override
    public ExceptionHandlingResult handleException(ExceptionInformation exceptionInformation, OperationRetrySettings retrySettings) {

        /* if exceptionInformation.getException() is known and is transient (can be retried without re-resolving)
         */
        result = new ExceptionHandlingRetryResult(exceptionInformation.getException(), true, retrySettings, retrySettings.getDefaultMaxRetryCount());
        return true;


        /* if exceptionInformation.getException() is known and is not transient (indicates a new service endpoint address must be resolved)
         */
        result = new ExceptionHandlingRetryResult(exceptionInformation.getException(), false, retrySettings, retrySettings.getDefaultMaxRetryCount());
        return true;

        /* if exceptionInformation.getException() is unknown (let the next ExceptionHandler attempt to handle it)
         */
        result = null;
        return false;

    }
}
```
### <a name="putting-it-all-together"></a>Сборка
`ICommunicationClient(C#) / CommunicationClient(Java)`, `ICommunicationClientFactory(C#) / CommunicationClientFactory(Java)` и `IExceptionHandler(C#) / ExceptionHandler(Java)` основаны на протоколе связи, а `ServicePartitionClient(C#) / FabricServicePartitionClient(Java)` создает совместную оболочку и предоставляет цикл обработки ошибок и разрешения адреса секции службы вокруг этих компонентов.

```csharp
private MyCommunicationClientFactory myCommunicationClientFactory;
private Uri myServiceUri;

var myServicePartitionClient = new ServicePartitionClient<MyCommunicationClient>(
    this.myCommunicationClientFactory,
    this.myServiceUri,
    myPartitionKey);

var result = await myServicePartitionClient.InvokeWithRetryAsync(async (client) =>
   {
      // Communicate with the service using the client.
   },
   CancellationToken.None);

```
```java
private MyCommunicationClientFactory myCommunicationClientFactory;
private URI myServiceUri;

FabricServicePartitionClient myServicePartitionClient = new FabricServicePartitionClient<MyCommunicationClient>(
    this.myCommunicationClientFactory,
    this.myServiceUri,
    myPartitionKey);

CompletableFuture<?> result = myServicePartitionClient.invokeWithRetryAsync(client -> {
      /* Communicate with the service using the client.
       */
   });

```

## <a name="next-steps"></a>Дальнейшие действия
* [ASP.NET Core в Service Fabric Reliable Services](service-fabric-reliable-services-communication-aspnetcore.md)
* [Удаленное взаимодействие службы с Reliable Services](service-fabric-reliable-services-communication-remoting.md)
* [Коммуникационный стек WCF для надежных служб](service-fabric-reliable-services-communication-wcf.md)
