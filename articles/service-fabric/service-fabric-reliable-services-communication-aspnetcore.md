---
title: Взаимодействие со службой с ASP.NET Core
description: Узнайте, как использовать ASP.NET Core в приложениях Azure Service Fabric с отслеживанием состояния и без отслеживания состояния Reliable Services.
ms.topic: conceptual
ms.date: 10/12/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: a125c6a1972b51f518175a4c69248119f71ada7c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98791600"
---
# <a name="aspnet-core-in-azure-service-fabric-reliable-services"></a>ASP.NET Core в Azure Service Fabric Reliable Services

ASP.NET Core является платформой с открытым исходным кодом и кросс-платформенной платформой. Эта платформа предназначена для создания облачных приложений, подключенных к Интернету, таких как веб-приложения, приложения IoT и серверные компоненты для мобильных устройств.

Эта статья содержит подробное руководство по размещению ASP.NET Core служб в Service Fabric Reliable Services с помощью **Microsoft. ServiceFabric. AspNetCore.** набор пакетов NuGet.

Вводное руководство по ASP.NET Core в Service Fabric и инструкции по настройке среды разработки см. в разделе [учебник. Создание и развертывание приложения с помощью ASP.NET Core интерфейсной службы веб-API и серверной службы с отслеживанием состояния](service-fabric-tutorial-create-dotnet-app.md).

В оставшейся части этой статьи предполагается, что вы уже знакомы с ASP.NET Core. Если нет, прочтите [ASP.NET Core основ](/aspnet/core/fundamentals/index).

## <a name="aspnet-core-in-the-service-fabric-environment"></a>ASP.NET Core в среде Service Fabric

Приложения ASP.NET Core и Service Fabric могут работать на .NET Core или в полной платформа .NET Framework. В Service Fabric можно использовать ASP.NET Core двумя разными способами.
 - **Разместить в виде гостевого исполняемого файла**. Таким образом, в основном используется для запуска существующих приложений ASP.NET Core на Service Fabric без изменения кода.
 - **Запустите внутри надежной службы**. Таким образом обеспечивается лучшая интеграция со средой выполнения Service Fabric и обеспечивается ASP.NET Core служб с отслеживанием состояния.

В оставшейся части этой статьи объясняется, как использовать ASP.NET Core внутри надежной службы с помощью компонентов интеграции ASP.NET Core, поставляемых с пакетом SDK для Service Fabric.

## <a name="service-fabric-service-hosting"></a>Размещение службы Service Fabric

В Service Fabric один или несколько экземпляров и (или) реплик службы выполняются в *процессе узла службы*: исполняемый файл, в котором выполняется код службы. Вы, как автор службы, владеете ведущим процессом службы, и Service Fabric активирует и отслеживает его.

Обычно ASP.NET (до MVC 5) тесно связан с IIS через System.Web.dll. ASP.NET Core разделяет веб-сервер и веб-приложение. Такое разделение позволяет перенести веб-приложения между разными веб-серверами. Это также позволяет размещать веб-серверы в *автономном* виде. Это означает, что можно запустить веб-сервер в собственном процессе, в отличие от процесса, который принадлежит выделенному программному обеспечению веб-сервера, такому как IIS.

Чтобы объединить Service Fabricную службу и ASP.NET в качестве гостевого исполняемого файла или в надежной службе, необходимо иметь возможность запускать ASP.NET внутри хост-процесса службы. Автономное размещение ASP.NET Core позволяет это сделать.

## <a name="hosting-aspnet-core-in-a-reliable-service"></a>Размещение ASP.NET Core в надежной службе
Как правило, автономно размещаемые приложения ASP.NET Core создают WebHost в точке входа приложения, например метод `static void Main()` в `Program.cs`. В этом случае жизненный цикл узла привязывается к жизненному циклу процесса.

![Размещение ASP.NET Core в процессе][0]

Но точка входа приложения не является верным местом для создания узла в надежной службе. Это обусловлено тем, что точка входа приложения используется только для регистрации типа службы в среде выполнения Service Fabric, чтобы она могла создавать экземпляры этого типа службы. Служба WebHost должна быть создана в самой надежной службе. В рамках процесса узла службы экземпляры службы и (или) реплики могут проходить через несколько жизненных циклов. 

Экземпляр Reliable Service представлен с помощью класса службы, производного от `StatelessService` или `StatefulService`. Стек связи для службы содержится в реализации `ICommunicationListener` класса службы. `Microsoft.ServiceFabric.AspNetCore.*`Пакеты NuGet содержат реализации `ICommunicationListener` , которые запускают и управляют ASP.NET Coreм WebHost для Kestrel или HTTP.sys в надежной службе.

