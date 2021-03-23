---
title: Развертывание виртуальных машин Azure на месте с помощью PowerShell
description: Узнайте, как использовать Azure PowerShell для развертывания виртуальных машин Azure на месте, чтобы сэкономить на затратах.
author: cynthn
ms.service: virtual-machines
ms.subservice: spot
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 03/22/2021
ms.author: cynthn
ms.reviewer: jagaveer
ms.openlocfilehash: 9a2ad2eb197af613919efa4414da1759cd47e2e7
ms.sourcegitcommit: ba3a4d58a17021a922f763095ddc3cf768b11336
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104802749"
---
# <a name="deploy-azure-spot-virtual-machines-using-azure-powershell"></a>Развертывание виртуальных машин с использованием точки Azure с помощью Azure PowerShell


Используя [виртуальные машины Azure](../spot-vms.md) , вы можете использовать преимущества нашей неиспользуемой емкости с значительной экономией затрат. В любой момент, когда Azure нужна емкость, инфраструктура Azure будет выключать виртуальные машины Azure. Таким образом, виртуальные машины Azure для виртуальных машин отлично подходят для рабочих нагрузок, которые могут обрабатывать прерывания, такие как задания пакетной обработки, среды разработки и тестирования, большие вычислительные рабочие нагрузки и многое другое.

