---
title: Создание образов виртуальных машин для устройства GPU Azure Stack Edge Pro
description: Узнайте, как создавать образы виртуальных машин Linux и Windows для использования с устройством GPU Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 02/22/2021
ms.author: alkohli
ms.openlocfilehash: ee0587063c0fac67068869c58471ada58354fab7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102437678"
---
# <a name="create-custom-vm-images-for-your-azure-stack-edge-pro-device"></a>Создание пользовательских образов виртуальных машин для устройства Azure Stack Edge Pro

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

Для развертывания виртуальных машин на устройстве Azure Stack Edge Pro требуется возможность создавать пользовательские образы виртуальных машин, которые можно использовать для создания виртуальных машин. В этой статье описываются действия, которые необходимо выполнить для создания пользовательских образов виртуальных машин Linux или Windows, которые можно использовать для развертывания виртуальных машин на устройстве Azure Stack пограничной Pro.

## <a name="vm-image-workflow"></a>Рабочий процесс создания образа виртуальной машины

Рабочий процесс предусматривает создание виртуальной машины в Azure, ее настройку и подготовку к использованию, а также скачивание виртуального жесткого диска, соответствующего этой виртуальной машине. Этот обобщенный виртуальный жесткий диск отправляется в Azure Stack пограничной Pro. На основе этого виртуального жесткого диска создается управляемый диск. Образ создается из управляемого диска. И, наконец, виртуальные машины создаются из этого образа.

Дополнительные сведения см. в статье [Развертывание виртуальных машин на устройстве GPU Azure Stack Edge Pro с помощью Azure PowerShell](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md).


## <a name="create-a-windows-custom-vm-image"></a>Создание пользовательского образа виртуальной машины Windows

Чтобы создать образ виртуальной машины Windows, сделайте следующее:

1. Создайте виртуальную машину Windows. Дополнительные сведения см. в статье [Руководство. Создание виртуальных машин Windows и управление ими с помощью Azure PowerShell](../virtual-machines/windows/tutorial-manage-vm.md).

2. Скачайте существующий диск ОС.

    - Выполните инструкции, приведенные в статье [Скачивание виртуального жесткого диска Windows из Azure](../virtual-machines/windows/download-vhd.md).

    - Используйте следующую команду `sysprep` вместо той, которая указана в предыдущей процедуре:
    
        `c:\windows\system32\sysprep\sysprep.exe /oobe /generalize /shutdown /mode:vm`
   
       См. статью [Обзор Sysprep (подготовка системы)](/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview).

Используйте этот виртуальный жесткий диск, чтобы создать и развернуть виртуальную машину на устройстве Azure Stack Edge Pro.

## <a name="create-a-linux-custom-vm-image"></a>Создание пользовательского образа виртуальной машины Linux

Чтобы создать образ виртуальной машины Linux, сделайте следующее:

1. Создание виртуальной машины Linux. Дополнительные сведения см. в статье [Руководство. Создание виртуальных машин Linux и управление ими с помощью Azure CLI](../virtual-machines/linux/tutorial-manage-vm.md).

1. Отзовите виртуальную машину. С помощью агента виртуальной машины Azure удалите файлы и данные конкретной виртуальной машины. На исходной виртуальной машине Linux выполните команду `waagent` с параметром `-deprovision+user`. Дополнительные сведения см. в статье [Что такое агент Linux для Azure и как его использовать](../virtual-machines/extensions/agent-linux.md).

    1. Подключитесь к виртуальной машине Linux c помощью клиента SSH.
    2. В окне сеанса SSH введите следующую команду.
       
        ```bash
        sudo waagent -deprovision+user
        ```
       > [!NOTE]
       > Эту команду следует выполнять только на той виртуальной машине, образ которой требуется записать. Выполнение этой команды не гарантирует, что из образа будет удалена вся конфиденциальная информация и что он будет готов к повторному распространению. Параметр `+user` также удаляет последнюю подготовленную учетную запись пользователя. Чтобы сохранить учетные данные пользователя на виртуальной машине, используйте только параметр `-deprovision`.
     
    3. Для продолжения введите **y**. Чтобы предотвратить появление запроса на подтверждение, добавьте параметр `-force` .
    4. После выполнения команды введите **выйти**, чтобы закрыть SSH-клиент.  На этом этапе виртуальная машина будет по-прежнему работать.


1. [Скачайте существующий диск ОС.](../virtual-machines/linux/download-vhd.md)

Используйте этот виртуальный жесткий диск, чтобы создать и развернуть виртуальную машину на устройстве Azure Stack Edge Pro. Для создания пользовательских образов Linux можно использовать следующие два образа Azure Marketplace:

|Имя элемента.  |Описание  |Издатель  |
|---------|---------|---------|
|[Сервер Ubuntu](https://azuremarketplace.microsoft.com/marketplace/apps/canonical.ubuntuserver) |Ubuntu Server является самой популярной в мире системой Linux для облачных сред.|Canonical|
|[Debian 8 "Jessie"](https://azuremarketplace.microsoft.com/marketplace/apps/credativ.debian) |Debian GNU/Linux является одним из наиболее распространенных дистрибутивов Linux.     |credativ|

Полный список рабочих образов из Azure Marketplace (пока не протестированы) см. в статье [Элементы Azure Marketplace, доступные для Azure Stack Hub](/azure-stack/operator/azure-stack-marketplace-azure-items?view=azs-1910&preserve-view=true).


## <a name="next-steps"></a>Дальнейшие действия

[Развертывание виртуальных машин на устройстве Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md)