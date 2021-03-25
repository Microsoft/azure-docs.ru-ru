---
title: Поиск и использование сведений о плане покупки Marketplace с помощью PowerShell
description: Используйте Azure PowerShell, чтобы найти образы URN и параметры плана приобретения, такие как издатель, предложение, номер SKU и версия, для образов виртуальных машин Marketplace.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 03/17/2021
ms.author: cynthn
ms.custom: contperf-fy21q3
ms.openlocfilehash: 34fd6720b93a1462836b51856d73573a86809367
ms.sourcegitcommit: a8ff4f9f69332eef9c75093fd56a9aae2fe65122
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105022829"
---
# <a name="find-and-use-azure-marketplace-vm-images-with-azure-powershell"></a>Поиск и использование образов виртуальных машин Azure Marketplace с помощью Azure PowerShell

В этой статье описывается, как с помощью Azure PowerShell находить образы виртуальных машин в Azure Marketplace. Затем можно указать образ Marketplace и сведения о плане при создании виртуальной машины.

Вы также можете просмотреть доступные изображения и предложения с помощью [Azure Marketplace](https://azuremarketplace.microsoft.com/) или [Azure CLI](../linux/cli-ps-findimage.md). 

## <a name="terminology"></a>Терминология

Атрибуты образа Marketplace в Azure:

* **Издатель.** Организация, создавшая образ. Примеры: Canonical, MicrosoftWindowsServer.
* **Предложение.** Имя группы связанных образов, созданных издателем. Примеры: UbuntuServer, WindowsServer
* **Номер SKU.** Экземпляр предложения, например основной выпуск дистрибутива. Примеры: 18,04-LTS, 2019-Datacenter
* **Версия.** Номер версии SKU образа. 

Эти значения можно передавать по отдельности или в виде *URN* образа, объединяя значения, разделенные двоеточием (:). Например: *Издатель*:*предложение*:*SKU*:*версия*. Вы можете заменить номер версии в URN на, `latest` чтобы использовать последнюю версию образа. 

Если издатель образа предоставляет дополнительные условия лицензии и покупки, необходимо принять их, прежде чем можно будет использовать образ.  Дополнительные сведения см. в разделе [принятие условий плана приобретения](#accept-purchase-plan-terms).

## <a name="list-images"></a>Вывод списка образов

С помощью PowerShell можно сократить список образов. Замените значения переменных в соответствии с вашими потребностями.

1. Перечислите издатели образов с помощью команды [Get-азвмимажепублишер](/powershell/module/az.compute/get-azvmimagepublisher).
    
    ```powershell
    $locName="<location>"
    Get-AzVMImagePublisher -Location $locName | Select PublisherName
    ```
1. Для данного издателя перечислите свои предложения с помощью команды [Get-азвмимажеоффер](/powershell/module/az.compute/get-azvmimageoffer).
    
    ```powershell
    $pubName="<publisher>"
    Get-AzVMImageOffer -Location $locName -PublisherName $pubName | Select Offer
    ```
1. Для конкретного издателя и предложения выведите список номеров SKU, доступных с помощью команды [Get-азвмимажеску](/powershell/module/az.compute/get-azvmimagesku).
    
    ```powershell
    $offerName="<offer>"
    Get-AzVMImageSku -Location $locName -PublisherName $pubName -Offer $offerName | Select Skus
    ```
1. Для номера SKU перечислите версии образа с помощью команды [Get-азвмимаже](/powershell/module/az.compute/get-azvmimage).

    ```powershell
    $skuName="<SKU>"
    Get-AzVMImage -Location $locName -PublisherName $pubName -Offer $offerName -Sku $skuName | Select Version
    ```
    Можно также использовать `latest` , если вы хотите использовать последнюю версию образа, а не определенную более старую.


Теперь выбранного издателя, предложение, номер SKU и версию можно объединить в URN (значения, разделенные ":"). Передайте этот URN с параметром `--image` при создании виртуальной машины с помощью командлета [New-AzVM](/powershell/module/az.compute/new-azvm). Также можно заменить номер версии в URN на, `latest` чтобы получить последнюю версию образа.

При развертывании виртуальной машины с помощью шаблона Resource Manager установите отдельные параметры образа в свойствах `imageReference`. Ознакомьтесь со статьей о [справочнике по шаблонам](/azure/templates/microsoft.compute/virtualmachines).


## <a name="view-purchase-plan-properties"></a>Просмотр свойств плана покупки

Для некоторых образов виртуальных машин в Azure Marketplace применяются дополнительные условия лицензии и приобретения, которые нужно принять перед программным развертыванием. Необходимо принять условия использования образа один раз для каждой подписки.

Чтобы просмотреть сведения о плане покупки образа, выполните командлет `Get-AzVMImage`. Если в выходных данных значением свойства `PurchasePlan` не является `null`, в образе появятся условия использования, которые необходимо принять перед программным развертыванием.  

Например, образ *Windows Server 2016 Datacenter* не содержит дополнительные условия, так как в `PurchasePlan` указано значение `null`.

```powershell
$version = "2016.127.20170406"
Get-AzVMImage -Location $locName -PublisherName $pubName -Offer $offerName -Skus $skuName -Version $version
```

Вывод имеет следующий вид:

```output
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

Вывод имеет следующий вид:

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

Чтобы просмотреть условия лицензии, используйте командлет [Get-AzMarketplaceterms](/powershell/module/az.marketplaceordering/get-azmarketplaceterms) и передайте параметры плана приобретения. В выходных данных содержится ссылка на условия для образа в Marketplace, которая показывает, были ли эти условия приняты раннее. Обязательно используйте только строчные буквы в значениях параметров.

```powershell
Get-AzMarketplaceterms -Publisher "microsoft-ads" -Product "windows-data-science-vm" -Name "windows2016"
```

Вывод имеет следующий вид:

```output
Publisher         : microsoft-ads
Product           : windows-data-science-vm
Plan              : windows2016
LicenseTextLink   : https://storelegalterms.blob.core.windows.net/legalterms/3E5ED_legalterms_MICROSOFT%253a2DADS%253a24WINDOWS%253a2DDATA%253a2DSCIENCE%253a2DVM%253a24WINDOWS2016%253a24OC5SKMQOXSED66BBSNTF4XRCS4XLOHP7QMPV54DQU7JCBZWYFP35IDPOWTUKXUC7ZAG7W6ZMDD6NHWNKUIVSYBZUTZ245F44SU5AD7Q.txt
PrivacyPolicyLink : https://www.microsoft.com/EN-US/privacystatement/OnlineServices/Default.aspx
Signature         : 2UMWH6PHSAIM4U22HXPXW25AL2NHUJ7Y7GRV27EBL6SUIDURGMYG6IIDO3P47FFIBBDFHZHSQTR7PNK6VIIRYJRQ3WXSE6BTNUNENXA
Accepted          : False
Signdate          : 1/25/2019 7:43:00 PM
```

## <a name="accept-purchase-plan-terms"></a>Принятие условий плана приобретения

Чтобы принять или отклонить условия, используйте командлет [Set-AzMarketplaceterms](/powershell/module/az.marketplaceordering/set-azmarketplaceterms). Необходимо принять условия соглашения для каждой подписки в образе. Обязательно используйте только строчные буквы в значениях параметров. 

```powershell
$agreementTerms=Get-AzMarketplaceterms -Publisher "microsoft-ads" -Product "windows-data-science-vm" -Name "windows2016"

Set-AzMarketplaceTerms -Publisher "microsoft-ads" -Product "windows-data-science-vm" -Name "windows2016" -Terms $agreementTerms -Accept
```



```output
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

Если вы получите сообщение о принятии условий образа, см. предыдущий раздел " [принятие условий плана приобретения](#accept-purchase-plan-terms)".

## <a name="create-a-new-vm-from-a-vhd-with-purchase-plan-information"></a>Создание новой виртуальной машины из виртуального жесткого диска с информацией о плане покупки

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


## <a name="next-steps"></a>Следующие шаги

Инструкции по быстрому созданию виртуальной машины с помощью командлета `New-AzVM` на основе полученных данных образа см. в статье [Создание виртуальной машины Windows с помощью PowerShell](quick-create-powershell.md).

Дополнительные сведения об использовании образов Azure Marketplace для создания пользовательских образов в общей коллекции образов см. в статье [предоставление сведений о плане покупки Azure Marketplace при создании образов](../marketplace-images.md).
