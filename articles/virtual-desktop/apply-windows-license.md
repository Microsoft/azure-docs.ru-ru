---
title: Применение лицензии Windows к виртуальным машинам узла сеансов — Azure
description: Описание применения лицензии Windows для виртуальных машин виртуальных рабочих столов Windows.
author: ChristianMontoya
ms.topic: how-to
ms.date: 08/14/2019
ms.author: chrimo
ms.openlocfilehash: 5f3749be36f5f035e49fcb862f92180e4902101f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88010146"
---
# <a name="apply-windows-license-to-session-host-virtual-machines"></a>Применение лицензии Windows к виртуальным машинам узла сеансов

Клиенты, имеющие право на выполнение рабочих нагрузок виртуальных рабочих столов Windows, могут применить лицензию Windows к виртуальным машинам узла сеансов и запускать их без оплаты за другую лицензию. Дополнительные сведения см. в разделе [цены на виртуальные рабочие столы Windows](https://azure.microsoft.com/pricing/details/virtual-desktop/).

## <a name="ways-to-use-your-windows-virtual-desktop-license"></a>Способы использования лицензии на виртуальные рабочие столы Windows
Лицензирование виртуальных рабочих столов Windows позволяет применять лицензии к любой виртуальной машине Windows или Windows Server, зарегистрированной как узел сеанса в пуле узлов и принимающей подключения пользователей. Эта лицензия не распространяется на виртуальные машины, работающие в качестве серверов файловых ресурсов, контроллеров домена и т. д.

Существует несколько способов использования лицензии на виртуальные рабочие столы Windows.
- Вы можете создать пул узлов и виртуальные машины узла сеансов с помощью [предложения Azure Marketplace](./create-host-pools-azure-marketplace.md). Виртуальные машины, созданные таким образом, автоматически применяют лицензию.
- Пул узлов и виртуальные машины узла сеансов можно создать с помощью [шаблона Azure Resource Manager GitHub](./virtual-desktop-fall-2019/create-host-pools-arm-template.md). Виртуальные машины, созданные таким образом, автоматически применяют лицензию.
- Вы можете применить лицензию к существующей виртуальной машине узла сеансов. Для этого сначала следуйте инструкциям в разделе [Создание пула узлов с помощью PowerShell](./create-host-pools-powershell.md) для создания пула узлов и связанных виртуальных машин, а затем вернитесь к этой статье, чтобы узнать, как применить эту лицензию.

## <a name="apply-a-windows-license-to-a-session-host-vm"></a>Применение лицензии Windows к виртуальной машине узла сеансов
Убедитесь, что у вас [установлена и настроена последняя версия Azure PowerShell](/powershell/azure/). Выполните следующий командлет PowerShell, чтобы применить лицензию Windows:

```powershell
$vm = Get-AzVM -ResourceGroup <resourceGroupName> -Name <vmName>
$vm.LicenseType = "Windows_Client"
Update-AzVM -ResourceGroupName <resourceGroupName> -VM $vm
```

## <a name="verify-your-session-host-vm-is-utilizing-the-licensing-benefit"></a>Проверка того, что виртуальная машина узла сеансов использует преимущества лицензирования
После развертывания виртуальной машины выполните командлет, чтобы проверить тип лицензии:
```powershell
Get-AzVM -ResourceGroupName <resourceGroupName> -Name <vmName>
```

Виртуальная машина узла сеансов с примененной лицензией Windows отобразит примерно следующее:

```powershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              : Windows_Client
```

Виртуальные машины без примененной лицензии Windows будут показывать примерно следующее:

```powershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              :
```

Выполните следующий командлет, чтобы просмотреть список всех виртуальных машин узла сеансов, к которым назначена лицензия Windows в вашей подписке Azure:

```powershell
$vms = Get-AzVM
$vms | Where-Object {$_.LicenseType -like "Windows_Client"} | Select-Object ResourceGroupName, Name, LicenseType
```
