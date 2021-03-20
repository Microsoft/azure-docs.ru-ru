---
title: Доступ к хранилищам данных и файловым ресурсам с помощью проверки подлинности Windows
description: Узнайте, как настроить каталог служб MSSQL Integration Services в Базе данных SQL Azure и Azure-SSIS Integration Runtime в Фабрике данных Azure для запуска пакетов с доступом к хранилищам данных и общим папкам с использованием проверки подлинности Windows.
ms.date: 10/27/2020
ms.topic: conceptual
ms.prod: sql
ms.prod_service: integration-services
ms.custom: ''
ms.technology: integration-services
author: swinarko
ms.author: sawinark
ms.reviewer: maghan
ms.openlocfilehash: 77283596e4b0d7e25d757c9de76c284ca4f5d4f2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98556405"
---
# <a name="access-data-stores-and-file-shares-with-windows-authentication-from-ssis-packages-in-azure"></a>Доступ к хранилищам данных и общим папкам из пакетов служб Integration Services в Azure с использованием проверки подлинности Windows

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Можно использовать проверку подлинности Windows для доступа к хранилищам данных, таким как серверы SQL Server, общие папки, Файлы Azure и т. д., из пакетов служб Integration Services, запущенных в среде выполнения интеграции Azure-SSIS Integration Runtime в Фабрике данных Azure (ADF). Хранилища данных могут быть локальными, размещаться на виртуальных машинах Azure или выполняться в Azure в качестве управляемых служб. Если они находятся в локальной среде, необходимо присоединить Azure-SSIS IR к виртуальной сети (виртуальная сеть Microsoft Azure), подключенной к локальной сети, см. в разделе [соединение Azure-SSIS IR с виртуальная сеть Microsoft Azure](./join-azure-ssis-integration-runtime-virtual-network.md). Есть четыре способа доступа к хранилищам данных с использованием проверки подлинности Windows из пакетов служб Integration Services, выполняющихся в Azure-SSIS IR.

