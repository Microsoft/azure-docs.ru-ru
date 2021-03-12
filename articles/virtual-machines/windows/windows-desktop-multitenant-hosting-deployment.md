---
title: Как развернуть Windows 10 в Azure с правами на мультитенантное размещение
description: Узнайте, как максимально использовать преимущества Windows Software Assurance для предоставления локальных лицензий в Azure с правами на размещение клиентов.
author: mimckitt
ms.service: virtual-machines
ms.collection: windows
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 2/2/2021
ms.author: mimckitt
ms.custom: rybaker, chmimckitt
ms.openlocfilehash: bb86ba6867ad796ef0f5eeb1357a6df9e93e9f9e
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102555777"
---
# <a name="how-to-deploy-windows-10-on-azure-with-multitenant-hosting-rights"></a>Как развернуть Windows 10 в Azure с правами на мультитенантное размещение 
Клиентам, использующим Windows 10 Корпоративная E3 или Windows 10 Корпоративная E5 для каждого пользователя, либо Windows VDA для каждого пользователя (лицензии на подписку пользователя или дополнительные лицензии на подписку пользователя), права на мультитенантное размещение для Windows 10 позволяют перенести лицензии Windows 10 в облако и запустить виртуальные машины Windows 10 в Azure без необходимости платить за другую лицензию. Права на размещение для клиентов доступны только для Windows 10 (версия 1703 или более поздняя).

