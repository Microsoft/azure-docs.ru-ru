---
title: Диспетчер ресурсов кластера Service Fabric. группы приложений
description: Общие сведения о назначении групп приложений в диспетчере кластерных ресурсов службы Fabric Service
author: masnider
ms.topic: conceptual
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: 988c7ce52125800c16aa785d5b1458604a927ecd
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "75452152"
---
# <a name="introduction-to-application-groups"></a>Введение в группы приложений
Диспетчер кластерных ресурсов Service Fabric, который управляет ресурсами кластера, обычно равномерно распределяет нагрузку (представленную в [метриках](service-fabric-cluster-resource-manager-metrics.md)) по всему кластеру. Service Fabric управляет емкостью узлов в кластере и [емкостью](service-fabric-cluster-resource-manager-cluster-description.md) кластера в целом. Метрики и емкость отлично работают для большинства рабочих нагрузок, но для структур, активно использующих разные экземпляры приложений Service Fabric, иногда возникают дополнительные требования. Например, может потребоваться следующее.

- Резервирование определенной емкости на узлах в кластере для служб внутри какого-либо именованного экземпляра приложения.
- Ограничение общего числа узлов, на которых работают службы в именованном экземпляре приложения (вместо распределения по всему кластеру).
- Определение емкости для именованного экземпляра приложения, чтобы ограничить число служб в нем или общее потребление ресурсов этими службами.

Для соблюдения этих требований диспетчер кластерных ресурсов Service Fabric поддерживает группы приложений.

## <a name="limiting-the-maximum-number-of-nodes"></a>Ограничение максимального количества узлов
В простейшем варианте использования емкость приложения позволяет ограничить количество узлов, на которых создаются экземпляры приложения. Это позволяет объединить все службы, выполняющиеся в этом экземпляре приложения, на заданном количестве компьютеров. Консолидация удобна в тех случаях, когда вы пытаетесь спрогнозировать или ограничить использование физических ресурсов службами в этом именованном экземпляре приложения. 

Следующий рисунок демонстрирует экземпляры приложения, для которых определено или не определено максимальное количество узлов.

<center>

![Определение максимального количества узлов для экземпляров приложения][Image1]
</center>

В примере слева для приложения с тремя службами не определено максимальное число узлов. Менеджер кластерных ресурсов распределил реплики по всем шести доступным узлам, чтобы сбалансировать нагрузку на кластер (поведение по умолчанию). В примере справа мы видим то же приложение, но с ограничением до трех узлов.

Такое поведение управляется параметром MaximumNodes. Этот параметр можно задать при создании приложения или изменить для уже выполняющегося экземпляра приложения.

PowerShell

``` posh
New-ServiceFabricApplication -ApplicationName fabric:/AppName -ApplicationTypeName AppType1 -ApplicationTypeVersion 1.0.0.0 -MaximumNodes 3
Update-ServiceFabricApplication –ApplicationName fabric:/AppName –MaximumNodes 5
```

C#

``` csharp
ApplicationDescription ad = new ApplicationDescription();
ad.ApplicationName = new Uri("fabric:/AppName");
ad.ApplicationTypeName = "AppType1";
ad.ApplicationTypeVersion = "1.0.0.0";
ad.MaximumNodes = 3;
await fc.ApplicationManager.CreateApplicationAsync(ad);

ApplicationUpdateDescription adUpdate = new ApplicationUpdateDescription(new Uri("fabric:/AppName"));
adUpdate.MaximumNodes = 5;
await fc.ApplicationManager.UpdateApplicationAsync(adUpdate);

```

В наборе узлов диспетчер кластерных ресурсов не гарантирует, какие объекты службы размещаются вместе и какие узлы используются.

## <a name="application-metrics-load-and-capacity"></a>Метрики, нагрузка и емкость приложения
Группы приложений позволяют также определять метрики для конкретного именованного экземпляра приложения и устанавливать емкость экземпляра приложения для этих метрик. Метрики приложения позволяют отслеживать, резервировать и ограничивать потребление ресурсов службами внутри конкретного экземпляра приложения.

Для каждой метрики приложения можно установить два значения.

- **Общая емкость приложения**. Это значение представляет общую емкость всего приложения по определенной метрике. Диспетчер кластерных ресурсов запрещает создание новых служб для этого экземпляра приложения, если такое создание приведет к превышению указанного значения общей емкости. Рассмотрим такой пример: для экземпляра приложения указано ограничение емкости 10, а текущая нагрузка составляет 5. Создание службы, для которой указана стандартная нагрузка 10, будет запрещено.
- **Максимальная емкость узла**. Это значение определяет максимальную общую нагрузку для приложения на отдельном узле. Если нагрузка превышает емкость, диспетчер кластерных ресурсов перемещает реплики на другие узлы, тем самым уменьшая нагрузку.


PowerShell:

