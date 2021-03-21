---
title: Использование сценариев PowerShell AzureRM для управления устройствами StorSimple
description: Узнайте, как использовать Azure Resource Manager сценарии на основе пакета SDK для управления устройством серии StorSimple 8000.
author: alkohli
ms.service: storsimple
ms.topic: how-to
ms.date: 10/03/2017
ms.author: alkohli
ms.openlocfilehash: e3339f883d177366a64187e0c090752527802c8d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94968780"
---
# <a name="use-azure-resource-manager-sdk-based-scripts-to-manage-storsimple-devices"></a>Использование сценариев Azure Resource Manager для управления устройствами StorSimple на основе пакета SDK

В этой статье находится инструкция по использованию сценариев Azure Resource Manager на основе пакета SDK для управления устройством StorSimple серии 8000. Пример сценария также включен для выполнения шагов по настройке среды для запуска этих сценариев.

Эта статья относится только к устройствам StorSimple серии 8000 на портале Azure.

## <a name="sample-scripts"></a>Примеры сценариев

Для автоматизации различных заданий StorSimple доступны приведенные ниже образцы сценариев.

#### <a name="table-of-azure-resource-manager-sdk-based-sample-scripts"></a>Таблица сценариев на основе пакета SDK для Azure Resource Manager

| Сценарии Azure Resource Manager                    | Описание                                                                                                                                                                                                       |
|--------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Authorize-ServiceEncryptionRollover.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Authorize-ServiceEncryptionRollover.ps1)          | Этот сценарий позволяет авторизовать устройство StorSimple для изменения ключей шифрования данных службы.                                                                                                           |
| [Create-StorSimpleCloudAppliance.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Create-StorSimpleCloudAppliance.ps1)              | В этом сценарии создается облачное устройство StorSimple 8010 или 8020. Затем можно настроить и зарегистрировать облачное устройство в службе диспетчера данных StorSimple.                                                       |
| [CreateOrUpdate-Volume.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/CreateOrUpdate-Volume.ps1)                        | В этом сценарии создаются или изменяются тома StorSimple.                                                                                                                                                             |
| [Get-DeviceBackup.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceBackup.ps1)                             | В этом сценарии перечисляются все резервные копии для устройства, зарегистрированного в службе диспетчера устройств StorSimple.                                                                                                          |
| [Get-DeviceBackupPolicy.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceBackupPolicy.ps1)                       | С помощью этого сценария можно получить все политики резервного копирования для устройства StorSimple.                                                                                                                                                 |
| [Get-DeviceJobs.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceJobs.ps1)                               | Этот сценарий получает все задания StorSimple, запущенные в службе диспетчера устройств StorSimple.                                                                                                                     |
| [Get-DeviceUpdateAvailability.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceUpdateAvailability.ps1)                 | В этом сценарии проверяется сервер обновлений и приходят оповещения о доступных обновлениях для установки на устройство StorSimple.                                                                                          |
| [Install-DeviceUpdate.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Install-DeviceUpdate.ps1)                         | В этом сценарии устанавливаются доступные обновления на устройство StorSimple.                                                                                                                                           |
| [Manage-CloudSnapshots.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Manage-CloudSnapshots.ps1)                        | В этом сценарии создаются и удаляются облачные моментальные снимки вручную (удаляются только снимки, хранящиеся дольше указанной продолжительности хранения).                                                                                                   |
| [Monitor-Backups.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Monitor-Backups.ps1)                              | В этом сценарии PowerShell Runbook службы автоматизации Azure отображается статус всех заданий резервного копирования.                                                                                                              |
| [Remove-DeviceBackup.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Remove-DeviceBackup.ps1)                          | В этом сценарии удаляется объект резервного копирования.                                                                                                                                                           |
| [Start-DeviceBackupJob.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Start-DeviceBackupJob.ps1)                        | В этом сценарии резервное копирование на устройстве StorSimple запускается вручную.                                                                                                                                       |
| [Update-CloudApplianceServiceEncryptionKey.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Update-CloudApplianceServiceEncryptionKey.ps1)    | В этом сценарии обновляется ключ шифрования данных службы для всех зарегистрированных устройств StorSimple серии 8010 или 8020 облака в службе диспетчера устройств StorSimple.                                     |
| [Verify-BackupScheduleAndBackup.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Verify-BackupScheduleAndBackup.ps1)               | В этом сценарии выделяются отсутствующие резервные копии после анализа всех расписаний, связанных с политиками резервного копирования. В нем также проверяется каталог со списком доступных резервных копий.             |