Дополнительные сведения см. в статье [Размещение нескольких клиентов для Windows 10](https://www.microsoft.com/en-us/CloudandHosting/licensing_sca.aspx).

> [!NOTE]
> - Чтобы использовать образы Windows 7, 8,1 и 10 для разработки или тестирования, см. раздел [клиент Windows в Azure для сценариев разработки и тестирования](client-images.md) .
> - Чтобы воспользоваться преимуществами лицензирования Windows Server, см. статью [Преимущество гибридного использования Azure для Windows Server](hybrid-use-benefit-licensing.md).

## <a name="subscription-licenses-that-qualify-for-multitenant-hosting-rights"></a>Лицензии на подписку, подходящие для прав на размещение клиентов

С помощью [центра администрирования Майкрософт](/microsoft-365/admin/admin-overview/about-the-admin-center)можно подтвердить, что пользователю назначена поддерживаемая лицензия Windows 10.

> [!IMPORTANT]
> Для использования образов Windows 10 в Azure пользователи должны иметь одну из следующих лицензий подписки. Если у вас нет одной из этих лицензий, их можно приобрести через [партнера по облачным службам](https://azure.microsoft.com/overview/choosing-a-cloud-service-provider/) или напрямую в [корпорации Майкрософт](https://www.microsoft.com/microsoft-365?rtc=1).

**Подходящие лицензии на подписку:**

-   Microsoft 365 E3/с 
-   Microsoft 365 F3 
-   Microsoft 365 a3/A5 
-   Windows 10 Корпоративная E3/с
-   Windows 10 для образовательных учреждений a3/A5 
-   Windows VDA E3/в.


## <a name="deploying-windows-10-image-from-azure-marketplace"></a>Развертывание образа Windows 10 из Azure Marketplace 
Для развертывания шаблонов PowerShell, CLI и Azure Resource Manager можно найти образы Windows 10 с помощью `PublisherName: MicrosoftWindowsDesktop` и `Offer: Windows-10` . Для прав на размещение клиентов поддерживается обновление "создатели версий Windows 10" (1809) или более поздней версии. 

```powershell
Get-AzVmImageSku -Location '$location' -PublisherName 'MicrosoftWindowsDesktop' -Offer 'Windows-10'

Skus                        Offer      PublisherName           Location 
----                        -----      -------------           -------- 
rs4-pro                     Windows-10 MicrosoftWindowsDesktop eastus   
rs4-pron                    Windows-10 MicrosoftWindowsDesktop eastus   
rs5-enterprise              Windows-10 MicrosoftWindowsDesktop eastus   
rs5-enterprisen             Windows-10 MicrosoftWindowsDesktop eastus   
rs5-pro                     Windows-10 MicrosoftWindowsDesktop eastus   
rs5-pron                    Windows-10 MicrosoftWindowsDesktop eastus  
```

Дополнительные сведения о доступных образах см. в статье [Поиск и использование образов виртуальных машин Azure Marketplace с помощью Azure PowerShell](./cli-ps-findimage.md)

## <a name="uploading-windows-10-vhd-to-azure"></a>Загрузка виртуального жесткого диска Windows 10 в Azure
Если вы отправляете обобщенный виртуальный жесткий диск (VHD) Windows 10, учтите, что в Windows 10 встроенная учетная запись администратора не включена по умолчанию. Чтобы включить ее, добавьте следующую команду в качестве элемента расширения пользовательского скрипта.

```powershell
Net user <username> /active:yes
```

Следующий фрагмент кода PowerShell позволяет пометить все учетные записи администраторов как активные, включая встроенного администратора. Вам подойдет этот пример, если имя пользователя встроенной учетной записи администратора неизвестно.
```powershell
$adminAccount = Get-WmiObject Win32_UserAccount -filter "LocalAccount=True" | ? {$_.SID -Like "S-1-5-21-*-500"}
if($adminAccount.Disabled)
{
    $adminAccount.Disabled = $false
    $adminAccount.Put()
}
```
Дополнительные сведения 
* [Отправка универсального диска VHD и создание виртуальных машин с его помощью в Azure](upload-generalized-managed.md)
* [Подготовка диска VHD или VHDX для Windows к отправке в Azure](prepare-for-upload-vhd-image.md)


## <a name="deploying-windows-10-with-multitenant-hosting-rights"></a>Развертывание Windows 10 с правами на мультитенантное размещение
Убедитесь, что у вас [установлена и настроена последняя версия Azure PowerShell](/powershell/azure/). Чтобы передать подготовленный виртуальный жесткий диск в учетную запись хранения Azure, выполните командлет `Add-AzVhd` со следующими параметрами:

```powershell
Add-AzVhd -ResourceGroupName "myResourceGroup" -LocalFilePath "C:\Path\To\myvhd.vhd" `
    -Destination "https://mystorageaccount.blob.core.windows.net/vhds/myvhd.vhd"
```


**Развертывание с помощью шаблона Azure Resource Manager** В шаблонах Resource Manager можно указать дополнительный параметр для `licenseType`. Дополнительные сведения о [создании шаблонов Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md)см. в этой статье. После загрузки виртуального жесткого диска в Azure необходимо изменить шаблон Resource Manager, чтобы включить в него тип лицензирования как часть поставщика вычислительных ресурсов и развернуть шаблон в обычном режиме.
```json
"properties": {
    "licenseType": "Windows_Client",
    "hardwareProfile": {
        "vmSize": "[variables('vmSize')]"
    }
```

**Развертывание через PowerShell** При развертывании виртуальной машины Windows Server с помощью PowerShell доступен дополнительный параметр для `-LicenseType`. После передачи виртуального жесткого диска в Azure необходимо создать новую виртуальную машину с помощью командлета `New-AzVM` и указать тип лицензирования следующим образом.
```powershell
New-AzVM -ResourceGroupName "myResourceGroup" -Location "West US" -VM $vm -LicenseType "Windows_Client"
```

## <a name="verify-your-vm-is-utilizing-the-licensing-benefit"></a>Проверка наличия льготы на гибридное использование Azure для Windows Server
После развертывания виртуальной машины с помощью Resource Manager или PowerShell проверьте тип лицензии с помощью командлета `Get-AzVM` следующим образом.
```powershell
Get-AzVM -ResourceGroup "myResourceGroup" -Name "myVM"
```

Выходные данные аналогичны приведенному ниже примеру для Windows 10 с правильным типом лицензии:

```powershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              : Windows_Client
```

Эти выходные данные отличаются от полученных при развертывании виртуальной машины без преимуществ гибридного использования Azure (например, при развертывании виртуальной машины непосредственно из коллекции Azure).

```powershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              :
```

## <a name="additional-information-about-joining-azure-ad"></a>Дополнительная информация о присоединении к Azure AD
Azure подготавливает все виртуальные машины Windows со встроенной учетной записью администратора, которую нельзя использовать для присоединения к AAD. Например, выбор команды *Параметры > Учетная запись > Доступ к рабочей или учебной учетной записи > + Подключиться* не сработает. Чтобы присоединиться к Azure AD вручную, необходимо создать журнал и войти в систему, используя учетную запись второго администратора. Вы также можете настроить Azure AD с помощью пакета подготовки, воспользовавшись ссылкой в разделе *дальнейшие действия* , чтобы получить дополнительные сведения.

## <a name="next-steps"></a>Следующие шаги
- Узнайте больше о [настройке VDA для Windows 10](/windows/deployment/vda-subscription-activation).
- Узнайте больше о [правах на мультитенантное размещение для Windows 10](https://www.microsoft.com/en-us/CloudandHosting/licensing_sca.aspx).