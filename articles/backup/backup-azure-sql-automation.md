---
title: Резервное копирование базы данных SQL в виртуальной машине Azure & восстановление с помощью PowerShell
description: Резервное копирование и восстановление баз данных SQL на виртуальных машинах Azure с помощью Azure Backup и PowerShell.
ms.topic: conceptual
ms.date: 03/15/2019
ms.assetid: 57854626-91f9-4677-b6a2-5d12b6a866e1
ms.openlocfilehash: 0a3467ffa3a67ac9ad593748948cea8da59e3e6b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97734544"
---
# <a name="back-up-and-restore-sql-databases-in-azure-vms-with-powershell"></a>Резервное копирование и восстановление баз данных SQL на виртуальных машинах Azure с помощью PowerShell

В этой статье описывается, как использовать Azure PowerShell для резервного копирования и восстановления базы данных SQL в виртуальной машине Azure с помощью [Azure Backup](backup-overview.md) хранилища служб восстановления.

В статье описывается выполнение следующих задач:

> [!div class="checklist"]
>
> * Настройте PowerShell и зарегистрируйте поставщик служб восстановления Azure.
> * Создайте хранилище служб восстановления,
> * Настройте резервное копирование для базы данных SQL на виртуальной машине Azure.
> * Запустите задание резервного копирования.
> * Восстановите резервную копию базы данных SQL.
> * Мониторинг заданий резервного копирования и восстановления.

## <a name="before-you-start"></a>Перед началом работы