Цены на виртуальные машины Azure для машинного хранения являются переменными на основе региона и SKU. Дополнительные сведения см. на страницах с информацией о ценах на виртуальные машины [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) и [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/). Дополнительные сведения о настройке максимальной цены см. в статье о [ценах на виртуальные машины Azure](../spot-vms.md#pricing).

Вы можете задать максимальную цену, которую вы хотите платить, в час, для виртуальной машины. Максимальная цена для виртуальной машины точки Azure может быть задана в долларах США (USD), в которой используется до 5 десятичных разрядов. Например, значение `0.98765` определяет максимальную цену 0,98765 долларов США в час. Если вы укажете для максимальной цены значение `-1`, виртуальная машине не будет вытесняться по критерию цены. Цена на такую виртуальную машину будет определяться меньшим из двух значений: текущая цена точечных виртуальных машин или цена на стандартные виртуальные машины, но только при условии наличия емкости и соблюдения квоты.


## <a name="create-the-vm"></a>Создание виртуальной машины

Создайте Спотвм с помощью [New-азвмконфиг](/powershell/module/az.compute/new-azvmconfig) , чтобы создать конфигурацию. Включите `-Priority Spot` и задайте `-MaxPrice` одно из следующих действий:
- `-1` так что виртуальная машина не будет исключена на основе цены.
- сумма в долларах, до 5 цифр. Например, это `-MaxPrice .98765` означает, что виртуальная машина будет освобождена, когда цена за спотвм займет около $. 98765 в час.


В этом примере создается Спотвм, который не будет освобожден на основе ценообразования (только в том случае, если Azure требуется обратная емкость). Политика вытеснения устанавливается таким образом, чтобы освободить виртуальную машину, чтобы ее можно было перезапустить позднее. Если вы хотите удалить виртуальную машину и базовый диск при удалении виртуальной машины, присвойте параметру `-EvictionPolicy` значение `Delete` в `New-AzVMConfig` .


```azurepowershell-interactive
$resourceGroup = "mySpotRG"
$location = "eastus"
$vmName = "mySpotVM"
$cred = Get-Credential -Message "Enter a username and password for the virtual machine."
New-AzResourceGroup -Name $resourceGroup -Location $location
$subnetConfig = New-AzVirtualNetworkSubnetConfig `
   -Name mySubnet -AddressPrefix 192.168.1.0/24
$vnet = New-AzVirtualNetwork -ResourceGroupName $resourceGroup `
   -Location $location -Name MYvNET -AddressPrefix 192.168.0.0/16 `
   -Subnet $subnetConfig
$pip = New-AzPublicIpAddress -ResourceGroupName $resourceGroup -Location $location `
  -Name "mypublicdns$(Get-Random)" -AllocationMethod Static -IdleTimeoutInMinutes 4
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleRDP  -Protocol Tcp `
  -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
  -DestinationPortRange 3389 -Access Allow
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $resourceGroup -Location $location `
  -Name myNetworkSecurityGroup -SecurityRules $nsgRuleRDP
$nic = New-AzNetworkInterface -Name myNic -ResourceGroupName $resourceGroup -Location $location `
  -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id

# Create a virtual machine configuration and set this to be an Azure Spot Virtual Machine

$vmConfig = New-AzVMConfig -VMName $vmName -VMSize Standard_D1 -Priority "Spot" -MaxPrice -1 -EvictionPolicy Deallocate | `
Set-AzVMOperatingSystem -Windows -ComputerName $vmName -Credential $cred | `
Set-AzVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2016-Datacenter -Version latest | `
Add-AzVMNetworkInterface -Id $nic.Id

New-AzVM -ResourceGroupName $resourceGroup -Location $location -VM $vmConfig
```

После создания виртуальной машины можно выполнить запрос, чтобы увидеть максимальную цену для всех виртуальных машин в группе ресурсов.

```azurepowershell-interactive
Get-AzVM -ResourceGroupName $resourceGroup | `
   Select-Object Name,@{Name="maxPrice"; Expression={$_.BillingProfile.MaxPrice}}
```

## <a name="simulate-an-eviction"></a>Имитация вытеснения

Вы можете имитировать вытеснение виртуальной машины в машинном коде Azure с помощью функции RESTFUL, PowerShell или интерфейса командной строки, чтобы проверить, насколько хорошо ваше приложение будет реагировать на внезапное вытеснение.

В большинстве случаев необходимо использовать REST API [виртуальные машины — имитировать вытеснение](/rest/api/compute/virtualmachines/simulateeviction) , чтобы помочь в автоматическом тестировании приложений. Для остальных компонентов `Response Code: 204` означает, что имитация вытеснения прошла успешно. Имитацию вытеснений можно объединить с [запланированной службой событий](scheduled-events.md), чтобы автоматизировать реакцию приложения на то, что виртуальная машина будет удалена.

Чтобы просмотреть запланированные события в действии, просмотрите [пятницу Azure с помощью запланированные события Azure, чтобы подготовиться к обслуживанию виртуальных машин](https://channel9.msdn.com/Shows/Azure-Friday/Using-Azure-Scheduled-Events-to-Prepare-for-VM-Maintenance).


### <a name="quick-test"></a>Быстрый тест

Для быстрой проверки того, как будет работать имитация вытеснения, давайте рассмотрим запрос запланированной службы событий, чтобы увидеть, как она выглядит при имитации вытеснения с помощью PowerShell.

Служба запланированных событий включена для службы при первом выполнении запроса событий. 

Выполните удаленную работу в виртуальной машине, а затем откройте командную строку. 

В командной строке на виртуальной машине введите:

```
curl -H Metadata:true http://169.254.169.254/metadata/scheduledevents?api-version=2019-08-01
```

Первый ответ может занять до 2 минут. Теперь они должны отображать выходные данные практически сразу же.

На компьютере с установленным модулем AZ PowerShell (например, на локальном компьютере) имитируйте вытеснение с помощью командлета [Set-AzVM](https://docs.microsoft.com/powershell/module/az.compute/set-azvm). Замените имя группы ресурсов и имя виртуальной машины собственными. 

```azurepowershell-interactive
Set-AzVM -ResourceGroupName "mySpotRG" -Name "mySpotVM" -SimulateEviction
```

Выходные данные ответа будут иметься, `Status: Succeeded` Если запрос был успешно выполнен.

Быстро вернитесь к удаленному подключению к своей виртуальной машине и повторите запрос к конечной точке Запланированные события. Повторите следующую команду, пока не получите выходные данные, содержащие дополнительные сведения:

```
curl -H Metadata:true http://169.254.169.254/metadata/scheduledevents?api-version=2019-08-01
```

Когда служба запланированных событий получает уведомление о вытеснении, вы получите ответ, похожий на следующий:

```output
{"DocumentIncarnation":1,"Events":[{"EventId":"A123BC45-1234-5678-AB90-ABCDEF123456","EventStatus":"Scheduled","EventType":"Preempt","ResourceType":"VirtualMachine","Resources":["myspotvm"],"NotBefore":"Tue, 16 Mar 2021 00:58:46 GMT","Description":"","EventSource":"Platform"}]}
```

Видно, что `"EventType":"Preempt"` ресурс является ресурсом виртуальной машины `"Resources":["myspotvm"]` . 

Вы также можете увидеть, когда виртуальная машина будет удалена путем проверки `"NotBefore"` значения. Виртуальная машина не будет удалена до времени, указанного в `NotBefore` , поэтому ваше окно приложения должно быть корректно закрыто.


## <a name="next-steps"></a>Следующие шаги

Вы также можете создать виртуальную машину Azure с помощью [Azure CLI](../linux/spot-cli.md), [портала](../spot-portal.md) или [шаблона](../linux/spot-template.md).

Запросите текущие сведения о ценах с помощью [API розничных цен Azure](/rest/api/cost-management/retail-prices/azure-retail-prices) , чтобы получить сведения о ценах на виртуальные машины Azure для машинного характера. Объект `meterName` и `skuName` будет содержать `Spot` .

Если возникает ошибка, см. раздел [коды ошибок](../error-codes-spot.md).
