---
title: Настройка репликации между управляемыми экземплярами
titleSuffix: Azure SQL Managed Instance
description: В этом руководстве показано, как настроить репликацию транзакций между Управляемыми экземплярами SQL Azure, выполняющими роли издателя или распространителя и подписчика.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: data-movement
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: MashaMSFT
ms.author: ferno
ms.reviewer: mathoma
ms.date: 04/28/2020
ms.openlocfilehash: 47ea5ea73c581313f90791ca6d7892ebad3f666b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101690691"
---
# <a name="tutorial-configure-replication-between-two-managed-instances"></a>Руководство по Настройка репликации между двумя управляемыми экземплярами

[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Репликация транзакций позволяет вам реплицировать данные из одной базы данных в другую, размещенную на SQL Server или на [Управляемом экземпляре SQL Azure](sql-managed-instance-paas-overview.md). Управляемый экземпляр SQL может быть издателем, распространителем или подписчиком в топологии репликации. Сведения о доступных конфигурациях см. [здесь](replication-transactional-overview.md#common-configurations). 

В настоящее время репликация транзакций предоставляется в общедоступной предварительной версии Управляемого экземпляра SQL. 

В этом руководстве описано следующее:

> [!div class="checklist"]
>
> - Настройка Управляемого экземпляра в качестве издателя и распространителя репликации.
> - Настройка управляемого экземпляра в качестве подписчика репликации.

![Репликация между двумя управляемыми экземплярами](./media/replication-between-two-instances-configure-tutorial/sqlmi-sqlmi-repl.png)

Это руководство предназначено для опытных пользователей. Предполагается, что пользователь знаком с процедурами развертывания и подключения для управляемых экземпляров и виртуальных машин SQL Server в Azure. 


> [!NOTE]
> - В этой статье описывается использование [репликации транзакций](/sql/relational-databases/replication/transactional/transactional-replication) в Управляемом экземпляре SQL Azure. Она не связана с [группами отработки отказа](../database/auto-failover-group-overview.md), которые представляют собой функцию Управляемого экземпляра SQL Azure, позволяющую создавать полные, доступные для чтения реплики отдельных экземпляров. При настройке [репликации транзакций с группами отработки отказа](replication-transactional-overview.md#with-failover-groups) следует учитывать дополнительные факторы.



## <a name="requirements"></a>Requirements (Требования)

Для настройки Управляемого экземпляра в качестве издателя и (или) распространителя должны выполняться следующие требования:

- Управляемый экземпляр издателя должен находиться в той же виртуальной сети, что и распространитель и подписчик, или между [шлюзами виртуальных сетей](../../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md) всех трех сущностей должен быть настроен пиринг. 
- При подключении используется аутентификация SQL между участниками репликации.
- Общий ресурс учетной записи хранения Azure для рабочей папки репликации.
- Открытый порт 445 (исходящее TCP-подключение) в правилах безопасности, настроенных для группы безопасности сети, чтобы управляемые экземпляры могли получить доступ к общей папке Azure.  Если возникает ошибка `failed to connect to azure storage <storage account name> with os error 53`, значит нужно добавить правило исходящего трафика в NSG подсети соответствующего Управляемого экземпляра SQL.

## <a name="1---create-a-resource-group"></a>1\. Создание группы ресурсов

С помощью [портала Azure](https://portal.azure.com) создайте группу ресурсов с именем `SQLMI-Repl`.  

## <a name="2---create-managed-instances"></a>2\. Создание управляемых экземпляров

С помощью [портала Azure](https://portal.azure.com) создайте два [Управляемых экземпляра SQL](instance-create-quickstart.md) в одной виртуальной сети и подсети. Например, присвойте управляемым экземплярам следующие имена:

- `sql-mi-pub` (и еще несколько символов, чтобы имя было уникальным);
- `sql-mi-sub` (и еще несколько символов, чтобы имя было уникальным).

Кроме того, вам нужно [настроить виртуальную машину Azure для подключения](connect-vm-instance-configure.md) к управляемым экземплярам. 

## <a name="3---create-an-azure-storage-account"></a>3\. Создание учетной записи хранения Azure

[Создайте учетную запись хранения Azure](../../storage/common/storage-account-create.md#create-a-storage-account) для рабочей папки, а затем создайте в этой учетной записи [общую папку](../../storage/files/storage-how-to-create-file-share.md). 

Скопируйте путь к общей папке в формате `\\storage-account-name.file.core.windows.net\file-share-name`.

Например, `\\replstorage.file.core.windows.net\replshare`.

Скопируйте ключи доступа к хранилищу в следующем формате: `DefaultEndpointsProtocol=https;AccountName=<Storage-Account-Name>;AccountKey=****;EndpointSuffix=core.windows.net`.

Например, `DefaultEndpointsProtocol=https;AccountName=replstorage;AccountKey=dYT5hHZVu9aTgIteGfpYE64cfis0mpKTmmc8+EP53GxuRg6TCwe5eTYWrQM4AmQSG5lb3OBskhg==;EndpointSuffix=core.windows.net`.

См. сведения о том, как [управлять ключами доступа к учетной записи хранения](../../storage/common/storage-account-keys-manage.md). 

## <a name="4---create-a-publisher-database"></a>4\. Создание базы данных издателя

Подключитесь к управляемому экземпляру `sql-mi-pub` с помощью SQL Server Management Studio и выполните следующий код Transact-SQL (T-SQL) для создания базы данных издателя:

```sql
USE [master]
GO

CREATE DATABASE [ReplTran_PUB]
GO

USE [ReplTran_PUB]
GO
CREATE TABLE ReplTest (
  ID INT NOT NULL PRIMARY KEY,
  c1 VARCHAR(100) NOT NULL,
  dt1 DATETIME NOT NULL DEFAULT getdate()
)
GO


USE [ReplTran_PUB]
GO

INSERT INTO ReplTest (ID, c1) VALUES (6, 'pub')
INSERT INTO ReplTest (ID, c1) VALUES (2, 'pub')
INSERT INTO ReplTest (ID, c1) VALUES (3, 'pub')
INSERT INTO ReplTest (ID, c1) VALUES (4, 'pub')
INSERT INTO ReplTest (ID, c1) VALUES (5, 'pub')
GO
SELECT * FROM ReplTest
GO
```

## <a name="5---create-a-subscriber-database"></a>5\. Создание базы данных подписчика

Подключитесь к управляемому экземпляру `sql-mi-sub` с помощью SQL Server Management Studio и выполните следующий код T-SQL для создания пустой базы данных подписчика:

```sql
USE [master]
GO

CREATE DATABASE [ReplTran_SUB]
GO

USE [ReplTran_SUB]
GO
CREATE TABLE ReplTest (
  ID INT NOT NULL PRIMARY KEY,
  c1 VARCHAR(100) NOT NULL,
  dt1 DATETIME NOT NULL DEFAULT getdate()
)
GO
```

## <a name="6---configure-distribution"></a>6\. Настройка распространения

Подключитесь к управляемому экземпляру `sql-mi-pub` с помощью SQL Server Management Studio и выполните следующий код T-SQL для настройки базы данных распространителя:

```sql
USE [master]
GO

EXEC sp_adddistributor @distributor = @@ServerName;
EXEC sp_adddistributiondb @database = N'distribution';
GO
```

## <a name="7---configure-publisher-to-use-distributor"></a>7\. Настройка использования распространителя для издателя

В Управляемом экземпляре SQL `sql-mi-pub` для издателя измените режим выполнения запроса на [SQLCMD](/sql/ssms/scripting/edit-sqlcmd-scripts-with-query-editor) и выполните следующий код, чтобы зарегистрировать новый распространитель в этом издателе.

```sql
:setvar username loginUsedToAccessSourceManagedInstance
:setvar password passwordUsedToAccessSourceManagedInstance
:setvar file_storage "\\storage-account-name.file.core.windows.net\file-share-name"
-- example: file_storage "\\replstorage.file.core.windows.net\replshare"
:setvar file_storage_key "DefaultEndpointsProtocol=https;AccountName=<Storage-Account-Name>;AccountKey=****;EndpointSuffix=core.windows.net"
-- example: file_storage_key "DefaultEndpointsProtocol=https;AccountName=replstorage;AccountKey=dYT5hHZVu9aTgIteGfpYE64cfis0mpKTmmc8+EP53GxuRg6TCwe5eTYWrQM4AmQSG5lb3OBskhg==;EndpointSuffix=core.windows.net"

USE [master]
EXEC sp_adddistpublisher
  @publisher = @@ServerName,
  @distribution_db = N'distribution',
  @security_mode = 0,
  @login = N'$(username)',
  @password = N'$(password)',
  @working_directory = N'$(file_storage)',
  @storage_connection_string = N'$(file_storage_key)'; -- Remove this parameter for on-premises publishers
```

   > [!NOTE]
   > В параметре file_storage используйте только символы обратной черты (`\`). Использование символа прямой косой черты (`/`) может привести к ошибке при подключении к общей папке.

Этот скрипт настраивает локальный издатель на управляемом экземпляре, добавляет связанный сервер и создает набор заданий для агента SQL Server.

## <a name="8---create-publication-and-subscriber"></a>8\. Создание публикации и подписчика

Используя режим [SQLCMD](/sql/ssms/scripting/edit-sqlcmd-scripts-with-query-editor), выполните следующий скрипт T-SQL, чтобы включить для базы данных репликацию, и настройте репликацию между издателем, распространителем и подписчиком.

```sql
-- Set variables
:setvar username sourceLogin
:setvar password sourcePassword
:setvar source_db ReplTran_PUB
:setvar publication_name PublishData
:setvar object ReplTest
:setvar schema dbo
:setvar target_server "sql-mi-sub.wdec33262scj9dr27.database.windows.net"
:setvar target_username targetLogin
:setvar target_password targetPassword
:setvar target_db ReplTran_SUB

-- Enable replication for your source database
USE [$(source_db)]
EXEC sp_replicationdboption
  @dbname = N'$(source_db)',
  @optname = N'publish',
  @value = N'true';

-- Create your publication
EXEC sp_addpublication
  @publication = N'$(publication_name)',
  @status = N'active';


-- Configure your log reader agent
EXEC sp_changelogreader_agent
  @publisher_security_mode = 0,
  @publisher_login = N'$(username)',
  @publisher_password = N'$(password)',
  @job_login = N'$(username)',
  @job_password = N'$(password)';

-- Add the publication snapshot
EXEC sp_addpublication_snapshot
  @publication = N'$(publication_name)',
  @frequency_type = 1,
  @publisher_security_mode = 0,
  @publisher_login = N'$(username)',
  @publisher_password = N'$(password)',
  @job_login = N'$(username)',
  @job_password = N'$(password)';

-- Add the ReplTest table to the publication
EXEC sp_addarticle
  @publication = N'$(publication_name)',
  @type = N'logbased',
  @article = N'$(object)',
  @source_object = N'$(object)',
  @source_owner = N'$(schema)';

-- Add the subscriber
EXEC sp_addsubscription
  @publication = N'$(publication_name)',
  @subscriber = N'$(target_server)',
  @destination_db = N'$(target_db)',
  @subscription_type = N'Push';

-- Create the push subscription agent
EXEC sp_addpushsubscription_agent
  @publication = N'$(publication_name)',
  @subscriber = N'$(target_server)',
  @subscriber_db = N'$(target_db)',
  @subscriber_security_mode = 0,
  @subscriber_login = N'$(target_username)',
  @subscriber_password = N'$(target_password)',
  @job_login = N'$(username)',
  @job_password = N'$(password)';

-- Initialize the snapshot
EXEC sp_startpublication_snapshot
  @publication = N'$(publication_name)';
```

## <a name="9---modify-agent-parameters"></a>9\. Изменение параметров агента

В Управляемом экземпляре SQL Azure сейчас возникают определенные серверные проблемы с подключением к агентам репликации. Пока эта проблема решается, вы можете в качестве обходного решения увеличить значения времени ожидания входа для агентов репликации.

Выполните следующую команду T-SQL на издателе, чтобы увеличить время ожидания входа:

```sql
-- Increase login timeout to 150s
update msdb..sysjobsteps set command = command + N' -LoginTimeout 150'
where subsystem in ('Distribution','LogReader','Snapshot') and command not like '%-LoginTimeout %'
```

Снова выполните следующую команду T-SQL, чтобы вернуть для времени ожидания входа значение по умолчанию, если это потребуется:

```sql
-- Increase login timeout to 30
update msdb..sysjobsteps set command = command + N' -LoginTimeout 30'
where subsystem in ('Distribution','LogReader','Snapshot') and command not like '%-LoginTimeout %'
```

Перезапустите все три агента, чтобы применить эти изменения.

## <a name="10---test-replication"></a>10. Тестовая репликация

Настроенную репликацию можно протестировать, добавив новые элементы в издатель и убедившись, что изменения распространяются на подписчик.

Выполните следующий фрагмент кода T-SQL, чтобы просмотреть строки в подписчике:

```sql
select * from dbo.ReplTest
```

Выполните следующий фрагмент кода T-SQL, чтобы вставить в издатель дополнительные строки, а затем снова проверьте строки в подписчике.

```sql
INSERT INTO ReplTest (ID, c1) VALUES (15, 'pub')
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить публикацию, выполните следующую команду T-SQL:

```sql
-- Drops the publication
USE [ReplTran_PUB]
EXEC sp_droppublication @publication = N'PublishData'
GO
```

Чтобы удалить параметр репликации из базы данных, выполните следующую команду T-SQL:

```sql
-- Disables publishing of the database
USE [ReplTran_PUB]
EXEC sp_removedbreplication
GO
```

Чтобы отключить публикацию и распространение, выполните следующую команду T-SQL:

```sql
-- Drops the distributor
USE [master]
EXEC sp_dropdistributor @no_checks = 1
GO
```

Вы можете очистить ресурсы Azure, [удалив ресурсы Управляемого экземпляра SQL из группы ресурсов](../../azure-resource-manager/management/manage-resources-portal.md#delete-resources), а затем удалив группу ресурсов `SQLMI-Repl`. 

## <a name="next-steps"></a>Дальнейшие действия

Вы также можете получить дополнительные сведения о [репликации транзакций с помощью Управляемого экземпляра SQL Azure](replication-transactional-overview.md) и [настройке репликации между Управляемым экземпляром SQL в роли издателя или распространителя и SQL на виртуальной машине Azure в роли подписчика](replication-two-instances-and-sql-server-configure-tutorial.md).