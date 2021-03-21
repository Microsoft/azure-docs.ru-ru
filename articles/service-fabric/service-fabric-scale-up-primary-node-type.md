---
title: Масштабирование типа первичного узла Azure Service Fabric
description: Вертикальное масштабирование кластера Service Fabric путем добавления нового типа узла и удаления предыдущего.
ms.date: 12/11/2020
ms.author: pepogors
ms.topic: how-to
ms.openlocfilehash: 325ece761481077171a670c52e9d98071237601a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98251187"
---
# <a name="scale-up-a-service-fabric-cluster-primary-node-type"></a>Масштабирование типа первичного узла кластера Service Fabric

В этой статье описывается, как масштабировать тип первичного узла кластера Service Fabric с минимальным временем простоя. Общая стратегия обновления типа узла кластера Service Fabric:

1. Добавьте новый тип узла в кластер Service Fabric, поддерживаемый обновленным (или измененным) SKU и конфигурацией масштабируемого набора виртуальных машин. Этот шаг также включает настройку новой подсистемы балансировки нагрузки, подсети и общедоступного IP-адреса для масштабируемого набора.

1. После того как исходные и обновленные масштабируемые наборы работают параллельно, отключите экземпляры исходного узла по одному за раз, чтобы системные службы (или реплики служб с отслеживанием состояния) переводились в новый масштабируемый набор.

1. Убедитесь, что кластер и новые узлы работоспособны, а затем удалите исходный масштабируемый набор (и связанные ресурсы) и состояние узла для удаленных узлов.

Ниже описан процесс обновления размера виртуальной машины и операционной системы для виртуальных машин типа "основной узел" в примере кластера со значением " [Серебряная устойчивость](service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster)", поддерживаемой одним масштабируемым набором с пятью узлами. Мы будем обновлять тип первичного узла:

- От размера виртуальной машины *Standard_D2_V2* до уровня " *Стандартный" D4_V2* и
- Из операционной системы виртуальной машины *Windows server 2016 Datacenter with Containers* to *Windows Server 2019 Datacenter with Containers*.

> [!WARNING]
> Прежде чем выполнять эту процедуру для кластера рабочей среды, рекомендуется изучить образцы шаблонов и применить шаги для тестового кластера. Кластер также может быть недоступен в течение короткого периода времени.
>
> Не пытайтесь выполнить процедуру масштабирования типа первичного узла, если кластер находится в неработоспособном состоянии, так как это приведет к дальнейшему дестабилизации работы кластера.

Ниже приведены пошаговые шаблоны развертывания Azure, которые будут использоваться для выполнения этого примера сценария обновления. https://github.com/microsoft/service-fabric-scripts-and-templates/tree/master/templates/nodetype-upgrade

## <a name="set-up-the-test-cluster"></a>Настройка тестового кластера

