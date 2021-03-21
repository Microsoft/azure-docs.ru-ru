---
title: Резервное копирование и восстановление виртуальных машин Azure с помощью PowerShell
description: В этой статье описывается, как выполнять резервное копирование и восстановление виртуальных машин Azure с использованием Azure Backup с помощью PowerShell.
ms.topic: conceptual
ms.date: 09/11/2019
ms.openlocfilehash: f59c18aecf577bc7f7d0b1360dd36504305af893
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100633195"
---
# <a name="back-up-and-restore-azure-vms-with-powershell"></a>Резервное копирование и восстановление виртуальных машин Azure с помощью PowerShell

В этой статье объясняется, как создать резервную копию и восстановить виртуальную машину Azure в хранилище служб [Azure Backup](backup-overview.md) восстановления с помощью командлетов PowerShell.

В этой статье раскрываются следующие темы:

> [!div class="checklist"]
>
> * Создание служб восстановления и указание контекста хранилища.
> * Определение политики архивации.
> * Применение политики архивации для защиты нескольких виртуальных машин.
> * Запустите задание резервного копирования по запросу для защищенных виртуальных машин. Прежде чем можно будет переходить к резервному копированию (или защите) виртуальной машины, вам потребуется выполнить [необходимые условия](backup-azure-arm-vms-prepare.md), чтобы подготовить среду для защиты виртуальных машин.

## <a name="before-you-start"></a>Перед началом работы

