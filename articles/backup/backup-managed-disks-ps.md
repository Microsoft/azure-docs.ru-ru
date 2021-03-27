---
title: Резервное копирование управляемых дисков Azure с помощью Azure PowerShell
description: Узнайте, как выполнять резервное копирование управляемых дисков Azure с помощью Azure PowerShell.
ms.topic: conceptual
ms.date: 03/26/2021
ms.openlocfilehash: 2e286128185c1ac7fe4861a9b06de52e10d4139a
ms.sourcegitcommit: a9ce1da049c019c86063acf442bb13f5a0dde213
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/27/2021
ms.locfileid: "105630497"
---
# <a name="back-up-azure-managed-disks-using-azure-powershell"></a>Резервное копирование управляемых дисков Azure с помощью Azure PowerShell

В этой статье объясняется, как выполнить резервное копирование [управляемого диска Azure](../virtual-machines/managed-disks-overview.md) с помощью Azure PowerShell.

В этой статье вы узнаете, как выполнять следующие задачи.

- Создание резервного хранилища

- создание политики архивации;

- Настройка резервной копии диска Azure

- Выполнение задания резервного копирования по запросу

Сведения о доступности региона резервного копирования на диске Azure, поддерживаемых сценариях и ограничениях см. в [матрице поддержки](disk-backup-support-matrix.md).

## <a name="create-a-backup-vault"></a>Создание резервного хранилища

Хранилище службы архивации — это сущность хранилища в Azure, которая содержит данные резервного копирования для различных более новых рабочих нагрузок, которые Azure Backup поддерживаются, например для серверов базы данных Azure для PostgreSQL и дисков Azure. Резервные хранилища упрощают организацию резервных копий данных, сокращая при этом расходы на управление. Резервные хранилища основаны на модели Azure Resource Manager Azure, которая предоставляет расширенные возможности для защиты данных резервного копирования.

