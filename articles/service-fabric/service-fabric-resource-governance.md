---
title: управление ресурсами для контейнеров и служб;
description: Azure Service Fabric позволяет указывать запросы ресурсов и ограничения для служб, выполняющихся в качестве процессов или контейнеров.
ms.topic: conceptual
ms.date: 8/9/2017
ms.openlocfilehash: d760766870c8c2be0a2d2384f6d012b75bc92fbd
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101735664"
---
# <a name="resource-governance"></a>Управление ресурсами

При запуске нескольких служб на одном узле или кластере может потребоваться, чтобы одна служба использовала больше ресурсов, неоставляя другие службы в процессе. Это также известно как проблема "шумного соседа". Azure Service Fabric позволяет разработчику управлять этим поведением, задавая запросы и ограничения на службу для ограничения использования ресурсов.

> Прежде чем продолжить работу с этой статьей, рекомендуется ознакомиться с [моделью приложения Service Fabric][application-model-link] и [моделью размещения Service Fabric][hosting-model-link].
>

## <a name="resource-governance-metrics"></a>Метрики управления ресурсами

Управление ресурсами поддерживается в Service Fabric в соответствии с каждым [пакетом службы][application-model-link]. Ресурсы, которые назначены пакету службы, могут быть дополнительно разделены между пакетами кода. Service Fabric поддерживает управление ЦП и памятью для каждого пакета службы с помощью двух встроенных [метрик](service-fabric-cluster-resource-manager-metrics.md):

* *ЦП* (имя метрики `servicefabric:/_CpuCores` ) — логическое ядро, доступное на хост-компьютере. Веса всех ядер для всех узлов считаются одинаковыми.

* *Память* (имя метрики `servicefabric:/_MemoryInMB` ): память выражается в мегабайтах и сопоставляется с физической памятью, доступной на компьютере.

Для этих двух метрик [кластерный диспетчер ресурсов (CRM)][cluster-resource-manager-description-link] отслеживает общую емкость кластера, нагрузку на каждый узел в кластере и остальные ресурсы в кластере. Эти две метрики аналогичны любой другой пользовательской или настраиваемой метрике. Их можно использовать для всех существующих функций:

* Кластер может быть [сбалансирован](service-fabric-cluster-resource-manager-balancing.md) в соответствии с этими двумя метриками (реакция на событие по умолчанию).
* Кластер может быть [дефрагментирован](service-fabric-cluster-resource-manager-defragmentation-metrics.md) в соответствии с этими двумя метриками.
* При [описании кластера][cluster-resource-manager-description-link] для этих двух метрик можно задать емкость буфера.

> [!NOTE]
> [Отчеты о динамической нагрузке](service-fabric-cluster-resource-manager-metrics.md) не поддерживаются для этих метрик. нагрузки для этих метрик определяются во время создания.

## <a name="resource-governance-mechanism"></a>Механизм управления ресурсами

Начиная с версии 7,2, среда выполнения Service Fabric поддерживает спецификацию запросов и ограничений для ресурсов ЦП и памяти.

> [!NOTE]
> Service Fabric версии среды выполнения, более ранние чем 7,2, поддерживают только модель, в которой одно значение является **запросом** и **ограничением** для определенного ресурса (ЦП или памяти). Это описывается в спецификации **рекуестсонли** в этом документе.

* *Запросы:* Значения запросов ЦП и памяти представляют нагрузки, используемые [кластером диспетчер ресурсов (CRM)][cluster-resource-manager-description-link] для `servicefabric:/_CpuCores` `servicefabric:/_MemoryInMB` метрик и. Иными словами, CRM считает потребление ресурсов службой равным ее значениям запроса и использует эти значения при принятии решений о размещении.

* *Ограничения:* Значения ограничений ЦП и памяти представляют фактические ограничения ресурсов, применяемые при активации процесса или контейнера на узле.

