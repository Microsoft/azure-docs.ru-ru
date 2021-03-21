---
title: 'PowerShell: миграция SQL Server в базу данных SQL'
titleSuffix: Azure Database Migration Service
description: Узнайте, как перенести базу данных из SQL Server в базу данных SQL Azure с помощью Azure PowerShell с Azure Database Migration Service.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019, devx-track-azurepowershell
ms.topic: how-to
ms.date: 02/20/2020
ms.openlocfilehash: a8f7e14500fb377b46f651b53e2704d8477aea7a
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102520665"
---
# <a name="migrate-a-sql-server-database-to-azure-sql-database-using-azure-powershell"></a>Перенос базы данных SQL Server в базу данных SQL Azure с помощью Azure PowerShell

В этой статье вы перенесите базу данных **Adventureworks2012** , которая была восстановлена в локальный экземпляр SQL Server 2016 или более поздней версии, в базу данных SQL Azure с помощью Microsoft Azure PowerShell. Вы можете перенести базы данных из экземпляра SQL Server в базу данных SQL Azure с помощью `Az.DataMigration` модуля в Microsoft Azure PowerShell.

Вы узнаете, как выполнять следующие задачи:
> [!div class="checklist"]
>
> * Создайте группу ресурсов.
> * создание экземпляра Azure Database Migration Service;
> * создание проекта миграции в экземпляре Azure Database Migration Service;
> * выполнение миграции.

## <a name="prerequisites"></a>Предварительные условия

Для выполнения этих действий вам потребуется следующее:

