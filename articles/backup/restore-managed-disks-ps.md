---
title: Восстановление управляемых дисков Azure с помощью Azure PowerShell
description: Узнайте, как восстановить управляемые диски Azure с помощью Azure PowerShell.
ms.topic: conceptual
ms.date: 03/26/2021
ms.openlocfilehash: 0ddf552947c39692ea01d0dea7e67f147d754fcc
ms.sourcegitcommit: a9ce1da049c019c86063acf442bb13f5a0dde213
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/27/2021
ms.locfileid: "105630447"
---
# <a name="restore-azure-managed-disks-using-azure-powershell"></a>Восстановление управляемых дисков Azure с помощью Azure PowerShell

В этой статье объясняется, как восстановить [управляемые диски Azure](../virtual-machines/managed-disks-overview.md) из точки восстановления, созданной Azure Backup.

В настоящее время параметр восстановления Original-Location (OLR) при восстановлении путем замены существующего исходного диска, с которого были сделаны резервные копии, не поддерживается. Можно выполнить восстановление из точки восстановления, чтобы создать новый диск в той же группе ресурсов, где находится исходный диск, с которого были созданы резервные копии или в любой другой группе ресурсов. Это называется восстановлением Alternate-Location (ALR), что позволяет сохранить исходный диск и восстановленный (новый) диск.

В этой статье вы узнаете, как выполнять следующие задачи.

- Восстановление для создания нового диска

- Отслеживание состояния операции восстановления

Мы будем называть существующее хранилище резервных копий "Тестбкпваулт" в группе ресурсов "Тестбкпваултрг" в примерах.

```azurepowershell-interactive
$TestBkpVault = Get-AzDataProtectionBackupVault -VaultName TestBkpVault -ResourceGroupName "testBkpVaultRG"
```

## <a name="restore-to-create-a-new-disk"></a>Восстановление для создания нового диска

### <a name="setting-up-permissions"></a>Настройка разрешений

Хранилище службы архивации использует управляемое удостоверение для доступа к другим ресурсам Azure. Для восстановления из резервной копии управляемое удостоверение хранилища службы архивации требует наличия набора разрешений на группу ресурсов, в которую будет восстановлен диск.

Хранилище службы архивации использует управляемое системой удостоверение, которое ограничено по одному на ресурс и связано с жизненным циклом этого ресурса. Вы можете предоставить разрешения управляемому удостоверению с помощью управления доступом на основе ролей Azure (Azure RBAC). Управляемое удостоверение — это субъект-Служба специального типа, который может использоваться только с ресурсами Azure. Дополнительные сведения об [управляемых удостоверениях](../active-directory/managed-identities-azure-resources/overview.md).

Назначьте соответствующие разрешения управляемому удостоверению, назначенному системой хранилища, в Целевой группе ресурсов, где диски будут восстановлены или созданы, как упоминалось [здесь](restore-managed-disks.md#restore-to-create-a-new-disk).

### <a name="fetching-the-relevant-recovery-point"></a>Получение соответствующей точки восстановления

Извлеките все экземпляры с помощью команды [Get-аздатапротектионбаккупинстанце](/powershell/module/az.dataprotection/get-azdataprotectionbackupinstance?view=azps-5.7.0&preserve-view=true) и найдите соответствующий экземпляр.

```azurepowershell-interactive
$AllInstances = Get-AzDataProtectionBackupInstance -ResourceGroupName "testBkpVaultRG" -VaultName $TestBkpVault.Name
```

Для поиска по экземплярам во многих хранилищах и подписках можно также использовать команду **AZ. ресаурцеграф** и командлет [Search-аздатапротектионбаккупинстанцеиназграф](/powershell/module/az.dataprotection/search-azdataprotectionbackupinstanceinazgraph?view=azps-5.7.0&preserve-view=true) .

```azurepowershell-interactive
$AllInstances = Search-AzDataProtectionBackupInstanceInAzGraph -ResourceGroupName "testBkpVaultRG" -VaultName $TestBkpVault.Name -DatasourceType AzureDisk -ProtectionStatus ProtectionConfigured
```

После определения экземпляра извлеките соответствующую точку восстановления.

```azurepowershell-interactive
$rp = Get-AzDataProtectionRecoveryPoint -ResourceGroupName "testBkpVaultRG" -VaultName $TestBkpVault.Name -BackupInstanceName $AllInstances[2].BackupInstanceName
```

### <a name="preparing-the-restore-request"></a>Подготовка запроса на восстановление

Создайте идентификатор ARM нового диска, который будет создан с помощью целевой группы ресурсов, для которой назначены разрешения, как описано [выше](#setting-up-permissions), и необходимое имя диска. Например, диск можно назвать **PSTestDisk2** в группе ресурсов **targetrg** с другой подпиской.

```azurepowershell-interactive
$targetDiskId = /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx/resourceGroups/targetrg/providers/Microsoft.Compute/disks/PSTestDisk2
```

Используйте команду [Initialize-аздатапротектионресторерекуест](/powershell/module/az.dataprotection/initialize-azdataprotectionrestorerequest?view=azps-5.7.0&preserve-view=true) , чтобы подготовить запрос на восстановление со всеми релевантными сведениями.

```azurepowershell-interactive
$restorerequest = Initialize-AzDataProtectionRestoreRequest -DatasourceType AzureDisk -SourceDataStore OperationalStore -RestoreLocation $TestBkpVault.Location  -RestoreType AlternateLocation -TargetResourceId $targetDiskId -RecoveryPoint $rp[0].Name
```

### <a name="trigger-the-restore"></a>Активация восстановления

Используйте команду [Start-аздатапротектионбаккупинстанцересторе](/powershell/module/az.dataprotection/start-azdataprotectionbackupinstancerestore?view=azps-5.7.0&preserve-view=true) , чтобы активировать восстановление с помощью запроса, подготовленного выше.

```azurepowershell-interactive
Start-AzDataProtectionBackupInstanceRestore -BackupInstanceName $AllInstances[2].BackupInstanceName -ResourceGroupName "testBkpVaultRG" -VaultName $TestBkpVault.Name -Parameter $restorerequest
```

## <a name="tracking-job"></a>Задание отслеживания

Отследите все задания с помощью команды [Get-аздатапротектионжоб](/powershell/module/az.dataprotection/get-azdataprotectionjob?view=azps-5.7.0&preserve-view=true) . Можно вывести список всех заданий и получить сведения о конкретном задании.

Можно также использовать **AZ. ресаурцеграф** для трассировки всех заданий во всех резервных хранилищах. Используйте команду [Search-аздатапротектионжобиназграф](/powershell/module/az.dataprotection/search-azdataprotectionjobinazgraph?view=azps-5.7.0&preserve-view=true) , чтобы получить соответствующее задание, которое может быть в любом хранилище службы архивации.

```azurepowershell-interactive
$job = Search-AzDataProtectionJobInAzGraph -Subscription $sub -ResourceGroupName "testBkpVaultRG" -Vault $TestBkpVault.Name -DatasourceType AzureDisk -Operation OnDemandBackup
```

## <a name="next-steps"></a>Дальнейшие шаги

- [Вопросы и ответы по резервному копированию дисков Azure](disk-backup-faq.md)