Перед созданием резервного хранилища выберите избыточность хранилища данных в хранилище. Затем перейдите к созданию хранилища архивации с такой избыточностью хранилища и расположению. В этой статье мы создадим хранилище службы архивации "Тестбкпваулт" в регионе "westus" в группе ресурсов "Тестбкпваултрг". Используйте команду [New-аздатапротектионбаккупваулт](/powershell/module/az.dataprotection/new-azdataprotectionbackupvault?view=azps-5.7.0&preserve-view=true) , чтобы создать хранилище службы архивации. Дополнительные сведения о [создании хранилища службы архивации](./backup-vault-overview.md#create-a-backup-vault).

```azurepowershell-interactive
$storageSetting = New-AzDataProtectionBackupVaultStorageSettingObject -Type LocallyRedundant/GeoRedundant -DataStoreType VaultStore
New-AzDataProtectionBackupVault -ResourceGroupName testBkpVaultRG -VaultName TestBkpVault -Location westus -StorageSetting $storageSetting
$TestBkpVault = Get-AzDataProtectionBackupVault -VaultName TestBkpVault
$TestBKPVault | fl
ETag                :
Id                  : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx/resourceGroups/testBkpVaultRG/providers/Microsoft.DataProtection/backupVaults/TestBkpVault
Identity            : Microsoft.Azure.PowerShell.Cmdlets.DataProtection.Models.Api20210201Preview.DppIdentityDetails
IdentityPrincipalId :
IdentityTenantId    :
IdentityType        :
Location            : westus
Name                : TestBkpVault
ProvisioningState   : Succeeded
StorageSetting      : {Microsoft.Azure.PowerShell.Cmdlets.DataProtection.Models.Api20210201Preview.StorageSetting}
SystemData          : Microsoft.Azure.PowerShell.Cmdlets.DataProtection.Models.Api20210201Preview.SystemData
Tag                 : Microsoft.Azure.PowerShell.Cmdlets.DataProtection.Models.Api20210201Preview.DppTrackedResourceTags
Type                : Microsoft.DataProtection/backupVaults
```

После создания хранилища создадим политику резервного копирования для защиты дисков Azure.

## <a name="create-a-backup-policy"></a>Создание политики архивации

Чтобы узнать о внутренних компонентах политики резервного копирования дисков Azure, извлеките шаблон политики с помощью команды [Get-аздатапротектионполицитемплате](/powershell/module/az.dataprotection/get-azdataprotectionpolicytemplate?view=azps-5.7.0&preserve-view=true). Эта команда возвращает шаблон политики по умолчанию для заданного типа источника данных. Используйте этот шаблон политики для создания новой политики.

```azurepowershell-interactive
$policyDefn = Get-AzDataProtectionPolicyTemplate -DatasourceType AzureDisk
$policyDefn | fl


DatasourceType : {Microsoft.Compute/disks}
ObjectType     : BackupPolicy
PolicyRule     : {BackupHourly, Default}

$policyDefn.PolicyRule | fl


BackupParameter           : Microsoft.Azure.PowerShell.Cmdlets.DataProtection.Models.Api20210201Preview.AzureBackupParams
BackupParameterObjectType : AzureBackupParams
DataStoreObjectType       : DataStoreInfoBase
DataStoreType             : OperationalStore
Name                      : BackupHourly
ObjectType                : AzureBackupRule
Trigger                   : Microsoft.Azure.PowerShell.Cmdlets.DataProtection.Models.Api20210201Preview.ScheduleBasedTriggerContext
TriggerObjectType         : ScheduleBasedTriggerContext

IsDefault  : True
Lifecycle  : {Microsoft.Azure.PowerShell.Cmdlets.DataProtection.Models.Api20210201Preview.SourceLifeCycle}
Name       : Default
ObjectType : AzureRetentionRule
```

Шаблон политики состоит из триггера (который определяет, что инициирует резервное копирование) и жизненного цикла (который принимает решение об удалении, копировании и перемещении резервной копии). В резервной копии диска Azure значение по умолчанию для триггера — это запланированный ежечасный триггер на каждые 4 часа (PT4H) и хранить каждую резервную копию в течение 7 дней.

```azurepowershell-interactive
 $policyDefn.PolicyRule[0].Trigger | fl


ObjectType                    : ScheduleBasedTriggerContext
ScheduleRepeatingTimeInterval : {R/2020-04-05T13:00:00+00:00/PT4H}
TaggingCriterion              : {Default}
```

```azurepowershell-interactive
$policyDefn.PolicyRule[1].Lifecycle | fl


DeleteAfterDuration        : P7D
DeleteAfterObjectType      : AbsoluteDeleteOption
SourceDataStoreObjectType  : DataStoreInfoBase
SourceDataStoreType        : OperationalStore
TargetDataStoreCopySetting :
```

Служба архивации дисков Azure предлагает несколько резервных копий в день. Если требуется более частые резервные копии, выберите периодичность резервного **копирования с** возможностью создания резервных копий с интервалами в 4, 6, 8 или 12 часов. Резервные копии планируются на основе выбранного интервала **времени** . Например, если выбрать **каждые 4 часа**, резервные копии будут выполняться приблизительно в интервале каждые 4 часа, поэтому резервные копии распределяются в один и тот же день. Если один раз в день достаточно резервного копирования, выберите периодичность **ежедневного** резервного копирования. При ежедневном резервном копировании можно указать время суток, в которое будет выполняться резервное копирование. Важно отметить, что время суток указывает время начала резервного копирования, а не время завершения резервного копирования. Время, необходимое для выполнения операции резервного копирования, зависит от различных факторов, включая размер диска и частоту обновлений между последовательными резервными копиями. Однако резервное копирование дисков Azure — это резервная копия без агента, которая использует [добавочные моментальные снимки](../virtual-machines/disks-incremental-snapshots.md), что не влияет на производительность рабочего приложения.

   >[!NOTE]
   > Хотя в выбранном хранилище может быть задан параметр глобального резервирования, в настоящее время резервная копия диска Azure поддерживает только хранилище данных моментальных снимков. Все резервные копии хранятся в группе ресурсов в подписке и не копируются в хранилище резервных копий.

Дополнительные сведения о создании политик см. в документе о [политике резервного копирования дисков Azure](backup-managed-disks.md#create-backup-policy) .

Если вы хотите изменить почасовую частоту или срок хранения, используйте команды [Edit-аздатапротектионполицитригжерклиентобжект](/powershell/module/az.dataprotection/edit-azdataprotectionpolicytriggerclientobject?view=azps-5.7.0&preserve-view=true) и/или [Edit-аздатапротектионполициретентионрулеклиентобжект](/powershell/module/az.dataprotection/edit-azdataprotectionpolicyretentionruleclientobject?view=azps-5.7.0&preserve-view=true) . Когда объект политики будет иметь все нужные значения, перейдите к созданию новой политики из объекта политики с помощью команды [New-аздатапротектионбаккупполици](/powershell/module/az.dataprotection/new-azdataprotectionbackuppolicy?view=azps-5.7.0&preserve-view=true).

```azurepowershell-interactive
New-AzDataProtectionBackupPolicy -ResourceGroupName "testBkpVaultRG" -VaultName $TestBkpVault.Name -Name diskBkpPolicy -Policy $policyDefn

Name                   Type
----                   ----
diskBkpPolicy       Microsoft.DataProtection/backupVaults/backupPolicies

$diskBkpPol = Get-AzDataProtectionBackupPolicy -ResourceGroupName "testBkpVaultRG" -VaultName $TestBkpVault.Name -Name "diskBkpPolicy"
```

## <a name="configure-backup"></a>Настройка резервного копирования

После создания хранилища и политики существует 3 важных пункта, которые необходимо учитывать пользователю для защиты диска Azure.

### <a name="key-entities-involved"></a>Участвующие ключевые сущности

#### <a name="disk-to-be-protected"></a>Защищаемый диск

Получение идентификатора ARM для защищаемого диска. Он будет служить идентификатором диска. Мы будем использовать пример диска с именем "Пстестдиск" в группе ресурсов "дискрг" в другой подписке.

```azurepowershell-interactive
$DiskId = "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx/resourcegroups/diskrg/providers/Microsoft.Compute/disks/PSTestDisk"
```

#### <a name="snapshot-resource-group"></a>Группа ресурсов моментальных снимков

Моментальные снимки диска хранятся в группе ресурсов в вашей подписке. Рекомендуется создать выделенную группу ресурсов в качестве хранилища данных моментальных снимков для использования службой Azure Backup. Наличие выделенной группы ресурсов позволяет ограничивать права доступа к группе ресурсов, обеспечивая безопасность и простоту управления резервными копиями данных. Обратите внимание на идентификатор ARM для группы ресурсов, в которой нужно разместить моментальные снимки диска.

```azurepowershell-interactive
$snapshotrg = "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx/resourceGroups/snapshotrg"
```

#### <a name="backup-vault"></a>Хранилище службы архивации

Хранилищам архивации требуются разрешения на диск и группу ресурсов моментальных снимков, чтобы иметь возможность активировать моментальные снимки и управлять жизненным циклом. Назначенное системой управляемое удостоверение хранилища используется для назначения таких разрешений. Используйте команду [Update-азрековерисервицесваулт](/powershell/module/az.recoveryservices/update-azrecoveryservicesvault?view=azps-5.7.0&preserve-view=true) для включения управляемого удостоверения, назначенного системой для хранилища служб восстановления.

### <a name="assign-permissions"></a>Назначение разрешений

Пользователю необходимо назначить несколько разрешений через RBAC в хранилище (представленное MSI-файлом хранилища), а также на соответствующем диске и (или) диске RG. Их можно выполнить с помощью портала или PowerShell. Все связанные разрешения подробно описаны в точках 1, 2 и 3 в [этом разделе](backup-managed-disks.md#configure-backup).

### <a name="prepare-the-request"></a>Подготовка запроса

После установки всех соответствующих разрешений Настройка резервного копирования выполняется в два этапа. Сначала мы подготовим соответствующий запрос с помощью соответствующей группы ресурсов хранилища, политики, диска и снимка с помощью команды [Initialize-аздатапротектионбаккупинстанце](/powershell/module/az.dataprotection/initialize-azdataprotectionbackupinstance?view=azps-5.7.0&preserve-view=true) . Затем мы отправим запрос на защиту диска с помощью команды [New-аздатапротектионбаккупинстанце](/powershell/module/az.dataprotection/new-azdataprotectionbackupinstance?view=azps-5.7.0&preserve-view=true) .

```azurepowershell-interactive
$instance = Initialize-AzDataProtectionBackupInstance -DatasourceType AzureDisk -DatasourceLocation $TestBkpvault.Location -PolicyId $diskBkpPol[0].Id -DatasourceId $DiskId 
$instance.Property.PolicyInfo.PolicyParameter.DataStoreParametersList[0].ResourceGroupId = $snapshotrg
New-AzDataProtectionBackupInstance -ResourceGroupName "testBkpVaultRG" -VaultName $TestBkpVault.Name -BackupInstance $instance

Name                                                       Type                                                  BackupInstanceName
----                                                       ----                                                  ------------------
diskrg-PSTestDisk-3df6ac08-9496-4839-8fb5-8b78e594f166 Microsoft.DataProtection/backupVaults/backupInstances diskrg-PSTestDisk-3df6ac08-9496-4839-8fb5-8b78e594f166
```

## <a name="run-an-on-demand-backup"></a>Выполнение резервного копирования по запросу

Получите соответствующий экземпляр резервной копии, на который пользователь хочет активировать резервное копирование с помощью команды [Get-аздатапротектионбаккупинстанце](/powershell/module/az.dataprotection/get-azdataprotectionbackupinstance?view=azps-5.7.0&preserve-view=true) .

```azurepowershell-interactive
$instance = Get-AzDataProtectionBackupInstance -SubscriptionId "xxxx-xxx-xxx" -ResourceGroupName "testBkpVaultRG" -VaultName $TestBkpVault.Name -Name "BackupInstanceName"
```

При активации резервного копирования можно указать правило хранения. Чтобы просмотреть правила хранения в политике, просмотрите объект политики для правил хранения. В приведенном ниже примере отображается правило с именем Default, которое будет использоваться для резервного копирования по запросу.

```azurepowershell-interactive
$policyDefn.PolicyRule | fl


BackupParameter           : Microsoft.Azure.PowerShell.Cmdlets.DataProtection.Models.Api20210201Preview.AzureBackupParams
BackupParameterObjectType : AzureBackupParams
DataStoreObjectType       : DataStoreInfoBase
DataStoreType             : OperationalStore
Name                      : BackupHourly
ObjectType                : AzureBackupRule
Trigger                   : Microsoft.Azure.PowerShell.Cmdlets.DataProtection.Models.Api20210201Preview.ScheduleBasedTriggerContext
TriggerObjectType         : ScheduleBasedTriggerContext

IsDefault  : True
Lifecycle  : {Microsoft.Azure.PowerShell.Cmdlets.DataProtection.Models.Api20210201Preview.SourceLifeCycle}
Name       : Default
ObjectType : AzureRetentionRule
```

Активируйте резервную копию по запросу с помощью команды [BACKUP-аздатапротектионбаккупинстанцеадхок](/powershell/module/az.dataprotection/backup-azdataprotectionbackupinstanceadhoc?view=azps-5.7.0&preserve-view=true) .

```azurepowershell-interactive
$AllInstances = Get-AzDataProtectionBackupInstance -ResourceGroupName "testBkpVaultRG" -VaultName $TestBkpVault.Name
Backup-AzDataProtectionBackupInstanceAdhoc -BackupInstanceName $AllInstances[0].Name -ResourceGroupName "testBkpVaultRG" -VaultName $TestBkpVault.Name -BackupRuleOptionRuleName "Default"
```

## <a name="tracking-jobs"></a>Отслеживание заданий

Отследите все задания с помощью команды [Get-аздатапротектионжоб](/powershell/module/az.dataprotection/get-azdataprotectionjob?view=azps-5.7.0&preserve-view=true) . Можно вывести список всех заданий и получить сведения о конкретном задании.

Можно также использовать AZ. Ресаурцеграф для трассировки всех заданий во всех резервных хранилищах. Используйте команду [Search-аздатапротектионжобиназграф](/powershell/module/az.dataprotection/search-azdataprotectionjobinazgraph?view=azps-5.7.0&preserve-view=true) , чтобы получить соответствующее задание, которое может быть в любом хранилище службы архивации.

```azurepowershell-interactive
  $job = Search-AzDataProtectionJobInAzGraph -Subscription $sub -ResourceGroupName "testBkpVaultRG" -Vault $TestBkpVault.Name -DatasourceType AzureDisk -Operation OnDemandBackup
```

## <a name="next-steps"></a>Дальнейшие шаги

- [Восстановление управляемых дисков Azure с помощью Azure PowerShell](restore-managed-disks-ps.md)