| Метод подключения | Область действия | Этап настройки | Метод доступа в пакетах | Число наборов учетных данных и подключенных ресурсов | Тип подключенных ресурсов | 
|---|---|---|---|---|---|
| Настройка контекста выполнения на уровне действий | Для каждого выполняемого действия пакета служб Integration Services | Настройте свойство **проверки подлинности Windows** для настройки контекста выполнения или запуска как при выполнении пакетов служб Integration Services в виде действий "Выполнение пакета SSIS" в конвейерах ADF.<br/><br/> Дополнительные сведения см. в руководстве по [настройке действий "Выполнение пакета SSIS"](./how-to-invoke-ssis-package-ssis-activity.md). | Доступ к ресурсам напрямую в пакетах, например, используйте UNC-путь для доступа к файловым ресурсам или файлам Azure: `\\YourFileShareServerName\YourFolderName` или `\\YourAzureStorageAccountName.file.core.windows.net\YourFolderName` | Поддержка только одного набора учетных данных для всех подключенных ресурсов: | — общие файловые ресурсы в локальной среде и (или) на виртуальных машинах Azure;<br/><br/> — файлы Azure, как описано в [этой статье](../storage/files/storage-how-to-use-files-windows.md).<br/><br/> – серверы SQL Server в локальной среде или на виртуальных машинах Azure с проверкой подлинности Windows;<br/><br/> – другие ресурсы с проверкой подлинности Windows. |
| Настройка контекста выполнения на уровне каталога | На Azure-SSIS IR, но переопределяется при настройке контекста выполнения на уровне действия (см. выше). | Выполните хранимую процедуру SSISDB `catalog.set_execution_credential`, чтобы настроить контекст выполнения или запуска как.<br/><br/> Дополнительные сведения см. ниже в этой статье. | Доступ к ресурсам напрямую в пакетах, например, используйте UNC-путь для доступа к файловым ресурсам или файлам Azure: `\\YourFileShareServerName\YourFolderName` или `\\YourAzureStorageAccountName.file.core.windows.net\YourFolderName` | Поддержка только одного набора учетных данных для всех подключенных ресурсов: | — общие файловые ресурсы в локальной среде и (или) на виртуальных машинах Azure;<br/><br/> — файлы Azure, как описано в [этой статье](../storage/files/storage-how-to-use-files-windows.md).<br/><br/> – серверы SQL Server в локальной среде или на виртуальных машинах Azure с проверкой подлинности Windows;<br/><br/> – другие ресурсы с проверкой подлинности Windows. |
| Сохранение учетных данных посредством команды `cmdkey` | На Azure-SSIS IR, но переопределяется при настройке контекста выполнения действия или уровня каталога (см. выше). | Выполните `cmdkey` команду в пользовательском скрипте установки ( `main.cmd` ) при подготовке Azure-SSIS IR, например при использовании файловых ресурсов, файлов Azure или SQL Server:<br/><br/> `cmdkey /add:YourFileShareServerName /user:YourDomainName\YourUsername /pass:YourPassword`,<br/><br/> `cmdkey /add:YourAzureStorageAccountName.file.core.windows.net /user:azure\YourAzureStorageAccountName /pass:YourAccessKey`или<br/><br/> `cmdkey /add:YourSQLServerFullyQualifiedDomainNameOrIPAddress:YorSQLServerPort /user:YourDomainName\YourUsername /pass:YourPassword`.<br/><br/> Дополнительные сведения см. в статье [Пользовательская установка для среды выполнения интеграции Azure–SSI](./how-to-configure-azure-ssis-ir-custom-setup.md). | Доступ к ресурсам напрямую в пакетах, например, используйте UNC-путь для доступа к файловым ресурсам или файлам Azure: `\\YourFileShareServerName\YourFolderName` или `\\YourAzureStorageAccountName.file.core.windows.net\YourFolderName` | Поддержка нескольких наборов учетных данных для разных подключенных ресурсов: | — общие файловые ресурсы в локальной среде и (или) на виртуальных машинах Azure;<br/><br/> — файлы Azure, как описано в [этой статье](../storage/files/storage-how-to-use-files-windows.md).<br/><br/> – серверы SQL Server в локальной среде или на виртуальных машинах Azure с проверкой подлинности Windows;<br/><br/> – другие ресурсы с проверкой подлинности Windows. |
| Подключение дисков во время выполнения пакета (без сохранения состояния) | Для каждого пакета | Выполните команду `net use` в задаче "Выполнение процесса", которая добавляется в начало потока управления в пакетах, например, `net use D: \\YourFileShareServerName\YourFolderName`. | Обращение к общим папкам через подключенные диски. | Поддержка нескольких дисков для разных файловых ресурсов: | — общие файловые ресурсы в локальной среде и (или) на виртуальных машинах Azure;<br/><br/> — файлы Azure, как описано в [этой статье](../storage/files/storage-how-to-use-files-windows.md). |
|||||||

> [!WARNING]
> Если вы не используете ни один из указанных выше методов для доступа к хранилищам данных с помощью проверки подлинности Windows, пакеты, зависящие от проверки подлинности Windows, не смогут получить доступ к ним и завершиться сбоем во время выполнения. 

В оставшейся части этой статьи описывается настройка каталога служб SSIS (SSISDB), размещенного в базе данных SQL или SQL Управляемый экземпляр, для запуска пакетов на Azure-SSIS IR, использующих проверку подлинности Windows для доступа к хранилищам данных. 

## <a name="you-can-only-use-one-set-of-credentials"></a>Можно использовать только один набор учетных данных

Если в пакете служб Integration Services применяется проверка подлинности Windows, можно использовать только один набор учетных данных. Учетные данные домена, указываемые при выполнении инструкций из этой статьи, применяются ко всем операциям (интерактивным или запланированным) в среде Azure-SSIS IR, пока эти данные не будут изменены или удалены. Если пакет должен подключаться к нескольким хранилищам данных с разными наборами учетных данных, следует рассмотреть другие описанные выше способы.