## <a name="configure-and-run-a-sample-script"></a>Настройка и запуск образца сценария

В этом разделе приведен пример сценария и подробно описаны различные шаги, необходимые для его запуска.

### <a name="prerequisites"></a>Предварительные условия

Перед началом работы убедитесь, что у вас есть следующие компоненты:

*   Azure PowerShell установлен. Чтобы установить модули Azure PowerShell, сделайте следующее:
    * В среде Windows следуйте инструкциям в статье [Установка и настройка Azure PowerShell](/powershell/azure/install-az-ps). Если необходимо, вы можете установить Azure PowerShell на узле Windows Server для StorSimple.
    * В среде Linux или MacOS следуйте инструкциям в статье [Установка и настройка Azure PowerShell в macOS и Linux](/powershell/azure/install-az-ps).

Дополнительные сведения об использовании Azure PowerShell см. в статье [Начало работы с Azure PowerShell](/powershell/azure/get-started-azureps).

### <a name="run-azure-powershell-script"></a>Запуск сценария Azure PowerShell

В сценарии, использованном в данном примере, перечислены все задания на устройстве StorSimple, включая успешно выполненные задания, задания со сбоем или выполняющиеся. Выполните следующие действия, чтобы скачать и запустить сценарий.

1. Запустите Azure PowerShell. Создайте папку и замените каталог на новую папку.

    ```
        mkdir C:\scripts\StorSimpleSDKTools
        cd C:\scripts\StorSimpleSDKTools
    ```    