Давайте создадим начальный тестовый кластер Service Fabric. Сначала [Скачайте](https://github.com/microsoft/service-fabric-scripts-and-templates/tree/master/templates/nodetype-upgrade) примеры шаблонов Azure Resource Manager, которые будут использоваться для выполнения этого сценария.

Затем войдите в учетную запись Azure.

```powershell
# Sign in to your Azure account
Login-AzAccount -SubscriptionId "<subscription ID>"
```

Затем откройте [*parameters.jsв*](https://github.com/microsoft/service-fabric-scripts-and-templates/tree/master/templates/nodetype-upgrade/parameters.json) файле и обновите значение на `clusterName` что-то уникальное (в Azure).

Следующие команды помогут создать новый самозаверяющий сертификат и развернуть тестовый кластер. Если у вас уже есть сертификат, который вы хотите использовать, перейдите к [использованию существующего сертификата для развертывания кластера](#use-an-existing-certificate-to-deploy-the-cluster).

### <a name="generate-a-self-signed-certificate-and-deploy-the-cluster"></a>Создание самозаверяющего сертификата и развертывание кластера

Сначала назначьте переменные, которые потребуются для развертывания кластера Service Fabric. Измените значения `resourceGroupName`, `certSubjectName`, `parameterFilePath` и `templateFilePath` в соответствии со своей учетной записью и средой.

```powershell
# Assign deployment variables
$resourceGroupName = "sftestupgradegroup"
$certOutputFolder = "c:\certificates"
$certPassword = "Password!1" | ConvertTo-SecureString -AsPlainText -Force
$certSubjectName = "sftestupgrade.southcentralus.cloudapp.azure.com"
$parameterFilePath = "C:\parameters.json"
$templateFilePath = "C:\Initial-TestClusterSetup.json"
```

> [!NOTE]
> `certOutputFolder`Перед выполнением команды для развертывания нового кластера Service Fabric убедитесь, что расположение существует на локальном компьютере.

Затем разверните тестовый кластер Service Fabric.

```powershell
# Deploy the initial test cluster
New-AzServiceFabricCluster `
    -ResourceGroupName $resourceGroupName `
    -CertificateOutputFolder $certOutputFolder `
    -CertificatePassword $certPassword `
    -CertificateSubjectName $certSubjectName `
    -TemplateFile $templateFilePath `
    -ParameterFile $parameterFilePath
```

После завершения развертывания найдите *PFX*-файл (`$certPfx`) на локальном компьютере и импортируйте его в хранилище сертификатов.

```powershell
cd c:\certificates
$certPfx = ".\sftestupgradegroup20200312121003.pfx"

Import-PfxCertificate `
     -FilePath $certPfx `
     -CertStoreLocation Cert:\CurrentUser\My `
     -Password (ConvertTo-SecureString Password!1 -AsPlainText -Force)
```

Операция вернет отпечаток сертификата, который теперь можно использовать для [подключения к новому кластеру](#connect-to-the-new-cluster-and-check-health-status) и проверки его состояния работоспособности. (Пропустите следующий раздел, который является альтернативным подходом к развертыванию кластера.)

### <a name="use-an-existing-certificate-to-deploy-the-cluster"></a>Использование существующего сертификата для развертывания кластера

Кроме того, для развертывания тестового кластера можно использовать существующий сертификат Azure Key Vault. Для этого необходимо [получить ссылки на Key Vault](#obtain-your-key-vault-references) и отпечаток сертификата.

```powershell
# Key Vault variables
$certUrlValue = "https://sftestupgradegroup.vault.azure.net/secrets/sftestupgradegroup20200309235308/dac0e7b7f9d4414984ccaa72bfb2ea39"
$sourceVaultValue = "/subscriptions/########-####-####-####-############/resourceGroups/sftestupgradegroup/providers/Microsoft.KeyVault/vaults/sftestupgradegroup"
$thumb = "BB796AA33BD9767E7DA27FE5182CF8FDEE714A70"
```

Затем назначьте имя группы ресурсов для кластера и укажите `templateFilePath` `parameterFilePath` расположения и.

> [!NOTE]
> Указанная группа ресурсов должна уже существовать и находиться в том же регионе, что и Key Vault.

```powershell
$resourceGroupName = "sftestupgradegroup"
$templateFilePath = "C:\Initial-TestClusterSetup.json"
$parameterFilePath = "C:\parameters.json"
```

Наконец, выполните следующую команду, чтобы развернуть исходный тестовый кластер:

```powershell
# Deploy the initial test cluster
New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $templateFilePath `
    -TemplateParameterFile $parameterFilePath `
    -CertificateThumbprint $thumb `
    -CertificateUrlValue $certUrlValue `
    -SourceVaultValue $sourceVaultValue `
    -Verbose
```

### <a name="connect-to-the-new-cluster-and-check-health-status"></a>Подключение к новому кластеру и проверка состояния работоспособности

Подключитесь к кластеру и убедитесь, что все пять его узлов работоспособны (замените `clusterName` `thumb` переменные и собственными значениями):

```powershell
# Connect to the cluster
$clusterName = "sftestupgrade.southcentralus.cloudapp.azure.com:19000"
$thumb = "BB796AA33BD9767E7DA27FE5182CF8FDEE714A70"

Connect-ServiceFabricCluster `
    -ConnectionEndpoint $clusterName `
    -KeepAliveIntervalInSec 10 `
    -X509Credential `
    -ServerCertThumbprint $thumb  `
    -FindType FindByThumbprint `
    -FindValue $thumb `
    -StoreLocation CurrentUser `
    -StoreName My

# Check cluster health
Get-ServiceFabricClusterHealth
```

После этого мы готовы начать процедуру обновления.

## <a name="deploy-a-new-primary-node-type-with-upgraded-scale-set"></a>Развертывание нового типа первичного узла с обновленным масштабируемым набором

Чтобы обновить (вертикально масштабировать) тип узла, сначала необходимо развернуть новый тип узла, поддерживаемый новым масштабируемым набором и вспомогательными ресурсами. Новый масштабируемый набор будет помечен как основной ( `isPrimary: true` ), как и исходный масштабируемый набор (если не выполняется обновление типа первичного узла). Ресурсы, созданные в следующем разделе, в конечном итоге станут новым типом первичного узла в кластере, а исходные ресурсы типа первичного узла будут удалены.

### <a name="update-the-cluster-template-with-the-upgraded-scale-set"></a>Обновление шаблона кластера с помощью обновленного масштабируемого набора

Ниже приведены изменения в разделе шаблона развертывания исходного кластера для добавления нового типа первичного узла и вспомогательных ресурсов.

Необходимые изменения для этого шага уже внесены в [*Step1-AddPrimaryNodeType.js*](https://github.com/microsoft/service-fabric-scripts-and-templates/tree/master/templates/nodetype-upgrade/Step1-AddPrimaryNodeType.json) в файле шаблона, и далее эти изменения будут объяснены подробно. При желании вы можете пропустить объяснение и продолжить [получать ссылки на Key Vault](#obtain-your-key-vault-references) и [развернуть обновленный шаблон](#deploy-the-updated-template) , который добавляет новый тип первичного узла в кластер.

> [!Note]
> Убедитесь, что используются имена, уникальные для исходного типа узла, масштабируемого набора, подсистемы балансировки нагрузки, общедоступного IP-адреса и подсети исходного типа первичного узла, так как эти ресурсы будут удалены на более позднем этапе процесса.

#### <a name="create-a-new-subnet-in-the-existing-virtual-network"></a>Создание новой подсети в существующей виртуальной сети

```json
{
    "name": "[variables('subnet1Name')]",
    "properties": {
        "addressPrefix": "[variables('subnet1Prefix')]"
    }
}
```

#### <a name="create-a-new-public-ip-with-a-unique-domainnamelabel"></a>Создание нового общедоступного IP-адреса с уникальным domainNameLabel

```json
{
    "apiVersion": "[variables('publicIPApiVersion')]",
    "type": "Microsoft.Network/publicIPAddresses",
    "name": "[concat(variables('lbIPName'),'-',variables('vmNodeType1Name'))]",
    "location": "[variables('computeLocation')]",
    "properties": {
    "dnsSettings": {
        "domainNameLabel": "[concat(variables('dnsName'),'-','nt1')]"
    },
    "publicIPAllocationMethod": "Dynamic"
    },
    "tags": {
    "resourceType": "Service Fabric",
    "clusterName": "[parameters('clusterName')]"
    }
}
```

#### <a name="create-a-new-load-balancer-for-the-public-ip"></a>Создание новой подсистемы балансировки нагрузки для общедоступного IP-адреса

```json
"dependsOn": [
    "[concat('Microsoft.Network/publicIPAddresses/',concat(variables('lbIPName'),'-',variables('vmNodeType1Name')))]"
]
```

#### <a name="create-a-new-virtual-machine-scale-set-with-upgraded-vm-and-os-skus"></a>Создание нового масштабируемого набора виртуальных машин (с обновленными виртуальными машинами и номерами SKU ОС)

Тип узла ref

```json
"nodeTypeRef": "[variables('vmNodeType1Name')]"
```

SKU виртуальной машины

```json
"sku": {
    "name": "[parameters('vmNodeType1Size')]",
    "capacity": "[parameters('nt1InstanceCount')]",
    "tier": "Standard"
}
```

SKU ОС

```json
"imageReference": {
    "publisher": "[parameters('vmImagePublisher1')]",
    "offer": "[parameters('vmImageOffer1')]",
    "sku": "[parameters('vmImageSku1')]",
    "version": "[parameters('vmImageVersion1')]"
}
```

Кроме того, убедитесь, что включены все дополнительные расширения, необходимые для вашей рабочей нагрузки.

#### <a name="add-a-new-primary-node-type-to-the-cluster"></a>Добавление нового типа первичного узла в кластер

Теперь, когда новый тип узла (vmNodeType1Name) имеет собственное имя, подсеть, IP-адрес, подсистему балансировки нагрузки и масштабируемый набор, он может повторно использовать все остальные переменные из исходного типа узла (например `nt0applicationEndPort` ,, `nt0applicationStartPort` и `nt0fabricTcpGatewayPort` ):

```json
"name": "[variables('vmNodeType1Name')]",
"applicationPorts": {
    "endPort": "[variables('nt0applicationEndPort')]",
    "startPort": "[variables('nt0applicationStartPort')]"
},
"clientConnectionEndpointPort": "[variables('nt0fabricTcpGatewayPort')]",
"durabilityLevel": "Bronze",
"ephemeralPorts": {
    "endPort": "[variables('nt0ephemeralEndPort')]",
    "startPort": "[variables('nt0ephemeralStartPort')]"
},
"httpGatewayEndpointPort": "[variables('nt0fabricHttpGatewayPort')]",
"isPrimary": true,
"reverseProxyEndpointPort": "[variables('nt0reverseProxyEndpointPort')]",
"vmInstanceCount": "[parameters('nt1InstanceCount')]"
```

После реализации всех изменений в файлах шаблонов и параметров перейдите к следующему разделу, чтобы получить ссылки на Key Vault и развернуть обновления в кластере.

### <a name="obtain-your-key-vault-references"></a>Получение ссылок на Key Vault

Для развертывания обновленной конфигурации потребуется несколько ссылок на сертификат кластера, хранящийся в Key Vault. Самый простой способ найти эти значения — с помощью портал Azure. Что вам понадобится

* **URL-адрес Key Vault сертификата кластера.** Из Key Vault в портал Azure выберите Сертификаты для   >    >  **идентификатора секрета** нужного сертификата:

    ```powershell
    $certUrlValue="https://sftestupgradegroup.vault.azure.net/secrets/sftestupgradegroup20200309235308/dac0e7b7f9d4414984ccaa72bfb2ea39"
    ```

* **Отпечаток сертификата кластера.** (Возможно, у вас уже есть [Подключение к первоначальному кластеру](#connect-to-the-new-cluster-and-check-health-status) , чтобы проверить его состояние работоспособности.) В той же колонке сертификата (**Сертификаты**, для которой  >  *нужен нужный сертификат*) в портал Azure скопируйте **отпечаток X. 509 SHA-1 (в шестнадцатеричном формате)**:

    ```powershell
    $thumb = "BB796AA33BD9767E7DA27FE5182CF8FDEE714A70"
    ```

* **Идентификатор ресурса Key Vault.** В Key Vault в портал Azure выберите **Свойства**  >  **идентификатор ресурса**:

    ```powershell
    $sourceVaultValue = "/subscriptions/########-####-####-####-############/resourceGroups/sftestupgradegroup/providers/Microsoft.KeyVault/vaults/sftestupgradegroup"
    ```

### <a name="deploy-the-updated-template"></a>Развертывание обновленного шаблона

Внесите `templateFilePath` необходимые изменения и выполните следующую команду:

```powershell
# Deploy the new node type and its resources
$templateFilePath = "C:\Step1-AddPrimaryNodeType.json"

New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $templateFilePath `
    -TemplateParameterFile $parameterFilePath `
    -CertificateThumbprint $thumb `
    -CertificateUrlValue $certUrlValue `
    -SourceVaultValue $sourceVaultValue `
    -Verbose
```

После завершения развертывания снова проверьте работоспособность кластера и убедитесь, что все узлы на обоих типах узлов работоспособны.

```powershell
Get-ServiceFabricClusterHealth
```

## <a name="migrate-seed-nodes-to-the-new-node-type"></a>Перенос начальных узлов в новый тип узла

Теперь можно обновить исходный тип узла как не являющийся первичным и начать отключать его узлы. При отключении узлов системные службы и начальные узлы кластера переносятся в новый масштабируемый набор.

### <a name="unmark-the-original-node-type-as-primary"></a>Снять пометку с исходного типа узла в качестве первичного

Сначала удалите `isPrimary` обозначение в шаблоне из исходного типа узла.

```json
{
    "isPrimary": false,
}
```

Затем разверните шаблон с помощью обновления. Это приведет к инициации миграции начальных узлов в новый масштабируемый набор.

```powershell
$templateFilePath = "C:\Step2-UnmarkOriginalPrimaryNodeType.json"

New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $templateFilePath `
    -TemplateParameterFile $parameterFilePath `
    -CertificateThumbprint $thumb `
    -CertificateUrlValue $certUrlValue `
    -SourceVaultValue $sourceVaultValue `
    -Verbose
```

> [!Note]
> Для завершения миграции узла начального значения в новый масштабируемый набор потребуется некоторое время. Для обеспечения согласованности данных в каждый момент времени допускается изменение только одного начального узла. Для изменения каждого начального узла требуется обновление кластера. Соответственно, для замены начального узла требуются два обновления кластера (одно для добавления и второе для удаления узла). Для обновления пяти начальных узлов в этом примере сценария потребуются 10 обновлений кластера.

Используйте Service Fabric Explorer для мониторинга миграции начальных узлов в новый масштабируемый набор. Узлы исходного типа узла (nt0vm) должны иметь *значение false* в столбце " **начальное значение узла** ", а значения нового типа узла (nt1vm) будут *истинными*.

### <a name="disable-the-nodes-in-the-original-node-type-scale-set"></a>Отключение узлов в наборе масштабирования типа исходного узла

После переноса всех начальных узлов в новый масштабируемый набор можно отключить узлы исходного масштабируемого набора.

```powershell
# Disable the nodes in the original scale set.
$nodeType = "nt0vm"
$nodes = Get-ServiceFabricNode

Write-Host "Disabling nodes..."
foreach($node in $nodes)
{
  if ($node.NodeType -eq $nodeType)
  {
    $node.NodeName

    Disable-ServiceFabricNode -Intent RemoveNode -NodeName $node.NodeName -Force
  }
}
```

Используйте Service Fabric Explorer, чтобы отслеживать ход выполнения узлов в исходном масштабируемом наборе от *отключения* к *отключенному* состоянию.

:::image type="content" source="./media/scale-up-primary-node-type/service-fabric-explorer-node-status.png" alt-text="Service Fabric Explorer отображение состояния отключенных узлов":::

Для уровня устойчивости "серебро" и "Gold" некоторые узлы отключаются в отключенном состоянии, а другие могут оставаться в состоянии *отключения* . В Service Fabric Explorer просмотрите вкладку **сведения о** узлах в отключенном состоянии. Если они показывают *незавершенную проверку безопасности* типа *енсурепартитионкуорем* (обеспечение кворума для секций службы инфраструктуры), то можно продолжать работу.

:::image type="content" source="./media/scale-up-primary-node-type/service-fabric-explorer-node-status-disabling.png" alt-text="Вы можете продолжить процесс остановки данных и удалить узлы в состоянии &quot;отключено&quot;, если они показывают незавершенную проверку безопасности типа &quot;Енсурепартитионкуорум&quot;.":::

Если кластер имеет бронзовую устойчивость, дождитесь, пока все узлы выйдет из состояния " *отключено* ".

### <a name="stop-data-on-the-disabled-nodes"></a>Завершение данных на отключенных узлах

Теперь можно отключить данные на отключенных узлах.

```powershell
# Stop data on the disabled nodes.
foreach($node in $nodes)
{
  if ($node.NodeType -eq $nodeType)
  {
    $node.NodeName

    Start-ServiceFabricNodeTransition -Stop -OperationId (New-Guid) -NodeInstanceId $node.NodeInstanceId -NodeName $node.NodeName -StopDurationInSeconds 10000
  }
}
```

## <a name="remove-the-original-node-type-and-cleanup-its-resources"></a>Удаление исходного типа узла и очистка его ресурсов

Мы готовы удалить исходный тип узла и связанные с ним ресурсы для завершения процедуры вертикального масштабирования.

### <a name="remove-the-original-scale-set"></a>Удаление исходного масштабируемого набора

Сначала удалите резервный набор данных типа узла.

```powershell
$scaleSetName = "nt0vm"
$scaleSetResourceType = "Microsoft.Compute/virtualMachineScaleSets"

Remove-AzResource -ResourceName $scaleSetName -ResourceType $scaleSetResourceType -ResourceGroupName $resourceGroupName -Force
```

### <a name="delete-the-original-ip-and-load-balancer-resources"></a>Удаление исходных ресурсов IP-адреса и подсистемы балансировки нагрузки

Теперь можно удалить исходный IP-адрес и ресурсы балансировщика нагрузки. На этом шаге будет также обновлен DNS-имя.

> [!Note]
> Этот шаг является необязательным, если вы уже используете общедоступный IP-адрес SKU уровня " *стандартный* " и балансировщик нагрузки. В этом случае в одной подсистеме балансировки нагрузки может быть несколько масштабируемых наборов или типов узлов.

Выполните следующие команды, изменив `$lbname` значение при необходимости.

```powershell
# Delete the original IP and load balancer resources
$lbName = "LB-sftestupgrade-nt0vm"
$lbResourceType = "Microsoft.Network/loadBalancers"
$ipResourceType = "Microsoft.Network/publicIPAddresses"
$oldPublicIpName = "PublicIP-LB-FE-nt0vm"
$newPublicIpName = "PublicIP-LB-FE-nt1vm"

$oldPrimaryPublicIP = Get-AzPublicIpAddress -Name $oldPublicIpName  -ResourceGroupName $resourceGroupName
$primaryDNSName = $oldPrimaryPublicIP.DnsSettings.DomainNameLabel
$primaryDNSFqdn = $oldPrimaryPublicIP.DnsSettings.Fqdn

Remove-AzResource -ResourceName $lbName -ResourceType $lbResourceType -ResourceGroupName $resourceGroupName -Force
Remove-AzResource -ResourceName $oldPublicIpName -ResourceType $ipResourceType -ResourceGroupName $resourceGroupName -Force

$PublicIP = Get-AzPublicIpAddress -Name $newPublicIpName  -ResourceGroupName $resourceGroupName
$PublicIP.DnsSettings.DomainNameLabel = $primaryDNSName
$PublicIP.DnsSettings.Fqdn = $primaryDNSFqdn
Set-AzPublicIpAddress -PublicIpAddress $PublicIP
```

### <a name="remove-node-state-from-the-original-node-type"></a>Удалить состояние узла из исходного типа узла

Узлы исходного типа узла теперь будут показывать *ошибку* **состояния работоспособности**. Удалите состояние узла из кластера.

```powershell
# Remove state of the obsolete nodes from the cluster
$nodeType = "nt0vm"
$nodes = Get-ServiceFabricNode

Write-Host "Removing node state..."
foreach($node in $nodes)
{
  if ($node.NodeType -eq $nodeType)
  {
    $node.NodeName

    Remove-ServiceFabricNodeState -NodeName $node.NodeName -Force
  }
}
```

Теперь Service Fabric Explorer должны отражать только пять узлов нового типа узла (nt1vm) со значениями состояния работоспособности *ОК*. Состояние работоспособности кластера по-прежнему будет отображаться как *Ошибка*. Мы покажем следующее, обновив шаблон, чтобы отразить последние изменения и повторное развертывание.

### <a name="update-the-deployment-template-to-reflect-the-newly-scaled-up-primary-node-type"></a>Обновление шаблона развертывания для отражения нового масштабируемого типа первичного узла

Необходимые изменения для этого шага уже внесены в [*Step3-CleanupOriginalPrimaryNodeType.js*](https://github.com/microsoft/service-fabric-scripts-and-templates/tree/master/templates/nodetype-upgrade/Step3-CleanupOriginalPrimaryNodeType.json) в файле шаблона, а в следующих разделах подробно рассматриваются эти изменения шаблона. При желании вы можете пропустить объяснение и продолжить [развертывание обновленного шаблона](#deploy-the-finalized-template) и завершить работу с учебником.

#### <a name="update-the-cluster-management-endpoint"></a>Обновление конечной точки управления кластером

Обновите кластер `managementEndpoint` в шаблоне развертывания, чтобы он ссылался на новый IP-адрес (путем обновления *vmNodeType0Name* с помощью *vmNodeType1Name*).

```json
  "managementEndpoint": "[concat('https://',reference(concat(variables('lbIPName'),'-',variables('vmNodeType1Name'))).dnsSettings.fqdn,':',variables('nt0fabricHttpGatewayPort'))]",
```

#### <a name="remove-the-original-node-type-reference"></a>Удалить ссылку на исходный тип узла

Удалите ссылку на тип исходного узла из ресурса Service Fabric в шаблоне развертывания:

```json
"name": "[variables('vmNodeType0Name')]",
"applicationPorts": {
    "endPort": "[variables('nt0applicationEndPort')]",
    "startPort": "[variables('nt0applicationStartPort')]"
},
"clientConnectionEndpointPort": "[variables('nt0fabricTcpGatewayPort')]",
"durabilityLevel": "Bronze",
"ephemeralPorts": {
    "endPort": "[variables('nt0ephemeralEndPort')]",
    "startPort": "[variables('nt0ephemeralStartPort')]"
},
"httpGatewayEndpointPort": "[variables('nt0fabricHttpGatewayPort')]",
"isPrimary": true,
"reverseProxyEndpointPort": "[variables('nt0reverseProxyEndpointPort')]",
"vmInstanceCount": "[parameters('nt0InstanceCount')]"
```

#### <a name="configure-health-policies-to-ignore-existing-errors"></a>Настройка политик работоспособности для пропуска существующих ошибок

Только для отказоустойчивых и более высоких кластеров, обновите ресурс кластера в шаблоне и настройте политики исправности для игнорирования `fabric:/System` работоспособности приложения, добавив *аппликатионделтахеалсполиЦиес* в свойствах ресурса кластера, как указано ниже. Приведенная ниже политика игнорирует существующие ошибки, но не разрешает новые ошибки работоспособности.

```json
"upgradeDescription":  
{ 
 "forceRestart": false, 
 "upgradeReplicaSetCheckTimeout": "10675199.02:48:05.4775807", 
 "healthCheckWaitDuration": "00:05:00", 
 "healthCheckStableDuration": "00:05:00", 
 "healthCheckRetryTimeout": "00:45:00", 
 "upgradeTimeout": "12:00:00", 
 "upgradeDomainTimeout": "02:00:00", 
 "healthPolicy": { 
   "maxPercentUnhealthyNodes": 100, 
   "maxPercentUnhealthyApplications": 100 
 }, 
 "deltaHealthPolicy":  
 { 
   "maxPercentDeltaUnhealthyNodes": 0, 
   "maxPercentUpgradeDomainDeltaUnhealthyNodes": 0, 
   "maxPercentDeltaUnhealthyApplications": 0, 
   "applicationDeltaHealthPolicies":  
   { 
       "fabric:/System":  
       { 
           "defaultServiceTypeDeltaHealthPolicy":  
           { 
                   "maxPercentDeltaUnhealthyServices": 0 
           } 
       } 
   } 
 } 
}
```

#### <a name="remove-supporting-resources-for-the-original-node-type"></a>Удаление вспомогательных ресурсов для исходного типа узла

Удалите все другие ресурсы, связанные с исходным типом узла, из шаблона ARM и файла параметров. Удалите:

```json
    "vmImagePublisher": {
      "value": "MicrosoftWindowsServer"
    },
    "vmImageOffer": {
      "value": "WindowsServer"
    },
    "vmImageSku": {
      "value": "2016-Datacenter-with-Containers"
    },
    "vmImageVersion": {
      "value": "latest"
    },
```

#### <a name="deploy-the-finalized-template"></a>Развертывание окончательного шаблона

Наконец, разверните измененный шаблон Azure Resource Manager.

```powershell
# Deploy the updated template file
$templateFilePath = "C:\Step3-CleanupOriginalPrimaryNodeType"

New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $templateFilePath `
    -TemplateParameterFile $parameterFilePath `
    -CertificateThumbprint $thumb `
    -CertificateUrlValue $certUrlValue `
    -SourceVaultValue $sourceVaultValue `
    -Verbose
```

> [!NOTE]
> Этот шаг займет некоторое время, как правило, до двух часов.

Обновление изменит параметры на *инфраструктуресервице*; Поэтому требуется перезагрузка узла. В этом случае *форцерестарт* игнорируется. Параметр `upgradeReplicaSetCheckTimeout` задает максимальное время, которое Service Fabric ожидает, когда Секция находится в надежном состоянии, если она еще не находится в надежном состоянии. После проверки безопасности для всех секций на узле Service Fabric переходит к обновлению на этом узле. Значение параметра `upgradeTimeout` может быть сокращено до 6 часов, но для максимальной безопасности следует использовать 12 часов.

После завершения развертывания проверьте в портал Azure, что Service Fabric ресурс имеет состояние " *Готово*". Убедитесь, что вы можете подключиться к новой конечной точке Service Fabric Explorer, **состояние работоспособности кластера** — *ОК*, а все развернутые приложения работают правильно.

В результате вы вертикально масштабируюте тип основного узла кластера.

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте, как [добавить тип узла в кластер](virtual-machine-scale-set-scale-node-type-scale-out.md).
* Дополнительные сведения об [обновлениях приложений](service-fabric-concepts-scalability.md).
* [Руководство. Масштабирование кластера Service Fabric](service-fabric-tutorial-scale-cluster.md)
* [Масштабируйте кластер Azure программно](service-fabric-cluster-programmatic-scaling.md), используя свободный пакет SDK для вычислений Azure.
* [Увеличение и уменьшение масштаба автономного кластера](service-fabric-cluster-windows-server-add-remove-nodes.md).
