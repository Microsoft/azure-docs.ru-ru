---
title: Просмотр агрегированной работоспособности сущностей Azure Service Fabric
description: Описание того, как отправлять запросы, просматривать и оценивать сводные данные о работоспособности сущностей Azure Service Fabric при помощи запросов о работоспособности и общих запросов.
ms.topic: conceptual
ms.date: 2/28/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 261343537b6d18c98c27c9cc9453d2a64f064216
ms.sourcegitcommit: a9ce1da049c019c86063acf442bb13f5a0dde213
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/27/2021
ms.locfileid: "105628073"
---
# <a name="view-service-fabric-health-reports"></a>Просмотр отчетов о работоспособности Service Fabric
В платформе Azure Service Fabric используется [модель работоспособности](service-fabric-health-introduction.md) с сущностями работоспособности, на основе которых компоненты системы и модули наблюдения создают отчеты о состоянии отслеживаемых локальных условий. [Хранилище данных о работоспособности](service-fabric-health-introduction.md#health-store) содержит все данные о работоспособности, с помощью которых можно определить состояние работоспособности сущностей.

Кластер автоматически заполняется отчетами о работоспособности, отправляемыми компонентами системы. Подробнее об этом см. в статье [Использование отчетов о работоспособности системы для устранения неполадок](service-fabric-understand-and-troubleshoot-with-system-health-reports.md).

В Service Fabric доступны несколько способов получения сводных данных о работоспособности сущностей:

* [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) или другие средства визуализации
* запросы работоспособности (с помощью PowerShell, API или REST);
* общие запросы, возвращающие перечень сущностей, среди свойств которых есть работоспособность (с помощью PowerShell, API или REST).

Для демонстрации этих параметров мы будем использовать локальный кластер с пятью узлами и [приложение fabric:/WordCount](https://github.com/Azure-Samples/service-fabric-wordcount/raw/master/WordCountV1.sfpkg). Приложение **fabric:/WordCount** содержит две службы по умолчанию, службу с отслеживанием состояния типа `WordCountServiceType` и службу без отслеживания состояния типа `WordCountWebServiceType`. Мной был изменен файл `ApplicationManifest.xml`, чтобы потребовать семь целевых реплик для службы с отслеживанием состояния и одной секции. Так как в кластере только пять узлов, компоненты системы выдадут предупреждение о секции службы, ведь она не соответствует целевому количеству.

```xml
<Service Name="WordCountService">
  <StatefulService ServiceTypeName="WordCountServiceType" TargetReplicaSetSize="7" MinReplicaSetSize="2">
    <UniformInt64Partition PartitionCount="[WordCountService_PartitionCount]" LowKey="1" HighKey="26" />
  </StatefulService>
</Service>
```

## <a name="health-in-service-fabric-explorer"></a>Работоспособность в обозревателе Service Fabric
Обозреватель Service Fabric предоставляет визуальное представление кластера. На изображении ниже видно следующее:

* Приложение **fabric:/WordCount** выделено красным (ошибка), так как служба **MyWatchdog** вывела сообщение об ошибке при проверке свойства **Доступность**.
* Одна из его служб **fabric:/WordCount/WordCountService** выделена желтым (предупреждение). Для службы настроены семь реплик, а в кластере пять узлов, поэтому две реплики негде разместить. Хотя здесь это не показано, секция службы выделена желтым в соответствии с отчетом системы от `System.FM`, в котором сообщается, что `Partition is below target replica or instance count` (Секция меньше целевой реплики или числа экземпляров). Если раздел выделен желтым цветом, таким же цветом будет выделена и служба.
* Кластер выделен красным, так как приложение выделено тем же цветом.

При оценке используются политики по умолчанию из манифеста кластера и манифеста приложения. Это строгие политики, при которых не допускаются сбои.

Так выглядит кластер в обозревателе Service Fabric:

![Вид кластера в обозревателе Service Fabric.][1]

[1]: ./media/service-fabric-view-entities-aggregated-health/servicefabric-explorer-cluster-health.png


> [!NOTE]
> Дополнительные сведения об [обозревателе Service Fabric](service-fabric-visualizing-your-cluster.md).
>
>

## <a name="health-queries"></a>Запросы о работоспособности
Service Fabric предоставляет запросы о работоспособности по каждому поддерживаемому [типу сущностей](service-fabric-health-introduction.md#health-entities-and-hierarchy). Их можно просмотреть с помощью API (используя методы в [FabricClient.HealthManager](/dotnet/api/system.fabric.fabricclient.healthmanager)), командлетов PowerShell и REST. Эти запросы возвращают все сведения о работоспособности сущности: сводное состояние работоспособности, события работоспособности сущности, состояние работоспособности дочерних элементов (если применимо) и оценки неработоспособности (если сущность неработоспособна) и статистику работоспособности дочерних элементов (если применимо).

> [!NOTE]
> Сущность работоспособности возвращается после ее полного заполнения в хранилище данных о работоспособности. Сущность должна быть активной (ее не следует удалять) и у нее должен быть системный отчет. У ее родительских сущностей в цепи иерархии также должны быть системные отчеты. Если любое из этих условий не выполнено, запросы работоспособности будут возвращать исключение [FabricException](/dotnet/api/system.fabric.fabricexception) с кодом ошибки [FabricErrorCode](/dotnet/api/system.fabric.fabricerrorcode) `FabricHealthEntityNotFound`, из-за которой сущность не удалось возвратить.
>
>

Для запросов о работоспособности требуется передача идентификатора сущности, зависящего от ее типа. Запросы принимают необязательные параметры политики работоспособности. Если политики работоспособности не указаны, при оценке будут использоваться [политики работоспособности](service-fabric-health-introduction.md#health-policies) из манифеста кластера или приложения. Если манифесты не содержат определение политик работоспособности, то для оценки используется политика работоспособности по умолчанию. Политики работоспособности по умолчанию не допускают наличие каких-либо ошибок. Запросы также принимают фильтры, позволяя возвращать только частичные сведения о дочерних элементах или события, соответствующие условиям указанных фильтров. Еще один фильтр позволяет исключить статистику дочерних элементов.

> [!NOTE]
> На стороне сервера применяются выходные фильтры для сокращения размера ответных сообщений. Рекомендуем использовать выходные фильтры для ограничения объема возвращаемых данных, вместо того чтобы применять фильтры на стороне клиента.
>
>

Данные о работоспособности сущности содержат:

* Сводное состояние работоспособности сущности. Эти данные вычисляются в хранилище данных о работоспособности на основе отчетов о работоспособности сущности, состояния работоспособности дочерних элементов (в соответствующих случаях) и политик работоспособности. См. дополнительные сведения об [оценке работоспособности сущности](service-fabric-health-introduction.md#health-evaluation).  
* События работоспособности, связанные с сущностью.
* Коллекцию состояний работоспособности всех дочерних элементов сущностей, у которых есть такие элементы. В данных каждого состояния работоспособности содержится идентификатор сущности и сводное состояние работоспособности. Чтобы получить все сведения о работоспособности дочернего элемента, вызовите запрос о работоспособности для типа дочерней сущности и передайте идентификатор дочернего элемента.
* Оценки неработоспособности с данными об отчете, из-за которого активировано такое состояние сущности (если она неработоспособна). Оценка выполняется рекурсивно, с учетом оценки работоспособности дочерних элементов, которые активировали текущее состояние работоспособности. Например, модуль наблюдения сообщил об ошибке реплики. Состояние приложения оценивается как неработоспособное из-за неработоспособной службы. Служба неработоспособна из-за секции, указанной в сообщении об ошибке. Секция неработоспособна из-за реплики в сообщении об ошибке. Реплика неработоспособна из-за отчета о работоспособности модуля наблюдения, содержащего ошибку.
* Статистика работоспособности дочерних элементов для всех типов сущностей, у которых имеются дочерние элементы. Например, данные о работоспособности кластера содержат общее число приложений, служб, секций, реплик и развернутых сущностей в кластере. Данные о работоспособности службы содержат общее число секций и реплик в группе указанной службе.

## <a name="get-cluster-health"></a>Получение сведений о работоспособности кластера
Возвращает сведения о сущности кластера. В них содержатся данные о состоянии работоспособности приложений и узлов (дочерних элементов кластера). Входные данные:

* политика работоспособности кластера, используемая для оценки узлов и событий кластера (необязательный параметр);
* сопоставление политики работоспособности приложения с политиками работоспособности, которые используются для переопределения политик манифеста приложения (необязательный параметр);
* фильтры для событий, узлов и приложений, с помощью которых можно указать, какие записи представляют интерес и будут возвращены в результате, например только с ошибками или с предупреждениями и ошибками (необязательные параметры). При оценке сводного состояния работоспособности сущности используются все события, узлы и приложения, независимо от выбранного фильтра.
* [Необязательно.] Используйте фильтр, исключающий статистику работоспособности.
* [Необязательно.] Используйте фильтр, чтобы добавить в статистику работоспособности данные о fabric:/System. Применяется, только если статистика работоспособности не исключена. По умолчанию статистика работоспособности содержит только статистику приложений пользователей, а не приложения System.

### <a name="api"></a>API
Чтобы получить сведения о работоспособности кластера, создайте `FabricClient` и вызовите метод [GetClusterHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getclusterhealthasync) в **HealthManager**.

Следующий вызов получает сведения о работоспособности кластера.

```csharp
ClusterHealth clusterHealth = await fabricClient.HealthManager.GetClusterHealthAsync();
```

Следующий код получает сведения о работоспособности кластера, используя настраиваемую политику работоспособности кластера и фильтры для узлов и приложений. Он указывает, что статистика работоспособности должна содержать данные о fabric:/System. Он создает [ClusterHealthQueryDescription](/dotnet/api/system.fabric.description.clusterhealthquerydescription), где содержатся входные данные.

```csharp
var policy = new ClusterHealthPolicy()
{
    MaxPercentUnhealthyNodes = 20
};
var nodesFilter = new NodeHealthStatesFilter()
{
    HealthStateFilterValue = HealthStateFilter.Error | HealthStateFilter.Warning
};
var applicationsFilter = new ApplicationHealthStatesFilter()
{
    HealthStateFilterValue = HealthStateFilter.Error
};
var healthStatisticsFilter = new ClusterHealthStatisticsFilter()
{
    ExcludeHealthStatistics = false,
    IncludeSystemApplicationHealthStatistics = true
};
var queryDescription = new ClusterHealthQueryDescription()
{
    HealthPolicy = policy,
    ApplicationsFilter = applicationsFilter,
    NodesFilter = nodesFilter,
    HealthStatisticsFilter = healthStatisticsFilter
};

ClusterHealth clusterHealth = await fabricClient.HealthManager.GetClusterHealthAsync(queryDescription);
```

### <a name="powershell"></a>PowerShell
Командлет для получения сведений о работоспособности кластера — [Get-ServiceFabricClusterHealth](/powershell/module/servicefabric/get-servicefabricclusterhealth). Сначала подключитесь к кластеру с помощью командлета [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster) .

В этом примере кластер состоит из пяти узлов, а приложение системы и fabric:/WordCount настроены, как описано выше.

Следующий командлет возвращает сведения о работоспособности кластера, используя политики работоспособности по умолчанию. Сводное состояние работоспособности — "Предупреждение", так как приложение fabric:/WordCount находится в таком же состоянии. Обратите внимание, что в данных оценки неработоспособности содержатся подробные сведения об условиях, из-за которых активировано такое сводное состояние.

```xml
PS D:\ServiceFabric> Get-ServiceFabricClusterHealth


AggregatedHealthState   : Warning
UnhealthyEvaluations    : 
                          Unhealthy applications: 100% (1/1), MaxPercentUnhealthyApplications=0%.

                          Unhealthy application: ApplicationName='fabric:/WordCount', AggregatedHealthState='Warning'.

                            Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                            Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Warning'.

                                Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                Unhealthy partition: PartitionId='af2e3e44-a8f8-45ac-9f31-4093eb897600', AggregatedHealthState='Warning'.

                                    Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=false.


NodeHealthStates        : 
                          NodeName              : _Node_4
                          AggregatedHealthState : Ok

                          NodeName              : _Node_3
                          AggregatedHealthState : Ok

                          NodeName              : _Node_2
                          AggregatedHealthState : Ok

                          NodeName              : _Node_1
                          AggregatedHealthState : Ok

                          NodeName              : _Node_0
                          AggregatedHealthState : Ok

ApplicationHealthStates : 
                          ApplicationName       : fabric:/System
                          AggregatedHealthState : Ok

                          ApplicationName       : fabric:/WordCount
                          AggregatedHealthState : Warning

HealthEvents            : None
HealthStatistics        : 
                          Node                  : 5 Ok, 0 Warning, 0 Error
                          Replica               : 6 Ok, 0 Warning, 0 Error
                          Partition             : 1 Ok, 1 Warning, 0 Error
                          Service               : 1 Ok, 1 Warning, 0 Error
                          DeployedServicePackage : 6 Ok, 0 Warning, 0 Error
                          DeployedApplication   : 5 Ok, 0 Warning, 0 Error
                          Application           : 0 Ok, 1 Warning, 0 Error
```

Следующий командлет PowerShell возвращает сведения о работоспособности кластера, используя настраиваемую политику приложений. Он фильтрует результаты, позволяя возвращать только приложения и узлы с состоянием "Ошибка" или "Предупреждение". В результате ни один узел не будет возвращен, так как все узлы работоспособны. Только приложение fabric:/WordCount соответствует условиям фильтра приложений. Так как по условиям настраиваемой политики предупреждение для приложения fabric:/WordCount считается ошибкой, состояние приложения, как и кластера, оценивается как "Ошибка".

```powershell
PS D:\ServiceFabric> $appHealthPolicy = New-Object -TypeName System.Fabric.Health.ApplicationHealthPolicy
$appHealthPolicy.ConsiderWarningAsError = $true
$appHealthPolicyMap = New-Object -TypeName System.Fabric.Health.ApplicationHealthPolicyMap
$appUri1 = New-Object -TypeName System.Uri -ArgumentList "fabric:/WordCount"
$appHealthPolicyMap.Add($appUri1, $appHealthPolicy)
Get-ServiceFabricClusterHealth -ApplicationHealthPolicyMap $appHealthPolicyMap -ApplicationsFilter "Warning,Error" -NodesFilter "Warning,Error" -ExcludeHealthStatistics


AggregatedHealthState   : Error
UnhealthyEvaluations    : 
                          Unhealthy applications: 100% (1/1), MaxPercentUnhealthyApplications=0%.

                          Unhealthy application: ApplicationName='fabric:/WordCount', AggregatedHealthState='Error'.

                            Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                            Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Error'.

                                Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                Unhealthy partition: PartitionId='af2e3e44-a8f8-45ac-9f31-4093eb897600', AggregatedHealthState='Error'.

                                    Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=true.


NodeHealthStates        : None
ApplicationHealthStates : 
                          ApplicationName       : fabric:/WordCount
                          AggregatedHealthState : Error

HealthEvents            : None
```

### <a name="rest"></a>REST
Вы можете получить сведения о работоспособности кластера с помощью [запроса GET](/rest/api/servicefabric/get-the-health-of-a-cluster) или [запроса POST](/rest/api/servicefabric/get-the-health-of-a-cluster-by-using-a-health-policy), в который входят политики работоспособности, описанные в основном тексте.

## <a name="get-node-health"></a>Получение сведений о работоспособности узла
Возвращает сведения о работоспособности сущности узла и содержит события работоспособности, связанные с узлом. Входные данные:

* имя узла, определяющее узел (обязательный параметр);
* параметры политики работоспособности кластера, используемые для оценки работоспособности (необязательные параметры);
* фильтры для событий, с помощью которых можно указать, какие записи представляют интерес и будут возвращены в результате, например только с ошибками или с предупреждениями и ошибками (необязательные параметры). При оценке сводного состояния работоспособности сущности используются все события, независимо от выбранного фильтра.

### <a name="api"></a>API
Чтобы получить сведения о работоспособности узла через API, создайте `FabricClient` и вызовите метод [GetNodeHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getnodehealthasync) в HealthManager.

Следующий код получает сведения о работоспособности узла по указанному имени.

```csharp
NodeHealth nodeHealth = await fabricClient.HealthManager.GetNodeHealthAsync(nodeName);
```

Следующий код получает сведения о работоспособности узла по указанному имени и передает фильтр событий и настраиваемую политику через [NodeHealthQueryDescription](/dotnet/api/system.fabric.description.nodehealthquerydescription).

```csharp
var queryDescription = new NodeHealthQueryDescription(nodeName)
{
    HealthPolicy = new ClusterHealthPolicy() {  ConsiderWarningAsError = true },
    EventsFilter = new HealthEventsFilter() { HealthStateFilterValue = HealthStateFilter.Warning },
};

NodeHealth nodeHealth = await fabricClient.HealthManager.GetNodeHealthAsync(queryDescription);
```

### <a name="powershell"></a>PowerShell
Командлет для получения сведений о работоспособности узла — [Get-ServiceFabricNodeHealth](/powershell/module/servicefabric/get-servicefabricnodehealth). Сначала подключитесь к кластеру с помощью командлета [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster) .
Следующий командлет возвращает сведения о работоспособности узла, используя политики работоспособности по умолчанию:

```powershell
PS D:\ServiceFabric> Get-ServiceFabricNodeHealth _Node_1


NodeName              : _Node_1
AggregatedHealthState : Ok
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 3
                        SentAt                : 7/13/2017 4:39:23 PM
                        ReceivedAt            : 7/13/2017 4:40:47 PM
                        TTL                   : Infinite
                        Description           : Fabric node is up.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/13/2017 4:40:47 PM, LastWarning = 1/1/0001 12:00:00 AM
```

Следующий командлет возвращает сведения о работоспособности всех узлов в кластере:

```powershell
PS D:\ServiceFabric> Get-ServiceFabricNode | Get-ServiceFabricNodeHealth | select NodeName, AggregatedHealthState | ft -AutoSize

NodeName AggregatedHealthState
-------- ---------------------
_Node_4                     Ok
_Node_3                     Ok
_Node_2                     Ok
_Node_1                     Ok
_Node_0                     Ok
```

### <a name="rest"></a>REST
Вы можете получить сведения о работоспособности узла с помощью [запроса GET](/rest/api/servicefabric/get-the-health-of-a-node) или [запроса POST](/rest/api/servicefabric/get-the-health-of-a-node-by-using-a-health-policy), в который входят политики работоспособности, описанные в основном тексте.

## <a name="get-application-health"></a>Получение сведений о работоспособности приложения
Возвращает сведения о работоспособности сущности приложения. В них содержатся данные о состоянии работоспособности развернутого приложения и дочерних элементов службы. Входные данные:

* имя приложения (URI), определяющее приложение (обязательный параметр);
* политика работоспособности приложения, используемая для переопределения политик манифеста приложения (необязательный параметр);
* фильтры для событий, служб и развернутых приложений, с помощью которых можно указать, какие записи представляют интерес и будут возвращены в результате, например только с ошибками или с предупреждениями и ошибками (необязательные параметры). При оценке сводного состояния работоспособности сущности используются все события, службы и развернутые приложения, независимо от выбранного фильтра.
* [Необязательно.] Используйте фильтр, исключающий статистику работоспособности. Если не он указан, то статистика работоспособности будет содержать количественные данные об исправности, предупреждениях и ошибках для всех дочерних элементов приложения: служб, секций, реплик, развернутых приложений и развернутых пакетов служб.

### <a name="api"></a>API
Чтобы получить сведения о работоспособности приложения, создайте `FabricClient` и вызовите метод [GetApplicationHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getapplicationhealthasync) в HealthManager.

Следующий код получает сведения о работоспособности приложения по указанному имени (URI).

```csharp
ApplicationHealth applicationHealth = await fabricClient.HealthManager.GetApplicationHealthAsync(applicationName);
```

Следующий код получает сведения о работоспособности приложения по заданному имени (URI) с указанием фильтров и настраиваемых политик через [ApplicationHealthQueryDescription](/dotnet/api/system.fabric.description.applicationhealthquerydescription).

```csharp
HealthStateFilter warningAndErrors = HealthStateFilter.Error | HealthStateFilter.Warning;
var serviceTypePolicy = new ServiceTypeHealthPolicy()
{
    MaxPercentUnhealthyPartitionsPerService = 0,
    MaxPercentUnhealthyReplicasPerPartition = 5,
    MaxPercentUnhealthyServices = 0,
};
var policy = new ApplicationHealthPolicy()
{
    ConsiderWarningAsError = false,
    DefaultServiceTypeHealthPolicy = serviceTypePolicy,
    MaxPercentUnhealthyDeployedApplications = 0,
};

var queryDescription = new ApplicationHealthQueryDescription(applicationName)
{
    HealthPolicy = policy,
    EventsFilter = new HealthEventsFilter() { HealthStateFilterValue = warningAndErrors },
    ServicesFilter = new ServiceHealthStatesFilter() { HealthStateFilterValue = warningAndErrors },
    DeployedApplicationsFilter = new DeployedApplicationHealthStatesFilter() { HealthStateFilterValue = warningAndErrors },
};

ApplicationHealth applicationHealth = await fabricClient.HealthManager.GetApplicationHealthAsync(queryDescription);
```

### <a name="powershell"></a>PowerShell
Командлет для получения сведений о работоспособности приложения — [Get-ServiceFabricApplicationHealth](/powershell/module/servicefabric/get-servicefabricapplicationhealth). Сначала подключитесь к кластеру с помощью командлета [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster) .

Следующий командлет получает сведения о работоспособности приложения **fabric:/WordCount** .

```powershell
PS D:\ServiceFabric> Get-ServiceFabricApplicationHealth fabric:/WordCount


ApplicationName                 : fabric:/WordCount
AggregatedHealthState           : Warning
UnhealthyEvaluations            : 
                                  Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                                  Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Warning'.

                                    Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                    Unhealthy partition: PartitionId='af2e3e44-a8f8-45ac-9f31-4093eb897600', AggregatedHealthState='Warning'.

                                        Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=false.

ServiceHealthStates             : 
                                  ServiceName           : fabric:/WordCount/WordCountWebService
                                  AggregatedHealthState : Ok

                                  ServiceName           : fabric:/WordCount/WordCountService
                                  AggregatedHealthState : Warning

DeployedApplicationHealthStates : 
                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_4
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_3
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_0
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_2
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_1
                                  AggregatedHealthState : Ok

HealthEvents                    : 
                                  SourceId              : System.CM
                                  Property              : State
                                  HealthState           : Ok
                                  SequenceNumber        : 282
                                  SentAt                : 7/13/2017 5:57:05 PM
                                  ReceivedAt            : 7/13/2017 5:57:05 PM
                                  TTL                   : Infinite
                                  Description           : Application has been created.
                                  RemoveWhenExpired     : False
                                  IsExpired             : False
                                  Transitions           : Error->Ok = 7/13/2017 5:57:05 PM, LastWarning = 1/1/0001 12:00:00 AM

HealthStatistics                : 
                                  Replica               : 6 Ok, 0 Warning, 0 Error
                                  Partition             : 1 Ok, 1 Warning, 0 Error
                                  Service               : 1 Ok, 1 Warning, 0 Error
                                  DeployedServicePackage : 6 Ok, 0 Warning, 0 Error
                                  DeployedApplication   : 5 Ok, 0 Warning, 0 Error
```

Следующий командлет PowerShell передает настраиваемые политики. Кроме того, он выполняет фильтрацию дочерних элементов и событий.

```powershell
PS D:\ServiceFabric> Get-ServiceFabricApplicationHealth -ApplicationName fabric:/WordCount -ConsiderWarningAsError $true -ServicesFilter Error -EventsFilter Error -DeployedApplicationsFilter Error -ExcludeHealthStatistics


ApplicationName                 : fabric:/WordCount
AggregatedHealthState           : Error
UnhealthyEvaluations            : 
                                  Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                                  Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Error'.

                                    Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                    Unhealthy partition: PartitionId='af2e3e44-a8f8-45ac-9f31-4093eb897600', AggregatedHealthState='Error'.

                                        Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=true.

ServiceHealthStates             : 
                                  ServiceName           : fabric:/WordCount/WordCountService
                                  AggregatedHealthState : Error

DeployedApplicationHealthStates : None
HealthEvents                    : None
```

### <a name="rest"></a>REST
Вы можете получить сведения о работоспособности приложения с помощью [запроса GET](/rest/api/servicefabric/get-the-health-of-an-application) или [запроса POST](/rest/api/servicefabric/get-the-health-of-an-application-by-using-an-application-health-policy), в который входят политики работоспособности, описанные в основном тексте.

## <a name="get-service-health"></a>Получение сведений о работоспособности службы
Возвращает сведения о работоспособности сущности службы. В них содержатся данные о состоянии работоспособности разделов. Входные данные:

* имя службы (URI), определяющее службу (обязательный параметр);
* политика работоспособности приложения, используемая для переопределения политики манифеста приложения (необязательный параметр);
* фильтры событий и разделов, с помощью которых можно указать, какие записи представляют интерес и будут возвращены в результате, например только с ошибками или с предупреждениями и ошибками (необязательные параметры). При оценке сводного состояния работоспособности сущности используются все события и разделы, независимо от выбранного фильтра.
* [Необязательно.] Используйте фильтр, исключающий статистику работоспособности. Если он не указан, то статистика работоспособности содержит количественные данные об исправности, предупреждениях и ошибках для всех секций и реплик службы.

### <a name="api"></a>API
Чтобы получить сведения о работоспособности службы через API, создайте `FabricClient` и вызовите метод [GetServiceHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getservicehealthasync) в HealthManager.

В следующем примере возвращаются сведения о работоспособности службы с указанным именем (URI):

```csharp
ServiceHealth serviceHealth = await fabricClient.HealthManager.GetServiceHealthAsync(serviceName);
```

Следующий код получает сведения о работоспособности службы по указанному имени, то есть универсальному коду ресурса (URI) службы, с указанием фильтров и настраиваемой политики через [ServiceHealthQueryDescription](/dotnet/api/system.fabric.description.servicehealthquerydescription).

```csharp
var queryDescription = new ServiceHealthQueryDescription(serviceName)
{
    EventsFilter = new HealthEventsFilter() { HealthStateFilterValue = HealthStateFilter.All },
    PartitionsFilter = new PartitionHealthStatesFilter() { HealthStateFilterValue = HealthStateFilter.Error },
};

ServiceHealth serviceHealth = await fabricClient.HealthManager.GetServiceHealthAsync(queryDescription);
```

### <a name="powershell"></a>PowerShell
Командлет для получения сведений о работоспособности службы — [Get-ServiceFabricServiceHealth](/powershell/module/servicefabric/get-servicefabricservicehealth). Сначала подключитесь к кластеру с помощью командлета [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster) .

Следующий командлет возвращает сведения о работоспособности службы, используя политику работоспособности по умолчанию:

```powershell
PS D:\ServiceFabric> Get-ServiceFabricServiceHealth -ServiceName fabric:/WordCount/WordCountService


ServiceName           : fabric:/WordCount/WordCountService
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                        Unhealthy partition: PartitionId='af2e3e44-a8f8-45ac-9f31-4093eb897600', AggregatedHealthState='Warning'.

                            Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=false.

PartitionHealthStates : 
                        PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
                        AggregatedHealthState : Warning

HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 15
                        SentAt                : 7/13/2017 5:57:05 PM
                        ReceivedAt            : 7/13/2017 5:57:18 PM
                        TTL                   : Infinite
                        Description           : Service has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM

HealthStatistics      : 
                        Replica               : 5 Ok, 0 Warning, 0 Error
                        Partition             : 0 Ok, 1 Warning, 0 Error
```

### <a name="rest"></a>REST
Вы можете получить сведения о работоспособности службы с помощью [запроса GET](/rest/api/servicefabric/get-the-health-of-a-service) или [запроса POST](/rest/api/servicefabric/get-the-health-of-a-service-by-using-a-health-policy), в который входят политики работоспособности, описанные в основном тексте.

## <a name="get-partition-health"></a>Получение сведений о работоспособности раздела
Возвращает сведения о работоспособности сущности раздела. В них содержатся данные о состоянии работоспособности реплик. Входные данные:

* идентификатор раздела (GUID), определяющий раздел (обязательный параметр);
* политика работоспособности приложения, используемая для переопределения политики манифеста приложения (необязательный параметр);
* фильтры для событий и реплик, с помощью которых можно указать, какие записи представляют интерес и будут возвращены в результате, например только с ошибками или с предупреждениями и ошибками (необязательные параметры). При оценке сводного состояния работоспособности сущности используются все события и реплики, независимо от выбранного фильтра.
* [Необязательно.] Используйте фильтр, исключающий статистику работоспособности. Если он не указан, то статистика работоспособности показывает, сколько реплик находится в исправном состояния, состоянии предупреждения и состоянии ошибки.

### <a name="api"></a>API
Чтобы получить сведения о работоспособности раздела через API, создайте `FabricClient` и вызовите метод [GetPartitionHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getpartitionhealthasync) в HealthManager. Чтобы задать необязательные параметры, создайте [PartitionHealthQueryDescription](/dotnet/api/system.fabric.description.partitionhealthquerydescription).

```csharp
PartitionHealth partitionHealth = await fabricClient.HealthManager.GetPartitionHealthAsync(partitionId);
```

### <a name="powershell"></a>PowerShell
Командлет для получения сведений о работоспособности раздела — [Get-ServiceFabricPartitionHealth](/powershell/module/servicefabric/get-servicefabricpartitionhealth). Сначала подключитесь к кластеру с помощью командлета [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster) .

Следующий командлет позволяет получить сведения о работоспособности всех секций службы **fabric:/WordCount/WordCountService** и отфильтровать состояния работоспособности реплик.

```powershell
PS D:\ServiceFabric> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricPartitionHealth -ReplicasFilter None

PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=false.

ReplicaHealthStates   : None
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Warning
                        SequenceNumber        : 72
                        SentAt                : 7/13/2017 5:57:29 PM
                        ReceivedAt            : 7/13/2017 5:57:48 PM
                        TTL                   : Infinite
                        Description           : Partition is below target replica or instance count.
                        fabric:/WordCount/WordCountService 7 2 af2e3e44-a8f8-45ac-9f31-4093eb897600
                          N/P RD _Node_2 Up 131444422260002646
                          N/S RD _Node_4 Up 131444422293113678
                          N/S RD _Node_3 Up 131444422293113679
                          N/S RD _Node_1 Up 131444422293118720
                          N/S RD _Node_0 Up 131444422293118721
                          (Showing 5 out of 5 replicas. Total available replicas: 5.)

                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Ok->Warning = 7/13/2017 5:57:48 PM, LastError = 1/1/0001 12:00:00 AM

                        SourceId              : System.PLB
                        Property              : ServiceReplicaUnplacedHealth_Secondary_af2e3e44-a8f8-45ac-9f31-4093eb897600
                        HealthState           : Warning
                        SequenceNumber        : 131444445174851664
                        SentAt                : 7/13/2017 6:35:17 PM
                        ReceivedAt            : 7/13/2017 6:35:18 PM
                        TTL                   : 00:01:05
                        Description           : The Load Balancer was unable to find a placement for one or more of the Service's Replicas:
                        Secondary replica could not be placed due to the following constraints and properties:  
                        TargetReplicaSetSize: 7
                        Placement Constraint: N/A
                        Parent Service: N/A

                        Constraint Elimination Sequence:
                        Existing Secondary Replicas eliminated 4 possible node(s) for placement -- 1/5 node(s) remain.
                        Existing Primary Replica eliminated 1 possible node(s) for placement -- 0/5 node(s) remain.

                        Nodes Eliminated By Constraints:

                        Existing Secondary Replicas -- Nodes with Partition's Existing Secondary Replicas/Instances:
                        --
                        FaultDomain:fd:/4 NodeName:_Node_4 NodeType:NodeType4 UpgradeDomain:4 UpgradeDomain: ud:/4 Deactivation Intent/Status: None/None
                        FaultDomain:fd:/3 NodeName:_Node_3 NodeType:NodeType3 UpgradeDomain:3 UpgradeDomain: ud:/3 Deactivation Intent/Status: None/None
                        FaultDomain:fd:/1 NodeName:_Node_1 NodeType:NodeType1 UpgradeDomain:1 UpgradeDomain: ud:/1 Deactivation Intent/Status: None/None
                        FaultDomain:fd:/0 NodeName:_Node_0 NodeType:NodeType0 UpgradeDomain:0 UpgradeDomain: ud:/0 Deactivation Intent/Status: None/None

                        Existing Primary Replica -- Nodes with Partition's Existing Primary Replica or Secondary Replicas:
                        --
                        FaultDomain:fd:/2 NodeName:_Node_2 NodeType:NodeType2 UpgradeDomain:2 UpgradeDomain: ud:/2 Deactivation Intent/Status: None/None


                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : Error->Warning = 7/13/2017 5:57:48 PM, LastOk = 1/1/0001 12:00:00 AM

HealthStatistics      : 
                        Replica               : 5 Ok, 0 Warning, 0 Error
```

### <a name="rest"></a>REST
Вы можете получить сведения о работоспособности раздела с помощью [запроса GET](/rest/api/servicefabric/sfclient-api-getpartitionhealth) или [запроса POST](/rest/api/servicefabric/get-the-health-of-a-partition-by-using-a-health-policy), в который входят политики работоспособности, описанные в основном тексте.

## <a name="get-replica-health"></a>Получение сведений о работоспособности реплики
Возвращает сведения о работоспособности реплики службы с отслеживанием состояния или экземпляра службы без отслеживания состояния. Входные данные:

* идентификатор раздела (GUID) и идентификатор реплики, определяющий реплику (обязательные параметры);
* параметры политики работоспособности приложения, используемые для переопределения политик манифеста приложения (необязательные параметры);
* фильтры для событий, с помощью которых можно указать, какие записи представляют интерес и будут возвращены в результате, например только с ошибками или с предупреждениями и ошибками (необязательные параметры). При оценке сводного состояния работоспособности сущности используются все события, независимо от выбранного фильтра.

### <a name="api"></a>API
Чтобы получить сведения о работоспособности реплики через API, создайте `FabricClient` и вызовите метод [GetReplicaHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getreplicahealthasync) в HealthManager. Чтобы указать дополнительные параметры, используйте [ReplicaHealthQueryDescription](/dotnet/api/system.fabric.description.replicahealthquerydescription).

```csharp
ReplicaHealth replicaHealth = await fabricClient.HealthManager.GetReplicaHealthAsync(partitionId, replicaId);
```

### <a name="powershell"></a>PowerShell
Командлет для получения сведений о работоспособности реплики — [Get-ServiceFabricReplicaHealth](/powershell/module/servicefabric/get-servicefabricreplicahealth). Сначала подключитесь к кластеру с помощью командлета [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster) .

Следующий командлет возвращает сведения о работоспособности первичной реплики для всех разделов службы:

```powershell
PS D:\ServiceFabric> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricReplica | where {$_.ReplicaRole -eq "Primary"} | Get-ServiceFabricReplicaHealth


PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
ReplicaId             : 131444422260002646
AggregatedHealthState : Ok
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 131444422263668344
                        SentAt                : 7/13/2017 5:57:06 PM
                        ReceivedAt            : 7/13/2017 5:57:18 PM
                        TTL                   : Infinite
                        Description           : Replica has been created._Node_2
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="rest"></a>REST
Вы можете получить сведения о работоспособности реплики с помощью [запроса GET](/rest/api/servicefabric/get-the-health-of-a-replica) или [запроса POST](/rest/api/servicefabric/get-the-health-of-a-replica-by-using-a-health-policy), в который входят политики работоспособности, описанные в основном тексте.

## <a name="get-deployed-application-health"></a>Получение сведений о работоспособности развернутого приложения
Возвращает сведения о работоспособности приложения, развернутого на сущности узла. В них содержатся данные о состоянии работоспособности развернутых пакетов служб. Входные данные:

* имя приложения (URI) и имя узла (строка), определяющие развернутое приложение (обязательные параметры);
* политика работоспособности приложения, используемая для переопределения политик манифеста приложения (необязательный параметр);
* фильтры для событий и развернутых пакетов служб, с помощью которых можно указать, какие записи представляют интерес и будут возвращены в результате, например только с ошибками или с предупреждениями и ошибками (необязательные параметры). При оценке сводного состояния работоспособности сущности используются все события и развернутые пакеты служб, независимо от выбранного фильтра.
* [Необязательно.] Используйте фильтр, исключающий статистику работоспособности. Если он не указан, то статистика работоспособности показывает, сколько развернутых пакетов служб находится в исправном состояния, состоянии предупреждения и состоянии ошибки.

### <a name="api"></a>API
Чтобы получить сведения о работоспособности приложения, развернутого на узле, через API, создайте `FabricClient` и вызовите метод [GetDeployedApplicationHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getdeployedapplicationhealthasync) в HealthManager. Чтобы задать необязательные параметры, используйте [DeployedApplicationHealthQueryDescription](/dotnet/api/system.fabric.description.deployedapplicationhealthquerydescription).

```csharp
DeployedApplicationHealth health = await fabricClient.HealthManager.GetDeployedApplicationHealthAsync(
    new DeployedApplicationHealthQueryDescription(applicationName, nodeName));
```

### <a name="powershell"></a>PowerShell
Командлет для получения сведений о работоспособности развернутых приложений — [Get-ServiceFabricDeployedApplicationHealth](/powershell/module/servicefabric/get-servicefabricdeployedapplicationhealth). Сначала подключитесь к кластеру с помощью командлета [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster) . Чтобы узнать, где развернуто приложение, выполните командлет [Get-ServiceFabricApplicationHealth](/powershell/module/servicefabric/get-servicefabricapplicationhealth) и просмотрите дочерние элементы развернутого приложения.

Следующий командлет возвращает сведения о работоспособности приложения **fabric:/WordCount**, развернутого на узле **_Node_2**.

```powershell
PS D:\ServiceFabric> Get-ServiceFabricDeployedApplicationHealth -ApplicationName fabric:/WordCount -NodeName _Node_0


ApplicationName                    : fabric:/WordCount
NodeName                           : _Node_0
AggregatedHealthState              : Ok
DeployedServicePackageHealthStates : 
                                     ServiceManifestName   : WordCountServicePkg
                                     ServicePackageActivationId : 
                                     NodeName              : _Node_0
                                     AggregatedHealthState : Ok

                                     ServiceManifestName   : WordCountWebServicePkg
                                     ServicePackageActivationId : 
                                     NodeName              : _Node_0
                                     AggregatedHealthState : Ok

HealthEvents                       : 
                                     SourceId              : System.Hosting
                                     Property              : Activation
                                     HealthState           : Ok
                                     SequenceNumber        : 131444422261848308
                                     SentAt                : 7/13/2017 5:57:06 PM
                                     ReceivedAt            : 7/13/2017 5:57:17 PM
                                     TTL                   : Infinite
                                     Description           : The application was activated successfully.
                                     RemoveWhenExpired     : False
                                     IsExpired             : False
                                     Transitions           : Error->Ok = 7/13/2017 5:57:17 PM, LastWarning = 1/1/0001 12:00:00 AM

HealthStatistics                   : 
                                     DeployedServicePackage : 2 Ok, 0 Warning, 0 Error
```

### <a name="rest"></a>REST
Вы можете получить сведения о работоспособности развернутого приложения с помощью [запроса GET](/rest/api/servicefabric/get-the-health-of-a-deployed-application) или [запроса POST](/rest/api/servicefabric/get-the-health-of-a-deployed-application-by-using-a-health-policy), в который входят политики работоспособности, описанные в основном тексте.

## <a name="get-deployed-service-package-health"></a>Получение сведений о работоспособности развернутого пакета службы
Возвращает сведения о работоспособности сущности развернутого пакета службы. Входные данные:

* имя приложения (URI), имя узла (строка) и имя манифеста служб (строка), определяющие развернутый пакет службы (обязательные параметры);
* политика работоспособности приложения, используемая для переопределения политики манифеста приложения (необязательный параметр);
* фильтры для событий, с помощью которых можно указать, какие записи представляют интерес и будут возвращены в результате, например только с ошибками или с предупреждениями и ошибками (необязательные параметры). При оценке сводного состояния работоспособности сущности используются все события, независимо от выбранного фильтра.

### <a name="api"></a>API
Чтобы получить сведения о работоспособности развернутого пакета службы через API, создайте `FabricClient` и вызовите метод [GetDeployedServicePackageHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getdeployedservicepackagehealthasync) в HealthManager. Чтобы задать необязательные параметры, используйте [DeployedServicePackageHealthQueryDescription](/dotnet/api/system.fabric.description.deployedservicepackagehealthquerydescription).

```csharp
DeployedServicePackageHealth health = await fabricClient.HealthManager.GetDeployedServicePackageHealthAsync(
    new DeployedServicePackageHealthQueryDescription(applicationName, nodeName, serviceManifestName));
```

### <a name="powershell"></a>PowerShell
Командлет для получения сведений о работоспособности развернутого пакета службы — [Get-ServiceFabricDeployedServicePackageHealth](/powershell/module/servicefabric/get-servicefabricdeployedservicepackagehealth). Сначала подключитесь к кластеру с помощью командлета [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster) . Чтобы узнать, где развернуто приложение, выполните командлет [Get-ServiceFabricApplicationHealth](/powershell/module/servicefabric/get-servicefabricapplicationhealth) и просмотрите развернутые приложения. Чтобы увидеть пакеты служб в приложении, просмотрите дочерние элементы развернутого пакета службы в выходных данных командлета [Get-ServiceFabricDeployedApplicationHealth](/powershell/module/servicefabric/get-servicefabricdeployedapplicationhealth) .

Следующий командлет возвращает сведения о работоспособности пакета службы **WordCountServicePkg** приложения **fabric:/WordCount**, развернутого на узле **_Node2_**. Эта сущность составляет отчеты **System.Hosting** при успешной активации пакета службы и точки входа, а также при успешной регистрации типа службы.

```powershell
PS D:\ServiceFabric> Get-ServiceFabricDeployedApplication -ApplicationName fabric:/WordCount -NodeName _Node_2 | Get-ServiceFabricDeployedServicePackageHealth -ServiceManifestName WordCountServicePkg


ApplicationName            : fabric:/WordCount
ServiceManifestName        : WordCountServicePkg
ServicePackageActivationId : 
NodeName                   : _Node_2
AggregatedHealthState      : Ok
HealthEvents               : 
                             SourceId              : System.Hosting
                             Property              : Activation
                             HealthState           : Ok
                             SequenceNumber        : 131444422267693359
                             SentAt                : 7/13/2017 5:57:06 PM
                             ReceivedAt            : 7/13/2017 5:57:18 PM
                             TTL                   : Infinite
                             Description           : The ServicePackage was activated successfully.
                             RemoveWhenExpired     : False
                             IsExpired             : False
                             Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM

                             SourceId              : System.Hosting
                             Property              : CodePackageActivation:Code:EntryPoint
                             HealthState           : Ok
                             SequenceNumber        : 131444422267903345
                             SentAt                : 7/13/2017 5:57:06 PM
                             ReceivedAt            : 7/13/2017 5:57:18 PM
                             TTL                   : Infinite
                             Description           : The CodePackage was activated successfully.
                             RemoveWhenExpired     : False
                             IsExpired             : False
                             Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM

                             SourceId              : System.Hosting
                             Property              : ServiceTypeRegistration:WordCountServiceType
                             HealthState           : Ok
                             SequenceNumber        : 131444422272458374
                             SentAt                : 7/13/2017 5:57:07 PM
                             ReceivedAt            : 7/13/2017 5:57:18 PM
                             TTL                   : Infinite
                             Description           : The ServiceType was registered successfully.
                             RemoveWhenExpired     : False
                             IsExpired             : False
                             Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="rest"></a>REST
Вы можете получить сведения о работоспособности развернутого пакета службы с помощью [запроса GET](/rest/api/servicefabric/get-the-health-of-a-service-package) или [запроса POST](/rest/api/servicefabric/get-the-health-of-a-service-package-by-using-a-health-policy), в который входят политики работоспособности, описанные в основном тексте.

## <a name="health-chunk-queries"></a>Запросы фрагментов данных о работоспособности
Запросы фрагментов данных о работоспособности могут возвращать дочерние элементы многоуровневого кластера (рекурсивно) через входные фильтры. Он поддерживает расширенные фильтры, обеспечивающие большую гибкость в выборе возвращаемых дочерних элементов. Фильтры позволяют задать дочерние элементы по уникальному идентификатору или другим идентификаторам групп и (или) состояниям работоспособности. По умолчанию дочерние элементы не обрабатываются, в отличие от команд работоспособности, которые обязательно включают дочерние элементы первого уровня.

[Запросы о работоспособности](service-fabric-view-entities-aggregated-health.md#health-queries) возвращают только дочерние элементы первого уровня указанной сущности, которые проходят через необходимые фильтры. Чтобы получить дочерние элементы дочерних элементов, необходимо вызвать дополнительные интерфейсы API работоспособности для каждой выбранной сущности. Аналогичным образом, чтобы получить сведения о работоспособности определенных сущностей, необходимо вызвать один интерфейс API работоспособности для каждой требуемой сущности. Усовершенствованные методы фильтрации запроса фрагментов данных позволяют указывать несколько элементов в одном запросе, сводя к минимуму размеры сообщений и их количество.

Преимущество запроса фрагмента заключается в том, что пользователи могут получить сведения о работоспособности для нескольких сущностей кластеров (потенциально для всех сущностей кластеров, начиная с обязательной корневой) в одном вызове. Например, можно составить сложный запрос на сведения о работоспособности.

* Возвращать только приложения с ошибками и включать для таких приложений все службы с предупреждением или ошибкой. Для возвращенных служб включать все разделы.
* Возвращать только сведения о состоянии четырех приложений, имена которых указаны.
* Возвращать только сведения о состоянии приложений требуемого типа.
* Возвращать все развернутые сущности на узле. Возвращает все приложения, все развернутые приложения на указанном узле и все развернутые пакеты служб на этом узле.
* Возвращать все реплики с ошибками. Возвращает все приложения, службы, разделы и только реплики с ошибками.
* Возвращать все приложения. Включить все разделы для указанной службы.

В настоящее время возможность запроса фрагмента данных о работоспособности предоставляется только для сущности кластера. Этот запрос возвращает фрагмент данных о работоспособности кластера, который содержит:

* объединенные сведения о работоспособности кластера;
* список узлов с учетом входных фильтров;
* список приложений с учетом входных фильтров; Каждый фрагмент данных о работоспособности приложения содержит список всех служб, для которых используются входные фильтры, и список всех развернутых приложений, которые поддерживают такие фильтры. То же касается дочерних элементов служб и развернутых приложений. Таким образом, все сущности в кластере при запросе могут быть возвращены в иерархическом виде.

### <a name="cluster-health-chunk-query"></a>Запрос фрагмента данных о работоспособности кластера
Возвращает сведения о работоспособности сущности кластера и содержит иерархические фрагменты данных о работоспособности для требуемых дочерних элементов. Входные данные:

* политика работоспособности кластера, используемая для оценки узлов и событий кластера (необязательный параметр);
* сопоставление политики работоспособности приложения с политиками работоспособности, которые используются для переопределения политик манифеста приложения (необязательный параметр);
* фильтры для узлов и приложений, с помощью которых можно указать, какие записи представляют интерес и будут возвращены в результате (необязательный параметр). Фильтры соответствуют конкретной сущности или группам сущностей либо же применимы ко всем сущностям на этом уровне. Список фильтров может содержать один общий фильтр или фильтры для определенных идентификаторов сущностей с тонкой настройкой, которые возвращены запросом. Если настройки не установлены, по умолчанию дочерние элементы не возвращаются.
  См. дополнительные сведения о фильтрах в [NodeHealthStateFilter](/dotnet/api/system.fabric.health.nodehealthstatefilter) и [ApplicationHealthStateFilter](/dotnet/api/system.fabric.health.applicationhealthstatefilter). Фильтры приложений могут рекурсивно указывать расширенные фильтры для дочерних элементов.

Результаты для фрагмента содержат дочерние элементы, которые поддерживают фильтры.

В настоящее время запрос фрагмента не возвращает оценки неработоспособности или события сущностей. Эти дополнительные сведения можно получить, используя имеющийся запрос на сведения о работоспособности кластера.

### <a name="api"></a>API
Чтобы получить фрагмент данных о работоспособности кластера, создайте `FabricClient` и вызовите метод [GetClusterHealthChunkAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getclusterhealthchunkasync) в **HealthManager**. Можно передать данные в [ClusterHealthQueryDescription](/dotnet/api/system.fabric.description.clusterhealthchunkquerydescription) , чтобы описать политики работоспособности и расширенные фильтры.

Следующий код получает фрагмент данных о работоспособности кластера с расширенными фильтрами.

```csharp
var queryDescription = new ClusterHealthChunkQueryDescription();
queryDescription.ApplicationFilters.Add(new ApplicationHealthStateFilter()
    {
        // Return applications only if they are in error
        HealthStateFilter = HealthStateFilter.Error
    });

// Return all replicas
var wordCountServiceReplicaFilter = new ReplicaHealthStateFilter()
    {
        HealthStateFilter = HealthStateFilter.All
    };

// Return all replicas and all partitions
var wordCountServicePartitionFilter = new PartitionHealthStateFilter()
    {
        HealthStateFilter = HealthStateFilter.All
    };
wordCountServicePartitionFilter.ReplicaFilters.Add(wordCountServiceReplicaFilter);

// For specific service, return all partitions and all replicas
var wordCountServiceFilter = new ServiceHealthStateFilter()
{
    ServiceNameFilter = new Uri("fabric:/WordCount/WordCountService"),
};
wordCountServiceFilter.PartitionFilters.Add(wordCountServicePartitionFilter);

// Application filter: for specific application, return no services except the ones of interest
var wordCountApplicationFilter = new ApplicationHealthStateFilter()
    {
        // Always return fabric:/WordCount application
        ApplicationNameFilter = new Uri("fabric:/WordCount"),
    };
wordCountApplicationFilter.ServiceFilters.Add(wordCountServiceFilter);

queryDescription.ApplicationFilters.Add(wordCountApplicationFilter);

var result = await fabricClient.HealthManager.GetClusterHealthChunkAsync(queryDescription);
```

### <a name="powershell"></a>PowerShell
Командлет для получения сведений о работоспособности кластера — [Get-ServiceFabricClusterChunkHealth](/powershell/module/servicefabric/get-servicefabricclusterhealthchunk). Сначала подключитесь к кластеру с помощью командлета [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster) .

Следующий код получает данные узлов только в том случае, если они находятся в состоянии ошибки, за исключением определенного узла, который будет возвращаться при любых обстоятельствах.

```xml
PS D:\ServiceFabric> $errorFilter = [System.Fabric.Health.HealthStateFilter]::Error;
$allFilter = [System.Fabric.Health.HealthStateFilter]::All;

$nodeFilter1 = New-Object System.Fabric.Health.NodeHealthStateFilter -Property @{HealthStateFilter=$errorFilter}
$nodeFilter2 = New-Object System.Fabric.Health.NodeHealthStateFilter -Property @{NodeNameFilter="_Node_1";HealthStateFilter=$allFilter}
# Create node filter list that will be passed in the cmdlet
$nodeFilters = New-Object System.Collections.Generic.List[System.Fabric.Health.NodeHealthStateFilter]
$nodeFilters.Add($nodeFilter1)
$nodeFilters.Add($nodeFilter2)

Get-ServiceFabricClusterHealthChunk -NodeFilters $nodeFilters


HealthState                  : Warning
NodeHealthStateChunks        : 
                               TotalCount            : 1

                               NodeName              : _Node_1
                               HealthState           : Ok

ApplicationHealthStateChunks : None
```

Следующий командлет возвращает фрагмент данных кластера с фильтрами приложения.

```xml
PS D:\ServiceFabric> $errorFilter = [System.Fabric.Health.HealthStateFilter]::Error;
$allFilter = [System.Fabric.Health.HealthStateFilter]::All;

# All replicas
$replicaFilter = New-Object System.Fabric.Health.ReplicaHealthStateFilter -Property @{HealthStateFilter=$allFilter}

# All partitions
$partitionFilter = New-Object System.Fabric.Health.PartitionHealthStateFilter -Property @{HealthStateFilter=$allFilter}
$partitionFilter.ReplicaFilters.Add($replicaFilter)

# For WordCountService, return all partitions and all replicas
$svcFilter1 = New-Object System.Fabric.Health.ServiceHealthStateFilter -Property @{ServiceNameFilter="fabric:/WordCount/WordCountService"}
$svcFilter1.PartitionFilters.Add($partitionFilter)

$svcFilter2 = New-Object System.Fabric.Health.ServiceHealthStateFilter -Property @{HealthStateFilter=$errorFilter}

$appFilter = New-Object System.Fabric.Health.ApplicationHealthStateFilter -Property @{ApplicationNameFilter="fabric:/WordCount"}
$appFilter.ServiceFilters.Add($svcFilter1)
$appFilter.ServiceFilters.Add($svcFilter2)

$appFilters = New-Object System.Collections.Generic.List[System.Fabric.Health.ApplicationHealthStateFilter]
$appFilters.Add($appFilter)

Get-ServiceFabricClusterHealthChunk -ApplicationFilters $appFilters


HealthState                  : Error
NodeHealthStateChunks        : None
ApplicationHealthStateChunks : 
                               TotalCount            : 1

                               ApplicationName       : fabric:/WordCount
                               ApplicationTypeName   : WordCount
                               HealthState           : Error
                               ServiceHealthStateChunks : 
                                TotalCount            : 1

                                ServiceName           : fabric:/WordCount/WordCountService
                                HealthState           : Error
                                PartitionHealthStateChunks : 
                                    TotalCount            : 1

                                    PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
                                    HealthState           : Error
                                    ReplicaHealthStateChunks : 
                                        TotalCount            : 5

                                        ReplicaOrInstanceId   : 131444422293118720
                                        HealthState           : Ok

                                        ReplicaOrInstanceId   : 131444422293118721
                                        HealthState           : Ok

                                        ReplicaOrInstanceId   : 131444422293113678
                                        HealthState           : Ok

                                        ReplicaOrInstanceId   : 131444422293113679
                                        HealthState           : Ok

                                        ReplicaOrInstanceId   : 131444422260002646
                                        HealthState           : Error
```

Следующий командлет возвращает все сущности, развернутые на узле.

```xml
PS D:\ServiceFabric> $errorFilter = [System.Fabric.Health.HealthStateFilter]::Error;
$allFilter = [System.Fabric.Health.HealthStateFilter]::All;

$dspFilter = New-Object System.Fabric.Health.DeployedServicePackageHealthStateFilter -Property @{HealthStateFilter=$allFilter}
$daFilter =  New-Object System.Fabric.Health.DeployedApplicationHealthStateFilter -Property @{HealthStateFilter=$allFilter;NodeNameFilter="_Node_2"}
$daFilter.DeployedServicePackageFilters.Add($dspFilter)

$appFilter = New-Object System.Fabric.Health.ApplicationHealthStateFilter -Property @{HealthStateFilter=$allFilter}
$appFilter.DeployedApplicationFilters.Add($daFilter)

$appFilters = New-Object System.Collections.Generic.List[System.Fabric.Health.ApplicationHealthStateFilter]
$appFilters.Add($appFilter)
Get-ServiceFabricClusterHealthChunk -ApplicationFilters $appFilters


HealthState                  : Error
NodeHealthStateChunks        : None
ApplicationHealthStateChunks : 
                               TotalCount            : 2

                               ApplicationName       : fabric:/System
                               HealthState           : Ok
                               DeployedApplicationHealthStateChunks : 
                                TotalCount            : 1

                                NodeName              : _Node_2
                                HealthState           : Ok
                                DeployedServicePackageHealthStateChunks :
                                    TotalCount            : 1

                                    ServiceManifestName   : FAS
                                    ServicePackageActivationId : 
                                    HealthState           : Ok



                               ApplicationName       : fabric:/WordCount
                               ApplicationTypeName   : WordCount
                               HealthState           : Error
                               DeployedApplicationHealthStateChunks : 
                                TotalCount            : 1

                                NodeName              : _Node_2
                                HealthState           : Ok
                                DeployedServicePackageHealthStateChunks :
                                    TotalCount            : 1

                                    ServiceManifestName   : WordCountServicePkg
                                    ServicePackageActivationId : 
                                    HealthState           : Ok
```

### <a name="rest"></a>REST
Вы можете получить фрагмент данных о работоспособности кластера с помощью [запроса GET](/rest/api/servicefabric/get-the-health-of-a-cluster-using-health-chunks) или [запроса POST](/rest/api/servicefabric/health-of-cluster), в который входят политики работоспособности, описанные в основном тексте.

## <a name="general-queries"></a>Общие запросы
Общие запросы возвращают перечень сущностей Service Fabric указанного типа. Их можно просмотреть с помощью API (используя методы **FabricClient.QueryManager**), а также командлетов PowerShell и REST. Эти запросы состоят из вложенных запросов от нескольких компонентов. Один из них — [хранилище данных о работоспособности](service-fabric-health-introduction.md#health-store), заполняющее сводные данные о состоянии работоспособности в каждом результате запроса.  

> [!NOTE]
> Общие запросы возвращают сводные данные о состоянии работоспособности сущности и не содержат подробных данных о работоспособности. Если сущность неработоспособна, вы можете отправить запросы о работоспособности, чтобы получить всю информацию о работоспособности, в том числе события, состояние работоспособности дочерних элементов и оценку неработоспособности.
>
>

Если после выполнения общего запроса возвращается неизвестное состояние, возможно, в хранилище данных о работоспособности нет всех данных о сущности. Но также возможно, что вложенный запрос к хранилищу завершился с ошибкой (например, в результате ошибки связи или регулирования хранилища данных о работоспособности). Отправьте дополнительный запрос о работоспособности сущности. При возникновении временных ошибок вложенных запросов, таких как проблемы с сетью, выполнение следующего запроса может завершиться успешно. В нем также могут содержаться дополнительные сведения из хранилища данных о работоспособности о том, почему не предоставлена сущность.

Ниже приведены запросы, возвращающие сведения о состоянии работоспособности ( **HealthState** ) для сущностей:

* Список узлов — возвращает список узлов в кластере (с разбивкой на страницы).
  * API — [FabricClient.QueryClient.GetNodeListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getnodelistasync)
  * PowerShell: Get-ServiceFabricNode.
* Список приложений — возвращает список приложений в кластере (с разбивкой на страницы).
  * API — [FabricClient.QueryClient.GetApplicationListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getapplicationlistasync)
  * PowerShell: Get-ServiceFabricApplication.
* Список служб — возвращает список служб в приложении (с разбивкой на страницы).
  * API — [FabricClient.QueryClient.GetServiceListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getservicelistasync)
  * PowerShell: Get-ServiceFabricService.
* Список разделов — возвращает список разделов в службе (с разбивкой на страницы).
  * API — [FabricClient.QueryClient.GetPartitionListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getpartitionlistasync)
  * PowerShell: Get-ServiceFabricPartition.
* Список реплик — возвращает список реплик в разделе (с разбивкой на страницы).
  * API — [FabricClient.QueryClient.GetReplicaListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getreplicalistasync)
  * PowerShell: Get-ServiceFabricReplica.
* Список развернутых приложений — возвращает список развернутых приложений на узле.
  * API — [FabricClient.QueryClient.GetDeployedApplicationListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getdeployedapplicationlistasync)
  * PowerShell: Get-ServiceFabricDeployedApplication.
* Список развернутых пакетов служб — возвращает список пакетов служб в развернутом приложении.
  * API — [FabricClient.QueryClient.GetDeployedServicePackageListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getdeployedservicepackagelistasync)
  * PowerShell: Get-ServiceFabricDeployedApplication.

> [!NOTE]
> Некоторые запросы возвращают результаты с разбивкой на страницы. В результатах этих запросов возвращается список, производный от [PagedList\<T>](/dotnet/api/system.fabric.query.pagedlist-1). Если результаты не соответствуют сообщению, возвращается только страница и устанавливается маркер ContinuationToken, который отслеживает остановку перечисления. Чтобы получить следующие результаты, необходимо продолжить вызов того же запроса, передавая маркер продолжения из предыдущего запроса.

### <a name="examples"></a>Примеры
Следующий код возвращает неработоспособные приложения в кластере.

```csharp
var applications = fabricClient.QueryManager.GetApplicationListAsync().Result.Where(
  app => app.HealthState == HealthState.Error);
```

Следующий командлет возвращает сведения о приложении fabric:/WordCount. Обратите внимание, что состояние работоспособности — "Предупреждение".

```powershell
PS C:\> Get-ServiceFabricApplication -ApplicationName fabric:/WordCount

ApplicationName        : fabric:/WordCount
ApplicationTypeName    : WordCount
ApplicationTypeVersion : 1.0.0
ApplicationStatus      : Ready
HealthState            : Warning
ApplicationParameters  : { "WordCountWebService_InstanceCount" = "1";
                         "_WFDebugParams_" = "[{"ServiceManifestName":"WordCountWebServicePkg","CodePackageName":"Code","EntryPointType":"Main","Debug
                         ExePath":"C:\\Program Files (x86)\\Microsoft Visual Studio
                         14.0\\Common7\\Packages\\Debugger\\VsDebugLaunchNotify.exe","DebugArguments":" {74f7e5d5-71a9-47e2-a8cd-1878ec4734f1} -p
                         [ProcessId] -tid [ThreadId]","EnvironmentBlock":"_NO_DEBUG_HEAP=1\u0000"},{"ServiceManifestName":"WordCountServicePkg","CodeP
                         ackageName":"Code","EntryPointType":"Main","DebugExePath":"C:\\Program Files (x86)\\Microsoft Visual Studio
                         14.0\\Common7\\Packages\\Debugger\\VsDebugLaunchNotify.exe","DebugArguments":" {2ab462e6-e0d1-4fda-a844-972f561fe751} -p
                         [ProcessId] -tid [ThreadId]","EnvironmentBlock":"_NO_DEBUG_HEAP=1\u0000"}]" }
```

Следующий командлет возвращает службы с состоянием работоспособности "Ошибка".

```powershell
PS D:\ServiceFabric> Get-ServiceFabricApplication | Get-ServiceFabricService | where {$_.HealthState -eq "Error"}


ServiceName            : fabric:/WordCount/WordCountService
ServiceKind            : Stateful
ServiceTypeName        : WordCountServiceType
IsServiceGroup         : False
ServiceManifestVersion : 1.0.0
HasPersistedState      : True
ServiceStatus          : Active
HealthState            : Error
```

## <a name="cluster-and-application-upgrades"></a>Обновление кластера и приложений
Во время отслеживаемого обновления кластера и приложений Service Fabric проверяет все компоненты, чтобы обеспечить их непрерывную работоспособность. Если в результате оценки с использованием настроенных политик работоспособности окажется, что у сущности неработоспособное состояние, при обновлении будут применены соответствующие ему политики, чтобы определить, какое действие должно быть следующим. Обновление можно приостановить, чтобы пользователь мог внести изменения (например, исправить ошибки или изменить политики). Кроме того, можно выполнить автоматический откат до предыдущей рабочей версии.

Во время обновления *кластера* можно узнать состояние этого процесса. Эти сведения включают в себя данные об оценках неработоспособности, которые указывают, что не работает в кластере. Если выполняется откат обновления из-за проблем работоспособности, состояние обновления будет содержать последние причины неработоспособности. Используя эти данные, администраторы могут определить, что пошло не так после отката или остановки обновления.

Кроме того, во время обновления *приложения* можно просмотреть результаты оценки неработоспособности (содержатся в состоянии обновления).

Далее показано состояние обновления для измененного приложения fabric:/WordCount. Устройство наблюдения сообщило об ошибке на одной из реплик. Выполняется откат обновления из-за несоответствия условиям проверок работоспособности.

```powershell
PS C:\> Get-ServiceFabricApplicationUpgrade fabric:/WordCount

ApplicationName               : fabric:/WordCount
ApplicationTypeName           : WordCount
TargetApplicationTypeVersion  : 1.0.0.0
ApplicationParameters         : {}
StartTimestampUtc             : 4/21/2017 5:23:26 PM
FailureTimestampUtc           : 4/21/2017 5:23:37 PM
FailureReason                 : HealthCheck
UpgradeState                  : RollingBackInProgress
UpgradeDuration               : 00:00:23
CurrentUpgradeDomainDuration  : 00:00:00
CurrentUpgradeDomainProgress  : UD1

                                NodeName            : _Node_1
                                UpgradePhase        : Upgrading

                                NodeName            : _Node_2
                                UpgradePhase        : Upgrading

                                NodeName            : _Node_3
                                UpgradePhase        : PreUpgradeSafetyCheck
                                PendingSafetyChecks :
                                EnsurePartitionQuorum - PartitionId: 30db5be6-4e20-4698-8185-4bd7ca744020
NextUpgradeDomain             : UD2
UpgradeDomainsStatus          : { "UD1" = "Completed";
                                "UD2" = "Pending";
                                "UD3" = "Pending";
                                "UD4" = "Pending" }
UnhealthyEvaluations          :
                                Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                                  Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Error'.

                                      Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                      Unhealthy partition: PartitionId='a1f83a35-d6bf-4d39-b90d-28d15f39599b', AggregatedHealthState='Error'.

                                          Unhealthy replicas: 20% (1/5), MaxPercentUnhealthyReplicasPerPartition=0%.

                                          Unhealthy replica: PartitionId='a1f83a35-d6bf-4d39-b90d-28d15f39599b',
                                  ReplicaOrInstanceId='131031502346844058', AggregatedHealthState='Error'.

                                              Error event: SourceId='DiskWatcher', Property='Disk'.

UpgradeKind                   : Rolling
RollingUpgradeMode            : UnmonitoredAuto
ForceRestart                  : False
UpgradeReplicaSetCheckTimeout : 00:15:00
```

Дополнительные сведения см. в статье [Обновление приложения Service Fabric](service-fabric-application-upgrade.md).

## <a name="use-health-evaluations-to-troubleshoot"></a>Использование оценки работоспособности для устранения неполадок
При возникновении проблемы с кластером или приложением просмотрите сведения о работоспособности кластера или приложения, чтобы выявить неполадку. Данные оценки неработоспособности содержат подробные сведения о причине активации текущего состояния неработоспособности. При необходимости можно детализировать неработоспособные дочерние сущности, чтобы определить основную причину.

Например предположим, что приложение неработоспособно, так как поступил отчет об ошибке в одной из ее реплик. Приведенный ниже командлет PowerShell показывает оценку неработоспособности.

```powershell
PS D:\ServiceFabric> Get-ServiceFabricApplicationHealth fabric:/WordCount -EventsFilter None -ServicesFilter None -DeployedApplicationsFilter None -ExcludeHealthStatistics


ApplicationName                 : fabric:/WordCount
AggregatedHealthState           : Error
UnhealthyEvaluations            : 
                                  Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                                  Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Error'.

                                    Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                    Unhealthy partition: PartitionId='af2e3e44-a8f8-45ac-9f31-4093eb897600', AggregatedHealthState='Error'.

                                        Unhealthy replicas: 20% (1/5), MaxPercentUnhealthyReplicasPerPartition=0%.

                                        Unhealthy replica: PartitionId='af2e3e44-a8f8-45ac-9f31-4093eb897600', ReplicaOrInstanceId='131444422260002646', AggregatedHealthState='Error'.

                                            Error event: SourceId='MyWatchdog', Property='Memory'.

ServiceHealthStates             : None
DeployedApplicationHealthStates : None
HealthEvents                    : None
```

Можно просмотреть реплику, чтобы получить дополнительные сведения.

```powershell
PS D:\ServiceFabric> Get-ServiceFabricReplicaHealth -ReplicaOrInstanceId 131444422260002646 -PartitionId af2e3e44-a8f8-45ac-9f31-4093eb897600


PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
ReplicaId             : 131444422260002646
AggregatedHealthState : Error
UnhealthyEvaluations  : 
                        Error event: SourceId='MyWatchdog', Property='Memory'.

HealthEvents          : 
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 131444422263668344
                        SentAt                : 7/13/2017 5:57:06 PM
                        ReceivedAt            : 7/13/2017 5:57:18 PM
                        TTL                   : Infinite
                        Description           : Replica has been created._Node_2
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM

                        SourceId              : MyWatchdog
                        Property              : Memory
                        HealthState           : Error
                        SequenceNumber        : 131444451657749403
                        SentAt                : 7/13/2017 6:46:05 PM
                        ReceivedAt            : 7/13/2017 6:46:05 PM
                        TTL                   : Infinite
                        Description           : 
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Warning->Error = 7/13/2017 6:46:05 PM, LastOk = 1/1/0001 12:00:00 AM
```

> [!NOTE]
> Оценки неработоспособности указывают на первую причину, по которой была получена текущая оценка работоспособности сущности. Возможно, есть другие события, которые вызывают это состояние, но они не отражаются в оценках. Чтобы получить дополнительную информацию, определите все отчеты о неработоспособности в кластере, просмотрев подробные сведения о сущностях работоспособности.
>
>

## <a name="next-steps"></a>Дальнейшие шаги
[Устранение неполадок с помощью отчетов о работоспособности системы](service-fabric-understand-and-troubleshoot-with-system-health-reports.md)

[Добавление настраиваемых отчетов о работоспособности Service Fabric](service-fabric-report-health.md)

[Проверка работоспособности службы и оповещение о проблемах](service-fabric-diagnostics-how-to-report-and-check-service-health.md)

[Мониторинг и диагностика состояния служб в локальной среде разработки](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)

[Обновление приложения Service Fabric](service-fabric-application-upgrade.md)
