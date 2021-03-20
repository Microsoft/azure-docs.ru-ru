---
title: Развертывание Управляемый экземпляр SQL в пуле экземпляров
titleSuffix: Azure SQL Managed Instance
description: В этой статье описывается создание пулов Управляемый экземпляр Azure SQL (Предварительная версия) и управление ими.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: bonova
ms.author: bonova
ms.reviewer: sstein
ms.date: 09/05/2019
ms.openlocfilehash: 732a9bab018103321a9a3a759b31495bd67a209e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92793115"
---
# <a name="deploy-azure-sql-managed-instance-to-an-instance-pool"></a>Развертывание Управляемый экземпляр SQL Azure в пуле экземпляров
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

В этой статье приводятся сведения о том, как создать [пул экземпляров](instance-pools-overview.md) и развернуть в нем управляемый экземпляр SQL Azure. 

## <a name="instance-pool-operations"></a>Операции с пулом экземпляров

В следующей таблице показаны доступные операции, связанные с пулами экземпляров, и их доступность на портале Azure и PowerShell.

|Get-Help|Портал Azure|PowerShell|
|:---|:---|:---|
|Создание пула экземпляров|Нет|Да|
|Обновление пула экземпляров (ограниченное число свойств)|Нет |Да |
|Проверка использования и свойств пула экземпляров|Нет|Да |
|Удаление пула экземпляров|Нет|Да|
|Создание управляемого экземпляра в пуле экземпляров|Нет|Да|
|Обновление использования ресурсов для управляемого экземпляра|Да |Да|
|Проверка использования и свойств управляемого экземпляра|Да|Да|
|Удаление управляемого экземпляра из пула|Да|Да|
|Создание базы данных в экземпляре в пуле|Да|Да|
|Удаление базы данных из SQL Управляемый экземпляр|Да|Да|

Доступные [команды PowerShell](/powershell/module/az.sql/):

|Командлет |Описание |
|:---|:---|
|[New-AzSqlInstancePool](/powershell/module/az.sql/new-azsqlinstancepool/) | Создает пул Управляемый экземпляр SQL. |
|[Get-AzSqlInstancePool](/powershell/module/az.sql/get-azsqlinstancepool/) | Возвращает сведения о пуле экземпляров. |
|[Set-AzSqlInstancePool](/powershell/module/az.sql/set-azsqlinstancepool/) | Задает свойства для пула экземпляров в Управляемый экземпляр SQL. |
|[Remove-AzSqlInstancePool](/powershell/module/az.sql/remove-azsqlinstancepool/) | Удаляет пул экземпляров в Управляемый экземпляр SQL. |
|[Get-AzSqlInstancePoolUsage](/powershell/module/az.sql/get-azsqlinstancepoolusage/) | Возвращает сведения об использовании пула SQL Управляемый экземпляр. |