![Схема для размещения ASP.NET Core в надежной службе][1]

## <a name="aspnet-core-icommunicationlisteners"></a>Объекты ICommunicationListener в ASP.NET Core
`ICommunicationListener`Реализации Kestrel и HTTP.sys в `Microsoft.ServiceFabric.AspNetCore.*` пакетах NuGet имеют похожие шаблоны использования. Но они выполняют несколько различных действий, характерных для каждого веб-сервера. 

Оба прослушивателя связи предоставляют конструктор, который принимает следующие аргументы:
 - **`ServiceContext serviceContext`**— Это `ServiceContext` объект, содержащий сведения о работающей службе.
 - **`string endpointName`**: Это имя `Endpoint` конфигурации в ServiceManifest.xml. В основном это место, где два прослушивателя связи различаются. Для  HTTP.sys требуется `Endpoint` Конфигурация, а Kestrel — нет.
 - **`Func<string, AspNetCoreCommunicationListener, IWebHost> build`**: Это лямбда-выражение, которое реализуется, в котором вы создаете и возвращаете `IWebHost` . Он позволяет настроить `IWebHost` способ, который обычно будет в ASP.NET Core приложении. Лямбда-выражение предоставляет URL-адрес, который создается автоматически в зависимости от используемых параметров интеграции Service Fabric и `Endpoint` предоставленной конфигурации. Затем этот URL-адрес можно изменить или использовать для запуска веб-сервера.

## <a name="service-fabric-integration-middleware"></a>ПО промежуточного уровня для интеграции Service Fabric
`Microsoft.ServiceFabric.AspNetCore`Пакет NuGet включает `UseServiceFabricIntegration` метод расширения для `IWebHostBuilder` , который добавляет по промежуточного слоя с учетом Service Fabric. Это по промежуточного слоя настраивает Kestrel или HTTP.sys `ICommunicationListener` для регистрации уникального URL-адреса службы с Service Fabric служба именования. Затем он проверяет клиентские запросы, чтобы обеспечить подключение клиентов к правильной службе. 

Этот шаг необходим для предотвращения ошибочного подключения клиентов к неверной службе. Это связано с тем, что в среде общего размещения, такой как Service Fabric, несколько веб-приложений могут работать на одном физическом компьютере или виртуальной машине, но не используют уникальные имена узлов. Данный сценарий описан более подробно в следующем разделе.

### <a name="a-case-of-mistaken-identity"></a>Причина ошибочный идентификации
Реплики службы, независимо от протокола, прослушивают уникальное сочетание "IP-адрес:порт". Когда реплика службы начинает прослушивать конечную точку IP: Port, она сообщает этому адресу конечной точки Service Fabric Служба именования. Они могут обнаружить клиенты или другие службы. Если службы используют динамически назначенные порты приложений, реплика службы может по-своему использовать одну и ту же конечную точку IP: Port другой службы, которая ранее находилась на том же физическом компьютере или виртуальной машине. Это может привести к ошибочному подключению клиента к неверной службе, Этот сценарий может быть получен, если происходит следующая последовательность событий:

 1. Служба А прослушивает 10.0.0.1:30000 по протоколу HTTP. 
 2. Клиент разрешает службу а и получает адрес 10.0.0.1:30000.
 3. Служба А перемещается на другой узел.
 4. Служба Б помещается в 10.0.0.1 и случайно использует тот же порт 30000.
 5. Клиент пытается подключиться к службе А с помощью кэшированного адреса 10.0.0.1:30000.
 6. Теперь клиент успешно подключен к службе б, не зная, что он подключен к неверной службе.

Это может периодически вызывать ошибки, которые трудно диагностировать.

### <a name="using-unique-service-urls"></a>Использование уникальных URL-адресов службы
Чтобы предотвратить эти ошибки, службы могут опубликовать конечную точку в Служба именования с уникальным идентификатором, а затем проверить этот уникальный идентификатор во время клиентских запросов. Это совместное действие между службами в доверенной среде дружественного клиента. Она не обеспечивает безопасную проверку подлинности службы в среде враждебного клиента.

В доверенной среде по промежуточного слоя, добавленного `UseServiceFabricIntegration` методом, автоматически добавляет уникальный идентификатор к адресу, отправленному в Служба именования. Он проверяет этот идентификатор в каждом запросе. Если идентификатор не совпадает, по промежуточного слоя немедленно возвращает ответ HTTP 410.

Службы, использующие динамически назначаемый порт, должны использовать это ПО промежуточного слоя.

