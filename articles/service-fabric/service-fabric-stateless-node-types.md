---
title: Развертывание типов узлов только без отслеживания состояния в кластере Service Fabric
description: Узнайте, как создавать и развертывать типы узлов без отслеживания состояния в кластере Azure Service Fabric.
author: peterpogorski
ms.topic: conceptual
ms.date: 09/25/2020
ms.author: pepogors
ms.openlocfilehash: eb19005019a6e4e878f6b0bd6a145048d4a2804c
ms.sourcegitcommit: 18a91f7fe1432ee09efafd5bd29a181e038cee05
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103563782"
---
# <a name="deploy-an-azure-service-fabric-cluster-with-stateless-only-node-types-preview"></a>Развертывание кластера Azure Service Fabric с типами узлов только без отслеживания состояния (Предварительная версия)
Service Fabric типы узлов поставляются с предположением, что в некоторый момент времени службы с отслеживанием состояния могут размещаться на узлах. Типы узлов без отслеживания состояния отменяют это предположение для типа узла, тем самым позволяя типу узла использовать другие функции, такие как более быстрые операции масштабирования, поддержку автоматического обновления ОС на бронзовой устойчивости и масштабирование до более чем 100 узлов в одном масштабируемом наборе виртуальных машин.

* Типы первичных узлов не могут быть настроены без отслеживания состояния
* Типы узлов без отслеживания состояния поддерживаются только с уровнями устойчивости бронзовой.
* Типы узлов без отслеживания состояния поддерживаются только в среде выполнения Service Fabric версии 7.1.409 или выше.


Примеры шаблонов доступны: [шаблон типов узлов без отслеживания состояния Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates)

## <a name="enabling-stateless-node-types-in-service-fabric-cluster"></a>Включение типов узлов без отслеживания состояния в кластере Service Fabric
Чтобы задать в ресурсе кластера один или несколько типов узлов без отслеживания состояния, задайте для свойства **без сохранения состояния** значение "true". При развертывании кластера Service Fabric с типами узлов без отслеживания состояния не забывайте, что в ресурсе кластера должен быть по крайней мере один тип первичного узла.

* Service Fabric кластерный ресурс apiVersion должен иметь значение "2020-12-01-Preview" или более поздней версии.

```json
{
    "nodeTypes": [
    {
        "name": "[parameters('vmNodeType0Name')]",
        "applicationPorts": {
            "endPort": "[parameters('nt0applicationEndPort')]",
            "startPort": "[parameters('nt0applicationStartPort')]"
        },
        "clientConnectionEndpointPort": "[parameters('nt0fabricTcpGatewayPort')]",
        "durabilityLevel": "Silver",
        "ephemeralPorts": {
            "endPort": "[parameters('nt0ephemeralEndPort')]",
            "startPort": "[parameters('nt0ephemeralStartPort')]"
        },
        "httpGatewayEndpointPort": "[parameters('nt0fabricHttpGatewayPort')]",
        "isPrimary": true,
        "isStateless": false,
        "vmInstanceCount": "[parameters('nt0InstanceCount')]"
    },
    {
        "name": "[parameters('vmNodeType1Name')]",
        "applicationPorts": {
            "endPort": "[parameters('nt1applicationEndPort')]",
            "startPort": "[parameters('nt1applicationStartPort')]"
        },
        "clientConnectionEndpointPort": "[parameters('nt1fabricTcpGatewayPort')]",
        "durabilityLevel": "Bronze",
        "ephemeralPorts": {
            "endPort": "[parameters('nt1ephemeralEndPort')]",
            "startPort": "[parameters('nt1ephemeralStartPort')]"
        },
        "httpGatewayEndpointPort": "[parameters('nt1fabricHttpGatewayPort')]",
        "isPrimary": false,
        "isStateless": true,
        "vmInstanceCount": "[parameters('nt1InstanceCount')]"
    }    
    ],
}
```

## <a name="configuring-virtual-machine-scale-set-for-stateless-node-types"></a>Настройка масштабируемого набора виртуальных машин для типов узлов без отслеживания состояния
Чтобы включить типы узлов без отслеживания состояния, необходимо настроить базовый ресурс масштабируемого набора виртуальных машин следующим образом:

* Свойство Value  **singlePlacementGroup** , которое должно иметь значение **false** , если требуется масштабировать до 100 виртуальных машин.
* **Режим** **upgradePolicy** для масштабируемого набора должен быть установлен в значение " **Пошаговое**".
* Для режима последовательного обновления необходимо настроить расширение работоспособности приложения или проверки работоспособности. Настройте пробу работоспособности с конфигурацией по умолчанию для типов узлов без отслеживания состояния, как показано ниже. После развертывания приложений в типе узла можно изменить порты проверки работоспособности и расширения работоспособности, чтобы отслеживать работоспособность приложений.