Service Fabric допускает использование **рекуестсонли, лимитсонли** и обеих спецификаций **рекуестсандлимитс** для ЦП и памяти.
* При использовании спецификации Рекуестсонли Service Fabric также использует значения запроса в качестве ограничений.
* При использовании спецификации Лимитсонли Service Fabric считает значения запросов равными 0.
* При использовании спецификации Рекуестсандлимитс значения ограничений должны быть больше или равны значениям запроса.

Чтобы лучше понять механизм управления ресурсами, рассмотрим пример сценария размещения со спецификацией **рекуестсонли** для ресурса ЦП (механизм управления памятью эквивалентен). Рассмотрим узел с двумя ядрами ЦП и двумя пакетами служб, которые будут размещены на нем. Первый пакет службы, который необходимо разместить, состоит из всего одного пакета кода контейнера и указывает только запрос одного ядра ЦП. Второй пакет службы, который необходимо разместить, состоит из всего одного пакета кода на основе процесса, который также указывает запрос только одного ядра ЦП. Так как оба пакета служб имеют спецификацию Рекуестсонли, их значения ограничений задаются для их значений запроса.

1. Сначала пакет службы на основе контейнера, запрашивающий один процессор, помещается на узел. Среда выполнения активирует контейнер и устанавливает ограничение ЦП на одно ядро. Этот контейнер не сможет использовать более 1 ядра.

2. Затем пакет службы на основе процесса, запрашивающий один процессор, помещается на узел. Среда выполнения активирует процесс службы и устанавливает для его ограничения ЦП одно ядро.

На этом этапе сумма запросов равна емкости узла. CRM не будет размещать больше контейнеров или процессов службы с запросами ЦП на этом узле. На узле процесс и контейнер выполняются с одним ядром и не будут конкурировать друг с другом для ЦП.

Теперь давайте перейдем к нашему примеру со спецификацией **рекуестсандлимитс** . На этот раз пакет службы на основе контейнера указывает запрос одного ядра ЦП и ограничение двух ядер ЦП. Пакет службы на основе процесса задает как запрос, так и ограничение одного ядра ЦП.
  1. Сначала пакет службы на основе контейнера помещается на узел. Среда выполнения активирует контейнер и устанавливает ограничение ЦП на два ядра. Контейнер не сможет использовать более двух ядер.
  2. Затем пакет службы на основе процесса размещается на узле. Среда выполнения активирует процесс службы и устанавливает для его ограничения ЦП одно ядро.

  На этом этапе сумма запросов ЦП для пакетов служб, размещенных на узле, равна емкости ЦП узла. CRM не будет размещать больше контейнеров или процессов службы с запросами ЦП на этом узле. Однако на узле сумма ограничений (двух ядер для контейнера и одного ядра для процесса) превышает емкость двух ядер. Если контейнер и процесс находятся в одном и том же времени, существует вероятность состязаний за ресурсы ЦП. Такое состязание будет осуществляться с помощью базовой ОС для платформы. В этом примере контейнер может разгрузить до двух ядер ЦП, в результате чего процесс запроса одного ядра ЦП не гарантируется.

> [!NOTE]
> Как показано в предыдущем примере, значения запроса ЦП и памяти не приводят **к резервированию ресурсов на узле**. Эти значения представляют потребление ресурсов, которое кластер диспетчер ресурсов считает при принятии решений о размещении. Ограничения значений представляют фактические ограничения ресурсов, применяемые при активации процесса или контейнера на узле.


Существует несколько ситуаций, в которых может возникнуть состязание за использование ЦП. В таких ситуациях процесс и контейнер из нашего примера могут столкнуться с проблемой бесшумного соседа:

* *Сочетание управляемых и не управляемых служб и контейнеров*: если пользователь создает службу и не указывает управление ресурсами, среда выполнения решит, что она не использует ресурсы, и разместит ее на узле из нашего примера. В таком случае этот новый процесс будет активно использовать некоторые ресурсы ЦП за счет служб, которые уже запущены на узле. Существует два решения этой проблемы. Для этого не следует смешивать управляемые и не управляемые службы в одном кластере или следует использовать [ограничения размещения](service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies.md), чтобы службы этих двух типов использовали разные наборы узлов.

