---
title: Политики размещения диспетчер ресурсов кластера Service Fabric
description: Общие сведения о дополнительных политиках размещения и правилах для служб Service Fabric
author: masnider
ms.topic: conceptual
ms.date: 08/18/2017
ms.author: masnider
ms.custom: devx-track-csharp
ms.openlocfilehash: 263e45928642aa74d682fc490e424a24deeb8076
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98790687"
---
# <a name="placement-policies-for-service-fabric-services"></a>Политики размещения для служб Service Fabric
Политики размещения — это дополнительные правила, с помощью которых можно управлять размещением служб в некоторых специальных, менее распространенных сценариях. Вот несколько примеров.

- Кластер Service Fabric охватывает значительные географические расстояния. Например, он распределен между несколькими локальными центрами обработки данных или регионами Azure.
- Среда охватывает несколько геополитических или юридических зон (или какая-либо другая ситуация, в которой нужно применять границы политик).
- Имеются различные вопросы производительность и задержки связи из-за больших расстояний либо использования медленных или менее надежных сетевых соединений.
- Необходимо по мере возможности обеспечить выравнивание определенных рабочих нагрузок относительно других рабочих нагрузок или вблизи от клиентов.
- Требуется несколько экземпляров секции без отслеживания состояния на одном узле

Большинство этих требований соответствуют физической структуре кластера, представленной в виде доменов сбоя кластера. 

В реализации таких сценариев могу помочь расширенные политики размещения:

1. с недопустимыми доменами;
2. с обязательными доменами;
3. с предпочтительными доменами;
4. при запрете упаковки реплик.
5. Разрешить несколько экземпляров без отслеживания состояния на узле

Большинство перечисленных ниже элементов управления можно настроить, используя свойства узла и ограничения на размещение, но бывают и более сложные случаи. Чтобы упростить процесс, диспетчер кластерных ресурсов Service Fabric предоставляет дополнительные политики размещения. Политики размещения настаиваются для отдельных экземпляров именованных служб. Они могут также обновляться динамически.

## <a name="specifying-invalid-domains"></a>Указание недопустимых доменов
Политика размещения **InvalidDomain** позволяет указать, что для данной службы определенный домен сбоя является недопустимым. Эта политика предотвращает выполнение определенной службы в определенном регионе, что может быть связано с геополитическими причинами или корпоративной политикой организации. Используя отдельные политики, можно указать несколько недопустимых доменов.

<center>

![Пример недопустимого домена][Image1]
</center>

Код:

```csharp
ServicePlacementInvalidDomainPolicyDescription invalidDomain = new ServicePlacementInvalidDomainPolicyDescription();
invalidDomain.DomainName = "fd:/DCEast"; //regulations prohibit this workload here
serviceDescription.PlacementPolicies.Add(invalidDomain);
```

