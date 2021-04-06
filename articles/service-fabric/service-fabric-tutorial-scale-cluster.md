---
title: Масштабирование кластера Service Fabric в Azure
description: В этом руководстве описано, как масштабировать кластер Service Fabric из службы Azure и в нее, а также как очищать излишние ресурсы.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: cdc7ba8d6c83ae72ffb8f1afae3954b3a46dc6ec
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98788033"
---
# <a name="tutorial-scale-a-service-fabric-cluster-in-azure"></a>Руководство по Масштабирование кластера Service Fabric в Azure

Это руководство представляет собой третью часть серии. В нем показано, как масштабировать имеющийся кластер. Завершив работу с этим руководством, вы будете знать, как масштабировать кластер и очистить все остающиеся ресурсы.  Дополнительные сведения о масштабировании запущенного в Azure кластера, см. в руководстве по [масштабированию кластеров Service Fabric](service-fabric-cluster-scaling.md).

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Добавление и удаление узлов (горизонтальное увеличение и уменьшение масштаба)
> * Добавление и удаление типов узлов (горизонтальное увеличение и уменьшение масштаба)
> * увеличение ресурсов узла (вертикальное масштабирование).

Из этого цикла руководств вы узнаете, как выполнять следующие задачи:
> [!div class="checklist"]
> * создание безопасного [кластера Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) в Azure с помощью шаблона;
> * [Мониторинг кластера](service-fabric-tutorial-monitor-cluster.md)
> * увеличение или уменьшение масштаба кластера;
> * [Обновление среды выполнения кластера](service-fabric-tutorial-upgrade-cluster.md)
> * [Удаление кластера](service-fabric-tutorial-delete-cluster.md)


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы с этим руководством выполните следующие действия:

* Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Установите [Azure PowerShell](/powershell/azure/install-az-ps) или [Azure CLI](/cli/azure/install-azure-cli).
* Создание защищенного [кластера Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) в Azure

## <a name="important-considerations-and-guidelines"></a>Важные рекомендации и советы

