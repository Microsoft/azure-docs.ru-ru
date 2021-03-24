---
title: Подключение к консоли виртуальной машины на устройстве с графическим процессором Azure Stack ребра Pro
description: В этой статье описывается подключение к консоли виртуальной машины на виртуальной машине, работающей на устройстве Azure Stack с помощью GPU Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 03/22/2021
ms.author: alkohli
ms.openlocfilehash: 68cf157a512e9b1f6caee4734869c5bb4b836e2f
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104962690"
---
# <a name="connect-to-a-virtual-machine-console-on-an-azure-stack-edge-pro-gpu-device"></a>Подключение к консоли виртуальной машины на устройстве с ПРОЦЕССОРом Azure Stack ребра Pro

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

Решение Azure Stack для GPU с пограничными Pro выполняет рабочие нагрузки, не являющиеся контейнерами, через виртуальные машины. В этой статье описывается подключение к консоли виртуальной машины, развернутой на устройстве. 

Консоль виртуальных машин позволяет получать доступ к виртуальным машинам с помощью клавиатуры, мыши и функций экрана, используя широко доступные средства удаленного рабочего стола. Вы можете получить доступ к консоли и устранить неполадки, возникшие при развертывании виртуальной машины на устройстве. Вы можете подключиться к консоли виртуальной машины, даже если не удалось выполнить инициализацию виртуальной машины.


## <a name="workflow"></a>Рабочий процесс

Рабочий процесс высокого уровня включает в себя следующие шаги:

1. Подключитесь к интерфейсу PowerShell на устройстве.
1. Включите консольный доступ к виртуальной машине.
1. Подключитесь к виртуальной машине с помощью протокола удаленного рабочего стола (RDP).
1. Отозвать доступ консоли к виртуальной машине.

## <a name="prerequisites"></a>Предварительные условия

Перед началом убедитесь, что выполнены следующие предварительные требования:

#### <a name="for-your-device"></a>Для вашего устройства

У вас должен быть доступ к активному устройству GPU Azure Stack ребра. На устройстве должна быть развернута одна или несколько виртуальных машин. Виртуальные машины можно развертывать с помощью Azure PowerShell, с помощью шаблонов или с помощью портал Azure.

#### <a name="for-client-accessing-the-device"></a>Для клиента, обращающегося к устройству

Убедитесь, что у вас есть доступ к клиентской системе, которая:

- Может получить доступ к интерфейсу PowerShell устройства. Клиент работает под управлением [поддерживаемой операционной системы](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device).
- На клиенте работает PowerShell 7,0 или более поздней версии. Эта версия PowerShell работает для клиентов Windows, Mac и Linux. См. инструкции по [установке PowerShell 7,0](/powershell/scripting/whats-new/what-s-new-in-powershell-70?view=powershell-7.1&preserve-view=true).
- Имеет возможности удаленного рабочего стола. В зависимости от того, используете ли вы Windows, macOS или Linux, следует установить один из этих [клиентов удаленного рабочего стола](/windows-server/remote/remote-desktop-services/clients/remote-desktop-clients). Эта статья содержит инструкции по [Windows удаленный рабочий стол](/windows-server/remote/remote-desktop-services/clients/windowsdesktop#install-the-client) и [фрирдп](https://www.freerdp.com/). <!--Which version of FreeRDP to use?-->


## <a name="connect-to-vm-console"></a>Подключение к консоли виртуальной машины

Выполните следующие действия, чтобы подключиться к консоли виртуальной машины на устройстве.

### <a name="connect-to-the-powershell-interface-on-your-device"></a>Подключение к интерфейсу PowerShell на устройстве

Первый шаг — [Подключение к интерфейсу PowerShell](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface) устройства. 

### <a name="enable-console-access-to-the-vm"></a>Включение доступа консоли к виртуальной машине

1.  В интерфейсе PowerShell выполните следующую команду, чтобы разрешить доступ к консоли виртуальной машины.

    ```powershell
    Grant-HcsVMConnectAccess -ResourceGroupName <VM resource group> -VirtualMachineName <VM name>
    ```
2. В примере выходных данных запишите идентификатор виртуальной машины. Это потребуется для последующего этапа.

    ```powershell
    [10.100.10.10]: PS>Grant-HcsVMConnectAccess -ResourceGroupName mywindowsvm1rg -VirtualMachineName mywindowsvm1
        
    VirtualMachineId       : 81462e0a-decb-4cd4-96e9-057094040063
    VirtualMachineHostName : 3V78B03
    ResourceGroupName      : mywindowsvm1rg
    VirtualMachineName     : mywindowsvm1
    Id                     : 81462e0a-decb-4cd4-96e9-057094040063
    [10.100.10.10]: PS>
    ```

### <a name="connect-to-the-vm"></a>Подключение к виртуальной машине

Теперь можно использовать клиент удаленный рабочий стол для подключения к консоли виртуальной машины.

#### <a name="use-windows-remote-desktop"></a>Использование удаленный рабочий стол Windows

1. Создайте новый текстовый файл и введите следующий текст.

    ```
    pcb:s:<VM ID from PowerShell>;EnhancedMode=0
    full address:s:<IP address of the device>   
    server port:i:2179
    username:s:EdgeARMUser
    negotiate security layer:i:0
    ```
1. Сохраните файл как **. RDP* в клиентской системе. Этот профиль будет использоваться для подключения к виртуальной машине.
1. Дважды щелкните профиль, чтобы подключиться к виртуальной машине. Укажите следующие учетные данные:

    - **Имя пользователя**: Войдите как еджеармусер.
    - **Пароль**: укажите локальный пароль Azure Resource Manager для устройства. Если вы забыли пароль, [сбросьте Azure Resource Manager пароль с помощью портал Azure](azure-stack-edge-gpu-set-azure-resource-manager-password.md#reset-password-via-the-azure-portal). 

#### <a name="use-freerdp"></a>Использование Фрирдп

При использовании Фрирдп на клиенте Linux выполните следующую команду: 

```powershell
./wfreerdp /u:EdgeARMUser /vmconnect:<VM ID from PowerShell> /v:<IP address of the device>
```

## <a name="revoke-vm-console-access"></a>Отозвать доступ к консоли виртуальной машины

Чтобы отозвать доступ к консоли виртуальной машины, вернитесь к интерфейсу PowerShell устройства. Выполните следующую команду:

```
Revoke-HcsVMConnectAccess -ResourceGroupName <VM resource group> -VirtualMachineName <VM name>
```
Пример выходных данных:

```powershell
[10.100.10.10]: PS>Revoke-HcsVMConnectAccess -ResourceGroupName mywindowsvm1rg -VirtualMachineName mywindowsvm1

VirtualMachineId       : 81462e0a-decb-4cd4-96e9-057094040063
VirtualMachineHostName : 3V78B03
ResourceGroupName      : mywindowsvm1rg
VirtualMachineName     : mywindowsvm1
Id                     : 81462e0a-decb-4cd4-96e9-057094040063

[10.100.10.10]: PS>
```
> [!NOTE] 
> После завершения работы с консолью виртуальной машины рекомендуется отозвать доступ или закрыть окно PowerShell, чтобы выйти из сеанса. 

## <a name="next-steps"></a>Дальнейшие действия

- Устранение неполадок [Azure Stack пограничных Pro](azure-stack-edge-gpu-troubleshoot.md) в портал Azure.