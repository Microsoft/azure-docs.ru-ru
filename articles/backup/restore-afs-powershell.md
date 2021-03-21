---
title: Восстановление файлов Azure с помощью PowerShell
description: Из этой статьи вы узнаете, как восстановить файлы Azure с помощью службы Azure Backup и PowerShell.
ms.topic: conceptual
ms.date: 1/27/2020
ms.openlocfilehash: 60c9848e12de80bcafe4553a9e8f3e27e8876d41
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96021390"
---
# <a name="restore-azure-files-with-powershell"></a>Восстановление файлов Azure с помощью PowerShell

В этой статье объясняется, как восстановить всю общую папку или отдельные файлы из точки восстановления, созданной службой [Azure Backup](backup-overview.md) с помощью Azure PowerShell.

Можно восстановить весь файловый ресурс или отдельные файлы в общей папке. Можно выполнить восстановление в исходное расположение или в другое расположение.

> [!WARNING]
> Убедитесь, что версия PowerShell обновлена до минимальной версии "az. RecoveryServices 2.6.0" для резервных копий AFS. Дополнительные сведения см. [в разделе](backup-azure-afs-automation.md#important-notice-backup-item-identification) Структурирование требований к этому изменению.

>[!NOTE]
>Azure Backup теперь поддерживает восстановление нескольких файлов или папок в исходное или альтернативное расположение с помощью PowerShell. Обратитесь к [этому разделу](#restore-multiple-files-or-folders-to-original-or-alternate-location) документа, чтобы узнать, как это делать.

## <a name="fetch-recovery-points"></a>Получение точек восстановления

Используйте [Get-азрековерисервицесбаккупрековерипоинт](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprecoverypoint) , чтобы получить список всех точек восстановления для архивированного элемента.

В следующем скрипте:

* Переменная **$RP** представляет собой массив точек восстановления для выбранного элемента резервного копирования за последние семь дней.
* Массив сортируется по времени в обратном порядке, так что последняя точка восстановления получает индекс **0**.
* Используйте стандартное индексирование массива PowerShell для выбора точки восстановления.
* В этом примере **$RP [0]** выбирает последнюю точку восстановления.

```powershell
$vault = Get-AzRecoveryServicesVault -ResourceGroupName "azurefiles" -Name "azurefilesvault"
$Container = Get-AzRecoveryServicesBackupContainer -ContainerType AzureStorage -Status Registered -FriendlyName "afsaccount" -VaultId $vault.ID
$BackupItem = Get-AzRecoveryServicesBackupItem -Container $Container -WorkloadType AzureFiles -VaultId $vault.ID -FriendlyName "azurefiles"
$startDate = (Get-Date).AddDays(-7)
$endDate = Get-Date
$rp = Get-AzRecoveryServicesBackupRecoveryPoint -Item $BackupItem -VaultId $vault.ID -StartDate $startdate.ToUniversalTime() -EndDate $enddate.ToUniversalTime()
$rp[0] | fl
```

Выходные данные похожи на приведенные ниже.

```powershell
FileShareSnapshotUri : https://testStorageAcct.file.core.windows.net/testAzureFS?sharesnapshot=2018-11-20T00:31:04.00000
                       00Z
RecoveryPointType    : FileSystemConsistent
RecoveryPointTime    : 11/20/2018 12:31:05 AM
RecoveryPointId      : 86593702401459
ItemName             : testAzureFS
Id                   : /Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/Micros                      oft.RecoveryServices/vaults/testVault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;teststorageRG;testStorageAcct/protectedItems/AzureFileShare;testAzureFS/recoveryPoints/86593702401462
WorkloadType         : AzureFiles
ContainerName        : storage;teststorageRG;testStorageAcct
ContainerType        : AzureStorage
BackupManagementType : AzureStorage
```

После выбора соответствующей точки восстановления вы восстанавливаете общую папку или файл в исходное расположение или в альтернативное расположение.

## <a name="restore-an-azure-file-share-to-an-alternate-location"></a>Восстановление файлового ресурса Azure в альтернативное расположение

Используйте [RESTORE-азрековерисервицесбаккупитем](/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem) для восстановления в выбранную точку восстановления. Укажите эти параметры, чтобы определить альтернативное расположение:

* **Таржетсторажеаккаунтнаме**: учетная запись хранения, в которую восстанавливается резервное содержимое. Целевая учетная запись хранения должна быть в том же расположении, что и хранилище.
* **Таржетфилешаренаме**. файловые ресурсы в целевой учетной записи хранения, в которую восстанавливается резервное содержимое.
* **TargetFolder**: папка в общей папке, в которую восстанавливаются данные. Если содержимое резервной копии необходимо восстановить в корневую папку, укажите значения целевой папки в виде пустой строки.
* **ResolveConflict**: инструкция при конфликте с восстановленными данными. Принимает **перезаписать** или **пропустить**.

Выполните командлет с параметрами следующим образом:

```powershell
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -TargetStorageAccountName "TargetStorageAcct" -TargetFileShareName "DestAFS" -TargetFolder "testAzureFS_restored" -ResolveConflict Overwrite
```

Эта команда возвращает задания для отслеживания с идентификатором, как показано в следующем примере.

```powershell
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
testAzureFS        Restore              InProgress           12/10/2018 9:56:38 AM                               9fd34525-6c46-496e-980a-3740ccb2ad75
```

## <a name="restore-an-azure-file-to-an-alternate-location"></a>Восстановление файла Azure в альтернативное расположение

Используйте [RESTORE-азрековерисервицесбаккупитем](/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem) для восстановления в выбранную точку восстановления. Укажите эти параметры, чтобы определить альтернативное расположение, и, чтобы однозначно определить файл, который требуется восстановить.

* **Таржетсторажеаккаунтнаме**: учетная запись хранения, в которую восстанавливается резервное содержимое. Целевая учетная запись хранения должна быть в том же расположении, что и хранилище.
* **Таржетфилешаренаме**. файловые ресурсы в целевой учетной записи хранения, в которую восстанавливается резервное содержимое.
* **TargetFolder**: папка в общей папке, в которую восстанавливаются данные. Если содержимое резервной копии необходимо восстановить в корневую папку, укажите значения целевой папки в виде пустой строки.
* **SourceFilePath**: абсолютный путь к файлу, восстанавливаемому в общей папке, в виде строки. Этот же путь используется в командлете PowerShell **Get-AzStorageFile**.
* **Саурцефилетипе**: указывает, выбран ли каталог или файл. Принимает **Каталог** или **файл**.
* **ResolveConflict**: инструкция при конфликте с восстановленными данными. Принимает **перезаписать** или **пропустить**.

Дополнительные параметры (SourceFilePath и Саурцефилетипе) относятся только к отдельному файлу, который требуется восстановить.

```powershell
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -TargetStorageAccountName "TargetStorageAcct" -TargetFileShareName "DestAFS" -TargetFolder "testAzureFS_restored" -SourceFileType File -SourceFilePath "TestDir/TestDoc.docx" -ResolveConflict Overwrite
```

Эта команда возвращает задание с ИДЕНТИФИКАТОРом для трассировки, как показано в предыдущем разделе.

## <a name="restore-azure-file-shares-and-files-to-the-original-location"></a>Восстановить файловые ресурсы и файлы Azure в исходном расположении

При восстановлении в исходное расположение не нужно указывать параметры, связанные с назначением и целевым объектом. Нужно только указать **ResolveConflict**.

### <a name="overwrite-an-azure-file-share"></a>Перезапись файлового ресурса Azure

```powershell
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -ResolveConflict Overwrite
```

### <a name="overwrite-an-azure-file"></a>Перезапись файла Azure

```powershell
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -SourceFileType File -SourceFilePath "TestDir/TestDoc.docx" -ResolveConflict Overwrite
```

## <a name="restore-multiple-files-or-folders-to-original-or-alternate-location"></a>Восстановление нескольких файлов или папок в исходном или альтернативном расположении

Используйте команду [RESTORE-азрековерисервицесбаккупитем](/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem) , передав путь ко всем файлам или папкам, которые требуется восстановить, в качестве значения параметра **мултиплесаурцефилепас** .

### <a name="restore-multiple-files"></a>Восстановление нескольких файлов

В следующем сценарии мы пытаемся восстановить файлы *FileSharePage.png* и *MyTestFile.txt* .

```powershell
$vault = Get-AzRecoveryServicesVault -ResourceGroupName "azurefiles" -Name "azurefilesvault"

$Container = Get-AzRecoveryServicesBackupContainer -ContainerType AzureStorage -Status Registered -FriendlyName "afsaccount" -VaultId $vault.ID

$BackupItem = Get-AzRecoveryServicesBackupItem -Container $Container -WorkloadType AzureFiles -VaultId $vault.ID -FriendlyName "azurefiles"

$RP = Get-AzRecoveryServicesBackupRecoveryPoint -Item $BackupItem -VaultId $vault.ID

$files = ("FileSharePage.png", "MyTestFile.txt")

Restore-AzRecoveryServicesBackupItem -RecoveryPoint $RP[0] -MultipleSourceFilePath $files -SourceFileType File -ResolveConflict Overwrite -VaultId $vault.ID -VaultLocation $vault.Location
```

### <a name="restore-multiple-directories"></a>Восстановление нескольких каталогов

В следующем сценарии мы пытаемся восстановить *zrs1_restore* и *восстановить* каталоги.

```powershell
$vault = Get-AzRecoveryServicesVault -ResourceGroupName "azurefiles" -Name "azurefilesvault"

$Container = Get-AzRecoveryServicesBackupContainer -ContainerType AzureStorage -Status Registered -FriendlyName "afsaccount" -VaultId $vault.ID

$BackupItem = Get-AzRecoveryServicesBackupItem -Container $Container -WorkloadType AzureFiles -VaultId $vault.ID -FriendlyName "azurefiles"

$RP = Get-AzRecoveryServicesBackupRecoveryPoint -Item $BackupItem -VaultId $vault.ID

$files = ("Restore","zrs1_restore")

Restore-AzRecoveryServicesBackupItem -RecoveryPoint $RP[0] -MultipleSourceFilePath $files -SourceFileType Directory -ResolveConflict Overwrite -VaultId $vault.ID -VaultLocation $vault.Location
```

Результат будет выглядеть примерно так:

```output
WorkloadName         Operation         Status          StartTime                EndTime       JobID
------------         ---------         ------          ---------                -------       -----
azurefiles           Restore           InProgress      4/5/2020 8:01:24 AM                    cd36abc3-0242-44b1-9964-0a9102b74d57
```

Если вы хотите восстановить несколько файлов или папок в альтернативное расположение, используйте приведенные выше сценарии, указав значения параметров, связанные с целевым расположением, как описано выше в примере [восстановления файла Azure в альтернативное расположение](#restore-an-azure-file-to-an-alternate-location).

## <a name="next-steps"></a>Дальнейшие действия

[Дополнительные сведения о](restore-afs.md) восстановлении файлов Azure в портал Azure.
