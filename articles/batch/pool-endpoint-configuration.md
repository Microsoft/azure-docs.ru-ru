---
title: Настройка конечных точек узла в пуле пакетной службы Azure
description: Сведения о том, как настроить или отключить доступ к портам SSH или RDP на вычислительных узлах пула пакетной службы Azure.
ms.topic: how-to
ms.date: 02/13/2018
ms.openlocfilehash: 4e7df7da539be75ef1befdff4b4e1fe5244c1702
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92109312"
---
# <a name="configure-or-disable-remote-access-to-compute-nodes-in-an-azure-batch-pool"></a>Настройка или отключение удаленного доступа к вычислительным узлам пула пакетной службы Azure

По умолчанию пакетная служба позволяет [пользователю узла](/rest/api/batchservice/computenode/adduser) с возможностью подключения к сети подключиться извне к вычислительному узлу в пуле пакетной службы. Например, пользователь может подключиться с удаленного рабочего стола (RDP) к порту 3389 на вычислительном узле в пуле Windows. Аналогичным образом по умолчанию пользователь может подключиться через Secure Shell (SSH) к порту 22 на вычислительном узле в пуле Linux. 

В своей среде вам может потребоваться ограничить или отключить эти параметры внешнего доступа по умолчанию. Эти параметры можно изменить с помощью API пакетной службы для установки свойства [PoolEndpointConfiguration](/rest/api/batchservice/pool/add#poolendpointconfiguration). 

## <a name="about-the-pool-endpoint-configuration"></a>О конфигурации конечной точки пула
Конфигурация конечной точки состоит из одного или нескольких [пулов преобразования сетевых адресов (NAT)](/rest/api/batchservice/pool/add#inboundnatpool) интерфейсных портов. (Не путайте пул NAT с пулом пакетной службы вычислительных узлов.) Настройте каждый пул NAT на переопределение параметров подключения по умолчанию на вычислительных узлах пула. 

Каждая конфигурация пула NAT включает одно или несколько правил [групп безопасности сети](/rest/api/batchservice/pool/add#networksecuritygrouprule), каждое из которых разрешает или запрещает определенный сетевой трафик к конечной точке. Вы можете разрешить или запретить весь трафик, трафик, определяемый по [тегу службы](../virtual-network/network-security-groups-overview.md#service-tags) (например, "Интернет"), или трафик из определенных IP-адресов или подсетей.

### <a name="considerations"></a>Рекомендации
* Конфигурация конечной точки пула является частью [сетевой конфигурации](/rest/api/batchservice/pool/add#networkconfiguration) пула. В сетевую конфигурацию при необходимости можно включить параметры для присоединения пула к [виртуальной сети Azure](batch-virtual-network.md). При настройке пула в виртуальной сети можно создать правила NSG, которые используют параметры адреса в виртуальной сети.
* При настройке пула NAT можно настроить несколько правил группы безопасности сети (NSG). Правила проверяются в порядке приоритета. Когда применяется правило, соответствие других правил не проверяется.


## <a name="example-deny-all-rdp-traffic"></a>Пример запрет всего трафика RDP

В следующем фрагменте кода C# показано, как настроить конечную точку RDP на вычислительных узлах в пуле Windows, чтобы запретить весь сетевой трафик. Конечная точка использует пул интерфейсных портов в диапазоне *60000–60099*. 

```csharp
pool.NetworkConfiguration = new NetworkConfiguration
{
    EndpointConfiguration = new PoolEndpointConfiguration(new InboundNatPool[]
    {
      new InboundNatPool("RDP", InboundEndpointProtocol.Tcp, 3389, 60000, 60099, new NetworkSecurityGroupRule[]
        {
            new NetworkSecurityGroupRule(162, NetworkSecurityGroupRuleAccess.Deny, "*"),
        })
    })    
};
```

## <a name="example-deny-all-ssh-traffic-from-the-internet"></a>Пример запрет на получение всего трафика SSH из Интернета

В следующем фрагменте кода Python показано, как настроить конечную точку SSH на вычислительных узлах в пуле Linux, чтобы запретить весь сетевой трафик. Конечная точка использует пул интерфейсных портов в диапазоне *4000–4100*. 

```python
pool.network_configuration = batchmodels.NetworkConfiguration(
    endpoint_configuration=batchmodels.PoolEndpointConfiguration(
        inbound_nat_pools=[batchmodels.InboundNATPool(
            name='SSH',
            protocol='tcp',
            backend_port=22,
            frontend_port_range_start=4000,
            frontend_port_range_end=4100,
            network_security_group_rules=[
                batchmodels.NetworkSecurityGroupRule(
                    priority=170,
                    access=batchmodels.NetworkSecurityGroupRuleAccess.deny,
                    source_address_prefix='Internet'
                )
            ]
        )
        ]
    )
)
```

## <a name="example-allow-rdp-traffic-from-a-specific-ip-address"></a>Пример разрешение трафика RDP с определенного IP-адреса

В следующем фрагменте кода C# показано, как настроить конечную точку RDP на вычислительных узлах в пуле Windows, чтобы предоставить доступ RDP только с одного IP-адреса: *198.51.100.7*. Второе правило NSG запрещает трафик, для которого не соответствует IP-адрес.

```csharp
pool.NetworkConfiguration = new NetworkConfiguration
{
    EndpointConfiguration = new PoolEndpointConfiguration(new InboundNatPool[]
    {
        new InboundNatPool("RDP", InboundEndpointProtocol.Tcp, 3389, 7500, 8000, new NetworkSecurityGroupRule[]
        {   
            new NetworkSecurityGroupRule(179,NetworkSecurityGroupRuleAccess.Allow, "198.51.100.7"),
            new NetworkSecurityGroupRule(180,NetworkSecurityGroupRuleAccess.Deny, "*")
        })
    })    
};
```

## <a name="example-allow-ssh-traffic-from-a-specific-subnet"></a>Пример разрешение трафика SSH из определенной подсети

В следующем фрагменте кода Python показано, как настроить конечную точку SSH на вычислительных узлах в пуле Linux, чтобы разрешить доступ только из подсети по адресу *192.168.1.0/24*. Второе правило NSG запрещает трафик, который не соответствует подсети.

```python
pool.network_configuration = batchmodels.NetworkConfiguration(
    endpoint_configuration=batchmodels.PoolEndpointConfiguration(
        inbound_nat_pools=[batchmodels.InboundNATPool(
            name='SSH',
            protocol='tcp',
            backend_port=22,
            frontend_port_range_start=4000,
            frontend_port_range_end=4100,
            network_security_group_rules=[
                batchmodels.NetworkSecurityGroupRule(
                    priority=170,
                    access='allow',
                    source_address_prefix='192.168.1.0/24'
                ),
                batchmodels.NetworkSecurityGroupRule(
                    priority=175,
                    access='deny',
                    source_address_prefix='*'
                )
            ]
        )
        ]
    )
)
```

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте подробнее о [рабочем процессе и основных ресурсах пакетной службы](batch-service-workflow-features.md), таких как пулы, узлы, задания и задачи.
- Дополнительные сведения о правилах NSG см. в статье [Фильтрация сетевого трафика с помощью групп безопасности сети](../virtual-network/network-security-groups-overview.md).