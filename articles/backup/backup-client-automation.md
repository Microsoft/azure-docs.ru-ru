---
title: Использование PowerShell для архивации Windows Server в Azure
description: Из этой статьи вы узнаете, как использовать PowerShell для настройки Azure Backup в Windows Server или на клиенте Windows, а также управлять резервным копированием и восстановлением.
ms.topic: conceptual
ms.date: 12/2/2019
ms.openlocfilehash: 582d8123f16b2d5a543d862b8eb3e45895087e4a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90987107"
---
# <a name="deploy-and-manage-backup-to-azure-for-windows-serverwindows-client-using-powershell"></a>Развертывание резервного копирования в Azure для Windows Server или клиента Windows и управление им с помощью PowerShell

В этой статье показано, как использовать PowerShell для настройки Azure Backup в Windows Server или на клиенте Windows, а также для управления резервным копированием и восстановлением.

## <a name="install-azure-powershell"></a>Установите Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Для начала работы [установите последнюю версию PowerShell](/powershell/azure/install-az-ps).

## <a name="create-a-recovery-services-vault"></a>Создание хранилища Служб восстановления

Чтобы создать хранилище служб восстановления, выполните описанные ниже действия. Хранилище служб восстановления отличается от хранилища службы архивации.

1. Если вы используете Azure Backup в первый раз, необходимо использовать командлет **Register-азресаурцепровидер** для регистрации поставщика службы восстановления Azure в подписке.

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

2. Хранилище служб восстановления — это Azure Resource Manager ресурс, поэтому его необходимо поместить в группу ресурсов. Вы можете выбрать существующую группу ресурсов или создать новую. При создании группы ресурсов укажите ее имя и расположение.  

    ```powershell
    New-AzResourceGroup –Name "test-rg" –Location "WestUS"
    ```

3. Выполните командлет **New-AzRecoveryServicesVault**, чтобы создать хранилище. Разместите хранилище там же, где находится группа ресурсов.

    ```powershell
    New-AzRecoveryServicesVault -Name "testvault" -ResourceGroupName " test-rg" -Location "WestUS"
    ```

