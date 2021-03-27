---
title: Поиск и устранение неполадок с помощью отчетов о работоспособности системы
description: Содержит описание отчетов о работоспособности, отправляемых компонентами Azure Service Fabric, и их использовании для устранения неполадок с кластерами и приложениями.
ms.topic: conceptual
ms.date: 2/28/2018
ms.openlocfilehash: 483483746b2cce66588e9481bca7e0de391070b8
ms.sourcegitcommit: a9ce1da049c019c86063acf442bb13f5a0dde213
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/27/2021
ms.locfileid: "105625897"
---
# <a name="use-system-health-reports-to-troubleshoot"></a>Устранение неполадок с помощью отчетов о работоспособности системы
Компоненты Azure Service Fabric предоставляют готовые системные отчеты о работоспособности для всех сущностей в кластере. В [хранилище данных о работоспособности](service-fabric-health-introduction.md#health-store) сущности создаются и удаляются на основании отчетов системы. Кроме того, эти сущности упорядочиваются в иерархию с учетом взаимодействия между ними.

> [!NOTE]
> Чтобы лучше разобраться в основных понятиях, связанных с работоспособностью, см. статью [Общие сведения о наблюдении за работоспособностью системы в Service Fabric](service-fabric-health-introduction.md).
> 
> 

В системных отчетах о работоспособности отражаются показатели функциональности кластеров и приложений, а также отмечаются проблемы. Что касается приложений и служб, отчеты о работоспособности системы проверяют правильность реализации и поведения сущностей на уровне платформы Service Fabric. Отчеты не обеспечивают наблюдение за работоспособностью бизнес-логики службы или обнаружение процессов, которые не отвечают. Пользовательские службы могут расширить данные о работоспособности сведениями, характерными для их логики.

> [!NOTE]
> Отчеты о работоспособности, отправляемые модулями наблюдения, отображаются только *после* того, как компоненты системы создают сущность. При удалении сущности хранилище данных о работоспособности автоматически удаляет все связанные с ней отчеты о работоспособности. Это же происходит при создании нового экземпляра сущности (например, нового экземпляра реплики хранимой службы с отслеживанием состояния). Все отчеты, связанные со старым экземпляром, полностью удаляются из хранилища.
> 
> 

На отчеты о компонентах системы указывает их источник, который начинается с префикса **System**  c префиксом "group.". Устройства наблюдения не могут использовать такой же префикс для своих источников, так как из-за недопустимых параметров отчеты отклоняются.

Рассмотрим некоторые системные отчеты, чтобы понять, как они активируются, и узнаем, как устранить возможные проблемы, отраженные в них.

> [!NOTE]
> Платформа Service Fabric постоянно добавляет отчеты на основе заданных условий, которые позволяют лучше понять, что происходит в кластере и приложениях. Существующие отчеты можно дополнить более подробными данными, чтобы ускорить устранение проблем.
> 
> 

## <a name="cluster-system-health-reports"></a>Системные отчеты о работоспособности кластеров
В хранилище данных о работоспособности автоматически создается сущность работоспособности кластера. Если все работает надлежащим образом, в этом хранилище нет системного отчета.

### <a name="neighborhood-loss"></a>Потеря окружения
**System.Federation** сообщает об ошибке при обнаружении потери окружения. Отчеты приходят с отдельных узлов. В имя свойства включается идентификатор узла. Если во всей кольцевой сети Service Fabric теряется одно окружение, могут произойти два события (отчеты создаются по обе стороны разрыва). Если теряется несколько окружений, происходит больше событий.

В отчете в качестве срока жизни указывается время ожидания глобальной аренды (TTL). Отчет повторно отправляется дважды в течение срока жизни, пока условие актуально. Событие автоматически удаляется по истечении срока его действия. Такая реакция на событие обеспечивает правильное удаление отчетов из хранилища данных о работоспособности даже в том случае, если узел, с которого отправляется отчет, не работает.

* **SourceId**: System.Federation.
* **Свойство**: начинается с **окружения** и включает сведения об узле.
* **Дальнейшие действия:** выясните, почему теряется окружение (например, проверьте связь между узлами кластера).

### <a name="rebuild"></a>Перестроение

Служба "Диспетчер отработки отказов" (FM) управляет сведениями об узлах кластера. Если диспетчер отработки отказов теряет свои данные и переходит к потере данных, он не может гарантировать наличие актуальных сведений об узлах кластера. В этом случае система проходит через перестройку и System.FM собирает данные со всех узлов в кластере, чтобы перестроить состояние. Иногда из-за проблем с сетью или узлами при перестроении могут возникнуть сбои или приостановка процесса. Это же может произойти со службой мастера диспетчера отработки отказов (FMM). Мастер диспетчера отработки отказов — это системная служба без учета состояния, отслеживающая расположение всех диспетчеров отработки отказов в кластере. Мастер диспетчера отработки отказов — это всегда узел с идентификатором, наиболее близким к 0. При удалении этого узла активируется перестройка.
**System.FM** или **System.FMM** помечают возникновение одного из предыдущих условий с помощью отчета об ошибке. Перестроение может зависнуть на одном из двух этапов.

* **Ожидание трансляции**. Диспетчер отработки отказов или мастер диспетчера отработки отказов ожидают ответные широковещательные сообщения от других узлов.

  * **Дальнейшие действия.** Проверьте, нет ли проблем с сетевым подключением между узлами.
* **Ожидание узлов**. Диспетчер отработки отказов или мастер диспетчера отработки отказов уже получили ответ трансляции от других узлов и ожидают ответа от указанных узлов. В отчете о работоспособности перечислены узлы, для которых диспетчер отработки отказов или мастер диспетчера отработки отказов ожидают ответ.
   * **Дальнейшие действия.** Проверьте сетевое подключение между диспетчером отработки отказов или мастером диспетчера отработки отказов и указанными узлами. Исследуйте каждый узел, указанный для других возможных проблем.

* **Идентификатор источника**: System.FM или System.FMM.
* **Свойство**: перестроение.
* **Дальнейшие действия**. Проверьте сетевое подключение между узлами, а также состояние какого-либо определенного узла, указанного в описании отчета о работоспособности.

### <a name="seed-node-status"></a>Состояние начального узла
**System.FM** сообщает о предупреждении уровня кластера, если некоторые начальные узлы неработоспособны. Начальные узлы — это узлы, которые поддерживают доступность базового кластера. Эти узлы гарантируют работу кластера, арендуя другие узлы и выступая в роли прерывателя связи во время определенных сбоев сети. Если большинство начальных узлов в кластере отключены и не возвращаются, кластер автоматически завершает работу. 

Начальный узел неработоспособен, если его состояние узла — "отключено", "удалено" или "неизвестно".
В отчете "предупреждение" для состояния узла начального значения будут перечислены все неработоспособные узлы начальных значений с подробными сведениями.

* **SourceID**: System.FM
* **Свойство**: сиднодестатус
* **Дальнейшие действия**. Если это предупреждение отображается в кластере, следуйте приведенным ниже инструкциям по устранению неполадки. для кластера, работающего Service Fabric версии 6,5 или более поздней: для кластера Service Fabric в Azure после отключения начального узла Service Fabric попытается автоматически изменить его на узел, не являющийся начальным. Чтобы сделать это, убедитесь, что число узлов, не являющихся начальными, в типе первичного узла больше или равно числу начальных узлов. При необходимости добавьте дополнительные узлы в тип первичного узла, чтобы добиться этого.
В зависимости от состояния кластера для устранения проблемы может потребоваться некоторое время. После этого отчет о предупреждениях будет автоматически сброшен.

Для Service Fabric автономного кластера, чтобы очистить отчет о предупреждениях, все начальные узлы должны стать работоспособными. В зависимости от того, почему начальные узлы являются неработоспособными, необходимо выполнить различные действия. Если начальный узел не работает, пользователи должны вернуть этот узел в исходное состояние. Если начальный узел удален или неизвестен, этот начальный узел необходимо [удалить из кластера](./service-fabric-cluster-windows-server-add-remove-nodes.md).
Отчет о предупреждениях автоматически удаляется, когда все начальные узлы становятся работоспособными.

Для кластера под управлением Service Fabric версии более 6,5: в этом случае отчет о предупреждениях необходимо очистить вручную. **Прежде чем очищать отчет, пользователи должны убедиться в работоспособности всех начальных узлов**: если начальный узел не работает, пользователи должны вернуть этот узел в исходное состояние. Если начальный узел удален или неизвестен, этот начальный узел необходимо удалить из кластера.
После того как все начальные узлы станут работоспособными, используйте следующую команду из PowerShell, чтобы [очистить отчет о предупреждениях](/powershell/module/servicefabric/send-servicefabricclusterhealthreport):

```powershell
PS C:\> Send-ServiceFabricClusterHealthReport -SourceId "System.FM" -HealthProperty "SeedNodeStatus" -HealthState OK

## Node system health reports
System.FM, which represents the Failover Manager service, is the authority that manages information about cluster nodes. Each node should have one report from System.FM showing its state. The node entities are removed when the node state is removed. For more information, see [RemoveNodeStateAsync](/dotnet/api/system.fabric.fabricclient.clustermanagementclient.removenodestateasync).

### Node up/down
System.FM reports as OK when the node joins the ring (it's up and running). It reports an error when the node departs the ring (it's down, either for upgrading or simply because it has failed). The health hierarchy built by the health store acts on deployed entities in correlation with System.FM node reports. It considers the node a virtual parent of all deployed entities. The deployed entities on that node are exposed through queries if the node is reported as up by System.FM, with the same instance as the instance associated with the entities. When System.FM reports that the node is down or restarted, as a new instance, the health store automatically cleans up the deployed entities that can exist only on the down node or on the previous instance of the node.

* **SourceId**: System.FM
* **Property**: State.
* **Next steps**: If the node is down for an upgrade, it should come back up after it's been upgraded. In this case, the health state should switch back to OK. If the node doesn't come back or it fails, the problem needs more investigation.

The following example shows the System.FM event with a health state of OK for node up:

```powershell
PS C:\> Get-ServiceFabricNodeHealth  _Node_0

NodeName              : _Node_0
AggregatedHealthState : Ok
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 8
                        SentAt                : 7/14/2017 4:54:51 PM
                        ReceivedAt            : 7/14/2017 4:55:14 PM
                        TTL                   : Infinite
                        Description           : Fabric node is up.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/14/2017 4:55:14 PM, LastWarning = 1/1/0001 12:00:00 AM
```


### <a name="certificate-expiration"></a>Истечение срока действия сертификата
**System.FabricNode** выдает предупреждение, когда приближается истечение срока действия сертификатов, используемых узлом. У каждого узла есть три сертификата: **Certificate_cluster**, **Certificate_server** и **Certificate_default_client**. Если сертификат остается действительным по крайней мере еще в течение двух недель, в отчете сообщается об отсутствии проблем. Если срок действия истекает в течение двух недель, отображается отчет с предупреждением. Срок жизни этих событий не ограничен. Они удаляются при выходе узла из кластера.

* **SourceId**: System.FabricNode.
* **Свойство**: начинается с **Certificate** и содержит дополнительные сведения о типе сертификата.
* **Дальнейшие действия**: обновите сертификаты, если срок их действия вскоре истечет.

### <a name="load-capacity-violation"></a>Нарушение емкости для загрузки
Подсистема балансировки нагрузки Service Fabric выдает предупреждение, если обнаруживает нарушение емкости узла.

* **SourceId**: System.PLB.
* **Свойство**: начинается с **Capacity**.
* **Дальнейшие действия**: проверьте предоставленные метрики и просмотрите данные о текущей емкости на узле.

### <a name="node-capacity-mismatch-for-resource-governance-metrics"></a>Несоответствие емкости узла и метрик управления ресурсами
System.Hosting выдает предупреждение, если определенные емкости узлов в манифесте кластера превышают реальные емкости узлов для метрик управления ресурсами (память и количество ядер ЦП). Отчет о работоспособности отобразится, когда первый пакет службы, использующий [систему управления ресурсами](service-fabric-resource-governance.md), зарегистрируется на указанном узле.

* **SourceId**: System.Hosting.
* **Свойство:****ResourceGovernance**.
* **Дальнейшие действия:** может возникнут проблема, так как регулирующие пакеты службы не выполняются должным образом и [система управления ресурсами](service-fabric-resource-governance.md) работает неправильно. Обновите манифест кластера, используя правильные значения емкости узлов для этих метрик, или не указывайте их, тогда Service Fabric автоматически обнаружит доступные ресурсы.

## <a name="application-system-health-reports"></a>Системные отчеты о работоспособности приложений
System.CM представляет собой службу диспетчера кластеров. Это система, которая управляет сведениями о приложении.

### <a name="state"></a>Состояние
Система System.CM сообщает об отсутствии проблем во время создания или обновления приложения. Она сообщает хранилищу данных о работоспособности об удалении приложения, чтобы обеспечить его удаление из хранилища.

* **SourceId**: System.CM.
* **Свойство**: State.
* **Дальнейшие действия**: если приложение создано или обновлено, оно должно включать отчет о работоспособности диспетчера кластера. Или проверьте состояние приложения, выполнив запрос (например, командлет PowerShell **Get-ServiceFabricApplication -ApplicationName** *имя_приложения*).

В следующем примере показано событие состояния в приложении **fabric:/WordCount** .

```powershell
PS C:\> Get-ServiceFabricApplicationHealth fabric:/WordCount -ServicesFilter None -DeployedApplicationsFilter None -ExcludeHealthStatistics

ApplicationName                 : fabric:/WordCount
AggregatedHealthState           : Ok
ServiceHealthStates             : None
DeployedApplicationHealthStates : None
HealthEvents                    : 
                                  SourceId              : System.CM
                                  Property              : State
                                  HealthState           : Ok
                                  SequenceNumber        : 282
                                  SentAt                : 7/13/2017 5:57:05 PM
                                  ReceivedAt            : 7/14/2017 4:55:10 PM
                                  TTL                   : Infinite
                                  Description           : Application has been created.
                                  RemoveWhenExpired     : False
                                  IsExpired             : False
                                  Transitions           : Error->Ok = 7/13/2017 5:57:05 PM, LastWarning = 1/1/0001 12:00:00 AM
```

## <a name="service-system-health-reports"></a>Системные отчеты о работоспособности служб
System.FM представляет собой службу диспетчера отработки отказов. Это система, которая управляет сведениями о службах.

### <a name="state"></a>Состояние
Система System.FM сообщает об отсутствии проблем, когда служба создана. Она удаляет сущность из хранилища данных о работоспособности, когда служба удалена.

* **SourceId**: System.FM.
* **Свойство**: State.

В следующем примере показано событие состояния в службе **fabric:/WordCount/WordCountWebService**.

```powershell
PS C:\> Get-ServiceFabricServiceHealth fabric:/WordCount/WordCountWebService -ExcludeHealthStatistics


ServiceName           : fabric:/WordCount/WordCountWebService
AggregatedHealthState : Ok
PartitionHealthStates : 
                        PartitionId           : 8bbcd03a-3a53-47ec-a5f1-9b77f73c53b2
                        AggregatedHealthState : Ok
                        
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 14
                        SentAt                : 7/13/2017 5:57:05 PM
                        ReceivedAt            : 7/14/2017 4:55:10 PM
                        TTL                   : Infinite
                        Description           : Service has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="service-correlation-error"></a>Ошибка сопоставления службы
**System.PLB** сообщает об ошибке, когда обнаруживает, что обновление службы для сопоставления с другой службой приводит к созданию цепочки сходства. Отчет очищается, когда происходит успешное обновление.

* **SourceId**: System.PLB.
* **Свойство:****ServiceDescription**.
* **Дальнейшие действия**: проверьте описания сопоставленных служб.

## <a name="partition-system-health-reports"></a>Системные отчеты о работоспособности разделов
System.FM представляет собой службу диспетчера отработки отказов. Это система, которая управляет сведениями о разделах служб.

### <a name="state"></a>Состояние
Система System.FM сообщает об отсутствии проблем, если создан работоспособный раздел. Она удаляет сущность из хранилища данных о работоспособности при удалении раздела.

Если в разделе нет минимального требуемого количества реплик, она сообщает об ошибке. Если в разделе есть минимальное требуемое количество реплик, но оно меньше целевого, она отображает предупреждение. Если в разделе наблюдается потеря кворума, система System.FM сообщает об ошибке.

Другие важные события: предупреждение о том, что перенастройка или сборка занимают больше времени, чем ожидалось. Ожидаемое время, необходимое для сборки или перенастройки, настраивается на основе сценариев службы. Если служба, например база данных SQL Azure, содержит терабайт данных о состоянии, то сборка занимает больше времени, чем для службы с меньшим объемом данных о состоянии.

* **SourceId**: System.FM.
* **Свойство**: State.
* **Дальнейшие действия**: если с состоянием работоспособности наблюдаются проблемы, возможно, некоторые реплики не созданы, не открыты либо их уровень не повышен до первичной или вторичной реплики должным образом. 

Если в описании указана потеря кворума, просмотрите подробный отчет о работоспособности и найдите реплики, которые не работают. Восстановите их работоспособность, это поможет вернуть секцию в сеть.

Если в описание указано, что заблокирована [перенастройка](service-fabric-concepts-reconfiguration.md) секции, то дополнительные сведения можно будет получить из отчета о работоспособности первичной реплики.

Кроме того, могут быть доступны другие отчеты о работоспособности System.FM, в том числе для реплик, секции, службы или других компонентов системы. 

Некоторые из этих отчетов описываются в приведенных ниже примерах. 

Ниже приведен пример работоспособного раздела.

```powershell
PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountWebService | Get-ServiceFabricPartitionHealth -ExcludeHealthStatistics -ReplicasFilter None

PartitionId           : 8bbcd03a-3a53-47ec-a5f1-9b77f73c53b2
AggregatedHealthState : Ok
ReplicaHealthStates   : None
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 70
                        SentAt                : 7/13/2017 5:57:05 PM
                        ReceivedAt            : 7/14/2017 4:55:10 PM
                        TTL                   : Infinite
                        Description           : Partition is healthy.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM
```

В следующем примере показаны данные о работоспособности секции, количество реплик в которой меньше целевого значения. Далее необходимо получить описание секции со сведениями о ее конфигурации: **MinReplicaSetSize** равно 3, а **TargetReplicaSetSize** — 7. Затем получите количество узлов в кластере. В данном случае оно равно пяти. В этом случае мы не сможем разместить две реплики, так как требуемое число реплик больше, чем число доступных узлов.

```powershell
PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricPartitionHealth -ReplicasFilter None -ExcludeHealthStatistics


PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=false.
                        
ReplicaHealthStates   : None
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Warning
                        SequenceNumber        : 123
                        SentAt                : 7/14/2017 4:55:39 PM
                        ReceivedAt            : 7/14/2017 4:55:44 PM
                        TTL                   : Infinite
                        Description           : Partition is below target replica or instance count.
                        fabric:/WordCount/WordCountService 7 2 af2e3e44-a8f8-45ac-9f31-4093eb897600
                          N/S Ready _Node_2 131444422260002646
                          N/S Ready _Node_4 131444422293113678
                          N/S Ready _Node_3 131444422293113679
                          N/S Ready _Node_1 131444422293118720
                          N/P Ready _Node_0 131444422293118721
                          (Showing 5 out of 5 replicas. Total available replicas: 5)
                        
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 7/14/2017 4:55:44 PM, LastOk = 1/1/0001 12:00:00 AM
                        
                        SourceId              : System.PLB
                        Property              : ServiceReplicaUnplacedHealth_Secondary_af2e3e44-a8f8-45ac-9f31-4093eb897600
                        HealthState           : Warning
                        SequenceNumber        : 131445250939703027
                        SentAt                : 7/14/2017 4:58:13 PM
                        ReceivedAt            : 7/14/2017 4:58:14 PM
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
                        FaultDomain:fd:/2 NodeName:_Node_2 NodeType:NodeType2 UpgradeDomain:2 UpgradeDomain: ud:/2 Deactivation Intent/Status: None/None
                        FaultDomain:fd:/1 NodeName:_Node_1 NodeType:NodeType1 UpgradeDomain:1 UpgradeDomain: ud:/1 Deactivation Intent/Status: None/None
                        
                        Existing Primary Replica -- Nodes with Partition's Existing Primary Replica or Secondary Replicas:
                        --
                        FaultDomain:fd:/0 NodeName:_Node_0 NodeType:NodeType0 UpgradeDomain:0 UpgradeDomain: ud:/0 Deactivation Intent/Status: None/None
                        
                        
                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : Error->Warning = 7/14/2017 4:56:14 PM, LastOk = 1/1/0001 12:00:00 AM

PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | select MinReplicaSetSize,TargetReplicaSetSize

MinReplicaSetSize TargetReplicaSetSize
----------------- --------------------
                2                    7                        

PS C:\> @(Get-ServiceFabricNode).Count
5
```

В следующем примере показана работоспособность секции, перенастройка которой заблокирована из-за того, что пользователь не учел маркер отмены в методе **RunAsync**. Просмотр отчета о работоспособности любой реплики, помеченной как первичная (P), может помочь более детально изучить проблему.

```powershell
PS C:\utilities\ServiceFabricExplorer\ClientPackage\lib> Get-ServiceFabricPartitionHealth 0e40fd81-284d-4be4-a665-13bc5a6607ec -ExcludeHealthStatistics 


PartitionId           : 0e40fd81-284d-4be4-a665-13bc5a6607ec
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', 
                        ConsiderWarningAsError=false.
                                               
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Warning
                        SequenceNumber        : 7
                        SentAt                : 8/27/2017 3:43:09 AM
                        ReceivedAt            : 8/27/2017 3:43:32 AM
                        TTL                   : Infinite
                        Description           : Partition reconfiguration is taking longer than expected.
                        fabric:/app/test1 3 1 0e40fd81-284d-4be4-a665-13bc5a6607ec
                          P/S Ready Node1 131482789658160654
                          S/P Ready Node2 131482789688598467
                          S/S Ready Node3 131482789688598468
                          (Showing 3 out of 3 replicas. Total available replicas: 3)                        
                        
                        For more information see: https://aka.ms/sfhealth
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Ok->Warning = 8/27/2017 3:43:32 AM, LastError = 1/1/0001 12:00:00 AM
```
Этот отчет о работоспособности содержит состояние реплик секции на этапе перенастройки. 

```
  P/S Ready Node1 131482789658160654
  S/P Ready Node2 131482789688598467
  S/S Ready Node3 131482789688598468
```

Для каждой реплики отчет о работоспособности содержит:
- роль в предыдущей конфигурации;
- роль в текущей конфигурации;
- [состояние реплики](service-fabric-concepts-replica-lifecycle.md);
- узел, на котором выполняется реплика;
- идентификатор реплики.

В случае, пример которого приведен, требуется углубленное изучение. Изучите работоспособность каждой отдельной реплик, начав с реплик, которые помечены как `Primary` и `Secondary` (131482789658160654 и 131482789688598467) в предыдущем примере.

### <a name="replica-constraint-violation"></a>Нарушение ограничений в отношении реплик
**System.PLB** выдает предупреждение, если обнаруживает нарушение ограничений реплик и не может разместить все реплики секции. В сведениях отчета показано, какие ограничения и свойства блокируют размещение реплики.

* **SourceId**: System.PLB.
* **Свойство**: начинается с **репликаконстраинтвиолатион**.

## <a name="replica-system-health-reports"></a>Системные отчеты о работоспособности реплик
**System.RA** представляет собой компонент агента перенастройки. Это система для отслеживания состояния реплики.

### <a name="state"></a>Состояние
System.RA сообщает об отсутствии сбоев при создании реплики.

* **SourceId**: System.RA.
* **Свойство**: State.

Ниже приведен пример работоспособной реплики.

```powershell
PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricReplica | where {$_.ReplicaRole -eq "Primary"} | Get-ServiceFabricReplicaHealth

PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
ReplicaId             : 131444422293118721
AggregatedHealthState : Ok
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 131445248920273536
                        SentAt                : 7/14/2017 4:54:52 PM
                        ReceivedAt            : 7/14/2017 4:55:13 PM
                        TTL                   : Infinite
                        Description           : Replica has been created._Node_0
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/14/2017 4:55:13 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="replicaopenstatus-replicaclosestatus-replicachangerolestatus"></a>ReplicaOpenStatus, ReplicaCloseStatus, ReplicaChangeRoleStatus
Это свойство используется для указания предупреждений или сбоев при попытке открыть реплику, закрыть реплику или изменить роль реплики. Ознакомьтесь с [жизненным циклом реплики](service-fabric-concepts-replica-lifecycle.md). Ошибками могут быть исключения, порожденные вызовами API или аварийным завершением процесса узла службы в течение этого времени. Для сбоев из-за вызовов API из кода C# платформа Service Fabric добавляет в отчет о работоспособности исключение и трассировку стека.

Эти предупреждения о работоспособности создаются после определенного числа попыток выполнить действие локально (в зависимости от политики). Платформа Service Fabric повторяет действие, пока не будет достигнуто максимальное пороговое значение. По достижении максимального значения порога платформа может попытаться исправить ситуацию. Данная попытка может привести к очистке этих предупреждений, если платформа оставит попытки выполнить действие на данном узле. Пример. Если не удастся открыть реплику на узле, Service Fabric выдаст предупреждение о работоспособности. Если реплику по-прежнему не удается открыть, Service Fabric выполняет самостоятельное восстановление. При этом может быть предпринята попытка выполнить ту же самую операцию на другом узле. Это приведет к очистке предупреждения об этой реплике. 

* **SourceId**: System.RA.
* **Свойство**: **ReplicaOpenStatus**, **ReplicaCloseStatus** и **ReplicaChangeRoleStatus**.
* **Дальнейшие действия**. Изучите код службы и аварийные дампы, чтобы определить, почему происходит сбой операции.

В следующем примере показана работоспособность реплики, порождающей `TargetInvocationException` в методе open. Описание содержит точку сбоя (**IStatefulServiceReplica.Open**), тип исключения (**TargetInvocationException**) и трассировку стека.

```powershell
PS C:\> Get-ServiceFabricReplicaHealth -PartitionId 337cf1df-6cab-4825-99a9-7595090c0b1b -ReplicaOrInstanceId 131483509874784794


PartitionId           : 337cf1df-6cab-4825-99a9-7595090c0b1b
ReplicaId             : 131483509874784794
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.RA', Property='ReplicaOpenStatus', HealthState='Warning', 
                        ConsiderWarningAsError=false.
                        
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : ReplicaOpenStatus
                        HealthState           : Warning
                        SequenceNumber        : 131483510001453159
                        SentAt                : 8/27/2017 11:43:20 PM
                        ReceivedAt            : 8/27/2017 11:43:21 PM
                        TTL                   : Infinite
                        Description           : Replica had multiple failures during open on _Node_0 API call: IStatefulServiceReplica.Open(); Error = System.Reflection.TargetInvocationException (-2146232828)
Exception has been thrown by the target of an invocation.
   at Microsoft.ServiceFabric.Replicator.RecoveryManager.d__31.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.CompilerServices.TaskAwaiter.ThrowForNonSuccess(Task task)
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.ServiceFabric.Replicator.LoggingReplicator.d__137.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.CompilerServices.TaskAwaiter.ThrowForNonSuccess(Task task)
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.ServiceFabric.Replicator.DynamicStateManager.d__109.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.CompilerServices.TaskAwaiter.ThrowForNonSuccess(Task task)
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.ServiceFabric.Replicator.TransactionalReplicator.d__79.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.CompilerServices.TaskAwaiter.ThrowForNonSuccess(Task task)
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.ServiceFabric.Replicator.StatefulServiceReplica.d__21.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.CompilerServices.TaskAwaiter.ThrowForNonSuccess(Task task)
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.ServiceFabric.Services.Runtime.StatefulServiceReplicaAdapter.d__0.MoveNext()

    For more information see: https://aka.ms/sfhealth
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 8/27/2017 11:43:21 PM, LastOk = 1/1/0001 12:00:00 AM                        
```

В следующем примере показана реплика, работа которой постоянно аварийно завершается во время закрытия.

```powershell
C:>Get-ServiceFabricReplicaHealth -PartitionId dcafb6b7-9446-425c-8b90-b3fdf3859e64 -ReplicaOrInstanceId 131483565548493142


PartitionId           : dcafb6b7-9446-425c-8b90-b3fdf3859e64
ReplicaId             : 131483565548493142
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.RA', Property='ReplicaCloseStatus', HealthState='Warning', 
                        ConsiderWarningAsError=false.
                        
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : ReplicaCloseStatus
                        HealthState           : Warning
                        SequenceNumber        : 131483565611258984
                        SentAt                : 8/28/2017 1:16:01 AM
                        ReceivedAt            : 8/28/2017 1:16:03 AM
                        TTL                   : Infinite
                        Description           : Replica had multiple failures during close on _Node_1. The application 
                        host has crashed.
                        
                        For more information see: https://aka.ms/sfhealth
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 8/28/2017 1:16:03 AM, LastOk = 1/1/0001 12:00:00 AM
```

### <a name="reconfiguration"></a>Reconfiguration
Это свойство используется, указать обнаруженную остановку или блокировку [перенастройки](service-fabric-concepts-reconfiguration.md) реплики. Этот отчет о работоспособности реплики, которая в данный момент является первичной (за исключением случаев перенастройки с переключением первичной реплики, в которых ее уровень может быть понижен с первичной до активной вторичной реплики).

Перенастройка может быть заблокирована по одной из следующих причин:

- Действие с локальной репликой (для которой выполняется перенастройка) не завершается. В этом случае следует изучить отчеты о работоспособности этой реплики из других компонентов (System.RAP или System.RE). Они могут содержать дополнительные сведения.

- Не завершается действие с удаленной репликой. Реплики, действия с которыми ожидают выполнения, будут указаны в отчете о работоспособности. Необходимо подробнее изучить отчеты о работоспособности этих удаленных реплик. Также возможны проблемы связи между текущим узлом и удаленным узлом.

В редких случаях перенастройка может быть заблокирована из-за проблем связи или других проблем между текущим узлом и службой диспетчера отработки отказов.

* **SourceId**: System.RA.
* **Свойство:** Reconfiguration.
* **Дальнейшие действия**. Изучите локальные или удаленные реплики в зависимости от описания в отчете о работоспособности.

В следующем примере показан отчет о работоспособности в ситуации, когда перенастройка была заблокирована из-за локальной реплики. В этом примере это произошло из-за того, что служба не учитывает маркер отмены.

```powershell
PS C:\> Get-ServiceFabricReplicaHealth -PartitionId 9a0cedee-464c-4603-abbc-1cf57c4454f3 -ReplicaOrInstanceId 131483600074836703


PartitionId           : 9a0cedee-464c-4603-abbc-1cf57c4454f3
ReplicaId             : 131483600074836703
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.RA', Property='Reconfiguration', HealthState='Warning', 
                        ConsiderWarningAsError=false.
                        
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : Reconfiguration
                        HealthState           : Warning
                        SequenceNumber        : 131483600309264482
                        SentAt                : 8/28/2017 2:13:50 AM
                        ReceivedAt            : 8/28/2017 2:13:57 AM
                        TTL                   : Infinite
                        Description           : Reconfiguration is stuck. Waiting for response from the local replica
                        
                        For more information see: https://aka.ms/sfhealth
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 8/28/2017 2:13:57 AM, LastOk = 1/1/0001 12:00:00 AM
```

В следующем примере показан отчет о работоспособности в ситуации, когда перенастройка остановилась в ожидании ответа от двух удаленных реплик. В этом примере в секции существуют три реплики, включая текущую первичную реплику. 

```Powershell
PS C:\> Get-ServiceFabricReplicaHealth -PartitionId  579d50c6-d670-4d25-af70-d706e4bc19a2 -ReplicaOrInstanceId 131483956274977415


PartitionId           : 579d50c6-d670-4d25-af70-d706e4bc19a2
ReplicaId             : 131483956274977415
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.RA', Property='Reconfiguration', HealthState='Warning', ConsiderWarningAsError=false.
                        
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : Reconfiguration
                        HealthState           : Warning
                        SequenceNumber        : 131483960376212469
                        SentAt                : 8/28/2017 12:13:57 PM
                        ReceivedAt            : 8/28/2017 12:14:07 PM
                        TTL                   : Infinite
                        Description           : Reconfiguration is stuck. Waiting for response from 2 replicas
                        
                        Pending Replicas: 
                        P/I Down 40 131483956244554282
                        S/S Down 20 131483956274972403
                        
                        For more information see: https://aka.ms/sfhealth
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 8/28/2017 12:07:37 PM, LastOk = 1/1/0001 12:00:00 AM
```

Этот отчет о работоспособности показывает, что перенастройка была заблокирована из-за ожидания ответа от двух реплик. 

```
    P/I Down 40 131483956244554282
    S/S Down 20 131483956274972403
```

В нем представлена следующая информация для каждой реплики:
- роль в предыдущей конфигурации;
- роль в текущей конфигурации;
- [состояние реплики](service-fabric-concepts-replica-lifecycle.md);
- идентификатор узла;
- идентификатор реплики.

Чтобы разблокировать перенастройку нужно:
- восстановить работоспособность **нерабочих** реплик; 
- завершить сборку реплик **в состоянии сборки** и перевести их в состояние готовности.

### <a name="slow-service-api-call"></a>Медленный вызов API службы
Системы **System.RAP** и **System.Replicator** выдают предупреждение, если вызов пользовательского кода службы длится больше заданного периода. Предупреждение удаляется по завершении вызова.

* **SourceId**: System.RAP или System.Replicator.
* **Свойство**: имя медленного API. В описании приводятся дополнительные сведения о времени, в течение которого в работе API наблюдалась задержка.
* **Дальнейшие действия**: выясните причину, по которой вызов занимает больше времени, чем ожидалось.

В следующем примере показано событие работоспособности из System.RAP для надежной службы, которая не учитывает маркер отмены в методе **RunAsync**.

```powershell
PS C:\> Get-ServiceFabricReplicaHealth -PartitionId 5f6060fb-096f-45e4-8c3d-c26444d8dd10 -ReplicaOrInstanceId 131483966141404693


PartitionId           : 5f6060fb-096f-45e4-8c3d-c26444d8dd10
ReplicaId             : 131483966141404693
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.RA', Property='Reconfiguration', HealthState='Warning', ConsiderWarningAsError=false.
                        
HealthEvents          :                         
                        SourceId              : System.RAP
                        Property              : IStatefulServiceReplica.ChangeRole(S)Duration
                        HealthState           : Warning
                        SequenceNumber        : 131483966663476570
                        SentAt                : 8/28/2017 12:24:26 PM
                        ReceivedAt            : 8/28/2017 12:24:56 PM
                        TTL                   : Infinite
                        Description           : The api IStatefulServiceReplica.ChangeRole(S) on _Node_1 is stuck. Start Time (UTC): 2017-08-28 12:23:56.347.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 8/28/2017 12:24:56 PM, LastOk = 1/1/0001 12:00:00 AM
                        
```

Свойство и текст указывают, какие API привели к блокировке. Дальнейшие действия отличаются и зависят от программных интерфейсов, вызвавших блокировку. Любой API в *IStatefulServiceReplica* или *IStatelessServiceInstance* обычно является ошибкой в коде службы. В следующем разделе описано, как эти преобразования относятся к [модели Reliable Services](service-fabric-reliable-services-lifecycle.md).

- **IStatefulServiceReplica. Open**: это предупреждение означает, что вызов `CreateServiceInstanceListeners` , `ICommunicationListener.OpenAsync` или, если он переопределен, `OnOpenAsync` будет остановлен.

- **IStatefulServiceReplica.Close** и **IStatefulServiceReplica.Abort**. Наиболее распространенный случай — когда служба не учитывает маркер отмены, передаваемый в `RunAsync`. Возможна также блокировка `ICommunicationListener.CloseAsync` или `OnCloseAsync` (в случае переопределения).

- **IStatefulServiceReplica.ChangeRole(S)** и **IStatefulServiceReplica.ChangeRole(N)**. Наиболее распространенный случай — когда служба не учитывает маркер отмены, передаваемый в `RunAsync`. В этом сценарии лучшим решением будет перезапуск реплики.

- **IStatefulServiceReplica. ChangeRole (P)**. наиболее распространенным случаем является то, что служба не вернула задачу из `RunAsync` .

Другие вызовы API, которые могут быть задержаны, находятся в интерфейсе **IReplicator** . Пример:

- **IReplicator.CatchupReplicaSet**. Это предупреждение означает одну из следующих ситуаций: недостаточно реплик (это можно определить, посмотрев состояние реплик в секции или отчет о работоспособности System.FM для заблокированной перенастройки); реплики не подтверждают операции. С помощью командлета PowerShell `Get-ServiceFabricDeployedReplicaDetail` можно узнать ход выполнения всех реплик. Проблема связана с репликами, у которых значение `LastAppliedReplicationSequenceNumber` ниже значения `CommittedSequenceNumber` первичной реплики.

- **IReplicator.BuildReplica (\<Remote ReplicaId>)**. Это предупреждение указывает на проблему в процессе сборки. Ознакомьтесь с [жизненным циклом реплики](service-fabric-concepts-replica-lifecycle.md). Проблема может быть вызвана неправильной настройкой адреса репликатора. Дополнительные сведения см. в разделах [Настройка надежных служб с отслеживанием состояния](service-fabric-reliable-services-configuration.md) и [Указание ресурсов в манифесте служб](service-fabric-service-manifest-resources.md). Также возможно наличие проблемы на удаленном узле.

### <a name="replicator-system-health-reports"></a>Системные отчеты о работоспособности репликатора
**Переполнение очереди репликации:** 
 **System. репликатор** сообщает о предупреждении при заполнении очереди репликации. На сервере-источнике это обычно происходит, так как одна или несколько вторичных реплик выполняются слишком медленно для подтверждения операций. На сервере-получателе это обычно происходит, если служба медленно применяет операции. Предупреждение удаляется при очистке очереди.

* **SourceId**: System.Replicator.
* **Свойство**: **PrimaryReplicationQueueStatus** или **SecondaryReplicationQueueStatus**, в зависимости от роли реплики.
* **Дальнейшие действия.** Если отчет находится на сервере-источнике, проверьте подключение между узлами в кластере. Если все подключения находятся в работоспособном состоянии, возможно, по крайней мере одна первичная реплика с высокой задержкой диска применяет операции. Если отчет находится на вторичной реплике, сначала проверьте использование диска и производительность на узле, а затем исходящее соединение из медленного узла к первичной реплике.

**RemoteReplicatorConnectionStatus:** 
 **System. репликатор** в первичной реплике сообщает о том, что подключение к дополнительному (удаленному) репликатору неработоспособно. Адрес удаленного репликатора можно посмотреть в сообщении отчетов. Это упрощает определение ложных конфигураций и сетевых проблем между репликаторами.

* **SourceId**: System.Replicator.
* **Свойство:****RemoteReplicatorConnectionStatus**.
* **Дальнейшие действия.** Проверьте сообщение об ошибке и убедитесь, что адрес удаленного репликатора настроен правильно (например, если удаленный репликатор открывается с адресом ожидания передачи данных "localhost", внешний доступ к нему невозможен). Если адрес выглядит правильно, проверьте подключение между первичным узлом и удаленным адресом, чтобы найти возможные проблемы в сети.

### <a name="replication-queue-full"></a>Переполнение очереди репликации
**System.Replicator** выдает предупреждение при переполнении очереди репликации. На сервере-источнике это обычно происходит, так как одна или несколько вторичных реплик выполняются слишком медленно для подтверждения операций. На сервере-получателе это обычно происходит, если служба медленно применяет операции. Предупреждение удаляется при очистке очереди.

* **SourceId**: System.Replicator.
* **Свойство**: **PrimaryReplicationQueueStatus** или **SecondaryReplicationQueueStatus**, в зависимости от роли реплики.

### <a name="slow-naming-operations"></a>Медленные операции именования
**System.NamingService** сообщает о работоспособности первичной реплики, когда операция именования длится больше допустимого. Примеры операций именования — [CreateServiceAsync](/dotnet/api/system.fabric.fabricclient.servicemanagementclient.createserviceasync) и [DeleteServiceAsync](/dotnet/api/system.fabric.fabricclient.servicemanagementclient.deleteserviceasync). Дополнительные методы можно найти в статьях, посвященных FabricClient (например, в статье о [методах управления службами](/dotnet/api/system.fabric.fabricclient.servicemanagementclient) или [методах управления свойствами](/dotnet/api/system.fabric.fabricclient.propertymanagementclient)).

> [!NOTE]
> Служба именования присваивает имена служб определенному расположению в кластере, позволяя пользователям управлять именами и свойствами служб. Это секционированная материализованная служба Service Fabric. Одна из секций представляет имя *Authority Owner*, содержащее метаданные о всех именах и службах Service Fabric. Имена Service Fabric сопоставляются с разными секциями, называемыми секциями *Name Owner*. Таким образом, служба является расширяемой. Узнайте больше о [службе именования](service-fabric-architecture.md).
> 
> 

Когда операция именования длится дольше, чем ожидалось, для нее задается состояние "Warning" в первичной реплике секции службы именования, выполняющей операцию. Если операция завершается успешно, это состояние удаляется. Если операция завершается с ошибкой, отчет о работоспособности будет содержать сведения об ошибке.

* **SourceId**: System.NamingService.
* **Свойство:** начинается с префикса **Duration_** и определяет медленную операцию и имя экземпляра Service Fabric, к которому применяется операция. Например, если создание службы с именем **Fabric:/MyApp/MyService** занимает слишком много времени, свойство имеет значение **Duration_AOCreateService. Fabric:/MyApp/MyService**. Authority Owner указывает на роль секции именования для этого имени и операции.
* **Дальнейшие действия**: проверьте, почему происходит сбой операции именования. Каждая операция может завершаться сбоем по различным причинам. Например, удаление службы может быть заблокировано. Служба может быть заблокирована, так как узел приложения постоянно аварийно завершается на узле из-за ошибки пользователя в коде службы.

В следующем примере показана операция создания службы. Длительность операции превышает заданное время. Authority Owner выполняет повторную попытку и отправляет работу к Name Owner. Name Owner завершает последнюю операцию с истечением времени ожидания. В этом случае одна и та же реплика является первичной для ролей Authority Owner и Name Owner.

```powershell
PartitionId           : 00000000-0000-0000-0000-000000001000
ReplicaId             : 131064359253133577
AggregatedHealthState : Warning
UnhealthyEvaluations  :
                        Unhealthy event: SourceId='System.NamingService', Property='Duration_AOCreateService.fabric:/MyApp/MyService', HealthState='Warning', ConsiderWarningAsError=false.

HealthEvents          :
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 131064359308715535
                        SentAt                : 4/29/2016 8:38:50 PM
                        ReceivedAt            : 4/29/2016 8:39:08 PM
                        TTL                   : Infinite
                        Description           : Replica has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 4/29/2016 8:39:08 PM, LastWarning = 1/1/0001 12:00:00 AM

                        SourceId              : System.NamingService
                        Property              : Duration_AOCreateService.fabric:/MyApp/MyService
                        HealthState           : Warning
                        SequenceNumber        : 131064359526778775
                        SentAt                : 4/29/2016 8:39:12 PM
                        ReceivedAt            : 4/29/2016 8:39:38 PM
                        TTL                   : 00:05:00
                        Description           : The AOCreateService started at 2016-04-29 20:39:08.677 is taking longer than 30.000.
                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : Error->Warning = 4/29/2016 8:39:38 PM, LastOk = 1/1/0001 12:00:00 AM

                        SourceId              : System.NamingService
                        Property              : Duration_NOCreateService.fabric:/MyApp/MyService
                        HealthState           : Warning
                        SequenceNumber        : 131064360657607311
                        SentAt                : 4/29/2016 8:41:05 PM
                        ReceivedAt            : 4/29/2016 8:41:08 PM
                        TTL                   : 00:00:15
                        Description           : The NOCreateService started at 2016-04-29 20:39:08.689 completed with FABRIC_E_TIMEOUT in more than 30.000.
                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : Error->Warning = 4/29/2016 8:39:38 PM, LastOk = 1/1/0001 12:00:00 AM
```

## <a name="deployedapplication-system-health-reports"></a>Системные отчеты о работоспособности DeployedApplication
**System.Hosting** — это центр развернутых сущностей.

### <a name="activation"></a>Активация
Система System.Hosting сообщает об отсутствии проблем, если приложение успешно активировано на узле. В противном случае отображается сообщение об ошибке.

* **SourceId**: System.Hosting.
* **Свойство:****Activation**, включая версию выпуска.
* **Дальнейшие действия**: если в приложении есть проблемы с работоспособностью, выясните причину ошибки при активации.

Ниже приведен пример успешной активации.

```powershell
PS C:\> Get-ServiceFabricDeployedApplicationHealth -NodeName _Node_1 -ApplicationName fabric:/WordCount -ExcludeHealthStatistics

ApplicationName                    : fabric:/WordCount
NodeName                           : _Node_1
AggregatedHealthState              : Ok
DeployedServicePackageHealthStates : 
                                     ServiceManifestName   : WordCountServicePkg
                                     ServicePackageActivationId : 
                                     NodeName              : _Node_1
                                     AggregatedHealthState : Ok
                                     
HealthEvents                       : 
                                     SourceId              : System.Hosting
                                     Property              : Activation
                                     HealthState           : Ok
                                     SequenceNumber        : 131445249083836329
                                     SentAt                : 7/14/2017 4:55:08 PM
                                     ReceivedAt            : 7/14/2017 4:55:14 PM
                                     TTL                   : Infinite
                                     Description           : The application was activated successfully.
                                     RemoveWhenExpired     : False
                                     IsExpired             : False
                                     Transitions           : Error->Ok = 7/14/2017 4:55:14 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="download"></a>Скачать
System.Hosting сообщает об ошибке, если скачивание пакета приложения завершается сбоем.

* **SourceId**: System.Hosting.
* **Свойство:****Download**, включая версию выпуска.
* **Дальнейшие действия**: выясните причину ошибки при скачивании на узле.

## <a name="deployedservicepackage-system-health-reports"></a>Системные отчеты о работоспособности DeployedServicePackage
**System.Hosting** — это центр развернутых сущностей.

### <a name="service-package-activation"></a>Активация пакета службы
Система System.Hosting сообщает об отсутствии проблем, если пакет службы на узле успешно активирован. В противном случае отображается сообщение об ошибке.

* **SourceId**: System.Hosting.
* **Свойство**: Activation.
* **Дальнейшие действия**: выясните причину ошибки при активации.

### <a name="code-package-activation"></a>Активация пакета кода
System.Hosting сообщает об отсутствии проблем, если каждый пакет кода успешно активирован. Если активация завершилась ошибкой, выводится заданное предупреждение. Если не удалось активировать **CodePackage** или этот элемент завершил работу с ошибкой и превышением значения настроенного параметра **CodePackageHealthErrorThreshold**, система выдает сообщение об ошибке. Если пакет службы содержит несколько пакетов кода, для каждого из них создается отчет об активации.

* **SourceId**: System.Hosting.
* **Свойство:** использует префикс **CodePackageActivation**, а также содержит имя пакета кода и точку входа *CodePackageActivation:CodePackageName:SetupEntryPoint/EntryPoint*. Например, **CodePackageActivation:Code:SetupEntryPoint**.

### <a name="service-type-registration"></a>Регистрация типа службы
System.Hosting сообщает об отсутствии проблем, если тип службы успешно зарегистрирован. Он сообщает об ошибке, если регистрация не выполнялась вовремя, как настроено с помощью **параметра servicetyperegistrationtimeout**. При закрытии среды выполнения регистрация типа службы на узле отменяется и служба Hosting отображает предупреждение.

* **SourceId**: System.Hosting.
* **Свойство**: использует префикс **ServiceTypeRegistration** и содержит имя типа службы. Например, **ServiceTypeRegistration:FileStoreServiceType**.

Ниже приведен пример работоспособного развернутого пакета службы.

```powershell
PS C:\> Get-ServiceFabricDeployedServicePackageHealth -NodeName _Node_1 -ApplicationName fabric:/WordCount -ServiceManifestName WordCountServicePkg


ApplicationName            : fabric:/WordCount
ServiceManifestName        : WordCountServicePkg
ServicePackageActivationId : 
NodeName                   : _Node_1
AggregatedHealthState      : Ok
HealthEvents               : 
                             SourceId              : System.Hosting
                             Property              : Activation
                             HealthState           : Ok
                             SequenceNumber        : 131445249084026346
                             SentAt                : 7/14/2017 4:55:08 PM
                             ReceivedAt            : 7/14/2017 4:55:14 PM
                             TTL                   : Infinite
                             Description           : The ServicePackage was activated successfully.
                             RemoveWhenExpired     : False
                             IsExpired             : False
                             Transitions           : Error->Ok = 7/14/2017 4:55:14 PM, LastWarning = 1/1/0001 12:00:00 AM
                             
                             SourceId              : System.Hosting
                             Property              : CodePackageActivation:Code:EntryPoint
                             HealthState           : Ok
                             SequenceNumber        : 131445249084306362
                             SentAt                : 7/14/2017 4:55:08 PM
                             ReceivedAt            : 7/14/2017 4:55:14 PM
                             TTL                   : Infinite
                             Description           : The CodePackage was activated successfully.
                             RemoveWhenExpired     : False
                             IsExpired             : False
                             Transitions           : Error->Ok = 7/14/2017 4:55:14 PM, LastWarning = 1/1/0001 12:00:00 AM
                             
                             SourceId              : System.Hosting
                             Property              : ServiceTypeRegistration:WordCountServiceType
                             HealthState           : Ok
                             SequenceNumber        : 131445249088096842
                             SentAt                : 7/14/2017 4:55:08 PM
                             ReceivedAt            : 7/14/2017 4:55:14 PM
                             TTL                   : Infinite
                             Description           : The ServiceType was registered successfully.
                             RemoveWhenExpired     : False
                             IsExpired             : False
                             Transitions           : Error->Ok = 7/14/2017 4:55:14 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="download"></a>Скачать
System.Hosting сообщает об ошибке при скачивании пакета службы.

* **SourceId**: System.Hosting.
* **Свойство:****Download**, включая версию выпуска.
* **Дальнейшие действия**: выясните причину ошибки при скачивании на узле.

### <a name="upgrade-validation"></a>Проверка обновления
System.Hosting сообщает об ошибке проверки во время обновления или об ошибке обновления на узле.

* **SourceId**: System.Hosting.
* **Свойство**: использует префикс **FabricUpgradeValidation** и содержит версию обновления.
* **Описание**: указывает на обнаруженную ошибку.

### <a name="undefined-node-capacity-for-resource-governance-metrics"></a>Неопределенная емкость узла для метрик управления ресурсами
System.Hosting сообщает о предупреждении, если емкости узла не определены в манифесте кластера и конфигурация автоматического обнаружения отключена. В Service Fabric появится предупреждение о работоспособности, когда первый пакет службы, использующий [систему управления ресурсами](service-fabric-resource-governance.md), зарегистрируется на указанном узле.

* **SourceId**: System.Hosting.
* **Свойство:****ResourceGovernance**.
* **Дальнейшие действия.** Предпочтительный способ решения этой проблемы — изменение манифеста кластера, чтобы включить автоматическое обнаружение доступных ресурсов. Другой способ — обновление манифеста кластера, путем указания правильных значений емкости узла для этих метрик.

## <a name="next-steps"></a>Дальнейшие шаги
* [Просмотр отчетов о работоспособности Service Fabric](service-fabric-view-entities-aggregated-health.md)

* [Проверка работоспособности службы и оповещение о проблемах](service-fabric-diagnostics-how-to-report-and-check-service-health.md)

* [Мониторинг и диагностика состояния служб в локальной среде разработки](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)

* [Обновление приложения Service Fabric](service-fabric-application-upgrade.md)