Службы, использующие фиксированный уникальный порт, не имеют этой проблемы в среде с совместным использованием. Постоянный уникальный порт обычно используется для внешних служб, которым необходим известный порт для подключения клиентских приложений. Например, большинство веб-приложений, подключенных к Интернету, будут использовать порт 80 или 443 для подключений к веб-браузеру. В этом случае не следует включать уникальный идентификатор.

На схеме ниже показан поток запроса с включенным ПО промежуточного слоя.

![Интеграция Service Fabric ASP.NET Core][2]

Реализации Kestrel и HTTP.sys `ICommunicationListener` используют этот механизм точно таким же образом. Хотя HTTP.sys может внутренне различать запросы на основе уникальных путей URL-адресов с помощью базовой функции совместного использования портов **HTTP.sys** , эта функция *не* используется `ICommunicationListener` реализацией HTTP.sys. Это происходит из-за того, что в сценарии, описанном выше, приведены коды состояния ошибок HTTP 503 и HTTP 404. Это, в свою очередь, затрудняет для клиентов определение цели ошибки, так как HTTP 503 и HTTP 404 часто используются для указания других ошибок. 

Таким способом, реализации Kestrel и HTTP.sys `ICommunicationListener` стандартизируют по промежуточного слоя, предоставляемого `UseServiceFabricIntegration` методом расширения. Таким образом, клиентам необходимо только выполнить действие повторного разрешения конечной точки службы для ответов HTTP 410.

## <a name="httpsys-in-reliable-services"></a>HTTP.sys в Reliable Services
Вы можете использовать HTTP.sys в Reliable Services, импортировав пакет NuGet **Microsoft. ServiceFabric. AspNetCore. HttpSys** . Этот пакет содержит `HttpSysCommunicationListener` реализацию `ICommunicationListener` . `HttpSysCommunicationListener` позволяет создать ASP.NET Core веб-узел в надежной службе с помощью HTTP.sys в качестве Web Server.

HTTP.sys построена на основе [API HTTP-сервера Windows](/windows/win32/http/http-api-start-page). Этот API-интерфейс использует драйвер ядра **HTTP.sys** для обработки HTTP-запросов и направления их в процессы, выполняющие приложения. Это позволяет нескольким процессам на одном физическом компьютере или виртуальной машине размещать веб-приложения на одном и том же порту с неоднозначностью по уникальному URL-адресу или имени узла. Эти функции в Service Fabric полезны для размещения нескольких веб-сайтов в одном кластере.

>[!NOTE]
>HTTP.sys реализация работает только на платформе Windows.

На следующей схеме показано, как HTTP.sys использует драйвер ядра **HTTP.sys** в Windows для совместного использования портов.

![Схема HTTP.sys][3]

### <a name="httpsys-in-a-stateless-service"></a>HTTP.sys в службе без отслеживания состояния
Для использования `HttpSys` в службе без отслеживания состояния переопределите метод `CreateServiceInstanceListeners` и верните экземпляр `HttpSysCommunicationListener`:

```csharp
protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
{
    return new ServiceInstanceListener[]
    {
        new ServiceInstanceListener(serviceContext =>
            new HttpSysCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) =>
                new WebHostBuilder()
                    .UseHttpSys()
                    .ConfigureServices(
                        services => services
                            .AddSingleton<StatelessServiceContext>(serviceContext))
                    .UseContentRoot(Directory.GetCurrentDirectory())
                    .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.None)
                    .UseStartup<Startup>()
                    .UseUrls(url)
                    .Build()))
    };
}
```

### <a name="httpsys-in-a-stateful-service"></a>HTTP.sys в службе с отслеживанием состояния

`HttpSysCommunicationListener` в настоящее время не предназначен для использования в службах с отслеживанием состояния из-за сложностей с базовой функцией совместного использования портов **HTTP.sys** . Дополнительные сведения см. в следующем разделе динамическое выделение портов с помощью HTTP.sys. Для служб с отслеживанием состояния Kestrel — предлагаемый веб-сервер.

### <a name="endpoint-configuration"></a>Конфигурация конечной точки

`Endpoint`Для веб-серверов, использующих API HTTP-сервера Windows, требуется настройка, в том числе HTTP.sys. Веб-серверы, использующие API сервера HTTP Windows, должны сначала зарезервировать свой URL-адрес с помощью HTTP.sys (обычно это выполняется с помощью средства [netsh](/windows/win32/http/netsh-commands-for-http) ). 

Для этого действия требуются повышенные привилегии, которые у служб нет по умолчанию. Параметры "http" или "HTTPS" для `Protocol` свойства `Endpoint` конфигурации в ServiceManifest.xml используются специально, чтобы указать Service Fabric времени выполнения зарегистрировать URL-адрес с HTTP.sys от вашего имени. Для этого используется строгий префикс URL-адреса с [*подстановочными знаками*](/windows/win32/http/urlprefix-strings) .