* [SQL Server 2016 или более поздней версии](https://www.microsoft.com/sql-server/sql-server-downloads) (любой выпуск).
* При установке SQL Server Express включите протокол TCP/IP, который отключен по умолчанию. Для этого выполните инструкции в разделе [Использование диспетчера конфигурации SQL Server](/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol#SSMSProcedure) статьи "Включение или отключение сетевого протокола сервера".
* [Настройте брандмауэр Windows для доступа к ядру СУБД](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Экземпляр Базы данных SQL Azure. Экземпляр базы данных SQL Azure можно создать, следуя сведениям в статье [Создание базы данных в базе данных SQL Azure на портал Azure](../azure-sql/database/single-database-create-quickstart.md).
* [Помощник по миграции данных](https://www.microsoft.com/download/details.aspx?id=53595) 3.3 или более поздней версии.
* Чтобы создать виртуальная сеть Microsoft Azure с помощью модели развертывания Azure Resource Manager, которая предоставляет Azure Database Migration Service с подключением типа "сеть — сеть" к локальным исходным серверам с помощью [ExpressRoute](../expressroute/expressroute-introduction.md) или [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md).
* Чтобы выполнить оценку локальной базы данных и миграции схемы с помощью Помощник по миграции данных, как описано в статье [выполнение SQL Serverной оценки миграции](/sql/dma/dma-assesssqlonprem) .
* Чтобы скачать и установить модуль AZ. Migration из коллекция PowerShell, используйте [командлет PowerShell Install-Module](/powershell/module/powershellget/Install-Module). не забудьте открыть командное окно PowerShell с помощью команды Запуск от имени администратора.
* Убедитесь, что учетные данные, используемые для подключения к исходному экземпляру SQL Server, имеют разрешение [CONTROL SERVER](/sql/t-sql/statements/grant-server-permissions-transact-sql).
* Убедитесь, что учетные данные, используемые для подключения к целевому экземпляру Базы данных SQL Azure, имеют разрешение на управление базами данных в целевой базе данных SQL Azure.
* Подписка Azure. Если у вас ее нет, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись, прежде чем начинать работу.

## <a name="log-in-to-your-microsoft-azure-subscription"></a>Вход в подписку Microsoft Azure

Чтобы войти в подписку Azure с помощью PowerShell, следуйте инструкциям в статье [Вход с помощью Azure PowerShell](/powershell/azure/authenticate-azureps).

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. Прежде чем создать виртуальную машину, создайте группу ресурсов.

Создайте группу ресурсов с помощью команды [New-азресаурцеграуп](/powershell/module/az.resources/new-azresourcegroup) .

В следующем примере создается группа ресурсов с именем *myResourceGroup* в регионе *EastUS* .

```powershell
New-AzResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```

## <a name="create-an-instance-of-azure-database-migration-service"></a>Создание экземпляра Azure Database Migration Service

Вы можете создать экземпляр Azure Database Migration Service, выполнив командлет `New-AzDataMigrationService`. Для этого командлета нужно передать следующие обязательные параметры:

* *Имя группы ресурсов Azure*. Вы можете использовать команду [New-азресаурцеграуп](/powershell/module/az.resources/new-azresourcegroup) , чтобы создать группу ресурсов Azure, как показано выше, и указать ее имя в качестве параметра.
* *Имя службы*. Строка, соответствующая требуемому уникальному имени службы для Azure Database Migration Service. 
* *Расположение*. Указывает расположение службы. Укажите расположение центра обработки данных Azure, например западная часть США или Юго-Восточная Азия.
* *Номер SKU*. Этот параметр соответствует имени SKU DMS. Поддерживаемое сейчас имя Sku — *GeneralPurpose_4vCores*.
* *VirtualSubnetId.* Для создания подсети можно использовать командлет [New-азвиртуалнетворксубнетконфиг](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) . 

В указанном ниже примере создается служба с именем *MyDMS* в группе ресурсов *MyDMSResourceGroup*, которая расположена в регионе *Восточная часть США*, с использованием виртуальной сети с именем *MyVNET* и подсети с именем *MySubnet*.

```powershell
 $vNet = Get-AzVirtualNetwork -ResourceGroupName MyDMSResourceGroup -Name MyVNET

$vSubNet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vNet -Name MySubnet

$service = New-AzDms -ResourceGroupName myResourceGroup `
  -ServiceName MyDMS `
  -Location EastUS `
  -Sku Basic_2vCores `  
  -VirtualSubnetId $vSubNet.Id`
```

## <a name="create-a-migration-project"></a>Создание проекта миграции

Создав экземпляр Azure Database Migration Service, создайте проект миграции. Для проекта Azure Database Migration Service необходимо иметь сведения о подключении для исходных и целевых экземпляров, а также список баз данных, которые требуется перенести как часть проекта.

### <a name="create-a-database-connection-info-object-for-the-source-and-target-connections"></a>Создание объекта сведений о подключении к базе данных для исходных и целевых подключений

Объект сведений о подключении к базе данных можно создать, выполнив командлет `New-AzDmsConnInfo`. Для этого командлета требуются следующие параметры:

* *ServerType*. Тип запрошенного подключения к базе данных, например SQL, Oracle или MySQL. Используйте SQL для SQL Server и SQL Azure.
* *Источник данных*. Имя или IP-адрес SQL Server экземпляра или базы данных SQL Azure.
* *AuthType*. Тип проверки подлинности для подключения. Это может быть SqlAuthentication или WindowsAuthentication.
* Параметр *TrustServerCertificate* задает значение, указывающее, будет ли шифроваться канал при проходе по цепочке сертификатов для проверки доверия. Значение может быть true или false.

В приведенном ниже примере создается объект сведений о подключении для исходного SQL Server MySourceSQLServer с использованием проверки подлинности SQL:

```powershell
$sourceConnInfo = New-AzDmsConnInfo -ServerType SQL `
  -DataSource MySourceSQLServer `
  -AuthType SqlAuthentication `
  -TrustServerCertificate:$true
```

В следующем примере показано создание сведений о соединении для сервера с именем Склазуретаржет с использованием проверки подлинности SQL:

```powershell
$targetConnInfo = New-AzDmsConnInfo -ServerType SQL `
  -DataSource "sqlazuretarget.database.windows.net" `
  -AuthType SqlAuthentication `
  -TrustServerCertificate:$false
```

### <a name="provide-databases-for-the-migration-project"></a>Указание баз данных для проекта миграции

Создайте список объектов `AzDataMigrationDatabaseInfo`, в котором указаны базы данных, переносимые в рамках проекта миграции баз данных Azure. Его можно предоставить в виде параметра для создания проекта. Командлет `New-AzDataMigrationDatabaseInfo` можно использовать для создания аздатамигратиондатабасеинфо. 

В следующем примере создается проект `AzDataMigrationDatabaseInfo` для базы данных **AdventureWorks2016**, который добавляется в список, чтобы его можно было предоставить в виде параметра для создания проекта.

```powershell
$dbInfo1 = New-AzDataMigrationDatabaseInfo -SourceDatabaseName AdventureWorks2016
$dbList = @($dbInfo1)
```

### <a name="create-a-project-object"></a>Создание объекта проекта

Наконец, можно создать проект миграции баз данных Azure с именем *MyDMSProject*, который расположен в *восточной части США*, выполнив команду `New-AzDataMigrationProject` и добавив раннее созданные исходные и целевые подключения и список баз данных для миграции.

```powershell
$project = New-AzDataMigrationProject -ResourceGroupName myResourceGroup `
  -ServiceName $service.Name `
  -ProjectName MyDMSProject `
  -Location EastUS `
  -SourceType SQL `
  -TargetType SQLDB `
  -SourceConnection $sourceConnInfo `
  -TargetConnection $targetConnInfo `
  -DatabaseInfo $dbList
```

## <a name="create-and-start-a-migration-task"></a>Создание и запуск задачи миграции

Наконец, создайте и запустите задачу миграции баз данных Azure. Помимо информации, предоставленной с проектом, созданным в качестве предварительного условия, для задачи миграции баз данных Azure необходимо иметь учетные данные для исходного и целевого подключения, а также список таблиц базы данных, которые необходимо перенести.

### <a name="create-credential-parameters-for-source-and-target"></a>Создание параметров учетных данных для исходного и целевого подключения

Учетные данные безопасности подключения можно создать в качестве объекта [PSCredential](/dotnet/api/system.management.automation.pscredential?view=powershellsdk-1.1.0).

В следующем примере показано создание объектов *PSCredential* для исходного и целевого подключения. Пароли представлены в качестве строковых переменных *$sourcePassword* и *$targetPassword*.

```powershell
$secpasswd = ConvertTo-SecureString -String $sourcePassword -AsPlainText -Force
$sourceCred = New-Object System.Management.Automation.PSCredential ($sourceUserName, $secpasswd)
$secpasswd = ConvertTo-SecureString -String $targetPassword -AsPlainText -Force
$targetCred = New-Object System.Management.Automation.PSCredential ($targetUserName, $secpasswd)
```

### <a name="create-a-table-map-and-select-source-and-target-parameters-for-migration"></a>Создание сопоставления таблицы и выбор исходных и целевых параметров для миграции

Еще один параметр, необходимый для миграции, — это сопоставление исходных и целевых таблиц для миграции. Создайте словарь таблиц, который обеспечивает сопоставление между исходными и целевыми таблицами для миграции. В следующем примере показано сопоставление между исходными и целевыми таблицами схемы управления персоналом для базы данных AdventureWorks 2016.

```powershell
$tableMap = New-Object 'system.collections.generic.dictionary[string,string]'
$tableMap.Add("HumanResources.Department", "HumanResources.Department")
$tableMap.Add("HumanResources.Employee","HumanResources.Employee")
$tableMap.Add("HumanResources.EmployeeDepartmentHistory","HumanResources.EmployeeDepartmentHistory")
$tableMap.Add("HumanResources.EmployeePayHistory","HumanResources.EmployeePayHistory")
$tableMap.Add("HumanResources.JobCandidate","HumanResources.JobCandidate")
$tableMap.Add("HumanResources.Shift","HumanResources.Shift")
```

Теперь необходимо выбрать исходную и целевую базы данных и обеспечить сопоставление мигрируемых таблиц в качестве параметра, выполнив командлет `New-AzDmsSelectedDB`, как показано в следующем примере:

```powershell
$selectedDbs = New-AzDmsSelectedDB -MigrateSqlServerSqlDb -Name AdventureWorks2016 `
  -TargetDatabaseName AdventureWorks2016 `
  -TableMap $tableMap
```

### <a name="create-the-migration-task-and-start-it"></a>Создание задачи миграции и ее запуск

Выполните командлет `New-AzDataMigrationTask`, чтобы создать и запустить задачу миграции. Для этого командлета требуются следующие параметры:

* *TaskType.* Для типа миграции "из SQL Server в Базу данных SQL Azure" в создаваемой задаче миграции ожидается тип *MigrateSqlServerSqlDb*. 
* *Имя группы ресурсов*. Имя группы ресурсов Azure, в которой будет создана задача.
* *ServiceName*. Экземпляр Azure Database Migration Service, в котором будет создана задача.
* *Имя_проекта*. Имя проекта Azure Database Migration Service, в котором будет создана задача. 
* *Имя_задания*. Имя задачи, которая будет создана. 
* *SourceConnection*. Объект Аздмсконнинфо, представляющий соединение с источником SQL Server.
* *TargetConnection.* Объект Аздмсконнинфо, представляющий целевое подключение к базе данных SQL Azure.
* *SourceCred.* Объект [PSCredential](/dotnet/api/system.management.automation.pscredential?view=powershellsdk-1.1.0) для подключения к исходному серверу.
* *TargetCred.* Объект [PSCredential](/dotnet/api/system.management.automation.pscredential?view=powershellsdk-1.1.0) для подключения к целевому серверу.
* *SelectedDatabase.* Объект Аздатамигратионселектеддб, представляющий сопоставление исходного и целевого базы данных.
* *Счемавалидатион*. (необязательный, параметр Switch) После миграции выполняется сравнение сведений о схеме между источником и целевым объектом.
* *Датаинтегритивалидатион*. (необязательный, параметр Switch) После миграции выполняется проверка целостности данных на основе контрольной суммы между исходным и целевым версиями.
* *Куеряналисисвалидатион*. (необязательный, параметр Switch) После миграции выполняется быстрый и интеллектуальный анализ запросов путем извлечения запросов из базы данных-источника и выполнения их в целевом объекте.

В следующем примере создается и запускается задача миграции с именем myDMSTask:

```powershell
$migTask = New-AzDataMigrationTask -TaskType MigrateSqlServerSqlDb `
  -ResourceGroupName myResourceGroup `
  -ServiceName $service.Name `
  -ProjectName $project.Name `
  -TaskName myDMSTask `
  -SourceConnection $sourceConnInfo `
  -SourceCred $sourceCred `
  -TargetConnection $targetConnInfo `
  -TargetCred $targetCred `
  -SelectedDatabase  $selectedDbs `
```

В следующем примере создается и запускается та же задача миграции, что и ранее, но также выполняются все три проверки:

```powershell
$migTask = New-AzDataMigrationTask -TaskType MigrateSqlServerSqlDb `
  -ResourceGroupName myResourceGroup `
  -ServiceName $service.Name `
  -ProjectName $project.Name `
  -TaskName myDMSTask `
  -SourceConnection $sourceConnInfo `
  -SourceCred $sourceCred `
  -TargetConnection $targetConnInfo `
  -TargetCred $targetCred `
  -SelectedDatabase  $selectedDbs `
  -SchemaValidation `
  -DataIntegrityValidation `
  -QueryAnalysisValidation `
```

## <a name="monitor-the-migration"></a>Мониторинг миграции

Вы можете отслеживать выполняемую задачу миграции, запросив свойство состояния задачи, как показано в следующем примере:

```powershell
if (($mytask.ProjectTask.Properties.State -eq "Running") -or ($mytask.ProjectTask.Properties.State -eq "Queued"))
{
  write-host "migration task running"
}
```

## <a name="deleting-the-dms-instance"></a>Удаление экземпляра DMS

После завершения миграции можно удалить экземпляр Azure Database Migration Service.

```powershell
Remove-AzDms -ResourceGroupName myResourceGroup -ServiceName MyDMS
```

## <a name="next-step"></a>Следующий шаг

* Просмотрите [руководство по миграции базы данных Майкрософт](https://datamigration.microsoft.com/).