## <a name="provide-domain-credentials-for-windows-authentication"></a>Указание учетных данных домена для проверки подлинности Windows

Чтобы задать учетные данные домена, с помощью которых пакеты смогут подключаться к локальным хранилищам данных, используя проверку подлинности Windows, сделайте следующее:

1. С помощью SQL Server Management Studio (SSMS) или другого средства подключитесь к базе данных SQL или Управляемый экземпляр SQL, где размещается SSISDB. Дополнительные сведения см. в статье [Подключение к каталогу SSIS (SSISDB) в Azure](/sql/integration-services/lift-shift/ssis-azure-connect-to-catalog-database).

2. Откройте окно запроса для текущей базы данных SSISDB.

3. Выполните следующую хранимую процедуру и укажите соответствующие учетные данные домена:

   ```sql
   catalog.set_execution_credential @user='<your user name>', @domain='<your domain name>', @password='<your password>'
   ```

4. Запустите пакеты служб SSIS. Пакеты используют учетные данные, предоставленные для доступа к локальным хранилищам данных с проверкой подлинности Windows.

### <a name="view-domain-credentials"></a>Просмотр учетных данных домена

Чтобы просмотреть активные учетные данные домена, выполните указанные ниже действия.

1. С помощью SSMS или другого средства подключитесь к базе данных SQL или Управляемый экземпляр SQL, где размещается SSISDB. Дополнительные сведения см. в статье [Подключение к каталогу SSIS (SSISDB) в Azure](/sql/integration-services/lift-shift/ssis-azure-connect-to-catalog-database).

2. Откройте окно запроса для текущей базы данных SSISDB.

3. Выполните следующую хранимую процедуру и просмотрите выходные данные:

   ```sql
   SELECT * 
   FROM catalog.master_properties
   WHERE property_name = 'EXECUTION_DOMAIN' OR property_name = 'EXECUTION_USER'
   ```

### <a name="clear-domain-credentials"></a>Удаление учетных данных домена
Чтобы удалить учетные данные, заданные согласно инструкциям в этой статье, выполните указанные ниже действия.

1. С помощью SSMS или другого средства подключитесь к базе данных SQL или Управляемый экземпляр SQL, где размещается SSISDB. Дополнительные сведения см. в статье [Подключение к каталогу SSIS (SSISDB) в Azure](/sql/integration-services/lift-shift/ssis-azure-connect-to-catalog-database).

2. Откройте окно запроса для текущей базы данных SSISDB.

3. Выполните следующую хранимую процедуру:

   ```sql
   catalog.set_execution_credential @user='', @domain='', @password=''
   ```

## <a name="connect-to-a-sql-server-on-premises"></a>Подключение к SQL Server в локальной среде

Чтобы проверить возможность подключения к SQL Server в локальной среде, сделайте следующее:

1. Чтобы выполнить эту проверку, найдите компьютер, не присоединенный к домену.

2. На не присоединенном к домену компьютере выполните следующую команду, чтобы запустить SSMS с требуемыми учетными данными домена:

   ```cmd
   runas.exe /netonly /user:<domain>\<username> SSMS.exe
   ```

3. В среде SSMS проверьте возможность подключения к требуемому серверу SQL Server в локальной среде.

### <a name="prerequisites"></a>Предварительные условия

Для доступа к SQL Server в локальной среде из пакетов, выполняющихся в Azure, сделайте следующее:

1.  В диспетчере конфигурации SQL Server включите протокол TCP/IP.

2. Разрешите доступ в брандмауэре Windows. Дополнительные сведения см. в руководстве по [настройке брандмауэра Windows для включения доступа к SQL Server](/sql/sql-server/install/configure-the-windows-firewall-to-allow-sql-server-access).

3. Присоедините Azure-SSIS IR к виртуальная сеть Microsoft Azure, который подключен к SQL Server локальной среде.  Дополнительные сведения см. в разделе [Присоединение Azure-SSIS IR к виртуальная сеть Microsoft Azure](./join-azure-ssis-integration-runtime-virtual-network.md).

