---
title: Запрос событий кластера с помощью API-интерфейсов Евентсторе
description: Узнайте, как использовать интерфейсы API EventStore в Azure Service Fabric для запроса событий платформы.
ms.topic: conceptual
ms.date: 02/25/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: bc8909a6f25c3adcba60de639d822769baff703f
ms.sourcegitcommit: a9ce1da049c019c86063acf442bb13f5a0dde213
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/27/2021
ms.locfileid: "105628815"
---
# <a name="query-eventstore-apis-for-cluster-events"></a>Выполнение запросов к интерфейсам API EventStore для получения событий кластера

В этой статье описывается, как выполнить запрос к интерфейсам API EventStore, которые доступны в Service Fabric версии 6.2 и более поздней версии. Если вы хотите узнать больше о службе EventStore, прочитайте [обзор службы EventStore](service-fabric-diagnostics-eventstore.md). В настоящее время службе EventStore доступны только данные за последние 7 дней (это обусловлено диагностической политикой хранения данных кластера).

>[!NOTE]
>API EventStore предоставляются как общедоступная версия в рамках Service Fabric 6.4 для кластеров Windows, работающих в Azure.

К интерфейсам API EventStore можно обращаться напрямую через конечную точку REST или программно. В зависимости от запроса требуется указать несколько параметров, которые необходимы для сбора соответствующих данных. Как правило, это:
* `api-version`: версия интерфейсов API EventStore, которую вы используете.
* `StartTimeUtc`: определяет начало запрашиваемого периода.
* `EndTimeUtc`: конец периода времени.

Кроме этого, доступны дополнительные параметры, в том числе:
* `timeout`: переопределяет время ожидания по умолчанию (60 с) для выполнения операции запроса.
* `eventstypesfilter`: позволяет фильтровать события по типу.
* `ExcludeAnalysisEvents`: исключает из результатов события анализа. По умолчанию запросы EventStore будут возвращать события анализа, когда это возможно. События анализа — это события операционного канала, которые содержат дополнительный контекст и более подробные сведения по сравнению с обычными событиями Service Fabric.
* `SkipCorrelationLookup`: позволяет не выполнять поиск потенциальных связанных событий в кластере. По умолчанию EventStore попытается сопоставить события в кластере и связать события между собой, когда это возможно. 

У каждой сущности в кластере можно запросить события. Можно также запрашивать события для всех сущностей определенного типа. Например, можно запросить события для конкретного узла или для всех узлов в кластере. Текущий набор сущностей, для которых вы можете запрашивать события (и структура запроса):
* кластер: `/EventsStore/Cluster/Events`;
* Узлы: `/EventsStore/Nodes/Events`
* узел: `/EventsStore/Nodes/<NodeName>/$/Events`;
* приложения: `/EventsStore/Applications/Events`;
* приложение: `/EventsStore/Applications/<AppName>/$/Events`;
* службы: `/EventsStore/Services/Events`;
* служба: `/EventsStore/Services/<ServiceName>/$/Events`;
* секции: `/EventsStore/Partitions/Events`;
* секция: `/EventsStore/Partitions/<PartitionID>/$/Events`;
* реплики: `/EventsStore/Partitions/<PartitionID>/$/Replicas/Events`;
* реплика: `/EventsStore/Partitions/<PartitionID>/$/Replicas/<ReplicaID>/$/Events`.

>[!NOTE]
>При ссылке на имя приложения или службы запрос может не содержать префикс fabric:/. Кроме того, если имена приложений или служб содержат косую черту "/", замените ее знаком "~", чтобы сохранить работоспособность запроса. Например, если приложение отображается как fabric:/App1/FrontendApp, то в запросах к этому приложению оно имело бы следующий вид: `/EventsStore/Applications/App1~FrontendApp/$/Events`.
>Кроме того, сейчас отчеты о работоспособности служб отображаются под соответствующим приложением, поэтому следует запрашивать события `DeployedServiceHealthReportCreated` для соответствующей сущности приложения. 

## <a name="query-the-eventstore-via-rest-api-endpoints"></a>Отправка запроса к EventStore через конечные точки REST API

Вы можете отправить запрос к EventStore напрямую через конечную точку REST, выполнив запросы `GET` к `<your cluster address>/EventsStore/<entity>/Events/`.

Например, запрос всех событий кластера с `2018-04-03T18:00:00Z` до `2018-04-04T18:00:00Z` выглядит следующим образом.

```
Method: GET 
URL: http://mycluster:19080/EventsStore/Cluster/Events?api-version=6.4&StartTimeUtc=2018-04-03T18:00:00Z&EndTimeUtc=2018-04-04T18:00:00Z
```

Он может не возвращать события или вернуть список событий в формате JSON:

```json
Response: 200
Body:
[
  {
    "Kind": "ClusterUpgradeStart",
    "CurrentClusterVersion": "0.0.0.0:",
    "TargetClusterVersion": "6.2:1.0",
    "UpgradeType": "Rolling",
    "RollingUpgradeMode": "UnmonitoredAuto",
    "FailureAction": "Manual",
    "EventInstanceId": "090add3c-8f56-4d35-8d57-a855745b6064",
    "TimeStamp": "2018-04-03T20:18:59.4313064Z",
    "HasCorrelatedEvents": false
  },
  {
    "Kind": "ClusterUpgradeDomainComplete",
    "TargetClusterVersion": "6.2:1.0",
    "UpgradeState": "RollingForward",
    "UpgradeDomains": "(0 1 2)",
    "UpgradeDomainElapsedTimeInMs": "78.5288",
    "EventInstanceId": "090add3c-8f56-4d35-8d57-a855745b6064",
    "TimeStamp": "2018-04-03T20:19:59.5729953Z",
    "HasCorrelatedEvents": false
  },
  {
    "Kind": "ClusterUpgradeDomainComplete",
    "TargetClusterVersion": "6.2:1.0",
    "UpgradeState": "RollingForward",
    "UpgradeDomains": "(3 4)",
    "UpgradeDomainElapsedTimeInMs": "0",
    "EventInstanceId": "090add3c-8f56-4d35-8d57-a855745b6064",
    "TimeStamp": "2018-04-03T20:20:59.6271949Z",
    "HasCorrelatedEvents": false
  },
  {
    "Kind": "ClusterUpgradeComplete",
    "TargetClusterVersion": "6.2:1.0",
    "OverallUpgradeElapsedTimeInMs": "120196.5212",
    "EventInstanceId": "090add3c-8f56-4d35-8d57-a855745b6064",
    "TimeStamp": "2018-04-03T20:20:59.8134457Z",
    "HasCorrelatedEvents": false
  }
]
```

Как видите, после первого запуска с `2018-04-03T18:00:00Z` до `2018-04-04T18:00:00Z` этот кластер успешно завершил свое первое обновление с `"CurrentClusterVersion": "0.0.0.0:"` до `"TargetClusterVersion": "6.2:1.0"` за `"OverallUpgradeElapsedTimeInMs": "120196.5212"` мс.

## <a name="query-the-eventstore-programmatically"></a>Отправка запроса к EventStore программным способом