PowerShell.

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 3 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("InvalidDomain,fd:/DCEast”)
```
## <a name="specifying-required-domains"></a>Указание обязательных доменов
Политика размещения обязательного домена требует, чтобы служба размещалась только в указанном домене. Используя отдельные политики, можно указать несколько обязательных доменов.

<center>

![Пример обязательного домена][Image2]
</center>

Код:

```csharp
ServicePlacementRequiredDomainPolicyDescription requiredDomain = new ServicePlacementRequiredDomainPolicyDescription();
requiredDomain.DomainName = "fd:/DC01/RK03/BL2";
serviceDescription.PlacementPolicies.Add(requiredDomain);
```

PowerShell.

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 3 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("RequiredDomain,fd:/DC01/RK03/BL2")
```

## <a name="specifying-a-preferred-domain-for-the-primary-replicas-of-a-stateful-service"></a>Указание предпочтительного домена для первичных реплик службы с отслеживанием состояния
Предпочтительный домен первичных реплик задает домен сбоя для размещения первичных реплик. Если все компоненты работоспособны, первичная реплика оказывается в этом домене. В случае сбоя или завершения работы домена или первичной реплики она перемещается в другое местоположение, в идеальном случае — в том же домене. Если это расположение находится не в предпочтительном домене, диспетчер кластерных ресурсов при первой возможности возвращает реплику в предпочтительный домен. Обычно этот параметр имеет смысл только для служб с отслеживанием состояния. Эту политику особенно полезно использовать в кластерах, которые охватывают несколько регионов Azure или центров обработки данных, но содержат службы, для которых предпочтительно размещение в определенном расположении. Размещение первичных реплик вблизи от их пользователей или других служб помогает снизить задержку, особенно для операций чтения, которые обрабатываются первичными репликами по умолчанию.

<center>

![Предпочтительные домены для первичных реплик и отработка отказа][Image3]
</center>

```csharp
ServicePlacementPreferPrimaryDomainPolicyDescription primaryDomain = new ServicePlacementPreferPrimaryDomainPolicyDescription();
primaryDomain.DomainName = "fd:/EastUS/";
serviceDescription.PlacementPolicies.Add(primaryDomain);
```

PowerShell.

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 3 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("PreferredPrimaryDomain,fd:/EastUS")
```

## <a name="requiring-replica-distribution-and-disallowing-packing"></a>Требование распределения реплик и запрещение группирования
_Обычно_ реплики распределены между доменами сбоя и обновления, когда кластер находится в работоспособном состоянии. Тем не менее существуют ситуации, в которых более одной реплики для заданной секции может быть временно сгруппировано в одном домене. Например, в кластере есть девять узлов в трех доменах сбоя (fd:/0, fd:/1 и fd:/2). Предположим, что у вашей службы три реплики. Предположим, что узлы, которые использовались для этих реплик в доменах fd:/1 и fd:/2, вышли из строя. В такой ситуации диспетчер кластерных ресурсов, как правило, использует другие узлы в этих же доменах сбоя. В нашем же примере, скажем, из-за проблем с емкостью все узлы в этих доменах являются недопустимыми. Если диспетчер кластерных ресурсов попытается заменить эти реплики, будут выбраны узлы в домене fd:/0. Но _тогда_ возникает ситуация, когда нарушается ограничение домена сбоя. Группирование реплик повышает вероятность того, что весь набор реплик выйдет из строя или будет утерян. 

> [!NOTE]
> Дополнительные сведения об общих ограничениях и приоритетах ограничений см. в [этом разделе](service-fabric-cluster-resource-manager-management-integration.md#constraint-priorities).
>

Если вы увидели сообщение о работоспособности наподобие такого: `The Load Balancer has detected a Constraint Violation for this Replica:fabric:/<some service name> Secondary Partition <some partition ID> is violating the Constraint: FaultDomain`, это значит, что у вас возникла описанная ошибка или сходная с ней. Обычно временно группируется только одна или две реплики. Пока число реплик в заданном домене не превышает кворум, все в порядке. Группирование происходит редко, но оно возможна. Как правило, такие ситуации являются временными, так как узлы возвращаются в работоспособное состояние. Если узлы остаются в неработоспособном состоянии, а диспетчеру кластерных ресурсов требуется выполнить замену, для этого в идеальной ситуации в кластерах сбоя доступны другие узлы.

Для некоторых рабочих нагрузок лучше всегда иметь целевое число реплик, даже если они сгруппированы в меньшем числе доменов. Такие рабочие нагрузки обеспечивают защиту от общего числа постоянных одновременных сбоев в домене, позволяя восстановить локальное состояние. А для других рабочих нагрузок предпочтительнее будет перейти в состояние простоя, чем подвергнуться риску потери данных или нарушения их целостности. Большинство производственных рабочих нагрузок работает больше чем с тремя репликами, больше чем с тремя доменами сбоя и множеством допустимых узлов в каждом домене сбоя. По этой причине группирование в доменах разрешено по умолчанию. Это поведение по умолчанию позволяет с помощью обычной балансировки и отработки отказа обрабатывать такие крайние случаи, даже если это означает временное группирование в домены.

Если вы хотите отключить такой способ группирования для определенной рабочей нагрузки, то для службы можно указать политику `RequireDomainDistribution`. Если задать эту политику, диспетчер кластерных ресурсов запретит выполнять две реплики из одной и той же секции в одном домене сбоя или обновления.

Код:

```csharp
ServicePlacementRequireDomainDistributionPolicyDescription distributeDomain = new ServicePlacementRequireDomainDistributionPolicyDescription();
serviceDescription.PlacementPolicies.Add(distributeDomain);
```

PowerShell.

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 3 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("RequiredDomainDistribution")
```

Можно ли использовать эти конфигурации для служб в кластере, который не распределен между разными географическими регионами? Можно, но не нужно. Конфигураций с обязательными, недопустимыми и предпочтительными доменами следует избегать, если только без них невозможно обойтись. Не имеет смысла пытаться принудительно запускать определенную рабочую нагрузку в одной стойке или использовать какой-либо сегмент локального кластера вместо другого. Разные аппаратные конфигурации следует распределить между доменами сбоя и обрабатывать с использованием обычных ограничений размещения и свойств узлов.

## <a name="placement-of-multiple-stateless-instances-of-a-partition-on-single-node"></a>Размещение нескольких экземпляров секции без отслеживания состояния на одном узле
Политика размещения **алловмултиплестателессинстанцесонноде** позволяет размещать несколько экземпляров секции без отслеживания состояния на одном узле. По умолчанию несколько экземпляров одной секции не могут быть помещены на узел. Даже при наличии службы-1 невозможно масштабировать количество экземпляров за пределами числа узлов в кластере для данной именованной службы. Эта политика размещения удаляет это ограничение и позволяет указывать InstanceCount выше, чем количество узлов.

Если вы увидели сообщение о работоспособности наподобие такого: `The Load Balancer has detected a Constraint Violation for this Replica:fabric:/<some service name> Secondary Partition <some partition ID> is violating the Constraint: ReplicaExclusion`, это значит, что у вас возникла описанная ошибка или сходная с ней. 

Указав `AllowMultipleStatelessInstancesOnNode` политику для службы, InstanceCount можно задать за пределами числа узлов в кластере.

Код:

```csharp
ServicePlacementAllowMultipleStatelessInstancesOnNodePolicyDescription allowMultipleInstances = new ServicePlacementAllowMultipleStatelessInstancesOnNodePolicyDescription();
serviceDescription.PlacementPolicies.Add(allowMultipleInstances);
```

PowerShell.

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName -Stateless –PartitionSchemeSingleton –PlacementPolicy @(“AllowMultipleStatelessInstancesOnNode”) -InstanceCount 10 -ServicePackageActivationMode ExclusiveProcess 
```

> [!NOTE]
> Политика размещения в настоящее время находится на этапе предварительной версии и находится за `EnableUnsupportedPreviewFeatures` параметром кластера. Так как это предварительная версия функции, установка конфигурации предварительного просмотра предотвращает обновление кластера до или с. Иными словами, необходимо создать новый кластер, чтобы испытать функцию.
>

> [!NOTE]
> В настоящее время политика поддерживается только для служб без отслеживания состояния с [режимом активации пакета службы](/dotnet/api/system.fabric.description.servicepackageactivationmode)ExclusiveProcess.
>

> [!WARNING]
> Политика не поддерживается при использовании с конечными точками статических портов. Использование обоих в сочетании может привести к неработоспособному кластеру, так как несколько экземпляров на одном узле пытаются выполнить привязку к одному и тому же порту и не могут быть получены. 
>

> [!NOTE]
> Использование высокого значения [мининстанцекаунт](/dotnet/api/system.fabric.description.statelessservicedescription.mininstancecount) с этой политикой размещения может привести к зависанию обновлений приложений. Например, если имеется кластер с пятью узлами и Set InstanceCount = 10, на каждом узле будет два экземпляра. Если задать Мининстанцекаунт = 9, попытка обновления приложения может быть задержана; с Мининстанцекаунт = 8 это можно избежать.
>

## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения о настройке служб см. в разделе [Настройка параметров Cluster Resource Manager для служб Service Fabric](service-fabric-cluster-resource-manager-configure-services.md).

[Image1]:./media/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies/cluster-invalid-placement-domain.png
[Image2]:./media/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies/cluster-required-placement-domain.png
[Image3]:./media/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies/cluster-preferred-primary-domain.png