>[!NOTE]
> Необходимо, чтобы число доменов сбоя платформы обновлялось до 5, если тип узла без отслеживания состояния поддерживается масштабируемым набором виртуальных машин, охватывающим несколько зон. Дополнительные сведения см. в этом [шаблоне](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/15-VM-2-NodeTypes-Windows-Stateless-CrossAZ-Secure) .
> 
> **platformFaultDomainCount: 5**
```json
{
    "apiVersion": "2018-10-01",
    "type": "Microsoft.Compute/virtualMachineScaleSets",
    "name": "[parameters('vmNodeType1Name')]",
    "location": "[parameters('computeLocation')]",
    "properties": {
        "overprovision": "[variables('overProvision')]",
        "upgradePolicy": {
          "mode": "Rolling",
          "automaticOSUpgradePolicy": {
            "enableAutomaticOSUpgrade": true
          }
        }
    }
    "virtualMachineProfile": {
    "extensionProfile": {
    "extensions": [
    {
    "name": "[concat(parameters('vmNodeType1Name'),'_ServiceFabricNode')]",
    "properties": {
        "type": "ServiceFabricNode",
        "autoUpgradeMinorVersion": false,
        "publisher": "Microsoft.Azure.ServiceFabric",
        "settings": {
            "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
            "nodeTypeRef": "[parameters('vmNodeType1Name')]",
            "dataPath": "D:\\\\SvcFab",
            "durabilityLevel": "Bronze",
            "certificate": {
                "thumbprint": "[parameters('certificateThumbprint')]",
                "x509StoreName": "[parameters('certificateStoreValue')]"
            },
            "systemLogUploadSettings": {
                "Enabled": true
            },
        },
        "typeHandlerVersion": "1.0"
    }
    },
    {
        "type": "extensions",
        "name": "HealthExtension",
        "properties": {
            "publisher": "Microsoft.ManagedServices",
            "type": "ApplicationHealthWindows",
            "autoUpgradeMinorVersion": true,
            "typeHandlerVersion": "1.0",
            "settings": {
            "protocol": "tcp",
            "port": "19000"
            }
            }
        },
    ]
}
```

## <a name="networking-requirements"></a>Требования к сети
### <a name="public-ip-and-load-balancer-resource"></a>Общедоступный IP-адрес и Load Balancer ресурс
Чтобы включить масштабирование до более чем 100 виртуальных машин в ресурсе масштабируемого набора виртуальных машин, подсистема балансировки нагрузки и ресурс IP, на который ссылается этот масштабируемый набор виртуальных машин, должны использовать номер SKU " *стандартный* ". При создании подсистемы балансировки нагрузки или ресурса IP-адреса без свойства SKU будет создан базовый номер SKU, который не поддерживает масштабирование до более чем 100 виртуальных машин. Балансировщик нагрузки уровня "Стандартный" будет блокировать весь трафик извне по умолчанию; чтобы разрешить внешний трафик, необходимо развернуть NSG в подсети.

```json
{
    "apiVersion": "2018-11-01",
    "type": "Microsoft.Network/publicIPAddresses",
    "name": "[concat('LB','-', parameters('clusterName')]",
    "location": "[parameters('computeLocation')]",
    "sku": {
        "name": "Standard"
    }
}
{
    "apiVersion": "2018-11-01",
    "type": "Microsoft.Network/loadBalancers",
    "name": "[concat('LB','-', parameters('clusterName')]", 
    "location": "[parameters('computeLocation')]",
    "dependsOn": [
        "[concat('Microsoft.Network/networkSecurityGroups/', concat('nsg', parameters('subnet0Name')))]"
    ],
    "properties": {
        "addressSpace": {
            "addressPrefixes": [
                "[parameters('addressPrefix')]"
            ]
        },
        "subnets": [
        {
            "name": "[parameters('subnet0Name')]",
            "properties": {
                "addressPrefix": "[parameters('subnet0Prefix')]",
                "networkSecurityGroup": {
                "id": "[resourceId('Microsoft.Network/networkSecurityGroups', concat('nsg', parameters('subnet0Name')))]"
              }
            }
          }
        ]
    },
    "sku": {
        "name": "Standard"
    }
}
```

>[!NOTE]
> Невозможно выполнить изменение SKU на месте в ресурсах общедоступного IP-адреса и балансировщика нагрузки. Если выполняется миграция с существующих ресурсов, имеющих базовый номер SKU, см. раздел миграция этой статьи.

### <a name="virtual-machine-scale-set-nat-rules"></a>Правила NAT для масштабируемого набора виртуальных машин
Правила NAT для входящего трафика подсистемы балансировки нагрузки должны соответствовать пулам NAT из масштабируемого набора виртуальных машин. Каждый масштабируемый набор виртуальных машин должен иметь уникальный пул NAT для входящего трафика.