* Дополнительные [сведения](backup-azure-recovery-services-vault-overview.md) о хранилищах служб восстановления.
* [Изучите](backup-architecture.md#architecture-built-in-azure-vm-backup) архитектуру для резервного копирования виртуальных машин Azure, [Узнайте о](backup-azure-vms-introduction.md) процессе резервного копирования и [Ознакомьтесь](backup-support-matrix-iaas.md) со сведениями о поддержке, ограничениях и предварительных требованиях.
* Проверьте иерархию объектов PowerShell для служб восстановления.

## <a name="recovery-services-object-hierarchy"></a>Иерархия объектов служб восстановления

Иерархия объектов представлена на следующей схеме.

![Иерархия объектов служб восстановления](./media/backup-azure-vms-arm-automation/recovery-services-object-hierarchy.png)

Ознакомьтесь со справочником по [командлету](/powershell/module/az.recoveryservices/) **AZ. RecoveryServices** в библиотеке Azure.

## <a name="set-up-and-register"></a>Настройка и регистрация

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Чтобы начать работу, сделайте следующее:

1. [Скачайте последнюю версию PowerShell](/powershell/azure/install-az-ps)

2. Чтобы получить список доступных командлетов PowerShell для службы архивации Azure, введите следующую команду:

    ```powershell
    Get-Command *azrecoveryservices*
    ```

    Отображаются псевдонимы и командлеты для службы архивации Azure, Azure Site Recovery и хранилища служб восстановления. Ниже приведен пример того, что вы увидите. Это не полный список командлетов.

    ![Список служб восстановления](./media/backup-azure-vms-automation/list-of-recoveryservices-ps.png)

3. Чтобы войти в учетную запись Azure, используйте командлет **Connect-AzAccount**. Откроется веб-станица, на которой пользователю предлагается ввести данные для входа в учетную запись:

    * Кроме того, учетные данные можно добавить в качестве параметра в командлет **Connect-AzAccount**, используя параметр **-Credential**.
    * Если вы являетесь партнером CSP, работающим от имени клиента, укажите клиента в качестве клиента, используя его идентификатор tenantID или основное доменное имя клиента. Например: **Connect-азаккаунт-клиент "fabrikam.com"**

4. Свяжите подписку, которую собираетесь использовать, с учетной записью, так как последняя может иметь несколько подписок:

    ```powershell
    Select-AzSubscription -SubscriptionName $SubscriptionName
    ```

5. Если вы используете Azure Backup в первый раз, необходимо использовать командлет **[Register-азресаурцепровидер](/powershell/module/az.resources/register-azresourceprovider)** для регистрации поставщика службы восстановления Azure в подписке.

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

6. Вы можете проверить, зарегистрированы ли поставщики, выполнив следующие команды:

    ```powershell
    Get-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

    В выходных данных команды для **RegistrationState** должно быть установлено значение **Registered**. Если нет, просто выполните командлет **[Register-азресаурцепровидер](/powershell/module/az.resources/register-azresourceprovider)** еще раз.

## <a name="create-a-recovery-services-vault"></a>Создание хранилища Служб восстановления

Чтобы создать хранилище служб восстановления, выполните описанные ниже действия. Хранилище служб восстановления отличается от хранилища службы архивации.

1. Хранилище служб восстановления представляет собой ресурс Resource Manager, поэтому вам потребуется разместить его в группе ресурсов. Можно использовать существующую группу ресурсов или создать группу ресурсов с помощью командлета **[New-азресаурцеграуп](/powershell/module/az.resources/new-azresourcegroup)** . При создании группы ресурсов укажите ее имя и расположение.  

    ```powershell
    New-AzResourceGroup -Name "test-rg" -Location "West US"
    ```

2. Воспользуйтесь командлетом [New-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/new-azrecoveryservicesvault), чтобы создать хранилище Служб восстановления. Разместите хранилище там же, где находится группа ресурсов.

    ```powershell
    New-AzRecoveryServicesVault -Name "testvault" -ResourceGroupName "test-rg" -Location "West US"
    ```

3. Укажите необходимый тип избыточности хранилища: Можно использовать [локально избыточное хранилище (LRS)](../storage/common/storage-redundancy.md#locally-redundant-storage), [геоизбыточное хранилище (GRS)](../storage/common/storage-redundancy.md#geo-redundant-storage)или [хранилище, избыточное в виде зоны (ZRS)](../storage/common/storage-redundancy.md#zone-redundant-storage). В следующем примере показано, что для параметра **-BackupStorageRedundancy** для *testvault* задано значение **GeoRedundant**.

    ```powershell
    $vault1 = Get-AzRecoveryServicesVault -Name "testvault"
    Set-AzRecoveryServicesBackupProperty  -Vault $vault1 -BackupStorageRedundancy GeoRedundant
    ```

   > [!TIP]
   > Для многих командлетов службы архивации Azure требуется объект хранилища служб восстановления в качестве входных данных. По этой причине объект хранилища Служб восстановления резервных копий удобно хранить в переменной.
   >
   >

## <a name="view-the-vaults-in-a-subscription"></a>Просмотр хранилищ в подписке

Чтобы просмотреть все хранилища в подписке, используйте команду [Get-азрековерисервицесваулт](/powershell/module/az.recoveryservices/get-azrecoveryservicesvault):

```powershell
Get-AzRecoveryServicesVault
```

Результат будет аналогичен приведенному ниже; обратите внимание, что указаны соответствующие ResourceGroupName и Location.

```output
Name              : Contoso-vault
ID                : /subscriptions/1234
Type              : Microsoft.RecoveryServices/vaults
Location          : WestUS
ResourceGroupName : Contoso-docs-rg
SubscriptionId    : 1234-567f-8910-abc
Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
```

## <a name="back-up-azure-vms"></a>Резервное копирование виртуальных машин Azure

Используйте хранилище служб восстановления для защиты виртуальной машины. Прежде чем применить защиту, задайте контекст хранилища (тип данных, защиту которых обеспечивает хранилище) и проверьте политику защиты. Политика защиты — это график выполнения заданий резервного копирования и срок хранения каждого моментального снимка резервной копии.

### <a name="set-vault-context"></a>Задание контекста хранилища

Перед включением защиты на виртуальной машине используйте [Set-азрековерисервицесваултконтекст](/powershell/module/az.recoveryservices/set-azrecoveryservicesvaultcontext) , чтобы задать контекст хранилища. Заданный контекст хранилища применяется ко всем последующим командлетам. В следующем примере задается контекст для хранилища *testvault*.

```powershell
Get-AzRecoveryServicesVault -Name "testvault" -ResourceGroupName "Contoso-docs-rg" | Set-AzRecoveryServicesVaultContext
```

### <a name="fetch-the-vault-id"></a>Получение идентификатора хранилища

Мы планируем использовать параметр контекста хранилища в соответствии с рекомендациями Azure PowerShell. Вместо этого можно сохранить или извлечь идентификатор хранилища и передать его в соответствующие команды. Поэтому, если вы не указали контекст хранилища или хотите указать команду для выполнения в определенном хранилище, передайте идентификатор хранилища как "-Ваултид" во всю соответствующую команду, как показано ниже:

```powershell
$targetVault = Get-AzRecoveryServicesVault -ResourceGroupName "Contoso-docs-rg" -Name "testvault"
$targetVault.ID
```

либо

```powershell
$targetVaultID = Get-AzRecoveryServicesVault -ResourceGroupName "Contoso-docs-rg" -Name "testvault" | select -ExpandProperty ID
```

### <a name="modifying-storage-replication-settings"></a>Изменение параметров репликации хранилища

Используйте команду [Set-азрековерисервицесбаккуппроперти](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupproperty) , чтобы задать для хранилища конфигурацию репликации хранилища LRS/GRS

```powershell
Set-AzRecoveryServicesBackupProperty -Vault $targetVault -BackupStorageRedundancy GeoRedundant/LocallyRedundant
```

> [!NOTE]
> Избыточность хранилища можно изменить только в том случае, если в хранилище нет защищенных резервных копий.

### <a name="create-a-protection-policy"></a>Создание политики защиты

Создаваемое хранилище служб восстановления поставляется с политиками защиты и хранения по умолчанию. Политика защиты по умолчанию запускает задание резервного копирования каждый день в указанное время. Политика хранения по умолчанию хранит ежедневную точку восстановления в течение 30 дней. С помощью политики по умолчанию можно быстро защитить виртуальную машину. Кроме того, политику можно изменить позже, задав другие сведения.

Используйте **[Get-азрековерисервицесбаккуппротектионполици](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupprotectionpolicy)** для просмотра политик защиты, доступных в хранилище. С его помощью можно получить конкретную политику или просмотреть политики, связанные с типом рабочей нагрузки. В следующем примере возвращаются политики для типа рабочей нагрузки AzureVM.

```powershell
Get-AzRecoveryServicesBackupProtectionPolicy -WorkloadType "AzureVM" -VaultId $targetVault.ID
```

Вы должны увидеть результат, аналогичный приведенному ниже.

```output
Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
----                 ------------       -------------------- ----------                ----------
DefaultPolicy        AzureVM            AzureVM              4/14/2016 5:00:00 PM
```

> [!NOTE]
> Часовой пояс поля BackupTime в PowerShell — UTC. Однако при отображении времени архивации на портале Azure время меняется в соответствии с локальным часовым поясом.
>
>

Политика защиты архивации связана по крайней мере с одной политикой хранения. Политика хранения определяет продолжительность хранения точки восстановления до ее удаления.

* Для просмотра политики хранения по умолчанию используется командлет [Get-AzRecoveryServicesBackupRetentionPolicyObject](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupretentionpolicyobject).
* Аналогичным образом можно использовать [Get-азрековерисервицесбаккупсчедулеполициобжект](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupschedulepolicyobject) для получения политики расписания по умолчанию.
* Командлет [New-AzRecoveryServicesBackupProtectionPolicy](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupprotectionpolicy) создает объект PowerShell, который содержит сведения о политике архивации.
* Объекты политик расписания и хранения используются в качестве входных данных в командлете New-AzRecoveryServicesBackupProtectionPolicy.

По умолчанию время начала определяется в объекте политики Schedule. Используйте следующий пример, чтобы изменить время начала на нужное время начала. Требуемое время начала должно быть также в формате UTC. В следующем примере предполагается, что требуемое время начала для ежедневного резервного копирования составляет 01:00 часов по ГРИНВИЧу.

```powershell
$schPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "AzureVM"
$UtcTime = Get-Date -Date "2019-03-20 01:00:00Z"
$UtcTime = $UtcTime.ToUniversalTime()
$schpol.ScheduleRunTimes[0] = $UtcTime
```

> [!IMPORTANT]
> Необходимо указать время начала в 30 минут, кратное. В приведенном выше примере это может быть только "01:00:00" или "02:30:00". Время начала не может быть "01:15:00"

В следующем примере показано сохранение политик расписания и хранения в переменных. В примере эти переменные используются для определения параметров при создании политики защиты *NewPolicy*.

```powershell
$retPol = Get-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType "AzureVM"
New-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -WorkloadType "AzureVM" -RetentionPolicy $retPol -SchedulePolicy $schPol -VaultId $targetVault.ID
```

Вы должны увидеть результат, аналогичный приведенному ниже.

```output
Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
----                 ------------       -------------------- ----------                ----------
NewPolicy           AzureVM            AzureVM              4/24/2016 1:30:00 AM
```

### <a name="enable-protection"></a>Включить защиту

После определения политики защиты по-прежнему необходимо включить политику для элемента. Используйте [Enable-азрековерисервицесбаккуппротектион](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection) , чтобы включить защиту. Защита включается для двух объектов: элемента и политики. После того как политика сопоставится с хранилищем, рабочий процесс архивации будет активироваться по времени, определенному в политике расписания.

> [!IMPORTANT]
> При использовании PowerShell для одновременного включения резервного копирования нескольких виртуальных машин убедитесь, что с одной политикой не связано более 100 виртуальных машин. Это [рекомендуемый метод](./backup-azure-vm-backup-faq.yml#is-there-a-limit-on-number-of-vms-that-can-be-associated-with-the-same-backup-policy). В настоящее время клиент PowerShell не выполняет явную блокировку при наличии более 100 виртуальных машин, но в будущем планируется добавить проверку.

В следующих примерах включается защита для элемента V2VM с помощью политики NewPolicy. Примеры отличаются в зависимости от того, зашифрована ли виртуальная машина и какой у нее тип шифрования.

Включение защиты на **виртуальных машинах Resource Manager без шифрования**

```powershell
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -VaultId $targetVault.ID
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1" -VaultId $targetVault.ID
```

Чтобы включить защиту на зашифрованных виртуальных машинах (зашифрованных с помощью BEK и KEK), необходимо предоставить службе архивации Azure разрешения на доступ для чтения ключей и секретов из хранилища ключей.

```powershell
Set-AzKeyVaultAccessPolicy -VaultName "KeyVaultName" -ResourceGroupName "RGNameOfKeyVault" -PermissionsToKeys backup,get,list -PermissionsToSecrets get,list -ServicePrincipalName 262044b1-e2ce-469f-a196-69ab7ada62d3
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -VaultId $targetVault.ID
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1" -VaultId $targetVault.ID
```

Чтобы включить защиту на **зашифрованных виртуальных машинах (зашифрованных только с помощью BEK)**, необходимо предоставить службе архивации Azure разрешения на доступ для чтения секретов из хранилища ключей.

```powershell
Set-AzKeyVaultAccessPolicy -VaultName "KeyVaultName" -ResourceGroupName "RGNameOfKeyVault" -PermissionsToSecrets backup,get,list -ServicePrincipalName 262044b1-e2ce-469f-a196-69ab7ada62d3
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -VaultId $targetVault.ID
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1" -VaultId $targetVault.ID
```

> [!NOTE]
> Если вы используете облако Azure для государственных организаций, используйте значение `ff281ffe-705c-4f53-9f37-a40e6f2c68f3` параметра "перестройка  " в командлете [Set-азкэйваултакцессполици](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy) .
>

Если вы хотите выборочно создать резервную копию нескольких дисков и исключить другие, как упоминалось в [этих сценариях](selective-disk-backup-restore.md#scenarios), можно настроить защиту и резервное копирование только соответствующих дисков, как описано [здесь](selective-disk-backup-restore.md#enable-backup-with-powershell).

## <a name="monitoring-a-backup-job"></a>Наблюдение за выполнением задания резервного копирования

Вы можете отслеживать длительные операции, например задания резервного копирования, без использования портала Azure. Чтобы получить состояние выполняемого задания, используйте командлет [Get-азрековерисервицесбаккупжоб](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupjob) . Этот командлет возвращает задания резервного копирования для конкретного хранилища, и это хранилище указывается в контексте хранилища. Следующий пример возвращает состояние выполняющегося задания в виде массива и сохраняет состояние в переменной $joblist.

```powershell
$joblist = Get-AzRecoveryservicesBackupJob –Status "InProgress" -VaultId $targetVault.ID
$joblist[0]
```

Вы должны увидеть результат, аналогичный приведенному ниже.

```output
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   ----------
V2VM             Backup               InProgress            4/23/2016                5:00:30 PM                cf4b3ef5-2fac-4c8e-a215-d2eba4124f27
```

Вместо опроса этих заданий на завершение — это необязательный дополнительный код — используйте командлет [Wait-азрековерисервицесбаккупжоб](/powershell/module/az.recoveryservices/wait-azrecoveryservicesbackupjob) . Он приостанавливает выполнение сценария до завершения задания или до достижения конкретного значения времени ожидания.

```powershell
Wait-AzRecoveryServicesBackupJob -Job $joblist[0] -Timeout 43200 -VaultId $targetVault.ID
```

## <a name="manage-azure-vm-backups"></a>Управление резервными копиями виртуальных машин Azure

### <a name="modify-a-protection-policy"></a>Изменение политики защиты

Чтобы изменить политику защиты, используйте [Set-азрековерисервицесбаккуппротектионполици](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupprotectionpolicy) , чтобы изменить объекты Счедулеполици или RetentionPolicy.

#### <a name="modifying-scheduled-time"></a>Изменение запланированного времени

При создании политики защиты ей по умолчанию назначается время запуска. В следующих примерах показано, как изменить время начала политики защиты.

````powershell
$SchPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "AzureVM"
$UtcTime = Get-Date -Date "2019-03-20 01:00:00Z" (This is the time that you want to start the backup)
$UtcTime = $UtcTime.ToUniversalTime()
$SchPol.ScheduleRunTimes[0] = $UtcTime
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -VaultId $targetVault.ID
Set-AzRecoveryServicesBackupProtectionPolicy -Policy $pol  -SchedulePolicy $SchPol -VaultId $targetVault.ID
````

#### <a name="modifying-retention"></a>Изменение срока хранения

В следующем примере количество дней хранения точки восстановления изменяется на 365.

```powershell
$retPol = Get-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType "AzureVM"
$retPol.DailySchedule.DurationCountInDays = 365
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -VaultId $targetVault.ID
Set-AzRecoveryServicesBackupProtectionPolicy -Policy $pol  -RetentionPolicy $RetPol -VaultId $targetVault.ID
```

#### <a name="configuring-instant-restore-snapshot-retention"></a>Настройка хранения моментальных снимков мгновенного восстановления

> [!NOTE]
> Начиная с версии Azure PowerShell 1.6.0, можно обновить срок хранения моментальных снимков мгновенного восстановления в политике с помощью PowerShell.

````powershell
$bkpPol = Get-AzRecoveryServicesBackupProtectionPolicy -WorkloadType "AzureVM" -VaultId $targetVault.ID
$bkpPol.SnapshotRetentionInDays=7
Set-AzRecoveryServicesBackupProtectionPolicy -policy $bkpPol -VaultId $targetVault.ID
````

Значение по умолчанию будет равно 2. Можно задать значение не менее 1 и не более 5. Для политик еженедельного резервного копирования точка имеет значение 5 и не может быть изменена.

#### <a name="creating-azure-backup-resource-group-during-snapshot-retention"></a>Создание Azure Backup группы ресурсов во время хранения моментального снимка

> [!NOTE]
> Начиная с версии Azure PowerShell 3.7.0, можно создать и изменить группу ресурсов, созданную для хранения мгновенных моментальных снимков.

Дополнительные сведения о правилах создания групп ресурсов и других релевантных деталях см. в документации по [Azure Backup группе ресурсов для виртуальных машин](./backup-during-vm-creation.md#azure-backup-resource-group-for-virtual-machines) .

```powershell
$bkpPol = Get-AzureRmRecoveryServicesBackupProtectionPolicy -name "DefaultPolicyForVMs"
$bkpPol.AzureBackupRGName="Contosto_"
$bkpPol.AzureBackupRGNameSuffix="ForVMs"
Set-AzureRmRecoveryServicesBackupProtectionPolicy -policy $bkpPol
```

### <a name="exclude-disks-for-a-protected-vm"></a>Исключение дисков для защищенной виртуальной машины

Служба архивации виртуальных машин Azure позволяет выборочно исключать или включать диски, которые полезны в [этих сценариях](selective-disk-backup-restore.md#scenarios). Если виртуальная машина уже защищена с помощью резервного копирования виртуальных машин Azure и при этом выполняется резервное копирование всех дисков, можно изменить защиту, чтобы выборочно включить или исключить диски, как описано [здесь](selective-disk-backup-restore.md#modify-protection-for-already-backed-up-vms-with-powershell).

### <a name="trigger-a-backup"></a>Активация архивации

Используйте [BACKUP-азрековерисервицесбаккупитем](/powershell/module/az.recoveryservices/backup-azrecoveryservicesbackupitem) для запуска задания резервного копирования. Если это начальная архивация, она будет полной. При последующем выполнении архивации резервная копия будет добавочной. В следующем примере резервная копия виртуальной машины будет храниться в течение 60 дней.

```powershell
$namedContainer = Get-AzRecoveryServicesBackupContainer -ContainerType "AzureVM" -Status "Registered" -FriendlyName "V2VM" -VaultId $targetVault.ID
$item = Get-AzRecoveryServicesBackupItem -Container $namedContainer -WorkloadType "AzureVM" -VaultId $targetVault.ID
$endDate = (Get-Date).AddDays(60).ToUniversalTime()
$job = Backup-AzRecoveryServicesBackupItem -Item $item -VaultId $targetVault.ID -ExpiryDateTimeUTC $endDate
```

Вы должны увидеть результат, аналогичный приведенному ниже.

```output
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   ----------
V2VM              Backup              InProgress          4/23/2016                  5:00:30 PM                cf4b3ef5-2fac-4c8e-a215-d2eba4124f27
```

> [!NOTE]
> Часовой пояс для полей StartTime и EndTime в PowerShell — UTC. Однако при отображении времени на портале Azure время меняется в соответствии с локальным часовым поясом.
>
>

### <a name="change-policy-for-backup-items"></a>Изменение политики для элементов архивации

Можно изменить существующую политику или изменить политику резервного элемента с Policy1 на Policy2. Чтобы переключить политики для архивированного элемента, извлеките соответствующую политику и элемент резервного копирования и используйте команду [Enable-азрековерисервицес](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection) с элементом Backup в качестве параметра.

````powershell
$TargetPol1 = Get-AzRecoveryServicesBackupProtectionPolicy -Name <PolicyName> -VaultId $targetVault.ID
$anotherBkpItem = Get-AzRecoveryServicesBackupItem -WorkloadType AzureVM -BackupManagementType AzureVM -Name "<BackupItemName>" -VaultId $targetVault.ID
Enable-AzRecoveryServicesBackupProtection -Item $anotherBkpItem -Policy $TargetPol1 -VaultId $targetVault.ID
````

Команда ожидает, пока не завершится Настройка резервного копирования и вернет следующие выходные данные.

```powershell
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
TestVM           ConfigureBackup      Completed            3/18/2019 8:00:21 PM      3/18/2019 8:02:16 PM      654e8aa2-4096-402b-b5a9-e5e71a496c4e
```

### <a name="stop-protection"></a>Остановить защиту

#### <a name="retain-data"></a>Сохранение данных

Если вы хотите отключить защиту, можно использовать командлет PowerShell [Disable-азрековерисервицесбаккуппротектион](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupprotection) . Это приведет к отмене запланированных резервных копий, но данные будут сохранены до тех пор, пока не будет храниться неограниченное время.

````powershell
$bkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureVM -WorkloadType AzureVM -Name "<backup item name>" -VaultId $targetVault.ID
Disable-AzRecoveryServicesBackupProtection -Item $bkpItem -VaultId $targetVault.ID
````

#### <a name="delete-backup-data"></a>удаление резервных копий;

Чтобы полностью удалить сохраненные резервные копии данных в хранилище, добавьте флаг "-Ремоверековерипоинтс" или параметр в [команду "отключить"](#retain-data).

````powershell
Disable-AzRecoveryServicesBackupProtection -Item $bkpItem -VaultId $targetVault.ID -RemoveRecoveryPoints
````

## <a name="restore-an-azure-vm"></a>Восстановление виртуальной машины Azure

Существует важное различие между восстановлением виртуальной машины с помощью портал Azure и восстановлением виртуальной машины с помощью PowerShell. При использовании PowerShell операция восстановления завершается созданием дисков и сведений о конфигурации из точки восстановления. Виртуальная машина при операции восстановления не создается. Чтобы создать виртуальную машину с диска, см. раздел [Создание виртуальной машины с восстановленного диска](backup-azure-vms-automation.md#create-a-vm-from-restored-disks). Чтобы восстановить несколько файлов из резервной копии виртуальной машины Azure без восстановления всей виртуальной машины, изучите раздел о [восстановлении файлов](backup-azure-vms-automation.md#restore-files-from-an-azure-vm-backup).

> [!Tip]
> Виртуальная машина при операции восстановления не создается.
>
>

На представленной ниже схеме показана иерархия объектов от RecoveryServicesVault до BackupRecoveryPoint.

![Иерархия объектов служб восстановления с BackupContainer](./media/backup-azure-vms-arm-automation/backuprecoverypoint-only.png)

Чтобы восстановить данные резервной копии, определите архивный элемент и точку восстановления, которая содержит данные на определенный момент времени. Используйте [RESTORE-азрековерисервицесбаккупитем](/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem) для восстановления данных из хранилища в учетную запись.

Ниже перечислены основные шаги для восстановления виртуальной машины Azure.

* Выберите виртуальную машину.
* Выберите точку восстановления.
* Восстановите диски.
* Создайте виртуальную машину с хранимых дисков.

### <a name="select-the-vm-when-restoring-files"></a>Выбор виртуальной машины (при восстановлении файлов)

Чтобы получить объект PowerShell, определяющий правильный архивный элемент, начните с контейнера в хранилище и пройдите постепенно вниз по иерархии объектов. Чтобы выбрать контейнер, представляющий виртуальную машину, используйте командлет [Get-азрековерисервицесбаккупконтаинер](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupcontainer) и конвейер, который будет использоваться в командлете [Get-азрековерисервицесбаккупитем](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupitem) .

```powershell
$namedContainer = Get-AzRecoveryServicesBackupContainer  -ContainerType "AzureVM" -Status "Registered" -FriendlyName "V2VM" -VaultId $targetVault.ID
$backupitem = Get-AzRecoveryServicesBackupItem -Container $namedContainer  -WorkloadType "AzureVM" -VaultId $targetVault.ID
```

### <a name="choose-a-recovery-point-when-restoring-files"></a>Выбор точки восстановления (при восстановлении файлов)

Выполните командлет [Get-AzRecoveryServicesBackupRecoveryPoint](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprecoverypoint), чтобы получить полный список точек восстановления. Выберите нужную точку восстановления. Если вы не уверены, какую точку восстановления следует использовать, рекомендуется выбрать последнюю выбрать = Аппконсистент в списке.

В следующем сценарии переменная **$rp** представляет собой массив точек восстановления для выбранного архивного элемента за последние семь дней. Массив сортируется по времени в обратном порядке, так что последняя точка восстановления получает индекс 0. Используйте стандартное индексирование массива PowerShell для выбора точки восстановления. Например, $rp[0] обозначает последнюю точку восстановления.

```powershell
$startDate = (Get-Date).AddDays(-7)
$endDate = Get-Date
$rp = Get-AzRecoveryServicesBackupRecoveryPoint -Item $backupitem -StartDate $startdate.ToUniversalTime() -EndDate $enddate.ToUniversalTime() -VaultId $targetVault.ID
$rp[0]
```

Вы должны увидеть результат, аналогичный приведенному ниже.

```output
RecoveryPointAdditionalInfo :
SourceVMStorageType         : NormalStorage
Name                        : 15260861925810
ItemName                    : VM;iaasvmcontainer;RGName1;V2VM
RecoveryPointId             : /subscriptions/XX/resourceGroups/ RGName1/providers/Microsoft.RecoveryServices/vaults/testvault/backupFabrics/Azure/protectionContainers/IaasVMContainer;iaasvmcontainer;RGName1;V2VM/protectedItems/VM;iaasvmcontainer; RGName1;V2VM/recoveryPoints/15260861925810
RecoveryPointType           : AppConsistent
RecoveryPointTime           : 4/23/2016 5:02:04 PM
WorkloadType                : AzureVM
ContainerName               : IaasVMContainer;iaasvmcontainer; RGName1;V2VM
ContainerType               : AzureVM
BackupManagementType        : AzureVM
```

### <a name="restore-the-disks"></a>Восстановление дисков

Используйте командлет [RESTORE-азрековерисервицесбаккупитем](/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem) , чтобы восстановить данные и конфигурацию элемента резервного копирования в точку восстановления. Используйте выбранную точку восстановления как значение для параметра **-RecoveryPoint**. В приведенном выше примере используемой точкой восстановления является **$RP [0]** . В следующем примере кода **$rp[0]** является точкой восстановления, которая используется для восстановления диска.

Восстановление дисков и сведений о конфигурации

```powershell
$restorejob = Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -StorageAccountName "DestAccount" -StorageAccountResourceGroupName "DestRG" -VaultId $targetVault.ID
$restorejob
```

#### <a name="restore-managed-disks"></a>Восстановление управляемых дисков

> [!NOTE]
> Если у виртуальной машины есть управляемые диски и вы хотите восстановить их как управляемые диски, мы предоставили эту возможность из модуля Azure PowerShell RM v 6.7.0. Начиная.
>
>

Укажите дополнительный параметр **TargetResourceGroupName** для указания группы ресурсов, в которой будут восстановлены управляемые диски.

> [!IMPORTANT]
> Настоятельно рекомендуется использовать параметр **TargetResourceGroupName** для восстановления управляемых дисков, так как это приводит к значительному улучшению производительности. Если этот параметр не указан, вы не сможете воспользоваться функцией мгновенного восстановления, и операция восстановления будет выполняться медленнее. Если целью является восстановление управляемых дисков в виде неуправляемых дисков, не следует указывать этот параметр и сделать намерением ясно, предоставив `-RestoreAsUnmanagedDisks` параметр. `-RestoreAsUnmanagedDisks`Параметр доступен в Azure PowerShell 3.7.0/назад. В будущих версиях будет обязательно указывать один из этих параметров для правильного восстановления.
>
>

```powershell
$restorejob = Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -StorageAccountName "DestAccount" -StorageAccountResourceGroupName "DestRG" -TargetResourceGroupName "DestRGforManagedDisks" -VaultId $targetVault.ID
```

Файл **VMConfig.JSON** будет восстановлен в учетной записи хранения, а управляемые диски — в указанной целевой группе ресурсов.

Вы должны увидеть результат, аналогичный приведенному ниже.

```output
WorkloadName     Operation          Status               StartTime                 EndTime            JobID
------------     ---------          ------               ---------                 -------          ----------
V2VM              Restore           InProgress           4/23/2016 5:00:30 PM                        cf4b3ef5-2fac-4c8e-a215-d2eba4124f27
```

Чтобы дождаться завершения задания восстановления, используйте командлет [Wait-азрековерисервицесбаккупжоб](/powershell/module/az.recoveryservices/wait-azrecoveryservicesbackupjob) .

```powershell
Wait-AzRecoveryServicesBackupJob -Job $restorejob -Timeout 43200
```

После завершения задания восстановления используйте командлет [Get-азрековерисервицесбаккупжобдетаилс](/powershell/module/az.recoveryservices/wait-azrecoveryservicesbackupjob) , чтобы получить сведения об операции восстановления. Свойство JobDetails содержит сведения, необходимые для повторного создания виртуальной машины.

```powershell
$restorejob = Get-AzRecoveryServicesBackupJob -Job $restorejob -VaultId $targetVault.ID
$details = Get-AzRecoveryServicesBackupJobDetails -Job $restorejob -VaultId $targetVault.ID
```

#### <a name="restore-selective-disks"></a>Восстановление выборочных дисков

Пользователь может выборочно восстановить несколько дисков вместо полного резервного набора данных. Укажите необходимый диск LUN в качестве параметра, чтобы восстановить их только вместо всего набора, как описано [здесь](selective-disk-backup-restore.md#restore-selective-disks-with-powershell).

> [!IMPORTANT]
> Один из них должен выборочно выполнять резервное копирование дисков для выборочного восстановления дисков. Дополнительные сведения приведены [здесь](selective-disk-backup-restore.md#selective-disk-restore).

Восстановив диски, перейдите к следующему разделу по созданию виртуальной машины.

#### <a name="restore-disks-to-a-secondary-region"></a>Восстановление дисков в дополнительный регион

Если в хранилище, с которым защищены виртуальные машины, включено восстановление между регионами, резервные копии данных реплицируются в дополнительный регион. Данные резервного копирования можно использовать для выполнения восстановления. Чтобы активировать восстановление в дополнительном регионе, выполните следующие действия.

1. Получите [идентификатор хранилища](#fetch-the-vault-id) , с которым защищены виртуальные машины.
1. Выберите [правильный элемент резервного копирования для восстановления](#select-the-vm-when-restoring-files).
1. Выберите соответствующую точку восстановления в дополнительном регионе, который будет использоваться для восстановления.

    Чтобы выполнить этот шаг, выполните следующую команду:

    ```powershell
    $rp=Get-AzRecoveryServicesBackupRecoveryPoint -UseSecondaryRegion -Item $backupitem -VaultId $targetVault.ID
    $rp=$rp[0]
    ```

1. Выполните командлет [RESTORE-азрековерисервицесбаккупитем](/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem) с `-RestoreToSecondaryRegion` параметром, чтобы активировать восстановление в дополнительном регионе.

    Чтобы выполнить этот шаг, выполните следующую команду:

    ```powershell
    $restorejob = Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -StorageAccountName "DestAccount" -StorageAccountResourceGroupName "DestRG" -TargetResourceGroupName "DestRGforManagedDisks" -VaultId $targetVault.ID -VaultLocation $targetVault.Location -RestoreToSecondaryRegion -RestoreOnlyOSDisk
    ```

    Результат будет выглядеть примерно так:

    ```output
    WorkloadName     Operation             Status              StartTime                 EndTime          JobID
    ------------     ---------             ------              ---------                 -------          ----------
    V2VM             CrossRegionRestore   InProgress           4/23/2016 5:00:30 PM                       cf4b3ef5-2fac-4c8e-a215-d2eba4124f27
    ```

1. Выполните командлет [Get-азрековерисервицесбаккупжоб](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupjob) с `-UseSecondaryRegion` параметром для отслеживания задания восстановления.

    Чтобы выполнить этот шаг, выполните следующую команду:

    ```powershell
    Get-AzRecoveryServicesBackupJob -From (Get-Date).AddDays(-7).ToUniversalTime() -To (Get-Date).ToUniversalTime() -UseSecondaryRegion -VaultId $targetVault.ID
    ```

    Результат будет выглядеть примерно так:

    ```output
    WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
    ------------     ---------            ------               ---------                 -------                   -----
    V2VM             CrossRegionRestore   InProgress           2/8/2021 4:24:57 PM                                 2d071b07-8f7c-4368-bc39-98c7fb2983f7
    ```

## <a name="replace-disks-in-azure-vm"></a>Замена дисков в виртуальной машине Azure

Чтобы заменить диски и сведения о конфигурации, выполните следующие действия.

* Шаг 1. [Восстановление дисков](backup-azure-vms-automation.md#restore-the-disks)
* Шаг 2. [Отключение диска данных с помощью PowerShell](../virtual-machines/windows/detach-disk.md#detach-a-data-disk-using-powershell)
* Шаг 3. [Подключение диска данных к виртуальной машине Windows с помощью PowerShell](../virtual-machines/windows/attach-disk-ps.md)

## <a name="create-a-vm-from-restored-disks"></a>Создание виртуальной машины с восстановленного диска

После восстановления дисков создайте и настройте виртуальную машину с диска, выполнив описанные ниже действия.

> [!NOTE]
>
> 1. Требуется модуль Азуреаз 3.0.0 или более поздней версии. <br>
> 2. При создании зашифрованных виртуальных машин с помощью восстановленных дисков у роли Azure должно быть разрешение на выполнение действия **Microsoft.KeyVault/vaults/deploy/action**. Если у роли нет этого разрешения, создайте настраиваемую роль с этим действием. Дополнительные сведения см. в статье [пользовательские роли Azure](../role-based-access-control/custom-roles.md). <br>
> 3. После восстановления дисков можно получить шаблон развертывания, который можно использовать непосредственно для создания виртуальной машины. Для создания управляемых и неуправляемых виртуальных машин, которые шифруются и не шифруются, не требуются разные командлеты PowerShell.<br>
> <br>

### <a name="create-a-vm-using-the-deployment-template"></a>Создание виртуальной машины с помощью шаблона развертывания

В подробных сведениях о результирующем задании указан URI шаблона, который можно получить и развернуть.

```powershell
   $properties = $details.properties
   $storageAccountName = $properties["Target Storage Account Name"]
   $containerName = $properties["Config Blob Container Name"]
   $templateBlobURI = $properties["Template Blob Uri"]
```

Этот шаблон недоступен напрямую, так как он находится в учетной записи хранения клиента, в указанном контейнере. Для доступа к этому шаблону требуется полный URL-адрес (вместе с временным маркером SAS).

1. Сначала извлеките имя шаблона из Темплатеблобури. Этот формат упоминается ниже. Для извлечения окончательного имени шаблона из этого URL-адреса можно использовать операцию разбиения в PowerShell.

    ```http
    https://<storageAccountName.blob.core.windows.net>/<containerName>/<templateName>
    ```

2. Затем можно создать полный URL-адрес, как описано [здесь](../azure-resource-manager/templates/secure-template-with-sas-token.md?tabs=azure-powershell#provide-sas-token-during-deployment).

    ```powershell
    Set-AzCurrentStorageAccount -Name $storageAccountName -ResourceGroupName <StorageAccount RG name>
    $templateBlobFullURI = New-AzStorageBlobSASToken -Container $containerName -Blob <templateName> -Permission r -FullUri
    ```

3. Разверните шаблон, чтобы создать новую виртуальную машину, как описано [здесь](../azure-resource-manager/templates/deploy-powershell.md).

    ```powershell
    New-AzResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateUri $templateBlobFullURI
    ```

### <a name="create-a-vm-using-the-config-file"></a>Создание виртуальной машины с помощью файла конфигурации

В следующем разделе перечислены шаги, необходимые для создания виртуальной машины с помощью файла VMConfig.

> [!NOTE]
> Для создания виртуальной машины настоятельно рекомендуется использовать шаблон развертывания, описанный выше. Действия, указанные в этом разделе (пункты 1–6), скоро перестанут использоваться.

1. Запросите свойства восстановленного диска для получения сведений о задании.

   ```powershell
   $properties = $details.properties
   $storageAccountName = $properties["Target Storage Account Name"]
   $containerName = $properties["Config Blob Container Name"]
   $configBlobName = $properties["Config Blob Name"]
   ```

2. Задайте контекст хранилища Azure и восстановите файл конфигурации JSON.

   ```powershell
   Set-AzCurrentStorageAccount -Name $storageaccountname -ResourceGroupName "testvault"
   $destination_path = "C:\vmconfig.json"
   Get-AzStorageBlobContent -Container $containerName -Blob $configBlobName -Destination $destination_path
   $obj = ((Get-Content -Path $destination_path -Raw -Encoding Unicode)).TrimEnd([char]0x00) | ConvertFrom-Json
   ```

3. Используйте файл конфигурации JSON для создания конфигурации виртуальной машины.

   ```powershell
   $vm = New-AzVMConfig -VMSize $obj.'properties.hardwareProfile'.vmSize -VMName "testrestore"
   ```

4. Подключите диск операционной системы и диски данных. Здесь приводятся примеры для различных конфигураций управляемых и зашифрованных виртуальных машин. Используйте пример, подходящий для вашей конфигурации виртуальных машин.

    * Используйте следующий пример для **неуправляемой виртуальной машины без шифрования**.

    ```powershell
        Set-AzVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.StorageProfile'.osDisk.vhd.Uri -CreateOption "Attach"
        $vm.StorageProfile.OsDisk.OsType = $obj.'properties.StorageProfile'.OsDisk.OsType
        foreach($dd in $obj.'properties.StorageProfile'.DataDisks)
        {
            $vm = Add-AzVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
        }
    ```

    * **Для неуправляемых зашифрованных виртуальных машин с Azure AD (зашифрованных только с помощью BEK)** необходимо восстановить секрет в хранилище ключей до подключения дисков. Дополнительные сведения см. в статье [Восстановление ключа и секрета в хранилище ключей для зашифрованных виртуальных машин с помощью службы Azure Backup](backup-azure-restore-key-secret.md). В следующем примере показано, как подключить диски операционной системы и диски данных к зашифрованной виртуальной машине. При указании диска операционной системы указывайте соответствующий тип операционной системы.

    ```powershell
        $dekUrl = "https://ContosoKeyVault.vault.azure.net:443/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
        $dekUrl = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
        Set-AzVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.storageProfile'.osDisk.vhd.uri -DiskEncryptionKeyUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -CreateOption "Attach" -Windows/Linux
        $vm.StorageProfile.OsDisk.OsType = $obj.'properties.storageProfile'.osDisk.osType
        foreach($dd in $obj.'properties.storageProfile'.dataDisks)
        {
        $vm = Add-AzVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
        }
    ```

    * **Для неуправляемых зашифрованных виртуальных машин с Azure AD (зашифрованных с помощью BEK и KEK)** необходимо восстановить ключ и секрет в хранилище ключей до подключения дисков. Дополнительные сведения см. в статье [Восстановление ключа и секрета в хранилище ключей для зашифрованных виртуальных машин с помощью службы Azure Backup](backup-azure-restore-key-secret.md). В следующем примере показано, как подключить диски операционной системы и диски данных к зашифрованной виртуальной машине.

    ```powershell
        $dekUrl = "https://ContosoKeyVault.vault.azure.net:443/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
        $kekUrl = "https://ContosoKeyVault.vault.azure.net:443/keys/ContosoKey007/x9xxx00000x0000x9b9949999xx0x006"
        $keyVaultId = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
        Set-AzVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.storageProfile'.osDisk.vhd.uri -DiskEncryptionKeyUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -KeyEncryptionKeyUrl $kekUrl -KeyEncryptionKeyVaultId $keyVaultId -CreateOption "Attach" -Windows
        $vm.StorageProfile.OsDisk.OsType = $obj.'properties.storageProfile'.osDisk.osType
        foreach($dd in $obj.'properties.storageProfile'.dataDisks)
        {
        $vm = Add-AzVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
        }
    ```

    * **Для неуправляемых зашифрованных виртуальных машин без Azure AD (зашифрованных только с помощью BEK)**, если в исходном **хранилище ключей недоступны секреты**, восстановите секреты в хранилище ключей, выполнив действия, описанные в статье [Восстановление ключа и секрета в хранилище ключей для зашифрованных виртуальных машин с помощью службы Azure Backup](backup-azure-restore-key-secret.md). Затем выполните следующие скрипты, чтобы задать сведения о шифровании для восстановленного большого двоичного объекта ОС (этот шаг не требуется для больших двоичных объектов данных). $dekurl можно извлечь из восстановленного хранилища ключей.

    Следующий скрипт должен выполняться, только если исходный keyVault/секрет недоступен.

    ```powershell
        $dekUrl = "https://ContosoKeyVault.vault.azure.net/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
        $keyVaultId = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
        $encSetting = "{""encryptionEnabled"":true,""encryptionSettings"":[{""diskEncryptionKey"":{""sourceVault"":{""id"":""$keyVaultId""},""secretUrl"":""$dekUrl""}}]}"
        $osBlobName = $obj.'properties.StorageProfile'.osDisk.name + ".vhd"
        $osBlob = Get-AzStorageBlob -Container $containerName -Blob $osBlobName
        $osBlob.ICloudBlob.Metadata["DiskEncryptionSettings"] = $encSetting
        $osBlob.ICloudBlob.SetMetadata()
    ```

    После того как **секреты станут доступны** и будут установлены сведения о шифровании для большого двоичного объекта операционной системы, подключите диски, используя скрипт, приведенный ниже.

    Если источник keyVault/секреты уже доступны, приведенный выше сценарий не должен выполняться.

    ```powershell
        Set-AzVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.StorageProfile'.osDisk.vhd.Uri -CreateOption "Attach"
        $vm.StorageProfile.OsDisk.OsType = $obj.'properties.StorageProfile'.OsDisk.OsType
        foreach($dd in $obj.'properties.StorageProfile'.DataDisks)
        {
        $vm = Add-AzVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
        }
    ```

    * **Для неуправляемых зашифрованных виртуальных машин без Azure AD (зашифрованных с помощью BEK и KEK)**, если в исходном **хранилище ключей недоступны ключи и секреты**, восстановите ключи и секреты в хранилище ключей, выполнив действия, описанные в статье [Восстановление ключа и секрета в хранилище ключей для зашифрованных виртуальных машин с помощью службы Azure Backup](backup-azure-restore-key-secret.md). Затем выполните следующие скрипты, чтобы задать сведения о шифровании для восстановленного большого двоичного объекта ОС (этот шаг не требуется для больших двоичных объектов данных). $dekurl и $kekurl можно извлечь из восстановленного хранилища ключей.

    Приведенный ниже сценарий должен выполняться, только если исходный keyVault/ключ или секрет недоступен.

    ```powershell
        $dekUrl = "https://ContosoKeyVault.vault.azure.net/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
        $kekUrl = "https://ContosoKeyVault.vault.azure.net/keys/ContosoKey007/x9xxx00000x0000x9b9949999xx0x006"
        $keyVaultId = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
        $encSetting = "{""encryptionEnabled"":true,""encryptionSettings"":[{""diskEncryptionKey"":{""sourceVault"":{""id"":""$keyVaultId""},""secretUrl"":""$dekUrl""},""keyEncryptionKey"":{""sourceVault"":{""id"":""$keyVaultId""},""keyUrl"":""$kekUrl""}}]}"
        $osBlobName = $obj.'properties.StorageProfile'.osDisk.name + ".vhd"
        $osBlob = Get-AzStorageBlob -Container $containerName -Blob $osBlobName
        $osBlob.ICloudBlob.Metadata["DiskEncryptionSettings"] = $encSetting
        $osBlob.ICloudBlob.SetMetadata()
    ```

    После того **как ключи или секреты станут доступны** и будут установлены сведения о шифровании для большого двоичного объекта операционной системы, подключите диски, используя скрипт, приведенный ниже.

    Если источник keyVault/ключ/секреты доступны, приведенный выше сценарий не должен выполняться.

    ```powershell
        Set-AzVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.StorageProfile'.osDisk.vhd.Uri -CreateOption "Attach"
        $vm.StorageProfile.OsDisk.OsType = $obj.'properties.StorageProfile'.OsDisk.OsType
        foreach($dd in $obj.'properties.StorageProfile'.DataDisks)
        {
        $vm = Add-AzVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
        }
    ```

    * **Управляемые и незашифрованные виртуальные машины**. Для управляемых незашифрованных виртуальных машин подключите восстановленные управляемые диски. Более подробные сведения см. в статье [Подключение диска данных к виртуальной машине Windows с помощью PowerShell](../virtual-machines/windows/attach-disk-ps.md).

    * **Для управляемых зашифрованных виртуальных машин с Azure AD (зашифрованных только с помощью BEK)** подключите восстановленные управляемые диски. Более подробные сведения см. в статье [Подключение диска данных к виртуальной машине Windows с помощью PowerShell](../virtual-machines/windows/attach-disk-ps.md).

    * **Для управляемых зашифрованных виртуальных машин с Azure AD (зашифрованных с помощью BEK и KEK)** подключите восстановленные управляемые диски. Более подробные сведения см. в статье [Подключение диска данных к виртуальной машине Windows с помощью PowerShell](../virtual-machines/windows/attach-disk-ps.md).

    * **Управляемые и зашифрованные виртуальные машины без Azure AD (только для BEK)** . для управляемых зашифрованных виртуальных машин без Azure AD (с шифрованием только с помощью BEK), если исходный **keyVault/секрет недоступен** , восстановите секреты в хранилище ключей, используя процедуру, описанную в разделе [Восстановление незашифрованной виртуальной машины из Azure Backup точки восстановления](backup-azure-restore-key-secret.md). Затем выполните следующие скрипты, чтобы задать сведения о шифровании восстановленного диска ОС (этот шаг не требуется для диска данных). $dekurl можно извлечь из восстановленного хранилища ключей.

    Приведенный ниже сценарий должен выполняться, только если исходный keyVault/секрет недоступен.  

    ```powershell
    $dekUrl = "https://ContosoKeyVault.vault.azure.net/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
    $keyVaultId = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
    $diskupdateconfig = New-AzDiskUpdateConfig -EncryptionSettingsEnabled $true
    $encryptionSettingsElement = New-Object Microsoft.Azure.Management.Compute.Models.EncryptionSettingsElement
    $encryptionSettingsElement.DiskEncryptionKey = New-Object Microsoft.Azure.Management.Compute.Models.KeyVaultAndSecretReference
    $encryptionSettingsElement.DiskEncryptionKey.SourceVault = New-Object Microsoft.Azure.Management.Compute.Models.SourceVault
    $encryptionSettingsElement.DiskEncryptionKey.SourceVault.Id = $keyVaultId
    $encryptionSettingsElement.DiskEncryptionKey.SecretUrl = $dekUrl
    $diskupdateconfig.EncryptionSettingsCollection.EncryptionSettings = New-Object System.Collections.Generic.List[Microsoft.Azure.Management.Compute.Models.EncryptionSettingsElement]
    $diskupdateconfig.EncryptionSettingsCollection.EncryptionSettings.Add($encryptionSettingsElement)
    $diskupdateconfig.EncryptionSettingsCollection.EncryptionSettingsVersion = "1.1"
    Update-AzDisk -ResourceGroupName "testvault" -DiskName $obj.'properties.StorageProfile'.osDisk.name -DiskUpdate $diskupdateconfig
    ```

    После того как секреты станут доступны и на диске ОС будут установлены сведения о шифровании, выполните действия для подключения восстановленных управляемых дисков, приведенные в статье [Подключение диска данных к виртуальной машине Windows с помощью PowerShell](../virtual-machines/windows/attach-disk-ps.md).

    * **Управляемые и зашифрованные виртуальные машины без Azure AD (BEK и KEK)** — для управляемых зашифрованных виртуальных машин без Azure AD (зашифрованных с помощью BEK & KEK). Если источник **keyVault/ключ/секрет недоступен** , восстановите ключ и секреты в хранилище ключей, используя процедуру, описанную в разделе [Восстановление незашифрованной виртуальной машины из Azure Backup точки восстановления](backup-azure-restore-key-secret.md). Затем выполните следующие скрипты, чтобы задать сведения о шифровании восстановленного диска ОС (этот шаг не требуется для дисков данных). $dekurl и $kekurl можно извлечь из восстановленного хранилища ключей.

    Следующий скрипт должен выполняться, только если исходный keyVault/ключ или секрет недоступен.

    ```powershell
    $dekUrl = "https://ContosoKeyVault.vault.azure.net/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
    $kekUrl = "https://ContosoKeyVault.vault.azure.net/keys/ContosoKey007/x9xxx00000x0000x9b9949999xx0x006"
    $keyVaultId = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
    $diskupdateconfig = New-AzDiskUpdateConfig -EncryptionSettingsEnabled $true
    $encryptionSettingsElement = New-Object Microsoft.Azure.Management.Compute.Models.EncryptionSettingsElement
    $encryptionSettingsElement.DiskEncryptionKey = New-Object Microsoft.Azure.Management.Compute.Models.KeyVaultAndSecretReference
    $encryptionSettingsElement.DiskEncryptionKey.SourceVault = New-Object Microsoft.Azure.Management.Compute.Models.SourceVault
    $encryptionSettingsElement.DiskEncryptionKey.SourceVault.Id = $keyVaultId
    $encryptionSettingsElement.DiskEncryptionKey.SecretUrl = $dekUrl
    $encryptionSettingsElement.KeyEncryptionKey = New-Object Microsoft.Azure.Management.Compute.Models.KeyVaultAndKeyReference
    $encryptionSettingsElement.KeyEncryptionKey.SourceVault = New-Object Microsoft.Azure.Management.Compute.Models.SourceVault
    $encryptionSettingsElement.KeyEncryptionKey.SourceVault.Id = $keyVaultId
    $encryptionSettingsElement.KeyEncryptionKey.KeyUrl = $kekUrl
    $diskupdateconfig.EncryptionSettingsCollection.EncryptionSettings = New-Object System.Collections.Generic.List[Microsoft.Azure.Management.Compute.Models.EncryptionSettingsElement]
    $diskupdateconfig.EncryptionSettingsCollection.EncryptionSettings.Add($encryptionSettingsElement)
    $diskupdateconfig.EncryptionSettingsCollection.EncryptionSettingsVersion = "1.1"
    Update-AzDisk -ResourceGroupName "testvault" -DiskName $obj.'properties.StorageProfile'.osDisk.name -DiskUpdate $diskupdateconfig
    ```

    После того как секреты станут доступны и на диске ОС будут установлены сведения о шифровании, выполните действия для подключения восстановленных управляемых дисков, приведенные в статье [Подключение диска данных к виртуальной машине Windows c помощью PowerShell](../virtual-machines/windows/attach-disk-ps.md).

5. Задайте параметры сети.

    ```powershell
    $nicName="p1234"
    $pip = New-AzPublicIpAddress -Name $nicName -ResourceGroupName "test" -Location "WestUS" -AllocationMethod Dynamic
    $virtualNetwork = New-AzVirtualNetwork -ResourceGroupName "test" -Location "WestUS" -Name "testvNET" -AddressPrefix 10.0.0.0/16
    $virtualNetwork | Set-AzVirtualNetwork
    $vnet = Get-AzVirtualNetwork -Name "testvNET" -ResourceGroupName "test"
    $subnetindex=0
    $nic = New-AzNetworkInterface -Name $nicName -ResourceGroupName "test" -Location "WestUS" -SubnetId $vnet.Subnets[$subnetindex].Id -PublicIpAddressId $pip.Id
    $vm=Add-AzVMNetworkInterface -VM $vm -Id $nic.Id
    ```

6. Создайте виртуальную машину.

    ```powershell  
    New-AzVM -ResourceGroupName "test" -Location "WestUS" -VM $vm
    ```

7. Отправьте расширение ADE.
   Если расширения ADE не отправляются, диски данных будут помечены как незашифрованные, поэтому для выполнения следующих действий обязательно выполните следующие действия.

   * **Для виртуальных машин с Azure AD** используйте следующую команду, чтобы вручную включить шифрование для дисков данных.  

     **Виртуальные машины, использующие только BEK**

      ```powershell  
      Set-AzVMDiskEncryptionExtension -ResourceGroupName $RG -VMName $vm.Name -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -VolumeType Data
      ```

     **Виртуальные машины, использующие BEK и KEK**

      ```powershell  
      Set-AzVMDiskEncryptionExtension -ResourceGroupName $RG -VMName $vm.Name -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId  -KeyEncryptionKeyUrl $kekUrl -KeyEncryptionKeyVaultId $keyVaultId -VolumeType Data
      ```

   * **Для виртуальных машин без Azure AD** используйте следующую команду, чтобы вручную включить шифрование для дисков данных.

     Если во время выполнения команды запрашивается Аадклиентид, необходимо обновить Azure PowerShell.

     **Виртуальные машины, использующие только BEK**

      ```powershell  
      Set-AzVMDiskEncryptionExtension -ResourceGroupName $RG -VMName $vm.Name -DiskEncryptionKeyVaultUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -SkipVmBackup -VolumeType "All"
      ```

      **Виртуальные машины, использующие BEK и KEK**

      ```powershell  
      Set-AzVMDiskEncryptionExtension -ResourceGroupName $RG -VMName $vm.Name -DiskEncryptionKeyVaultUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -KeyEncryptionKeyUrl $kekUrl -KeyEncryptionKeyVaultId $keyVaultId -SkipVmBackup -VolumeType "All"
      ```

> [!NOTE]
> Обязательно вручную удалите файлы Джейсон, созданные в процессе восстановления диска с зашифрованной виртуальной машиной.

## <a name="restore-files-from-an-azure-vm-backup"></a>Восстановление файлов из резервной копии виртуальной машины Azure

Из резервной копии виртуальной машины Azure можно восстанавливать не только диски, но и отдельные файлы. Функция восстановления отдельных файлов предоставляет доступ ко всем файлам в точке восстановления. Управление файлами ведется через проводник, как обычно.

Основные шаги по восстановлению файла из резервной копии виртуальной машины Azure:

* Выбор виртуальной машины.
* Выбор точки восстановления
* Подключение дисков точки восстановления
* Копирование необходимых файлов.
* Отключение диска.

### <a name="select-the-vm-when-restoring-the-vm"></a>Выбор виртуальной машины (при восстановлении виртуальной машины)

Чтобы получить объект PowerShell, определяющий правильный архивный элемент, начните с контейнера в хранилище и пройдите постепенно вниз по иерархии объектов. Чтобы выбрать контейнер, представляющий виртуальную машину, используйте командлет [Get-азрековерисервицесбаккупконтаинер](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupcontainer) и конвейер, который будет использоваться в командлете [Get-азрековерисервицесбаккупитем](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupitem) .

```powershell
$namedContainer = Get-AzRecoveryServicesBackupContainer  -ContainerType "AzureVM" -Status "Registered" -FriendlyName "V2VM" -VaultId $targetVault.ID
$backupitem = Get-AzRecoveryServicesBackupItem -Container $namedContainer  -WorkloadType "AzureVM" -VaultId $targetVault.ID
```

### <a name="choose-a-recovery-point-when-restoring-the-vm"></a>Выбор точки восстановления (при восстановлении виртуальной машины)

Выполните командлет [Get-AzRecoveryServicesBackupRecoveryPoint](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprecoverypoint), чтобы получить полный список точек восстановления. Выберите нужную точку восстановления. Если вы не уверены, какую точку восстановления следует использовать, рекомендуется выбрать последнюю выбрать = Аппконсистент в списке.

В следующем сценарии переменная **$rp** представляет собой массив точек восстановления для выбранного архивного элемента за последние семь дней. Массив сортируется по времени в обратном порядке, так что последняя точка восстановления получает индекс 0. Используйте стандартное индексирование массива PowerShell для выбора точки восстановления. Например, $rp[0] обозначает последнюю точку восстановления.

```powershell
$startDate = (Get-Date).AddDays(-7)
$endDate = Get-Date
$rp = Get-AzRecoveryServicesBackupRecoveryPoint -Item $backupitem -StartDate $startdate.ToUniversalTime() -EndDate $enddate.ToUniversalTime() -VaultId $targetVault.ID
$rp[0]
```

Вы должны увидеть результат, аналогичный приведенному ниже.

```output
RecoveryPointAdditionalInfo :
SourceVMStorageType         : NormalStorage
Name                        : 15260861925810
ItemName                    : VM;iaasvmcontainer;RGName1;V2VM
RecoveryPointId             : /subscriptions/XX/resourceGroups/ RGName1/providers/Microsoft.RecoveryServices/vaults/testvault/backupFabrics/Azure/protectionContainers/IaasVMContainer;iaasvmcontainer;RGName1;V2VM/protectedItems/VM;iaasvmcontainer; RGName1;V2VM/recoveryPoints/15260861925810
RecoveryPointType           : AppConsistent
RecoveryPointTime           : 4/23/2016 5:02:04 PM
WorkloadType                : AzureVM
ContainerName               : IaasVMContainer;iaasvmcontainer; RGName1;V2VM
ContainerType               : AzureVM
BackupManagementType        : AzureVM
```

### <a name="mount-the-disks-of-recovery-point"></a>Подключение дисков точки восстановления

Используйте командлет [Get-азрековерисервицесбаккупрпмаунтскрипт](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprpmountscript) , чтобы получить скрипт для подключения всех дисков точки восстановления.

> [!NOTE]
> Диски подключаются к виртуальной машине, на которой выполняется скрипт, в качестве присоединенных дисков iSCSI. Подключение выполняется немедленно, плата не взимается.
>
>

```powershell
Get-AzRecoveryServicesBackupRPMountScript -RecoveryPoint $rp[0] -VaultId $targetVault.ID
```

Вы должны увидеть результат, аналогичный приведенному ниже.

```output
OsType  Password        Filename
------  --------        --------
Windows e3632984e51f496 V2VM_wus2_8287309959960546283_451516692429_cbd6061f7fc543c489f1974d33659fed07a6e0c2e08740.exe
```

Запустите скрипт на виртуальной машине, на которой нужно восстановить файлы. Чтобы выполнить сценарий, необходимо ввести предоставленный пароль. Присоединив диски, используйте проводник Windows для просмотра новых томов и файлов. Дополнительные сведения см. в статье резервное копирование, [Восстановление файлов из резервной копии виртуальной машины Azure](backup-azure-restore-files-from-vm.md).

### <a name="unmount-the-disks"></a>Отключение дисков

После копирования необходимых файлов используйте [Disable-азрековерисервицесбаккупрпмаунтскрипт](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackuprpmountscript) , чтобы отключить диски. Убедитесь, что диски отключены, чтобы удалить доступ к файлам точки восстановления.

```powershell
Disable-AzRecoveryServicesBackupRPMountScript -RecoveryPoint $rp[0] -VaultId $targetVault.ID
```

## <a name="next-steps"></a>Дальнейшие действия

Если вы предпочитаете использовать PowerShell для взаимодействия с ресурсами Azure, см. статью [Развертывание резервного копирования в Azure для Windows Server или клиента Windows и управление им с помощью PowerShell](backup-client-automation.md). Сведения об управлении резервными копиями DPM см. в статье [Развертывание службы архивации для DPM и управление ею](backup-dpm-automation.md).
