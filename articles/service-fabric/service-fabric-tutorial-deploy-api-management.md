---
title: Развертывание управления API с помощью Service Fabric в Azure
description: Узнайте, как быстро приступить к работе со службой управления API Azure и перенаправлять трафик во внутреннюю службу в Service Fabric.
ms.topic: conceptual
ms.date: 07/10/2019
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 681a1c5241743a0164d83d73753efa0b6c446109
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101735596"
---
# <a name="integrate-api-management-with-service-fabric-in-azure"></a>Развертывание управления API с помощью Service Fabric в Azure

Расширенный сценарий развертывания для службы управления API Azure в Service Fabric.  Управление API позволяет публиковать API-интерфейсы с широким набором правил маршрутизации для служб серверной части Service Fabric. Обычно, облачным приложениям требуется интерфейсный шлюз, который предоставляет единую точку передачи входящего трафика пользователей, устройств или других приложений. В Service Fabric шлюзом может быть любая служба без отслеживания состояния, предназначенная для обработки входящего трафика, например приложение ASP.NET Core, Центры событий, Центр Интернета вещей или служба управления API Azure.

В этой статье показано, как настроить [службу управления API Azure](../api-management/api-management-key-concepts.md) с помощью Service Fabric для перенаправления трафика во внутреннюю службу в Service Fabric.  Выполнив инструкции из этого руководства, вы развернете службу управления API в виртуальной сети и настроите API для отправки трафика во внутренние службы без отслеживания состояния. Дополнительные сведения о сценариях службы управления API Azure и Service Fabric см. в [обзорной статье](service-fabric-api-management-overview.md).


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="availability"></a>Доступность

> [!IMPORTANT]
> Эта функция доступна в ценовой категории **Премиум** и **Разработка** управления API, так как необходима поддержка виртуальной сети.

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы

* Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Установите [Azure PowerShell](/powershell/azure/install-az-ps) или [Azure CLI](/cli/azure/install-azure-cli).
* Создайте защищенный [кластер Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) в группе безопасности сети.
* Если вы развертываете кластер Windows, настройте среду разработки Windows. [Установите Visual Studio 2019](https://www.visualstudio.com), а также рабочие нагрузки **Разработка для Azure**, **ASP.NET и разработка веб-приложений** и **Кроссплатформенная разработка .NET Core**.  Теперь настройте [среду разработки .NET](service-fabric-get-started.md).

## <a name="network-topology"></a>Топология сети

Теперь, когда в Azure имеется защищенный [кластер Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md), можно развернуть службу управления API в подсети виртуальной сети и NSG, предназначенных для нее. Для этой статьи шаблон Resource Manager управления API предварительно настроен для использования имен виртуальной сети, подсети и группы безопасности сети, указанных в [руководстве по созданию кластера Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md). В этой статье мы развернем в Azure следующую топологию, в которой управление API и Service Fabric находятся в подсетях одной и той же виртуальной сети:

 ![Рисунок][sf-apim-topology-overview]

## <a name="sign-in-to-azure-and-select-your-subscription"></a>Вход в Azure и выбор подписки

Войдите в учетную запись Azure и выберите подписку, прежде чем выполнять команды Azure.

```powershell
Connect-AzAccount
Get-AzSubscription
Set-AzContext -SubscriptionId <guid>
```

```azurecli
az login
az account set --subscription <guid>
```

## <a name="deploy-a-service-fabric-back-end-service"></a>Развертывание внутренней службы Service Fabric

Прежде чем настроить в управлении API маршрутизацию трафика к серверной службе Service Fabric, необходимо создать и запустить эту службу для приема запросов.  

Создайте базовую надежную службу ASP.NET Core без отслеживания состояния с помощью стандартного шаблона проекта веб-API. Так мы получим конечную точку HTTP для службы, которую вы позднее предоставите через службу управления API Azure.

Запустите Visual Studio от имени администратора и создайте службу ASP.NET Core:

 1. В Visual Studio выберите последовательно «Файл» -> «Создать проект».
 2. Выберите шаблон приложения Service Fabric в облаке и присвойте ему имя **ApiApplication**.
 3. Выберите шаблон службы ASP.NET Core без отслеживания состояния и присвойте проекту имя **WebApiService**.
 4. Выберите шаблон проекта веб-API ASP.NET Core 2,1.
 5. После создания проекта откройте файл `PackageRoot\ServiceManifest.xml` и удалите атрибут `Port` из конфигурации ресурса конечной точки:

    ```xml
    <Resources>
      <Endpoints>
        <Endpoint Protocol="http" Name="ServiceEndpoint" Type="Input" />
      </Endpoints>
    </Resources>
    ```

    Удаление порта позволяет Service Fabric указать порт динамически из диапазона портов приложения, который открывается через группу безопасности сети в шаблоне диспетчер ресурсов кластера, что позволяет передавать трафик в него из управления API.

 6. Нажмите клавишу F5 в Visual Studio, чтобы убедиться, что веб-API доступен локально.

    Откройте Service Fabric Explorer и просмотрите подробные сведения об определенном экземпляре службы ASP.NET Core, чтобы увидеть базовый адрес, который прослушивает служба. Добавьте `/api/values` к базовому адресу и откройте его в браузере, чтобы вызвать метод Get из ValuesController в шаблоне веб-API. Он вернет ответ по умолчанию, предоставленный шаблоном, — массив JSON, который содержит две строки:

    ```json
    ["value1", "value2"]`
    ```

    Эту конечную точку вы предоставите через службу управления API в Azure.

 7. Теперь вы можете развернуть приложение в кластере в Azure. В Visual Studio щелкните правой кнопкой мыши проект приложения и выберите **Опубликовать**. Укажите конечную точку кластера (например, `mycluster.southcentralus.cloudapp.azure.com:19000`) для развертывания приложения в кластер Service Fabric в Azure.

В кластере Service Fabric в Azure должна запуститься служба ASP.NET Core без отслеживания состояния с именем `fabric:/ApiApplication/WebApiService`.

## <a name="download-and-understand-the-resource-manager-templates"></a>Загрузка и изучение шаблонов Resource Manager

Загрузите и сохраните следующие шаблоны Resource Manager и файл параметров:

* [network-apim.json][network-arm]
* [network-apim.parameters.json][network-parameters-arm]
* [apim.json][apim-arm]
* [apim.parameters.json][apim-parameters-arm]

Шаблон *network-apim.json* развертывает новую подсеть и группу безопасности сети в виртуальной сети, где развертывается кластер Service Fabric.

В следующих разделах описаны ресурсы, которые определяются в шаблоне *apim.json*. Дополнительные сведения вы можете получить по ссылкам на справочную документацию по шаблонам, которые представлены в каждом разделе. Параметры, определяемые в файле параметров *apim.parameters.json*, описаны ниже в этой статье.

### <a name="microsoftapimanagementservice"></a>Microsoft.ApiManagement/service

В [Microsoft.ApiManagement/service](/azure/templates/microsoft.apimanagement/service) описан экземпляр службы управления API: имя, номер SKU или уровень, расположение группы ресурсов, сведения об издателе, виртуальная сеть.

### <a name="microsoftapimanagementservicecertificates"></a>Microsoft.ApiManagement/service/certificates

При помощи [Microsoft.ApiManagement/service/certificates](/azure/templates/microsoft.apimanagement/service/certificates) настраивается защита для управления API. Управление API должно пройти аутентификацию с помощью кластера Service Fabric для обнаружения службы с помощью сертификата клиента, у которого есть доступ к кластеру. В этой статье используется сертификат, указанный при создании [кластера Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md#createvaultandcert_anchor), который по умолчанию предоставляет доступ к кластеру.

В этой статье используется один сертификат для аутентификации клиента и безопасности обмена данными между узлами кластера. Вы можете использовать отдельный сертификат клиента, если у вас есть сертификат, настроенный для доступа к кластеру Service Fabric. Укажите **имя**, **пароль** и **строку данных** (в формате base-64) закрытого ключа (PFX-файл) сертификата кластера, который вы указали при создании кластера Service Fabric.

### <a name="microsoftapimanagementservicebackends"></a>Microsoft.ApiManagement/service/backends

В [Microsoft.ApiManagement/service/backends](/azure/templates/microsoft.apimanagement/service/backends) описана серверную службу, к которой перенаправляется трафик.

Для серверных систем Service Fabric роль серверной службы выполняет сам кластер Service Fabric, а не определенная служба Service Fabric. Это позволяет направлять политике запросы не в одну службу в кластере, а в несколько. Поле **url** здесь представляет полное доменное имя службы в кластере. По нему по умолчанию направляются все запросы, для которых во внутренней политике не указано имя службы. Вы можете использовать фиктивное имя службы, например "fabric:/fake/service", если вам не нужна резервная служба. **resourceId** определяет конечную точку управления кластером.  **clientCertificateThumbprint** и **serverCertificateThumbprints** определяют сертификаты, которые используются для аутентификации в кластере.

### <a name="microsoftapimanagementserviceproducts"></a>Microsoft.ApiManagement/service/products

При помощи [Microsoft.ApiManagement/service/products](/azure/templates/microsoft.apimanagement/service/products) создается продукт. В службе управления API Azure продукт содержит один или несколько API вместе с квотой и условиями использования. Как только продукт будет опубликован, разработчики могут подписаться на него и начать использовать его интерфейсы API.

Введите для продукта описательное имя **displayName** и описание **description**. Для этой статьи вам нужна подписка, но ее не обязательно подтверждать у администратора.  Для этого продукта **состояние** имеет значение "опубликовано". Это значит, что он доступен для подписчиков.

### <a name="microsoftapimanagementserviceapis"></a>Microsoft.ApiManagement/service/apis

При помощи [Microsoft.ApiManagement/service/apis](/azure/templates/microsoft.apimanagement/service/apis) создается API-интерфейс. API в API Management представляет набор операций, которые могут быть вызваны клиентскими приложениями. После добавления операций интерфейс API добавляется к продукту и может быть опубликован. После публикации API на него могут подписаться разработчики для дальнейшего использования.

* Для параметра **displayName** можно указать любое имя API-интерфейса. Для этой статьи используйте имя Service Fabric App.
* Параметр **name** содержит уникальное описательное имя для API-интерфейса, например service-fabric-app. Оно выводится на порталах разработчика и издателя.
* Параметр **serviceUrl** определяет HTTP-службу, которая реализует API-интерфейс. Портал управления API направит запросы по этому адресу. Для серверных систем Service Fabric это значение URL-адреса не используется. Здесь вы можете использовать любое значение. Для этой статьи, например "http: \/ /сервицефабрик".
* Значение **path** добавляется к основному URL-адресу вашей службы управления API. Основной URL-адрес является общим для всех интерфейсов API, размещенных в экземпляре службы API Management. API Management отличает интерфейсы API по их суффиксу. Следовательно, суффикс должен быть уникальным для каждого API для заданного издателя.
* Параметр **protocols** определяет, какие протоколы можно использовать для доступа к API. Для этой статьи укажите протоколы **http** и **https**.
* Параметр **path** содержит суффикс для API-интерфейса. Для этой статьи используйте суффикс myapp.

### <a name="microsoftapimanagementserviceapisoperations"></a>Microsoft.ApiManagement/service/apis/operations

[Microsoft.ApiManagement/service/apis/operations](/azure/templates/microsoft.apimanagement/service/apis/operations) содержит операции, которые нужно добавить в API-интерфейс, прежде чем служба управления API сможет его использовать.  Внешние клиенты могут использовать операции для взаимодействия со службой ASP.NET Core без отслеживания состояния, которая выполняется в кластере Service Fabric.

Чтобы добавить операцию к внешнему API-интерфейсу, укажите следующие значения:

* **displayName** (отображаемое имя) и **description** (описание) для идентификации операции. Для этой статьи используйте Values.
* **method** (метод) — определяет команду HTTP.  В этой статье укажите **GET**.
* Шаблон URL-адреса **urlTemplate**, добавляемый к базовому URL-адресу API-интерфейса, идентифицирует конкретную HTTP-операцию.  В этой статье укажите `/api/values`, если вы добавили серверную службу .NET, или `getMessage`, если вы добавили серверную службу Java.  По умолчанию указанный здесь URL-адрес представляет собой URL-адрес, отправленный во внутреннюю службу Service Fabric. Если здесь вы используете тот же URL-адрес, который использует служба (/api/values), операция будет выполняться без дополнительных настроек. Вы также можете указать здесь URL-адрес, отличный от URL-адреса, используемого внутренней службой Service Fabric. В этом случае следует настроить перезапись пути в политике работы.

### <a name="microsoftapimanagementserviceapispolicies"></a>Microsoft.ApiManagement/service/apis/policies

При помощи [Microsoft.ApiManagement/service/apis/policies](/azure/templates/microsoft.apimanagement/service/apis/policies) создается политика серверной части, которая объединяет все данные. Она требуется при настройке внутренней службы Service Fabric, в которую направляются запросы. Вы можете применить эту политику к любой API-операции.  Дополнительные сведения см. в [обзоре политик](../api-management/api-management-howto-policies.md).

[Конфигурация серверной части для Service Fabric](../api-management/api-management-transformation-policies.md#SetBackendService) предоставляет следующие элементы управления маршрутизации запросов:

* Выбор экземпляра службы с помощью указания имени экземпляра службы Service Fabric. Оно может быть жестко запрограммированным (например, `"fabric:/myapp/myservice"`) или созданным с помощью HTTP-запроса (например, `"fabric:/myapp/users/" + context.Request.MatchedParameters["name"]`).
* Разрешение раздела путем создания ключа секции с помощью любой схемы секционирования Service Fabric.
* Выбор реплики для служб с отслеживанием состояния.
* Условия, при которых разрешение выполняется повторно, что позволяет указать условия для повторного разрешения расположения службы и повторной отправки запроса.

**policyContent** содержит политику в JSON-сериализованном формате XML.  В рамках этой статьи создайте внутреннюю политику, направляющую запросы напрямую в службу .NET или Java без отслеживания состояния, которую вы развернули ранее. Добавьте политику `set-backend-service` в узел входящих политик.  Замените значение *sf-service-instance-name* значением `fabric:/ApiApplication/WebApiService`, если вы развернули серверную службу .NET, или значением `fabric:/EchoServerApplication/EchoServerService`, если вы используете службу Java.  *backend-id* ссылается на внутренний ресурс, в нашем примере это ресурс `Microsoft.ApiManagement/service/backends`, определенный в шаблоне *apim.json*. *backend-id* может также ссылаться на другой внутренний ресурс, созданный с помощью API-интерфейсов управления API. В этой статье для *backend-id* присваивается значение параметра *service_fabric_backend_name*.

```xml
<policies>
  <inbound>
    <base/>
    <set-backend-service
        backend-id="servicefabric"
        sf-service-instance-name="service-name"
        sf-resolve-condition="@(context.LastError?.Reason == "BackendConnectionFailure")" />
  </inbound>
  <backend>
    <base/>
  </backend>
  <outbound>
    <base/>
  </outbound>
</policies>
```

Чтобы ознакомиться с полным набором атрибутов серверной политики Service Fabric, см. дополнительные сведения в разделе [Задание внутренней службы](../api-management/api-management-transformation-policies.md#SetBackendService).

## <a name="set-parameters-and-deploy-api-management"></a>Настройка параметров и развертывание управления API

Для описанных ниже параметров без значений в файле *cluster.parameters.json* укажите сведения о своем развертывании.

|Параметр|Значение|
|---|---|
|apimInstanceName|sf-apim|
|apimPublisherEmail|myemail@contosos.com|
|apimSku|Разработчик|
|serviceFabricCertificateName|sfclustertutorialgroup320171031144217|
|certificatePassword|q6D7nN%6ck@6|
|serviceFabricCertificateThumbprint|C4C1E541AD512B8065280292A8BA6079C3F26F10 |
|serviceFabricCertificate|&lt;строка в формате base-64&gt;|
|url_path|/api/values|
|clusterHttpManagementEndpoint|`https://mysfcluster.southcentralus.cloudapp.azure.com:19080`|
|inbound_policy|&lt;строка XML&gt;|

Значения *certificatePassword* и *serviceFabricCertificateThumbprint* должны соответствовать сертификату кластера, который вы использовали для настройки кластера.

*serviceFabricCertificate* содержит сертификат в формате строки base-64, которую можно создать с помощью следующего скрипта:

```powershell
$bytes = [System.IO.File]::ReadAllBytes("C:\mycertificates\sfclustertutorialgroup220171109113527.pfx");
$b64 = [System.Convert]::ToBase64String($bytes);
[System.Io.File]::WriteAllText("C:\mycertificates\sfclustertutorialgroup220171109113527.txt", $b64);
```

В узле *inbound_policy* замените значение *sf-service-instance-name* значением `fabric:/ApiApplication/WebApiService`, если вы развернули серверную службу .NET, или значением `fabric:/EchoServerApplication/EchoServerService`, если вы используете службу Java. *backend-id* ссылается на внутренний ресурс, в нашем примере это ресурс `Microsoft.ApiManagement/service/backends`, определенный в шаблоне *apim.json*. *backend-id* может также ссылаться на другой внутренний ресурс, созданный с помощью API-интерфейсов управления API. В этой статье для *backend-id* присваивается значение параметра *service_fabric_backend_name*.

```xml
<policies>
  <inbound>
    <base/>
    <set-backend-service
        backend-id="servicefabric"
        sf-service-instance-name="service-name"
        sf-resolve-condition="@(context.LastError?.Reason == "BackendConnectionFailure")" />
  </inbound>
  <backend>
    <base/>
  </backend>
  <outbound>
    <base/>
  </outbound>
</policies>
```

Используйте приведенный ниже сценарий, чтобы развернуть шаблон Resource Manager и файлы параметров для службы управления API.

```powershell
$groupname = "sfclustertutorialgroup"
$clusterloc="southcentralus"
$templatepath="C:\clustertemplates"

New-AzResourceGroupDeployment -ResourceGroupName $groupname -TemplateFile "$templatepath\network-apim.json" -TemplateParameterFile "$templatepath\network-apim.parameters.json" -Verbose

New-AzResourceGroupDeployment -ResourceGroupName $groupname -TemplateFile "$templatepath\apim.json" -TemplateParameterFile "$templatepath\apim.parameters.json" -Verbose
```

```azurecli
ResourceGroupName="sfclustertutorialgroup"
az deployment group create --name ApiMgmtNetworkDeployment --resource-group $ResourceGroupName --template-file network-apim.json --parameters @network-apim.parameters.json

az deployment group create --name ApiMgmtDeployment --resource-group $ResourceGroupName --template-file apim.json --parameters @apim.parameters.json
```

## <a name="test-it"></a>Тестирование

Теперь попробуйте отправить запрос к внутренней службе Service Fabric через службу управления API непосредственно с [портала Azure](https://portal.azure.com).

 1. В службе управления API выберите **API**.
 2. В API **Service Fabric App**, созданном на предыдущих шагах, перейдите на вкладку **Тест** и выберите операцию **Values**.
 3. Нажмите кнопку **Отправка**, чтобы отправить тестовый запрос во внутреннюю службу.  Ответ HTTP будет выглядеть следующим образом:

    ```http
    HTTP/1.1 200 OK

    Transfer-Encoding: chunked

    Content-Type: application/json; charset=utf-8

    Vary: Origin

    Ocp-Apim-Trace-Location: https://apimgmtstodhwklpry2xgkdj.blob.core.windows.net/apiinspectorcontainer/PWSQOq_FCDjGcaI1rdMn8w2-2?sv=2015-07-08&sr=b&sig=MhQhzk%2FEKzE5odlLXRjyVsgzltWGF8OkNzAKaf0B1P0%3D&se=2018-01-28T01%3A04%3A44Z&sp=r&traceId=9f8f1892121e445ea1ae4d2bc8449ce4

    Date: Sat, 27 Jan 2018 01:04:44 GMT


    ["value1", "value2"]
    ```

## <a name="clean-up-resources"></a>Очистка ресурсов

Помимо собственных ресурсов кластер содержит другие ресурсы Azure. Чтобы удалить кластер и все ресурсы, который он использует, проще всего удалить группу ресурсов.

Войдите в Azure и выберите идентификатор подписки, в которой вы хотите удалить кластер.  Идентификатор подписки можно найти, войдя в [портал Azure](https://portal.azure.com). Удалите группу ресурсов и все ресурсы кластера с помощью [командлета Remove-азресаурцеграуп](/powershell/module/az.resources/remove-azresourcegroup).

```powershell
$ResourceGroupName = "sfclustertutorialgroup"
Remove-AzResourceGroup -Name $ResourceGroupName -Force
```

```azurecli
ResourceGroupName="sfclustertutorialgroup"
az group delete --name $ResourceGroupName
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об использовании [управления API](../api-management/import-and-publish.md).

Вы также можете использовать [портал Azure](../api-management/how-to-configure-service-fabric-backend.md) для создания и управления Service Fabricми на концах для управления API.

[azure-powershell]: /powershell/azure/

[apim-arm]:https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/service-integration/apim.json
[apim-parameters-arm]:https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/service-integration/apim.parameters.json

[network-arm]: https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/service-integration/network-apim.json
[network-parameters-arm]: https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/service-integration/network-apim.parameters.json

<!-- pics -->
[sf-apim-topology-overview]: ./media/service-fabric-tutorial-deploy-api-management/sf-apim-topology-overview.png

<!-- pics -->
[sf-apim-topology-overview]: ./media/service-fabric-tutorial-deploy-api-management/sf-apim-topology-overview.png