4. Укажите необходимый тип избыточности хранилища: Можно использовать [локально избыточное хранилище (LRS)](../storage/common/storage-redundancy.md#locally-redundant-storage), [геоизбыточное хранилище (GRS)](../storage/common/storage-redundancy.md#geo-redundant-storage) или [хранилище, избыточное в виде зоны (ZRS)](../storage/common/storage-redundancy.md#zone-redundant-storage). В следующем примере показан параметр **-BackupStorageRedundancy** для *testvault задано* , для которого задано значение " **геоизбыточность**".

   > [!TIP]
   > Для многих командлетов службы архивации Azure требуется объект хранилища служб восстановления в качестве входных данных. По этой причине объект хранилища Служб восстановления резервных копий удобно хранить в переменной.
   >
   >

    ```powershell
    $Vault1 = Get-AzRecoveryServicesVault –Name "testVault"
    Set-AzRecoveryServicesBackupProperties -Vault $Vault1 -BackupStorageRedundancy GeoRedundant
    ```

## <a name="view-the-vaults-in-a-subscription"></a>Просмотр хранилищ в подписке

Чтобы получить список всех хранилищ в текущей подписке, используйте командлет **Get- AzRecoveryServicesVault**. С помощью этой команды можно проверить, создано ли новое хранилище, или узнать, какие хранилища доступны в подписке.

Выполнив команду **Get-AzRecoveryServicesVault**, вы получите список всех хранилищ в подписке.

```powershell
Get-AzRecoveryServicesVault
```

```Output
Name              : Contoso-vault
ID                : /subscriptions/1234
Type              : Microsoft.RecoveryServices/vaults
Location          : WestUS
ResourceGroupName : Contoso-docs-rg
SubscriptionId    : 1234-567f-8910-abc
Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
```

[!INCLUDE [backup-upgrade-mars-agent.md](../../includes/backup-upgrade-mars-agent.md)]

## <a name="installing-the-azure-backup-agent"></a>Установка агента службы архивации Azure.

Прежде чем устанавливать агент службы архивации Azure, необходимо загрузить установщик и разместить его в системе Windows Server. Последнюю версию установщика можно загрузить в [центре загрузки Майкрософт](https://aka.ms/azurebackup_agent) или на странице панели мониторинга для хранилища служб восстановления. Сохраните установщик в удобном для вас месте, например в папке `C:\Downloads\*`.

Можно также получить установщик с помощью PowerShell:

 ```powershell
 $MarsAURL = 'https://aka.ms/Azurebackup_Agent'
 $WC = New-Object System.Net.WebClient
 $WC.DownloadFile($MarsAURL,'C:\downloads\MARSAgentInstaller.EXE')
 C:\Downloads\MARSAgentInstaller.EXE /q
 ```

Чтобы установить агент, в консоли PowerShell с повышенными привилегиями выполните следующую команду:

```powershell
MARSAgentInstaller.exe /q
```

Агент будет установлен с параметрами по умолчанию. Установка займет всего несколько минут и пройдет в фоновом режиме. Если не указать параметр */Nu* , то в конце установки откроется окно **Центр обновления Windows** , чтобы проверить наличие обновлений. После установки агент появится в списке установленных программ.

Чтобы просмотреть список установленных программ, выберите **Панель управления** > **Программы** > **Программы и компоненты**.

![Агент установлен](./media/backup-client-automation/installed-agent-listing.png)

### <a name="installation-options"></a>Варианты установки

Чтобы просмотреть все доступные в командной строке параметры, используйте следующую команду:

```powershell
MARSAgentInstaller.exe /?
```

Доступны следующие параметры.

| Параметр | Сведения | По умолчанию |
| --- | --- | --- |
| /q |Позволяет выполнить тихую установку. |- |
| /p:"расположение" |Путь к папке установки для агента архивации Azure. |C:\Program Files\Microsoft Azure Recovery Services Agent |
| /s:"расположение" |Путь к папке кэша для агента архивации Azure. |C:\Program Files\Microsoft Azure Recovery Services Agent\Scratch |
| /m |Позволяет явно согласиться на использование Центра обновления Майкрософт. |- |
| /nu |Не проверять наличие обновлений после завершения установки |- |
| /d |Удаляет агент служб восстановления Microsoft Azure. |- |
| /ph |Адрес узла прокси-сервера. |- |
| /po |Номер порта узла прокси-сервера. |- |
| /pu |Имя пользователя узла прокси-сервера. |- |
| /pw |Пароль прокси-сервера. |- |

## <a name="registering-windows-server-or-windows-client-machine-to-a-recovery-services-vault"></a>Регистрация Windows Server или клиентского компьютера Windows в хранилище служб восстановления

После создания хранилища служб восстановления скачайте последнюю версию агента и учетные данные хранилища и сохраните их в удобном расположении, например C:\Downloads.

```powershell
$CredsPath = "C:\downloads"
$CredsFilename = Get-AzRecoveryServicesVaultSettingsFile -Backup -Vault $Vault1 -Path $CredsPath
```

### <a name="registering-using-the-ps-az-module"></a>Регистрация с помощью модуля Az для PowerShell

> [!NOTE]
> Ошибка, возникающая при создании сертификата хранилища, исправлена в выпуске Az 3.5.0. Для скачивания сертификата хранилища используйте выпуск Az 3.5.0 или более поздний.

В последней версии модуля AZ PowerShell из-за ограничений базовой платформы для загрузки учетных данных хранилища требуется самозаверяющий сертификат. В следующем примере показано, как предоставить самозаверяющий сертификат и скачать учетные данные хранилища.

```powershell
$dt = $(Get-Date).ToString("M-d-yyyy")
$cert = New-SelfSignedCertificate -CertStoreLocation Cert:\CurrentUser\My -FriendlyName 'test-vaultcredentials' -subject "Windows Azure Tools" -KeyExportPolicy Exportable -NotAfter $(Get-Date).AddHours(48) -NotBefore $(Get-Date).AddHours(-24) -KeyProtection None -KeyUsage None -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.2") -Provider "Microsoft Enhanced Cryptographic Provider v1.0"
$certficate = [convert]::ToBase64String($cert.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pfx))
$CredsFilename = Get-AzRecoveryServicesVaultSettingsFile -Backup -Vault $Vault -Path $CredsPath -Certificate $certficate
```

На сервере Windows Server или DPM запустите командлет [Start-OBRegistration](/powershell/module/msonlinebackup/start-obregistration) , чтобы зарегистрировать компьютер в хранилище.
Это и другие командлеты, используемые для резервного копирования, из модуля MSONLINE, который К установщика агента MARS добавил как часть процесса установки.

Установщик агента не обновляет $Env:P переменную Смодулепас. Это означает, что автоматическая загрузка модуля завершается ошибкой. Чтобы устранить эту проблему, выполните следующие действия.

```powershell
$Env:PSModulePath += ';C:\Program Files\Microsoft Azure Recovery Services Agent\bin\Modules'
```

Кроме того, модуль можно вручную загрузить в скрипте следующим образом:

```powershell
Import-Module -Name 'C:\Program Files\Microsoft Azure Recovery Services Agent\bin\Modules\MSOnlineBackup'
```

Когда вы загрузите командлеты Online Backup, зарегистрируйте учетные данные хранилища:

```powershell
Start-OBRegistration -VaultCredentials $CredsFilename.FilePath -Confirm:$false
```

```Output
CertThumbprint      : 7a2ef2caa2e74b6ed1222a5e89288ddad438df2
SubscriptionID      : ef4ab577-c2c0-43e4-af80-af49f485f3d1
ServiceResourceName : testvault
Region              : WestUS
Machine registration succeeded.
```

> [!IMPORTANT]
> Не используйте относительные пути для указания файла учетных данных хранилища. Укажите абсолютный путь в качестве входных данных командлета.
>
>

## <a name="networking-settings"></a>Параметры сети

Если подключение компьютера под управлением Windows к Интернету осуществляется через прокси-сервер, параметры этого прокси-сервера могут сообщаться агенту. В этом примере нет прокси-сервера, поэтому мы явно удаляем все сведения, связанные с прокси-сервером.

Управлять использованием пропускной способности для выбранных дней недели можно с помощью параметров `work hour bandwidth` и `non-work hour bandwidth`

Внесение сведений о прокси-сервере и пропускной способности выполняется с помощью командлета [Set-OBMachineSetting](/powershell/module/msonlinebackup/set-obmachinesetting) :

```powershell
Set-OBMachineSetting -NoProxy
```

```Output
Server properties updated successfully.
```

```powershell
Set-OBMachineSetting -NoThrottle
```

```Output
Server properties updated successfully.
```

## <a name="encryption-settings"></a>Параметры шифрования.

Для защиты конфиденциальности данных резервные копии данных, отправляемые в службу архивации Azure, зашифровываются. Используемая для шифрования парольная фраза является "паролем" для расшифровки данных во время их восстановления.

Чтобы создать ПИН-код безопасности, выберите действие **Создать** в меню **Параметры** > **Свойства** > **ПИН-код безопасности** раздела **Хранилище Служб восстановления** на портале Azure.

>[!NOTE]
> ПИН-код безопасности можно создать только на портале Azure.

После этого примените его в качестве `generatedPIN` в следующей команде:

```powershell
$PassPhrase = ConvertTo-SecureString -String "Complex!123_STRING" -AsPlainText -Force
Set-OBMachineSetting -EncryptionPassPhrase $PassPhrase -SecurityPin "<generatedPIN>"
```

```Output
Server properties updated successfully
```

> [!IMPORTANT]
> Обеспечьте безопасность и безопасность сведений парольной фразы после ее установки. Восстановить данные из Azure без этой парольной фразы нельзя.
>
>

## <a name="back-up-files-and-folders"></a>Резервное копирование файлов и папок

Для управления всеми резервными копиями с серверов и рабочих станций Windows, которые имеются в службе резервного копирования Azure, применяется соответствующая политика. Политика состоит из трех частей:

1. **Расписание резервного копирования** — определяет, когда следует создавать резервные копии и синхронизировать их со службой.
2. **Расписание хранения** — определяет период хранения точек восстановления в Azure.
3. **Указания включения/исключения файлов** — определяет, для каких элементов следует создавать резервные копии.

Поскольку мы будем использовать автоматическое резервное копирование данных, в этой статье предполагается, что заданных настроек нет. Начнем с создания новой политики резервного копирования с помощью командлета [New-OBPolicy](/powershell/module/msonlinebackup/new-obpolicy) .

```powershell
$NewPolicy = New-OBPolicy
```

В настоящее время политика пуста, и другие командлеты необходимы для определения элементов, которые будут включены или исключены, когда будут выполняться резервные копии и где будут храниться резервные копии.

### <a name="configuring-the-backup-schedule"></a>Настройка расписания резервного копирования

Первым из трех компонентов политики является расписание резервного копирования, которое создается с помощью командлета [New-OBSchedule](/powershell/module/msonlinebackup/new-obschedule). В расписании резервного копирования указывается, когда необходимо выполнить резервное копирование. При создании расписания необходимо указать два входных параметра:

* **Дни недели** , в которые необходимо выполнять резервное копирование. Можно указать, чтобы задание резервного копирования выполнялось только один день или каждый день недели либо в определенный промежуток времени.
* **Время** запуска резервного копирования. Вы можете определить до трех разных значений времени дня, когда будет выполняться резервное копирование.

Например, политику резервного копирования можно настроить таким образом, чтобы соответствующее задание выполнялось в 16:00 каждые субботу и воскресенье.

```powershell
$Schedule = New-OBSchedule -DaysOfWeek Saturday, Sunday -TimesOfDay 16:00
```

Расписание резервного копирования должно быть связано с политикой. Этого можно добиться с помощью командлета [Set-OBSchedule](/powershell/module/msonlinebackup/set-obschedule).

```powershell
Set-OBSchedule -Policy $NewPolicy -Schedule $Schedule
```

```Output
BackupSchedule : 4:00 PM Saturday, Sunday, Every 1 week(s) DsList : PolicyName : RetentionPolicy : State : New PolicyState : Valid
```

### <a name="configuring-a-retention-policy"></a>Настройка политики хранения

Политика хранения определяет, как долго хранятся точки восстановления, созданные из заданий резервного копирования. При создании новой политики хранения с помощью командлета [New-OBRetentionPolicy](/powershell/module/msonlinebackup/new-obretentionpolicy) можно указать число дней, в течение которых точки восстановления резервных копий будут храниться в Azure Backup. В приведенном ниже примере для политики хранения задано 7 дней.

```powershell
$RetentionPolicy = New-OBRetentionPolicy -RetentionDays 7
```

Политику хранения следует связать с основной политикой с помощью командлета [Set-OBRetentionPolicy](/powershell/module/msonlinebackup/set-obretentionpolicy):

```powershell
Set-OBRetentionPolicy -Policy $NewPolicy -RetentionPolicy $RetentionPolicy
```

```Output
BackupSchedule  : 4:00 PM
                  Saturday, Sunday,
                  Every 1 week(s)
DsList          :
PolicyName      :
RetentionPolicy : Retention Days : 7

                  WeeklyLTRSchedule :
                  Weekly schedule is not set

                  MonthlyLTRSchedule :
                  Monthly schedule is not set

                  YearlyLTRSchedule :
                  Yearly schedule is not set

State           : New
PolicyState     : Valid
```

### <a name="including-and-excluding-files-to-be-backed-up"></a>Включение и исключение файлов для резервного копирования

Объект `OBFileSpec` определяет файлы для включения в резервную копию или исключения из нее. Это набор правил, которые определяют область защищенных файлов и папок на компьютере. Можно создать любое количество правил включения или исключения файлов и связать их с политикой. При создании нового объекта OBFileSpec, можно сделать следующее:

* указать файлы и папки для включения в резервную копию;
* указать файлы и папки для исключения из резервной копии;
* указать рекурсивное резервное копирование данных в папке либо указать, что в резервную копию следует включить только файлы верхнего уровня в указанной папке.

Для выполнения последней задачи необходимо установить флаг -NonRecursive в команде New-OBFileSpec.

В примере ниже выполняется резервное копирование томов C: и D: с исключением двоичных файлов операционной системы в папке Windows и других временных папках. Для этого мы создадим две спецификации файлов с помощью командлета [New-OBFileSpec](/powershell/module/msonlinebackup/new-obfilespec) — для включения и исключения. После создания спецификации файлов связываются с политикой с помощью командлета [Add-OBFileSpec](/powershell/module/msonlinebackup/add-obfilespec) .

```powershell
$Inclusions = New-OBFileSpec -FileSpec @("C:\", "D:\")
$Exclusions = New-OBFileSpec -FileSpec @("C:\windows", "C:\temp") -Exclude
Add-OBFileSpec -Policy $NewPolicy -FileSpec $Inclusions
```

```Output
BackupSchedule  : 4:00 PM
                  Saturday, Sunday,
                  Every 1 week(s)
DsList          : {DataSource
                  DatasourceId:0
                  Name:C:\
                  FileSpec:FileSpec
                  FileSpec:C:\
                  IsExclude:False
                  IsRecursive:True

                  , DataSource
                  DatasourceId:0
                  Name:D:\
                  FileSpec:FileSpec
                  FileSpec:D:\
                  IsExclude:False
                  IsRecursive:True

                  }
PolicyName      :
RetentionPolicy : Retention Days : 7

                  WeeklyLTRSchedule :
                  Weekly schedule is not set

                  MonthlyLTRSchedule :
                  Monthly schedule is not set

                  YearlyLTRSchedule :
                  Yearly schedule is not set

State           : New
PolicyState     : Valid
```

```powershell
Add-OBFileSpec -Policy $NewPolicy -FileSpec $Exclusions
```

```Output
BackupSchedule  : 4:00 PM
                  Saturday, Sunday,
                  Every 1 week(s)
DsList          : {DataSource
                  DatasourceId:0
                  Name:C:\
                  FileSpec:FileSpec
                  FileSpec:C:\
                  IsExclude:False
                  IsRecursive:True
                  ,FileSpec
                  FileSpec:C:\windows
                  IsExclude:True
                  IsRecursive:True
                  ,FileSpec
                  FileSpec:C:\temp
                  IsExclude:True
                  IsRecursive:True

                  , DataSource
                  DatasourceId:0
                  Name:D:\
                  FileSpec:FileSpec
                  FileSpec:D:\
                  IsExclude:False
                  IsRecursive:True

                  }
PolicyName      :
RetentionPolicy : Retention Days : 7

                  WeeklyLTRSchedule :
                  Weekly schedule is not set

                  MonthlyLTRSchedule :
                  Monthly schedule is not set

                  YearlyLTRSchedule :
                  Yearly schedule is not set

State           : New
PolicyState     : Valid
```

### <a name="applying-the-policy"></a>Применение политики

Объект политики готов и имеет связанные расписание резервного копирования, политику хранения, а также список включенных и исключенных файлов. На данном этапе эту политику можно зафиксировать в службе архивации Azure для использования. Перед применением новой политики убедитесь в отсутствии существующих политик резервного копирования, связанных с сервером, с помощью командлета [Remove-OBPolicy](/powershell/module/msonlinebackup/remove-obpolicy). При удалении политики будет запрошено соответствующее подтверждение. Чтобы пропустить подтверждение, используйте в командлете флаг `-Confirm:$false`.

```powershell
Get-OBPolicy | Remove-OBPolicy
```

```Output
Microsoft Azure Backup Are you sure you want to remove this backup policy? This will delete all the backed up data. [Y] Yes [A] Yes to All [N] No [L] No to All [S] Suspend [?] Help (default is "Y"):
```

Фиксация объекта политики выполняется с помощью командлета [Set-OBPolicy](/powershell/module/msonlinebackup/set-obpolicy) . При этом также будет запрашиваться соответствующее подтверждение. Чтобы пропустить подтверждение, используйте в командлете флаг `-Confirm:$false`.

```powershell
Set-OBPolicy -Policy $NewPolicy
```

```Output
Microsoft Azure Backup Do you want to save this backup policy ? [Y] Yes [A] Yes to All [N] No [L] No to All [S] Suspend [?] Help (default is "Y"):
BackupSchedule : 4:00 PM Saturday, Sunday, Every 1 week(s)
DsList : {DataSource
         DatasourceId:4508156004108672185
         Name:C:\
         FileSpec:FileSpec
         FileSpec:C:\
         IsExclude:False
         IsRecursive:True,

         FileSpec
         FileSpec:C:\windows
         IsExclude:True
         IsRecursive:True,

         FileSpec
         FileSpec:C:\temp
         IsExclude:True
         IsRecursive:True,

         DataSource
         DatasourceId:4508156005178868542
         Name:D:\
         FileSpec:FileSpec
         FileSpec:D:\
         IsExclude:False
         IsRecursive:True
    }
PolicyName : c2eb6568-8a06-49f4-a20e-3019ae411bac
RetentionPolicy : Retention Days : 7
              WeeklyLTRSchedule :
              Weekly schedule is not set

              MonthlyLTRSchedule :
              Monthly schedule is not set

              YearlyLTRSchedule :
              Yearly schedule is not set
State : Existing PolicyState : Valid
```

Чтобы просмотреть сведения о существующей политике резервного копирования, воспользуйтесь командлетом [Get-OBPolicy](/powershell/module/msonlinebackup/get-obpolicy) . Чтобы получить подробные сведения, воспользуйтесь командлетом [Get-OBSchedule](/powershell/module/msonlinebackup/get-obschedule) для расписания резервного копирования и командлетом [Get-OBRetentionPolicy](/powershell/module/msonlinebackup/get-obretentionpolicy) для политик хранения.

```powershell
Get-OBPolicy | Get-OBSchedule
```

```Output
SchedulePolicyName : 71944081-9950-4f7e-841d-32f0a0a1359a
ScheduleRunDays : {Saturday, Sunday}
ScheduleRunTimes : {16:00:00}
State : Existing
```

```powershell
Get-OBPolicy | Get-OBRetentionPolicy
```

```Output
RetentionDays : 7
RetentionPolicyName : ca3574ec-8331-46fd-a605-c01743a5265e
State : Existing
```

```powershell
Get-OBPolicy | Get-OBFileSpec
```

```Output
FileName : *
FilePath : \?\Volume{b835d359-a1dd-11e2-be72-2016d8d89f0f}\
FileSpec : D:\
IsExclude : False
IsRecursive : True

FileName : *
FilePath : \?\Volume{cdd41007-a22f-11e2-be6c-806e6f6e6963}\
FileSpec : C:\
IsExclude : False
IsRecursive : True

FileName : *
FilePath : \?\Volume{cdd41007-a22f-11e2-be6c-806e6f6e6963}\windows
FileSpec : C:\windows
IsExclude : True
IsRecursive : True

FileName : *
FilePath : \?\Volume{cdd41007-a22f-11e2-be6c-806e6f6e6963}\temp
FileSpec : C:\temp
IsExclude : True
IsRecursive : True
```

### <a name="performing-an-on-demand-backup"></a>Выполнение резервного копирования по требованию

После настройки соответствующей политики резервное копирование будет выполняться по расписанию. Вы также можете выполнить резервное копирование по запросу с помощью командлета [Start-OBBackup](/powershell/module/msonlinebackup/start-obbackup):

```powershell
Get-OBPolicy | Start-OBBackup
```

```Output
Initializing
Taking snapshot of volumes...
Preparing storage...
Generating backup metadata information and preparing the metadata VHD...
Data transfer is in progress. It might take longer since it is the first backup and all data needs to be transferred...
Data transfer completed and all backed up data is in the cloud. Verifying data integrity...
Data transfer completed
In progress...
Job completed.
The backup operation completed successfully.
```

## <a name="back-up-windows-server-system-state-in-mars-agent"></a>Резервное копирование состояния системы Windows Server в агенте MARS

В этом разделе описывается команда PowerShell для настройки состояния системы в агенте MARS.

### <a name="schedule"></a>Расписание

```powershell
$sched = New-OBSchedule -DaysOfWeek Sunday,Monday,Tuesday,Wednesday,Thursday,Friday,Saturday -TimesOfDay 2:00
```

### <a name="retention"></a>Сохранение

```powershell
$rtn = New-OBRetentionPolicy -RetentionDays 32 -RetentionWeeklyPolicy -RetentionWeeks 13 -WeekDaysOfWeek Sunday -WeekTimesOfDay 2:00  -RetentionMonthlyPolicy -RetentionMonths 13 -MonthDaysOfMonth 1 -MonthTimesOfDay 2:00
```

### <a name="configuring-schedule-and-retention"></a>Настройка расписания и времени хранения

```powershell
New-OBPolicy | Add-OBSystemState |  Set-OBRetentionPolicy -RetentionPolicy $rtn | Set-OBSchedule -Schedule $sched | Set-OBSystemStatePolicy
 ```

### <a name="verifying-the-policy"></a>Проверка политики

```powershell
Get-OBSystemStatePolicy
 ```

## <a name="restore-data-from-azure-backup"></a>Восстановление данных из службы архивации Azure

В этом разделе представлены шаги по настройке автоматического восстановления данных из службы архивации Azure. Выполнение настройки предполагает указанные ниже действия.

1. Выбор исходного тома
2. Выбор точки резервного копирования для восстановления
3. Выбор элемента для восстановления
4. Запуск процесса восстановления

### <a name="picking-the-source-volume"></a>Выбор исходного тома

Чтобы восстановить элемент из Azure Backup, необходимо сначала задать источник элемента. Поскольку мы выполняем команды в контексте Windows Server или клиента Windows, компьютер уже определен. Далее необходимо определить том, на котором находится источник элемента. Список томов или источников, для которых на данном компьютере выполнено резервное копирование, можно получить, выполнив командлет [Get-OBRecoverableSource](/powershell/module/msonlinebackup/get-obrecoverablesource) . Эта команда возвращает массив всех источников на сервере или клиенте, для которых созданы резервные копии.

```powershell
$Source = Get-OBRecoverableSource
$Source
```

```Output
FriendlyName : C:\
RecoverySourceName : C:\
ServerName : myserver.microsoft.com

FriendlyName : D:\
RecoverySourceName : D:\
ServerName : myserver.microsoft.com
```

### <a name="choosing-a-backup-point-from-which-to-restore"></a>Выбор точки резервного копирования для восстановления

Чтобы получить список точек резервного копирования, выполните командлет [Get-OBRecoverableItem](/powershell/module/msonlinebackup/get-obrecoverableitem) с соответствующими параметрами. В нашем примере мы выберем последнюю точку резервного копирования для исходного тома *C:* и применим ее для восстановления определенного файла.

```powershell
$Rps = Get-OBRecoverableItem $Source[0]
$Rps
```

```Output

IsDir                : False
ItemNameFriendly     : C:\
ItemNameGuid         : \\?\Volume{297cbf7a-0000-0000-0000-401f00000000}\
LocalMountPoint      : C:\
MountPointName       : C:\
Name                 : C:\
PointInTime          : 10/17/2019 7:52:13 PM
ServerName           : myserver.microsoft.com
ItemSize             :
ItemLastModifiedTime :

IsDir                : False
ItemNameFriendly     : C:\
ItemNameGuid         : \\?\Volume{297cbf7a-0000-0000-0000-401f00000000}\
LocalMountPoint      : C:\
MountPointName       : C:\
Name                 : C:\
PointInTime          : 10/16/2019 7:00:19 PM
ServerName           : myserver.microsoft.com
ItemSize             :
ItemLastModifiedTime :
```

Объект `$Rps` представляет собой массив точек резервного копирования. Первый элемент является последней точкой, а n-й элемент — самой старой. Чтобы выбрать последнюю точку, мы будем использовать `$Rps[0]` .

### <a name="specifying-an-item-to-restore"></a>Выбор элемента для восстановления

Чтобы восстановить конкретный файл, укажите имя этого файла относительно корневого тома. Например, чтобы получить файл C:\Test\Cat.job, воспользуйтесь следующей командой.

```powershell
$Item = New-OBRecoverableItem $Rps[0] "Test\cat.jpg" $FALSE
$Item
```

```Output
IsDir                : False
ItemNameFriendly     : C:\Test\cat.jpg
ItemNameGuid         :
LocalMountPoint      : C:\
MountPointName       : C:\
Name                 : cat.jpg
PointInTime          : 10/17/2019 7:52:13 PM
ServerName           : myserver.microsoft.com
ItemSize             :
ItemLastModifiedTime : 21-Jun-14 6:43:02 AM

```

### <a name="triggering-the-restore-process"></a>Запуск процесса восстановления

Чтобы запустить процесс восстановления, необходимо сначала указать параметры восстановления. Это можно сделать с помощью командлета [New-OBRecoveryOption](/powershell/module/msonlinebackup/new-obrecoveryoption) . В этом примере предположим, что мы хотим восстановить файлы в *C:\temp*. Также предположим, что мы хотим пропустить файлы, которые уже существуют в папке назначения *C:\temp*. Чтобы создать такой вариант восстановления, используйте следующую команду:

```powershell
$RecoveryOption = New-OBRecoveryOption -DestinationPath "C:\temp" -OverwriteType Skip
```

Теперь запустите процесс восстановления, выполнив команду [Start-OBRecovery](/powershell/module/msonlinebackup/start-obrecovery) со значением `$Item` из выходных данных командлета `Get-OBRecoverableItem`:

```powershell
Start-OBRecovery -RecoverableItem $Item -RecoveryOption $RecoveryOption
```

```Output
Estimating size of backup items...
Estimating size of backup items...
Estimating size of backup items...
Estimating size of backup items...
Job completed.
The recovery operation completed successfully.
```

## <a name="uninstalling-the-azure-backup-agent"></a>Удаление агента службы архивации Azure.

Удалить агент службы архивации Azure можно с помощью следующей команды:

```powershell
.\MARSAgentInstaller.exe /d /q
```

Удаление с компьютера двоичных файлов агента имеет некоторые последствия, которые следует учитывать.

* С компьютера удаляется фильтр файлов, а также останавливается отслеживание изменений.
* Все данные политики удаляются с компьютера, однако сведения о политике по-прежнему хранятся в службе.
* Удаляются все расписания резервного копирования, и последующие операции резервного копирования больше не выполняются.

Однако данные, хранящиеся в Azure, остаются и сохраняются в соответствии с настройкой политики хранения. Предыдущие точки восстановления автоматически рассматриваются как устаревшие.

## <a name="remote-management"></a>Удаленное управление

PowerShell обеспечивает удаленное управление агентом службы архивации Azure, политикой и источниками данных. Компьютер, который будет управляться удаленно, необходимо специально подготовить.

По умолчанию служба WinRM настроена на запуск вручную. Установите тип запуска *Automatic*. Служба запустится. Чтобы проверить работу службы WinRM, присвойте свойству Status значение *Running*.

```powershell
Get-Service -Name WinRM
```

```Output
Status   Name               DisplayName
------   ----               -----------
Running  winrm              Windows Remote Management (WS-Manag...
```

Для удаленного взаимодействия необходимо настроить PowerShell.

```powershell
Enable-PSRemoting -Force
```

```Output
WinRM is already set up to receive requests on this computer.
WinRM has been updated for remote management.
WinRM firewall exception enabled.
```

```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Force
```

Теперь компьютером можно управлять удаленно. Начните с установки агента. Например, следующий сценарий копирует агент на удаленный компьютер и устанавливает его.

```powershell
$DLoc = "\\REMOTESERVER01\c$\Windows\Temp"
$Agent = "\\REMOTESERVER01\c$\Windows\Temp\MARSAgentInstaller.exe"
$Args = "/q"
Copy-Item "C:\Downloads\MARSAgentInstaller.exe" -Destination $DLoc -Force

$Session = New-PSSession -ComputerName REMOTESERVER01
Invoke-Command -Session $Session -Script { param($D, $A) Start-Process -FilePath $D $A -Wait } -ArgumentList $Agent, $Args
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительная информация о службе Azure Backup для сервера или клиента Windows.

* [Общие сведения о службе архивации Azure](./backup-overview.md)
* [Резервное копирование серверов Windows](backup-windows-with-mars-agent.md)
