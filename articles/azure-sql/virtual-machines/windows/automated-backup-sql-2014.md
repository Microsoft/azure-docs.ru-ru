---
title: Автоматическое резервное копирование для виртуальных машин SQL Server 2014 в Azure
description: Описывает функцию автоматической архивации для виртуальных машин SQL Server 2014 в Azure. Данная статья относится к виртуальным машинам, использующим модель развертывания с помощью Resource Manager.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.assetid: bdc63fd1-db49-4e76-87d5-b5c6a890e53c
ms.service: virtual-machines-sql
ms.subservice: backup
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 05/03/2018
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 41add54ce767413982ab0503f7263c58aed4d4e2
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97359307"
---
# <a name="automated-backup-for-sql-server-2014-virtual-machines-resource-manager"></a>Автоматическое резервное копирование для виртуальных машин SQL Server 2014 (Resource Manager)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

> [!div class="op_single_selector"]
> * [SQL Server 2014](automated-backup-sql-2014.md)
> * [SQL Server 2016 или SQL Server 2017](automated-backup.md)

Служба автоматической архивации автоматически настраивает [управляемое резервное копирование на портал Microsoft Azure](/sql/relational-databases/backup-restore/sql-server-managed-backup-to-microsoft-azure) для всех существующих и новых баз данных на виртуальной машине Azure c SQL Server 2014 Standard или Enterprise. Это позволяет настроить регулярное резервное копирование базы данных с использованием надежного хранилища больших двоичных объектов Azure. Автоматическое резервное копирование зависит от [расширения агента IaaS для SQL Server](sql-server-iaas-agent-extension-automate-management.md).

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-rm-include.md)]

## <a name="prerequisites"></a>Предварительные требования
Для использования автоматической архивации необходимо выполнить следующие предварительные требования.


**Операционная система**.

- Windows Server 2012 и более поздние версии 

**Версия/выпуск SQL Server**

- SQL Server 2014 Standard
- SQL Server 2014 Enterprise

> [!NOTE]
> Для SQL 2016 и более поздних версий см. раздел [Автоматическое резервное копирование для SQL Server 2016](automated-backup.md).

**Конфигурация базы данных**