2. [Скачайте интерфейс командной строки NuGet](https://www.nuget.org/downloads) из папки, созданной на предыдущем шаге. Существуют различные версии _nuget.exe_. Выберите версию, соответствующую пакету SDK. Каждая ссылка для скачивания указывает непосредственно на файл _EXE_. Щелкните правой кнопкой мыши, сохраните файл на компьютере и не запускайте его из браузера.

    Следующую команду также можно выполнить, чтобы загрузить и сохранить сценарий в той же папке (созданной ранее).
    
    ```
        wget https://dist.nuget.org/win-x86-commandline/latest/nuget.exe -Out C:\scripts\StorSimpleSDKTools\nuget.exe
    ```
3. Загрузите зависимый пакет SDK.

    ```
        C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.Azure.Management.Storsimple8000series
        C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.IdentityModel.Clients.ActiveDirectory -Version 2.28.3
        C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.Rest.ClientRuntime.Azure.Authentication -Version 2.2.9-preview
    ```    
4. Загрузите сценарий из образца проекта GitHub.

    ```
        wget https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceJobs.ps1 -Out Get-DeviceJobs.ps1

    ```

5. Выполните скрипт. При запросе аутентификации укажите учетные данные Azure. Этот сценарий должен выводить отфильтрованный список всех заданий на устройстве StorSimple.
           
    ```           
        .\Get-StorSimpleJob.ps1 -SubscriptionId [subid] -TenantId [tenant id] -DeviceName [name of device] -ResourceGroupName [name of resource group] -ManagerName[name of device manager] -FilterByStatus [Filter for job status] -FilterByJobType [Filter for job type] -FilterByStartTime [Filter for start date time] -FilterByEndTime [Filter for end date time]

    ```

### <a name="sample-output"></a>Пример выходных данных

При запуске примера сценария можно получить следующие выходные данные. Выходные данные содержат все задания, работающие на зарегистрированном устройстве, запущенном 25 сентября 2017 г. и завершенном 2 октября 2017 г.

```
-----------------------------------------
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Scripts\StorSimpleSDKTools> wget https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceJobs.ps1 -Out Get-DeviceJobs.ps1
PS C:\Scripts\StorSimpleSDKTools> .\Get-DeviceJobs.ps1 -SubscriptionId 1234ab5c-678d-910e-9fc4-0accc9c0166e -TenantId 12a345bc-67d8-91ef-01ab-2c7cd123ef45 -DeviceName 8600-ABC1234567D89EF -ResourceGroupName Contoso -ManagerName ContosoDeviceMgr -FilterByStartTime "09/25/2017 08:10:02" -FilterByEndTime "10/02/2017 08:10:02"


Status            : Succeeded
StartTime         : 10/2/2017 10:30:02 PM
EndTime           : 10/2/2017 10:31:36 PM
PercentComplete   : 100
Error             :
JobType           : ScheduledBackup
DataStats         : Microsoft.Azure.Management.StorSimple8000Series.Models.DataStatistics
EntityLabel       : ss-asr-policy1
EntityType        : Microsoft.StorSimple/managers/devices/backupPolicies
JobStages         :
DeviceId          : /subscriptions/1234ab5c-678d-910e-9fc4-0accc9c0166e/resourceGroups/Contoso/providers/Microsoft.Stor
                    Simple/managers/ContosoDeviceMgr/devices/8600-SHG0997877L71FC
IsCancellable     : True
BackupType        : CloudSnapshot
SourceDeviceId    :
BackupPointInTime : 1/1/0001 12:00:00 AM
Id                : /subscriptions/1234ab5c-678d-910e-9fc4-0accc9c0166e/resourceGroups/Contoso/providers/Microsoft.Stor
                    Simple/managers/ContosoDeviceMgr/devices/8600-SHG0997877L71FC/jobs/75905c48-b153-4af1-8b21-4b9a2ff9
                    825b
Name              : 75905c48-b153-4af1-8b21-4b9a2ff9825b
Type              : Microsoft.StorSimple/managers/devices/jobs
Kind              : Series8000
-------------------------------------------
CUT              CUT  
-------------------------------------------
Status            : Succeeded
StartTime         : 9/26/2017 5:00:02 PM
EndTime           : 9/26/2017 5:01:20 PM
PercentComplete   : 100
Error             :
JobType           : ScheduledBackup
DataStats         : Microsoft.Azure.Management.StorSimple8000Series.Models.DataStatistics
EntityLabel       : 8010 policy
EntityType        : Microsoft.StorSimple/managers/devices/backupPolicies
JobStages         :
DeviceId          : /subscriptions/1234ab5c-678d-910e-9fc4-0accc9c0166e/resourceGroups/Contoso/providers/Microsoft.Stor
                    Simple/managers/ContosoDeviceMgr/devices/8600-ABC1234567D89EF
IsCancellable     : True
BackupType        : CloudSnapshot
SourceDeviceId    :
BackupPointInTime : 1/1/0001 12:00:00 AM
Id                : /subscriptions/1234ab5c-678d-910e-9fc4-0accc9c0166e/resourceGroups/Contoso/providers/Microsoft.Stor
                    Simple/managers/ContosoDeviceMgr/devices/8600-ABC1234567D89EF/jobs/3cfd8108-db60-4e9a-a8da-6d8fe457
                    8d2b
Name              : 3cfd8108-db60-4e9a-a8da-6d8fe4578d2b
Type              : Microsoft.StorSimple/managers/devices/jobs
Kind              : Series8000



PS C:\Scripts\StorSimpleSDKTools>
--------------------------------------------

```


## <a name="next-steps"></a>Дальнейшие действия

[Использование службы диспетчера устройств StorSimple для администрирования устройства StorSimple](storsimple-8000-manager-service-administration.md).