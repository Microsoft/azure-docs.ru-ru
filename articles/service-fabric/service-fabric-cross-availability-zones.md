---
title: Развертывание кластера в Зоны доступности
description: Узнайте, как создать кластер Azure Service Fabric в Зоны доступности.
author: peterpogorski
ms.topic: conceptual
ms.date: 04/25/2019
ms.author: pepogors
ms.openlocfilehash: 95ee4e5f326dd9b76645d22ff735bc36437c72fb
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104870128"
---
# <a name="deploy-an-azure-service-fabric-cluster-across-availability-zones"></a>Развертывание кластера Azure Service Fabric в Зоны доступности
Зоны доступности в Azure — это высокодоступное предложение, защищающее приложения и данные от сбоев центров обработки данных. Зона доступности — это уникальное физическое расположение, оснащенное независимым питанием, охлаждением и сетью в регионе Azure.

Service Fabric поддерживает кластеры, охватывающие Зоны доступности, путем развертывания типов узлов, которые закреплены в конкретных зонах. Это обеспечит высокую доступность ваших приложений. Зоны доступности Azure доступны только в выбранных регионах. Дополнительные сведения см. в разделе [зоны доступности Azure обзор](../availability-zones/az-overview.md).

Примеры шаблонов доступны: [шаблон зоны перекрестной доступности Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates)

## <a name="recommended-topology-for-primary-node-type-of-azure-service-fabric-clusters-spanning-across-availability-zones"></a>Рекомендуемая топология для типа первичного узла Service Fabric кластеров Azure, охватывающих несколько Зоны доступности
Кластер Service Fabric, распределенный между Зоны доступности обеспечивает высокий уровень доступности состояния кластера. Чтобы охватить кластер Service Fabric в разных зонах, необходимо создать тип первичного узла в каждой зоне доступности, поддерживаемой регионом. Начальные узлы распределяются равномерно по каждому из типов первичного узла.

Для топологии, рекомендуемой для типа первичного узла, требуются следующие ресурсы:

* Уровень надежности кластера, установленный в категории Platinum.
* Три типа узлов, помеченные как первичные.
    * Каждый тип узла должен быть сопоставлен с собственным масштабируемым набором виртуальных машин, расположенным в разных зонах.
    * Каждый масштабируемый набор виртуальных машин должен иметь по крайней мере пять узлов (серебристая устойчивость).
* Один ресурс общедоступного IP-адреса, использующий SKU "Стандартный".
* Один Load Balancer ресурс, использующий номер SKU "Стандартный".
* NSG, на который ссылается подсеть, в которой развертываются масштабируемые наборы виртуальных машин.

>[!NOTE]
> Свойство одной группы размещения масштабируемого набора виртуальных машин должно иметь значение true, так как Service Fabric не поддерживает один масштабируемый набор виртуальных машин, охватывающий зоны.

На этой схеме показана схема архитектуры зоны доступности Azure Service Fabric ![ , в которой показана архитектура зоны доступности azure Service Fabric.][sf-architecture]

Пример списка узлов, в котором показаны форматы "Демон/обновления" в масштабируемом наборе виртуальных машин, охватывающем зоны

 ![Пример списка узлов, в котором показаны форматы "Демон/обновления" в масштабируемом наборе виртуальных машин, охватывающем зоны.][sf-multi-az-nodes]

**Распределение реплик служб между зонами**. при развертывании службы на NodeType, охватывающих зоны, реплики размещаются, чтобы обеспечить их размещение в отдельных зонах. Это гарантируется, так как домен сбоя на узлах, имеющихся в каждом из этих nodeType, настроен с использованием сведений о зоне (например, Демон демона = Демон:/zone1/1 и т. д.). Например, для 5 реплик или экземпляров службы распределение будет 2-2-1, а среда выполнения попытается обеспечить равномерное распределение по AZs.

**Конфигурация реплики службы пользователя**: пользовательские службы с отслеживанием состояния, развернутые в зоне перекрестной доступности, должны быть настроены с помощью этой конфигурации: число реплик с целевым значением = 9, min = 5. Эта конфигурация поможет службе работать даже в том случае, если одна зона выходит из строя, так как 6 реплики будут по-прежнему находиться в двух других зонах. Обновление приложения в таком сценарии также будет проходить через.

