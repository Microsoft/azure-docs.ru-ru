---
title: Настройка параметров сети для управляемых кластеров Service Fabric (Предварительная версия)
description: Узнайте, как настроить управляемый кластер Service Fabric для правил NSG, доступа к портам RDP, правил балансировки нагрузки и многого другого.
ms.topic: how-to
ms.date: 03/02/2021
ms.openlocfilehash: e17251523c0720665c4c6f5b7811304eebc9923e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101746991"
---
# <a name="configure-network-settings-for-service-fabric-managed-clusters-preview"></a>Настройка параметров сети для управляемых кластеров Service Fabric (Предварительная версия)

Service Fabric управляемые кластеры создаются с конфигурацией сети по умолчанию. Эта конфигурация состоит из обязательных правил для необходимых функциональных возможностей кластера и нескольких необязательных правил, которые призваны упростить настройку клиентов.

Помимо конфигурации сети по умолчанию, правила сети можно изменить в соответствии с потребностями вашего сценария.

## <a name="nsg-rules-guidance"></a>Руководство по правилам NSG

Учитывайте эти соображения при создании новых правил NSG для управляемого кластера.

* Service Fabric управляемые кластеры Зарезервируйте диапазон приоритетов правил NSG от 0 до 999 для важных функциональных возможностей. Нельзя создавать настраиваемые правила NSG со значением приоритета менее 1000.
* Service Fabric управляемых кластерах для создания необязательных правил NSG резервируется диапазон приоритетов 3001 – 4000. Эти правила добавляются автоматически для быстрого и простого создания конфигураций. Эти правила можно переопределить, добавив настраиваемые правила NSG в диапазоне приоритета 1000 в 3000.
* Настраиваемые правила NSG должны иметь приоритет в диапазоне от 1000 до 3000.

## <a name="apply-nsg-rules"></a>Применить правила NSG