* *Когда на узле запущен другой процесс, который не относится к процессам Service Fabric (например, служба ОС)*: в этой ситуации процесс извне Service Fabric также будет состязаться за ресурсы ЦП с имеющимися службами. Избежать этой проблемы можно, правильно настроив емкость узла с учетом дополнительной нагрузки на ОС, как описано в разделе ниже.

* *Если запросы не равны ограничениям*: как описано в примере рекуестсандлимитс ранее, запросы не приводят к резервированию ресурсов на узле. Если служба с ограничениями, превышающими запросы, размещается на узле, она может потреблять ресурсы (если они доступны) до ограничений. В таких случаях другие службы на узле могут не иметь возможности использовать ресурсы вплоть до значений их запросов.

## <a name="cluster-setup-for-enabling-resource-governance"></a>Настройка кластера для включения управления ресурсами

Когда узел запустится и присоединится к кластеру, Service Fabric обнаружит доступный объем памяти и доступное количество ядер и задаст емкости узла для этих двух ресурсов.

Чтобы оставить место в буфере для операционной системы, а также для других процессов, которые могут выполняться на узле, Service Fabric использует только 80% доступных ресурсов на узле. Это процентное значение можно настроить и изменить в манифесте кластера.

Ниже приведен пример, как настроить Service Fabric для использования 50 % от доступных ресурсов ЦП и 70 % от доступной памяти:

```xml
<Section Name="PlacementAndLoadBalancing">
    <!-- 0.0 means 0%, and 1.0 means 100%-->
    <Parameter Name="CpuPercentageNodeCapacity" Value="0.5" />
    <Parameter Name="MemoryPercentageNodeCapacity" Value="0.7" />
</Section>
```

Для большинства клиентов и сценариев рекомендуется использовать автоматическое обнаружение емкости узлов ЦП и памяти (автоматическое обнаружение включено по умолчанию). Однако если требуется полная настройка емкости узлов вручную, их можно настроить для каждого типа узла с помощью механизма описания узлов в кластере. Ниже приведен пример настройки типа узла с четырьмя ядрами и 2 ГБ памяти:

```xml
    <NodeType Name="MyNodeType">
      <Capacities>
        <Capacity Name="servicefabric:/_CpuCores" Value="4"/>
        <Capacity Name="servicefabric:/_MemoryInMB" Value="2048"/>
      </Capacities>
    </NodeType>
```

Если включено автоматическое обнаружение доступных ресурсов и в манифесте кластера емкости узлов определены вручную, Service Fabric проверит, достаточно ли в узле ресурсов для поддержки определенной пользователем емкости.

* Если емкостей узлов, определенных в манифесте, недостаточно для доступных на узле ресурсов (или же хватает), Service Fabric будет использовать емкости, указанные в манифесте.

* Если емкости узлов, определенные в манифесте, превышают количество доступных ресурсов, Service Fabric будет использовать эти доступные ресурсы как емкости узла.

Если автоматическое обнаружение доступных ресурсов не требуется, его можно отключить. Для этого измените приведенный ниже параметр.

```xml
<Section Name="PlacementAndLoadBalancing">
    <Parameter Name="AutoDetectAvailableResources" Value="false" />
</Section>
```

Для достижения оптимальной производительности следующий параметр также должен быть включен в манифесте кластера:

```xml
<Section Name="PlacementAndLoadBalancing">
    <Parameter Name="PreventTransientOvercommit" Value="true" />
    <Parameter Name="AllowConstraintCheckFixesDuringApplicationUpgrade" Value="true" />
</Section>
```

