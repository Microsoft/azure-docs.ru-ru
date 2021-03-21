---
title: Как сопоставлять диски Azure с гостевыми дисками виртуальной машины Windows
description: Как определить диски Azure, ундерлай гостевые диски виртуальной машины Windows.
author: timbasham
ms.service: virtual-machines
ms.subservice: disks
ms.collection: windows
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 11/17/2020
ms.author: tibasham
ms.openlocfilehash: 7a4fad066af37217eb42060d5fc5a7ef716770c6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102560996"
---
# <a name="how-to-map-azure-disks-to-windows-vm-guest-disks"></a>Как сопоставлять диски Azure с гостевыми дисками виртуальной машины Windows

Возможно, вам потребуется определить диски Azure, которые будут выполнять резервное копирование гостевых дисков виртуальной машины. В некоторых случаях можно сравнить размер диска или тома с размером подключенных дисков Azure. В сценариях, где к виртуальной машине подключено несколько дисков Azure одного размера, необходимо использовать логический номер устройства (LUN) дисков данных. 

## <a name="what-is-a-lun"></a>Что такое LUN?

Логический номер устройства (LUN) — это число, которое используется для обнаружения определенного устройства хранения. Каждому устройству хранения назначается уникальный числовой идентификатор, начинающийся с нуля. Полный путь к устройству представлен номером шины, ИДЕНТИФИКАТОРом целевого объекта и логическим номером устройства (LUN). 

Например: ***номер шины 0, идентификатор целевого объекта 0, LUN 3***

Для нашего упражнения необходимо использовать только LUN.

## <a name="finding-the-lun"></a>Поиск LUN

Существует два способа поиска LUN, который, по вашему выбору, будет зависеть от того, используется [дисковые пространства](/windows-server/storage/storage-spaces/overview) или нет.

### <a name="disk-management"></a>Управление дисками

Если пулы носителей не используются, можно найти LUN с помощью [оснастки "Управление дисками](/windows-server/storage/disk-management/overview-of-disk-management) ".

1. Подключитесь к виртуальной машине и откройте оснастку "Управление дисками". Щелкните правой кнопкой мыши кнопку Пуск и выберите пункт "Управление дисками". Можно также ввести `diskmgmt.msc` в поле начать поиск.
1. В нижней области щелкните правой кнопкой мыши любой из дисков и выберите "Свойства".
1. LUN будет указан в свойстве "Location" на вкладке "Общие".

### <a name="storage-pools"></a>Пулы хранения

1. Подключитесь к виртуальной машине и откройте диспетчер сервера
1. Выберите "службы файлов и хранилища", "тома", "пулы носителей".
1. В правом нижнем углу диспетчер сервера появится раздел "физические диски". Ниже перечислены диски, составляющие пул носителей, а также номера LUN для каждого диска.

## <a name="finding-the-lun-for-the-azure-disks"></a>Поиск LUN для дисков Azure

Номер LUN для диска Azure можно узнать с помощью портал Azure, Azure CLI или Azure PowerShell

### <a name="finding-an-azure-disks-lun-in-the-azure-portal"></a>Поиск LUN диска Azure в портал Azure

1. В портал Azure выберите "виртуальные машины", чтобы отобразить список виртуальных машин.
1. Выберите виртуальную машину.
1. Выберите "диски"
1. Выберите диск данных из списка подключенных дисков.
1. LUN диска будет отображаться в области сведений о диске. Показанные здесь LUN сопоставляются с LUN, которые были просмотрены в гостевой системе с помощью диспетчер устройств или диспетчер сервера.

### <a name="finding-an-azure-disks-lun-using-azure-cli-or-azure-powershell"></a>Поиск LUN диска Azure с помощью Azure CLI или Azure PowerShell

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
```azurecli-interactive
az vm show -g myResourceGroup -n myVM --query "storageProfile.dataDisks"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)
```azurepowershell-interactive
$vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM
$vm.StorageProfile.DataDisks | ft
```
---