- Целевые _пользовательские_ базы данных должны использовать модель полного восстановления. Для системных баз данных нет необходимости использовать модель полного восстановления. Тем не менее, если требуется создавать резервные копии журналов для шаблона базы данных или MSDB, то необходимо использовать модель полного восстановления. Дополнительные сведения о влиянии модели полного восстановления на резервные копии см. в статье [Резервное копирование в модели полного восстановления](/previous-versions/sql/sql-server-2008-r2/ms190217(v=sql.105)). 
- SQL Server виртуальная машина зарегистрирована с расширением агента IaaS SQL в [режиме полного управления](sql-agent-extension-manually-register-single-vm.md#upgrade-to-full). 
-  Автоматическая архивация использует полное [расширение агента SQL Server IaaS](sql-server-iaas-agent-extension-automate-management.md). Таким образом, автоматическая архивация поддерживается только для целевых баз данных из экземпляра по умолчанию или одного именованного экземпляра. Если экземпляр по умолчанию отсутствует и несколько именованных экземпляров, то расширение IaaS SQL завершится сбоем, а автоматическая архивация не будет работать. 

## <a name="settings"></a>Настройки

В приведенной ниже таблице описаны параметры настройки автоматической архивации. Фактическая процедура настройки может варьироваться в зависимости от того, используете вы портал Azure или команды Azure Windows PowerShell.

| Параметр | Диапазон (по умолчанию) | Описание |
| --- | --- | --- |
| **Автоматическое резервное копирование** | Включено/отключено (отключено) | Включает или отключает автоматическую архивацию для виртуальной машины Azure под управлением SQL Server 2014 Standard или Enterprise. |
| **Срок хранения** | 1–30 дней (30 дней) | Число дней хранения резервной копии. |
| **Учетная запись хранения** | Учетная запись хранения Azure. | Учетная запись хранения Azure для хранения файлов автоматической архивации в хранилище больших двоичных объектов. Там же создается контейнер для хранения всех файлов резервных копий. Имя файла резервной копии содержит дату, время и имя компьютера. |
| **Шифрование** | Включено/отключено (отключено) | Включает или отключает шифрование. Если шифрование включено, то сертификаты, используемые для восстановления резервной копии, сохраняются в указанной учетной записи хранения в том же контейнере `automaticbackup` с использованием того же соглашения об именовании файлов. При изменении пароля создается новый сертификат; при этом старый сертификат сохраняется для восстановления предыдущих резервных копий. |
| **Пароль** | Текст пароля | Пароль для ключей шифрования. Требуется, только если шифрование включено. Для восстановления зашифрованной резервной копии требуется правильный пароль и соответствующий сертификат, который использовался при создании резервной копии. |


## <a name="configure-new-vms"></a>Настройка новых виртуальных машин

При создании новой виртуальной машины SQL Server 2014 с моделью развертывания с помощью Resource Manager настройте автоматическое резервное копирование, используя портал Azure.

Прокрутите вкладку **Настройки SQL Server** вниз до раздела **Автоматическая архивация** и нажмите **Включить**. На представленном ниже снимке экрана портала Azure показаны параметры **автоматического резервного копирования SQL**.

![Настройка автоматического резервного копирования SQL на портале Azure](./media/automated-backup-sql-2014/azure-sql-arm-autobackup.png)

## <a name="configure-existing-vms"></a>Настройка имеющихся виртуальных машин

[!INCLUDE [windows-virtual-machines-sql-use-new-management-blade](../../../../includes/windows-virtual-machines-sql-new-resource.md)]

Для существующих виртуальных машин SQL Server можно включить и отключить автоматическое резервное копирование, изменить период хранения, указать учетную запись хранения и включить шифрование на портале Azure. 

Перейдите к ресурсу [Виртуальные машины SQL](manage-sql-vm-portal.md#access-the-sql-virtual-machines-resource), к которому относится виртуальная машина SQL Server 2014, и выберите **Резервные копии**. 

![Автоматизированная архивация SQL для существующих виртуальных машин](./media/automated-backup-sql-2014/azure-sql-rm-autobackup-existing-vms.png)

По завершении в нижней части страницы **Резервные копии** нажмите кнопку **Применить**, чтобы сохранить изменения.

Если автоматизированная архивация включается впервые, Azure настроит агент IaaS SQL Server в фоновом режиме. При этом портал Azure может не сообщать о том, что выполняется настройка автоматической архивации. Установка и настройка агента занимают несколько минут. После этого новые параметры отобразятся на портале Azure.

> [!NOTE]
> Можно также настроить автоматизированную архивацию с помощью шаблона. Чтобы получить дополнительные сведения, ознакомьтесь с [шаблоном быстрого запуска Azure для автоматизированной архивации](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-sql-existing-autobackup-update).

## <a name="configure-with-powershell"></a>Настройка с помощью PowerShell

Для настройки автоматической архивации можно также использовать PowerShell. Предварительно необходимо выполнить следующее.

- [Скачайте и установите последнюю версию Azure PowerShell](https://aka.ms/webpi-azps).
- Откройте Windows PowerShell и свяжите его с учетной записью с помощью команды **Connect-AzAccount**. 

[!INCLUDE [updated-for-az.md](../../../../includes/updated-for-az.md)]

### <a name="install-the-sql-server-iaas-extension"></a>Установка расширения IaaS для SQL Server
Если вы подготовили виртуальную машину SQL Server на портале Azure, то на ней уже должно быть установлено расширение IaaS для SQL Server. Чтобы выяснить, так ли это, выполните команду **Get-AzVM** и проверьте свойство **Extensions**.

```powershell
$vmname = "vmname"
$resourcegroupname = "resourcegroupname"

(Get-AzVM -Name $vmname -ResourceGroupName $resourcegroupname).Extensions
```

Если расширение агента IaaS для SQL Server установлено, вы увидите его в списке как SqlIaaSAgent или SQLIaaSExtension. Свойство **ProvisioningState** для расширения должно иметь значение Succeeded.

Если расширение не установлено или его не удалось подготовить, то для его установки можно выполнить приведенную ниже команду. Кроме имени и группы ресурсов виртуальной машины, необходимо указать регион ( **$region**), в котором она расположена. Укажите тип лицензии для виртуальной машины SQL Server, выбрав в рамках [Преимущества гибридного использования Azure](https://azure.microsoft.com/pricing/hybrid-benefit/) один из вариантов: с оплатой по мере использования или с использованием собственной лицензии. Дополнительные сведения о лицензировании см. [здесь](licensing-model-azure-hybrid-benefit-ahb-change.md). 

```powershell
New-AzSqlVM  -Name $vmname `
    -ResourceGroupName $resourcegroupname `
    -Location $region -LicenseType <PAYG/AHUB>
```

> [!IMPORTANT]
> Если расширение не установлено, при его установке перезапускается SQL Server.

### <a name="verify-current-settings"></a><a id="verifysettings"></a> Проверка текущих параметров

Если вы включили автоматическую архивацию во время подготовки, то можете использовать PowerShell для проверки текущей конфигурации. Выполните команду **Get-AzVMSqlServerExtension** и изучите свойство **AutoBackupSettings**.

```powershell
(Get-AzVMSqlServerExtension -VMName $vmname -ResourceGroupName $resourcegroupname).AutoBackupSettings
```

Должен отобразиться результат, аналогичный приведенному ниже.

```
Enable                      : False
EnableEncryption            : False
RetentionPeriod             : -1
StorageUrl                  : NOTSET
StorageAccessKey            : 
Password                    : 
BackupSystemDbs             : False
BackupScheduleType          : 
FullBackupFrequency         : 
FullBackupStartTime         : 
FullBackupWindowHours       : 
LogBackupFrequency          : 
```

Если выходные данные показывают, что значение **Enable** равно **False**, то необходимо включить автоматическую архивацию. Хорошая новость состоит в том, что включить и настроить автоматическую архивацию можно точно так же. Этот процесс описан в следующем разделе.

> [!NOTE] 
> Если вы проверяете параметры сразу же после внесения изменений, возможно, вы увидите старые значения конфигурации. Подождите несколько минут и проверьте параметры, чтобы убедиться, что изменения были применены.

### <a name="configure-automated-backup"></a>Настройка автоматической архивации
Чтобы включить автоматическую архивацию, а также в любое время изменить ее конфигурацию и поведение, можно использовать PowerShell.

Сначала выберите или создайте учетную запись хранения для файлов резервных копий. Приведенный ниже сценарий выбирает учетную запись хранения или создает ее, если она не существует.

```powershell
$storage_accountname = "yourstorageaccount"
$storage_resourcegroupname = $resourcegroupname

$storage = Get-AzStorageAccount -ResourceGroupName $resourcegroupname `
    -Name $storage_accountname -ErrorAction SilentlyContinue
If (-Not $storage)
    { $storage = New-AzStorageAccount -ResourceGroupName $storage_resourcegroupname `
    -Name $storage_accountname -SkuName Standard_GRS -Location $region }
```

> [!NOTE]
> Служба автоматической архивации не поддерживает хранение резервных копий в хранилище уровня "Премиум", но может создавать резервные копии дисков виртуальных машин, которые используют хранилище уровня "Премиум".

Затем с помощью **New-AzVMSqlServerAutoBackupConfig** включите и настройте параметры автоматической архивации для хранения архивных копий в учетной записи хранения Azure. В этом примере резервные копии хранятся в течение 10 дней. Вторая команда, **Set-AzVMSqlServerExtension**, обновляет указанную виртуальную машину Azure в соответствии с заданными параметрами.

```powershell
$autobackupconfig = New-AzVMSqlServerAutoBackupConfig -Enable `
    -RetentionPeriodInDays 10 -StorageContext $storage.Context `
    -ResourceGroupName $storage_resourcegroupname

Set-AzVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname
```

Установка и настройка агента SQL Server IaaS занимают несколько минут.

> [!NOTE]
> Существуют и другие параметры для **New-AzVMSqlServerAutoBackupConfig**, которые применяются только для SQL Server 2016 и автоматической архивации версии 2. SQL Server 2014 не поддерживает следующие параметры: **BackupSystemDbs**, **BackupScheduleType**, **FullBackupFrequency**, **FullBackupStartHour**, **FullBackupWindowInHours** и **LogBackupFrequencyInMinutes**. При попытке настроить эти параметры на виртуальной машине SQL Server 2014 ошибки не возникают, но параметры не применяется. Если вы хотите использовать эти параметры на виртуальной машине SQL Server 2016, см. статью [Автоматическое резервное копирование (версия 2) для виртуальных машин SQL Server 2016 в Azure](automated-backup.md).

Чтобы включить шифрование, измените предыдущий сценарий таким образом, чтобы он передавал параметр **EnableEncryption** вместе с паролем (защищенной строкой) для параметра **CertificatePassword**. Следующий скрипт активирует параметры автоматической архивации их предыдущего примера и добавляет шифрование.

```powershell
$password = "P@ssw0rd"
$encryptionpassword = $password | ConvertTo-SecureString -AsPlainText -Force

$autobackupconfig = New-AzVMSqlServerAutoBackupConfig -Enable `
    -EnableEncryption -CertificatePassword $encryptionpassword `
    -RetentionPeriodInDays 10 -StorageContext $storage.Context `
    -ResourceGroupName $storage_resourcegroupname

Set-AzVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname
```

Чтобы убедиться, что параметры были применены, [проверьте конфигурацию автоматической архивации](#verifysettings).

### <a name="disable-automated-backup"></a>Отключение автоматической архивации

Чтобы отключить автоматическую архивацию, выполните тот же сценарий без параметра **-Enable** в команде **New-AzVMSqlServerAutoBackupConfig**. Отсутствие параметра **-Enable** означает, что функцию нужно отключить. Как и установка, отключение автоматической архивации занимает несколько минут.

```powershell
$autobackupconfig = New-AzVMSqlServerAutoBackupConfig -ResourceGroupName $storage_resourcegroupname

Set-AzVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname
```

### <a name="example-script"></a>Пример сценария

Ниже приведен сценарий, предоставляющий набор переменных, которые можно задать для включения и настройки автоматической архивации для виртуальной машины. Может потребоваться настроить этот сценарий в зависимости от ваших требований. Например, его нужно будет изменить, если вы захотите включить шифрование или отключить архивацию системных баз данных.

```powershell
$vmname = "yourvmname"
$resourcegroupname = "vmresourcegroupname"
$region = "Azure region name such as EASTUS2"
$storage_accountname = "storageaccountname"
$storage_resourcegroupname = $resourcegroupname
$retentionperiod = 10

# ResourceGroupName is the resource group which is hosting the VM where you are deploying the SQL Server IaaS Extension

Set-AzVMSqlServerExtension -VMName $vmname `
    -ResourceGroupName $resourcegroupname -Name "SQLIaasExtension" `
    -Version "2.0" -Location $region

# Creates/use a storage account to store the backups

$storage = Get-AzStorageAccount -ResourceGroupName $resourcegroupname `
    -Name $storage_accountname -ErrorAction SilentlyContinue
If (-Not $storage)
    { $storage = New-AzStorageAccount -ResourceGroupName $storage_resourcegroupname `
    -Name $storage_accountname -SkuName Standard_GRS -Location $region }

# Configure Automated Backup settings

$autobackupconfig = New-AzVMSqlServerAutoBackupConfig -Enable `
    -RetentionPeriodInDays $retentionperiod -StorageContext $storage.Context `
    -ResourceGroupName $storage_resourcegroupname

# Apply the Automated Backup settings to the VM

Set-AzVMSqlServerExtension -AutoBackupSettings $autobackupconfig `
    -VMName $vmname -ResourceGroupName $resourcegroupname
```

## <a name="monitoring"></a>Наблюдение

Для отслеживания автоматического резервного копирования в SQL Server 2014 имеются две основные возможности. Так как автоматическое резервное копирование использует функцию управляемого резервного копирования SQL Server, к обоим этим методам применяются одинаковые способы мониторинга.

Во-первых, можно опрашивать состояние, вызывая [msdb.smart_admin.sp_get_backup_diagnostics](/sql/relational-databases/system-stored-procedures/managed-backup-sp-get-backup-diagnostics-transact-sql). Можно также запрашивать функцию с табличным значением [msdb.smart_admin.fn_get_health_status](/sql/relational-databases/system-functions/managed-backup-fn-get-health-status-transact-sql).

> [!NOTE]
> Схема для управляемого резервного копирования в SQL Server 2014 — **msdb.smart_admin**. В SQL Server 2016 она изменена на **msdb.managed_backup**, и в справочных разделах используется именно новая схема. Но для SQL Server 2014 необходимо использовать схему **smart_admin** для всех объектов управляемого резервного копирования.

Другой вариант — воспользоваться преимуществами встроенного компонента Database Mail для отправки уведомлений.

1. Вызовите хранимую процедуру [msdb.smart_admin.sp_set_parameter](/sql/relational-databases/system-stored-procedures/managed-backup-sp-set-parameter-transact-sql), чтобы назначить адрес электронной почты параметру **SSMBackup2WANotificationEmailIds**. 
1. Включите [SendGrid](../../../sendgrid-dotnet-how-to-send-email.md) для отправки электронных сообщений из виртуальной машины Azure.
1. SMTP-сервер и имя пользователя позволяют настроить компонент Database Mail. Настроить компонент Database Mail можно в SQL Server Management Studio или с помощью команд Transact-SQL. Дополнительные сведения см. в разделе о [компоненте Database Mail](/sql/relational-databases/database-mail/database-mail).
1. [Настройте почту агента SQL Server для использования компонента Database Mail](/sql/relational-databases/database-mail/configure-sql-server-agent-mail-to-use-database-mail).
1. Убедитесь, что SMTP-порт открыт в локальном брандмауэре виртуальных машин и группе безопасности сети виртуальной машины.

## <a name="next-steps"></a>Дальнейшие действия

Автоматическая архивация настраивает управляемое резервное копирование на виртуальных машинах Azure. Поэтому очень важно [изучить документацию по управляемому резервному копированию в SQL Server 2014](/sql/relational-databases/backup-restore/sql-server-managed-backup-to-microsoft-azure).

Дополнительные указания по резервному копированию и восстановлению для SQL Server на виртуальных машинах Azure можно найти в статье [Резервное копирование и восстановление SQL Server на виртуальных машинах Azure](backup-restore.md).

Сведения о других доступных задачах автоматизации см. в разделе [Расширение агента IaaS для SQL Server](sql-server-iaas-agent-extension-automate-management.md).

Дополнительные сведения о запуске SQL Server на виртуальных машинах Azure см. в [обзоре использования SQL Server на виртуальных машинах Azure](sql-server-on-azure-vm-iaas-what-is-overview.md).