``` posh
New-ServiceFabricApplication -ApplicationName fabric:/AppName -ApplicationTypeName AppType1 -ApplicationTypeVersion 1.0.0.0 -Metrics @("MetricName:Metric1,MaximumNodeCapacity:100,MaximumApplicationCapacity:1000")
```

C#:

``` csharp
ApplicationDescription ad = new ApplicationDescription();
ad.ApplicationName = new Uri("fabric:/AppName");
ad.ApplicationTypeName = "AppType1";
ad.ApplicationTypeVersion = "1.0.0.0";

var appMetric = new ApplicationMetricDescription();
appMetric.Name = "Metric1";
appMetric.TotalApplicationCapacity = 1000;
appMetric.MaximumNodeCapacity = 100;
ad.Metrics.Add(appMetric);
await fc.ApplicationManager.CreateApplicationAsync(ad);
```

## <a name="reserving-capacity"></a>Резервирование емкости
Еще одно распространенное использование групп приложений — резервирование ресурсов кластера для конкретного экземпляра приложения. При создании экземпляра приложения всегда резервируется пространство.

Резервирование пространства в кластере для приложения происходит немедленно, даже когда:
- экземпляр приложения создан, но еще не содержит ни одной службы;
- число служб в экземпляре приложения каждый раз изменяется; 
- службы существуют, но не используют ресурсы. 

Чтобы зарезервировать ресурсы для экземпляра приложения, следует использовать два дополнительных параметра: *MinimumNodes* и *NodeReservationCapacity*.

- **MinimumNodes**. Этот параметр определяет минимальное количество узлов, на которых должен работать экземпляр приложения.  
- **NodeReservationCapacity**. Этот параметр задается на уровне метрики для приложения. Его значение — это объем данной метрики, резервируемый для приложения на любом узле, на котором выполняются службы в этом приложении.

Применяя **MinimumNodes** и **NodeReservationCapacity**, можно гарантировать резервирование минимальной нагрузки для приложения в кластере. Если оставшаяся емкость в кластере меньше, чем общая требуемая резервируемая емкость, то при создании приложения происходит сбой. 

Рассмотрим пример резервирования емкости.

<center>

![Определение резервируемой емкости для экземпляров приложения][Image2]
</center>

В примере слева для приложений не определены параметры емкости. Диспетчер кластерных ресурсов распределяет всю нагрузку по обычным правилам.

Допустим, в примере справа приложение Application1 было создано со следующими параметрами:

- MinimumNodes имеет значение 2;
- для приложения определена метрика со следующими значениями:
  - NodeReservationCapacity = 20;

PowerShell

 ``` posh
 New-ServiceFabricApplication -ApplicationName fabric:/AppName -ApplicationTypeName AppType1 -ApplicationTypeVersion 1.0.0.0 -MinimumNodes 2 -Metrics @("MetricName:Metric1,NodeReservationCapacity:20")
 ```

C#

 ``` csharp
ApplicationDescription ad = new ApplicationDescription();
ad.ApplicationName = new Uri("fabric:/AppName");
ad.ApplicationTypeName = "AppType1";
ad.ApplicationTypeVersion = "1.0.0.0";
ad.MinimumNodes = 2;

var appMetric = new ApplicationMetricDescription();
appMetric.Name = "Metric1";
appMetric.NodeReservationCapacity = 20;

ad.Metrics.Add(appMetric);

await fc.ApplicationManager.CreateApplicationAsync(ad);
```

Service Fabric резервирует емкость для Application1 на двух узлах и не разрешает службам из Application2 использовать эту емкость, даже если в это время службы в Application1 не потребляют ресурсы. Зарезервированная для приложения емкость считается потребляемой и будет учитываться при определении остатка доступной емкости на этом узле или в кластере.  Зарезервированная емкость мгновенно вычитается из оставшейся емкости кластера, однако из емкости определенного узла она вычитается только в том случае, если в нем размещается по крайней мере один объект службы. Этот второй этап резервирования позволяет более гибко и более эффективно использовать ресурсы, поскольку на конкретных узлах резервирование происходит только по мере необходимости.

## <a name="obtaining-the-application-load-information"></a>Получение сведений о нагрузке приложения
Для каждого приложения с емкостью, определенной для одной или нескольких метрик, вы можете получить сведения о суммарной нагрузке, создаваемой репликами его служб.

PowerShell:

``` posh
Get-ServiceFabricApplicationLoadInformation –ApplicationName fabric:/MyApplication1
```

C#

``` csharp
var v = await fc.QueryManager.GetApplicationLoadInformationAsync("fabric:/MyApplication1");
var metrics = v.ApplicationLoadMetricInformation;
foreach (ApplicationLoadMetricInformation metric in metrics)
{
    Console.WriteLine(metric.ApplicationCapacity);  //total capacity for this metric in this application instance
    Console.WriteLine(metric.ReservationCapacity);  //reserved capacity for this metric in this application instance
    Console.WriteLine(metric.ApplicationLoad);  //current load for this metric in this application instance
}
```