**Cluster ReliabilityLevel**: определяет количество начальных узлов в кластере, а также размер реплики системных служб. Так как Настройка зоны перекрестной доступности имеет большее количество узлов, распределенных между зонами с целью обеспечения устойчивости зоны, более высокий уровень надежности обеспечит наличие узлов с дополнительными начальными значениями и реплики системных служб, которые равномерно распределяются между зонами, поэтому в случае сбоя зоны кластер и системные службы остаются нетронутыми. "ReliabilityLevel = Platinum" обеспечит 9 начальных узлов, распределенных между зонами в кластере с 3 значениями заполнения в каждой зоне, поэтому рекомендуется для настройки зоны перекрестной доступности.

**Сценарий отключения зоны**. при отключении зоны все узлы в этой зоне будут отображаться в неработоспособном режиме. Реплики службы на этих узлах также будут отключены. Так как в других зонах есть реплики, служба продолжает отвечать с первичными репликами, переключаясь на зоны, которые работают. Службы будут отображаться в состоянии предупреждения, так как количество целевых реплик еще не достигнуто и так как число виртуальных машин по-прежнему превышает минимальный размер целевой реплики. Затем Service Fabric балансировщик нагрузки будет использовать реплики в рабочих зонах в соответствии с настроенным числом целевых реплик. На этом этапе службы будут отображаться как работоспособные. Когда зона, которая была отключена, поступает в резервную копию, все реплики службы будут равномерно распределены по всем зонам.

## <a name="networking-requirements"></a>Требования к сети
### <a name="public-ip-and-load-balancer-resource"></a>Общедоступный IP-адрес и Load Balancer ресурс
Чтобы включить свойство "зоны" в ресурсе масштабируемого набора виртуальных машин, подсистема балансировки нагрузки и ресурс IP, на который ссылается этот масштабируемый набор виртуальных машин, должны использовать номер SKU " *стандартный* ". При создании подсистемы балансировки нагрузки или ресурса IP-адреса без свойства SKU будет создан базовый номер SKU, который не поддерживает Зоны доступности. Балансировщик нагрузки уровня "Стандартный" будет блокировать весь трафик извне по умолчанию; чтобы разрешить внешний трафик, необходимо развернуть NSG в подсети.

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
```

```json
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


### <a name="enabling-zones-on-a-virtual-machine-scale-set"></a>Включение зон в масштабируемом наборе виртуальных машин
Чтобы включить зону, в масштабируемом наборе виртуальных машин необходимо включить следующие три значения в ресурс масштабируемого набора виртуальных машин.

* Первое значение — это свойство **Zones** , которое указывает, в какой зоне доступности будет развернут масштабируемый набор виртуальных машин.
* Вторым значением является свойство "singlePlacementGroup", которое должно иметь значение true.
* Третье значение — это свойство "Фаултдомаиноверриде" в Service Fabric расширении масштабируемого набора виртуальных машин. Значение этого свойства должно включать только зону, в которой будет размещен этот масштабируемый набор виртуальных машин. Пример: "Фаултдомаиноверриде": "AZ1". все ресурсы масштабируемого набора виртуальных машин должны быть помещены в один регион, так как кластеры Azure Service Fabric не поддерживают кросс-регион.

```json
{
    "apiVersion": "2018-10-01",
    "type": "Microsoft.Compute/virtualMachineScaleSets",
    "name": "[parameters('vmNodeType1Name')]",
    "location": "[parameters('computeLocation')]",
    "zones": ["1"],
    "properties": {
        "singlePlacementGroup": "true",
    },
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
            "durabilityLevel": "Silver",
            "certificate": {
                "thumbprint": "[parameters('certificateThumbprint')]",
                "x509StoreName": "[parameters('certificateStoreValue')]"
            },
            "systemLogUploadSettings": {
                "Enabled": true
            },
            "faultDomainOverride": "az1"
        },
        "typeHandlerVersion": "1.0"
    }
}
```