```json
{
"inboundNatPools": [
    {
        "name": "LoadBalancerBEAddressNatPool0",
        "properties": {
            "backendPort": "3389",
            "frontendIPConfiguration": {
                "id": "[variables('lbIPConfig0')]"
            },
            "frontendPortRangeEnd": "50999",
            "frontendPortRangeStart": "50000",
            "protocol": "tcp"
        }
    },
    {
        "name": "LoadBalancerBEAddressNatPool1",
        "properties": {
            "backendPort": "3389",
            "frontendIPConfiguration": {
                "id": "[variables('lbIPConfig0')]"
            },
            "frontendPortRangeEnd": "51999",
            "frontendPortRangeStart": "51000",
            "protocol": "tcp"
        }
    },
    {
        "name": "LoadBalancerBEAddressNatPool2",
        "properties": {
            "backendPort": "3389",
            "frontendIPConfiguration": {
                "id": "[variables('lbIPConfig0')]"
            },
            "frontendPortRangeEnd": "52999",
            "frontendPortRangeStart": "52000",
            "protocol": "tcp"
        }
    }
    ]
}
```

### <a name="standard-sku-load-balancer-outbound-rules"></a>Стандартные правила SKU Load Balancer исходящих подключений
Load Balancer (цен. категория "Стандартный") и стандартный общедоступный IP-адрес предоставляют новые возможности и различные варианты поведения для исходящего подключения по сравнению с использованием основных номеров SKU. Если вам требуется исходящее подключение при работе с номерами SKU категории "Стандартный", вы должны явно определить это подключение с использованием общедоступных IP-адресов категории "Стандартный" или общедоступного Load Balancer категории "Стандартный". Дополнительные сведения см. в статье [Исходящие подключения](../load-balancer/load-balancer-outbound-connections.md) и [Azure Load Balancer (цен. Категория "Стандартный")](../load-balancer/load-balancer-overview.md).

>[!NOTE]
> Стандартный шаблон ссылается на NSG, который разрешает по умолчанию весь исходящий трафик. Входящий трафик ограничивается портами, необходимыми для операций управления Service Fabric. Правила NSG можно изменить в соответствии с вашими требованиями.

>[!NOTE]
> Любой кластер Service Fabric, использующий номер SKU уровня "Стандартный", должен гарантировать, что каждый тип узла имеет правило, разрешающее исходящий трафик через порт 443. Это необходимо для завершения установки кластера, и любое развертывание без такого правила завершится ошибкой.



### <a name="migrate-to-using-stateless-node-types-from-a-cluster-using-a-basic-sku-load-balancer-and-a-basic-sku-ip"></a>Переход на использование типов узлов без отслеживания состояния в кластере с использованием базового номера SKU Load Balancer и IP-адреса базового SKU
Для всех сценариев миграции необходимо добавить новый тип узла только без отслеживания состояния. Существующий тип узла нельзя перенести в режим "только без отслеживания состояния".

Чтобы выполнить миграцию кластера, который использовал Load Balancer и IP-адрес с номером SKU "базовый", сначала необходимо создать совершенно новый Load Balancer и IP-ресурс, используя номер SKU "Стандартный". Невозможно обновить эти ресурсы на месте.

На новую балансировку нагрузки и IP-адрес следует ссылаться в новых типах узлов без отслеживания состояния, которые вы хотите использовать. В приведенном выше примере добавляются новые ресурсы масштабируемого набора виртуальных машин, которые будут использоваться для типов узлов без отслеживания состояния. Эти масштабируемые наборы виртуальных машин ссылаются на только что созданную балансировку нагрузки и IP-адрес и помечаются как типы узлов без отслеживания состояния в ресурсе кластера Service Fabric.

Для начала необходимо добавить новые ресурсы в существующий шаблон диспетчер ресурсов. К этим ресурсам относятся:
* Ресурс общедоступного IP-адреса, использующий SKU "Стандартный".
* Load Balancer ресурс с использованием номера SKU "Стандартный".
* NSG, на который ссылается подсеть, в которой развертываются масштабируемые наборы виртуальных машин.

После завершения развертывания ресурсов можно приступить к отключению узлов в типе узла, которые необходимо удалить из исходного кластера.

>[!NOTE]
> При использовании автомасштабирования с NodeType без отслеживания состояния с использованием бронзовой устойчивости после масштабирования состояние узла не очищается автоматически. Чтобы очистить Нодестате узлов во время автомасштабирования, рекомендуется использовать [вспомогательный модуль автомасштабирования Service Fabric](https://github.com/Azure/service-fabric-autoscale-helper) .

## <a name="next-steps"></a>Дальнейшие действия 
* [Надежные службы](service-fabric-reliable-services-introduction.md)
* [Типы узлов и масштабируемые наборы виртуальных машин](service-fabric-cluster-nodetypes.md)

