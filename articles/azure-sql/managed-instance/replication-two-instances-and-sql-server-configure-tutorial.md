---
title: Настройка репликации транзакций между Управляемым экземпляром SQL Azure и SQL Server
description: В этом руководстве показано, как настроить репликацию между управляемым экземпляром издателя, управляемым экземпляром распространителя и подписчиком SQL Server на виртуальной машине Azure. Также здесь описаны необходимые сетевые компоненты, такие как частная зона DNS и пиринг виртуальных сетей.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: sqldbrb=1
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 11/21/2019
ms.openlocfilehash: d2b45f5b51f4656294632aa46f679a7a09c06ed3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94593932"
---
# <a name="tutorial-configure-transactional-replication-between-azure-sql-managed-instance-and-sql-server"></a>Руководство по Настройка репликации транзакций между Управляемым экземпляром SQL Azure и SQL Server
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Репликация транзакций позволяет вам реплицировать данные из одной базы данных в другую, размещенную на SQL Server или на [Управляемом экземпляре SQL Azure](sql-managed-instance-paas-overview.md). Управляемый экземпляр SQL может быть издателем, распространителем или подписчиком в топологии репликации. Сведения о доступных конфигурациях см. [здесь](replication-transactional-overview.md#common-configurations). 

В настоящее время репликация транзакций предоставляется в общедоступной предварительной версии Управляемого экземпляра SQL. 

В этом руководстве описано следующее:

> [!div class="checklist"]
>
> - Настройка Управляемого экземпляра в качестве издателя репликации.
> - Настройка Управляемого экземпляра в качестве распространителя репликации.
> - Настройка SQL Server в качестве подписчика.

![Репликация между издателем управляемого экземпляра, распространителем управляемого экземпляра и подписчиком SQL Server](./media/replication-two-instances-and-sql-server-configure-tutorial/sqlmi-to-sql-replication.png)

Это руководство предназначено для опытных пользователей. Предполагается, что пользователь знаком с процедурами развертывания и подключения для управляемых экземпляров и виртуальных машин SQL Server в Azure. 


> [!NOTE]
> В этой статье описывается использование [репликации транзакций](/sql/relational-databases/replication/transactional/transactional-replication) в Управляемом экземпляре SQL Azure. Она не связана с [группами отработки отказа](../database/auto-failover-group-overview.md), которые представляют собой функцию Управляемого экземпляра SQL Azure, позволяющую создавать полные, доступные для чтения реплики отдельных экземпляров. При настройке [репликации транзакций с группами отработки отказа](replication-transactional-overview.md#with-failover-groups) следует учитывать дополнительные факторы.

## <a name="prerequisites"></a>Предварительные требования

Для работы с руководством требуется наличие следующих компонентов:

- [Подписка Azure](https://azure.microsoft.com/free/).
- Опыт развертывания двух управляемых экземпляров в одной виртуальной сети.
- Подписчик SQL Server, развернутый локально или на виртуальной машине Azure. В этом руководстве используется виртуальная машина Azure.  
- [SQL Server Management Studio (SSMS) версии не ниже 18.0](/sql/ssms/download-sql-server-management-studio-ssms).
- Последняя версия [Azure PowerShell](/powershell/azure/install-az-ps).
- Открытые порты 445 и 1433 для трафика SQL на брандмауэре Azure и брандмауэре Windows.

## <a name="create-the-resource-group"></a>Создание группы ресурсов

Чтобы создать группу ресурсов, используйте следующий фрагмент кода PowerShell.

```powershell-interactive
# set variables
$ResourceGroupName = "SQLMI-Repl"
$Location = "East US 2"

# Create a new resource group
New-AzResourceGroup -Name  $ResourceGroupName -Location $Location
```

## <a name="create-two-managed-instances"></a>Создание двух управляемых экземпляров

Создайте в новой группе ресурсов два управляемых экземпляра с помощью [портала Azure](https://portal.azure.com).

- Управляемому экземпляру издателя следует присвоить имя `sql-mi-publisher` (добавив несколько произвольных символов для уникальности), а виртуальной сети — имя `vnet-sql-mi-publisher`.
- Управляемому экземпляру распространителя следует присвоить имя `sql-mi-distributor` (добавив несколько произвольных символов для уникальности). Он должен располагаться _в той же виртуальной сети, что и управляемый экземпляр издателя_.

   ![Использование виртуальной сети издателя для распространителя](./media/replication-two-instances-and-sql-server-configure-tutorial/use-same-vnet-for-distributor.png)

См. сведения о том, как [создать управляемый экземпляр](instance-create-quickstart.md).

  > [!NOTE]
  > Для простоты в этом руководстве используется самая распространенная конфигурация с размещением управляемого экземпляра распространителя в той же виртуальной сети, где расположен издатель. Но вы можете создать распространитель в отдельной виртуальной сети. Для этого потребуется настроить пиринг между виртуальными сетями издателя и распространителя, а также пиринг между виртуальными сетями распространителя и подписчика.

## <a name="create-a-sql-server-vm"></a>Создание виртуальной машины SQL Server

Создайте виртуальную машину SQL Server с помощью [портала Azure](https://portal.azure.com). Виртуальная машина SQL Server должна иметь следующие характеристики.

- Имя: `sql-vm-sub`
- Образ: SQL Server 2016 или более поздней версии.
- Группа ресурсов: та же, что выбрана для управляемого экземпляра.
- Виртуальная сеть: `sql-vm-sub-vnet`.

См. сведения о развертывании виртуальной машины SQL Server в Azure в [кратком руководстве по созданию виртуальной машины SQL Server](../virtual-machines/windows/sql-vm-create-portal-quickstart.md).

## <a name="configure-vnet-peering"></a>Настройка пиринга виртуальных сетей

Настройте пиринг виртуальных сетей для взаимодействия между виртуальной сетью с двумя управляемыми экземплярами и виртуальной сетью SQL Server. Для этого используйте следующий фрагмент кода PowerShell.

```powershell-interactive
# Set variables
$SubscriptionId = '<SubscriptionID>'
$resourceGroup = 'SQLMI-Repl'
$pubvNet = 'sql-mi-publisher-vnet'
$subvNet = 'sql-vm-sub-vnet'
$pubsubName = 'Pub-to-Sub-Peer'
$subpubName = 'Sub-to-Pub-Peer'

$virtualNetwork1 = Get-AzVirtualNetwork `
  -ResourceGroupName $resourceGroup `
  -Name $pubvNet

 $virtualNetwork2 = Get-AzVirtualNetwork `
  -ResourceGroupName $resourceGroup `
  -Name $subvNet  

# Configure VNet peering from publisher to subscriber
Add-AzVirtualNetworkPeering `
  -Name $pubsubName `
  -VirtualNetwork $virtualNetwork1 `
  -RemoteVirtualNetworkId $virtualNetwork2.Id

# Configure VNet peering from subscriber to publisher
Add-AzVirtualNetworkPeering `
  -Name $subpubName `
  -VirtualNetwork $virtualNetwork2 `
  -RemoteVirtualNetworkId $virtualNetwork1.Id

# Check status of peering on the publisher VNet; should say connected
Get-AzVirtualNetworkPeering `
 -ResourceGroupName $resourceGroup `
 -VirtualNetworkName $pubvNet `
 | Select PeeringState

# Check status of peering on the subscriber VNet; should say connected
Get-AzVirtualNetworkPeering `
 -ResourceGroupName $resourceGroup `
 -VirtualNetworkName $subvNet `
 | Select PeeringState

```

Настроив пиринг виртуальных сетей, проверьте подключение, запустив SQL Server Management Studio (SSMS) на сервере SQL Server и подключившись к обоим управляемым экземплярам. См. сведения о том, как [создать подключение к управляемому экземпляру SQL с использованием SSMS](point-to-site-p2s-configure.md#connect-with-ssms).

![Проверка возможности подключения к управляемым экземплярам](./media/replication-two-instances-and-sql-server-configure-tutorial/test-connectivity-to-mi.png)

## <a name="create-a-private-dns-zone"></a>Создание частной зоны DNS

Частная зона DNS позволяет организовать маршрутизацию DNS между управляемыми экземплярами и SQL Server.

### <a name="create-a-private-dns-zone"></a>Создание частной зоны DNS

1. Войдите на [портал Azure](https://portal.azure.com).
1. Щелкните **Создать ресурс**, чтобы создать ресурс Azure.
1. Выполните в Azure Marketplace поиск по строке `private dns zone`.
1. Выберите ресурс **Частная зона DNS**, опубликованный корпорацией Майкрософт, и щелкните **Создать** для создания новой зоны DNS.
1. Выберите нужные подписку и группу ресурсов в раскрывающихся меню.
1. Укажите для зоны DNS произвольное имя, например `repldns.com`.

   ![Создание частной зоны DNS](./media/replication-two-instances-and-sql-server-configure-tutorial/create-private-dns-zone.png)

1. Выберите **Просмотреть и создать**. Просмотрите параметры частной зоны DNS и щелкните **Создать**, чтобы создать этот ресурс.

### <a name="create-an-a-record"></a>Создание записи A

1. Перейдите к новому ресурсу **Частная зона DNS** и щелкните **Обзор**.
1. Щелкните **+Набор записей**, чтобы создать новую запись A.
1. Укажите имя виртуальной машины SQL Server и частный внутренний IP-адрес.

   ![Настройка записи A](./media/replication-two-instances-and-sql-server-configure-tutorial/configure-a-record.png)

1. Щелкните **ОК**, чтобы создать запись A.

### <a name="link-the-virtual-network"></a>Связывание виртуальной сети

1. Перейдите к новому ресурсу **Частная зона DNS** и щелкните **Связи виртуальных сетей**.
1. Щелкните **+ Добавить**.
1. Укажите имя для связи, например `Pub-link`.
1. Выберите в раскрывающемся списке нужную подписку, а затем выберите виртуальную сеть для управляемого экземпляра издателя.
1. Установите флажок **Включить авторегистрацию**.

   ![Создание связи с виртуальной сетью](./media/replication-two-instances-and-sql-server-configure-tutorial/configure-vnet-link.png)

1. Щелкните **ОК**, чтобы установить связь с виртуальной сетью.
1. Повторите эти шаги, чтобы добавить ссылку на виртуальную сеть подписчика, указав для нее имя `Sub-link`.

## <a name="create-an-azure-storage-account"></a>Создание учетной записи хранения Azure

[Создайте учетную запись хранения Azure](../../storage/common/storage-account-create.md#create-a-storage-account) для рабочей папки, а затем создайте в этой учетной записи [общую папку](../../storage/files/storage-how-to-create-file-share.md).

Скопируйте путь к общей папке в формате `\\storage-account-name.file.core.windows.net\file-share-name`.

Например, `\\replstorage.file.core.windows.net\replshare`.

Скопируйте строку подключения к ключом доступа к хранилищу в формате `DefaultEndpointsProtocol=https;AccountName=<Storage-Account-Name>;AccountKey=****;EndpointSuffix=core.windows.net`.

Например, `DefaultEndpointsProtocol=https;AccountName=replstorage;AccountKey=dYT5hHZVu9aTgIteGfpYE64cfis0mpKTmmc8+EP53GxuRg6TCwe5eTYWrQM4AmQSG5lb3OBskhg==;EndpointSuffix=core.windows.net`.

См. сведения о том, как [управлять ключами доступа к учетной записи хранения](../../storage/common/storage-account-keys-manage.md).

## <a name="create-a-database"></a>Создание базы данных

Создайте новую базу данных в управляемом экземпляре издателя. Для этого выполните следующие действия.

1. Запустите SQL Server Management Studio на SQL Server.
1. Подключитесь к управляемому экземпляру `sql-mi-publisher`.
1. Откройте окно **Новый запрос** и выполните следующую инструкцию T-SQL, чтобы создать базу данных.

```sql
-- Create the databases
USE [master]
GO

-- Drop database if it exists
IF EXISTS (SELECT * FROM sys.sysdatabases WHERE name = 'ReplTutorial')
BEGIN
    DROP DATABASE ReplTutorial
END
GO

-- Create new database
CREATE DATABASE [ReplTutorial]
GO

-- Create table
USE [ReplTutorial]
GO
CREATE TABLE ReplTest (
   ID INT NOT NULL PRIMARY KEY,
   c1 VARCHAR(100) NOT NULL,
   dt1 DATETIME NOT NULL DEFAULT getdate()
)
GO

-- Populate table with data
USE [ReplTutorial]
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

## <a name="configure-distribution"></a>Настройка распространения

Итак, подключение установлено и у вас есть пример базы данных. Теперь вы можете настроить распространение в управляемом экземпляре `sql-mi-distributor`. Для этого выполните следующие действия.

1. Запустите SQL Server Management Studio на SQL Server.
1. Подключитесь к управляемому экземпляру `sql-mi-distributor`.
1. Откройте окно **Новый запрос** и выполните следующий код Transact-SQL, чтобы настроить распространение в управляемом экземпляре распространителя.

   ```sql
   EXEC sp_adddistributor @distributor = 'sql-mi-distributor.b6bf57.database.windows.net', @password = '<distributor_admin_password>'
   
   EXEC sp_adddistributiondb @database = N'distribution'
   
   EXEC sp_adddistpublisher @publisher = 'sql-mi-publisher.b6bf57.database.windows.net', -- primary publisher
        @distribution_db = N'distribution',
        @security_mode = 0,
        @login = N'azureuser',
        @password = N'<publisher_password>',
        @working_directory = N'\\replstorage.file.core.windows.net\replshare',
        @storage_connection_string = N'<storage_connection_string>'
        -- example: @storage_connection_string = N'DefaultEndpointsProtocol=https;AccountName=replstorage;AccountKey=dYT5hHZVu9aTgIteGfpYE64cfis0mpKTmmc8+EP53GxuRg6TCwe5eTYWrQM4AmQSG5lb3OBskhg==;EndpointSuffix=core.windows.net'

   ```

   > [!NOTE]
   > В параметре @working_directory используйте только символы обратной черты (`\`). Использование символа прямой косой черты (`/`) может привести к ошибке при подключении к общей папке.

1. Подключитесь к управляемому экземпляру `sql-mi-publisher`.
1. Откройте окно **Новый запрос** и выполните следующий код Transact-SQL, чтобы зарегистрировать распространитель в издателе.

   ```sql
   Use MASTER
   EXEC sys.sp_adddistributor @distributor = 'sql-mi-distributor.b6bf57.database.windows.net', @password = '<distributor_admin_password>'
   ```

## <a name="create-the-publication"></a>Создание публикации

Настроив распространение, можно создать публикацию. Для этого выполните следующие действия.

1. Запустите SQL Server Management Studio на SQL Server.
1. Подключитесь к управляемому экземпляру `sql-mi-publisher`.
1. В **обозревателе объектов** разверните узел **Репликация** и щелкните папку **Локальная публикация**. Щелкните **Создать публикацию**.
1. Щелкните **Далее**, чтобы перейти со страницы приветствия.
1. На странице **База данных публикации** выберите созданную ранее базу данных `ReplTutorial`. Выберите **Далее**.
1. На странице **Тип публикации** выберите **Публикация транзакций**. Выберите **Далее**.
1. На странице **Статьи** установите флажок **Таблицы**. Выберите **Далее**.
1. На странице **Фильтрация строк таблицы** щелкните **Далее**, не добавляя новых фильтров.
1. На странице **Агент моментальных снимков** установите флажок **Создать моментальный снимок немедленно и обеспечить доступ к нему для инициализации подписок**. Выберите **Далее**.
1. На странице **Безопасность агента** щелкните **Параметры безопасности...** . Укажите учетные данные SQL Server для агента моментальных снимков и подключения к издателю. Щелкните **OK**, чтобы закрыть страницу **Безопасность агента моментальных снимков**. Выберите **Далее**.

   ![Настройка безопасности агента моментальных снимков](./media/replication-two-instances-and-sql-server-configure-tutorial/snapshot-agent-security.png)

1. На странице **Действия мастера** щелкните **Создать публикацию** и (необязательно) выберите **Сформировать файл скрипта с этапами создания публикации**, если вам нужен скрипт для использования в будущем.
1. На странице **Завершение работы мастера** присвойте публикации имя `ReplTest` и щелкните **Далее**, чтобы создать эту публикацию.
1. Создав публикацию, обновите узел **Репликация** в **обозревателе объектов** и разверните узел **Локальные публикации**, чтобы увидеть новую публикацию.

## <a name="create-the-subscription"></a>Создание подписки

После создания публикации можно создать подписку. Для этого выполните следующие действия.

1. Запустите SQL Server Management Studio на SQL Server.
1. Подключитесь к управляемому экземпляру `sql-mi-publisher`.
1. Откройте окно **Новый запрос** и выполните следующий код Transact-SQL, чтобы добавить подписку и агент распространителя. Включите имя DNS в имя подписчика.

```sql
use [ReplTutorial]
exec sp_addsubscription
@publication = N'ReplTest',
@subscriber = N'sql-vm-sub.repldns.com', -- include the DNS configured in the private DNS zone
@destination_db = N'ReplSub',
@subscription_type = N'Push',
@sync_type = N'automatic',
@article = N'all',
@update_mode = N'read only',
@subscriber_type = 0

exec sp_addpushsubscription_agent
@publication = N'ReplTest',
@subscriber = N'sql-vm-sub.repldns.com', -- include the DNS configured in the private DNS zone
@subscriber_db = N'ReplSub',
@job_login = N'azureuser',
@job_password = '<Complex Password>',
@subscriber_security_mode = 0,
@subscriber_login = N'azureuser',
@subscriber_password = '<Complex Password>',
@dts_package_location = N'Distributor'
GO
```

## <a name="test-replication"></a>Проверка репликации

Настроенную репликацию можно протестировать, добавив новые элементы в издатель и убедившись, что изменения распространяются на подписчик.

Выполните следующий фрагмент кода T-SQL, чтобы просмотреть строки в подписчике:

```sql
Use ReplSub
select * from dbo.ReplTest
```

Выполните следующий фрагмент кода T-SQL, чтобы вставить в издатель дополнительные строки, а затем снова проверьте строки в подписчике.

```sql
Use ReplTutorial
INSERT INTO ReplTest (ID, c1) VALUES (15, 'pub')
```

## <a name="clean-up-resources"></a>Очистка ресурсов

1. На [портале Azure](https://portal.azure.com) перейдите к используемой группе ресурсов.
1. Выберите один или несколько управляемых экземпляров и щелкните **Удалить**. Введите `yes` в текстовое поле, чтобы подтвердить удаление ресурса, а затем щелкните **Удалить**. Этот процесс некоторое время выполняется в фоновом режиме, и до его завершения вы не сможете удалить *виртуальный кластер* и другие зависимые ресурсы. Вы можете проверить состояние процесса удаления на вкладке **Действие**, чтобы убедиться в удалении управляемого экземпляра.
1. После удаления управляемого экземпляра удалите *виртуальный кластер*, выбрав его в группе ресурсов и щелкнув **Удалить**. Введите `yes` в текстовое поле, чтобы подтвердить удаление ресурса, а затем щелкните **Удалить**.
1. Удалите все остальные ресурсы. Введите `yes` в текстовое поле, чтобы подтвердить удаление ресурса, а затем щелкните **Удалить**.
1. Удалите группу ресурсов, выбрав **Удалить группу ресурсов**, введя в текстовом поле имя группы ресурсов `myResourceGroup` и щелкнув **Удалить**.

## <a name="known-errors"></a>Известные ошибки

### <a name="windows-logins-are-not-supported"></a>Имена входа Windows не поддерживаются

`Exception Message: Windows logins are not supported in this version of SQL Server.`

Это означает, что для агента настроено имя входа Windows, вместо которого нужно использовать имя входа SQL Server. Чтобы указать имя входа SQL Server в качестве учетных данных, воспользуйтесь страницей **Безопасность агента** в разделе **Свойства публикации**.

### <a name="failed-to-connect-to-azure-storage"></a>Не удалось подключиться к службе хранилища Azure

`Connecting to Azure Files Storage '\\replstorage.file.core.windows.net\replshare' Failed to connect to Azure Storage '' with OS error: 53.`

2019-11-19 02:21:05.07 Obtained Azure Storage Connection String for replstorage 2019-11-19 02:21:05.07 Connecting to Azure Files Storage '\\replstorage.file.core.windows.net\replshare' 2019-11-19 02:21:31.21 Failed to connect to Azure Storage '' with OS error: 53.

Такая ошибка чаще всего возникает из-за того, что порт 445 закрыт на Брандмауэре Azure и (или) брандмауэре Windows.

`Connecting to Azure Files Storage '\\replstorage.file.core.windows.net\replshare' Failed to connect to Azure Storage '' with OS error: 55.`

Такую ошибку также может вызывать использование прямой косой черты вместо обратной косой черты в пути к общей папке.
  
  - Такой вариант работает нормально: `\\replstorage.file.core.windows.net\replshare`
  - Такой может привести к ошибке OS 55: `'\\replstorage.file.core.windows.net/replshare'`

### <a name="could-not-connect-to-subscriber"></a>Не удалось подключиться к подписчику

`The process could not connect to Subscriber 'SQL-VM-SUB`
`Could not open a connection to SQL Server [53].`
`A network-related or instance-specific error has occurred while establishing a connection to SQL Server. Server is not found or not accessible. Check if instance name is correct and if SQL Server is configured to allow remote connections.`

Возможные решения

- Убедитесь, что порт 1433 открыт.
- Убедитесь, что в подписчике разрешен протокол TCP/IP.
- Убедитесь, что при создании подписчика использовалось имя DNS.
- Убедитесь, что виртуальные сети правильно связаны в частной зоне DNS.
- Убедитесь, что запись A настроена правильно.
- Убедитесь, что пиринг виртуальных сетей настроен правильно.

### <a name="no-publications-to-which-you-can-subscribe"></a>Отсутствие публикаций, на которые можно подписаться

При добавлении новой подписки с помощью мастера **Новые подписки** на странице **Публикация** вы можете заметить, что там не указано ни одной доступной базы данных или публикации, и получить следующее сообщение об ошибке:

`There are no publications to which you can subscribe, either because this server has no publications or because you do not have sufficient privileges to access the publications.`

Такое сообщение может означать, что в издателе, к которому вы подключились, действительно отсутствуют доступные публикации или у вас нет для них достаточных разрешений. Но эта ошибка может быть вызвана и использованием устаревшей версии SQL Server Management Studio. Попробуйте выполнить обновление до SQL Server Management Studio 18.0 или более поздней версии, чтобы исключить эту причину.

## <a name="next-steps"></a>Дальнейшие действия

### <a name="enable-security-features"></a>Включение функций безопасности

Полный список способов защиты базы данных приведен в статье [Что такое Управляемый экземпляр SQL Azure?](sql-managed-instance-paas-overview.md#advanced-security-and-compliance). Рассматриваются следующие средства безопасности:

- [Аудит управляемого экземпляра SQL](auditing-configure.md)
- [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine)
- [Обнаружение угроз](threat-detection-configure.md)
- [Динамическое маскирование данных](/sql/relational-databases/security/dynamic-data-masking)
- [Безопасность на уровне строк](/sql/relational-databases/security/row-level-security)
- [Прозрачное шифрование данных (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql)

### <a name="sql-managed-instance-capabilities"></a>Возможности Управляемого экземпляра SQL

Полный обзор возможностей управляемого экземпляра см. по следующей ссылке:

> [!div class="nextstepaction"]
> [Возможности Управляемого экземпляра SQL](sql-managed-instance-paas-overview.md)