### <a name="enabling-multiple-primary-node-types-in-the-service-fabric-cluster-resource"></a>Включение нескольких типов первичных узлов в ресурс кластера Service Fabric
Чтобы задать один или несколько типов узлов в качестве первичных в ресурсе кластера, задайте для свойства "IsTrue" значение "true". При развертывании кластера Service Fabric в разных Зоны доступности необходимо иметь три типа узлов в разных зонах.

```json
{
    "reliabilityLevel": "Platinum",
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
        "vmInstanceCount": "[parameters('nt0InstanceCount')]"
    },
    {
        "name": "[parameters('vmNodeType1Name')]",
        "applicationPorts": {
            "endPort": "[parameters('nt1applicationEndPort')]",
            "startPort": "[parameters('nt1applicationStartPort')]"
        },
        "clientConnectionEndpointPort": "[parameters('nt1fabricTcpGatewayPort')]",
        "durabilityLevel": "Silver",
        "ephemeralPorts": {
            "endPort": "[parameters('nt1ephemeralEndPort')]",
            "startPort": "[parameters('nt1ephemeralStartPort')]"
        },
        "httpGatewayEndpointPort": "[parameters('nt1fabricHttpGatewayPort')]",
        "isPrimary": true,
        "vmInstanceCount": "[parameters('nt1InstanceCount')]"
    },
    {
        "name": "[parameters('vmNodeType2Name')]",
        "applicationPorts": {
            "endPort": "[parameters('nt2applicationEndPort')]",
            "startPort": "[parameters('nt2applicationStartPort')]"
        },
        "clientConnectionEndpointPort": "[parameters('nt2fabricTcpGatewayPort')]",
        "durabilityLevel": "Silver",
        "ephemeralPorts": {
            "endPort": "[parameters('nt2ephemeralEndPort')]",
            "startPort": "[parameters('nt2ephemeralStartPort')]"
        },
        "httpGatewayEndpointPort": "[parameters('nt2fabricHttpGatewayPort')]",
        "isPrimary": true,
        "vmInstanceCount": "[parameters('nt2InstanceCount')]"
    }
    ],
}
```

## <a name="migrate-to-using-availability-zones-from-a-cluster-using-a-basic-sku-load-balancer-and-a-basic-sku-ip"></a>Переход на использование Зоны доступности из кластера с использованием базового SKU Load Balancer и IP-адреса базового SKU
Чтобы выполнить миграцию кластера, который использовал Load Balancer и IP-адрес с номером SKU "базовый", сначала необходимо создать совершенно новый Load Balancer и IP-ресурс, используя номер SKU "Стандартный". Невозможно обновить эти ресурсы на месте.

На новые типы узлов зоны перекрестной доступности, которые вы хотите использовать, должны быть ссылки на новую ФУНТОВу и IP-адрес. В приведенном выше примере в зонах 1, 2 и 3 были добавлены три новых ресурса масштабируемого набора виртуальных машин. Эти масштабируемые наборы виртуальных машин ссылаются на только что созданную балансировку нагрузки и IP-адрес и помечаются как типы первичных узлов в ресурсе кластера Service Fabric.

Для начала необходимо добавить новые ресурсы в существующий шаблон диспетчер ресурсов. К этим ресурсам относятся:
* Ресурс общедоступного IP-адреса, использующий SKU "Стандартный".
* Load Balancer ресурс с использованием номера SKU "Стандартный".
* NSG, на который ссылается подсеть, в которой развертываются масштабируемые наборы виртуальных машин.
* Три типа узлов, помеченные как первичные.
    * Каждый тип узла должен быть сопоставлен с собственным масштабируемым набором виртуальных машин, расположенным в разных зонах.
    * Каждый масштабируемый набор виртуальных машин должен иметь по крайней мере пять узлов (серебристая устойчивость).

Пример этих ресурсов можно найти в [образце шаблона](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/10-VM-Ubuntu-2-NodeType-Secure).

```powershell
New-AzureRmResourceGroupDeployment `
    -ResourceGroupName $ResourceGroupName `
    -TemplateFile $Template `
    -TemplateParameterFile $Parameters