> [!IMPORTANT]
> Начиная с версии Service Fabric 7,0 мы обновили правило для вычисления емкости ресурсов узла в случаях, когда пользователь вручную предоставляет значения для емкости ресурсов узла. Рассмотрим следующий сценарий:
>
> * На узле всего 10 ядер ЦП.
> * SF настроен на использование 80% от общего количества ресурсов для пользовательских служб (значение по умолчанию), что оставляет буфер на 20% для других служб, работающих на узле (включая Service Fabric системные службы).
> * Пользователь решает вручную переопределить емкость ресурса узла для метрики ядра ЦП и установит для нее 5 ядер.
>
> Мы изменили правило того, как доступная емкость для пользовательских служб Service Fabric вычисляется следующим образом:
>
> * До Service Fabric 7,0, доступная емкость для пользовательских служб будет рассчитана на **5 ядер** (буфер емкости 20% игнорируется).
> * Начиная с Service Fabric 7,0, доступная емкость для пользовательских служб будет рассчитана на **4 ядра** (буфер емкости, равный 20%, не пропускается).

## <a name="specify-resource-governance"></a>Настройка управления ресурсами

Запросы и ограничения управления ресурсами указываются в манифесте приложения (раздел ServiceManifestImport). Рассмотрим несколько примеров:

**Пример 1. Спецификация Рекуестсонли**
```xml
<?xml version='1.0' encoding='UTF-8'?>
<ApplicationManifest ApplicationTypeName='TestAppTC1' ApplicationTypeVersion='vTC1' xsi:schemaLocation='http://schemas.microsoft.com/2011/01/fabric ServiceFabricServiceModel.xsd' xmlns='http://schemas.microsoft.com/2011/01/fabric' xmlns:xsi='https://www.w3.org/2001/XMLSchema-instance'>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName='ServicePackageA' ServiceManifestVersion='v1'/>
    <Policies>
      <ServicePackageResourceGovernancePolicy CpuCores="1"/>
      <ResourceGovernancePolicy CodePackageRef="CodeA1" CpuShares="512" MemoryInMB="1024" />
      <ResourceGovernancePolicy CodePackageRef="CodeA2" CpuShares="256" MemoryInMB="1024" />
    </Policies>
  </ServiceManifestImport>
```

В этом примере `CpuCores` атрибут используется для указания запроса 1 ядра ЦП для **ServicePackageA**. Так как ограничение ЦП ( `CpuCoresLimit` атрибут) не указано, Service Fabric также использует указанное значение запроса 1 ядро в качестве ограничения ЦП для пакета службы.

**ServicePackageA** будет размещен только на узле, где оставшийся объем ЦП после вычитания **суммы запросов ЦП всех пакетов служб, размещенных на этом узле,** больше или равна 1 ядру. На узле пакет службы будет ограничен одним ядром. Пакет службы содержит два пакета кода (**CodeA1** и **CodeA2**), и оба указывают `CpuShares` атрибут. Доля CpuShares 512:256 используется для вычисления ограничений на ЦП для отдельных пакетов кода. Таким же CodeA1 будет ограничена двумя третьями ядра, а CodeA2 будет ограничен третьим из них. Если CpuShares не указаны для всех пакетов кода, Service Fabric делит ограничение на ЦП между ними.

Хотя CpuShares, указанные для пакетов кода, представляют их относительную долю общего количества ЦП пакета службы, значения памяти для пакетов кода указываются в абсолютных терминах. В этом примере `MemoryInMB` атрибут используется для указания запросов памяти размером 1024 МБ для CodeA1 и CodeA2. Так как ограничение памяти ( `MemoryInMBLimit` атрибут) не указано, Service Fabric также использует указанные значения запроса в качестве ограничений для пакетов кода. Запрос памяти (и ограничение) для пакета службы вычисляется как сумма значений запросов (и ограничений) памяти составляющих его пакетов кода. Таким образом, для **ServicePackageA** запрос и ограничение памяти рассчитываются как 2048 МБ.