Например, чтобы зарезервировать `http://+:80` для службы, используйте следующую конфигурацию в ServiceManifest.xml.

```xml
<ServiceManifest ... >
    ...
    <Resources>
        <Endpoints>
            <Endpoint Name="ServiceEndpoint" Protocol="http" Port="80" />
        </Endpoints>
    </Resources>

</ServiceManifest>
```

Также необходимо передать имя конечной точки в конструктор `HttpSysCommunicationListener`.

```csharp
 new HttpSysCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) =>
 {
     return new WebHostBuilder()
         .UseHttpSys()
         .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.None)
         .UseUrls(url)
         .Build();
 })
```

#### <a name="use-httpsys-with-a-static-port"></a>Использование HTTP.sys со статическим портом
Чтобы использовать статический порт с HTTP.sys, укажите номер порта в `Endpoint` конфигурации:

```xml
  <Resources>
    <Endpoints>
      <Endpoint Protocol="http" Name="ServiceEndpoint" Port="80" />
    </Endpoints>
  </Resources>
```

#### <a name="use-httpsys-with-a-dynamic-port"></a>Использование HTTP.sys с динамическим портом
Чтобы использовать динамически назначаемый порт с HTTP.sys, опустите `Port` свойство в `Endpoint` конфигурации:

```xml
  <Resources>
    <Endpoints>
      <Endpoint Protocol="http" Name="ServiceEndpoint" />
    </Endpoints>
  </Resources>
```

Динамический порт, выделенный `Endpoint` конфигурацией, предоставляет только один порт *на хост-процесс*. Текущая модель размещения Service Fabric позволяет размещать несколько экземпляров службы и (или) реплик в одном процессе. Это означает, что каждый из них будет совместно использовать один и тот же порт при выделении через `Endpoint` конфигурацию. Несколько экземпляров **HTTP.sys** могут совместно использовать порт с помощью базовой функции совместного использования портов **HTTP.sys** . Но это не поддерживается `HttpSysCommunicationListener` из-за сложности, которую он создает для клиентских запросов. Для динамического использования портов Kestrel — предлагаемый веб-сервер.

## <a name="kestrel-in-reliable-services"></a>Kestrel в Reliable Services
Вы можете использовать Kestrel в Reliable Services, импортировав пакет NuGet **Microsoft. ServiceFabric. AspNetCore. Kestrel** . Этот пакет содержит `KestrelCommunicationListener` реализацию `ICommunicationListener` . `KestrelCommunicationListener` позволяет создать ASP.NET Core веб-узел в надежной службе с помощью Kestrel в качестве сервера.

Kestrel — это кроссплатформенный веб-сервер для ASP.NET Core. В отличие от HTTP.sys Kestrel не использует централизованный диспетчер конечных точек. Кроме того, в отличие от HTTP.sys Kestrel не поддерживает совместное использование портов между несколькими процессами. Каждый экземпляр Kestrel должен использовать уникальный порт. Дополнительные сведения о Kestrel см. в разделе [Подробности реализации](/aspnet/core/fundamentals/servers/kestrel).

![Схема Kestrel][4]

### <a name="kestrel-in-a-stateless-service"></a>Kestrel в службе без отслеживания состояния
Для использования `Kestrel` в службе без отслеживания состояния переопределите метод `CreateServiceInstanceListeners` и верните экземпляр `KestrelCommunicationListener`:

```csharp
protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
{
    return new ServiceInstanceListener[]
    {
        new ServiceInstanceListener(serviceContext =>
            new KestrelCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) =>
                new WebHostBuilder()
                    .UseKestrel()
                    .ConfigureServices(
                        services => services
                            .AddSingleton<StatelessServiceContext>(serviceContext))
                    .UseContentRoot(Directory.GetCurrentDirectory())
                    .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.UseUniqueServiceUrl)
                    .UseStartup<Startup>()
                    .UseUrls(url)
                    .Build();
            ))
    };
}
```

### <a name="kestrel-in-a-stateful-service"></a>Kestrel в службе с отслеживанием состояния
Для использования `Kestrel` в службе с отслеживанием состояния переопределите метод `CreateServiceReplicaListeners` и верните экземпляр `KestrelCommunicationListener`:

```csharp
protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
{
    return new ServiceReplicaListener[]
    {
        new ServiceReplicaListener(serviceContext =>
            new KestrelCommunicationListener(serviceContext, (url, listener) =>
                new WebHostBuilder()
                    .UseKestrel()
                    .ConfigureServices(
                         services => services
                             .AddSingleton<StatefulServiceContext>(serviceContext)
                             .AddSingleton<IReliableStateManager>(this.StateManager))
                    .UseContentRoot(Directory.GetCurrentDirectory())
                    .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.UseUniqueServiceUrl)
                    .UseStartup<Startup>()
                    .UseUrls(url)
                    .Build();
            ))
    };
}
```

В этом примере одноэлементный экземпляр `IReliableStateManager` предоставляется контейнеру внедрения зависимостей WebHost. Это не обязательно, но позволяет использовать `IReliableStateManager` и надежные коллекции в методах действий контроллера MVC.

Имя конфигурации `Endpoint`*не* предоставляется `KestrelCommunicationListener` в службе с отслеживанием состояния. Это объясняется более подробно в следующем разделе.

### <a name="configure-kestrel-to-use-https"></a>Настройка Kestrel для использования HTTPS
При включении HTTPS с Kestrel в службе необходимо задать несколько параметров прослушивания. Обновите, `ServiceInstanceListener` чтобы использовать конечную точку *ендпоинсттпс* и прослушивать конкретный порт (например, порт 443). При настройке веб-узла для использования веб-сервера Kestrel необходимо настроить Kestrel для прослушивания IPv6-адресов во всех сетевых интерфейсах: 

```csharp
new ServiceInstanceListener(
serviceContext =>
    new KestrelCommunicationListener(
        serviceContext,
        "EndpointHttps",
        (url, listener) =>
        {
            ServiceEventSource.Current.ServiceMessage(serviceContext, $"Starting Kestrel on {url}");

            return new WebHostBuilder()
                .UseKestrel(opt =>
                {
                    int port = serviceContext.CodePackageActivationContext.GetEndpoint("EndpointHttps").Port;
                    opt.Listen(IPAddress.IPv6Any, port, listenOptions =>
                    {
                        listenOptions.UseHttps(GetCertificateFromStore());
                        listenOptions.NoDelay = true;
                    });
                })
                .ConfigureAppConfiguration((builderContext, config) =>
                {
                    config.AddJsonFile("appsettings.json", optional: false, reloadOnChange: true);
                })

                .ConfigureServices(
                    services => services
                        .AddSingleton<HttpClient>(new HttpClient())
                        .AddSingleton<FabricClient>(new FabricClient())
                        .AddSingleton<StatelessServiceContext>(serviceContext))
                .UseContentRoot(Directory.GetCurrentDirectory())
                .UseStartup<Startup>()
                .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.None)
                .UseUrls(url)
                .Build();
        }))
```