```

После завершения развертывания ресурсов можно приступить к отключению узлов в типе первичного узла из исходного кластера. После отключения узлов системные службы будут перенесены на новый тип первичного узла, который был развернут на предыдущем шаге.

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint $ClusterName `
    -KeepAliveIntervalInSec 10 `
    -X509Credential `
    -ServerCertThumbprint $thumb  `
    -FindType FindByThumbprint `
    -FindValue $thumb `
    -StoreLocation CurrentUser `
    -StoreName My 

Write-Host "Connected to cluster"

$nodeNames = @("_nt0_0", "_nt0_1", "_nt0_2", "_nt0_3", "_nt0_4")

Write-Host "Disabling nodes..."
foreach($name in $nodeNames) {
    Disable-ServiceFabricNode -NodeName $name -Intent RemoveNode -Force
}
```

После отключения всех узлов системные службы будут работать на первичном узле типа, который распределяется между зонами. После этого Отключенные узлы можно удалить из кластера. После удаления узлов можно удалить исходные IP-адреса, Load Balancer и масштабируемые наборы виртуальных машин.

```powershell
foreach($name in $nodeNames){
    # Remove the node from the cluster
    Remove-ServiceFabricNodeState -NodeName $name -TimeoutSec 300 -Force
    Write-Host "Removed node state for node $name"
}

$scaleSetName="nt0"
Remove-AzureRmVmss -ResourceGroupName $groupname -VMScaleSetName $scaleSetName -Force

$lbname="LB-cluster-nt0"
$oldPublicIpName="LBIP-cluster-0"
$newPublicIpName="LBIP-cluster-1"

Remove-AzureRmLoadBalancer -Name $lbname -ResourceGroupName $groupname -Force
Remove-AzureRmPublicIpAddress -Name $oldPublicIpName -ResourceGroupName $groupname -Force
```

Затем следует удалить ссылки на эти ресурсы из развернутого шаблона диспетчер ресурсов.

На последнем шаге будет выполняться обновление DNS-имени и общедоступного IP-адреса.

```powershell
$oldprimaryPublicIP = Get-AzureRmPublicIpAddress -Name $oldPublicIpName  -ResourceGroupName $groupname
$primaryDNSName = $oldprimaryPublicIP.DnsSettings.DomainNameLabel
$primaryDNSFqdn = $oldprimaryPublicIP.DnsSettings.Fqdn

Remove-AzureRmLoadBalancer -Name $lbname -ResourceGroupName $groupname -Force
Remove-AzureRmPublicIpAddress -Name $oldPublicIpName -ResourceGroupName $groupname -Force

$PublicIP = Get-AzureRmPublicIpAddress -Name $newPublicIpName  -ResourceGroupName $groupname
$PublicIP.DnsSettings.DomainNameLabel = $primaryDNSName
$PublicIP.DnsSettings.Fqdn = $primaryDNSFqdn
Set-AzureRmPublicIpAddress -PublicIpAddress $PublicIP