**ServicePackageA** будет размещен только на узле, где оставшийся объем памяти после вычитания **суммы запросов к памяти для всех пакетов служб, размещенных на этом узле,** больше или равна 2048 МБ. На узле оба пакета кода будут ограничены 1024 МБ памяти. Пакеты кода (контейнеры или процессы) не смогут выделить больше памяти, чем это ограничение, и при попытке сделать это приведет к возникновению исключений нехватки памяти.

**Пример 2. Спецификация Лимитсонли**
```xml
<?xml version='1.0' encoding='UTF-8'?>
<ApplicationManifest ApplicationTypeName='TestAppTC1' ApplicationTypeVersion='vTC1' xsi:schemaLocation='http://schemas.microsoft.com/2011/01/fabric ServiceFabricServiceModel.xsd' xmlns='http://schemas.microsoft.com/2011/01/fabric' xmlns:xsi='https://www.w3.org/2001/XMLSchema-instance'>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName='ServicePackageA' ServiceManifestVersion='v1'/>
    <Policies>
      <ServicePackageResourceGovernancePolicy CpuCoresLimit="1"/>
      <ResourceGovernancePolicy CodePackageRef="CodeA1" CpuShares="512" MemoryInMBLimit="1024" />
      <ResourceGovernancePolicy CodePackageRef="CodeA2" CpuShares="256" MemoryInMBLimit="1024" />
    </Policies>
  </ServiceManifestImport>
```
В этом примере `CpuCoresLimit` используются `MemoryInMBLimit` атрибуты и, которые доступны только в версии 7,2 и более поздних версиях. Атрибут Кпукореслимит используется для указания ограничения ЦП, равного 1 ядру для **ServicePackageA**. Поскольку запрос ЦП ( `CpuCores` атрибут) не указан, он считается равным 0. `MemoryInMBLimit` атрибут используется для указания пределов памяти 1024 МБ для CodeA1 и CodeA2, а также, поскольку запросы ( `MemoryInMB` атрибут) не указаны, они считаются равными 0. Таким образом, запрос и ограничение памяти для **ServicePackageA** вычисляются как 0 и 2048 соответственно. Так как запросы ЦП и памяти для **ServicePackageA** равны 0, они не представляют нагрузки для CRM, которые следует учитывать при размещении, для `servicefabric:/_CpuCores` `servicefabric:/_MemoryInMB` метрик и. Таким образом, с точки зрения управления ресурсами **ServicePackageA** можно разместить на любом узле **независимо от оставшейся емкости**. Как и в примере 1, на узле CodeA1 будет ограничен двумя третьями ядра и 1024 МБ памяти, а CodeA2 будет ограничен третьим размером ядра и 1024 МБ памяти.

**Пример 3. Спецификация Рекуестсандлимитс**
```xml
<?xml version='1.0' encoding='UTF-8'?>
<ApplicationManifest ApplicationTypeName='TestAppTC1' ApplicationTypeVersion='vTC1' xsi:schemaLocation='http://schemas.microsoft.com/2011/01/fabric ServiceFabricServiceModel.xsd' xmlns='http://schemas.microsoft.com/2011/01/fabric' xmlns:xsi='https://www.w3.org/2001/XMLSchema-instance'>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName='ServicePackageA' ServiceManifestVersion='v1'/>
    <Policies>
      <ServicePackageResourceGovernancePolicy CpuCores="1" CpuCoresLimit="2"/>
      <ResourceGovernancePolicy CodePackageRef="CodeA1" CpuShares="512" MemoryInMB="1024" MemoryInMBLimit="3072" />
      <ResourceGovernancePolicy CodePackageRef="CodeA2" CpuShares="256" MemoryInMB="2048" MemoryInMBLimit="4096" />
    </Policies>
  </ServiceManifestImport>
```
Основываясь на первых двух примерах, в этом примере демонстрируется указание как запросов, так и ограничений для ЦП и памяти. **ServicePackageA** имеет запросы ЦП и памяти 1 ядра и 3072 (1024 + 2048) МБ соответственно. Его можно разместить только на узле, где осталось по крайней мере 1 ядро (и 3072 МБ) пропускной способности после вычитания суммы всех запросов ЦП (и памяти) всех пакетов служб, размещенных на узле, от общей емкости ЦП (и памяти) узла. На узле CodeA1 будет ограничен двумя третьями из 2 ядер и 3072 МБ памяти, тогда как CodeA2 будет ограничено одной третью от 2 ядер и 4096 МБ памяти.

