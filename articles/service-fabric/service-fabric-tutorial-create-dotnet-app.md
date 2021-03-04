---
title: Создание приложения .NET в Service Fabric в Azure
description: В этом руководстве вы узнаете, как создать приложение с клиентской частью ASP.NET Core и серверной частью в виде надежной службы с отслеживанием состояния и развернуть приложение в кластер.
ms.topic: tutorial
ms.date: 07/10/2019
ms.custom: mvc, devx-track-js, devx-track-csharp
ms.openlocfilehash: 8fe9f1fcb85e58122290f89819aa721c8f0e632a
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102035332"
---
# <a name="tutorial-create-and-deploy-an-application-with-an-aspnet-core-web-api-front-end-service-and-a-stateful-back-end-service"></a>Руководство по Создание и развертывание приложения с интерфейсной службой веб-API ASP.NET Core и серверной службой с отслеживанием состояния

Это руководство представляет первую часть цикла.  Здесь описывается, как создать приложение Azure Service Fabric с интерфейсной службой веб-API ASP.NET Core и серверной службой с отслеживанием состояния для хранения данных. После завершения этого руководства вы получите приложение для голосования с клиентской частью в виде веб-приложения ASP.NET Core, которое сохраняет результаты голосования во внутренней службе с отслеживанием состояния в кластере. Для работы с этой серией руководств требуется компьютер разработчика Windows. Если вы не хотите вручную создавать приложение для голосования, вы можете [скачать исходный код](https://github.com/Azure-Samples/service-fabric-dotnet-quickstart/) для завершенного приложения и сразу перейти к [описанию примера приложения для голосования](#walkthrough_anchor).  При желании вы можете просмотреть [видео-инструкцию](https://channel9.msdn.com/Events/Connect/2017/E100) к этому руководству.

![AngularJS + интерфейсная служба API ASP.NET Core, подключенная к серверной службе с отслеживанием состояния в Service Fabric](./media/service-fabric-tutorial-create-dotnet-app/application-diagram.png)

В первой части цикла вы узнаете, как выполнять такие задачи:

> [!div class="checklist"]
> * Создание службы веб-API ASP.NET Core как надежной службы с отслеживанием состояния
> * Создание службы веб-приложения ASP.NET Core как службы без отслеживания состояния
> * Использование обратного прокси-сервера для связи со службой с отслеживанием состояния

Из этого цикла руководств вы узнаете, как выполнять следующие задачи:
> [!div class="checklist"]
> * Создание приложения .NET Service Fabric.
> * [Развертывание приложения в удаленном кластере](service-fabric-tutorial-deploy-app-to-party-cluster.md).
> * [Добавление конечной точки HTTPS к интерфейсной службе ASP.NET Core](service-fabric-tutorial-dotnet-app-enable-https-endpoint.md)
> * [Настройка CI/CD с помощью Azure Pipelines](service-fabric-tutorial-deploy-app-with-cicd-vsts.md)
> * [Настройка мониторинга и диагностики приложения](service-fabric-tutorial-monitoring-aspnet.md)

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы с этим руководством выполните следующие действия:
* Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Установите Visual Studio 2019](https://www.visualstudio.com/) (15.5 или более поздней версии), а также рабочие нагрузки **Разработка для Azure** и **ASP.NET и разработка веб-приложений**.
* [Установите пакет SDK для Service Fabric](service-fabric-get-started.md)

## <a name="create-an-aspnet-web-api-service-as-a-reliable-service"></a>Создание службы веб-API ASP.NET как надежной службы

Сначала создайте веб-интерфейс приложения для голосования с помощью ASP.NET Core. ASP.NET Core — это простое кросс-платформенное средство для разработки веб-страниц, позволяющее создавать современные пользовательские веб-интерфейсы и интерфейсы веб-API. Чтобы получить полное представление об интеграции ASP.NET Core с Service Fabric, мы настоятельно рекомендуем ознакомиться со статьей [ASP.NET Core в Service Fabric Reliable Services](service-fabric-reliable-services-communication-aspnetcore.md). А пока можете воспользоваться этим руководством, чтобы быстро приступить к работе. Дополнительные сведения об ASP.NET Core см. в [документации по ASP.NET Core](/aspnet/core/).

1. Запустите Visual Studio от имени **администратора**.

2. Выберите **Файл**->**Создать**->**Проект**, чтобы создать проект.

3. В диалоговом окне **Создание проекта** выберите **Облако > Приложение Service Fabric**.

4. Присвойте приложению имя **Voting** и нажмите кнопку **ОК**.

   ![Диалоговое окно "Новый проект" в Visual Studio](./media/service-fabric-tutorial-create-dotnet-app/new-project-dialog.png)

5. На странице **Новая служба Service Fabric** выберите **ASP.NET Core без отслеживания состояния**, задайте службе имя **VotingWeb**, а затем нажмите кнопку **ОК**.
   
   ![Выбор веб-службы ASP.NET в диалоговом окне создания службы](./media/service-fabric-tutorial-create-dotnet-app/new-project-dialog-2.png) 

6. На следующей странице представлен набор шаблонов проекта ASP.NET Core. Для работы с этим руководством выберите **Веб-приложение (модель-представление-контроллер)** , а затем нажмите кнопку **ОК**.
   
   ![Выбор типа проекта ASP.NET](./media/service-fabric-tutorial-create-dotnet-app/vs-new-aspnet-project-dialog.png)

   Visual Studio создает приложение и проект службы и отображает их в обозревателе решений.

   ![Обозреватель решений после создания приложения и службы веб-API ASP.NET Core]( ./media/service-fabric-tutorial-create-dotnet-app/solution-explorer-aspnetcore-service.png)

### <a name="update-the-sitejs-file"></a>Обновление файла site.js
Откройте файл **wwwroot/js/site.js**.  Замените его содержимое следующим кодом JavaScript, используемым в представлениях Home, а затем сохраните изменения.

```javascript
var app = angular.module('VotingApp', ['ui.bootstrap']);
app.run(function () { });

app.controller('VotingAppController', ['$rootScope', '$scope', '$http', '$timeout', function ($rootScope, $scope, $http, $timeout) {

    $scope.refresh = function () {
        $http.get('api/Votes?c=' + new Date().getTime())
            .then(function (data, status) {
                $scope.votes = data;
            }, function (data, status) {
                $scope.votes = undefined;
            });
    };

    $scope.remove = function (item) {
        $http.delete('api/Votes/' + item)
            .then(function (data, status) {
                $scope.refresh();
            })
    };

    $scope.add = function (item) {
        var fd = new FormData();
        fd.append('item', item);
        $http.put('api/Votes/' + item, fd, {
            transformRequest: angular.identity,
            headers: { 'Content-Type': undefined }
        })
            .then(function (data, status) {
                $scope.refresh();
                $scope.item = undefined;
            })
    };
}]);
```

### <a name="update-the-indexcshtml-file"></a>Обновление файла Index.cshtml

Откройте файл **Views/Home/Index.cshtml** — представление, относящееся к контроллеру Home.  Замените его содержимое на приведенное ниже, а затем сохраните изменения.

```html
@{
    ViewData["Title"] = "Service Fabric Voting Sample";
}

<div ng-controller="VotingAppController" ng-init="refresh()">
    <div class="container-fluid">
        <div class="row">
            <div class="col-xs-8 col-xs-offset-2 text-center">
                <h2>Service Fabric Voting Sample</h2>
            </div>
        </div>

        <div class="row">
            <div class="col-xs-8 col-xs-offset-2">
                <form class="col-xs-12 center-block">
                    <div class="col-xs-6 form-group">
                        <input id="txtAdd" type="text" class="form-control" placeholder="Add voting option" ng-model="item"/>
                    </div>
                    <button id="btnAdd" class="btn btn-default" ng-click="add(item)">
                        <span class="glyphicon glyphicon-plus" aria-hidden="true"></span>
                        Add
                    </button>
                </form>
            </div>
        </div>

        <hr/>

        <div class="row">
            <div class="col-xs-8 col-xs-offset-2">
                <div class="row">
                    <div class="col-xs-4">
                        Click to vote
                    </div>
                </div>
                <div class="row top-buffer" ng-repeat="vote in votes.data">
                    <div class="col-xs-8">
                        <button class="btn btn-success text-left btn-block" ng-click="add(vote.key)">
                            <span class="pull-left">
                                {{vote.Key}}
                            </span>
                            <span class="badge pull-right">
                                {{vote.Value}} Votes
                            </span>
                        </button>
                    </div>
                    <div class="col-xs-4">
                        <button class="btn btn-danger pull-right btn-block" ng-click="remove(vote.key)">
                            <span class="glyphicon glyphicon-remove" aria-hidden="true"></span>
                            Remove
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```

### <a name="update-the-_layoutcshtml-file"></a>Обновление файла _Layout.cshtml

Откройте файл **Views/Shared/_Layout.cshtml** — стандартный макет для приложения ASP.NET.  Замените его содержимое на приведенное ниже, а затем сохраните изменения.


```html
<!DOCTYPE html>
<html ng-app="VotingApp" xmlns:ng="https://angularjs.org">
<head>
    <meta charset="utf-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>@ViewData["Title"]</title>

    <link href="~/lib/bootstrap/dist/css/bootstrap.css" rel="stylesheet"/>
    <link href="~/css/site.css" rel="stylesheet"/>

</head>
<body>
<div class="container body-content">
    @RenderBody()
</div>

<script src="~/lib/jquery/dist/jquery.js"></script>
<script src="~/lib/bootstrap/dist/js/bootstrap.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.7.2/angular.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular-ui-bootstrap/2.5.0/ui-bootstrap-tpls.js"></script>
<script src="~/js/site.js"></script>

@RenderSection("Scripts", required: false)
</body>
</html>
```

### <a name="update-the-votingwebcs-file"></a>Обновление файла VotingWeb.cs

Откройте файл *VotingWeb.cs*, который создает WebHost Core ASP.NET в пределах службы без отслеживания состояния с помощью веб-сервера WebListener.

Добавьте директиву `using System.Net.Http;` в начало файла.

Замените функцию `CreateServiceInstanceListeners()` следующим кодом, а затем сохраните изменения.

```csharp
protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
{
    return new ServiceInstanceListener[]
    {
        new ServiceInstanceListener(
            serviceContext =>
                new KestrelCommunicationListener(
                    serviceContext,
                    "ServiceEndpoint",
                    (url, listener) =>
                    {
                        ServiceEventSource.Current.ServiceMessage(serviceContext, $"Starting Kestrel on {url}");

                        return new WebHostBuilder()
                            .UseKestrel()
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
    };
}
```

Также добавьте следующий метод `GetVotingDataServiceName` под `CreateServiceInstanceListeners()`, а затем сохраните изменения. При опросе `GetVotingDataServiceName` возвращает имя службы.

```csharp
internal static Uri GetVotingDataServiceName(ServiceContext context)
{
    return new Uri($"{context.CodePackageActivationContext.ApplicationName}/VotingData");
}
```

### <a name="add-the-votescontrollercs-file"></a>Добавление файла VotesController.cs

Добавьте контроллер, определяющий действия голосования. Правой кнопкой щелкните папку **Контроллеры**, а затем выберите **Добавить -> Новый элемент -> Visual C# -> ASP.NET Core -> Класс**. Присвойте файлу имя **VotesController.cs**, а затем нажмите кнопку **Добавить**.  

Замените содержимое файла *VotesController.cs* следующим содержимым, а затем сохраните изменения.  Затем на этапе [Обновление файла VotesController.cs](#updatevotecontroller_anchor) этот файл будет изменен для чтения и записи данных голосования, полученных из серверной службы.  А пока контроллер возвращает статические строковые данные в представление.

```csharp
namespace VotingWeb.Controllers
{
    using System;
    using System.Collections.Generic;
    using System.Fabric;
    using System.Fabric.Query;
    using System.Linq;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Text;
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Mvc;
    using Newtonsoft.Json;

    [Produces("application/json")]
    [Route("api/Votes")]
    public class VotesController : Controller
    {
        private readonly HttpClient httpClient;

        public VotesController(HttpClient httpClient)
        {
            this.httpClient = httpClient;
        }

        // GET: api/Votes
        [HttpGet]
        public async Task<IActionResult> Get()
        {
            List<KeyValuePair<string, int>> votes= new List<KeyValuePair<string, int>>();
            votes.Add(new KeyValuePair<string, int>("Pizza", 3));
            votes.Add(new KeyValuePair<string, int>("Ice cream", 4));

            return Json(votes);
        }
     }
}
```

### <a name="configure-the-listening-port"></a>Настройка порта прослушивания

Когда создается интерфейсная служба VotingWeb, Visual Studio случайным образом выбирает порт для ее прослушивания.  Так как служба VotingWeb выступает в роли клиентской части этого приложения и принимает внешний трафик, мы привяжем эту службу к фиксированному и хорошо известному порту.  [Манифест служб](service-fabric-application-and-service-manifests.md) объявляет конечные точки службы.

В обозревателе решений откройте файл *VotingWeb/PackageRoot/ServiceManifest.xml*.  В разделе **Ресурсы** найдите элемент **Конечная точка** и задайте для **порта** значение **8080**. Для локального развертывания и запуска приложения порт прослушивания приложения должен быть открыт и доступен на компьютере.

```xml
<Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Protocol="http" Name="ServiceEndpoint" Type="Input" Port="8080" />
    </Endpoints>
  </Resources>
```

Обновите значение свойства "URL-адрес приложения" в проекте Voting, чтобы в веб-браузере открывался правильный порт при отладке с помощью клавиши F5.  В обозревателе решений выберите проект **Voting** и измените значение свойства **URL-адрес приложения** на **8080**.

### <a name="deploy-and-run-the-voting-application-locally"></a>Локальное развертывание и запуск приложения Voting
Теперь можно запустить приложение Voting для отладки. В Visual Studio нажмите клавишу **F5**, чтобы развернуть приложение в локальном кластере Service Fabric в режиме отладки. Если среда Visual Studio не была открыта ранее с правами **администратора**, приложение завершается неудачно.

> [!NOTE]
> Во время первого запуска и развертывания приложения локально Visual Studio создает локальный кластер Service Fabric для отладки.  Создание кластера может занять некоторое время. Процесс создания кластера можно наблюдать в окне вывода Visual Studio.

После развертывания приложения Voting в локальном кластере Service Fabric веб-приложение автоматически откроется на вкладке браузера и будет иметь следующий вид:

![Интерфейсная часть ASP.NET Core](./media/service-fabric-tutorial-create-dotnet-app/debug-front-end.png)

Чтобы остановить отладку приложения, вернитесь в Visual Studio и нажмите **SHIFT+F5**.

## <a name="add-a-stateful-back-end-service-to-your-application"></a>Добавление в приложение серверной службы с отслеживанием состояния

Теперь, когда в приложении есть служба веб-API ASP.NET, добавьте надежную службу с отслеживанием состояния, чтобы сохранить в приложении некоторые данные.

Service Fabric позволяет согласованно и надежно хранить данные прямо в службе с помощью надежных коллекций. Надежные коллекции — это набор классов коллекций с высокой доступностью и надежностью, работа с которыми не вызовет сложностей у пользователей коллекций C#.

В этом руководстве вы создадите службу, которая хранит значение счетчика в надежной коллекции.

1. В обозревателе решений в проекте приложения Voting щелкните правой кнопкой мыши **Службы** и последовательно выберите **Добавить > Новая служба Service Fabric**.
    
2. В диалоговом окне **Новая служба Service Fabric** выберите **Служба с отслеживанием состояния ASP.NET Core**, назовите службу **VotingData** и нажмите кнопку **ОК**.

    После создания проекта службы в вашем приложении будут две службы. По мере разработки приложения можно добавить дополнительные службы точно таким же образом. Каждая из этих служб может быть отдельно обновлена, в том числе до определенной версии.

3. На следующей странице представлен набор шаблонов проекта ASP.NET Core. Для работы с этим руководством выберите **API**.

    Visual Studio создаст проект службы VotingData и отобразит его в обозревателе решений.

    ![Обозреватель решений](./media/service-fabric-tutorial-create-dotnet-app/solution-explorer-aspnetcore-webapi-service.png)

### <a name="add-the-votedatacontrollercs-file"></a>Добавление файла VoteDataController.cs

В проекте **VotingData** щелкните правой кнопкой мыши папку **Контроллеры**, а затем выберите **Добавить -> Новый элемент -> Класс**. Назовите файл **VoteDataController.cs** и нажмите кнопку **Добавить**. Замените содержимое файла следующим образом, а затем сохраните изменения.

```csharp
namespace VotingData.Controllers
{
    using System.Collections.Generic;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.ServiceFabric.Data;
    using Microsoft.ServiceFabric.Data.Collections;

    [Route("api/[controller]")]
    public class VoteDataController : Controller
    {
        private readonly IReliableStateManager stateManager;

        public VoteDataController(IReliableStateManager stateManager)
        {
            this.stateManager = stateManager;
        }

        // GET api/VoteData
        [HttpGet]
        public async Task<IActionResult> Get()
        {
            CancellationToken ct = new CancellationToken();

            IReliableDictionary<string, int> votesDictionary = await this.stateManager.GetOrAddAsync<IReliableDictionary<string, int>>("counts");

            using (ITransaction tx = this.stateManager.CreateTransaction())
            {
                Microsoft.ServiceFabric.Data.IAsyncEnumerable<KeyValuePair<string, int>> list = await votesDictionary.CreateEnumerableAsync(tx);

                Microsoft.ServiceFabric.Data.IAsyncEnumerator<KeyValuePair<string, int>> enumerator = list.GetAsyncEnumerator();

                List<KeyValuePair<string, int>> result = new List<KeyValuePair<string, int>>();

                while (await enumerator.MoveNextAsync(ct))
                {
                    result.Add(enumerator.Current);
                }

                return this.Json(result);
            }
        }

        // PUT api/VoteData/name
        [HttpPut("{name}")]
        public async Task<IActionResult> Put(string name)
        {
            IReliableDictionary<string, int> votesDictionary = await this.stateManager.GetOrAddAsync<IReliableDictionary<string, int>>("counts");

            using (ITransaction tx = this.stateManager.CreateTransaction())
            {
                await votesDictionary.AddOrUpdateAsync(tx, name, 1, (key, oldvalue) => oldvalue + 1);
                await tx.CommitAsync();
            }

            return new OkResult();
        }

        // DELETE api/VoteData/name
        [HttpDelete("{name}")]
        public async Task<IActionResult> Delete(string name)
        {
            IReliableDictionary<string, int> votesDictionary = await this.stateManager.GetOrAddAsync<IReliableDictionary<string, int>>("counts");

            using (ITransaction tx = this.stateManager.CreateTransaction())
            {
                if (await votesDictionary.ContainsKeyAsync(tx, name))
                {
                    await votesDictionary.TryRemoveAsync(tx, name);
                    await tx.CommitAsync();
                    return new OkResult();
                }
                else
                {
                    return new NotFoundResult();
                }
            }
        }
    }
}
```

## <a name="connect-the-services"></a>Подключение служб

На следующем этапе свяжите две службы и сделайте так, чтобы веб-приложение интерфейсной части получало и обрабатывало данные голосования, полученные из серверной службы.

Service Fabric позволяет пользователям самим определять способ взаимодействия со службами Reliable Services. В составе одного приложения одни службы могут быть доступны по протоколу TCP. Другие службы могут быть доступны через API REST HTTP, третьи — через веб-сокеты. Сведения о доступных возможностях и их преимуществах и недостатках см. в статье [Подключение к службам в Service Fabric и взаимодействие с ними](service-fabric-connect-and-communicate-with-services.md).

Для выполнения задач этого руководства используется [веб-API ASP.NET Core](service-fabric-reliable-services-communication-aspnetcore.md) и [обратный прокси-сервер Service Fabric](service-fabric-reverseproxy.md). Это позволяет интерфейсной веб-службе VotingWeb взаимодействовать с серверной службой VotingData. Обратный прокси-сервер по умолчанию использует порт 19081 и должен работать в этом руководстве. Порт обратного прокси-сервера указан в шаблоне Azure Resource Manager, используемом для настройки кластера. Чтобы определить, какой порт будет использоваться, просмотрите ресурс **Microsoft.ServiceFabric/clusters** в шаблоне кластера: 

```json
"nodeTypes": [
          {
            ...
            "httpGatewayEndpointPort": "[variables('nt0fabricHttpGatewayPort')]",
            "isPrimary": true,
            "vmInstanceCount": "[parameters('nt0InstanceCount')]",
            "reverseProxyEndpointPort": "[parameters('SFReverseProxyPort')]"
          }
        ],
```
Чтобы найти порт обратного прокси-сервера, используемого на локальном кластере разработки, просмотрите данные об элементе **HttpApplicationGatewayEndpoint** в манифесте локального кластера Service Fabric.
1. Откройте окно браузера и перейдите по адресу http:\//localhost:19080, чтобы открыть средство Service Fabric Explorer.
2. Выберите **Кластер -> Манифест**.
3. Запомните порт элемента HttpApplicationGatewayEndpoint. По умолчанию это должен быть порт 19081. Если это не порт 19081, необходимо изменить порт в методе GetProxyAddress следующего кода VotesController.cs.

<a id="updatevotecontroller" name="updatevotecontroller_anchor"></a>

### <a name="update-the-votescontrollercs-file"></a>Обновление файла VotesController.cs

В проекте **VotingWeb** откройте файл *Controllers/VotesController.cs*.  Замените содержимое определения класса `VotesController` следующим образом, а затем сохраните изменения. Если порт обратного прокси-сервера, обнаруженный на предыдущем шаге, не является портом 19081, измените порт, используемый в методе GetProxyAddress, с 19081 на обнаруженный порт.

```csharp
public class VotesController : Controller
{
    private readonly HttpClient httpClient;
    private readonly FabricClient fabricClient;
    private readonly StatelessServiceContext serviceContext;

    public VotesController(HttpClient httpClient, StatelessServiceContext context, FabricClient fabricClient)
    {
        this.fabricClient = fabricClient;
        this.httpClient = httpClient;
        this.serviceContext = context;
    }

    // GET: api/Votes
    [HttpGet("")]
    public async Task<IActionResult> Get()
    {
        Uri serviceName = VotingWeb.GetVotingDataServiceName(this.serviceContext);
        Uri proxyAddress = this.GetProxyAddress(serviceName);

        ServicePartitionList partitions = await this.fabricClient.QueryManager.GetPartitionListAsync(serviceName);

        List<KeyValuePair<string, int>> result = new List<KeyValuePair<string, int>>();

        foreach (Partition partition in partitions)
        {
            string proxyUrl =
                $"{proxyAddress}/api/VoteData?PartitionKey={((Int64RangePartitionInformation) partition.PartitionInformation).LowKey}&PartitionKind=Int64Range";

            using (HttpResponseMessage response = await this.httpClient.GetAsync(proxyUrl))
            {
                if (response.StatusCode != System.Net.HttpStatusCode.OK)
                {
                    continue;
                }

                result.AddRange(JsonConvert.DeserializeObject<List<KeyValuePair<string, int>>>(await response.Content.ReadAsStringAsync()));
            }
        }

        return this.Json(result);
    }

    // PUT: api/Votes/name
    [HttpPut("{name}")]
    public async Task<IActionResult> Put(string name)
    {
        Uri serviceName = VotingWeb.GetVotingDataServiceName(this.serviceContext);
        Uri proxyAddress = this.GetProxyAddress(serviceName);
        long partitionKey = this.GetPartitionKey(name);
        string proxyUrl = $"{proxyAddress}/api/VoteData/{name}?PartitionKey={partitionKey}&PartitionKind=Int64Range";

        StringContent putContent = new StringContent($"{{ 'name' : '{name}' }}", Encoding.UTF8, "application/json");
        putContent.Headers.ContentType = new MediaTypeHeaderValue("application/json");

        using (HttpResponseMessage response = await this.httpClient.PutAsync(proxyUrl, putContent))
        {
            return new ContentResult()
            {
                StatusCode = (int) response.StatusCode,
                Content = await response.Content.ReadAsStringAsync()
            };
        }
    }

    // DELETE: api/Votes/name
    [HttpDelete("{name}")]
    public async Task<IActionResult> Delete(string name)
    {
        Uri serviceName = VotingWeb.GetVotingDataServiceName(this.serviceContext);
        Uri proxyAddress = this.GetProxyAddress(serviceName);
        long partitionKey = this.GetPartitionKey(name);
        string proxyUrl = $"{proxyAddress}/api/VoteData/{name}?PartitionKey={partitionKey}&PartitionKind=Int64Range";

        using (HttpResponseMessage response = await this.httpClient.DeleteAsync(proxyUrl))
        {
            if (response.StatusCode != System.Net.HttpStatusCode.OK)
            {
                return this.StatusCode((int) response.StatusCode);
            }
        }

        return new OkResult();
    }


    /// <summary>
    /// Constructs a reverse proxy URL for a given service.
    /// Example: http://localhost:19081/VotingApplication/VotingData/
    /// </summary>
    /// <param name="serviceName"></param>
    /// <returns></returns>
    private Uri GetProxyAddress(Uri serviceName)
    {
        return new Uri($"http://localhost:19081{serviceName.AbsolutePath}");
    }

    /// <summary>
    /// Creates a partition key from the given name.
    /// Uses the zero-based numeric position in the alphabet of the first letter of the name (0-25).
    /// </summary>
    /// <param name="name"></param>
    /// <returns></returns>
    private long GetPartitionKey(string name)
    {
        return Char.ToUpper(name.First()) - 'A';
    }
}
```

<a id="walkthrough" name="walkthrough_anchor"></a>

## <a name="walk-through-the-voting-sample-application"></a>Описание примера приложения для голосования

Приложение для голосования состоит из двух служб:

* Служба веб-интерфейса (VotingWeb) — служба веб-интерфейса ASP.NET Core, которая обслуживает веб-страницу и предоставляет доступ к веб-API для связи с внутренней службой.
* Внутренняя служба (VotingData) — веб-служба ASP.NET Core, которая предоставляет API для сохранения результатов голосования в надежном словаре на диске.

![Диаграмма приложения](./media/service-fabric-tutorial-create-dotnet-app/application-diagram.png)

Во время голосования в приложении происходят следующие события:

1. JavaScript отправляет запрос о голосовании веб-API в службе веб-интерфейса в виде запроса HTTP PUT.

2. Служба веб-интерфейса использует прокси, чтобы обнаружить и перенаправить запрос HTTP PUT внутренней службе.

3. Внутренняя служба принимает входящий запрос и сохраняет обновленный результат в надежном словаре, который реплицируется на несколько узлов в кластере и сохраняется на диске. Все данные приложения хранятся в кластере, поэтому база данных не требуется.

## <a name="debug-in-visual-studio"></a>Отладка в Visual Studio

При отладке приложения в Visual Studio вы используете локальный кластер разработки Service Fabric. Вы можете настроить отладку для своего сценария. В этом приложении храните данные во внутренней службе с помощью надежного словаря. По умолчанию при остановке отладчика Visual Studio удаляет приложение. При удалении приложения данные во внутренней службе также удаляются. Для сохранения данных между сеансами отладки можно изменить свойство **Режим отладки приложения** проекта **Voting** в Visual Studio.

Чтобы посмотреть, как выполняется код, сделайте следующее:

1. Откройте файл **VotingWeb\VotesController.cs** и установите точку останова в методе **Put** этого веб-API (строка 72).

2. Откройте файл **VotingData\VoteDataController.cs** и установите точку останова в методе **Put** этого веб-API (строка 54).

3. Нажмите клавишу **F5**, чтобы запустить приложение в режиме отладки.

4. Вернитесь в браузер и выберите один из вариантов голосования или добавьте новый вариант. Выполнение остановится на первой точке останова в контроллере API клиентского веб-интерфейса.
    

   1. Здесь код JavaScript в браузере отправляет запрос контроллеру веб-API в службе веб-интерфейса.

      ![Добавление интерфейсной службы Vote](./media/service-fabric-tutorial-create-dotnet-app/addvote-frontend.png)

   2. Сначала создайте URL-адрес обратного прокси-сервера внутренней службы **(1)** .
   3. Затем отправьте HTTP-запрос PUT к обратному прокси-серверу **(2)** .
   4. Наконец, верните ответ внутренней службой клиенту **(3)** .

5. Нажмите клавишу **F5**, чтобы продолжить.
   1. Теперь вы находитесь в точке останова внутренней службы.

      ![Добавление внутренней службы Vote](./media/service-fabric-tutorial-create-dotnet-app/addvote-backend.png)

   2. В первой строке метода **(1)** используйте `StateManager`, чтобы получить или добавить надежный словарь `counts`.
   3. Все взаимодействие с надежным словарем осуществляется с помощью транзакций. Для создания транзакции используется инструкция using **(2)** .
   4. В транзакции обновите значение соответствующего ключа для варианта голосования и зафиксируйте операцию **(3)** . После возврата метода фиксации данные в словаре обновляются и реплицируются на другие узлы в кластере. Теперь данные безопасно хранятся в кластере, и внутренняя служба может выполнять отработку отказа на другие узлы, сохраняя доступ к данным.
6. Нажмите клавишу **F5**, чтобы продолжить.

Чтобы остановить сеанс отладки, нажмите **SHIFT + F5**.

## <a name="next-steps"></a>Дальнейшие действия

В этой части руководства вы узнали, как выполнить следующие действия:

> [!div class="checklist"]
> * Создание службы веб-API ASP.NET Core как надежной службы с отслеживанием состояния
> * Создание службы веб-приложения ASP.NET Core как службы без отслеживания состояния
> * Использование обратного прокси-сервера для связи со службой с отслеживанием состояния

Перейдите к следующему руководству:
> [!div class="nextstepaction"]
> [Развертывание приложения в Azure](service-fabric-tutorial-deploy-app-to-party-cluster.md)