* Дополнительные [сведения](backup-azure-recovery-services-vault-overview.md) о хранилищах служб восстановления.
* Узнайте о возможностях [резервного копирования SQL баз данных на виртуальных машинах Azure](backup-azure-sql-database.md#before-you-start).
* Проверьте иерархию объектов PowerShell для служб восстановления.

### <a name="recovery-services-object-hierarchy"></a>Иерархия объектов служб восстановления

Иерархия объектов представлена на следующей схеме.

![Иерархия объектов служб восстановления](./media/backup-azure-vms-arm-automation/recovery-services-object-hierarchy.png)

Ознакомьтесь со справочником по [командлету](/powershell/module/az.recoveryservices) **AZ. RecoveryServices** в библиотеке Azure.

### <a name="set-up-and-install"></a>Установка и установка

Настройте PowerShell следующим образом.

1. [Скачайте последнюю версию Az PowerShell](/powershell/azure/install-az-ps). Минимальная требуемая версия — 1.5.0.

2. Найдите командлеты PowerShell Azure Backup с помощью следующей команды:

    ```powershell
    Get-Command *azrecoveryservices*
    ```

3. Проверьте псевдонимы и командлеты для Azure Backup и хранилища служб восстановления. Ниже приведен пример того, что можно увидеть. Это не полный список командлетов.

    ![Список командлетов Служб восстановления](./media/backup-azure-afs-automation/list-of-recoveryservices-ps-az.png)

4. Войдите в учетную запись Azure с помощью **Connect-азаккаунт**.
5. На появившейся веб-странице вам будет предложено ввести учетные данные вашей учетной записи.

    * Кроме того, можно включить учетные данные учетной записи в качестве параметра в командлет **Connect-азаккаунт** с параметром **-Credential**.
    * Если вы являетесь партнером CSP, работающим для клиента, укажите клиента в качестве клиента, используя его идентификатор клиента или основное доменное имя. Например: **Connect-AzAccount-Tenant** fabrikam.com.

6. Свяжите подписку, которую вы хотите использовать с учетной записью, так как у учетной записи может быть несколько подписок.

    ```powershell
    Select-AzSubscription -SubscriptionName $SubscriptionName
    ```

7. Если вы используете службу архивации Azure впервые, выполните командлет **Register-AzResourceProvider**, чтобы зарегистрировать поставщик Служб восстановления Azure в своей подписке.

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

8. Убедитесь, что поставщики успешно зарегистрированы:

    ```powershell
    Get-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

9. Убедитесь, что в выходных данных команды **RegistrationState** изменения **зарегистрированы**. Если это не так, выполните командлет **Register-азресаурцепровидер** еще раз.

## <a name="create-a-recovery-services-vault"></a>Создание хранилища Служб восстановления

Чтобы создать хранилище Служб восстановления, выполните описанные ниже действия.

Хранилище Служб восстановления представляет собой ресурс Resource Manager, поэтому вам потребуется разместить его в группе ресурсов. Вы можете использовать имеющуюся группу ресурсов или создать новую, выполнив командлет **New-AzResourceGroup**. При создании группы ресурсов укажите ее имя и расположение.

1. Хранилище помещается в группу ресурсов. Если у вас нет группы ресурсов, создайте ее с помощью [New-азресаурцеграуп](/powershell/module/az.resources/new-azresourcegroup). В этом примере мы создадим новую группу ресурсов в регионе "Западная часть США".

    ```powershell
    New-AzResourceGroup -Name "test-rg" -Location "West US"
    ```

2. Используйте командлет [New-азрековерисервицесваулт](/powershell/module/az.recoveryservices/new-azrecoveryservicesvault) , чтобы создать хранилище. Разместите хранилище там же, где находится группа ресурсов.

    ```powershell
    New-AzRecoveryServicesVault -Name "testvault" -ResourceGroupName "test-rg" -Location "West US"
    ```

3. Укажите тип избыточности, используемый для хранения хранилища.

    * Вы можете использовать [локально избыточное хранилище](../storage/common/storage-redundancy.md#locally-redundant-storage), [геоизбыточное хранилище](../storage/common/storage-redundancy.md#geo-redundant-storage) или [хранилище, избыточное](../storage/common/storage-redundancy.md#zone-redundant-storage) в виде зоны.
    * В следующем примере задается параметр **-BackupStorageRedundancy** для командлета [Set-азрековерисервицесбаккуппроперти](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupproperty) cmd для **testvault задано** , установленного в значение " **геоизбыточность**".

    ```powershell
    $vault1 = Get-AzRecoveryServicesVault -Name "testvault"
    Set-AzRecoveryServicesBackupProperties  -Vault $vault1 -BackupStorageRedundancy GeoRedundant
    ```

### <a name="view-the-vaults-in-a-subscription"></a>Просмотр хранилищ в подписке

Выполнив командлет [Get-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/get-azrecoveryservicesvault), вы получите список всех хранилищ в подписке.

```powershell
Get-AzRecoveryServicesVault
```

Выходные данные похожи на приведенные ниже. Предоставляются связанная группа ресурсов и расположение.

```output
Name              : Contoso-vault
ID                : /subscriptions/1234
Type              : Microsoft.RecoveryServices/vaults
Location          : WestUS
ResourceGroupName : Contoso-docs-rg
SubscriptionId    : 1234-567f-8910-abc
Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
```

### <a name="set-the-vault-context"></a>Задание контекста хранилища

Сохраните объект хранилища в переменной и задайте контекст хранилища.

* Многие командлеты Azure Backup нуждаются в объекте хранилища служб восстановления в качестве входных данных, поэтому удобно хранить объект хранилища в переменной.
* Контекст хранилища — это тип данных, защищаемых в хранилище. Задайте его с помощью [Set-азрековерисервицесваултконтекст](/powershell/module/az.recoveryservices/set-azrecoveryservicesvaultcontext). После установки контекста он применяется ко всем последующим командлетам.

В следующем примере задается контекст для хранилища **testvault**.

```powershell
Get-AzRecoveryServicesVault -Name "testvault" | Set-AzRecoveryServicesVaultContext
```

### <a name="fetch-the-vault-id"></a>Получение идентификатора хранилища

Мы планируем использовать параметр контекста хранилища в соответствии с рекомендациями Azure PowerShell. Вместо этого можно сохранить или извлечь идентификатор хранилища и передать его соответствующим командам следующим образом:

```powershell
$vaultID = Get-AzRecoveryServicesVault -ResourceGroupName "Contoso-docs-rg" -Name "testvault" | select -ExpandProperty ID
```

## <a name="configure-a-backup-policy"></a>Настройка политики резервного копирования

Политика архивации определяет расписание резервного копирования и время хранения резервных точек восстановления.

* Политика резервного копирования связана по крайней мере с одной политикой хранения. Политика хранения определяет продолжительность хранения точки восстановления до ее удаления.
* Просмотрите политику резервного копирования по умолчанию с помощью команды [Get-азрековерисервицесбаккупретентионполициобжект](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupretentionpolicyobject).
* Просмотрите расписание политики архивации по умолчанию с помощью команды [Get-азрековерисервицесбаккупсчедулеполициобжект](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupschedulepolicyobject).
* Для создания новой политики резервного копирования используется командлет [New-азрековерисервицесбаккуппротектионполици](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupprotectionpolicy) . Введите расписание и объекты политики хранения.

По умолчанию время начала определяется в объекте политики Schedule. Используйте следующий пример, чтобы изменить время начала на нужное время начала. Требуемое время начала должно быть также в формате UTC. В следующем примере предполагается, что требуемое время начала для ежедневного резервного копирования составляет 01:00 часов по ГРИНВИЧу.

```powershell
$schPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "MSSQL"
$UtcTime = Get-Date -Date "2019-03-20 01:30:00Z"
$UtcTime = $UtcTime.ToUniversalTime()
$schpol.ScheduleRunTimes[0] = $UtcTime
```

> [!IMPORTANT]
> Необходимо указать время начала в 30 минут, кратное. В приведенном выше примере это может быть только "01:00:00" или "02:30:00". Время начала не может быть "01:15:00".

В следующем примере показано сохранение политик расписания и хранения в переменных. Затем эти переменные используются в качестве параметров для новой политики (**невсклполици**). **Невсклполици** занимает ежедневное полное резервное копирование, оставляет его в течение 180 дней и создает резервную копию журнала каждые 2 часа.

```powershell
$schPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "MSSQL"
$retPol = Get-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType "MSSQL"
$NewSQLPolicy = New-AzRecoveryServicesBackupProtectionPolicy -Name "NewSQLPolicy" -WorkloadType "MSSQL" -RetentionPolicy $retPol -SchedulePolicy $schPol
```

Выходные данные похожи на приведенные ниже.

```output
Name                 WorkloadType       BackupManagementType BackupTime                Frequency                                IsDifferentialBackup IsLogBackupEnabled
                                                                                                                                Enabled
----                 ------------       -------------------- ----------                ---------                                -------------------- ------------------
NewSQLPolicy         MSSQL              AzureWorkload        3/15/2019 01:30:00 AM      Daily                                    False                True
```

## <a name="enable-backup"></a>Включение резервного копирования

### <a name="registering-the-sql-vm"></a>Регистрация виртуальной машины SQL

Для резервного копирования виртуальных машин Azure и файловых ресурсов Azure служба резервного копирования может подключиться к этим Azure Resource Managerным ресурсам и получить соответствующие сведения. Поскольку SQL является приложением на виртуальной машине Azure, службе архивации требуется разрешение на доступ к приложению и получение необходимых сведений. Для этого необходимо *зарегистрировать* виртуальную машину Azure, СОДЕРЖАЩУЮ приложение SQL, с хранилищем служб восстановления. После регистрации виртуальной машины SQL с хранилищем вы можете защитить баз данных SQL только в этом хранилище. Для регистрации виртуальной машины используйте командлет PowerShell [Register-азрековерисервицесбаккупконтаинер](/powershell/module/az.recoveryservices/register-azrecoveryservicesbackupcontainer) .

```powershell
 $myVM = Get-AzVM -ResourceGroupName <VMRG Name> -Name <VMName>
Register-AzRecoveryServicesBackupContainer -ResourceId $myVM.ID -BackupManagementType AzureWorkload -WorkloadType MSSQL -VaultId $targetVault.ID -Force
```

Команда вернет "контейнер резервной копии" этого ресурса, а состояние будет "зарегистрировано"

> [!NOTE]
> Если параметр Force не указан, появится запрос на подтверждение с текстом "отключить защиту для этого контейнера". Проигнорируйте этот текст и скажите "Y" для подтверждения. Это известная проблема, и мы работаем над удалением текста и требованием параметра Force.

### <a name="fetching-sql-dbs"></a>Получение баз данных SQL

После завершения регистрации служба архивации сможет перечислить все доступные компоненты SQL в виртуальной машине. Чтобы просмотреть все компоненты SQL, для которых еще не требуется резервное копирование в это хранилище, используйте командлет PowerShell [Get-азрековерисервицесбаккуппротектаблеитем](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupprotectableitem) .

```powershell
Get-AzRecoveryServicesBackupProtectableItem -WorkloadType MSSQL -VaultId $targetVault.ID
```

В выходных данных будут показаны все незащищенные компоненты SQL для всех виртуальных машин SQL, зарегистрированных в этом хранилище, с типом элемента и ServerName. Можно дополнительно отфильтровать определенную виртуальную машину SQL, передав параметр "-Container" или указав сочетание "Name" и "ServerName" вместе с флагом ItemType, чтобы приступить к уникальному элементу SQL.

```powershell
$SQLDB = Get-AzRecoveryServicesBackupProtectableItem -workloadType MSSQL -ItemType SQLDataBase -VaultId $targetVault.ID -Name "<Item Name>" -ServerName "<Server Name>"
```

### <a name="configuring-backup"></a>Настройка резервного копирования

Теперь, когда у нас есть необходимая база данных SQL и политика, с которой необходимо создать резервную копию, можно использовать командлет [Enable-азрековерисервицесбаккуппротектион](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection) , чтобы настроить резервное копирование для этой базы данных SQL.

```output
Enable-AzRecoveryServicesBackupProtection -ProtectableItem $SQLDB -Policy $NewSQLPolicy
```

Команда ожидает, пока не завершится Настройка резервного копирования и вернет следующие выходные данные.

```output
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
master           ConfigureBackup      Completed            3/18/2019 6:00:21 PM      3/18/2019 6:01:35 PM      654e8aa2-4096-402b-b5a9-e5e71a496c4e
```

### <a name="fetching-new-sql-dbs"></a>Получение новых баз данных SQL

После регистрации компьютера служба Backup Service выберет сведения о доступных баз данных. Если экземпляры SQL баз данных или SQL Server в дальнейшем добавляются на зарегистрированный компьютер, необходимо вручную запустить службу резервного копирования, чтобы выполнить новый запрос, чтобы получить **все** незащищенные баз данных (включая вновь добавленные). Используйте командлет PowerShell [Initialize-азрековерисервицесбаккупитем](/powershell/module/az.recoveryservices/initialize-azrecoveryservicesbackupprotectableitem) на ВИРТУАЛЬНОЙ машине SQL для выполнения нового запроса. Команда ожидает завершения операции. Позже используйте командлет PowerShell [Get-азрековерисервицесбаккуппротектаблеитем](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupprotectableitem) , чтобы получить список последних незащищенных компонентов SQL.

```powershell
$SQLContainer = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVMAppContainer -FriendlyName <VM name> -VaultId $targetvault.ID
Initialize-AzRecoveryServicesBackupProtectableItem -Container $SQLContainer -WorkloadType MSSQL -VaultId $targetvault.ID
Get-AzRecoveryServicesBackupProtectableItem -workloadType MSSQL -ItemType SQLDataBase -VaultId $targetVault.ID
```

После получения соответствующих защищаемых элементов включите резервное копирование, как [описано в приведенном выше разделе](#configuring-backup).
Если один из них не хочет вручную определять новые баз данных, он может выбрать автоматическую защиту, как описано [ниже](#enable-autoprotection).

## <a name="enable-autoprotection"></a>Включить автозащиту

Вы можете настроить резервное копирование таким образом, чтобы все баз данных, добавленные в будущем, были автоматически защищены с помощью определенной политики. Чтобы включить автозащиту, используйте командлет PowerShell [Enable-азрековерисервицесбаккупаутопротектион](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupautoprotection) .

Так как инструкция предназначена для резервного копирования всех будущих баз данных, операция выполняется на уровне SQLInstance.

```powershell
$SQLInstance = Get-AzRecoveryServicesBackupProtectableItem -workloadType MSSQL -ItemType SQLInstance -VaultId $targetVault.ID -Name "<Protectable Item name>" -ServerName "<Server Name>"
Enable-AzRecoveryServicesBackupAutoProtection -InputItem $SQLInstance -BackupManagementType AzureWorkload -WorkloadType MSSQL -Policy $NewSQLPolicy -VaultId $targetvault.ID
```

После того как цель автоматической защиты задана, запрос к компьютеру для получения только что добавленного баз данных выполняется как запланированная фоновая задача каждые 8 часов.

## <a name="restore-sql-dbs"></a>Восстановление SQL баз данных

Azure Backup можете восстановить базы данных SQL Server, работающие на виртуальных машинах Azure, следующим образом:

* Восстановление до определенной даты или времени (во второй) с помощью резервных копий журнала транзакций. Azure Backup автоматически определяет соответствующую полную разностную резервную копию и цепочку резервных копий журналов, которые требуются для восстановления в зависимости от выбранного времени.
* Восстановление определенной полной или разностной резервной копии для восстановления до определенной точки восстановления.

Перед восстановлением SQL баз данных проверьте предварительные требования, описанные [здесь](restore-sql-database-azure-vm.md#restore-prerequisites) .

Сначала выполните выборку соответствующей резервной копии базы данных SQL с помощью командлета PowerShell [Get-азрековерисервицесбаккупитем](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupitem) .

```powershell
$bkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureWorkload -WorkloadType MSSQL -Name "<backup item name>" -VaultId $targetVault.ID
```

### <a name="fetch-the-relevant-restore-time"></a>Получение соответствующего времени восстановления

Как описано выше, можно восстановить резервную копию базы данных SQL в полную или разностную копию **или** на момент времени в журнале.

#### <a name="fetch-distinct-recovery-points"></a>Получение уникальных точек восстановления

Используйте [Get-азрековерисервицесбаккупрековерипоинт](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprecoverypoint) для получения уникальных (полных или разностных) точек восстановления для резервной копии базы данных SQL.

```powershell
$startDate = (Get-Date).AddDays(-7).ToUniversalTime()
$endDate = (Get-Date).ToUniversalTime()
Get-AzRecoveryServicesBackupRecoveryPoint -Item $bkpItem -VaultId $targetVault.ID -StartDate $startdate -EndDate $endDate
```

Выходные данные похожи на приведенный ниже пример.

```output
RecoveryPointId    RecoveryPointType  RecoveryPointTime      ItemName                             BackupManagemen
                                                                                                  tType
---------------    -----------------  -----------------      --------                             ---------------
6660368097802      Full               3/18/2019 8:09:35 PM   MSSQLSERVER;model             AzureWorkload
```

Используйте фильтр "RecoveryPointId" или фильтр массива для выборки соответствующей точки восстановления.

```powershell
$FullRP = Get-AzRecoveryServicesBackupRecoveryPoint -Item $bkpItem -VaultId $targetVault.ID -RecoveryPointId "6660368097802"
```

#### <a name="fetch-point-in-time-recovery-point"></a>Получение точки восстановления до точки во времени

Если вы хотите восстановить базу данных на определенный момент времени, используйте командлет PowerShell [Get-азрековерисервицесбаккупрековерилогчаин](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprecoverylogchain) . Командлет возвращает список дат, представляющих время начала и окончания неразрывной непрерывной цепочки журналов для этого элемента резервного копирования SQL. Требуемый момент времени должен находиться в пределах этого диапазона.

```powershell
Get-AzRecoveryServicesBackupRecoveryLogChain -Item $bkpItem -VaultId $targetVault.ID
```

Выходные данные будут выглядеть так, как показано в следующем примере.

```output
ItemName                       StartTime                      EndTime
--------                       ---------                      -------
SQLDataBase;MSSQLSERVER;azu... 3/18/2019 8:09:35 PM           3/19/2019 12:08:32 PM
```

Приведенные выше выходные данные означают, что можно выполнить восстановление на любой момент времени между отображаемым временем начала и временем окончания. Время задаются в формате UTC. Создайте любые точки во времени в PowerShell, которые находятся в диапазоне, показанном выше.

> [!NOTE]
> Если для восстановления выбран параметр точка входа в систему, не нужно указывать начальную точку, то есть полное резервное копирование, из которого восстанавливается база данных. Azure Backup служба позаботится о плане восстановления, то есть о том, какую полную резервную копию следует выбрать, какие резервные копии журналов следует применить и т. д.

### <a name="determine-recovery-configuration"></a>Определение конфигурации восстановления

В случае восстановления базы данных SQL поддерживаются следующие сценарии восстановления.

* Переопределение резервной копии базы данных SQL данными из другой точки восстановления — Оригиналворклоадресторе
* Восстановление базы данных SQL в качестве новой базы данных в том же экземпляре SQL — Алтернатеворклоадресторе
* Восстановление базы данных SQL в качестве новой базы данных в другом экземпляре SQL в другой виртуальной машине SQL — Алтернатеворклоадресторе
* Восстановление базы данных SQL в виде BAK-файлов — Рестореасфилес

После получения соответствующей точки восстановления (DISTINCT или Log-On-Time) используйте командлет PowerShell [Get-азрековерисервицесбаккупворклоадрековериконфиг](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupworkloadrecoveryconfig) , чтобы получить объект конфигурации восстановления в соответствии с требуемым планом восстановления.

#### <a name="original-workload-restore"></a>Восстановление исходной рабочей нагрузки

Чтобы переопределить резервную базу данных с данными из точки восстановления, просто укажите правильный флаг и соответствующую точку восстановления, как показано в следующих примерах.

##### <a name="original-restore-with-distinct-recovery-point"></a>Исходное восстановление с отдельной точкой восстановления

```powershell
$OverwriteWithFullConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -RecoveryPoint $FullRP -OriginalWorkloadRestore -VaultId $targetVault.ID
```

##### <a name="original-restore-with-log-point-in-time"></a>Исходное восстановление с точкой входа в систему

```powershell
$OverwriteWithLogConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -PointInTime $PointInTime -Item $bkpItem  -OriginalWorkloadRestore -VaultId $targetVault.ID
```

#### <a name="alternate-workload-restore"></a>Восстановление альтернативной рабочей нагрузки

> [!IMPORTANT]
> Резервную копию базы данных SQL можно восстановить в качестве новой базы данных только для другого SQLInstance, на виртуальной машине Azure, зарегистрированной в этом хранилище.

Как описано выше, если Целевая SQLInstance находится на другой виртуальной машине Azure, убедитесь, что она [зарегистрирована в этом хранилище](#registering-the-sql-vm) , а соответствующая SQLInstance отображается как защищаемый элемент.

```powershell
$TargetInstance = Get-AzRecoveryServicesBackupProtectableItem -WorkloadType MSSQL -ItemType SQLInstance -Name "<SQLInstance Name>" -ServerName "<SQL VM name>" -VaultId $targetVault.ID
```

Затем просто передайте соответствующую точку восстановления, целевой экземпляр SQL с правым флагом, как показано ниже.

##### <a name="alternate-restore-with-distinct-recovery-point"></a>Альтернативное восстановление с отдельной точкой восстановления

```powershell
$AnotherInstanceWithFullConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -RecoveryPoint $FullRP -TargetItem $TargetInstance -AlternateWorkloadRestore -VaultId $targetVault.ID
```

##### <a name="alternate-restore-with-log-point-in-time"></a>Альтернативное восстановление с указанием времени входа в систему

```powershell
$AnotherInstanceWithLogConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -PointInTime $PointInTime -Item $bkpItem -AlternateWorkloadRestore -VaultId $targetVault.ID
```

##### <a name="restore-as-files"></a>Восстановить как файлы

Чтобы восстановить данные резервной копии в виде BAK-файлов вместо базы данных, выберите параметр **восстановить как файлы** . Резервную копию базы данных SQL можно восстановить на любую целевую виртуальную машину, зарегистрированную в этом хранилище.

```powershell
$TargetContainer= Get-AzRecoveryServicesBackupContainer -ContainerType AzureVMAppContainer -FriendlyName "VM name" -VaultId $vaultID
```

##### <a name="restore-as-files-with-distinct-recovery-point"></a>Восстановление в виде файлов с отдельной точкой восстановления

```powershell
$FileRestoreWithFullConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -RecoveryPoint $FullRP -TargetContainer $TargetContainer -RestoreAsFiles -FilePath "<>" -VaultId $targetVault.ID
```

##### <a name="restore-as-files-with-log-point-in-time-from-latest-full"></a>Восстановление в виде файлов с точкой входа в систему из последней полной

```powershell
$FileRestoreWithLogConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -PointInTime $PointInTime -TargetContainer $TargetContainer -RestoreAsFiles -FilePath "<>" -VaultId $targetVault.ID
```

##### <a name="restore-as-files-with-log-point-in-time-from-a-specified-full"></a>Восстановить в виде файлов с заданной полной точкой входа в систему

Если необходимо указать полное значение, которое должно использоваться для восстановления, используйте следующую команду:

```powershell
$FileRestoreWithLogAndSpecificFullConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -PointInTime $PointInTime -FromFull $FullRP -TargetContainer $TargetContainer -RestoreAsFiles -FilePath "<>" -VaultId $targetVault.ID
```

Конечный объект конфигурации точки восстановления, полученный из командлета PowerShell [Get-азрековерисервицесбаккупворклоадрековериконфиг](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupworkloadrecoveryconfig) , содержит все необходимые сведения для восстановления и, как показано ниже.

```output
TargetServer         : <SQL server name>
TargetInstance       : <Target Instance name>
RestoredDBName       : <Target Instance name>/azurebackup1_restored_3_19_2019_1850
OverwriteWLIfpresent : No
NoRecoveryMode       : Disabled
targetPhysicalPath   : {azurebackup1, azurebackup1_log}
ContainerId          : /Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.RecoveryServices/vaults/testVault/backupFabrics/Azure/protectionContainers/vmappcontainer;compute;computeRG;SQLVMName
SourceResourceId     : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/computeRG/VMAppContainer/SQLVMName
RestoreRequestType   : Alternate WL Restore
RecoveryPoint        : Microsoft.Azure.Commands.RecoveryServices.Backup.Cmdlets.Models.AzureWorkloadRecoveryPoint
PointInTime          : 1/1/0001 12:00:00 AM
```

Вы можете изменить поля имя восстановленной базы данных, Овервритевлифпресент, Норековеримоде и Таржетфисикалпас. Получите дополнительные сведения о путях к целевым файлам, как показано ниже.

```powershell
$AnotherInstanceWithFullConfig.targetPhysicalPath
```

```output
MappingType SourceLogicalName SourcePath                  TargetPath
----------- ----------------- ----------                  ----------
Data        azurebackup1      F:\Data\azurebackup1.mdf    F:\Data\azurebackup1_1553001753.mdf
Log         azurebackup1_log  F:\Log\azurebackup1_log.ldf F:\Log\azurebackup1_log_1553001753.ldf
```

Задайте соответствующие свойства PowerShell в виде строковых значений, как показано ниже.

```powershell
$AnotherInstanceWithFullConfig.OverwriteWLIfpresent = "Yes"
$AnotherInstanceWithFullConfig | fl
```

```output
TargetServer         : <SQL server name>
TargetInstance       : <Target Instance name>
RestoredDBName       : <Target Instance name>/azurebackup1_restored_3_19_2019_1850
OverwriteWLIfpresent : Yes
NoRecoveryMode       : Disabled
targetPhysicalPath   : {azurebackup1, azurebackup1_log}
ContainerId          : /Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.RecoveryServices/vaults/testVault/backupFabrics/Azure/protectionContainers/vmappcontainer;compute;computeRG;SQLVMName
SourceResourceId     : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/computeRG/VMAppContainer/SQLVMName
RestoreRequestType   : Alternate WL Restore
RecoveryPoint        : Microsoft.Azure.Commands.RecoveryServices.Backup.Cmdlets.Models.AzureWorkloadRecoveryPoint
PointInTime          : 1/1/0001 12:00:00 AM
```

> [!IMPORTANT]
> Убедитесь, что окончательный объект конфигурации восстановления имеет все необходимые и правильные значения, так как операция восстановления будет основана на объекте конфигурации.

### <a name="restore-with-relevant-configuration"></a>Восстановление с соответствующей конфигурацией

После получения и проверки соответствующего объекта конфигурации восстановления используйте командлет PowerShell [RESTORE-азрековерисервицесбаккупитем](/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem) , чтобы запустить процесс восстановления.

```powershell
Restore-AzRecoveryServicesBackupItem -WLRecoveryConfig $AnotherInstanceWithLogConfig -VaultId $targetVault.ID
```

Операция восстановления возвращает задание, которое необходимо отвести.

```output
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
MSSQLSERVER/m... Restore              InProgress           3/17/2019 10:02:45 AM                                3274xg2b-e4fg-5952-89b4-8cb566gc1748
```

## <a name="manage-sql-backups"></a>Управление резервными копиями SQL

### <a name="on-demand-backup"></a>Резервное копирование по запросу

После включения резервного копирования для базы данных можно также активировать резервное копирование базы данных по запросу с помощью командлета PowerShell [BACKUP-азрековерисервицесбаккупитем](/powershell/module/az.recoveryservices/backup-azrecoveryservicesbackupitem) . В следующем примере запускается полная резервная копия в базе данных SQL с включенным сжатием, а полная резервная копия должна храниться в течение 60 дней.

```powershell
$bkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureWorkload -WorkloadType MSSQL -Name "<backup item name>" -VaultId $targetVault.ID
$endDate = (Get-Date).AddDays(60).ToUniversalTime()
Backup-AzRecoveryServicesBackupItem -Item $bkpItem -BackupType Full -EnableCompression -VaultId $targetVault.ID -ExpiryDateTimeUTC $endDate
```

Команда Backup по запросу возвращает задание, которое необходимо отвести.

```output
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
MSSQLSERVER/m... Backup               InProgress           3/18/2019 8:41:27 PM                                2516bb1a-d3ef-4841-97a3-9ba455fb0637
```

Если выходные данные потеряны или вы хотите получить соответствующий идентификатор задания, [получите список заданий](#track-azure-backup-jobs) от Azure Backup службы, а затем отследите их и сведения о них.

### <a name="change-policy-for-backup-items"></a>Изменение политики для элементов архивации

Политику архивированного элемента можно изменить с *Policy1* на *Policy2*. Чтобы переключить политики для архивированного элемента, извлеките соответствующую политику и элемент резервного копирования и используйте команду [Enable-азрековерисервицес](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection) с элементом Backup в качестве параметра.

```powershell
$TargetPol1 = Get-AzRecoveryServicesBackupProtectionPolicy -Name <PolicyName>
$anotherBkpItem = Get-AzRecoveryServicesBackupItem -WorkloadType MSSQL -BackupManagementType AzureWorkload -Name "<BackupItemName>"
Enable-AzRecoveryServicesBackupProtection -Item $anotherBkpItem -Policy $TargetPol1
```

Команда ожидает, пока не завершится Настройка резервного копирования и вернет следующие выходные данные.

```output
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
master           ConfigureBackup      Completed            3/18/2019 8:00:21 PM      3/18/2019 8:02:16 PM      654e8aa2-4096-402b-b5a9-e5e71a496c4e
```

### <a name="edit-an-existing-backup-policy"></a>Изменение существующей политики архивации

Чтобы изменить существующую политику, используйте команду [Set-азрековерисервицесбаккуппротектионполици](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupprotectionpolicy) .

```powershell
Set-AzRecoveryServicesBackupProtectionPolicy -Policy $Pol -SchedulePolicy $SchPol -RetentionPolicy $RetPol
```

Проверьте задания резервного копирования после того, как прошло некоторое время, чтобы отвестися от сбоев. Если это так, необходимо устранить проблемы. Затем повторно выполните команду изменения политики с параметром **фиксфоринконсистентитемс** , чтобы повторить изменение политики для всех элементов резервного копирования, для которых операция завершилась сбоем.

```powershell
Set-AzRecoveryServicesBackupProtectionPolicy -Policy $Pol -FixForInconsistentItems
```

### <a name="re-register-sql-vms"></a>Повторная регистрация виртуальных машин SQL

> [!WARNING]
> Не забудьте прочитать этот [документ](backup-sql-server-azure-troubleshoot.md#re-registration-failures) , чтобы понять признаки сбоя и причины, прежде чем пытаться повторно зарегистрировать

Чтобы активировать повторную регистрацию виртуальной машины SQL, извлеките соответствующий контейнер резервного копирования и передайте его в командлет Register.

```powershell
$SQLContainer = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVMAppContainer -FriendlyName <VM name> -VaultId $targetvault.ID
Register-AzRecoveryServicesBackupContainer -Container $SQLContainer -BackupManagementType AzureWorkload -WorkloadType MSSQL -VaultId $targetVault.ID
```

### <a name="stop-protection"></a>Остановить защиту

#### <a name="retain-data"></a>Сохранение данных

Если вы хотите отключить защиту, можно использовать командлет PowerShell [Disable-азрековерисервицесбаккуппротектион](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupprotection) . Это приведет к отмене запланированных резервных копий, но данные будут сохранены до тех пор, пока не будет храниться неограниченное время.

```powershell
$bkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureWorkload -WorkloadType MSSQL -Name "<backup item name>" -VaultId $targetVault.ID
Disable-AzRecoveryServicesBackupProtection -Item $bkpItem -VaultId $targetVault.ID
```

#### <a name="delete-backup-data"></a>удаление резервных копий;

Чтобы полностью удалить сохраненные резервные копии данных в хранилище, просто добавьте флаг "-Ремоверековерипоинтс" или переключитесь в [команду "Disable"](#retain-data).

```powershell
Disable-AzRecoveryServicesBackupProtection -Item $bkpItem -VaultId $targetVault.ID -RemoveRecoveryPoints
```

#### <a name="disable-auto-protection"></a>Отключить автоматическую защиту

Если для SQLInstance была настроена автозащита, ее можно отключить с помощью командлета PowerShell [Disable-азрековерисервицесбаккупаутопротектион](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupautoprotection) .

```powershell
$SQLInstance = Get-AzRecoveryServicesBackupProtectableItem -workloadType MSSQL -ItemType SQLInstance -VaultId $targetVault.ID -Name "<Protectable Item name>" -ServerName "<Server Name>"
Disable-AzRecoveryServicesBackupAutoProtection -InputItem $SQLInstance -BackupManagementType AzureWorkload -WorkloadType MSSQL -VaultId $targetvault.ID
```

#### <a name="unregister-sql-vm"></a>Отмена регистрации виртуальной машины SQL

Если все баз данных SQL Server [больше не защищены и данные резервного копирования не существуют](#delete-backup-data), можно отменить регистрацию ВИРТУАЛЬНОЙ машины SQL из этого хранилища. Только после этого вы можете защитить баз данных в другом хранилище. Чтобы отменить регистрацию виртуальной машины SQL, используйте командлет PowerShell [Unregister-азрековерисервицесбаккупконтаинер](/powershell/module/az.recoveryservices/unregister-azrecoveryservicesbackupcontainer) .

```powershell
$SQLContainer = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVMAppContainer -FriendlyName <VM name> -VaultId $targetvault.ID
 Unregister-AzRecoveryServicesBackupContainer -Container $SQLContainer -VaultId $targetvault.ID
```

### <a name="track-azure-backup-jobs"></a>Мониторинг Azure Backup заданий

Важно отметить, что Azure Backup отслеживает только активируемые пользователем задания в резервной копии SQL. Запланированные резервные копии (включая резервные копии журналов) не отображаются на портале или в PowerShell. Однако при сбое запланированных заданий создается [оповещение о резервном копировании](backup-azure-monitoring-built-in-monitor.md#backup-alerts-in-recovery-services-vault) , которое отображается на портале. [Используйте Azure Monitor](backup-azure-monitoring-use-azuremonitor.md) для трассировки всех запланированных заданий и других важных сведений.

Пользователи могут контролировать операции, активируемые по запросу или пользователем, с помощью JobID, возвращаемого в [выходных данных](#on-demand-backup) асинхронных заданий, таких как Backup. Используйте командлет PowerShell [Get-азрековерисервицесбаккупжобдетаил](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupjobdetail) , чтобы отвести задания и сведения о нем.

```powershell
 Get-AzRecoveryServicesBackupJobDetails -JobId 2516bb1a-d3ef-4841-97a3-9ba455fb0637 -VaultId $targetVault.ID
```

Чтобы получить список заданий по запросу и их состояний из Azure Backup службы, используйте командлет PowerShell [Get-азрековерисервицесбаккупжоб](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupjob) . В следующем примере возвращаются все выполняющиеся задания SQL.

```powershell
Get-AzRecoveryServicesBackupJob -Status InProgress -BackupManagementType AzureWorkload
```

Чтобы отменить выполняющееся задание, используйте командлет PowerShell " [остановить-азрековерисервицесбаккупжоб](/powershell/module/az.recoveryservices/stop-azrecoveryservicesbackupjob) ".

## <a name="managing-sql-always-on-availability-groups"></a>Управление группами доступности SQL Always On

Для групп доступности SQL Always On убедитесь, что [зарегистрированы все узлы](#registering-the-sql-vm) группы доступности (AG). После завершения регистрации для всех узлов объект группы доступности SQL логически создается в разделе защищаемые элементы. Базы данных в группе доступности SQL будут перечислены как "SQLDatabase". Узлы будут отображаться как автономные экземпляры, а базы данных SQL по умолчанию в них будут перечислены также как базы данных SQL.

Например, предположим, что в группе доступности SQL есть два узла: *SQL-Server-0* и *SQL-Server-1* и 1 база данных SQL AG. После регистрации этих узлов при [перечислении защищаемых элементов](#fetching-sql-dbs)в нем перечисляются следующие компоненты.

* Объект SQL AG — защищаемый тип элемента как SQLAvailabilityGroup
* Тип защищаемого элемента базы данных SQL AG в виде SQLDatabase
* SQL-Server-0 — Тип защищаемого элемента — SQLInstance
* SQL-Server-1 — Тип защищаемого элемента — SQLInstance
* Все значения по умолчанию SQL баз данных (Master, Model, msdb) в SQL-Server-0 — защищаемый тип элемента как SQLDatabase
* Все значения по умолчанию SQL баз данных (Master, Model, msdb) в SQL-Server-1 — защищаемый тип элемента как SQLDatabase

SQL-Server-0, SQL-Server-1 также будет отображаться как "Азуревмаппконтаинер" при [отображении контейнеров резервного копирования](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupcontainer).

Просто выберете соответствующую базу данных, чтобы [включить резервное копирование](#configuring-backup) , а [командлеты PowerShell](#restore-sql-dbs) [для резервного копирования и восстановления по запросу](#on-demand-backup) идентичны.