### <a name="using-application-parameters"></a>Использование параметров приложения

При указании параметров управления ресурсами можно использовать [Параметры приложения](service-fabric-manage-multiple-environment-app-configuration.md) , чтобы управлять несколькими конфигурациями приложений. Ниже приведен пример с использованием параметров приложения.

```xml
<?xml version='1.0' encoding='UTF-8'?>
<ApplicationManifest ApplicationTypeName='TestAppTC1' ApplicationTypeVersion='vTC1' xsi:schemaLocation='http://schemas.microsoft.com/2011/01/fabric ServiceFabricServiceModel.xsd' xmlns='http://schemas.microsoft.com/2011/01/fabric' xmlns:xsi='https://www.w3.org/2001/XMLSchema-instance'>

  <Parameters>
    <Parameter Name="CpuCores" DefaultValue="4" />
    <Parameter Name="CpuSharesA" DefaultValue="512" />
    <Parameter Name="CpuSharesB" DefaultValue="512" />
    <Parameter Name="MemoryA" DefaultValue="2048" />
    <Parameter Name="MemoryB" DefaultValue="2048" />
  </Parameters>

  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName='ServicePackageA' ServiceManifestVersion='v1'/>
    <Policies>
      <ServicePackageResourceGovernancePolicy CpuCores="[CpuCores]"/>
      <ResourceGovernancePolicy CodePackageRef="CodeA1" CpuShares="[CpuSharesA]" MemoryInMB="[MemoryA]" />
      <ResourceGovernancePolicy CodePackageRef="CodeA2" CpuShares="[CpuSharesB]" MemoryInMB="[MemoryB]" />
    </Policies>
  </ServiceManifestImport>
```

В этом примере задаются значения параметров по умолчанию для рабочей среды, в которой каждому пакету службы выделяются 4 ядра и 2 ГБ памяти. Значения по умолчанию можно изменить с помощью файлов параметров приложения. В этом примере один файл параметров может использоваться для тестирования приложения в локальной среде, в которой ему предоставляется меньше ресурсов, чем в рабочей среде.

```xml
<!-- ApplicationParameters\Local.xml -->

<Application Name="fabric:/TestApplication1" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="CpuCores" DefaultValue="2" />
    <Parameter Name="CpuSharesA" DefaultValue="512" />
    <Parameter Name="CpuSharesB" DefaultValue="512" />
    <Parameter Name="MemoryA" DefaultValue="1024" />
    <Parameter Name="MemoryB" DefaultValue="1024" />
  </Parameters>
</Application>
```

> [!IMPORTANT]
> Указание системы управления ресурсами с помощью параметров приложения доступно, начиная с Service Fabric версии 6.1.<br>
>
> Если параметры приложения используются для указания системы управления ресурсами, то невозможно перейти на использование версии Service Fabric, более ранней, чем версия 6.1.

## <a name="enforcing-the-resource-limits-for-user-services"></a>Обеспечение ограничений ресурсов для пользовательских служб

При применении управления ресурсами к Service Fabricным службам гарантируется, что эти управляемые ресурсом службы не смогут превышать квоту ресурсов, многие пользователи все равно должны запускать некоторые из своих Service Fabric служб в неуправляемом режиме. При использовании неуправляемых Service Fabric служб можно выполнять ситуации, когда "неконтролируемые" неуправляемые службы потребляют все доступные ресурсы на узлах Service Fabric, что может привести к серьезным проблемам, например:

* Нехватка ресурсов других служб, работающих на узлах (включая Service Fabric системные службы)
* Узлы, которые завершаются в неработоспособном состоянии
* Не отвечающие Service Fabric API управления кластерами

