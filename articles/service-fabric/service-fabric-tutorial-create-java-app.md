---
title: Руководство. Создание приложения Java в Azure Service Fabric
description: В этом руководстве вы узнаете, как создать приложение Java надежной службы с внешним интерфейсом, создать серверную часть с отслеживанием состояния надежных служб и развернуть приложение в кластер.
ms.topic: tutorial
ms.date: 09/01/2018
ms.custom: mvc, seo-java-july2019, seo-java-august2019, seo-java-september2019, devx-track-java, devx-track-js
ms.openlocfilehash: d1f66177b1939d612a5ee80c99109c397b56cb37
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91532199"
---
# <a name="tutorial-create-an-application-with-a-java-api-front-end-service-and-a-stateful-back-end-service-on-azure-service-fabric"></a>Руководство. Создание приложения с интерфейсной службой API Java и серверной службой с отслеживанием состояния в Azure Service Fabric

Это руководство представляет первую часть цикла. Выполнив инструкции из этого руководства, вы получите приложение для голосования с клиентской частью в виде веб-приложения Java, которое сохраняет результаты голосования во внутренней службе с отслеживанием состояния в Azure Service Fabric. В этой серии руководств требуется, чтобы у вас был работающий компьютер для разработчиков Mac OSX или Linux. Если вы не хотите вручную создавать приложение для голосования, вы можете [скачать исходный код](https://github.com/Azure-Samples/service-fabric-java-quickstart) для завершенного приложения и сразу перейти к [описанию примера приложения для голосования](service-fabric-tutorial-create-java-app.md#walk-through-the-voting-sample-application). Кроме того, ознакомьтесь с [кратким руководством по развертыванию приложения Java в Azure Service Fabric](service-fabric-quickstart-java-reliable-services.md).

![Пример для голосования Service Fabric](./media/service-fabric-tutorial-create-java-app/service-fabric-java-voting-app-sample.png)

Из этого цикла руководств вы узнаете, как выполнять следующие задачи:
> [!div class="checklist"]
> * Развертывание приложения Java служб Service Fabric Reliable Services в Azure.
> * [Развертывание и отладка приложения в локальном кластере.](service-fabric-tutorial-debug-log-local-cluster.md)
> * [Развертывание приложения в кластере Azure.](service-fabric-tutorial-java-deploy-azure.md)
> * [Настройка мониторинга и диагностики приложения](service-fabric-tutorial-java-elk.md)
> * [Настройка процесса непрерывной интеграции и доставки](service-fabric-tutorial-java-jenkins.md)


В первой части цикла вы узнаете, как выполнять такие задачи:

> [!div class="checklist"]
> * Создание надежной службы с отслеживанием состояния Java.
> * Создание службы веб-приложения без отслеживания состояния Java.
> * Использование удаленного взаимодействия службы для связи со службой с отслеживанием состояния.
> * Развертывание приложения в локальном кластере Service Fabric.

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы с этим руководством выполните следующие действия:

* Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Настройте среду разработки для [Mac](service-fabric-get-started-mac.md) или [Linux](service-fabric-get-started-linux.md). Следуя инструкциям, установите подключаемый модуль Eclipse, Gradle, пакет SDK для Service Fabric и интерфейс командной строки Service Fabric (sfctl).

## <a name="create-the-front-end-java-stateless-service"></a>Создание интерфейсной службы без отслеживания состояния Java

Сначала создайте веб-интерфейс приложения для голосования. Пользовательский веб-интерфейс на базе AngularJS отправляет запросы в службу без отслеживания состояния Java, которая запускает упрощенный HTTP-сервер. Эта служба обрабатывает каждый запрос и отправляет удаленной вызов процедуры в службу с отслеживанием состояния для хранения голосов. 

1. Откройте Eclipse.

2. Создайте проект, последовательно выбрав **Файл** > **Создать** > **Другое** > **Service Fabric** > **Проект Service Fabric**.

    ![Новый проект Service Fabric в Eclipse](./media/service-fabric-tutorial-create-java-app/service-fabric-project-wizard.png)

3. В диалоговом окне **ServiceFabric Project Wizard** (Мастер проектов Service Fabric) назовите проект **Voting** и нажмите кнопку **Далее**.

    ![Выбор службы без отслеживания состояния Java в диалоговом окне создания службы](./media/service-fabric-tutorial-create-java-app/name-service-fabric-project-wizard.png) 

4. На странице **Добавление службы** выберите **Служба без отслеживания состояния** и укажите имя службы **VotingWeb**. Нажмите кнопку **Готово**, чтобы создать проект.

    ![Создание службы без отслеживания состояния для проекта Service Fabric]( ./media/service-fabric-tutorial-create-java-app/add-service-fabric-votingweb-service.png)

    Eclipse создает приложение и проект службы и отображает их в обозревателе пакетов.

    ![Обозреватель пакетов Eclipse после создания приложения]( ./media/service-fabric-tutorial-create-java-app/eclipse-package-explorer.png)

Таблица содержит краткое описание каждого элемента в обозревателе пакета с предыдущего снимка экрана. 

| **Элемент обозревателя пакетов** | **Описание** |
| --- | --- |
| PublishProfiles | Содержит файлы JSON, описывающие сведения о профилях локального кластера и кластера Azure Service Fabric. Содержимое этих файлов используется подключаемым модулем при развертывании приложения. |
| Скрипты | Содержит вспомогательные скрипты, которые можно использовать из командной строки, чтобы быстро управлять приложением с кластером. |
| VotingApplication | Содержит приложение Service Fabric, которое отправляется в кластер Service Fabric. |
| VotingWeb | Содержит исходные файлы интерфейсной службы без отслеживания состояния вместе со связанным файлом сборки Gradle. |
| build.gradle | Файл Gradle используется для управления проектом. |
| settings.gradle | Содержит имена проектов Gradle в этой папке. |

### <a name="add-html-and-javascript-to-the-votingweb-service"></a>Добавление HTML и JavaScript в службу VotingWeb

Чтобы добавить пользовательский интерфейс, который может быть преобразован для просмотра службой без отслеживания состояния, добавьте HTML-файл. Этот HTML-файл преобразовывается для просмотра на упрощенном HTTP-сервере, встроенном в службу Java без отслеживания состояния.

1. Разверните каталог *VotingApplication*, чтобы получить доступ к каталогу *VotingApplication/VotingWebPkg/Code*.

2. Щелкните правой кнопкой мыши каталог *Код*, а затем последовательно выберите **Создать** > **Папка**.

3. Назовите папку *wwwroot* и выберите **Готово**.

    ![Eclipse создает папку wwwroot](./media/service-fabric-tutorial-create-java-app/create-wwwroot-folder.png)

4. Добавьте файл в **wwwroot**, который называется **index.html**, и вставьте в него следующее содержимое.

```html
<!DOCTYPE html>
<html>
<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular-ui-bootstrap/0.13.4/ui-bootstrap-tpls.min.js"></script>
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
<body>

<script>
var app = angular.module('VotingApp', ['ui.bootstrap']);
app.controller("VotingAppController", ['$rootScope', '$scope', '$http', '$timeout', function ($rootScope, $scope, $http, $timeout) {
    $scope.votes = [];
    
    $scope.refresh = function () {
        $http.get('getStatelessList')
            .then(function successCallback(response) {
        $scope.votes = Object.assign(
            {},
            ...Object.keys(response.data) .
            map(key => ({[decodeURI(key)]: response.data[key]}))
        )
        },
        function errorCallback(response) {
            alert(response);
        });
    };

    $scope.remove = function (item) {
       $http.get("removeItem", {params: { item: encodeURI(item) }})
            .then(function successCallback(response) {
                $scope.refresh();
            },
            function errorCallback(response) {
                alert(response);
            });
    };

    $scope.add = function (item) {
        if (!item) {return;}
        $http.get("addItem", {params: { item: encodeURI(item) }})
            .then(function successCallback(response) {
                $scope.refresh();
            },
            function errorCallback(response) {
                alert(response);
            });
    };
}]);
</script>

<div ng-app="VotingApp" ng-controller="VotingAppController" ng-init="refresh()">
    <div class="container-fluid">
        <div class="row">
            <div class="col-xs-8 col-xs-offset-2 text-center">
                <h2>Service Fabric Voting Sample</h2>
            </div>
        </div>

        <div class="row">
            <div class="col-xs-offset-2">
                <form style="width:50% ! important;" class="center-block">
                    <div class="col-xs-6 form-group">
                        <input id="txtAdd" type="text" class="form-control" placeholder="Add voting option" ng-model="item" />
                    </div>
                    <button id="btnAdd" class="btn btn-default" ng-click="add(item)">
                        <span class="glyphicon glyphicon-plus" aria-hidden="true"></span>
                        Add
                    </button>
                </form>
            </div>
        </div>

        <hr />

        <div class="row">
            <div class="col-xs-8 col-xs-offset-2">
                <div class="row">
                    <div class="col-xs-4">
                        Click to vote
                    </div>
                </div>
                <div class="row top-buffer" ng-repeat="(key, value)  in votes">
                    <div class="col-xs-8">
                        <button class="btn btn-success text-left btn-block" ng-click="add(key)">
                            <span class="pull-left">
                                {{key}}
                            </span>
                            <span class="badge pull-right">
                                {{value}} Votes
                            </span>
                        </button>
                    </div>
                    <div class="col-xs-4">
                        <button class="btn btn-danger pull-right btn-block" ng-click="remove(key)">
                            <span class="glyphicon glyphicon-remove" aria-hidden="true"></span>
                            Remove
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

</body>
</html>
```

### <a name="update-the-votingwebjava-file"></a>Обновление файла VotingWeb.java

В подпроекте **VotingWeb** откройте файл *VotingWeb/src/statelessservice/VotingWeb.java*. Служба **VotingWeb** является шлюзом службы без отслеживания состояния, отвечающим за настройку прослушивателя для обмена данными для интерфейсного API.

Замените существующий метод **createServiceInstanceListeners** в файле следующим и сохраните изменения.

```java
@Override
protected List<ServiceInstanceListener> createServiceInstanceListeners() {

    EndpointResourceDescription endpoint = this.getServiceContext().getCodePackageActivationContext().getEndpoint(webEndpointName);
    int port = endpoint.getPort();

    List<ServiceInstanceListener> listeners = new ArrayList<ServiceInstanceListener>();
    listeners.add(new ServiceInstanceListener((context) -> new HttpCommunicationListener(context, port)));
    return listeners;
}
```

### <a name="add-the-httpcommunicationlistenerjava-file"></a>Добавление файла HTTPCommunicationListener.java

Прослушиватель для обмена данными HTTP действует в качестве контроллера, который настраивает сервер HTTP и предоставляет API, определяющий действия голосования. Щелкните правой кнопкой мыши пакет *statelessservice* в папке *VotingWeb/src/statelessservice*, а затем последовательно выберите **Создать** > **Файл**.  Назовите файл *HttpCommunicationListener.java* и щелкните **Готово**.

Замените содержимое файла следующим образом, а затем сохраните изменения.  Затем на этапе обновления файл HttpCommunicationListener.java будет изменен для преобразования для просмотра, чтения и записи данных голосования, полученных из серверной службы.  Теперь прослушиватель просто возвращает статические HTML для приложения для голосования.

```java
// ------------------------------------------------------------
//  Copyright (c) Microsoft Corporation.  All rights reserved.
//  Licensed under the MIT License (MIT). See License.txt in the repo root for license information.
// ------------------------------------------------------------

package statelessservice;

import com.google.gson.Gson;
import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import com.sun.net.httpserver.Headers;

import java.io.File;
import java.io.OutputStream;
import java.io.FileInputStream;

import java.net.InetSocketAddress;
import java.net.URI;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.CompletableFuture;
import java.util.logging.Level;
import java.util.logging.Logger;

import microsoft.servicefabric.services.communication.runtime.CommunicationListener;
import microsoft.servicefabric.services.runtime.StatelessServiceContext;
import microsoft.servicefabric.services.client.ServicePartitionKey;
import microsoft.servicefabric.services.remoting.client.ServiceProxyBase;
import microsoft.servicefabric.services.communication.client.TargetReplicaSelector;
import system.fabric.CancellationToken;

public class HttpCommunicationListener implements CommunicationListener {

    private static final Logger logger = Logger.getLogger(HttpCommunicationListener.class.getName());

    private static final String HEADER_CONTENT_TYPE = "Content-Type";
    private static final int STATUS_OK = 200;
    private static final int STATUS_NOT_FOUND = 404;
    private static final int STATUS_ERROR = 500;
    private static final String RESPONSE_NOT_FOUND = "404 (Not Found) \n";
    private static final String MIME = "text/html";
    private static final String ENCODING = "UTF-8";

    private static final String ROOT = "wwwroot/";
    private static final String FILE_NAME = "index.html";
    private StatelessServiceContext context;
    private com.sun.net.httpserver.HttpServer server;
    private ServicePartitionKey partitionKey;
    private final int port;

    public HttpCommunicationListener(StatelessServiceContext context, int port) {
        this.partitionKey = new ServicePartitionKey(0);
        this.context = context;
        this.port = port;
    }

    // Called by openAsync when the class is instantiated
    public void start() {
        try {
            logger.log(Level.INFO, "Starting Server");
            server = com.sun.net.httpserver.HttpServer.create(new InetSocketAddress(this.port), 0);
        } catch (Exception ex) {
            logger.log(Level.SEVERE, null, ex);
            throw new RuntimeException(ex);
        }

        // Responsible for rendering the HTML layout described in the previous step
        server.createContext("/", new HttpHandler() {
            @Override
            public void handle(HttpExchange t) {
                try {
                    File file = new File(ROOT + FILE_NAME).getCanonicalFile();

                    if (!file.isFile()) {
                      // Object does not exist or is not a file: reject with 404 error.
                      t.sendResponseHeaders(STATUS_NOT_FOUND, RESPONSE_NOT_FOUND.length());
                      OutputStream os = t.getResponseBody();
                      os.write(RESPONSE_NOT_FOUND.getBytes());
                      os.close();
                    } else {
                      Headers h = t.getResponseHeaders();
                      h.set(HEADER_CONTENT_TYPE, MIME);
                      t.sendResponseHeaders(STATUS_OK, 0);
    
                      OutputStream os = t.getResponseBody();
                      FileInputStream fs = new FileInputStream(file);
                      final byte[] buffer = new byte[0x10000];
                      int count = 0;
                      while ((count = fs.read(buffer)) >= 0) {
                        os.write(buffer,0,count);
                      }

                      fs.close();
                      os.close();
                    }
                } catch (Exception e) {
                    logger.log(Level.WARNING, null, e);
                }
            }
        });

        /*
        [Replace this entire comment block in the 'Connect the services' section]
        */

        server.setExecutor(null);
        server.start();
    }

    //Helper method to parse raw HTTP requests
    private Map<String, String> queryToMap(String query){
        Map<String, String> result = new HashMap<String, String>();
        for (String param : query.split("&")) {
            String pair[] = param.split("=");
            if (pair.length>1) {
                result.put(pair[0], pair[1]);
            }else{
                result.put(pair[0], "");
            }
        }
        return result;
    }

    //Called closeAsync when the service is shut down
    private void stop() {
        if (null != server)
            server.stop(0);
    }

    //Called by the Service Fabric runtime when this service is created on a node
    @Override
    public CompletableFuture<String> openAsync(CancellationToken cancellationToken) {
        this.start();
                    logger.log(Level.INFO, "Opened Server");
        String publishUri = String.format("http://%s:%d/", this.context.getNodeContext().getIpAddressOrFQDN(), port);
        return CompletableFuture.completedFuture(publishUri);
    }

    //Called by the Service Fabric runtime when the service is shut down
    @Override
    public CompletableFuture<?> closeAsync(CancellationToken cancellationToken) {
        this.stop();
        return CompletableFuture.completedFuture(true);
    }

    //Called by the Service Fabric runtime to forcibly shut this listener down
    @Override
    public void abort() {
        this.stop();
    }
}
```

### <a name="configure-the-listening-port"></a>Настройка порта прослушивания

Когда создается интерфейсная служба VotingWebService, Service Fabric выбирает порт для ее прослушивания.  Так как служба VotingWeb выступает в роли интерфейсной части этого приложения и принимает внешний трафик, мы привяжем эту службу к фиксированному и хорошо известному порту. В обозревателе пакетов откройте файл *VotingApplication/VotingWebPkg/ServiceManifest.xml*.  Найдите ресурс **Endpoint** в разделе **Resources** и задайте для свойства **Port** значение 8080 (мы продолжаем использовать этот порт в руководстве). Для локального развертывания и запуска приложения порт прослушивания приложения должен быть открыт и доступен на компьютере. Вставьте следующий фрагмент кода в элемент **ServiceManifest** (под элементом ```<DataPackage>```).

```xml
<Resources>
    <Endpoints>
        <!-- This endpoint is used by the communication listener to obtain the port on which to
            listen. Please note that if your service is partitioned, this port is shared with
            replicas of different partitions that are placed in your code. -->
        <Endpoint Name="WebEndpoint" Protocol="http" Port="8080" />
    </Endpoints>
  </Resources>
```

## <a name="add-a-stateful-back-end-service-to-your-application"></a>Добавление в приложение серверной службы с отслеживанием состояния

Теперь, когда схема службы веб-API Java завершена, продолжим и завершим серверную службу с отслеживанием состояния.

Service Fabric позволяет согласованно и надежно хранить данные прямо в службе с помощью надежных коллекций. Надежные коллекции представляют собой набор высокодоступных и надежных коллекций классов. Эти классы могут использовать все, кто использовал коллекции Java.

1. В обозревателе пакетов в проекте приложения щелкните правой кнопкой мыши **Voting** и последовательно выберите **Service Fabric** > **Add Service Fabric Service** (Добавить службу Service Fabric).

2. В диалоговом окне **Добавить службу** выберите **Служба с отслеживанием состояния**, назовите службу **VotingDataService** и щелкните **Добавить службу**.

    После создания проекта службы в вашем приложении будут две службы. По мере разработки приложения можно добавить дополнительные службы точно таким же образом. Каждая из этих служб может быть отдельно обновлена, в том числе до определенной версии.

3. Eclipse создает проект службы и отображает его в обозревателе пакетов.

    ![Обозреватель проектов Eclipse](./media/service-fabric-tutorial-create-java-app/service-fabric-package-explorer-java.png)

### <a name="add-the-votingdataservicejava-file"></a>Добавление файла VotingDataService.java

Файл *VotingDataService.java* содержит методы, в которых содержится логика для извлечения, добавления и удаления голосов из надежных коллекций. Добавьте следующие методы класса **VotingDataService** в файл *VotingDataService/src/statefulservice/VotingDataService.java*.

```java
package statefulservice;

import java.util.HashMap;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.atomic.AtomicInteger;

import microsoft.servicefabric.services.communication.runtime.ServiceReplicaListener;

import microsoft.servicefabric.services.runtime.StatefulService;

import microsoft.servicefabric.services.remoting.fabrictransport.runtime.FabricTransportServiceRemotingListener;

import microsoft.servicefabric.data.ReliableStateManager;
import microsoft.servicefabric.data.Transaction;
import microsoft.servicefabric.data.collections.ReliableHashMap;
import microsoft.servicefabric.data.utilities.AsyncEnumeration;
import microsoft.servicefabric.data.utilities.KeyValuePair;

import system.fabric.StatefulServiceContext;

import rpcmethods.VotingRPC;

class VotingDataService extends StatefulService implements VotingRPC {
    private static final String MAP_NAME = "votesMap";
    private ReliableStateManager stateManager;

    protected VotingDataService (StatefulServiceContext statefulServiceContext) {
        super (statefulServiceContext);
    }

    @Override
    protected List<ServiceReplicaListener> createServiceReplicaListeners() {
        this.stateManager = this.getReliableStateManager();
        ArrayList<ServiceReplicaListener> listeners = new ArrayList<>();

        listeners.add(new ServiceReplicaListener((context) -> {
            return new FabricTransportServiceRemotingListener(context,this);
        }));

        return listeners;
    }

    // Method that will be invoked via RPC from the front end to retrieve the complete set of votes in the map
    public CompletableFuture<HashMap<String,String>> getList() {
        HashMap<String, String> tempMap = new HashMap<String, String>();

        try {

            ReliableHashMap<String, String> votesMap = stateManager
                    .<String, String> getOrAddReliableHashMapAsync(MAP_NAME).get();

            Transaction tx = stateManager.createTransaction();
            AsyncEnumeration<KeyValuePair<String, String>> kv = votesMap.keyValuesAsync(tx).get();
            while (kv.hasMoreElementsAsync().get()) {
                KeyValuePair<String, String> k = kv.nextElementAsync().get();
                tempMap.put(k.getKey(), k.getValue());
            }

            tx.close();


        } catch (Exception e) {
            e.printStackTrace();
        }

        return CompletableFuture.completedFuture(tempMap);
    }

    // Method that will be invoked via RPC from the front end to add an item to the votes list or to increase the
    // vote count for a particular item
    public CompletableFuture<Integer> addItem(String itemToAdd) {
        AtomicInteger status = new AtomicInteger(-1);

        try {

            ReliableHashMap<String, String> votesMap = stateManager
                    .<String, String> getOrAddReliableHashMapAsync(MAP_NAME).get();

            Transaction tx = stateManager.createTransaction();
            votesMap.computeAsync(tx, itemToAdd, (k, v) -> {
                if (v == null) {
                    return "1";
                }
                else {
                    int numVotes = Integer.parseInt(v);
                    numVotes = numVotes + 1;
                    return Integer.toString(numVotes);
                }
            }).get();

            tx.commitAsync().get();
            tx.close();

            status.set(1);
        } catch (Exception e) {
            e.printStackTrace();
        }

        return CompletableFuture.completedFuture(new Integer(status.get()));
    }

    // Method that will be invoked via RPC from the front end to remove an item
    public CompletableFuture<Integer> removeItem(String itemToRemove) {
        AtomicInteger status = new AtomicInteger(-1);
        try {
            ReliableHashMap<String, String> votesMap = stateManager
                    .<String, String> getOrAddReliableHashMapAsync(MAP_NAME).get();

            Transaction tx = stateManager.createTransaction();
            votesMap.removeAsync(tx, itemToRemove).get();
            tx.commitAsync().get();
            tx.close();

            status.set(1);
        } catch (Exception e) {
            e.printStackTrace();
        }

        return CompletableFuture.completedFuture(new Integer(status.get()));
    }

}
```

Теперь создается схема для интерфейсной службы без отслеживания состояния и серверной службы.

## <a name="create-the-communication-interface-to-your-application"></a>Создание интерфейса для обмена данными с приложением

 Следующий шаг — подключение интерфейсной службы без отслеживания состояния и внутренней службы. Обе службы используют интерфейс VotingRPC, который определяет операции приложения для голосования. Этот интерфейс реализуется с помощью интерфейсных и серверных служб, включая удаленные вызовы процедур между двумя службами. К сожалению, Eclipse не поддерживает добавление подпроектов Gradle, поэтому пакет, который содержит интерфейс, нужно добавить вручную.

1. Щелкните правой кнопкой мыши проект **Voting** в обозревателе пакетов и выберите **Новая** > **Папка**. Назовите папку **VotingRPC/src/rpcmethods**.

    ![Создание пакета VotingRPC в обозревателе пакетов Eclipse](./media/service-fabric-tutorial-create-java-app/create-voting-rpc-package-java.png)

3. Создайте файл в папке *Voting/VotingRPC/src/rpcmethods*, который называется *VotingRPC.java*, и вставьте следующее в файл **VotingRPC.java**. 

    ```java
    package rpcmethods;
    
    import java.util.ArrayList;
    import java.util.concurrent.CompletableFuture;
    import java.util.List;
    import java.util.HashMap;
    
    import microsoft.servicefabric.services.remoting.Service;
    
    public interface VotingRPC extends Service {
        CompletableFuture<HashMap<String, String>> getList();
    
        CompletableFuture<Integer> addItem(String itemToAdd);
    
        CompletableFuture<Integer> removeItem(String itemToRemove);
    }
    ```

4. Создайте пустой файл *build.gradle* в каталоге *Voting/VotingRPC* и вставьте в него следующий код. Этот файл Gradle используется для создания JAR-файла, который импортируется другими службами. 

    ```gradle
    apply plugin: 'java'
    apply plugin: 'eclipse'
    
    sourceSets {
      main {
         java.srcDirs = ['src']
         output.classesDir = 'out/classes'
          resources {
           srcDirs = ['src']
         }
       }
    }
    
    clean.doFirst {
        delete "${projectDir}/out"
        delete "${projectDir}/VotingRPC.jar"
    }
    
    repositories {
        mavenCentral()
    }
    
    dependencies {
        compile ('com.microsoft.servicefabric:sf-actors:1.0.0')
    }
    
    jar {
        from configurations.compile.collect {
            (it.isDirectory() && !it.getName().contains("native")) ? it : zipTree(it)}

        manifest {
                attributes(
                'Main-Class': 'rpcmethods.VotingRPC')
            baseName "VotingRPC"
            destinationDir = file('./')
        }
    
        exclude 'META-INF/*.RSA', 'META-INF/*.SF','META-INF/*.DSA'
    }

    defaultTasks 'clean', 'jar'
    ```

5. В файле *Voting/settings.gradle* добавьте строку, которая включает только что созданный проект в сборку. 

    ```gradle
    include ':VotingRPC'
    ```

6. В файле *Voting/VotingWeb/src/statelessservice/HttpCommunicationListener.java* замените блок комментариев следующим.  

    ```java
    server.createContext("/getStatelessList", new HttpHandler() {
        @Override
        public void handle(HttpExchange t) {
            try {
                t.sendResponseHeaders(STATUS_OK,0);
                OutputStream os = t.getResponseBody();
    
                HashMap<String,String> list = ServiceProxyBase.create(VotingRPC.class, new URI("fabric:/VotingApplication/VotingDataService"), partitionKey, TargetReplicaSelector.DEFAULT, "").getList().get();
                String json = new Gson().toJson(list);
                os.write(json.getBytes(ENCODING));
                os.close();
            } catch (Exception e) {
                logger.log(Level.WARNING, null, e);
            }
        }
    });
    
    server.createContext("/removeItem", new HttpHandler() {
        @Override
        public void handle(HttpExchange t) {
            try {
                OutputStream os = t.getResponseBody();
                URI r = t.getRequestURI();
    
                Map<String, String> params = queryToMap(r.getQuery());
                String itemToRemove = params.get("item");

                Integer num = ServiceProxyBase.create(VotingRPC.class, new URI("fabric:/VotingApplication/VotingDataService"), partitionKey, TargetReplicaSelector.DEFAULT, "").removeItem(itemToRemove).get();
    
                if (num != 1)
                {
                    t.sendResponseHeaders(STATUS_ERROR, 0);
                } else {
                    t.sendResponseHeaders(STATUS_OK,0);
                }
    
                String json = new Gson().toJson(num);
                os.write(json.getBytes(ENCODING));
                os.close();
            } catch (Exception e) {
                logger.log(Level.WARNING, null, e);
            }
    
        }
    });
    
    server.createContext("/addItem", new HttpHandler() {
        @Override
        public void handle(HttpExchange t) {
            try {
                URI r = t.getRequestURI();
                Map<String, String> params = queryToMap(r.getQuery());
                String itemToAdd = params.get("item");

                OutputStream os = t.getResponseBody();
                Integer num = ServiceProxyBase.create(VotingRPC.class, new URI("fabric:/VotingApplication/VotingDataService"), partitionKey, TargetReplicaSelector.DEFAULT, "").addItem(itemToAdd).get();
                if (num != 1)
                {
                    t.sendResponseHeaders(STATUS_ERROR, 0);
                } else {
                    t.sendResponseHeaders(STATUS_OK,0);
                }
    
                String json = new Gson().toJson(num);
                os.write(json.getBytes(ENCODING));
                os.close();
            } catch (Exception e) {
                logger.log(Level.WARNING, null, e);
            }
        }
    });
    ```
7. Добавьте соответствующую инструкцию импорта в верхнюю часть файла *Voting/VotingWeb/src/statelessservice/HttpCommunicationListener.java*. 

    ```java
    import rpcmethods.VotingRPC; 
    ```

На этом этапе функции внешнего интерфейса, серверной части и RPC выполнены. Следующий этап — это настроить скрипты Gradle соответствующим образом прежде, чем развертывать их в кластер Service Fabric. 

## <a name="walk-through-the-voting-sample-application"></a>Описание примера приложения для голосования
Приложение для голосования состоит из двух служб:
- Служба веб-интерфейса (VotingWeb) — это служба веб-интерфейса Java, которая обслуживает веб-страницу и предоставляет доступ к API для связи с внутренней службой.
- Внутренняя служба (VotingDataService) — это веб-служба Java, которая определяет методы, вызываемые в удаленных вызовах процедуры для сохранения голосов.

![Образец схемы голосования](./media/service-fabric-tutorial-create-java-app/walkthrough-java-voting.png)

При выполнении действия в приложении (добавление элемента, голосование, удаление элемента) возникают следующие события:
1. JavaScript отправляет соответствующий запрос о голосовании веб-API в службе веб-интерфейса в виде HTTP-запроса.

2. Служба веб-интерфейса использует встроенную функцию удаленного взаимодействия со службой Service Fabric, чтобы найти и переадресовать запрос к внутренней службе. 

3. Внутренняя служба определяет методы, обновляющие результаты надежного словаря. Содержимое надежного словаря реплицируется на нескольких узлах в кластере и сохраняется на диске. Все данные приложений хранятся в кластере. 

## <a name="configure-gradle-scripts"></a>Настройка скриптов Gradle 

В этом разделе настраиваются скрипты Gradle для проекта. 

1. Замените содержимое файла *Voting/build.gradle* следующим.

    ```gradle
    apply plugin: 'java'
    apply plugin: 'eclipse'
    
    subprojects {
        apply plugin: 'java'
    }
    
    defaultTasks 'clean', 'jar', 'copyDeps'
    ```

2. Замените содержимое файла *Voting/VotingWeb/build.gradle* следующим.

    ```gradle
    apply plugin: 'java'
    apply plugin: 'eclipse'
    
    sourceSets {
      main {
         java.srcDirs = ['src']
         output.classesDir = 'out/classes'
          resources {
           srcDirs = ['src']
         }
       }
    }
    
    clean.doFirst {
        delete "${projectDir}/../lib"
        delete "${projectDir}/out"
        delete "${projectDir}/../VotingApplication/VotingWebPkg/Code/lib"
        delete "${projectDir}/../VotingApplication/VotingWebPkg/Code/VotingWeb.jar"
    }
    
    repositories {
        mavenCentral()
    }
    
    dependencies {
        compile ('com.microsoft.servicefabric:sf-actors:1.0.0-preview1')
        compile project(':VotingRPC')
    }
    
    task explodeDeps(type: Copy, dependsOn:configurations.compile) { task ->
        configurations.compile.filter {!it.toString().contains("native")}.each{
            from it
        }
    
        configurations.compile.filter {it.toString().contains("native")}.each{
            from zipTree(it)
        }
        into "../lib/"
        include "lib*.so", "*.jar"
    }
    
    task copyDeps<< {
        copy {
            from("../lib/")
            into("../VotingApplication/VotingWebPkg/Code/lib")
            include('lib*.so')
        }
    }
    
    compileJava.dependsOn(explodeDeps)
    
    jar {
        from configurations.compile.collect {(it.isDirectory() && !it.getName().contains("native")) ? it : zipTree(it)}
    
        manifest {
            attributes(
                'Main-Class': 'statelessservice.VotingWebServiceHost')
            baseName "VotingWeb"
            destinationDir = file('../VotingApplication/VotingWebPkg/Code/')
        }
    
        exclude 'META-INF/*.RSA', 'META-INF/*.SF','META-INF/*.DSA'
    }
    
    defaultTasks 'clean', 'jar', 'copyDeps'
    ```

3. Замените содержимое файла *Voting/VotingDataService/build.gradle*. 

    ```gradle
    apply plugin: 'java'
    apply plugin: 'eclipse'
    
    sourceSets {
      main {
         java.srcDirs = ['src']
         output.classesDir = 'out/classes'
          resources {
           srcDirs = ['src']
         }
       }
    }
    
    clean.doFirst {
        delete "${projectDir}/../lib"
        delete "${projectDir}/out"
        delete "${projectDir}/../VotingApplication/VotingDataServicePkg/Code/lib"
        delete "${projectDir}/../VotingApplication/VotingDataServicePkg/Code/VotingDataService.jar"
    }
    
    repositories {
        mavenCentral()
    }
    
    dependencies {
        compile ('com.microsoft.servicefabric:sf-actors:1.0.0-preview1')
        compile project(':VotingRPC')
    }
    
    task explodeDeps(type: Copy, dependsOn:configurations.compile) { task ->
        configurations.compile.filter {!it.toString().contains("native")}.each{
            from it
        }
    
        configurations.compile.filter {it.toString().contains("native")}.each{
            from zipTree(it)
        }
        into "../lib/"
        include "lib*.so", "*.jar"
    }
    
    compileJava.dependsOn(explodeDeps)
    
    task copyDeps<< {
        copy {
            from("../lib/")
            into("../VotingApplication/VotingDataServicePkg/Code/lib")
            include('lib*.so')
        }
    }
    
    jar {
        from configurations.compile.collect {
            (it.isDirectory() && !it.getName().contains("native")) ? it : zipTree(it)}
    
        manifest {
            attributes('Main-Class': 'statefulservice.VotingDataServiceHost')
    
            baseName "VotingDataService"
            destinationDir = file('../VotingApplication/VotingDataServicePkg/Code/')
        }
    
        exclude 'META-INF/*.RSA', 'META-INF/*.SF','META-INF/*.DSA'
    }
    
    defaultTasks 'clean', 'jar', 'copyDeps'
    ```

## <a name="deploy-application-to-local-cluster"></a>Развертывание приложения в локальном кластере

К этому моменту приложение готово к развертыванию в локальный кластер Service Fabric.

1. Щелкните правой кнопкой мыши проект **Voting** в обозревателе пакетов и выберите **Service Fabric** > **Создать приложение**, чтобы создать приложение.

2. Запустите удаленный кластер Service Fabric. Этот шаг зависит от среды разработки (Mac или Linux).

    Если используется Mac, запустите локальный кластер со следующей командой. Замените команду, переданную в параметр **-v**, путем к собственной рабочей области.

    ```bash
    docker run -itd -p 19080:19080 -p 8080:8080 -p --name sfonebox mcr.microsoft.com/service-fabric/onebox:latest
    ```
    Более подробные инструкции см. в статье [Настройка среды разработки для Mac OS X.](service-fabric-get-started-mac.md)

    При работе на компьютере под управлением Linux с помощью следующей команды запускается локальный кластер: 

    ```bash 
    sudo /opt/microsoft/sdk/servicefabric/common/clustersetup/devclustersetup.sh
    ```
    Более подробные инструкции см. в статье [Подготовка среды разработки в Linux](service-fabric-get-started-linux.md).

4. В обозревателе пакетов для Eclipse щелкните правой кнопкой мыши проект **Voting** и выберите **Service Fabric** > **Публикация приложения** 
5. В окне **Публикация приложения** выберите **Local.json** в раскрывающемся списке и выберите **Опубликовать**.
6. В веб-браузере перейдите по адресу http:\//localhost:8080, чтобы просмотреть выполняющееся приложение в локальном кластере Service Fabric. 

## <a name="next-steps"></a>Дальнейшие действия

В этой части руководства вы узнали, как выполнить следующие действия:

> [!div class="checklist"]
> * Создание службы Java как надежной службы с отслеживанием состояния.
> * Создание службы Java как веб-службы без отслеживания состояния.
> * Добавление интерфейса Java для обработки удаленных вызовов процедур между службами.
> * Настройка скриптов Gradle.
> * Создание и развертывание приложения в локальном кластере Service Fabric.

Перейдите к следующему руководству:
> [!div class="nextstepaction"]
> [Tutorial: Debug a Java application deployed on a local Service Fabric cluster](service-fabric-tutorial-debug-log-local-cluster.md) (Руководство. Отладка приложения Java, развернутого в локальном кластере Service Fabric)