При использовании классических (не управляемых) Service Fabric кластеров для [применения правил группы безопасности сети (NSG) к кластеру](https://github.com/Azure/azure-quickstart-templates/tree/master/service-fabric-secure-nsg-cluster-65-node-3-nodetype)необходимо объявить и управлять отдельным ресурсом *Microsoft. Network/networkSecurityGroups* . Service Fabric управляемые кластеры позволяют назначать правила NSG непосредственно в кластерном ресурсе шаблона развертывания.

Чтобы назначить правила NSG, используйте свойство [нетворксекуритирулес](/azure/templates/microsoft.servicefabric/managedclusters#managedclusterproperties-object) ресурса *Microsoft. ServiceFabric/манажедклустерс* (версия `2021-01-01-preview` или более поздней версии). Пример:

```json
            "apiVersion": "2021-01-01-preview",
            "type": "Microsoft.ServiceFabric/managedclusters",
            ...
            "properties": {
                ...
                "networkSecurityRules" : [
                    {
                        "name": "AllowCustomers",
                        "protocol": "*",
                        "sourcePortRange": "*",
                        "sourceAddressPrefix": "Internet",
                        "destinationAddressPrefix": "*",
                        "destinationPortRange": "33000-33499",
                        "access": "Allow",
                        "priority": 2001,
                        "direction": "Inbound" 
                    },
                    {
                        "name": "AllowARM",
                        "protocol": "*",
                        "sourcePortRange": "*",
                        "sourceAddressPrefix": "AzureResourceManager",
                        "destinationAddressPrefix": "*",
                        "destinationPortRange": "33500-33699",
                        "access": "Allow",
                        "priority": 2002,
                        "direction": "Inbound" 
                    },
                    {
                        "name": "DenyCustomers",
                        "protocol": "*",
                        "sourcePortRange": "*",
                        "sourceAddressPrefix": "Internet",
                        "destinationAddressPrefix": "*",
                        "destinationPortRange": "33700-33799",
                        "access": "Deny",
                        "priority": 2003,
                        "direction": "Outbound"
                    },
                    {
                        "name": "DenyRDP",
                        "protocol": "*",
                        "sourcePortRange": "*",
                        "sourceAddressPrefix": "*",
                        "destinationAddressPrefix": "VirtualNetwork",
                        "destinationPortRange": "3389",
                        "access": "Deny",
                        "priority": 2004,
                        "direction": "Inbound",
                        "description": "Override for optional SFMC_AllowRdpPort rule. This is required in tests to avoid Sev2 incident for security policy violation."
                    }
                ],
                "fabricSettings": [
                ...
                ]
            }
```

## <a name="rdp-ports"></a>Порты RDP

Service Fabric управляемые кластеры не разрешают доступ к портам RDP по умолчанию. Вы можете открыть порты RDP в Интернете, задав следующее свойство для Service Fabric управляемого ресурса кластера.

```json
"allowRDPAccess": true 
```

Если свойство allowRDPAccess имеет значение true, в развертывание кластера будет добавлено следующее правило NSG.  

```json
{
    "name": "SFMC_AllowRdpPort", 
    "type": "Microsoft.Network/networkSecurityGroups/securityRules",
    "properties": {
        "description": "Optional rule to open RDP ports.",
        "protocol": "tcp",
        "sourcePortRange": "*",
        "sourceAddressPrefix": "*",
        "destinationAddressPrefix": "VirtualNetwork",
        "access": "Allow",
        "priority": 3002,
        "direction": "Inbound",
        "sourcePortRanges": [],
        "destinationPortRange": "3389"
    }
}
```

## <a name="clientconnection-and-httpgatewayconnection-ports"></a>Порты Клиентконнектион и Хттпгатевайконнектион

### <a name="nsg-rule-sfmc_allowservicefabricgatewaytosfrp"></a>Правило NSG: SFMC_AllowServiceFabricGatewayToSFRP

Добавляется правило NSG по умолчанию, разрешающее поставщику ресурсов Service Fabric доступ к Клиентконнектионпорт и Хттпгатевайконнектионпорт кластера. Это правило разрешает доступ к портам через тег службы "ServiceFabric".

>[!NOTE]
>Это правило всегда добавляется и не может быть переопределено.

```json
{ 
    "name": "SFMC_AllowServiceFabricGatewayToSFRP", 
    "type": "Microsoft.Network/networkSecurityGroups/securityRules", 
    "properties": { 
        "description": "This is required rule to allow SFRP to connect to the cluster. This rule cannot be overridden.", 
        "protocol": "TCP", 
        "sourcePortRange": "*", 
        "sourceAddressPrefix": "ServiceFabric", 
        "destinationAddressPrefix": "VirtualNetwork", 
        "access": "Allow", 
        "priority": 500, 
        "direction": "Inbound", 
        "sourcePortRanges": [], 
        "destinationPortRanges": [ 
            "19000", 
            "19080" 
        ] 
    } 
}
```

### <a name="nsg-rule-sfmc_allowservicefabricgatewayports"></a>Правило NSG: SFMC_AllowServiceFabricGatewayPorts

Это необязательное правило NSG, разрешающее доступ к Клиентконнектионпорт и Хттпгатевайпорт из Интернета. Это правило позволяет клиентам получить доступ к SFX, подключиться к кластеру с помощью PowerShell и использовать Service Fabric конечные точки API кластера извне. 

>[!NOTE]
>Это правило не будет добавлено, если имеется настраиваемое правило с тем же доступом, направлением и значениями протокола для того же порта. Это правило можно переопределить с помощью настраиваемых правил NSG. 

```json
{ 
    "name": "SFMC_AllowServiceFabricGatewayPorts", 
    "type": "Microsoft.Network/networkSecurityGroups/securityRules", 
    "properties": { 
        "description": "Optional rule to open SF cluster gateway ports. To override add a custom NSG rule for gateway ports in priority range 1000-3000.", 
        "protocol": "tcp", 
        "sourcePortRange": "*", 
        "sourceAddressPrefix": "*", 
        "destinationAddressPrefix": "VirtualNetwork", 
        "access": "Allow", 
        "priority": 3001, 
        "direction": "Inbound", 
        "sourcePortRanges": [], 
        "destinationPortRanges": [ 
            "19000", 
            "19080" 
        ] 
    } 
}
```

## <a name="load-balancer-ports"></a>Порты подсистемы балансировки нагрузки

Service Fabric управляемых кластеров создайте правило NSG в диапазоне приоритетов по умолчанию для всех портов балансировщика нагрузки, настроенных в разделе "ЛоадбаланЦингрулес" в разделе "свойства *манажедклустер* ". Это правило открывает порты балансировки нагрузки для входящего трафика из Интернета.  

>[!NOTE]
>Это правило добавляется в необязательный диапазон приоритета и может быть переопределено путем добавления пользовательских правил NSG.

```json
{
    "name": "SFMC_AllowLoadBalancedPorts",
    "type": "Microsoft.Network/networkSecurityGroups/securityRules",
    "properties": {
        "description": "Optional rule to open LB ports",
        "protocol": "*", 
        "sourcePortRange": "*",
        "sourceAddressPrefix": "*",
        "destinationAddressPrefix": "VirtualNetwork",
        "access": "Allow",
        "priority": 3003,
        "direction": "Inbound",
        "sourcePortRanges": [],
        "destinationPortRanges": [
        "80", "8080", "4343"
        ]
    }
}
```

## <a name="load-balancer-probes"></a>Зонды подсистемы балансировки нагрузки

Service Fabric управляемые кластеры автоматически создают проверки балансировщика нагрузки для портов шлюза структуры, а также все порты, настроенные в разделе "ЛоадбаланЦингрулес" свойств управляемого кластера.

```json
{ 
  "value": [ 
    { 
        "name": "FabricTcpGateway", 
        "properties": { 
            "provisioningState": "Succeeded", 
            "protocol": "Tcp", 
            "port": 19000, 
            "intervalInSeconds": 5, 
            "numberOfProbes": 2, 
            "loadBalancingRules": [ 
                { 
                    "id": "<>"
                } 
            ] 
        }, 
        "type": "Microsoft.Network/loadBalancers/probes" 
    }, 
    { 
        "name": "FabricHttpGateway", 
        "properties": { 
            "provisioningState": "Succeeded", 
            "protocol": "Tcp", 
            "port": 19080, 
            "intervalInSeconds": 5, 
            "numberOfProbes": 2, 
            "loadBalancingRules": [ 
                { 
                    "id": "<>" 
                } 
            ]
        },
        "type": "Microsoft.Network/loadBalancers/probes" 
    },
    {
        "name": "probe1_tcp_8080", 
        "properties": { 
            "provisioningState": "Succeeded", 
            "protocol": "Tcp", 
            "port": 8080, 
            "intervalInSeconds": 5, 
            "numberOfProbes": 2, 
            "loadBalancingRules": [ 
            { 
                "id": "<>" 
            } 
        ] 
      }, 
      "type": "Microsoft.Network/loadBalancers/probes" 
    } 
  ] 
} 
```

## <a name="next-steps"></a>Дальнейшие действия

[Параметры конфигурации управляемого кластера Service Fabric](how-to-managed-cluster-configuration.md)

[Общие сведения об управляемых кластерах Service Fabric](overview-managed-cluster.md)
