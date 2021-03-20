---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 03/05/2019
ms.author: alkohli
ms.openlocfilehash: b657ee32e76dd90671f7e91337ced01b925889a1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "67185746"
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

    