Полный пример в руководстве см. в разделе [Настройка Kestrel для использования протокола HTTPS](service-fabric-tutorial-dotnet-app-enable-https-endpoint.md#configure-kestrel-to-use-https).


### <a name="endpoint-configuration"></a>Конфигурация конечной точки
`Endpoint`Для использования Kestrel не требуется конфигурация. 

Kestrel — это простой автономный веб-сервер. В отличие от HTTP.sys (или HttpListener), не требуется `Endpoint` Конфигурация в ServiceManifest.xml, так как она не требует регистрации URL-адреса перед запуском. 

#### <a name="use-kestrel-with-a-static-port"></a>Использование Kestrel и статического порта
Статический порт можно настроить в `Endpoint` конфигурации ServiceManifest.xml для использования с Kestrel. Хотя это не является обязательным, оно предлагает два возможных преимущества:
 - Если порт не попадает в диапазон портов приложения, он открывается через брандмауэр ОС, Service Fabric.
 - URL-адрес, предоставленный с помощью `KestrelCommunicationListener`, будет использовать этот порт.

```xml
  <Resources>
    <Endpoints>
      <Endpoint Protocol="http" Name="ServiceEndpoint" Port="80" />
    </Endpoints>
  </Resources>
```

Если `Endpoint` настроена, ее имя необходимо передать в конструктор `KestrelCommunicationListener`. 

```csharp
new KestrelCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) => ...
```

Если ServiceManifest.xml не использует `Endpoint` конфигурацию, не указывайте имя в `KestrelCommunicationListener` конструкторе. В этом случае будет использоваться динамический порт. Дополнительные сведения об этом см. в следующем разделе.

#### <a name="use-kestrel-with-a-dynamic-port"></a>Использование Kestrel и динамического порта
Kestrel не может использовать автоматическое назначение портов из `Endpoint` конфигурации в ServiceManifest.xml. Это связано с тем, что автоматическое назначение портов из `Endpoint` конфигурации назначает уникальный порт для каждого *хост-процесса*, и один ведущий процесс может содержать несколько экземпляров Kestrel. Это не работает с Kestrel, так как не поддерживает совместное использование портов. Поэтому каждый экземпляр Kestrel должен быть открыт на уникальном порту.

Чтобы использовать динамическое назначение портов с помощью Kestrel, `Endpoint` полностью пропустите конфигурацию в ServiceManifest.xml и не передавайте имя конечной точки `KestrelCommunicationListener` конструктору следующим образом:

```csharp
new KestrelCommunicationListener(serviceContext, (url, listener) => ...
```

В этой конфигурации `KestrelCommunicationListener` автоматически выберет неиспользуемый порт из диапазона портов приложения.

Для HTTPS должна быть настроена конечная точка с протоколом HTTPS без порта, указанного в ServiceManifest.xml, и передайте имя конечной точки в конструктор Кестрелкоммуникатионлистенер.


## <a name="service-fabric-configuration-provider"></a>Поставщик конфигурации Service Fabric
Конфигурация приложения в ASP.NET Core основана на парах "ключ-значение", установленной поставщиком конфигурации. Прочитайте [конфигурацию в ASP.NET Core](/aspnet/core/fundamentals/configuration/) , чтобы получить дополнительные сведения о поддержке общей конфигурации ASP.NET Core.

В этом разделе описано, как поставщик конфигурации Service Fabric интегрируется с конфигурацией ASP.NET Core путем импорта `Microsoft.ServiceFabric.AspNetCore.Configuration` пакета NuGet.

### <a name="addservicefabricconfiguration-startup-extensions"></a>Расширения запуска Аддсервицефабрикконфигуратион
После импорта `Microsoft.ServiceFabric.AspNetCore.Configuration` пакета NuGet необходимо зарегистрировать источник конфигурации Service Fabric с помощью API конфигурации ASP.NET Core. Для этого нужно проверить расширения **аддсервицефабрикконфигуратион** в `Microsoft.ServiceFabric.AspNetCore.Configuration` пространстве имен `IConfigurationBuilder` .

```csharp
using Microsoft.ServiceFabric.AspNetCore.Configuration;

public Startup(IHostingEnvironment env)
{
    var builder = new ConfigurationBuilder()
        .SetBasePath(env.ContentRootPath)
        .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
        .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
        .AddServiceFabricConfiguration() // Add Service Fabric configuration settings.
        .AddEnvironmentVariables();
    Configuration = builder.Build();
}

public IConfigurationRoot Configuration { get; }
```

Теперь ASP.NET Core служба может получить доступ к параметрам конфигурации Service Fabric, как и к любым другим параметрам приложения. Например, шаблон параметров можно использовать для загрузки параметров в строго типизированные объекты.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<MyOptions>(Configuration);  // Strongly typed configuration object.
    services.AddMvc();
}
```
### <a name="default-key-mapping"></a>Сопоставление ключей по умолчанию
По умолчанию поставщик конфигурации Service Fabric содержит имя пакета, имя раздела и имя свойства. Вместе они формируют ключ конфигурации ASP.NET Core следующим образом:
```csharp
$"{this.PackageName}{ConfigurationPath.KeyDelimiter}{section.Name}{ConfigurationPath.KeyDelimiter}{property.Name}"
```

Например, если имеется пакет конфигурации `MyConfigPackage` со следующим содержимым, то значение конфигурации будет доступно в ASP.NET Core `IConfiguration` через *миконфигпаккаже: миконфигсектион: MyParameter*.
```xml
<?xml version="1.0" encoding="utf-8" ?>
<Settings xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">  
  <Section Name="MyConfigSection">
    <Parameter Name="MyParameter" Value="Value1" />
  </Section>  
