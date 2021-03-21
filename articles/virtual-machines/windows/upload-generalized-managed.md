---
title: Создание виртуальной машины на основе отправленного обобщенного образа VHD
description: Отправка универсального диска VHD в Azure и создание виртуальных машин с его помощью в модели развертывания Resource Manager.
author: cynthn
ms.service: virtual-machines
ms.subservice: disks
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 12/12/2019
ms.author: cynthn
ms.openlocfilehash: 793bc5518664761a2a9b0cfd46e616d2fb72c3e6
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102562101"
---
# <a name="upload-a-generalized-vhd-and-use-it-to-create-new-vms-in-azure"></a>Отправка универсального диска VHD и создание виртуальных машин с его помощью в Azure

В этой статье описывается, как с помощью PowerShell отправить VHD универсальной виртуальной машины в Azure, создать образ на основе VHD и виртуальную машину на основе этого образа. Вы можете отправить диск VHD, экспортированный из локального инструмента виртуализации или из другого облака. При использовании [Управляемых дисков](../managed-disks-overview.md) для новой виртуальной машины упрощается управление виртуальными машинами и обеспечивается более высокий уровень доступности виртуальной машины, размещенной в группе доступности. 

Если необходим пример сценария, см. статью [Пример сценария для отправки VHD в Azure и создания виртуальной машины](/previous-versions/azure/virtual-machines/scripts/virtual-machines-windows-powershell-upload-generalized-script).

## <a name="before-you-begin"></a>Перед началом

- Прежде чем передавать в Azure какой-либо виртуальный жесткий диск, следует выполнить инструкции из статьи [Подготовка диска VHD или VHDX для Windows к отправке в Azure](prepare-for-upload-vhd-image.md).
- Прежде чем начать миграцию на [Управляемые диски](../managed-disks-overview.md), внимательно изучите [этот раздел](on-prem-to-azure.md#plan-for-the-migration-to-managed-disks).

 
## <a name="generalize-the-source-vm-by-using-sysprep"></a>Подготовка исходной виртуальной машины к использованию с помощью Sysprep

Если вы еще не сделали этого, перед отправкой виртуального жесткого диска в Azure необходимо подготовить виртуальную машину с помощью Sysprep. Помимо прочих действий Sysprep удаляет все сведения о вашей учетной записи и подготавливает машину к использованию в качестве образа. Дополнительные сведения о Sysprep приведены в [обзоре Sysprep](/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview).

Убедитесь, что Sysprep поддерживает роли сервера, запущенные на компьютере. Дополнительные сведения см. в разделе [Поддержка ролей сервера в Sysprep](/windows-hardware/manufacture/desktop/sysprep-support-for-server-roles).

> [!IMPORTANT]
> Если вы хотите запустить Sysprep перед отправкой виртуального жесткого диска в Azure в первый раз, [подготовьте виртуальную машину](prepare-for-upload-vhd-image.md). 
> 
> 

1. Выполните вход на виртуальную машину Windows.
1. Откройте окно командной строки с правами администратора. 
1. Удалите каталог пансер (К:\виндовс\пансер).
1. Измените каталог на %windir%\system32\sysprep и запустите файл `sysprep.exe`.
1. В диалоговом окне **Программа подготовки системы** выберите **Переход в окно приветствия системы (OOBE)** и убедитесь, что установлен флажок **Подготовка к использованию**.
1. В разделе **Параметры завершения работы** выберите **Завершение работы**.
1. Щелкните **ОК**.
   
    ![Запуск Sysprep](./media/upload-generalized-managed/sysprepgeneral.png)
1. После выполнения всех необходимых действий Sysprep завершает работу виртуальной машины. Не перезапускайте виртуальную машину.


## <a name="upload-the-vhd"></a>Отправка виртуального жесткого диска 

Теперь можно отправить VHD непосредственно в управляемый диск. Инструкции см. в статье [Отправка VHD в Azure с помощью Azure PowerShell](disks-upload-vhd-to-managed-disk-powershell.md).



После передачи виртуального жесткого диска на управляемый диск необходимо использовать командлет [Get-AzDisk](/powershell/module/az.compute/get-azdisk) для получения управляемого диска.

```azurepowershell-interactive
$disk = Get-AzDisk -ResourceGroupName 'myResourceGroup' -DiskName 'myDiskName'
```

## <a name="create-the-image"></a>Создание образа
Создайте управляемый образ с помощью универсального управляемого диска ОС. Подставьте собственные значения.

Сначала задайте несколько переменных:

```powershell
$location = 'East US'
$imageName = 'myImage'
$rgName = 'myResourceGroup'
```

Создайте образ с помощью управляемого диска.

```azurepowershell-interactive
$imageConfig = New-AzImageConfig `
   -Location $location
$imageConfig = Set-AzImageOsDisk `
   -Image $imageConfig `
   -OsState Generalized `
   -OsType Windows `
   -ManagedDiskId $disk.Id
```

Создайте образ.

```azurepowershell-interactive
$image = New-AzImage `
   -ImageName $imageName `
   -ResourceGroupName $rgName `
   -Image $imageConfig
```

## <a name="create-the-vm"></a>Создание виртуальной машины

Теперь, когда образ готов, из него можно создать одну или несколько виртуальных машин. В этом примере создается виртуальная машина *myVM* из образа *myImage* в группе ресурсов *myResourceGroup*.


```powershell
New-AzVm `
    -ResourceGroupName $rgName `
    -Name "myVM" `
    -Image $image.Id `
    -Location $location `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNSG" `
    -PublicIpAddressName "myPIP" `
    -OpenPorts 3389
```


## <a name="next-steps"></a>Дальнейшие действия

Войдите на свою новую виртуальную машину. Дополнительные сведения см. в статье [Как подключиться к виртуальной машине Azure под управлением Windows и войти на нее](connect-logon.md).