Чтобы использовать PowerShell, [установите последнюю версию PowerShell Core](/powershell/scripting/install/installing-powershell#powershell) и следуйте инструкциям по [установке модуля Azure PowerShell](/powershell/azure/install-az-ps).

Для операций, относящихся к экземплярам внутри пулов и отдельным экземплярам, используйте стандартные [команды управляемого экземпляра](api-references-create-manage-instance.md#powershell-create-and-configure-managed-instances), но при использовании этих команд для экземпляра в пуле необходимо заполнить свойство *имени пула экземпляров*.

## <a name="deployment-process"></a>Процесс развертывания

Чтобы развернуть управляемый экземпляр в пуле экземпляров, необходимо сначала развернуть пул экземпляров, который представляет собой одноразовую длительную операцию, в которой длительность совпадает с развертыванием [одного экземпляра, созданного в пустой подсети](sql-managed-instance-paas-overview.md#management-operations). После этого можно развернуть управляемый экземпляр в пуле, который является относительно быстрой операцией, которая обычно занимает до пяти минут. Параметр пула экземпляров должен быть явно указан в рамках этой операции.

В общедоступной предварительной версии оба действия поддерживаются только с помощью шаблонов PowerShell и Azure Resource Manager. Портал Azure в настоящее время недоступен.

После развертывания управляемого экземпляра в пуле *можно* использовать портал Azure для изменения свойств на странице ценовой категории.

## <a name="create-a-virtual-network-with-a-subnet"></a>Создание виртуальной сети с подсетью. 

Сведения о размещении нескольких пулов экземпляров в одной виртуальной сети см. в следующих статьях.

- [Определите размер подсети VNet для управляемый экземпляр Azure SQL](vnet-subnet-determine-size.md).
- Создайте новую виртуальную сеть и подсеть с помощью [шаблона портала Azure](virtual-network-subnet-create-arm-template.md) или следуйте инструкциям для [подготовки существующей виртуальной сети](vnet-existing-add-subnet.md).
 

## <a name="create-an-instance-pool"></a>Создание пула экземпляров 

После выполнения описанных выше действий можно приступать к созданию пула экземпляров.

На пулы экземпляров налагаются следующие ограничения.

- В общедоступной предварительной версии доступны только пулы общего назначения и 5-го поколения.
- Имя пула может содержать только строчные буквы, цифры и дефисы и не может начинаться с дефиса.
- Если вы хотите использовать Преимущество гибридного использования Azure, он применяется на уровне пула экземпляров. Вы можете задать тип лицензии во время создания пула или изменить его в любое время после создания.

> [!IMPORTANT]
> Развертывание пула экземпляров — это длительная операция, которая занимает примерно 4,5 часа.

Получение параметров сети:

```powershell
$virtualNetwork = Get-AzVirtualNetwork -Name "miPoolVirtualNetwork" -ResourceGroupName "myResourceGroup"
$subnet = Get-AzVirtualNetworkSubnetConfig -Name "miPoolSubnet" -VirtualNetwork $virtualNetwork
```

Создание пула экземпляров:

```powershell
$instancePool = New-AzSqlInstancePool `
  -ResourceGroupName "myResourceGroup" `
  -Name "mi-pool-name" `
  -SubnetId $subnet.Id `
  -LicenseType "LicenseIncluded" `
  -VCore 8 `
  -Edition "GeneralPurpose" `
  -ComputeGeneration "Gen5" `
  -Location "westeurope"
```

> [!IMPORTANT]
> Поскольку развертывание пула экземпляров является длительной операцией, необходимо подождать, пока она завершится, прежде чем выполнять следующие действия в этой статье.

## <a name="create-a-managed-instance"></a>Создание управляемого экземпляра

После успешного развертывания пула экземпляров найдется время создания управляемого экземпляра внутри него.

Чтобы создать управляемый экземпляр, выполните следующую команду:

```powershell
$instanceOne = $instancePool | New-AzSqlInstance -Name "mi-one-name" -VCore 2 -StorageSizeInGB 256
```

Развертывание экземпляра в пуле занимает несколько минут. После создания первого экземпляра можно создать дополнительные экземпляры:

```powershell
$instanceTwo = $instancePool | New-AzSqlInstance -Name "mi-two-name" -VCore 4 -StorageSizeInGB 512
```

## <a name="create-a-database"></a>Создание базы данных 

Для создания баз данных и управления ими в управляемом экземпляре в пуле, используйте команды одного экземпляра.

Создание базы данных в управляемом экземпляре:

```powershell
$poolinstancedb = New-AzSqlInstanceDatabase -Name "mipooldb1" -InstanceName "poolmi-001" -ResourceGroupName "myResourceGroup"
```


## <a name="get-pool-usage"></a>Получение сведений об использовании пула 
 
Получение списка экземпляров в пуле:

```powershell
$instancePool | Get-AzSqlInstance
```


Получение сведений об использовании ресурсов пула:

```powershell
$instancePool | Get-AzSqlInstancePoolUsage
```


Получение подробных сведений об использовании пула и экземпляров внутри него:

```powershell
$instancePool | Get-AzSqlInstancePoolUsage –ExpandChildren
```

Получение списка баз данных в экземпляре:

```powershell
$databases = Get-AzSqlInstanceDatabase -InstanceName "pool-mi-001" -ResourceGroupName "resource-group-name"
```


> [!NOTE]
> Существует ограничение в 100 баз данных на пул (не для каждого экземпляра).


## <a name="scale"></a>Масштабирование 


После заполнения управляемого экземпляра базами данных вы можете достичь ограничения на количество экземпляров в связи с хранением или производительностью. В этом случае, если ограничение на использование пула не превышено, можно масштабировать экземпляр.
Масштабирование управляемого экземпляра в пуле занимает несколько минут. Предварительные требования для масштабирования — виртуальные ядра и хранилище на уровне пула экземпляров.

Обновление числа виртуальных ядер и размера хранилища:

```powershell
$instanceOne | Set-AzSqlInstance -VCore 8 -StorageSizeInGB 512 -InstancePoolName "mi-pool-name"
```


Обновление только размера хранилища:

```powershell
$instance | Set-AzSqlInstance -StorageSizeInGB 1024 -InstancePoolName "mi-pool-name"
```

## <a name="connect"></a>Подключение 

Чтобы подключиться к управляемому экземпляру в пуле, необходимо выполнить два следующих действия:

1. [Включите общедоступную конечную точку для экземпляра](#enable-the-public-endpoint).
2. [Добавьте правило входящего трафика для группы безопасности сети](#add-an-inbound-rule-to-the-network-security-group).

После выполнения обоих шагов можно подключиться к экземпляру, используя адрес общедоступной конечной точки, порт и учетные данные, указанные во время создания экземпляра. 

### <a name="enable-the-public-endpoint"></a>Включение общедоступной конечной точки

Включение общедоступной конечной точки для экземпляра можно выполнить на портале Azure или с помощью следующей команды PowerShell:


```powershell
$instanceOne | Set-AzSqlInstance -InstancePoolName "pool-mi-001" -PublicDataEndpointEnabled $true
```

Этот параметр можно также задать во время создания экземпляра.

### <a name="add-an-inbound-rule-to-the-network-security-group"></a>Добавление правила входящего трафика для группы безопасности сети 

Этот шаг можно выполнить на портале Azure или с помощью команд PowerShell. Это можно сделать в любое время после подготовки подсети для управляемого экземпляра.

Дополнительные сведения см. в разделе [Разрешение общедоступного трафика конечной точки в группе безопасности сети](public-endpoint-configure.md#allow-public-endpoint-traffic-on-the-network-security-group).


## <a name="move-an-existing-single-instance-to-a-pool"></a>Перемещение существующего экземпляра в пул
 
Перемещение экземпляров в пул и из него является одним из ограничений общедоступной предварительной версии. Обходной путь зависит от восстановления баз данных на момент времени из экземпляра, находящегося за пределами пула, до экземпляра, который уже находится в пуле. 

Оба экземпляра должны относиться к той же подписке и тому же региону. Резервное копирование между регионами и восстановление между подписками в настоящее время не поддерживается.

Этот процесс подразумевает период простоя.

Перемещение существующих баз данных:

1. Приостановите рабочие нагрузки на управляемом экземпляре, с которого выполняется миграция.
2. Создайте скрипты для создания системных баз данных и выполните их на экземпляре, который находится внутри пула экземпляров.
3. Выполните восстановление каждой базы данных на момент времени из единичного экземпляра в экземпляр в пуле.

    ```powershell
    $resourceGroupName = "my resource group name"
    $managedInstanceName = "my managed instance name"
    $databaseName = "my source database name"
    $pointInTime = "2019-08-21T08:51:39.3882806Z"
    $targetDatabase = "name of the new database that will be created"
    $targetResourceGroupName = "resource group of instance pool"
    $targetInstanceName = "pool instance name"
       
    Restore-AzSqlInstanceDatabase -FromPointInTimeBackup `
      -ResourceGroupName $resourceGroupName `
      -InstanceName $managedInstanceName `
      -Name $databaseName `
      -PointInTime $pointInTime `
      -TargetInstanceDatabaseName $targetDatabase `
      -TargetResourceGroupName $targetResourceGroupName `
      -TargetInstanceName $targetInstanceName
    ```

4. Наведите приложение на новый экземпляр и возобновите его рабочие нагрузки.

Если имеется несколько баз данных, повторите эту процедуру для каждой базы данных.


## <a name="next-steps"></a>Дальнейшие действия

- Сведения о функциях и список сравнения см. в статье [Сравнение функций Базы данных SQL Azure и SQL Server](../database/features-comparison.md).
- Дополнительные сведения о конфигурации виртуальной сети см. в разделе [Архитектура подключения к Управляемому экземпляру SQL Azure](connectivity-architecture-overview.md).
- Сведения о создании управляемого экземпляра и восстановлении базы данных из файла резервной копии см. в разделе [Краткое руководство. Создание управляемого экземпляра Управляемого экземпляра SQL](instance-create-quickstart.md).
- Сведения о миграции с помощью Azure Database Migration Service см. в разделе [Руководство. Миграция из SQL Server в Управляемый экземпляр базы данных SQL Azure в автономном режиме с помощью DMS](../../dms/tutorial-sql-server-to-managed-instance.md).
- Сведения о расширенном мониторинге производительности базы данных Управляемого экземпляра SQL с использованием встроенных средств анализа проблем см. в статье о [мониторинге Управляемого экземпляра SQL Azure с помощью Аналитики SQL Azure](../../azure-monitor/insights/azure-sql.md).
- Сведения о ценах см. на странице [цен на SQL управляемый экземпляр](https://azure.microsoft.com/pricing/details/sql-database/managed/).