Рабочие нагрузки приложений со временем изменяются, и вашим службам может потребоваться больше (или меньше) ресурсов.  [Добавьте или удалите узлы](#add-nodes-to-or-remove-nodes-from-a-node-type) из типа узла, чтобы увеличить или уменьшить объем ресурсов кластера.

Вам нужно добавить в кластер более 100 узлов?  Один тип узла или масштабируемый набор Service Fabric не может содержать более 100 узлов или виртуальных машин.  Чтобы включить в кластер более 100 узлов, [добавьте больше типов узла](#add-nodes-to-or-remove-nodes-from-a-node-type).

Есть ли у вашего приложения несколько служб, среди которых есть службы, которые должны быть общедоступными или доступными из Интернета?  Типичные приложения содержат интерфейсную службу шлюза, которая получает входные данные от клиента, и одну или несколько внутренних служб, взаимодействующих с интерфейсными службами. В этом случае мы рекомендуем [добавить в кластер не менее двух типов узла](#add-nodes-to-or-remove-nodes-from-a-node-type).  

У вас есть службы с разными требованиями к инфраструктуре, например требующие больше ОЗУ или тактов центрального процессора? Предположим, что приложение содержит интерфейсную и серверную службы. Интерфейсная служба может выполняться на виртуальных машинах меньшего размера (например, D2) с портами, открытыми для доступа через Интернет. Однако внутренняя служба, которая будет выполнять вычисления, должна размещаться на более крупных виртуальных машинах (D4, D6, D15) без выхода в Интернет. В этом случае мы рекомендуем вам [добавить в кластер два или более типов узла](#add-nodes-to-or-remove-nodes-from-a-node-type). Так у каждого типа узла будут свои собственные свойства, например возможность подключения к Интернету или размер виртуальной машины. Кроме того, можно будет независимо менять число виртуальных машин.

При масштабировании кластера Azure учитывайте следующие рекомендации:

* Один тип узла или масштабируемый набор Service Fabric не может содержать более 100 узлов или виртуальных машин.  Чтобы включить в кластер более 100 узлов, добавьте больше типов узла.
* Основные типы узла, выполняющие производственные рабочие нагрузки, должны иметь [уровень устойчивости][durability] Gold или Silver и содержать не менее пяти узлов.
* Дополнительные типы узла, выполняющие производственные рабочие нагрузки с отслеживанием состояния, должны содержать не менее пяти узлов.
* Дополнительные типы узла, выполняющие производственные рабочие нагрузки без отслеживания состояния, должны содержать не менее двух узлов.
* Любой тип узла с [уровнем устойчивости][durability] Gold или Silver всегда должен иметь пять или более узлов.
* При уменьшении масштаба типа первичного узла (т. е. при удалении из него узлов) не допускайте, чтобы число экземпляров оказалось меньше необходимого для обеспечения его [уровня надежности][reliability].

Дополнительные сведения см. в [руководстве по планированию загрузки кластера](service-fabric-cluster-capacity.md).

## <a name="export-the-template-for-the-resource-group"></a>Экспорт шаблона для группы ресурсов

Успешно создав безопасный [кластер Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) и настроив группу ресурсов, экспортируйте шаблон Resource Manager для группы ресурсов. Экспорт шаблона позволяет автоматизировать будущие развертывания кластера и связанных ресурсов, так как этот шаблон содержит всю необходимую инфраструктуру.  Дополнительные сведения об экспорте шаблонов см. в руководстве по [управлению группами ресурсов Azure Resource Manager с помощью портала Azure](../azure-resource-manager/management/manage-resource-groups-portal.md).

1. Откройте [портал Azure](https://portal.azure.com), перейдите к группе ресурсов, которая содержит нужный кластер (в нашем примере это кластер **sfclustertutorialgroup**). 

2. На панели слева выберите **Развертывания** или щелкните ссылку под элементом **Развертывания**. 

3. Выберите из этого списка последнее успешное развертывание.

4. На панели слева выберите **Шаблон**, а затем щелкните **Скачать**, чтобы экспортировать этот шаблон в ZIP-файл.  Сохраните шаблон и параметры на локальном компьютере.

## <a name="add-nodes-to-or-remove-nodes-from-a-node-type"></a>Добавление узлов в тип узла и удаление узлов из него

Свертывание и развертывание (горизонтальное масштабирование) изменяет количество узлов в кластере. При горизонтальном увеличении или уменьшении масштаба вы добавляете дополнительные экземпляры виртуальных машин в масштабируемый набор. Эти экземпляры становятся узлами, которые использует Service Fabric. Службе Service Fabric известно, когда в масштабируемый набор добавляются экземпляры (путем развертывания), и она реагирует автоматически. Кластер можно масштабировать в любое время, даже когда в нем выполняются рабочие нагрузки.

### <a name="update-the-template"></a>Обновление шаблона

[Экспортируйте файл с шаблоном и параметрами](#export-the-template-for-the-resource-group) из группы ресурсов с самым свежим развертыванием.  Откройте файл *parameters.json*.  Развернув кластер с помощью [примера шаблона][template] из этого руководства, вы получите кластер с тремя узлами и тремя параметрами для настройки количества узлов для каждого типа: *nt0InstanceCount*, *nt1InstanceCount* и *nt2InstanceCount*.  Например, параметр *Nt1InstanceCount* задает количество экземпляров для второго типа узла и определяет количество виртуальных машин в соответствующем масштабируемом наборе виртуальных машин.

Таким образом, значение *nt1InstanceCount* изменяет количество узлов для второго типа узла.  Не забывайте, что для типа узла нельзя увеличить количество более чем на 100 узлов.  Дополнительные типы узла, выполняющие производственные рабочие нагрузки с отслеживанием состояния, должны содержать не менее пяти узлов. Дополнительные типы узла, выполняющие производственные рабочие нагрузки без отслеживания состояния, должны содержать не менее двух узлов.

При масштабировании типа узла (удалении из него узлов) с [уровнем устойчивости][durability] Bronze следует [вручную удалить состояние для этих узлов](service-fabric-cluster-scale-in-out.md#manually-remove-vms-from-a-node-typevirtual-machine-scale-set).  Для уровней устойчивости Silver и Gold эти шаги платформа выполняет для вас автоматически.

### <a name="deploy-the-updated-template"></a>Развертывание обновленного шаблона
Сохраните изменения, внесенные в файлы *template.json* и *parameters.json*.  Чтобы развернуть обновленный шаблон, выполните следующую команду:

```powershell
New-AzResourceGroupDeployment -ResourceGroupName sfclustertutorialgroup -TemplateFile c:\temp\template.json -TemplateParameterFile c:\temp\parameters.json -Name "ChangingInstanceCount"
```
Или такую команду Azure CLI:
```azurecli
az deployment group create --resource-group sfclustertutorialgroup --template-file c:\temp\template.json --parameters c:\temp\parameters.json
```

## <a name="add-a-node-type-to-the-cluster"></a>Добавление типа узла в кластер

Каждый тип узла, определенный в запущенном в Azure кластере Service Fabric, настроен как [отдельный масштабируемый набор виртуальных машин](service-fabric-cluster-nodetypes.md). Затем каждым типом узла можно управлять отдельно. Все типы узлов можно масштабировать независимо друг от друга, открывать разные наборы портов и использовать различные метрики производительности. Вы можете изменять номера SKU для ОС на каждом узле кластера независимо друг от друга, но при этом нельзя использовать в одном кластере виртуальные машины под управлением Windows и Linux. Каждый тип узла (масштабируемый набор) не может содержать более 100 узлов.  Вы можете масштабировать кластер горизонтально более чем на 100 узлов, добавляя новые типы узла (масштабируемые наборы). Кластер можно масштабировать в любое время, даже когда в нем выполняются рабочие нагрузки.

### <a name="update-the-template"></a>Обновление шаблона

[Экспортируйте файл с шаблоном и параметрами](#export-the-template-for-the-resource-group) из группы ресурсов с самым свежим развертыванием.  Откройте файл *parameters.json*.  Если вы развернули кластер с помощью [примера шаблона][template] из этого руководства, в этом кластере существуют три типа узлов.  В этом разделе описано, как добавить четвертый тип узла, обновив шаблон Resource Manager и развернув его. 

Кроме нового типа узла, вы добавите связанный масштабируемый набор виртуальных машин (который выполняется в отдельной подсети виртуальной сети) и группу безопасности сети.  Для нового масштабируемого набора вы можете добавить новые или применить существующие ресурсы общедоступного IP-адреса и подсистемы балансировки нагрузки Azure.  Новый тип узла будет имеет [уровень устойчивости][durability] Silver и размер Standard_D2_V2.

В файле *template.json* добавьте следующие параметры:
```json
"nt3InstanceCount": {
    "defaultValue": 5,
    "type": "Int",
    "metadata": {
        "description": "Instance count for node type"
    }
},
"vmNodeType3Size": {
    "defaultValue": "Standard_D2_V2",
    "type": "String"
},
```

В файле *template.json* добавьте следующие переменные:
```json
"lbID3": "[resourceId('Microsoft.Network/loadBalancers',concat('LB','-', parameters('clusterName'),'-',variables('vmNodeType3Name')))]",
"lbIPConfig3": "[concat(variables('lbID3'),'/frontendIPConfigurations/LoadBalancerIPConfig')]",
"lbPoolID3": "[concat(variables('lbID3'),'/backendAddressPools/LoadBalancerBEAddressPool')]",
"lbProbeID3": "[concat(variables('lbID3'),'/probes/FabricGatewayProbe')]",
"lbHttpProbeID3": "[concat(variables('lbID3'),'/probes/FabricHttpGatewayProbe')]",
"lbNatPoolID3": "[concat(variables('lbID3'),'/inboundNatPools/LoadBalancerBEAddressNatPool')]",
"vmNodeType3Name": "[toLower(concat('NT4', variables('vmName')))]",
"vmStorageAccountName3": "[toLower(concat(uniqueString(resourceGroup().id), '1', '3' ))]",
"nt3applicationStartPort": "20000",
"nt3applicationEndPort": "30000",
"nt3ephemeralStartPort": "49152",
"nt3ephemeralEndPort": "65534",
"nt3fabricTcpGatewayPort": "19000",
"nt3fabricHttpGatewayPort": "19080",
"nt3reverseProxyEndpointPort": "19081",
"subnet3Name": "Subnet-3",
"subnet3Prefix": "10.0.3.0/24",
"subnet3Ref": "[concat(variables('vnetID'),'/subnets/',variables('subnet3Name'))]",
```

В файле *template.json* добавьте новую подсеть в ресурс виртуальной сети:
```json
{
    "type": "Microsoft.Network/virtualNetworks",
    "name": "[variables('virtualNetworkName')]",
    "apiVersion": "2018-08-01",
    "location": "[variables('computeLocation')]",
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    },
    "properties": {
        "addressSpace": {
            "addressPrefixes": [
                "[variables('addressPrefix')]"
            ]
        },
        "subnets": [
            ...
            {
                "name": "[variables('subnet3Name')]",
                "properties": {
                    "addressPrefix": "[variables('subnet3Prefix')]",
                    "networkSecurityGroup": {
                        "id": "[resourceId('Microsoft.Network/networkSecurityGroups', concat('nsg', variables('subnet3Name')))]"
                    }
                }
            }
        ]
    },
    "dependsOn": [
        ...
        "[concat('Microsoft.Network/networkSecurityGroups/', concat('nsg', variables('subnet3Name')))]"
    ]
},
```

В *template.json* добавьте новые ресурсы общедоступного IP-адреса и подсистемы балансировки нагрузки:
```json
{
    "type": "Microsoft.Network/publicIPAddresses",
    "name": "[concat(variables('lbIPName'),'-',variables('vmNodeType3Name'))]",
    "apiVersion": "2018-08-01",
    "location": "[variables('computeLocation')]",
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    },
    "properties": {
        "dnsSettings": {
            "domainNameLabel": "[concat(variables('dnsName'),'-','nt4')]"
        },
        "publicIPAllocationMethod": "Dynamic"
    }
},
        {
    "type": "Microsoft.Network/loadBalancers",
    "name": "[concat('LB','-', parameters('clusterName'),'-',variables('vmNodeType3Name'))]",
    "apiVersion": "2018-08-01",
    "location": "[variables('computeLocation')]",
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    },
    "properties": {
        "frontendIPConfigurations": [
            {
                "name": "LoadBalancerIPConfig",
                "properties": {
                    "publicIPAddress": {
                        "id": "[resourceId('Microsoft.Network/publicIPAddresses',concat(variables('lbIPName'),'-',variables('vmNodeType3Name')))]"
                    }
                }
            }
        ],
        "backendAddressPools": [
            {
                "name": "LoadBalancerBEAddressPool",
                "properties": {}
            }
        ],
        "loadBalancingRules": [
            {
                "name": "LBRule",
                "properties": {
                    "backendAddressPool": {
                        "id": "[variables('lbPoolID3')]"
                    },
                    "backendPort": "[variables('nt3fabricTcpGatewayPort')]",
                    "enableFloatingIP": "false",
                    "frontendIPConfiguration": {
                        "id": "[variables('lbIPConfig3')]"
                    },
                    "frontendPort": "[variables('nt3fabricTcpGatewayPort')]",
                    "idleTimeoutInMinutes": "5",
                    "probe": {
                        "id": "[variables('lbProbeID3')]"
                    },
                    "protocol": "tcp"
                }
            },
            {
                "name": "LBHttpRule",
                "properties": {
                    "backendAddressPool": {
                        "id": "[variables('lbPoolID3')]"
                    },
                    "backendPort": "[variables('nt3fabricHttpGatewayPort')]",
                    "enableFloatingIP": "false",
                    "frontendIPConfiguration": {
                        "id": "[variables('lbIPConfig3')]"
                    },
                    "frontendPort": "[variables('nt3fabricHttpGatewayPort')]",
                    "idleTimeoutInMinutes": "5",
                    "probe": {
                        "id": "[variables('lbHttpProbeID3')]"
                    },
                    "protocol": "tcp"
                }
            },
            {
                "name": "AppPortLBRule1",
                "properties": {
                    "backendAddressPool": {
                        "id": "[variables('lbPoolID3')]"
                    },
                    "backendPort": "[parameters('loadBalancedAppPort1')]",
                    "enableFloatingIP": "false",
                    "frontendIPConfiguration": {
                        "id": "[variables('lbIPConfig3')]"
                    },
                    "frontendPort": "[parameters('loadBalancedAppPort1')]",
                    "idleTimeoutInMinutes": "5",
                    "probe": {
                        "id": "[concat(variables('lbID3'),'/probes/AppPortProbe1')]"
                    },
                    "protocol": "tcp"
                }
            },
            {
                "name": "AppPortLBRule2",
                "properties": {
                    "backendAddressPool": {
                        "id": "[variables('lbPoolID3')]"
                    },
                    "backendPort": "[parameters('loadBalancedAppPort2')]",
                    "enableFloatingIP": "false",
                    "frontendIPConfiguration": {
                        "id": "[variables('lbIPConfig3')]"
                    },
                    "frontendPort": "[parameters('loadBalancedAppPort2')]",
                    "idleTimeoutInMinutes": "5",
                    "probe": {
                        "id": "[concat(variables('lbID3'),'/probes/AppPortProbe2')]"
                    },
                    "protocol": "tcp"
                }
            }
        ],
        "probes": [
            {
                "name": "FabricGatewayProbe",
                "properties": {
                    "intervalInSeconds": 5,
                    "numberOfProbes": 2,
                    "port": "[variables('nt3fabricTcpGatewayPort')]",
                    "protocol": "tcp"
                }
            },
            {
                "name": "FabricHttpGatewayProbe",
                "properties": {
                    "intervalInSeconds": 5,
                    "numberOfProbes": 2,
                    "port": "[variables('nt3fabricHttpGatewayPort')]",
                    "protocol": "tcp"
                }
            },
            {
                "name": "AppPortProbe1",
                "properties": {
                    "intervalInSeconds": 5,
                    "numberOfProbes": 2,
                    "port": "[parameters('loadBalancedAppPort1')]",
                    "protocol": "tcp"
                }
            },
            {
                "name": "AppPortProbe2",
                "properties": {
                    "intervalInSeconds": 5,
                    "numberOfProbes": 2,
                    "port": "[parameters('loadBalancedAppPort2')]",
                    "protocol": "tcp"
                }
            }
        ],
        "inboundNatPools": [
            {
                "name": "LoadBalancerBEAddressNatPool",
                "properties": {
                    "backendPort": "3389",
                    "frontendIPConfiguration": {
                        "id": "[variables('lbIPConfig3')]"
                    },
                    "frontendPortRangeEnd": "4500",
                    "frontendPortRangeStart": "3389",
                    "protocol": "tcp"
                }
            }
        ]
    },
    "dependsOn": [
        "[concat('Microsoft.Network/publicIPAddresses/',concat(variables('lbIPName'),'-',variables('vmNodeType3Name')))]"
    ]
},
```

В файле *template.json* добавьте новые ресурсы группы сетевой безопасности и масштабируемого набора виртуальных машин.  Свойство NodeTypeRef входит в число свойств расширения Service Fabric для масштабируемого набора виртуальных машин и сопоставляет указанный тип узла с этим масштабируемым набором.

```json
{
    "type": "Microsoft.Network/networkSecurityGroups",
    "name": "[concat('nsg', variables('subnet3Name'))]",
    "apiVersion": "2018-08-01",
    "location": "[resourceGroup().location]",
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    },
    "properties": {
        "securityRules": [
            {
                "name": "allowSvcFabSMB",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "445",
                    "direction": "Inbound",
                    "priority": 3950,
                    "protocol": "*",
                    "sourceAddressPrefix": "VirtualNetwork",
                    "sourcePortRange": "*",
                    "description": "allow SMB traffic within the net, used by fabric to move packages around"
                }
            },
            {
                "name": "allowSvcFabCluser",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "1025-1027",
                    "direction": "Inbound",
                    "priority": 3920,
                    "protocol": "*",
                    "sourceAddressPrefix": "VirtualNetwork",
                    "sourcePortRange": "*",
                    "description": "allow ports within vnet that are used by the fabric to talk between nodes"
                }
            },
            {
                "name": "allowSvcFabEphemeral",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "[concat(variables('nt3ephemeralStartPort'), '-', variables('nt3ephemeralEndPort'))]",
                    "direction": "Inbound",
                    "priority": 3930,
                    "protocol": "*",
                    "sourceAddressPrefix": "VirtualNetwork",
                    "sourcePortRange": "*",
                    "description": "allow fabric ephemeral ports within the vnet"
                }
            },
            {
                "name": "allowSvcFabPortal",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "[variables('nt3fabricHttpGatewayPort')]",
                    "direction": "Inbound",
                    "priority": 3900,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "allow port used to access the fabric cluster web portal"
                }
            },
            {
                "name": "allowSvcFabClient",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "[variables('nt3fabricTcpGatewayPort')]",
                    "direction": "Inbound",
                    "priority": 3910,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "allow port used by the fabric client (includes powershell)"
                }
            },
            {
                "name": "allowSvcFabApplication",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "[concat(variables('nt3applicationStartPort'), '-', variables('nt3applicationEndPort'))]",
                    "direction": "Inbound",
                    "priority": 3940,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "allow fabric application ports within the vnet"
                }
            },
            {
                "name": "blockAll",
                "properties": {
                    "access": "Deny",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "*",
                    "direction": "Inbound",
                    "priority": 4095,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "block all traffic except what we've explicitly allowed"
                }
            },
            {
                "name": "allowVNetRDP",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "3389",
                    "direction": "Inbound",
                    "priority": 3960,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "allow RDP within the net"
                }
            },
            {
                "name": "allowSvcFabReverseProxy",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "[variables('nt3reverseProxyEndpointPort')]",
                    "direction": "Inbound",
                    "priority": 3980,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "allow port used to access the fabric cluster using reverse proxy"
                }
            },
            {
                "name": "allowAppPort1",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "[parameters('loadBalancedAppPort1')]",
                    "direction": "Inbound",
                    "priority": 2001,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "allow public application port 1"
                }
            },
            {
                "name": "allowAppPort2",
                "properties": {
                    "access": "Allow",
                    "destinationAddressPrefix": "*",
                    "destinationPortRange": "[parameters('loadBalancedAppPort2')]",
                    "direction": "Inbound",
                    "priority": 2002,
                    "protocol": "*",
                    "sourceAddressPrefix": "*",
                    "sourcePortRange": "*",
                    "description": "allow public application port 2"
                }
            }
        ]
    }
},
{
    "type": "Microsoft.Compute/virtualMachineScaleSets",
    "sku": {
        "name": "[parameters('vmNodeType3Size')]",
        "capacity": "[parameters('nt3InstanceCount')]",
        "tier": "Standard"
    },
    "name": "[variables('vmNodeType3Name')]",
    "apiVersion": "2018-10-01",
    "location": "[variables('computeLocation')]",
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    },
    "properties": {
        "overprovision": "[variables('overProvision')]",
        "upgradePolicy": {
            "mode": "Automatic"
        },
        "virtualMachineProfile": {
            "extensionProfile": {
                "extensions": [
                    {
                        "name": "[concat(variables('vmNodeType3Name'),'OMS')]",
                        "properties": {
                            "publisher": "Microsoft.EnterpriseCloud.Monitoring",
                            "type": "MicrosoftMonitoringAgent",
                            "typeHandlerVersion": "1.0",
                            "autoUpgradeMinorVersion": true,
                            "settings": {
                                "workspaceId": "[reference(resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename')), '2015-11-01-preview').customerId]"
                            },
                            "protectedSettings": {
                                "workspaceKey": "[listKeys(resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename')),'2015-11-01-preview').primarySharedKey]"
                            }
                        }
                    },
                    {
                        "name": "[concat('ServiceFabricNodeVmExt','_vmNodeType3Name')]",
                        "properties": {
                            "type": "ServiceFabricNode",
                            "autoUpgradeMinorVersion": true,
                            "protectedSettings": {
                                "StorageAccountKey1": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('supportLogStorageAccountName')),'2015-05-01-preview').key1]",
                                "StorageAccountKey2": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('supportLogStorageAccountName')),'2015-05-01-preview').key2]"
                            },
                            "publisher": "Microsoft.Azure.ServiceFabric",
                            "settings": {
                                "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
                                "nodeTypeRef": "[variables('vmNodeType3Name')]",
                                "dataPath": "D:\\SvcFab",
                                "durabilityLevel": "Silver",
                                "enableParallelJobs": true,
                                "nicPrefixOverride": "[variables('subnet3Prefix')]",
                                "certificate": {
                                    "thumbprint": "[parameters('certificateThumbprint')]",
                                    "x509StoreName": "[parameters('certificateStoreValue')]"
                                }
                            },
                            "typeHandlerVersion": "1.0"
                        }
                    },
                    {
                        "name": "[concat('VMDiagnosticsVmExt','_vmNodeType3Name')]",
                        "properties": {
                            "type": "IaaSDiagnostics",
                            "autoUpgradeMinorVersion": true,
                            "protectedSettings": {
                                "storageAccountName": "[variables('applicationDiagnosticsStorageAccountName')]",
                                "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('applicationDiagnosticsStorageAccountName')),'2015-05-01-preview').key1]",
                                "storageAccountEndPoint": "https://core.windows.net/"
                            },
                            "publisher": "Microsoft.Azure.Diagnostics",
                            "settings": {
                                "WadCfg": {
                                    "DiagnosticMonitorConfiguration": {
                                        "overallQuotaInMB": "50000",
                                        "EtwProviders": {
                                            "EtwEventSourceProviderConfiguration": [
                                                {
                                                    "provider": "Microsoft-ServiceFabric-Actors",
                                                    "scheduledTransferKeywordFilter": "1",
                                                    "scheduledTransferPeriod": "PT5M",
                                                    "DefaultEvents": {
                                                        "eventDestination": "ServiceFabricReliableActorEventTable"
                                                    }
                                                },
                                                {
                                                    "provider": "Microsoft-ServiceFabric-Services",
                                                    "scheduledTransferPeriod": "PT5M",
                                                    "DefaultEvents": {
                                                        "eventDestination": "ServiceFabricReliableServiceEventTable"
                                                    }
                                                }
                                            ],
                                            "EtwManifestProviderConfiguration": [
                                                {
                                                    "provider": "cbd93bc2-71e5-4566-b3a7-595d8eeca6e8",
                                                    "scheduledTransferLogLevelFilter": "Information",
                                                    "scheduledTransferKeywordFilter": "4611686018427387904",
                                                    "scheduledTransferPeriod": "PT5M",
                                                    "DefaultEvents": {
                                                        "eventDestination": "ServiceFabricSystemEventTable"
                                                    }
                                                }
                                            ]
                                        }
                                    }
                                },
                                "StorageAccount": "[variables('applicationDiagnosticsStorageAccountName')]"
                            },
                            "typeHandlerVersion": "1.5"
                        }
                    },
                    {
                        "name": "[concat('VMIaaSAntimalware','_vmNodeType3Name')]",
                        "properties": {
                            "publisher": "Microsoft.Azure.Security",
                            "type": "IaaSAntimalware",
                            "typeHandlerVersion": "1.5",
                            "settings": {
                                "AntimalwareEnabled": "true",
                                "Exclusions": {
                                    "Paths": "D:\\SvcFab;D:\\SvcFab\\Log;C:\\Program Files\\Microsoft Service Fabric",
                                    "Processes": "Fabric.exe;FabricHost.exe;FabricInstallerService.exe;FabricSetup.exe;FabricDeployer.exe;ImageBuilder.exe;FabricGateway.exe;FabricDCA.exe;FabricFAS.exe;FabricUOS.exe;FabricRM.exe;FileStoreService.exe"
                                },
                                "RealtimeProtectionEnabled": "true",
                                "ScheduledScanSettings": {
                                    "isEnabled": "true",
                                    "scanType": "Quick",
                                    "day": "7",
                                    "time": "120"
                                }
                            },
                            "protectedSettings": null
                        }
                    }
                ]
            },
            "networkProfile": {
                "networkInterfaceConfigurations": [
                    {
                        "name": "[concat(variables('nicName'), '-2')]",
                        "properties": {
                            "ipConfigurations": [
                                {
                                    "name": "[concat(variables('nicName'),'-',2)]",
                                    "properties": {
                                        "loadBalancerBackendAddressPools": [
                                            {
                                                "id": "[variables('lbPoolID3')]"
                                            }
                                        ],
                                        "loadBalancerInboundNatPools": [
                                            {
                                                "id": "[variables('lbNatPoolID3')]"
                                            }
                                        ],
                                        "subnet": {
                                            "id": "[variables('subnet3Ref')]"
                                        }
                                    }
                                }
                            ],
                            "primary": true
                        }
                    }
                ]
            },
            "osProfile": {
                "adminPassword": "[parameters('adminPassword')]",
                "adminUsername": "[parameters('adminUsername')]",
                "computernamePrefix": "[variables('vmNodeType3Name')]",
                "secrets": [
                    {
                        "sourceVault": {
                            "id": "[parameters('sourceVaultValue')]"
                        },
                        "vaultCertificates": [
                            {
                                "certificateStore": "[parameters('certificateStoreValue')]",
                                "certificateUrl": "[parameters('certificateUrlValue')]"
                            }
                        ]
                    }
                ]
            },
            "storageProfile": {
                "imageReference": {
                    "publisher": "[parameters('vmImagePublisher')]",
                    "offer": "[parameters('vmImageOffer')]",
                    "sku": "[parameters('vmImageSku')]",
                    "version": "[parameters('vmImageVersion')]"
                },
                "osDisk": {
                    "caching": "ReadOnly",
                    "createOption": "FromImage",
                    "managedDisk": {
                        "storageAccountType": "[parameters('storageAccountType')]"
                    }
                }
            }
        }
    },
    "dependsOn": [
        "[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]",
        "[concat('Microsoft.Network/loadBalancers/', concat('LB','-', parameters('clusterName'),'-',variables('vmNodeType3Name')))]",
        "[concat('Microsoft.Storage/storageAccounts/', variables('supportLogStorageAccountName'))]",
        "[concat('Microsoft.Storage/storageAccounts/', variables('applicationDiagnosticsStorageAccountName'))]"
    ]
},
```

В файле *template.json* обновите ресурс кластера, добавив новый тип узла:
```json
{
    "type": "Microsoft.ServiceFabric/clusters",
    "name": "[parameters('clusterName')]",
    "apiVersion": "2018-02-01",
    "location": "[parameters('clusterLocation')]",
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    },
    "properties": {
        "nodeTypes": [
            ...
            {
                "name": "[variables('vmNodeType3Name')]",
                "applicationPorts": {
                    "endPort": "[variables('nt3applicationEndPort')]",
                    "startPort": "[variables('nt3applicationStartPort')]"
                },
                "clientConnectionEndpointPort": "[variables('nt3fabricTcpGatewayPort')]",
                "durabilityLevel": "Silver",
                "ephemeralPorts": {
                    "endPort": "[variables('nt3ephemeralEndPort')]",
                    "startPort": "[variables('nt3ephemeralStartPort')]"
                },
                "httpGatewayEndpointPort": "[variables('nt3fabricHttpGatewayPort')]",
                "isPrimary": false,
                "reverseProxyEndpointPort": "[variables('nt3reverseProxyEndpointPort')]",
                "vmInstanceCount": "[parameters('nt3InstanceCount')]"
            }
        ],    
    }
}                
```

В файле *template.json* добавьте следующие параметры и переменные:
```json
"nt3InstanceCount": {
    "Value": 5    
},
"vmNodeType3Size": {
    "Value": "Standard_D2_V2"
},
```

### <a name="deploy-the-updated-template"></a>Развертывание обновленного шаблона
Сохраните изменения, внесенные в файлы *template.json* и *parameters.json*.  Чтобы развернуть обновленный шаблон, выполните следующую команду:

```powershell
New-AzResourceGroupDeployment -ResourceGroupName sfclustertutorialgroup -TemplateFile c:\temp\template.json -TemplateParameterFile c:\temp\parameters.json -Name "AddingNodeType"
```
Или такую команду Azure CLI:
```azurecli
az deployment group create --resource-group sfclustertutorialgroup --template-file c:\temp\template.json --parameters c:\temp\parameters.json
```

## <a name="remove-a-node-type-from-the-cluster"></a>Удаление типа узла из кластера
После создания кластера Service Fabric вы можете масштабировать кластер по горизонтали, удалив тип узла (масштабируемый набор виртуальных машин) и все его узлы. Кластер можно масштабировать в любое время, даже когда в нем выполняются рабочие нагрузки. Вместе с кластером автоматически масштабируются ваши приложения.

> [!WARNING]
> Использовать команду Remove-AzServiceFabricNodeType для удаления типа узла из рабочего кластера на регулярной основе не рекомендуется. Это опасная команда, так как она удаляет ресурс масштабируемого набора виртуальных машин, связанный с типом узла. 

Чтобы удалить тип узла, запустите командлет [Remove-AzServiceFabricNodeType](/powershell/module/az.servicefabric/remove-azservicefabricnodetype).  Тип узла должен иметь [уровень устойчивости][durability] Silver или Gold. Этот командлет удаляет масштабируемый набор, связанный с типом узла, и на это требуется некоторое время.  Затем запустите на каждом из удаляемых узлов командлет [Remove-ServiceFabricNodeState](/powershell/module/servicefabric/remove-servicefabricnodestate), который удаляет состояние узла и узлы из кластера. Если на таком узле есть службы, они сначала перемещаются на другой узел. Если диспетчеру кластеров не удается найти узел для реплики или службы, тогда операция откладывается или блокируется.

```powershell
$groupname = "sfclustertutorialgroup"
$nodetype = "nt4vm"
$clustername = "mysfcluster123"

Remove-AzServiceFabricNodeType -Name $clustername  -NodeType $nodetype -ResourceGroupName $groupname

Connect-ServiceFabricCluster -ConnectionEndpoint mysfcluster123.eastus.cloudapp.azure.com:19000 `
          -KeepAliveIntervalInSec 10 `
          -X509Credential -ServerCertThumbprint <thumbprint> `
          -FindType FindByThumbprint -FindValue <thumbprint> `
          -StoreLocation CurrentUser -StoreName My

$nodes = Get-ServiceFabricNode | Where-Object {$_.NodeType -eq $nodetype} | Sort-Object { $_.NodeName.Substring($_.NodeName.LastIndexOf('_') + 1) } -Descending

Foreach($node in $nodes)
{
    Remove-ServiceFabricNodeState -NodeName $node.NodeName -TimeoutSec 300 -Force 
}
```

## <a name="increase-node-resources"></a>Увеличение ресурсов узла 
После создания кластера Service Fabric можно масштабировать тип узла кластера вертикально (изменение ресурсов узлов) или обновить ОС виртуальных машин типа узла, заменив исходный тип узла новым типом (с обновленным SKU виртуальной машины или образом ОС). Дополнительные сведения см. в статье [Вертикальное масштабирование типа узла Azure Service Fabric](service-fabric-scale-up-primary-node-type.md).

> [!IMPORTANT]
> Ни в коем случае не пытайтесь выполнить замену VM виртуальной машины или образа ОС на месте — это опасная и неподдерживаемая операция.

Если это невозможно, создайте кластер и [восстановите состояние приложения](service-fabric-reliable-services-backup-restore.md) (если это применимо) из старого кластера. Не нужно восстанавливать состояние службы системы, так как при развертывании приложений в новом кластере они будут созданы заново. Если приложения в кластере выполнялись без отслеживания состояния, достаточно развернуть их в новом кластере. Восстановление выполнять не нужно.

### <a name="update-the-template"></a>Обновление шаблона

[Экспортируйте файл с шаблоном и параметрами](#export-the-template-for-the-resource-group) из группы ресурсов с самым свежим развертыванием.  Откройте файл *parameters.json*.  Если вы развернули кластер с помощью [примера шаблона][template] из этого руководства, в этом кластере существуют три типа узлов.  

Размер виртуальных машин во втором типе узла задается параметром *vmNodeType1Size*.  Измените значение параметра *vmNodeType1Size* со Standard_D2_V2 на [Standard_D3_V2](../virtual-machines/dv2-dsv2-series.md), чтобы удвоить ресурсы для каждого экземпляра виртуальной машины.

Номер SKU виртуальной машины для всех трех типов узла задается параметром *vmImageSku*.  Изменение номера SKU для типа узла также следует выполнять с крайней осторожностью. Мы не рекомендуем делать это для основного типа узла.

### <a name="deploy-the-updated-template"></a>Развертывание обновленного шаблона
Сохраните изменения, внесенные в файлы *template.json* и *parameters.json*.  Чтобы развернуть обновленный шаблон, выполните следующую команду:

```powershell
New-AzResourceGroupDeployment -ResourceGroupName sfclustertutorialgroup -TemplateFile c:\temp\template.json -TemplateParameterFile c:\temp\parameters.json -Name "ScaleUpNodeType"
```
Или такую команду Azure CLI:
```azurecli
az deployment group create --resource-group sfclustertutorialgroup --template-file c:\temp\template.json --parameters c:\temp\parameters.json
```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> * Добавление и удаление узлов (горизонтальное увеличение и уменьшение масштаба)
> * Добавление и удаление типов узлов (горизонтальное увеличение и уменьшение масштаба)
> * увеличение ресурсов узла (вертикальное масштабирование).

Теперь перейдите к следующему руководству, чтобы узнать, как обновить среду выполнения кластера.
> [!div class="nextstepaction"]
> [Обновление среды выполнения кластера](service-fabric-tutorial-upgrade-cluster.md)

[durability]: service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster
[reliability]: service-fabric-cluster-capacity.md#reliability-characteristics-of-the-cluster
[template]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Windows-3-NodeTypes-Secure-NSG/AzureDeploy.json
[parameters]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Windows-3-NodeTypes-Secure-NSG/AzureDeploy.Parameters.json
