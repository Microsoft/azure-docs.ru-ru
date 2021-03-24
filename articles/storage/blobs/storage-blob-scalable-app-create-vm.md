---
title: Создание виртуальной машины и учетной записи хранения для масштабируемого приложения в Azure
description: Узнайте, как развернуть виртуальную машину для запуска масштабируемого приложения, использующего хранилище BLOB-объектов Azure.
author: roygara
ms.service: storage
ms.topic: tutorial
ms.date: 02/20/2018
ms.author: rogarana
ms.subservice: blobs
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 4b8c52b03cb6dec6096565e9eac26b7b2c4a30e4
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89073273"
---
# <a name="create-a-virtual-machine-and-storage-account-for-a-scalable-application"></a>Создание виртуальной машины и учетной записи хранения для масштабируемого приложения

Это руководство представляет первую часть цикла. В этом руководстве рассматривается развертывание приложения, которое передает большие объемы случайных данных в учетную запись хранения Azure и загружает их из нее. По завершении этого руководства на вашей виртуальной машине будет выполняться консольное приложение, которое передает большие объемы данных в учетную запись хранения и скачивает их из нее.

В первой части цикла вы узнаете, как выполнять такие задачи:

> [!div class="checklist"]
> * Создание учетной записи хранения
> * Создание виртуальной машины
> * Настройка расширения пользовательских скриптов

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Чтобы установить и использовать PowerShell локально для работы с этим руководством, вам понадобится модуль Azure PowerShell 0.7 или более поздней версии. Чтобы узнать версию, выполните команду `Get-Module -ListAvailable Az`. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-Az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов Azure с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими.

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location EastUS
```

## <a name="create-a-storage-account"></a>Создание учетной записи хранения
 
В этом примере 50 больших файлов отправляются в контейнер больших двоичных объектов в учетной записи хранения Azure. Учетная запись хранения предоставляет уникальное пространство имен для хранения ваших объектов данных в службе хранилища Azure и доступа к ним. Создайте учетную запись хранения в созданной вами группе ресурсов с помощью команды [New-AzStorageAccount](/powershell/module/az.Storage/New-azStorageAccount).

В следующей команде замените `<blob_storage_account>` глобально уникальным именем учетной записи хранения больших двоичных объектов везде, где встречается этот заполнитель.

```powershell-interactive
$storageAccount = New-AzStorageAccount -ResourceGroupName myResourceGroup `
  -Name "<blob_storage_account>" `
  -Location EastUS `
  -SkuName Standard_LRS `
  -Kind Storage `
```

## <a name="create-a-virtual-machine"></a>Создание виртуальной машины

Создайте конфигурацию виртуальной машины. Эта конфигурация содержит параметры, которые используются при развертывании виртуальной машины, в том числе образ виртуальной машины, ее размер и настройки аутентификации. При выполнении этого шага будут запрошены учетные данные. В качестве вводимых значений указываются имя пользователя и пароль для виртуальной машины.

Создайте виртуальную машину с помощью команды [New-AzVM](/powershell/module/az.compute/new-azvm).

```azurepowershell-interactive
# Variables for common values
$resourceGroup = "myResourceGroup"
$location = "eastus"
$vmName = "myVM"

# Create user object
$cred = Get-Credential -Message "Enter a username and password for the virtual machine."

# Create a subnet configuration
$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name mySubnet -AddressPrefix 192.168.1.0/24

# Create a virtual network
$vnet = New-AzVirtualNetwork -ResourceGroupName $resourceGroup -Location $location `
  -Name MYvNET -AddressPrefix 192.168.0.0/16 -Subnet $subnetConfig

# Create a public IP address and specify a DNS name
$pip = New-AzPublicIpAddress -ResourceGroupName $resourceGroup -Location $location `
  -Name "mypublicdns$(Get-Random)" -AllocationMethod Static -IdleTimeoutInMinutes 4

# Create a virtual network card and associate with public IP address
$nic = New-AzNetworkInterface -Name myNic -ResourceGroupName $resourceGroup -Location $location `
  -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id

# Create a virtual machine configuration
$vmConfig = New-AzVMConfig -VMName myVM -VMSize Standard_DS14_v2 | `
    Set-AzVMOperatingSystem -Windows -ComputerName myVM -Credential $cred | `
    Set-AzVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer `
    -Skus 2016-Datacenter -Version latest | Add-AzVMNetworkInterface -Id $nic.Id

# Create a virtual machine
New-AzVM -ResourceGroupName $resourceGroup -Location $location -VM $vmConfig

Write-host "Your public IP address is $($pip.IpAddress)"
```

## <a name="deploy-configuration"></a>Развертывание конфигурации

В этом руководстве приведены необходимые компоненты, которые нужно установить на виртуальной машине. Расширение пользовательских скриптов используется для запуска скрипта PowerShell, который выполняет следующие задачи:

> [!div class="checklist"]
> * установка .NET Core 2.0;
> * установка Chocolatey;
> * установка GIT;
> * клонирование репозитория;
> * восстановление пакетов NuGet;
> * создание 50 файлов по 1 ГБ со случайными данными.

Запустите следующий командлет, чтобы завершить конфигурацию виртуальной машины. На выполнение этого шага может потребоваться от 5 до 15 минут.

```azurepowershell-interactive
# Start a CustomScript extension to use a simple PowerShell script to install .NET core, dependencies, and pre-create the files to upload.
Set-AzVMCustomScriptExtension -ResourceGroupName myResourceGroup `
    -VMName myVM `
    -Location EastUS `
    -FileUri https://raw.githubusercontent.com/azure-samples/storage-dotnet-perf-scale-app/master/setup_env.ps1 `
    -Run 'setup_env.ps1' `
    -Name DemoScriptExtension
```

## <a name="next-steps"></a>Дальнейшие действия

В первой части этой серии руководств вы узнали, как создать учетную запись хранения, развернуть виртуальную машину и установить на ней необходимые компоненты, в частности как выполнять следующие действия:

> [!div class="checklist"]
> * Создание учетной записи хранения
> * Создание виртуальной машины
> * Настройка расширения пользовательских скриптов

Перейдите к изучению второй части, чтобы узнать, как передавать большие объемы данных в учетную запись хранения с использованием экспоненциального повторения и параллелизма.

> [!div class="nextstepaction"]
> [Передача больших объемов случайных данных в параллельном режиме в службу хранилища Azure](storage-blob-scalable-app-upload-files.md)