Запрос ApplicationLoad возвращает основные сведения о параметрах емкости, указанных для приложения. Сюда входят данные о минимальном и максимальном числе узлов, а также о том, сколько узлов приложение занимает в настоящее время. Также вы получите информацию по каждой метрике нагрузки для приложения, в том числе перечисленные ниже сведения.

* Metric Name: имя метрики.
* Reservation Capacity: емкость, зарезервированная в кластере для этого приложения.
* Application Load: общая нагрузка, создаваемая дочерними репликами этого приложения.
* Application Capacity: максимально допустимое значение нагрузки приложения.

## <a name="removing-application-capacity"></a>Удаление емкости приложения
Когда для приложения будут установлены параметры емкости приложения, их можно будет удалить с помощью API-интерфейсов обновления приложения или командлетов PowerShell. Пример:

``` posh
Update-ServiceFabricApplication –Name fabric:/MyApplication1 –RemoveApplicationCapacity

```

Эта команда удаляет все параметры управления емкостью приложения, установленные для экземпляра приложения. Это относится к параметрам MinimumNodes, MaximumNodes и метрикам приложения, если они были настроены. Команда применяется немедленно. После выполнения этой команды диспетчер кластерных ресурсов использует режим по умолчанию для управления приложениями. Параметры емкости приложения можно задать снова, используя команду `Update-ServiceFabricApplication`/`System.Fabric.FabricClient.ApplicationManagementClient.UpdateApplicationAsync()`.

### <a name="restrictions-on-application-capacity"></a>Ограничения на емкость приложений
Существует ряд ограничений на параметры емкости приложений, которые необходимо соблюдать. При обнаружении ошибок во время проверки изменения не применяются.

- Все числовые параметры должны иметь неотрицательные значения.
- MinimumNodes не может иметь значение больше, чем MaximumNodes.
- Если определены емкости для метрики нагрузки, они должны соответствовать следующим правилам.
  - NodeReservationCapacity не может иметь значение больше, чем MaximumNodeCapacity. Например, вы не можете для метрики нагрузки на процессор на узле установить ограничение в две единицы и одновременно зарезервировать по три единицы этого ресурса на каждом узле.
  - Если указан параметр MaximumNodes, произведение значений MaximumNodes и MaximumNodeCapacity не может быть больше, чем значение TotalApplicationCapacity. Предположим, вы установили значение 8 для максимальной емкости узла для метрики нагрузки на процессор. При этом максимальное число узлов вы ограничили десятью. В этом случае общая емкость приложения для этой метрики нагрузки должна быть больше 80.

Ограничения применяются во время создания и обновления приложений.

## <a name="how-not-to-use-application-capacity"></a>Как не следует использовать емкость приложения
- Не пытайтесь с помощью групп приложений сделать так, чтобы приложение выполнялось на _определенном_ подмножестве узлов. Другими словами, вы можете указать, что для приложения нельзя использовать больше пяти узлов, но не можете выбрать, какие именно это будут узлы в кластере. Чтобы прикрепить приложение к конкретным узлам, используйте ограничения на размещение для служб.
- Не пытайтесь с помощью емкости приложения разместить две службы одного приложения на одном узле. Вместо этого используйте [сходство](service-fabric-cluster-resource-manager-advanced-placement-rules-affinity.md) или [ограничения размещения](service-fabric-cluster-resource-manager-cluster-description.md#node-properties-and-placement-constraints).

## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения о настройке служб см. в разделе [Настройка параметров Cluster Resource Manager для служб Service Fabric](service-fabric-cluster-resource-manager-configure-services.md).
- Чтобы узнать, как диспетчер кластерных ресурсов управляет нагрузкой кластера и балансирует ее, ознакомьтесь со статьей о [балансировке нагрузки](service-fabric-cluster-resource-manager-balancing.md)
- Начните с самого начала, [изучив общие сведения о диспетчере кластерных ресурсов Service Fabric](service-fabric-cluster-resource-manager-introduction.md)
- Дополнительные сведения об использовании метрик см. в статье [Управление потреблением ресурсов и нагрузкой в Service Fabric с помощью метрик](service-fabric-cluster-resource-manager-metrics.md).
- В Cluster Resource Manager предусмотрено много параметров для описания кластера. Чтобы узнать больше о них, ознакомьтесь с этой статьей, посвященной [описанию кластера Service Fabric](service-fabric-cluster-resource-manager-cluster-description.md)

[Image1]:./media/service-fabric-cluster-resource-manager-application-groups/application-groups-max-nodes.png
[Image2]:./media/service-fabric-cluster-resource-manager-application-groups/application-groups-reserved-capacity.png