4. Выполните хранимую процедуру SSISDB `catalog.set_execution_credential`, чтобы указать учетные данные, как описано в этой статье.

## <a name="connect-to-a-file-share-on-premises"></a>Подключение к общей папке в локальной среде

Чтобы проверить возможность подключения к общей папке в локальной среде, сделайте следующее:

1. Чтобы выполнить эту проверку, найдите компьютер, не присоединенный к домену.

2. На компьютере, не присоединенном к домену, выполните приведенные ниже команды. Эти команды открывают окно командной строки с требуемыми учетными данными домена, а затем проверяют возможность подключения к общей папке в локальной среде, получая список каталогов.

   ```cmd
   runas.exe /netonly /user:<domain>\<username> cmd.exe
   dir \\fileshare
   ```

3. Проверьте, возвращен ли список каталогов из общей папки в локальной среде.

### <a name="prerequisites"></a>Предварительные условия

Для доступа к общей папке в локальной среде из пакетов, выполняющихся в Azure, сделайте следующее:

1. Разрешите доступ в брандмауэре Windows.

2. Присоедините Azure-SSIS IR к виртуальная сеть Microsoft Azure, который подключен к общей папке в локальной среде.  Дополнительные сведения см. в разделе [Присоединение Azure-SSIS IR к виртуальная сеть Microsoft Azure](./join-azure-ssis-integration-runtime-virtual-network.md).

3. Выполните хранимую процедуру SSISDB `catalog.set_execution_credential`, чтобы указать учетные данные, как описано в этой статье.

## <a name="connect-to-a-file-share-on-azure-vm"></a>Подключение к общей папке на виртуальной машине Azure

Для включения доступа к общей папке на виртуальной машине Azure из пакетов, выполняющихся в Azure, сделайте следующее:

1. С помощью SSMS или другого средства подключитесь к базе данных SQL или Управляемый экземпляр SQL, где размещается SSISDB. Дополнительные сведения см. в статье [Подключение к каталогу SSIS (SSISDB) в Azure](/sql/integration-services/lift-shift/ssis-azure-connect-to-catalog-database).

2. Откройте окно запроса для текущей базы данных SSISDB.

3. Выполните следующую хранимую процедуру и укажите соответствующие учетные данные домена:

   ```sql
   catalog.set_execution_credential @domain = N'.', @user = N'username of local account on Azure virtual machine', @password = N'password'
   ```

## <a name="connect-to-a-file-share-in-azure-files"></a>Подключение к общей папке в службе файлов Azure

Дополнительные сведения о файлах Azure см. в разделе [Файлы Azure](https://azure.microsoft.com/services/storage/files/).

Для включения доступа к общей папке в Файлах Azure из пакетов, выполняющихся в Azure, сделайте следующее:

1. С помощью SSMS или другого средства подключитесь к базе данных SQL или Управляемый экземпляр SQL, где размещается SSISDB. Дополнительные сведения см. в статье [Подключение к каталогу SSIS (SSISDB) в Azure](/sql/integration-services/lift-shift/ssis-azure-connect-to-catalog-database).

2. Откройте окно запроса для текущей базы данных SSISDB.

3. Выполните следующую хранимую процедуру и укажите соответствующие учетные данные домена:

   ```sql
   catalog.set_execution_credential @domain = N'Azure', @user = N'<storage-account-name>', @password = N'<storage-account-key>'
   ```

## <a name="next-steps"></a>Дальнейшие действия

- Развертывание пакетов. Дополнительные сведения см. в статье [Развертывание проекта служб SSIS с помощью SQL Server Management Studio (SSMS)](/sql/integration-services/ssis-quickstart-deploy-ssms).
- Запуск пакетов. Дополнительные сведения см. в статье [Выполнение пакета служб SSIS с помощью SQL Server Management Studio (SSMS)](/sql/integration-services/ssis-quickstart-run-ssms).
- Планирование выполнения пакетов. Дополнительные сведения см. [в разделе Планирование пакетов служб SSIS в Azure](/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).