```

## <a name="preview-enable-multiple-availability-zones-in-single-virtual-machine-scale-set"></a>Образца Включение нескольких зон доступности в одном масштабируемом наборе виртуальных машин

В упомянутом выше решении используется один nodeType для AZ. Следующее решение позволит пользователям развернуть 3 AZ в одном и том же nodeType.

[Здесь](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/15-VM-Windows-Multiple-AZ-Secure)приведен полный пример шаблона.

![Архитектура зоны доступности Azure Service Fabric][sf-multi-az-arch]

### <a name="configuring-zones-on-a-virtual-machine-scale-set"></a>Настройка зон в масштабируемом наборе виртуальных машин
Чтобы включить зоны в масштабируемом наборе виртуальных машин, необходимо включить следующие три значения в ресурс масштабируемого набора виртуальных машин.

* Первое значение — это свойство **Zones** , которое указывает зоны доступности, присутствующую в масштабируемом наборе виртуальных машин.
* Вторым значением является свойство "singlePlacementGroup", которое должно иметь значение true. **Масштабируемый набор, развернутый в 3 AZ, может масштабировать до 300 виртуальных машин, даже если "singlePlacementGroup = true".**
* Третье значение — "zoneBalance", что обеспечивает гарантированную балансировку зоны. Значение должно быть "true". Это гарантирует, что распределение виртуальных машин между зонами не будет несбалансированным, гарантируя, что при отключении одной из зон другие две зоны будут иметь достаточные виртуальные машины, чтобы обеспечить бесперебойную работу кластера. Кластер с несбалансированным распределением виртуальных машин может не содержаться в сценарии отключения зоны, так как в этой зоне может быть большая часть виртуальных машин. Несбалансированное распределение виртуальных машин между зонами также приведет к возникновению проблем, связанных с размещением служб & обновлениями инфраструктуры. Прочитайте о [зонебаланЦинг](../virtual-machine-scale-sets/virtual-machine-scale-sets-use-availability-zones.md#zone-balancing).
* Переопределения FaultDomain и UpgradeDomain не обязательно должны быть настроены.

```json
{
    "apiVersion": "2018-10-01",
    "type": "Microsoft.Compute/virtualMachineScaleSets",
    "name": "[parameters('vmNodeType1Name')]",
    "location": "[parameters('computeLocation')]",
    "zones": ["1", "2", "3"],
    "properties": {
        "singlePlacementGroup": "true",
        "zoneBalance": true
    }
}
```

>[!NOTE]
> * **Кластеры Service Fabric должны иметь по крайней мере один первичный nodeType. Дурабилитилевел основного класса nodeType должен быть серебро или выше.**
> * С помощью команды AZ, охватывающей масштабируемый набор виртуальных машин, следует настроить по крайней мере 3 зоны доступности независимо от Дурабилитилевел.
> * AZ. объединение масштабируемого набора виртуальных машин с серебряной устойчивостью (или выше) должно включать не менее 15 виртуальных машин.
> * AZ охват масштабируемого набора виртуальных машин с помощью бронзовой устойчивости должен содержать не менее 6 виртуальных машин.

### <a name="enabling-the-support-for-multiple-zones-in-the-service-fabric-nodetype"></a>Включение поддержки нескольких зон в Service Fabric nodeType
Для поддержки нескольких зон доступности необходимо включить Service Fabric nodeType.

* Первое значение — **мултиплеаваилабилитизонес** , для которого необходимо задать значение true для NodeType.
* Второе значение — **сфзоналупградемоде** и является необязательным. Это свойство нельзя изменить, если NodeType с несколькими AZ уже существует в кластере.
  Свойство управляет логической группировкой виртуальных машин в доменах обновления.
  **Если значение равно "Parallel":** Виртуальные машины в NodeType будут сгруппированы в доменов обновления, игнорируя сведения о зоне в 5 доменов обновления. Это приведет к тому, что ДО0 будут обновлены во всех зонах. Этот режим развертывания ускоряется для обновления, но не рекомендуется, так как он относится к рекомендациям SDP, что позволяет применять эти обновления только к одной зоне за раз.
  **Если значение опущено или равно "иерархическое":** Виртуальные машины будут сгруппированы в соответствии с зональные распределением до 15 доменов обновления. Каждая из трех зон будет иметь 5 доменов обновления. Это гарантирует, что обновления попадают в зону Zone, переходя к следующей зоне только после завершения 5 доменов обновления в первой зоне, медленно в 15 доменов обновления (3 зоны, 5 доменов обновления), что более безопасно с точки зрения кластера и пользовательского приложения.
  Это свойство определяет только способ обновления приложения ServiceFabric и обновления кода. Базовые обновления масштабируемого набора виртуальных машин по-прежнему будут выполняться параллельно во всех AZ.
  Это свойство не влияет на распределение обновления для типов узлов, для которых не включено несколько зон.
* Третье значение — **вмссзоналупградемоде = Parallel**. Это *Обязательное* свойство, которое необходимо настроить в кластере, если добавляется NodeType с несколькими AZs. Это свойство определяет режим обновления для обновлений масштабируемого набора виртуальных машин, которые будут выполняться параллельно во всех AZ.
  Сейчас это свойство можно установить только для параллельного выполнения.
* Service Fabric кластерный ресурс apiVersion должен иметь значение "2020-12-01-Preview" или более поздней версии.
* Версия кода кластера должна быть "7.2.445" или выше.

```json
{
    "apiVersion": "2020-12-01-preview",
    "type": "Microsoft.ServiceFabric/clusters",
    "name": "[parameters('clusterName')]",
    "location": "[parameters('clusterLocation')]",
    "dependsOn": [
        "[concat('Microsoft.Storage/storageAccounts/', parameters('supportLogStorageAccountName'))]"
    ],
    "properties": {
        "reliabilityLevel": "Platinum",
        "SFZonalUpgradeMode": "Hierarchical",
        "VMSSZonalUpgradeMode": "Parallel",
        "nodeTypes": [
          {
                "name": "[parameters('vmNodeType0Name')]",
                "multipleAvailabilityZones": true,
          }
        ]
}
```

>[!NOTE]
> * Общедоступные IP-адреса и ресурсы Load Balancer должны использовать стандартный SKU, как описано выше в этой статье.
> * свойство "Мултиплеаваилабилитизонес" в nodeType может быть определено только во время создания nodeType и не может быть изменено позже. Таким образом, существующие nodeType нельзя настроить с помощью этого свойства.
> * Если параметр "Сфзоналупградемоде" опущен или имеет значение "иерархическое", развертывание кластера и приложений будет выполняться медленнее, так как в кластере больше доменов обновления. Важно правильно настроить время ожидания политики обновления для включения времени обновления в 15 доменов обновления. Необходимо обновить политику обновления для приложения и кластера, чтобы убедиться, что развертывание не превысит время ожидания развертывания Сербице ресурсов Azure 12hours. Это означает, что развертывание не должно потребовать больше 12hours для 15UDs, т. е. не следует использовать более 40 мин/обновления.
> * Установите кластер **reliabilityLevel = Platinum** , чтобы убедиться в том, что кластер продерживает ситуацию с одной зоной.

>[!NOTE]
> Для получения оптимальной методики рекомендуется Сфзоналупградемоде задать иерархическое значение или опустить. Развертывание будет соответствовать зональные распределению виртуальных машин, влияющих на меньший объем реплик и (или) экземпляров, что делает их более безопасными.
> Используйте параметр Сфзоналупградемоде в значение Parallel, если скорость развертывания является приоритетной, или в типе узла с несколькими AZ выполняется только Рабочая нагрузка без отслеживания состояния. Это приведет к тому, что обновления будет выполняться параллельно во всех AZ.

### <a name="migration-to-the-node-type-with-multiple-availability-zones"></a>Миграция на тип узла с несколькими Зоны доступности
Для всех сценариев миграции необходимо добавить новый nodeType, который будет иметь несколько поддерживаемых зон доступности. Невозможно перенести существующий nodeType для поддержки нескольких зон.
В этой [статье содержатся](./service-fabric-scale-up-primary-node-type.md) подробные инструкции по добавлению нового узла NodeType, а также добавлению других ресурсов, необходимых для нового типа узла, такого как ресурсы IP и балансировки нагрузки. В этой же статье также описывается снятие существующего узла nodeType после добавления в кластер типа nodeType с несколькими зонами доступности.

* Миграция с узла nodeType, который использует основные ресурсы балансировки нагрузки и IP-адресов: этот процесс уже описан [здесь](#migrate-to-using-availability-zones-from-a-cluster-using-a-basic-sku-load-balancer-and-a-basic-sku-ip) для решения с одним типом узла на AZ. 
    Единственное отличие от нового типа узла состоит в том, что существует только 1 масштабируемый набор виртуальных машин и 1 NodeType для всех AZ, а не 1 каждого в AZ.
* Миграция с узла nodeType, который использует стандартные ресурсы SKU и IP с NSG: выполните ту же процедуру, которая описана выше, за исключением того, что нет необходимости добавлять новые ресурсы фунтов, IP и NSG, и те же ресурсы можно повторно использовать в новом nodeType.


[sf-architecture]: ./media/service-fabric-cross-availability-zones/sf-cross-az-topology.png
[sf-multi-az-arch]: ./media/service-fabric-cross-availability-zones/sf-multi-az-topology.png
[sf-multi-az-nodes]: ./media/service-fabric-cross-availability-zones/sf-multi-az-nodes.png