Можно также выполнить запрос к EventStore программно при помощи [ клиентской библиотеки Service Fabric](/dotnet/api/overview/azure/service-fabric#client-library).

Настроив клиент Service Fabric, вы сможете запрашивать события, обращаясь к EventStore следующим образом: `sfhttpClient.EventStore.<request>`.

Ниже приведен пример запроса всех событий кластера с `2018-04-03T18:00:00Z` до `2018-04-04T18:00:00Z` с помощью функции `GetClusterEventListAsync`.

```csharp
var sfhttpClient = ServiceFabricClientFactory.Create(clusterUrl, settings);

var clstrEvents = sfhttpClient.EventsStore.GetClusterEventListAsync(
    "2018-04-03T18:00:00Z",
    "2018-04-04T18:00:00Z")
    .GetAwaiter()
    .GetResult()
    .ToList();
```

Вот еще один пример запроса и вывода состояния работоспособности кластера и всех событий узла за сентябрь 2018 г.

```csharp
  const int timeoutSecs = 60;
  var clusterUrl = new Uri(@"http://localhost:19080"); // This example is for a Local cluster
  var sfhttpClient = ServiceFabricClientFactory.Create(clusterUrl);

  var clusterHealth = sfhttpClient.Cluster.GetClusterHealthAsync().GetAwaiter().GetResult();
  Console.WriteLine("Cluster Health: {0}", clusterHealth.AggregatedHealthState.Value.ToString());

  
  Console.WriteLine("Querying for node events...");
  var nodesEvents = sfhttpClient.EventsStore.GetNodesEventListAsync(
      "2018-09-01T00:00:00Z",
      "2018-09-30T23:59:59Z",
      timeoutSecs,
      "NodeDown,NodeUp")
      .GetAwaiter()
      .GetResult()
      .ToList();
  Console.WriteLine("Result Count: {0}", nodesEvents.Count());

  foreach (var nodeEvent in nodesEvents)
  {
      Console.Write("Node event happened at {0}, Node name: {1} ", nodeEvent.TimeStamp, nodeEvent.NodeName);
      if (nodeEvent is NodeDownEvent)
      {
          var nodeDownEvent = nodeEvent as NodeDownEvent;
          Console.WriteLine("(Node is down, and it was last up at {0})", nodeDownEvent.LastNodeUpAt);
      }
      else if (nodeEvent is NodeUpEvent)
      {
          var nodeUpEvent = nodeEvent as NodeUpEvent;
          Console.WriteLine("(Node is up, and it was last down at {0})", nodeUpEvent.LastNodeDownAt);
      }
  }
```

## <a name="sample-scenarios-and-queries"></a>Примеры сценариев и запросов

Вот несколько примеров того, как можно вызвать интерфейсы REST API хранилища событий для получения представления о состоянии кластера.

*Обновления кластера:*

Чтобы узнать время последнего успешного обновления или попытки обновления кластера на прошлой неделе, можно отправить запрос к API и получить данные о недавно завершенных обновлениях кластера. Для этого нужно запросить события ClusterUpgradeCompleted в EventStore: `https://mycluster.cloudapp.azure.com:19080/EventsStore/Cluster/Events?api-version=6.4&starttimeutc=2017-04-22T17:01:51Z&endtimeutc=2018-04-29T17:02:51Z&EventsTypesFilter=ClusterUpgradeCompleted`.

*Проблемы при обновлении кластера*

Аналогично, если при последнем обновлении кластера возникли проблемы, может запросить все события для сущности кластера. Вы увидите различные события, включая запуск обновлений и каждый домен обновления, для которого обновление прошло успешно. Также вы увидите события на момент, когда начался откат, и соответствующие события работоспособности. Запрос, который можно использовать для получения этих сведений: `https://mycluster.cloudapp.azure.com:19080/EventsStore/Cluster/Events?api-version=6.4&starttimeutc=2017-04-22T17:01:51Z&endtimeutc=2018-04-29T17:02:51Z`

*Изменения состояния узлов*

Чтобы просмотреть изменения состояния узлов за последние несколько дней (запуск и остановка узлов, а также их активация или деактивация (платформой, службой тестирования в условиях хаоса или пользователем)), используйте следующий запрос: `https://mycluster.cloudapp.azure.com:19080/EventsStore/Nodes/Events?api-version=6.4&starttimeutc=2017-04-22T17:01:51Z&endtimeutc=2018-04-29T17:02:51Z`.

*События приложений*

Вы также можете отслеживать последние развертывания и обновления приложений. Используйте следующий запрос, чтобы увидеть все события, связанные с приложениями, в кластере: `https://mycluster.cloudapp.azure.com:19080/EventsStore/Applications/Events?api-version=6.4&starttimeutc=2017-04-22T17:01:51Z&endtimeutc=2018-04-29T17:02:51Z`.

*Хронологические данные о работоспособности приложения*

Помимо событий жизненного цикла приложений может понадобиться просмотреть хронологические данные о работоспособности определенного приложения. Для этого можно указать имя приложения, для которого нужно собрать данные. Используйте данный запрос, чтобы получить все события работоспособности приложения: `https://mycluster.cloudapp.azure.com:19080/EventsStore/Applications/myApp/$/Events?api-version=6.4&starttimeutc=2018-03-24T17:01:51Z&endtimeutc=2018-03-29T17:02:51Z&EventsTypesFilter=ApplicationNewHealthReport`. Если вы хотите включить в результаты события работоспособности с истекшим сроком действия (то есть с истекшим сроком жизни), добавьте `,ApplicationHealthReportExpired` в конец запроса, чтобы использовать фильтр по двум типам событий.

*Хронологические данные о работоспособности для всех служб в myApp*

В настоящее время события отчета о работоспособности для служб отображаются как события `DeployedServicePackageNewHealthReport` для соответствующей сущности приложения. Чтобы увидеть, как ваши службы функционируют для App1, используйте следующий запрос: `https://mycluster.cloudapp.azure.com:19080/EventsStore/Applications/myapp/$/Events?api-version=6.4&starttimeutc=2017-04-22T17:01:51Z&endtimeutc=2018-04-29T17:02:51Z&EventsTypesFilter=DeployedServicePackageNewHealthReport`.

*Перенастройка секций*

Чтобы просмотреть все перемещения секций, которые произошли в кластере, запросите событие `PartitionReconfigured`. Это поможет выяснить, когда и какие рабочие нагрузки выполнялись на том или ином узле, при диагностике проблем в кластере. Вот пример запроса для этого: `https://mycluster.cloudapp.azure.com:19080/EventsStore/Partitions/Events?api-version=6.4&starttimeutc=2018-04-22T17:01:51Z&endtimeutc=2018-04-29T17:02:51Z&EventsTypesFilter=PartitionReconfigured`.

*Служба тестирования в условиях хаоса*

Это событие запуска или остановки службы тестирования в условиях хаоса, представленной на уровне кластера. Для просмотра последних данных об использовании службы тестирования в условиях хаоса выполните следующий запрос: `https://mycluster.cloudapp.azure.com:19080/EventsStore/Cluster/Events?api-version=6.4&starttimeutc=2017-04-22T17:01:51Z&endtimeutc=2018-04-29T17:02:51Z&EventsTypesFilter=ChaosStarted,ChaosStopped`.
