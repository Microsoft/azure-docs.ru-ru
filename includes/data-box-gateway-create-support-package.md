---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/15/2020
ms.author: alkohli
ms.openlocfilehash: f002fec531a0355e803a1545990fc0b13b535742
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96582750"
---
При возникновении проблем с устройствами можно создать пакет поддержки из системных журналов. Служба поддержки Майкрософт использует этот пакет для устранения неполадок. Чтобы создать пакет поддержки, выполните следующие действия.

1. [Подключитесь к интерфейсу PowerShell устройства](#connect-to-the-powershell-interface).
2. Используйте `Get-HcsNodeSupportPackage` команду, чтобы создать пакет поддержки. Использование командлета выглядит следующим образом:

    ```powershell
    Get-HcsNodeSupportPackage [-Path] <string> [-Zip] [-ZipFileName <string>] [-Include {None | RegistryKeys | EtwLogs
            | PeriodicEtwLogs | LogFiles | DumpLog | Platform | FullDumps | MiniDumps | ClusterManagementLog | ClusterLog |
            UpdateLogs | CbsLogs | StorageCmdlets | ClusterCmdlets | ConfigurationCmdlets | KernelDump | RollbackLogs |
            Symbols | NetworkCmdlets | NetworkCmds | Fltmc | ClusterStorageLogs | UTElement | UTFlag | SmbWmiProvider |
            TimeCmds | LocalUILogs | ClusterHealthLogs | BcdeditCommand | BitLockerCommand | DirStats | ComputeRolesLogs |
            ComputeCmdlets | DeviceGuard | Manifests | MeasuredBootLogs | Stats | PeriodicStatLogs | MigrationLogs |
            RollbackSupportPackage | ArchivedLogs | Default}] [-MinimumTimestamp <datetime>] [-MaximumTimestamp <datetime>]
            [-IncludeArchived] [-IncludePeriodicStats] [-Credential <pscredential>]  [<CommonParameters>]
    ```

    Командлет собирает журналы с устройства и копирует эти журналы в указанную сеть или в локальную папку.

    Используются следующие параметры.

    - `-Path` — Укажите сеть или локальный путь для копирования пакета поддержки. (обязательно)
    - `-Credential` — Укажите учетные данные для доступа к защищенному пути.
    - `-Zip` — Укажите, чтобы создать ZIP-файл.
    - `-Include` — Укажите, чтобы включить компоненты, включаемые в пакет поддержки. Если значение не указано, `Default` предполагается.
    - `-IncludeArchived` — Укажите, чтобы включить архивированные журналы в пакет поддержки.
    - `-IncludePeriodicStats` — Укажите, чтобы включить периодические журналы статистики в пакет поддержки.

    
