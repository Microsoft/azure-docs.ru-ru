---
title: Поиск и использование образов и планов Azure Marketplace
description: Используйте Azure PowerShell, чтобы найти и использовать сведения о издателе, предложении, номере SKU, версии и плане для образов виртуальных машин Marketplace.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 12/07/2020
ms.author: cynthn
ms.openlocfilehash: 45e6b157dba5ef7410d8a5c0223fd3ecb52f39d0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96906273"
---
# <a name="find-and-use-azure-marketplace-vm-images-with-azure-powershell"></a>Поиск и использование образов виртуальных машин Azure Marketplace с помощью Azure PowerShell     

В этой статье описывается, как с помощью Azure PowerShell находить образы виртуальных машин в Azure Marketplace. Затем можно указать образ Marketplace и сведения о плане при создании виртуальной машины.

Вы также можете просмотреть доступные образы и предложения в онлайн-магазине [Microsoft Azure Marketplace](https://azuremarketplace.microsoft.com/), на [портале Azure](https://portal.azure.com) или с помощью [Azure CLI](../linux/cli-ps-findimage.md). 


[!INCLUDE [virtual-machines-common-image-terms](../../../includes/virtual-machines-common-image-terms.md)]


## <a name="create-a-vm-from-vhd-with-plan-information"></a>Создание виртуальной машины из виртуального жесткого диска с использованием сведений о плане

При наличии существующего виртуального жесткого диска, созданного с помощью образа Azure Marketplace, может потребоваться предоставить сведения о плане покупки при создании новой виртуальной машины из этого виртуального жесткого диска.

Если у вас по-прежнему есть исходная виртуальная машина или другая виртуальная машина, созданная из того же образа, можно получить имя плана, издателя и сведения о продукте с помощью команды Get-AzVM. Этот пример получает виртуальную машину с именем *myVM* в группе ресурсов *myResourceGroup* , а затем отображает сведения о плане покупки.

```azurepowershell-interactive
$vm = Get-azvm `
   -ResourceGroupName myResourceGroup `
   -Name myVM
$vm.Plan
```

Если вы не получили сведения о плане до удаления исходной виртуальной машины, можно [Отправить запрос в службу поддержки](https://ms.portal.azure.com/#create/Microsoft.Support). Им потребуется имя виртуальной машины, идентификатор подписки и метка времени операции удаления.

Сведения о создании виртуальной машины с помощью виртуального жесткого диска см. в статье [Создание виртуальной машины на основе специализированного виртуального жесткого диска](create-vm-specialized.md) и Добавление сведений о планах в конфигурацию виртуальной машины с помощью командлета [Set-азвмплан](/powershell/module/az.compute/set-azvmplan) , аналогичного следующему:

```azurepowershell-interactive
$vmConfig = Set-AzVMPlan `
   -VM $vmConfig `
   -Publisher "publisherName" `
   -Product "productName" `
   -Name "planName"
```

## <a name="create-a-new-vm-from-a-marketplace-image"></a>Создание новой виртуальной машины из образа Marketplace

Если у вас уже есть сведения о том, какой образ вы хотите использовать, эту информацию можно передать в командлет [Set-азвмсаурцеимаже](/powershell/module/az.compute/set-azvmsourceimage) , чтобы добавить сведения об образе в конфигурацию виртуальной машины. В следующих разделах содержатся сведения о поиске и перечислении образов, доступных в Marketplace.

Для некоторых платных образов также требуется предоставить сведения о плане покупки с помощью команды [Set-азвмплан](/powershell/module/az.compute/set-azvmplan). 

```powershell
...

$vmConfig = New-AzVMConfig -VMName "myVM" -VMSize Standard_D1

# Set the Marketplace image
$offerName = "windows-data-science-vm"
$skuName = "windows2016"
$version = "19.01.14"
$vmConfig = Set-AzVMSourceImage -VM $vmConfig -PublisherName $publisherName -Offer $offerName -Skus $skuName -Version $version

# Set the Marketplace plan information, if needed
$publisherName = "microsoft-ads"
$productName = "windows-data-science-vm"
$planName = "windows2016"
$vmConfig = Set-AzVMPlan -VM $vmConfig -Publisher $publisherName -Product $productName -Name $planName

...
```

Затем вы передадите конфигурацию виртуальной машины вместе с другими объектами конфигурации в `New-AzVM` командлет. Подробный пример использования конфигурации виртуальной машины с помощью PowerShell см. в этом [сценарии](https://github.com/Azure/azure-docs-powershell-samples/blob/master/virtual-machine/create-vm-detailed/create-windows-vm-detailed.ps1).

Если появится сообщение о принятии условий образа, см. раздел [примите условия](#accept-the-terms) , приведенные далее в этой статье.

## <a name="list-images"></a>Вывод списка образов

Один из способов поиска образа в расположении основан на запуске командлетов [Get-AzVMImagePublisher](/powershell/module/az.compute/get-azvmimagepublisher), [Get-AzVMImageOffer](/powershell/module/az.compute/get-azvmimageoffer) и [Get-AzVMImageSku](/powershell/module/az.compute/get-azvmimagesku) по порядку.

1. Получить список издателей образов.
2. Получить список предложений нужного издателя.
3. Получить список номеров SKU для требуемого предложения.

Затем, чтобы отобразить версии для развертывания, выполните команду [Get-AzVMImage](/powershell/module/az.compute/get-azvmimage) для выбранного SKU.

1. Отобразите список издателей:

    ```powershell
    $locName="<Azure location, such as West US>"
    Get-AzVMImagePublisher -Location $locName | Select PublisherName
    ```

2. Укажите название выбранного издателя и выведите список предложений:

    ```powershell
    $pubName="<publisher>"
    Get-AzVMImageOffer -Location $locName -PublisherName $pubName | Select Offer
    ```

3. Укажите название выбранного предложения и выведите список SKU:

    ```powershell
    $offerName="<offer>"
    Get-AzVMImageSku -Location $locName -PublisherName $pubName -Offer $offerName | Select Skus
    ```

4. Укажите название выбранного SKU и получите версию образа:

    ```powershell
    $skuName="<SKU>"
    Get-AzVMImage -Location $locName -PublisherName $pubName -Offer $offerName -Sku $skuName | Select Version
    ```
    
Версию образа для развертывания новой виртуальной машины можно выбрать из выходных данных команды `Get-AzVMImage`.

В следующем примере приведена полная последовательность команд и их выходных данных:

```powershell
$locName="West US"
Get-AzVMImagePublisher -Location $locName | Select PublisherName
```

Частичные выходные данные приведены ниже.

```
PublisherName
-------------
...
abiquo
accedian
accellion
accessdata-group
accops
Acronis
Acronis.Backup
actian-corp
actian_matrix
actifio
activeeon
adgs
advantech
advantech-webaccess
advantys
...
```

Для издателя *MicrosoftWindowsServer* :

```powershell
$pubName="MicrosoftWindowsServer"
Get-AzVMImageOffer -Location $locName -PublisherName $pubName | Select Offer
```

Выходные данные:

```
Offer
-----
Windows-HUB
WindowsServer
WindowsServerSemiAnnual
```

Для предложения *WindowsServer*:

```powershell
$offerName="WindowsServer"
Get-AzVMImageSku -Location $locName -PublisherName $pubName -Offer $offerName | Select Skus
```

Частичные выходные данные приведены ниже.

```
Skus
----
2008-R2-SP1
2008-R2-SP1-smalldisk
2012-Datacenter
2012-Datacenter-smalldisk
2012-R2-Datacenter
2012-R2-Datacenter-smalldisk
2016-Datacenter
2016-Datacenter-Server-Core
2016-Datacenter-Server-Core-smalldisk
2016-Datacenter-smalldisk
2016-Datacenter-with-Containers
2016-Datacenter-with-RDSH
2019-Datacenter
2019-Datacenter-Core
2019-Datacenter-Core-smalldisk
2019-Datacenter-Core-with-Containers
...
```

Затем для номера SKU *2019-Datacenter*:

```powershell
$skuName="2019-Datacenter"
Get-AzVMImage -Location $locName -PublisherName $pubName -Offer $offerName -Sku $skuName | Select Version
```

Теперь выбранного издателя, предложение, номер SKU и версию можно объединить в URN (значения, разделенные ":"). Передайте этот URN с параметром `--image` при создании виртуальной машины с помощью командлета [New-AzVM](/powershell/module/az.compute/new-azvm). При необходимости вы можете заменить номер версии в URN словом "latest" (последняя), чтобы получить последнюю версию образа.

При развертывании виртуальной машины с помощью шаблона Resource Manager установите отдельные параметры образа в свойствах `imageReference`. Ознакомьтесь со статьей о [справочнике по шаблонам](/azure/templates/microsoft.compute/virtualmachines).

[!INCLUDE [virtual-machines-common-marketplace-plan](../../../includes/virtual-machines-common-marketplace-plan.md)]

### <a name="view-plan-properties"></a>Просмотр свойств плана

Чтобы просмотреть сведения о плане покупки образа, выполните командлет `Get-AzVMImage`. Если в выходных данных значением свойства `PurchasePlan` не является `null`, в образе появятся условия использования, которые необходимо принять перед программным развертыванием.  

Например, образ *Windows Server 2016 Datacenter* не содержит дополнительные условия, так как в `PurchasePlan` указано значение `null`.

```powershell
$version = "2016.127.20170406"
Get-AzVMImage -Location $locName -PublisherName $pubName -Offer $offerName -Skus $skuName -Version $version
```

Выходные данные:

```
Id               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Providers/Microsoft.Compute/Locations/westus/Publishers/MicrosoftWindowsServer/ArtifactTypes/VMImage/Offers/WindowsServer/Skus/2016-Datacenter/Versions/2019.0.20190115
Location         : westus
PublisherName    : MicrosoftWindowsServer
Offer            : WindowsServer
Skus             : 2019-Datacenter
Version          : 2019.0.20190115
FilterExpression :
Name             : 2019.0.20190115
OSDiskImage      : {
                     "operatingSystem": "Windows"
                   }
PurchasePlan     : null
DataDiskImages   : []

```

В приведенном ниже примере показана аналогичная команда для образа *виртуальной машины для обработки и анализа данных — Windows 2016* , которая имеет следующие `PurchasePlan` Свойства: `name` , `product` и `publisher` . Некоторые образы также имеют свойство `promotion code`. Ознакомьтесь со следующими разделами для развертывания этого образа, чтобы принять условия соглашения и включить программное развертывание.

```powershell
Get-AzVMImage -Location "westus" -PublisherName "microsoft-ads" -Offer "windows-data-science-vm" -Skus "windows2016" -Version "0.2.02"
```

Выходные данные:

```
Id               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Providers/Microsoft.Compute/Locations/westus/Publishers/microsoft-ads/ArtifactTypes/VMImage/Offers/windows-data-science-vm/Skus/windows2016/Versions/19.01.14
Location         : westus
PublisherName    : microsoft-ads
Offer            : windows-data-science-vm
Skus             : windows2016
Version          : 19.01.14
FilterExpression :
Name             : 19.01.14
OSDiskImage      : {
                     "operatingSystem": "Windows"
                   }
PurchasePlan     : {
                     "publisher": "microsoft-ads",
                     "name": "windows2016",
                     "product": "windows-data-science-vm"
                   }
DataDiskImages   : []

```

### <a name="accept-the-terms"></a>Принятие условий

Чтобы просмотреть условия лицензии, используйте командлет [Get-AzMarketplaceterms](/powershell/module/az.marketplaceordering/get-azmarketplaceterms) и передайте параметры плана приобретения. В выходных данных содержится ссылка на условия для образа в Marketplace, которая показывает, были ли эти условия приняты раннее. Обязательно используйте только строчные буквы в значениях параметров.

```powershell
Get-AzMarketplaceterms -Publisher "microsoft-ads" -Product "windows-data-science-vm" -Name "windows2016"
```

Выходные данные:

```
Publisher         : microsoft-ads
Product           : windows-data-science-vm
Plan              : windows2016
LicenseTextLink   : https://storelegalterms.blob.core.windows.net/legalterms/3E5ED_legalterms_MICROSOFT%253a2DADS%253a24WINDOWS%253a2DDATA%253a2DSCIENCE%253a2DVM%253a24WINDOWS2016%253a24OC5SKMQOXSED66BBSNTF4XRCS4XLOHP7QMPV54DQU7JCBZWYFP35IDPOWTUKXUC7ZAG7W6ZMDD6NHWNKUIVSYBZUTZ245F44SU5AD7Q.txt
PrivacyPolicyLink : https://www.microsoft.com/EN-US/privacystatement/OnlineServices/Default.aspx
Signature         : 2UMWH6PHSAIM4U22HXPXW25AL2NHUJ7Y7GRV27EBL6SUIDURGMYG6IIDO3P47FFIBBDFHZHSQTR7PNK6VIIRYJRQ3WXSE6BTNUNENXA
Accepted          : False
Signdate          : 1/25/2019 7:43:00 PM
```

Чтобы принять или отклонить условия, используйте командлет [Set-AzMarketplaceterms](/powershell/module/az.marketplaceordering/set-azmarketplaceterms). Необходимо принять условия соглашения для каждой подписки в образе. Обязательно используйте только строчные буквы в значениях параметров. 

```powershell
$agreementTerms=Get-AzMarketplaceterms -Publisher "microsoft-ads" -Product "windows-data-science-vm" -Name "windows2016"

Set-AzMarketplaceTerms -Publisher "microsoft-ads" -Product "windows-data-science-vm" -Name "windows2016" -Terms $agreementTerms -Accept
```

Выходные данные:

```
Publisher         : microsoft-ads
Product           : windows-data-science-vm
Plan              : windows2016
LicenseTextLink   : https://storelegalterms.blob.core.windows.net/legalterms/3E5ED_legalterms_MICROSOFT%253a2DADS%253a24WINDOWS%253a2DDATA%253a2DSCIENCE%253a2DV
                    M%253a24WINDOWS2016%253a24OC5SKMQOXSED66BBSNTF4XRCS4XLOHP7QMPV54DQU7JCBZWYFP35IDPOWTUKXUC7ZAG7W6ZMDD6NHWNKUIVSYBZUTZ245F44SU5AD7Q.txt
PrivacyPolicyLink : https://www.microsoft.com/EN-US/privacystatement/OnlineServices/Default.aspx
Signature         : XXXXXXK3MNJ5SROEG2BYDA2YGECU33GXTD3UFPLPC4BAVKAUL3PDYL3KBKBLG4ZCDJZVNSA7KJWTGMDSYDD6KRLV3LV274DLBXXXXXX
Accepted          : True
Signdate          : 2/23/2018 7:49:31 PM
```



## <a name="next-steps"></a>Дальнейшие действия

Инструкции по быстрому созданию виртуальной машины с помощью командлета `New-AzVM` на основе полученных данных образа см. в статье [Создание виртуальной машины Windows с помощью PowerShell](quick-create-powershell.md).

Дополнительные сведения об использовании образов Azure Marketplace для создания пользовательских образов в общей коллекции образов см. в статье [предоставление сведений о плане покупки Azure Marketplace при создании образов](../marketplace-images.md).