</Settings>
```
### <a name="service-fabric-configuration-options"></a>Параметры конфигурации Service Fabric
Поставщик конфигурации Service Fabric также поддерживает `ServiceFabricConfigurationOptions` изменение поведения сопоставления ключей по умолчанию.

#### <a name="encrypted-settings"></a>Зашифрованные параметры
Service Fabric поддерживает зашифрованные параметры, как и поставщик конфигурации Service Fabric. Зашифрованные параметры не расшифровываются в ASP.NET Core по `IConfiguration` умолчанию. Вместо этого сохраняются зашифрованные значения. Но если вы хотите расшифровать значение, которое будет храниться в ASP.NET Core IConfiguration, можно установить для флага *декриптвалуе* значение false в `AddServiceFabricConfiguration` расширении следующим образом:

```csharp
public Startup()
{
    ICodePackageActivationContext activationContext = FabricRuntime.GetActivationContext();
    var builder = new ConfigurationBuilder()        
        .AddServiceFabricConfiguration(activationContext, (options) => options.DecryptValue = false); // set flag to decrypt the value
    Configuration = builder.Build();
}
```
#### <a name="multiple-configuration-packages"></a>Несколько пакетов конфигурации
Service Fabric поддерживает несколько пакетов конфигурации. По умолчанию имя пакета включается в ключ конфигурации. Но можно установить `IncludePackageName` для флага значение false, как показано ниже.
```csharp
public Startup()
{
    ICodePackageActivationContext activationContext = FabricRuntime.GetActivationContext();
    var builder = new ConfigurationBuilder()        
        // exclude package name from key.
        .AddServiceFabricConfiguration(activationContext, (options) => options.IncludePackageName = false); 
    Configuration = builder.Build();
}
```
#### <a name="custom-key-mapping-value-extraction-and-data-population"></a>Сопоставление пользовательских ключей, извлечение значений и заполнение данных
Поставщик конфигурации Service Fabric также поддерживает более сложные сценарии для настройки сопоставления ключей с `ExtractKeyFunc` и настраиваемого извлечения значений с помощью `ExtractValueFunc` . Можно даже изменить весь процесс заполнения данных из Service Fabric конфигурации на ASP.NET Core конфигурации с помощью `ConfigAction` .

В следующих примерах показано, как использовать `ConfigAction` для настройки заполнения данных.
```csharp
public Startup()
{
    ICodePackageActivationContext activationContext = FabricRuntime.GetActivationContext();
    
    this.valueCount = 0;
    this.sectionCount = 0;
    var builder = new ConfigurationBuilder();
    builder.AddServiceFabricConfiguration(activationContext, (options) =>
        {
            options.ConfigAction = (package, configData) =>
            {
                ILogger logger = new ConsoleLogger("Test", null, false);
                logger.LogInformation($"Config Update for package {package.Path} started");

                foreach (var section in package.Settings.Sections)
                {
                    this.sectionCount++;

                    foreach (var param in section.Parameters)
                    {
                        configData[options.ExtractKeyFunc(section, param)] = options.ExtractValueFunc(section, param);
                        this.valueCount++;
                    }
                }

                logger.LogInformation($"Config Update for package {package.Path} finished");
            };
        });
  Configuration = builder.Build();
}
```

### <a name="configuration-updates"></a>Обновления конфигурации
Поставщик конфигурации Service Fabric также поддерживает обновления конфигурации. Вы можете использовать ASP.NET Core `IOptionsMonitor` для получения уведомлений об изменениях, а затем использовать `IOptionsSnapshot` для повторной загрузки данных конфигурации. Дополнительные сведения см. в разделе [ASP.NET Core Options](/aspnet/core/fundamentals/configuration/options).

Эти параметры поддерживаются по умолчанию. Для включения обновлений конфигурации дальнейшее написание кода не требуется.

## <a name="scenarios-and-configurations"></a>Сценарии и конфигурации
В этом разделе представлено сочетание параметров веб-сервера, конфигурации портов, Service Fabric интеграции и прочих параметров, которые рекомендуется использовать для устранения следующих сценариев.
 - Доступные извне службы ASP.NET Core без отслеживания состояния
 - Только внутренние ASP.NET Core службы без отслеживания состояния
 - Службы с отслеживанием состояния только для внутреннего использования ASP.NET Core

**Внешней службой называется служба** , которая предоставляет конечную точку, которая вызывается извне кластера, обычно через подсистему балансировки нагрузки.

**Только внутренняя** служба, конечная точка которой вызывается только в пределах кластера.

> [!NOTE]
> Обычно конечные точки службы с отслеживанием состояния не должны быть доступны в Интернете. Кластеры за подсистемами балансировки нагрузки, не поддерживающие разрешение Service Fabric службы, такие как Azure Load Balancer, не смогут предоставлять службы с отслеживанием состояния. Это обусловлено тем, что балансировщик нагрузки не сможет найти и направить трафик в соответствующую реплику службы с отслеживанием состояния. 

### <a name="externally-exposed-aspnet-core-stateless-services"></a>Доступные извне службы ASP.NET Core без отслеживания состояния
Kestrel — предлагаемый веб-сервер для интерфейсных служб, которые предоставляют доступ к внешним конечным точкам HTTP с выходом в Интернет. В Windows HTTP.sys может предоставить возможность совместного использования портов, которая позволяет разместить несколько веб-служб на одном наборе узлов, используя один и тот же порт. В этом сценарии веб-службы отличаются по имени узла или пути, не полагаясь на интерфейсный прокси-сервер или шлюз для предоставления маршрутизации по протоколу HTTP.
 
При доступе к Интернету служба без отслеживания состояния должна использовать известную и устойчивую конечную точку, доступную через подсистему балансировки нагрузки. Этот URL-адрес будет предоставлен пользователям приложения. Используйте следующие конфигурации:

| Type | Рекомендация | Примечания |
| ---- | -------------- | ----- |
| Веб-сервер | Kestrel | Kestrel — предпочитаемый веб-сервер, так как он поддерживается в Windows и Linux. |
| Конфигурация порта | static | Хорошо известный статический порт необходимо настроить в конфигурации `Endpoints` файла ServiceManifest.xml, например 80 для HTTP и 443 для HTTPS. |
| ServiceFabricIntegrationOptions | Нет | Используйте `ServiceFabricIntegrationOptions.None` параметр при настройке по промежуточного слоя интеграции Service Fabric, чтобы служба не пыталась проверить входящие запросы на уникальный идентификатор. Внешние пользователи приложения не узнают уникальную идентификационную информацию, которую использует по промежуточного слоя. |
| Число экземпляров | -1 | В типичных случаях для параметра число экземпляров должно быть задано значение *-1*. Это делается для того, чтобы экземпляр был доступен на всех узлах, которые получают трафик от балансировщика нагрузки. |

Если несколько служб, предоставляемых извне, совместно используют один и тот же набор узлов, можно использовать HTTP.sys с уникальным, но стабильным путем URL-адреса. Это можно сделать, изменив URL-адрес, указанный при настройке IWebHost. Обратите внимание, что это относится только к HTTP.sys.

 ```csharp
 new HttpSysCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) =>
 {
     url += "/MyUniqueServicePath";
 
     return new WebHostBuilder()
         .UseHttpSys()
         ...
         .UseUrls(url)
         .Build();
 })
 ```

### <a name="internal-only-stateless-aspnet-core-service"></a>Служба ASP.NET Core без отслеживания состояния только для внутреннего использования
Службы без отслеживания состояния, которые вызываются только из кластера, должны использовать уникальные URL-адреса и динамически назначаемые порты для обеспечения взаимодействия между несколькими службами. Используйте следующие конфигурации:

| Type | Рекомендация | Примечания |
| ---- | -------------- | ----- |
| Веб-сервер | Kestrel | Несмотря на то, что можно использовать HTTP.sys для внутренних служб без отслеживания состояния, Kestrel является лучшим сервером, позволяющим нескольким экземплярам службы совместно использовать узел.  |
| Конфигурация порта | динамическое назначение | Несколько реплик службы с отслеживанием состояния могут совместно использовать хост-процесс или операционную систему узла и, таким же, потребуются уникальные порты. |
| ServiceFabricIntegrationOptions | UseUniqueServiceUrl | При динамическом назначении порта этот параметр предотвращает ошибочную идентификацию, описанную ранее. |
| InstanceCount | any | Для параметра количества экземпляров может быть присвоено любое значение, необходимое для работы службы. |

### <a name="internal-only-stateful-aspnet-core-service"></a>Служба ASP.NET Core с отслеживанием состояния только для внутреннего использования
Службы с отслеживанием состояния, которые вызываются только из кластера, должны использовать динамически назначаемые порты для обеспечения взаимодействия между несколькими службами. Используйте следующие конфигурации:

| Type | Рекомендация | Примечания |
| ---- | -------------- | ----- |
| Веб-сервер | Kestrel | `HttpSysCommunicationListener`Не предназначен для использования службами с отслеживанием состояния, в которых реплики совместно используют главный процесс. |
| Конфигурация порта | динамическое назначение | Несколько реплик службы с отслеживанием состояния могут совместно использовать хост-процесс или операционную систему узла и, таким же, потребуются уникальные порты. |
| ServiceFabricIntegrationOptions | UseUniqueServiceUrl | При динамическом назначении порта этот параметр предотвращает ошибочную идентификацию, описанную ранее. |

## <a name="next-steps"></a>Дальнейшие действия
[Отладка приложения Service Fabric с помощью Visual Studio](service-fabric-debugging-your-application.md)


<!--Image references-->
[0]:./media/service-fabric-reliable-services-communication-aspnetcore/webhost-standalone.png
[1]:./media/service-fabric-reliable-services-communication-aspnetcore/webhost-servicefabric.png
[2]:./media/service-fabric-reliable-services-communication-aspnetcore/integration.png
[3]:./media/service-fabric-reliable-services-communication-aspnetcore/httpsys.png
[4]:./media/service-fabric-reliable-services-communication-aspnetcore/kestrel.png