Чтобы предотвратить возникновение таких ситуаций, Service Fabric позволяет *применять ограничения ресурсов для всех Service Fabric пользовательских служб, выполняемых на узле* (как управляемых, так и неуправляемых), чтобы гарантировать, что пользовательские службы никогда не будут использовать больше указанного объема ресурсов. Это достигается путем установки значения ЕнфорцеусерсервицеметриккапаЦитиес config в разделе ПлацементандлоадбаланЦинг параметра ClusterManifest в значение true. По умолчанию этот параметр отключен.

```xml
<SectionName="PlacementAndLoadBalancing">
    <ParameterName="EnforceUserServiceMetricCapacities" Value="false"/>
</Section>
```

Дополнительные замечания:

* Ограничение использования ресурсов применяется только к `servicefabric:/_CpuCores` `servicefabric:/_MemoryInMB` метрикам ресурсов и
* Ограничение ресурсов работает только в том случае, если емкость узла для метрик ресурсов доступна для Service Fabric либо с помощью механизма автоматического обнаружения, либо через пользователей вручную, указав емкость узла (как описано в разделе [Настройка кластера для включения управления ресурсами](service-fabric-resource-governance.md#cluster-setup-for-enabling-resource-governance) ). Если емкость узла не настроена, возможность применения ограничения ресурсов не может быть использована, так как Service Fabric не может понять, сколько ресурсов резервируется для пользовательских служб. Service Fabric выдаст предупреждение о работоспособности, если значение "ЕнфорцеусерсервицеметриккапаЦитиес" равно true, а емкость узлов не настроена.

## <a name="other-resources-for-containers"></a>Другие ресурсы для контейнеров

Помимо ограничений ресурсов ЦП и памяти можно указать другие ограничения ресурсов для контейнеров. Эти ограничения указываются на уровне пакета кода и будут применены при запуске контейнера. В отличие от ограничений ресурсов ЦП и памяти, диспетчер кластерных ресурсов не будет учитывать эти ресурсы и не будет выполнять проверки емкости или балансировку нагрузки для них.

* *MemorySwapInMB* — объем памяти подкачки, которую может использовать контейнер.
* *MemoryReservationInMB* — мягкое ограничение по управлению памятью, которое применяется, только когда на узле обнаруживается состязание за память.
* *CpuPercent* — процент ресурсов ЦП, который может использовать контейнер. Если для пакета службы заданы запросы или ограничения ЦП, этот параметр фактически игнорируется.
* *MaximumIOps* — максимальное количество операций ввода-вывода, которое может использовать контейнер (для чтения и записи).
* *MaximumIOBytesps* — максимальное количество операций ввода-вывода (байт в секунду), которое может использовать контейнер (для чтения и записи).
* *BlockIOWeight* — число операций ввода-вывода в блоке по отношению к другим контейнерам.

Эти ресурсы можно объединять с ресурсами ЦП и памяти. Ниже приведен пример определения дополнительных ресурсов для контейнеров:

```xml
    <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="FrontendServicePackage" ServiceManifestVersion="1.0"/>
        <Policies>
            <ResourceGovernancePolicy CodePackageRef="FrontendService.Code" CpuPercent="5"
            MemorySwapInMB="4084" MemoryReservationInMB="1024" MaximumIOPS="20" />
        </Policies>
    </ServiceManifestImport>
```

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о диспетчере кластерных ресурсов см. в статье [Общие сведения о диспетчере кластерных ресурсов Service Fabric](service-fabric-cluster-resource-manager-introduction.md).
* Дополнительные сведения о модели приложения, пакетах служб и пакетах кода см. в статье [Моделирование приложения в структуре службы][application-model-link].

<!-- Links -->
[application-model-link]: service-fabric-application-model.md
[hosting-model-link]: service-fabric-hosting-model.md
[cluster-resource-manager-description-link]: service-fabric-cluster-resource-manager-cluster-description.md
