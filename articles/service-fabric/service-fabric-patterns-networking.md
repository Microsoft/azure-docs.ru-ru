---
title: Шаблоны сетей для Azure Service Fabric
description: Описываются распространенные схемы сетевых подключений для Service Fabric, а также создание кластера с использованием сетевых компонентов Azure.
ms.topic: conceptual
ms.date: 01/19/2018
ms.openlocfilehash: 20bd5e931307725016c3e2ad69dae91214b2caab
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87421473"
---
# <a name="service-fabric-networking-patterns"></a>Схемы сетевых подключений Service Fabric
Кластер Azure Service Fabric можно интегрировать с другими сетевыми компонентами Azure. В этой статье показано, как создавать кластеры, использующие следующие компоненты:

- [существующая виртуальная сеть или подсеть](#existingvnet);
- [Статический общедоступный IP-адрес](#staticpublicip)
- [Подсистема балансировки нагрузки только для внутреннего использования](#internallb)
- [внутренняя и внешняя подсистемы балансировки нагрузки](#internalexternallb).

Service Fabric выполняется в стандартном масштабируемом наборе виртуальных машин. Любые функции, которые можно использовать в масштабируемом наборе виртуальных машин, можно также использовать и в кластере Service Fabric. Сетевые компоненты шаблонов Azure Resource Manager для масштабируемых наборов виртуальных машин и Service Fabric идентичны. После развертывания в существующей виртуальной сети легко внедрить другие сетевые компоненты, такие как Azure ExpressRoute, VPN-шлюз Azure, группа безопасности сети и пиринговая виртуальная сеть.

### <a name="allowing-the-service-fabric-resource-provider-to-query-your-cluster"></a>Предоставление поставщику Service Fabric ресурсов разрешения на запрос к кластеру

Служба Service Fabric является уникальной среди других сетевых компонентов в одном аспекте. [Портал Azure](https://portal.azure.com) при внутренней обработке использует поставщик ресурсов Service Fabric, чтобы выполнять вызовы к кластеру для получения сведений об узлах и приложениях. Для работы поставщика ресурсов Service Fabric требуется открытый входящий доступ к HTTP-порту шлюза (по умолчанию — порт 19080) на конечной точке управления. [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) использует конечную точку управления для управления кластером. Поставщик ресурсов Service Fabric также использует этот порт для запроса сведений о вашем кластере, отображаемых на портале Azure. 

Если порт 19080 недоступен поставщику ресурсов Service Fabric, то на портале вы увидите сообщение, например *Nodes Not Found* (Узлы не найдены), и список узлов и приложений будет пуст. Если вы хотите видеть сведения о кластере на портале Azure, то ваша подсистема балансировки нагрузки должна предоставлять общедоступный IP-адрес, а группа безопасности сети должна разрешать входящий трафик через порт 19080. Если в вашей конфигурации эти требования не выполнены, то портал Azure не будет отображать состояние кластера.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="templates"></a>Шаблоны

Все шаблоны Service Fabric можно найти в репозитории [GitHub](https://github.com/Azure/service-fabric-scripts-and-templates/tree/master/templates/networking). Вы можете развертывать эти шаблоны "как есть", используя следующие команды PowerShell. В случае развертывания шаблона существующей виртуальной сети Azure или шаблона статического общедоступного IP-адреса необходимо сначала ознакомиться с разделом [Начальная настройка](#initialsetup) этой статьи.

<a id="initialsetup"></a>
## <a name="initial-setup"></a>Начальная настройка

### <a name="existing-virtual-network"></a>Существующая виртуальная сеть

В приведенном ниже примере мы начнем с существующей виртуальной сети ExistingRG-vnet в группе ресурсов **ExistingRG**. В ней есть подсеть default. Эти ресурсы по умолчанию создаются при создании стандартной виртуальной машины с помощью портала Azure. Вы можете создать виртуальную сеть и подсеть, не создавая виртуальную машину, но основная цель добавления кластера в существующую виртуальную сеть —обеспечить сетевые подключения к другим виртуальным машинам. Создание виртуальной машины дает конкретный пример того, как обычно используется существующая виртуальная сеть. Если кластер Service Fabric использует только внутреннюю подсистему балансировки нагрузки без общедоступного IP-адреса, то в качестве безопасного *сервера администрирования* можно использовать виртуальную машину с общедоступным IP-адресом.

### <a name="static-public-ip-address"></a>Статический общедоступный IP-адрес

Статический общедоступный IP-адрес обычно является выделенным ресурсом, управление которым осуществляется отдельно от виртуальной машины или машин, которым он назначен. Он подготавливается в выделенной группе сетевых ресурсов (в отличие от самой группы ресурсов кластера Service Fabric). Создайте статический общедоступный IP-адрес staticIP1 в той же группе ресурсов ExistingRG с помощью портала Azure или PowerShell.

```powershell
PS C:\Users\user> New-AzPublicIpAddress -Name staticIP1 -ResourceGroupName ExistingRG -Location westus -AllocationMethod Static -DomainNameLabel sfnetworking

Name                     : staticIP1
ResourceGroupName        : ExistingRG
Location                 : westus
Id                       : /subscriptions/1237f4d2-3dce-1236-ad95-123f764e7123/resourceGroups/ExistingRG/providers/Microsoft.Network/publicIPAddresses/staticIP1
Etag                     : W/"fc8b0c77-1f84-455d-9930-0404ebba1b64"
ResourceGuid             : 77c26c06-c0ae-496c-9231-b1a114e08824
ProvisioningState        : Succeeded
Tags                     :
PublicIpAllocationMethod : Static
IpAddress                : 40.83.182.110
PublicIpAddressVersion   : IPv4
IdleTimeoutInMinutes     : 4
IpConfiguration          : null
DnsSettings              : {
                             "DomainNameLabel": "sfnetworking",
                             "Fqdn": "sfnetworking.westus.cloudapp.azure.com"
                           }
```

### <a name="service-fabric-template"></a>Шаблон Service Fabric

В примерах в этой статье мы используем шаблон Service Fabric template.json. Этот шаблон можно скачать с портала перед созданием кластера с помощью стандартного мастера портала. Можно также использовать один из [примеров шаблонов](https://github.com/Azure-Samples/service-fabric-cluster-templates), например шаблон [кластера Service Fabric с пятью узлами](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure).

<a id="existingvnet"></a>
## <a name="existing-virtual-network-or-subnet"></a>Существующая виртуальная сеть или подсеть

1. Измените параметр подсети, указав имя существующей подсети, а затем добавьте два новых параметра для ссылки на существующую виртуальную сеть:

    ```json
        "subnet0Name": {
                "type": "string",
                "defaultValue": "default"
            },
            "existingVNetRGName": {
                "type": "string",
                "defaultValue": "ExistingRG"
            },

            "existingVNetName": {
                "type": "string",
                "defaultValue": "ExistingRG-vnet"
            },
            /*
            "subnet0Name": {
                "type": "string",
                "defaultValue": "Subnet-0"
            },
            "subnet0Prefix": {
                "type": "string",
                "defaultValue": "10.0.0.0/24"
            },*/
    ```

   Вы также можете закомментировать параметр с именем «virtualNetworkName», чтобы он не предложит ввести имя виртуальной сети дважды в колонке развертывание кластера в портал Azure.

2. Закомментируйте атрибут `nicPrefixOverride` в `Microsoft.Compute/virtualMachineScaleSets`, так как вы используете существующую подсеть и отключили эту переменную на шаге 1.

    ```json
            /*"nicPrefixOverride": "[parameters('subnet0Prefix')]",*/
    ```

3. Измените переменную `vnetID`, чтобы она указывала на существующую виртуальную сеть:

    ```json
            /*old "vnetID": "[resourceId('Microsoft.Network/virtualNetworks',parameters('virtualNetworkName'))]",*/
            "vnetID": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', parameters('existingVNetRGName'), '/providers/Microsoft.Network/virtualNetworks/', parameters('existingVNetName'))]",
    ```

4. Удалите `Microsoft.Network/virtualNetworks` из ресурсов, чтобы платформа Azure не создала виртуальную сеть:

    ```json
    /*{
    "apiVersion": "[variables('vNetApiVersion')]",
    "type": "Microsoft.Network/virtualNetworks",
    "name": "[parameters('virtualNetworkName')]",
    "location": "[parameters('computeLocation')]",
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
                    "addressPrefix": "[parameters('subnet0Prefix')]"
                }
            }
        ]
    },
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    }
    },*/
    ```

5. Закомментируйте виртуальную сеть из атрибута `dependsOn` в `Microsoft.Compute/virtualMachineScaleSets`, чтобы не зависеть от создания виртуальной сети:

    ```json
    "apiVersion": "[variables('vmssApiVersion')]",
    "type": "Microsoft.Computer/virtualMachineScaleSets",
    "name": "[parameters('vmNodeType0Name')]",
    "location": "[parameters('computeLocation')]",
    "dependsOn": [
        /*"[concat('Microsoft.Network/virtualNetworks/', parameters('virtualNetworkName'))]",
        */
        "[Concat('Microsoft.Storage/storageAccounts/', variables('uniqueStringArray0')[0])]",

    ```

6. Разверните шаблон.

    ```powershell
    New-AzResourceGroup -Name sfnetworkingexistingvnet -Location westus
    New-AzResourceGroupDeployment -Name deployment -ResourceGroupName sfnetworkingexistingvnet -TemplateFile C:\SFSamples\Final\template\_existingvnet.json
    ```

    После развертывания ваша виртуальная сеть должна содержать виртуальные машины нового масштабируемого набора. А тип узла масштабируемого набора виртуальных машин должен отображать существующие виртуальную сеть и подсеть. Вы также можете подключиться по протоколу удаленного рабочего стола (RDP) к виртуальной машине, которая уже была размещена в виртуальной сети, и проверить связь с виртуальными машинами нового масштабируемого набора:

    ```
    C:>\Users\users>ping 10.0.0.5 -n 1
    C:>\Users\users>ping NOde1000000 -n 1
    ```

Еще один пример развертывания нового масштабируемого набора, не относящийся к Service Fabric, приведен [здесь](https://github.com/gbowerman/azure-myriad/tree/main/existing-vnet).


<a id="staticpublicip"></a>
## <a name="static-public-ip-address"></a>Статический общедоступный IP-адрес

1. Добавьте параметры для имени группы ресурсов существующего статического IP-адреса, его имени и полного доменного имени (FQDN):

    ```json
    "existingStaticIPResourceGroup": {
                "type": "string"
            },
            "existingStaticIPName": {
                "type": "string"
            },
            "existingStaticIPDnsFQDN": {
                "type": "string"
    }
    ```

2. Удалите параметр `dnsName` (он уже есть в статическом IP-адресе).

    ```json
    /*
    "dnsName": {
        "type": "string"
    },
    */
    ```

3. Добавьте переменную для ссылки на существующий статический IP-адрес:

    ```json
    "existingStaticIP": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', parameters('existingStaticIPResourceGroup'), '/providers/Microsoft.Network/publicIPAddresses/', parameters('existingStaticIPName'))]",
    ```

4. Удалите `Microsoft.Network/publicIPAddresses` из ресурсов, чтобы платформа Azure не создала IP-адрес:

    ```json
    /*
    {
        "apiVersion": "[variables('publicIPApiVersion')]",
        "type": "Microsoft.Network/publicIPAddresses",
        "name": "[concat(parameters('lbIPName'),)'-', '0')]",
        "location": "[parameters('computeLocation')]",
        "properties": {
            "dnsSettings": {
                "domainNameLabel": "[parameters('dnsName')]"
            },
            "publicIPAllocationMethod": "Dynamic"        
        },
        "tags": {
            "resourceType": "Service Fabric",
            "clusterName": "[parameters('clusterName')]"
        }
    }, */
    ```

5. Закомментируйте IP-адрес из атрибута `dependsOn` в `Microsoft.Network/loadBalancers`, чтобы не зависеть от создания IP-адреса:

    ```json
    "apiVersion": "[variables('lbIPApiVersion')]",
    "type": "Microsoft.Network/loadBalancers",
    "name": "[concat('LB', '-', parameters('clusterName'), '-', parameters('vmNodeType0Name'))]",
    "location": "[parameters('computeLocation')]",
    /*
    "dependsOn": [
        "[concat('Microsoft.Network/publicIPAddresses/', concat(parameters('lbIPName'), '-', '0'))]"
    ], */
    "properties": {
    ```

6. В ресурсе `Microsoft.Network/loadBalancers` измените элемент `publicIPAddress` в группе элементов `frontendIPConfigurations`, чтобы ссылаться на существующий статический IP-адрес, а не на только что созданный:

    ```json
                "frontendIPConfigurations": [
                        {
                            "name": "LoadBalancerIPConfig",
                            "properties": {
                                "publicIPAddress": {
                                    /*"id": "[resourceId('Microsoft.Network/publicIPAddresses',concat(parameters('lbIPName'),'-','0'))]"*/
                                    "id": "[variables('existingStaticIP')]"
                                }
                            }
                        }
                    ],
    ```

7. В ресурсе `Microsoft.ServiceFabric/clusters` измените элемент `managementEndpoint`, указав полное доменное имя DNS-сервера статического IP-адреса. При использовании защищенного кластера обязательно измените *http://* на *https://*. (Обратите внимание, что этот шаг применяется только к кластерам Service Fabric. Если вы используете масштабируемый набор виртуальных машин, то пропустите этот шаг.)

    ```json
                    "fabricSettings": [],
                    /*"managementEndpoint": "[concat('http://',reference(concat(parameters('lbIPName'),'-','0')).dnsSettings.fqdn,':',parameters('nt0fabricHttpGatewayPort'))]",*/
                    "managementEndpoint": "[concat('http://',parameters('existingStaticIPDnsFQDN'),':',parameters('nt0fabricHttpGatewayPort'))]",
    ```

8. Разверните шаблон.

    ```powershell
    New-AzResourceGroup -Name sfnetworkingstaticip -Location westus

    $staticip = Get-AzPublicIpAddress -Name staticIP1 -ResourceGroupName ExistingRG

    $staticip

    New-AzResourceGroupDeployment -Name deployment -ResourceGroupName sfnetworkingstaticip -TemplateFile C:\SFSamples\Final\template\_staticip.json -existingStaticIPResourceGroup $staticip.ResourceGroupName -existingStaticIPName $staticip.Name -existingStaticIPDnsFQDN $staticip.DnsSettings.Fqdn
    ```

После развертывания вы можете увидеть, что подсистема балансировки нагрузки привязана к общедоступному статическому IP-адресу из другой группы ресурсов. Конечная точка подключения клиента Service Fabric и конечная точка [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) указывают на полное доменное имя DNS статического IP-адреса.

<a id="internallb"></a>
## <a name="internal-only-load-balancer"></a>Подсистема балансировки нагрузки только для внутреннего использования

В этом сценарии внешняя подсистема балансировки нагрузки в шаблоне по умолчанию Service Fabric заменяется подсистемой балансировки нагрузки только для внутреннего использования. Сведения о влиянии портал Azure и поставщика ресурсов Service Fabric см. [выше в статье](#allowing-the-service-fabric-resource-provider-to-query-your-cluster) .

1. Удалите параметр `dnsName` (он не требуется).

    ```json
    /*
    "dnsName": {
        "type": "string"
    },
    */
    ```

2. При необходимости, если используется метод статического выделения, можно добавить параметр статического IP-адреса. При использовании метода динамического выделения этот шаг выполнять не нужно.

    ```json
            "internalLBAddress": {
                "type": "string",
                "defaultValue": "10.0.0.250"
            }
    ```

3. Удалите `Microsoft.Network/publicIPAddresses` из ресурсов, чтобы платформа Azure не создала IP-адрес:

    ```json
    /*
    {
        "apiVersion": "[variables('publicIPApiVersion')]",
        "type": "Microsoft.Network/publicIPAddresses",
        "name": "[concat(parameters('lbIPName'),)'-', '0')]",
        "location": "[parameters('computeLocation')]",
        "properties": {
            "dnsSettings": {
                "domainNameLabel": "[parameters('dnsName')]"
            },
            "publicIPAllocationMethod": "Dynamic"        
        },
        "tags": {
            "resourceType": "Service Fabric",
            "clusterName": "[parameters('clusterName')]"
        }
    }, */
    ```

4. Удалите атрибут `dependsOn` IP-адреса в `Microsoft.Network/loadBalancers`, чтобы не зависеть от создания IP-адреса. Добавьте атрибут `dependsOn` виртуальной сети, так как теперь подсистема балансировки нагрузки зависит от подсети в виртуальной сети:

    ```json
                "apiVersion": "[variables('lbApiVersion')]",
                "type": "Microsoft.Network/loadBalancers",
                "name": "[concat('LB','-', parameters('clusterName'),'-',parameters('vmNodeType0Name'))]",
                "location": "[parameters('computeLocation')]",
                "dependsOn": [
                    /*"[concat('Microsoft.Network/publicIPAddresses/',concat(parameters('lbIPName'),'-','0'))]"*/
                    "[concat('Microsoft.Network/virtualNetworks/',parameters('virtualNetworkName'))]"
                ],
    ```

5. Измените параметр `frontendIPConfigurations` подсистемы балансировки нагрузки: вместо `publicIPAddress` укажите подсеть и `privateIPAddress`. `privateIPAddress` использует предварительно определенный статический внутренний IP-адрес. Чтобы использовать динамический IP-адрес, удалите элемент `privateIPAddress` и измените значение параметра `privateIPAllocationMethod` на **Dynamic**.

    ```json
                "frontendIPConfigurations": [
                        {
                            "name": "LoadBalancerIPConfig",
                            "properties": {
                                /*
                                "publicIPAddress": {
                                    "id": "[resourceId('Microsoft.Network/publicIPAddresses',concat(parameters('lbIPName'),'-','0'))]"
                                } */
                                "subnet" :{
                                    "id": "[variables('subnet0Ref')]"
                                },
                                "privateIPAddress": "[parameters('internalLBAddress')]",
                                "privateIPAllocationMethod": "Static"
                            }
                        }
                    ],
    ```

6. В ресурсе `Microsoft.ServiceFabric/clusters` измените значение параметра `managementEndpoint`, указав адрес внутренней подсистемы балансировки нагрузки. При использовании защищенного кластера обязательно измените *http://* на *https://*. (Обратите внимание, что этот шаг применяется только к кластерам Service Fabric. Если вы используете масштабируемый набор виртуальных машин, то пропустите этот шаг.)

    ```json
                    "fabricSettings": [],
                    /*"managementEndpoint": "[concat('http://',reference(concat(parameters('lbIPName'),'-','0')).dnsSettings.fqdn,':',parameters('nt0fabricHttpGatewayPort'))]",*/
                    "managementEndpoint": "[concat('http://',reference(variables('lbID0')).frontEndIPConfigurations[0].properties.privateIPAddress,':',parameters('nt0fabricHttpGatewayPort'))]",
    ```

7. Разверните шаблон.

    ```powershell
    New-AzResourceGroup -Name sfnetworkinginternallb -Location westus

    New-AzResourceGroupDeployment -Name deployment -ResourceGroupName sfnetworkinginternallb -TemplateFile C:\SFSamples\Final\template\_internalonlyLB.json
    ```

После развертывания подсистема балансировки нагрузки будет использовать частный статический IP-адрес 10.0.0.250. Если в этой виртуальной сети имеется другая виртуальная машина, то можно перейти к внутренней конечной точке [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md). Обратите внимание, что она подключается к одному из узлов, управляемых подсистемой балансировки нагрузки.

<a id="internalexternallb"></a>
## <a name="internal-and-external-load-balancer"></a>Внутренняя и внешняя подсистемы балансировки нагрузки

В этом сценарии к существующей внешней подсистеме балансировки нагрузки для одного типа узла добавляется внутренняя подсистема балансировки нагрузки для того же типа узла. Внутренний порт, подключенный к внутреннему пулу адресов, может быть назначен только одной подсистеме балансировки нагрузки. Выберите, какая подсистема балансировки нагрузки будет управлять портами вашего приложения, а какая — конечными точками управления (порты 19000 и 19080). Если вы поместили конечные точки управления во внутреннюю подсистему балансировки нагрузки, помните об ограничениях поставщика ресурсов Service Fabric, описанных [выше в этой статье](#allowing-the-service-fabric-resource-provider-to-query-your-cluster). В используемом примере конечные точки управления, остаются во внешней подсистеме балансировки нагрузки. Также мы добавляем порт приложения 80 и размещаем его во внутренней подсистеме балансировки нагрузки.

В кластере с узлами двух типов один из типов размещается во внешней подсистеме балансировки нагрузки. А другой тип — во внутренней. Чтобы использовать кластер с узлами двух типов, в созданном на портале шаблоне с двумя узлами (который содержит две подсистемы балансировки нагрузки) замените вторую подсистему балансировки нагрузки подсистемой балансировки нагрузки только для внутреннего использования. Дополнительные сведения см. в разделе [Подсистема балансировки нагрузки только для внутреннего использования](#internallb).

1. Добавьте параметр статического IP-адреса для внутренней подсистемы балансировки нагрузки (сведения об использовании динамического IP-адреса см. в предыдущих разделах этой статьи).

    ```json
            "internalLBAddress": {
                "type": "string",
                "defaultValue": "10.0.0.250"
            }
    ```

2. Добавьте параметр порта приложения 80.

3. Чтобы добавить внутренние версии существующих переменных сети, скопируйте и вставьте их, а также добавьте в имя окончание -Int:

    ```json
    /* Add internal load balancer networking variables */
            "lbID0-Int": "[resourceId('Microsoft.Network/loadBalancers', concat('LB','-', parameters('clusterName'),'-',parameters('vmNodeType0Name'), '-Internal'))]",
            "lbIPConfig0-Int": "[concat(variables('lbID0-Int'),'/frontendIPConfigurations/LoadBalancerIPConfig')]",
            "lbPoolID0-Int": "[concat(variables('lbID0-Int'),'/backendAddressPools/LoadBalancerBEAddressPool')]",
            "lbProbeID0-Int": "[concat(variables('lbID0-Int'),'/probes/FabricGatewayProbe')]",
            "lbHttpProbeID0-Int": "[concat(variables('lbID0-Int'),'/probes/FabricHttpGatewayProbe')]",
            "lbNatPoolID0-Int": "[concat(variables('lbID0-Int'),'/inboundNatPools/LoadBalancerBEAddressNatPool')]",
            /* Internal load balancer networking variables end */
    ```

4. Если вы используете созданный на портале шаблон с портом приложения 80, то он по умолчанию добавляет AppPort1 (порт 80) во внешнюю подсистему балансировки нагрузки. В этом случае удалите AppPort1 из `loadBalancingRules` и проб внешней подсистемы балансировки нагрузки, чтобы вы могли добавить этот порт во внутреннюю подсистему балансировки нагрузки:

    ```json
    "loadBalancingRules": [
        {
            "name": "LBHttpRule",
            "properties":{
                "backendAddressPool": {
                    "id": "[variables('lbPoolID0')]"
                },
                "backendPort": "[parameters('nt0fabricHttpGatewayPort')]",
                "enableFloatingIP": "false",
                "frontendIPConfiguration": {
                    "id": "[variables('lbIPConfig0')]"            
                },
                "frontendPort": "[parameters('nt0fabricHttpGatewayPort')]",
                "idleTimeoutInMinutes": "5",
                "probe": {
                    "id": "[variables('lbHttpProbeID0')]"
                },
                "protocol": "tcp"
            }
        } /* Remove AppPort1 from the external load balancer.
        {
            "name": "AppPortLBRule1",
            "properties": {
                "backendAddressPool": {
                    "id": "[variables('lbPoolID0')]"
                },
                "backendPort": "[parameters('loadBalancedAppPort1')]",
                "enableFloatingIP": "false",
                "frontendIPConfiguration": {
                    "id": "[variables('lbIPConfig0')]"            
                },
                "frontendPort": "[parameters('loadBalancedAppPort1')]",
                "idleTimeoutInMinutes": "5",
                "probe": {
                    "id": "[concate(variables('lbID0'), '/probes/AppPortProbe1')]"
                },
                "protocol": "tcp"
            }
        }*/

    ],
    "probes": [
        {
            "name": "FabricGatewayProbe",
            "properties": {
                "intervalInSeconds": 5,
                "numberOfProbes": 2,
                "port": "[parameters('nt0fabricTcpGatewayPort')]",
                "protocol": "tcp"
            }
        },
        {
            "name": "FabricHttpGatewayProbe",
            "properties": {
                "intervalInSeconds": 5,
                "numberOfProbes": 2,
                "port": "[parameters('nt0fabricHttpGatewayPort')]",
                "protocol": "tcp"
            }
        } /* Remove AppPort1 from the external load balancer.
        {
            "name": "AppPortProbe1",
            "properties": {
                "intervalInSeconds": 5,
                "numberOfProbes": 2,
                "port": "[parameters('loadBalancedAppPort1')]",
                "protocol": "tcp"
            }
        } */

    ],
    "inboundNatPools": [
    ```

5. Добавьте второй ресурс `Microsoft.Network/loadBalancers`. Он похож на внутреннюю подсистему балансировки нагрузки, созданную в разделе [Подсистема балансировки нагрузки только для внутреннего использования](#internallb), но использует переменные с окончанием -Int и реализует только порт приложения 80. Также удаляется `inboundNatPools`, чтобы оставить конечные точки RDP в общедоступной подсистеме балансировки нагрузки. Если требуется использовать RDP во внутренней подсистеме балансировки нагрузки, то переместите в нее `inboundNatPools` из внешней подсистемы балансировки нагрузки:

    ```json
            /* Add a second load balancer, configured with a static privateIPAddress and the "-Int" load balancer variables. */
            {
                "apiVersion": "[variables('lbApiVersion')]",
                "type": "Microsoft.Network/loadBalancers",
                /* Add "-Internal" to the name. */
                "name": "[concat('LB','-', parameters('clusterName'),'-',parameters('vmNodeType0Name'), '-Internal')]",
                "location": "[parameters('computeLocation')]",
                "dependsOn": [
                    /* Remove public IP dependsOn, add vnet dependsOn
                    "[concat('Microsoft.Network/publicIPAddresses/',concat(parameters('lbIPName'),'-','0'))]"
                    */
                    "[concat('Microsoft.Network/virtualNetworks/',parameters('virtualNetworkName'))]"
                ],
                "properties": {
                    "frontendIPConfigurations": [
                        {
                            "name": "LoadBalancerIPConfig",
                            "properties": {
                                /* Switch from Public to Private IP address
                                */
                                "publicIPAddress": {
                                    "id": "[resourceId('Microsoft.Network/publicIPAddresses',concat(parameters('lbIPName'),'-','0'))]"
                                }
                                */
                                "subnet" :{
                                    "id": "[variables('subnet0Ref')]"
                                },
                                "privateIPAddress": "[parameters('internalLBAddress')]",
                                "privateIPAllocationMethod": "Static"
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
                        /* Add the AppPort rule. Be sure to reference the "-Int" versions of backendAddressPool, frontendIPConfiguration, and the probe variables. */
                        {
                            "name": "AppPortLBRule1",
                            "properties": {
                                "backendAddressPool": {
                                    "id": "[variables('lbPoolID0-Int')]"
                                },
                                "backendPort": "[parameters('loadBalancedAppPort1')]",
                                "enableFloatingIP": "false",
                                "frontendIPConfiguration": {
                                    "id": "[variables('lbIPConfig0-Int')]"
                                },
                                "frontendPort": "[parameters('loadBalancedAppPort1')]",
                                "idleTimeoutInMinutes": "5",
                                "probe": {
                                    "id": "[concat(variables('lbID0-Int'),'/probes/AppPortProbe1')]"
                                },
                                "protocol": "tcp"
                            }
                        }
                    ],
                    "probes": [
                    /* Add the probe for the app port. */
                    {
                            "name": "AppPortProbe1",
                            "properties": {
                                "intervalInSeconds": 5,
                                "numberOfProbes": 2,
                                "port": "[parameters('loadBalancedAppPort1')]",
                                "protocol": "tcp"
                            }
                        }
                    ],
                    "inboundNatPools": [
                    ]
                },
                "tags": {
                    "resourceType": "Service Fabric",
                    "clusterName": "[parameters('clusterName')]"
                }
            },
    ```

6. В `networkProfile` ресурса `Microsoft.Compute/virtualMachineScaleSets` добавьте внутренний пул адресов:

    ```json
    "loadBalancerBackendAddressPools": [
                                                        {
                                                            "id": "[variables('lbPoolID0')]"
                                                        },
                                                        {
                                                            /* Add internal BE pool */
                                                            "id": "[variables('lbPoolID0-Int')]"
                                                        }
    ],
    ```

7. Разверните шаблон.

    ```powershell
    New-AzResourceGroup -Name sfnetworkinginternalexternallb -Location westus

    New-AzResourceGroupDeployment -Name deployment -ResourceGroupName sfnetworkinginternalexternallb -TemplateFile C:\SFSamples\Final\template\_internalexternalLB.json
    ```

После развертывания вы увидите две подсистемы балансировки нагрузки в группе ресурсов. Изучив их подробнее, вы увидите общедоступный IP-адрес и конечные точки управления (порты 19000 и 19080), назначенные общедоступному IP-адресу. А также можно увидеть статический внутренний IP-адрес и конечную точку приложения (порт 80), назначенные внутренней подсистеме балансировки нагрузки. При этом обе подсистемы балансировки нагрузки используют один внутренний пул масштабируемого набора виртуальных машин.

## <a name="notes-for-production-workloads"></a>Примечания для рабочих нагрузок в рабочей среде

Указанные выше шаблоны GitHub предназначены для работы с SKU по умолчанию для Azure Load Balancer (цен. категория "Стандартный") (SLB), базового SKU. Это SLB не имеет соглашения об уровне обслуживания, поэтому для производственных рабочих нагрузок следует использовать номер SKU "Стандартный". Дополнительные сведения об этом см. в [обзоре Load Balancer (цен. Категория "Стандартный") Azure](../load-balancer/load-balancer-overview.md). Любой кластер Service Fabric, использующий SKU уровня "Стандартный" для SLB, должен гарантировать, что каждый тип узла имеет правило, разрешающее исходящий трафик через порт 443. Это необходимо для завершения установки кластера, и любое развертывание без такого правила завершится ошибкой. В приведенном выше примере балансировщика нагрузки "только для внутреннего использования" в шаблон необходимо добавить дополнительный внешний балансировщик нагрузки с правилом, разрешающим исходящий трафик для порта 443.

## <a name="next-steps"></a>Дальнейшие действия
[Создание кластера](service-fabric-cluster-creation-via-arm.md)

После развертывания вы увидите две подсистемы балансировки нагрузки в группе ресурсов. Изучив их подробнее, вы увидите общедоступный IP-адрес и конечные точки управления (порты 19000 и 19080), назначенные общедоступному IP-адресу. А также можно увидеть статический внутренний IP-адрес и конечную точку приложения (порт 80), назначенные внутренней подсистеме балансировки нагрузки. При этом обе подсистемы балансировки нагрузки используют один внутренний пул масштабируемого набора виртуальных машин.
