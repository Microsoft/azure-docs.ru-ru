---
title: Настройка периода обслуживания (Предварительная версия)
description: Узнайте, как задать время выполнения планового обслуживания для баз данных SQL Azure, эластичных пулов и баз данных управляемых экземпляров.
services: sql-database
ms.service: sql-db-mi
ms.subservice: service
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 03/23/2021
ms.openlocfilehash: 8688458d85084f3d3dab4678fa91ed827a337739
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105047357"
---
# <a name="configure-maintenance-window-preview"></a>Настройка периода обслуживания (Предварительная версия)
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]


Настройка [периода обслуживания (Предварительная версия)](maintenance-window.md) для базы данных SQL Azure, эластичного пула или базы данных sql Azure управляемый экземпляр во время создания ресурса или в любое время после создания ресурса. 

*По умолчанию* период обслуживания системы 17:00 8:00 ежедневно (Местное время в регионе Azure, где находится ресурс), чтобы избежать пиковых сбоев в рабочих часах. Если стандартное окно обслуживания *по умолчанию* не является лучшим, выберите один из других доступных периодов обслуживания.

Возможность изменения в другом окне обслуживания недоступна для каждого уровня обслуживания или в каждом регионе. Дополнительные сведения о доступности см. в разделе [обслуживание периода обслуживания](maintenance-window.md#availability).

> [!Important]
> Настройка периода обслуживания — это длительно выполняющаяся асинхронная операция, аналогичная изменению уровня служб для ресурса SQL Azure. Ресурс доступен во время операции, за исключением короткой перенастройки, которая происходит в конце операции и обычно длится до 8 секунд даже в случае прерываний долго выполняющихся транзакций. Чтобы уменьшить влияние перенастройки, необходимо выполнить операцию за пределами пиковых часов.

## <a name="configure-maintenance-window-during-database-creation"></a>Настройка периода обслуживания во время создания базы данных 

# <a name="portal"></a>[Портал](#tab/azure-portal)

Чтобы настроить период обслуживания при создании базы данных, эластичного пула или управляемого экземпляра, задайте нужное **окно обслуживания** на странице **Дополнительные параметры** . 

## <a name="set-the-maintenance-window-while-creating-a-single-database-or-elastic-pool"></a>Задание периода обслуживания при создании отдельной базы данных или эластичного пула

Пошаговые инструкции по созданию новой базы данных или пула см. в статье Создание базы данных [SQL Azure с помощью единой базы](single-database-create-quickstart.md)данных.

   :::image type="content" source="media/maintenance-window-configure/additional-settings.png" alt-text="Вкладка &quot;Создание дополнительных параметров базы данных&quot;":::


## <a name="set-the-maintenance-window-while-creating-a-managed-instance"></a>Задание периода обслуживания при создании управляемого экземпляра

Пошаговые инструкции по созданию нового управляемого экземпляра см. в статье [создание управляемый экземпляр Azure SQL](../managed-instance/instance-create-quickstart.md).

   :::image type="content" source="media/maintenance-window-configure/additional-settings-mi.png" alt-text="Создание управляемого экземпляра вкладка &quot;Дополнительные параметры&quot;":::




# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

В следующих примерах показано, как настроить период обслуживания с помощью Azure PowerShell. Можно [установить Azure PowerShell](/powershell/azure/install-az-ps)или использовать Azure Cloud Shell.

## <a name="launch-azure-cloud-shell"></a>Запуск Azure Cloud Shell

Azure Cloud Shell — это бесплатная интерактивная оболочка, с помощью которой можно выполнять действия, описанные в этой статье. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью. 

Чтобы открыть Cloud Shell, просто выберите **Попробовать** в правом верхнем углу блока кода. Cloud Shell можно также запустить в отдельной вкладке браузера, перейдя на страницу [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Нажмите кнопку **Копировать**, чтобы скопировать блоки кода. Вставьте код в Cloud Shell и нажмите клавишу "ВВОД", чтобы выполнить его.


## <a name="discover-available-maintenance-windows"></a>Обнаружение доступных периодов обслуживания

При настройке периода обслуживания каждый регион имеет собственные параметры периода обслуживания, соответствующие часовому поясу для региона, в котором находится база данных или пул. 

### <a name="discover-sql-database-and-elastic-pool-maintenance-windows"></a>Обнаружение окон базы данных SQL и обслуживания эластичных пулов 

В следующем примере возвращаются доступные периоды обслуживания для региона *eastus2* с помощью командлета [Get-азмаинтенанцепубликконфигуратион](/powershell/module/az.maintenance/get-azmaintenancepublicconfiguration) . Для баз данных и эластичных пулов задайте значение `MaintenanceScope` `SQLDB` .

   ```powershell-interactive
   $location = "eastus2"

   Write-Host "Available maintenance schedules in ${location}:"
   $configurations = Get-AzMaintenancePublicConfiguration
   $configurations | ?{ $_.Location -eq $location -and $_.MaintenanceScope -eq "SQLDB"}
   ```

### <a name="discover-sql-managed-instance-maintenance-windows"></a>Обнаружение окон обслуживания Управляемый экземпляр SQL 

В следующем примере возвращаются доступные периоды обслуживания для региона *eastus2* с помощью командлета [Get-азмаинтенанцепубликконфигуратион](/powershell/module/az.maintenance/get-azmaintenancepublicconfiguration) . Для управляемых экземпляров задайте `MaintenanceScope` для значение `SQLManagedInstance` .

   ```powershell-interactive
   $location = "eastus2"

   Write-Host "Available maintenance schedules in ${location}:"
   $configurations = Get-AzMaintenancePublicConfiguration
   $configurations | ?{ $_.Location -eq $location -and $_.MaintenanceScope -eq "SQLManagedInstance"}
   ```


## <a name="set-the-maintenance-window-while-creating-a-single-database"></a>Задание периода обслуживания при создании отдельной базы данных

В следующем примере создается новая база данных и задается окно обслуживания с помощью командлета [New-азсклдатабасе](/powershell/module/az.sql/new-azsqldatabase) . `-MaintenanceConfigurationId`Должно быть задано допустимое значение для региона базы данных. Чтобы получить допустимые значения для вашего региона, см. раздел [обнаружение доступных периодов обслуживания](#discover-available-maintenance-windows).


   ```powershell-interactive
    # Set variables for your database
    $resourceGroupName = "your_resource_group_name"
    $serverName = "your_server_name"
    $databaseName = "your_db_name"
    
    # Set selected maintenance window
    $maintenanceConfig = "SQL_EastUS2_DB_1"

    Write-host "Creating a gen5 2 vCore database with maintenance window ${maintenanceConfig} ..."
    $database = New-AzSqlDatabase `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -DatabaseName $databaseName `
      -Edition GeneralPurpose `
      -ComputeGeneration Gen5 `
      -VCore 2 `
      -MaintenanceConfigurationId $maintenanceConfig
    $database
   ```



## <a name="set-the-maintenance-window-while-creating-an-elastic-pool"></a>Настройка периода обслуживания при создании эластичного пула

В следующем примере создается пул эластичных БД, а также задается период обслуживания с помощью командлета [New-азсклеластикпул](/powershell/module/az.sql/new-azsqlelasticpool) . Период обслуживания задается в эластичном пуле, поэтому все базы данных в пуле имеют расписание периода обслуживания пула. `-MaintenanceConfigurationId`Должно быть задано допустимое значение для региона пула. Чтобы получить допустимые значения для вашего региона, см. раздел [обнаружение доступных периодов обслуживания](#discover-available-maintenance-windows).


   ```powershell-interactive
    # Set variables for your pool
    $resourceGroupName = "your_resource_group_name"
    $serverName = "your_server_name"
    $poolName = "your_pool_name"
    
    # Set selected maintenance window
    $maintenanceConfig = "SQL_EastUS2_DB_2"

    Write-host "Creating a Standard 50 pool with maintenance window ${maintenanceConfig} ..."
    $pool = New-AzSqlElasticPool `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -ElasticPoolName $poolName `
      -Edition "Standard" `
      -Dtu 50 `
      -DatabaseDtuMin 10 `
      -DatabaseDtuMax 20 `
      -MaintenanceConfigurationId $maintenanceConfig
    $pool
   ```

## <a name="set-the-maintenance-window-while-creating-a-managed-instance"></a>Задание периода обслуживания при создании управляемого экземпляра

В следующем примере создается новый управляемый экземпляр и задается окно обслуживания с помощью командлета [New-азсклинстанце](/powershell/module/az.sql/new-azsqlinstance) . Окно обслуживания устанавливается на экземпляре, поэтому все базы данных в экземпляре имеют Расписание окна обслуживания экземпляра. Для `-MaintenanceConfigurationId` параметр *маинтенанцеконфигнаме* должен быть допустимым значением для региона вашего экземпляра. Чтобы получить допустимые значения для вашего региона, см. раздел [обнаружение доступных периодов обслуживания](#discover-available-maintenance-windows).


   ```powershell
   New-AzSqlInstance -Name "your_mi_name" `
     -ResourceGroupName "your_resource_group_name" `
     -Location "your_mi_location" `
     -SubnetId /subscriptions/{SubID}/resourceGroups/{ResourceGroup}/providers/Microsoft.Network/virtualNetworks/{VNETName}/subnets/{SubnetName} `
     -MaintenanceConfigurationId "/subscriptions/{SubID}/providers/Microsoft.Maintenance/publicMaintenanceConfigurations/SQL_{Region}_{MaintenanceConfigName}" `
     -AsJob
   ```

# <a name="cli"></a>[CLI](#tab/azure-cli)

В следующих примерах показано, как настроить период обслуживания с помощью Azure CLI. Можно [установить Azure CLI](/cli/azure/install-azure-cli)или использовать Azure Cloud Shell. 

Настройка периода обслуживания с Azure CLI доступна только для Управляемый экземпляр SQL.

## <a name="launch-azure-cloud-shell"></a>Запуск Azure Cloud Shell

Azure Cloud Shell — это бесплатная интерактивная оболочка, с помощью которой можно выполнять действия, описанные в этой статье. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью. 

Чтобы открыть Cloud Shell, просто выберите **Попробовать** в правом верхнем углу блока кода. Cloud Shell можно также запустить в отдельной вкладке браузера, перейдя на страницу [https://shell.azure.com/cli](https://shell.azure.com/cli). Нажмите кнопку **Копировать**, чтобы скопировать блоки кода. Вставьте код в Cloud Shell и нажмите клавишу "ВВОД", чтобы выполнить его.

## <a name="discover-available-maintenance-windows"></a>Обнаружение доступных периодов обслуживания

При настройке периода обслуживания каждый регион имеет собственные параметры периода обслуживания, соответствующие часовому поясу для региона, в котором находится база данных или пул.

### <a name="discover-sql-database-and-elastic-pool-maintenance-windows"></a>Обнаружение окон базы данных SQL и обслуживания эластичных пулов

В следующем примере возвращаются доступные периоды обслуживания для региона *eastus2* с помощью команды [AZ Maintenance Public-Configuration List](/cli/azure/ext/maintenance/maintenance/public-configuration#ext_maintenance_az_maintenance_public_configuration_list) . Для баз данных и эластичных пулов задайте значение `maintenanceScope` `SQLDB` .

   ```azurecli
   location="eastus2"

   az maintenance public-configuration list --query "[?location=='$location'&&contains(maintenanceScope,'SQLDB')]"
   ```

### <a name="discover-sql-managed-instance-maintenance-windows"></a>Обнаружение окон обслуживания Управляемый экземпляр SQL

В следующем примере возвращаются доступные периоды обслуживания для региона *eastus2* с помощью команды [AZ Maintenance Public-Configuration List](/cli/azure/ext/maintenance/maintenance/public-configuration#ext_maintenance_az_maintenance_public_configuration_list) . Для управляемых экземпляров задайте `maintenanceScope` для значение `SQLManagedInstance` .

   ```azurecli
   az maintenance public-configuration list --query "[?location=='eastus2'&&contains(maintenanceScope,'SQLManagedInstance')]"
   ```

## <a name="set-the-maintenance-window-while-creating-a-single-database"></a>Задание периода обслуживания при создании отдельной базы данных

В следующем примере создается новая база данных и задается окно обслуживания с помощью команды [AZ SQL DB Create](/cli/azure/sql/db#az_sql_db_create) . Параметру `--maint-config-id` (или `-m` ) должно быть присвоено допустимое значение для региона базы данных. Чтобы получить допустимые значения для вашего региона, см. раздел [обнаружение доступных периодов обслуживания](#discover-available-maintenance-windows).


   ```azurecli
    # Set variables for your database
    resourceGroupName="your_resource_group_name"
    serverName="your_server_name"
    databaseName="your_db_name"

    # Set selected maintenance window
    maintenanceConfig="SQL_EastUS2_DB_1"

    # Create database
    az sql db create \
      --resource-group $resourceGroupName \
      --server $serverName \
      --name $databaseName \
      --edition GeneralPurpose \
      --family Gen5 \
      --capacity 2 \
      --maint-config-id $maintenanceConfig
   ```

## <a name="set-the-maintenance-window-while-creating-an-elastic-pool"></a>Настройка периода обслуживания при создании эластичного пула

В следующем примере создается пул эластичных БД и задается окно обслуживания с помощью командлета [AZ SQL эластичного пула Create](/cli/azure/sql/elastic-pool#az_sql_elastic_pool_create) . Период обслуживания задается в эластичном пуле, поэтому все базы данных в пуле имеют расписание периода обслуживания пула. Параметру `--maint-config-id` (или `-m` ) должно быть присвоено допустимое значение для региона пула. Чтобы получить допустимые значения для вашего региона, см. раздел [обнаружение доступных периодов обслуживания](#discover-available-maintenance-windows).


   ```azurecli
    # Set variables for your pool
    resourceGroupName="your_resource_group_name"
    serverName="your_server_name"
    poolName="your_pool_name"

    # Set selected maintenance window
    maintenanceConfig="SQL_EastUS2_DB_2"

    # Create elastic pool
    az sql elastic-pool create \
      --resource-group $resourceGroupName \
      --server $serverName \
      --name $poolName \
      --edition GeneralPurpose \
      --family Gen5 \
      --capacity 2 \
      --maint-config-id $maintenanceConfig
   ```

## <a name="set-the-maintenance-window-while-creating-a-managed-instance"></a>Задание периода обслуживания при создании управляемого экземпляра

В следующем примере создается новый управляемый экземпляр и задается окно обслуживания с помощью команды [AZ SQL MI Create](/cli/azure/sql/mi#az_sql_mi_create). Окно обслуживания устанавливается на экземпляре, поэтому все базы данных в экземпляре имеют Расписание окна обслуживания экземпляра. *Маинтенанцеконфигнаме* должно быть допустимым значением для региона вашего экземпляра. Чтобы получить допустимые значения для вашего региона, см. раздел [обнаружение доступных периодов обслуживания](#discover-available-maintenance-windows).

   ```azurecli
   az sql mi create -g mygroup -n myinstance -l mylocation -i -u myusername -p mypassword --subnet /subscriptions/{SubID}/resourceGroups/{ResourceGroup}/providers/Microsoft.Network/virtualNetworks/{VNETName}/subnets/{SubnetName} -m /subscriptions/{SubID}/providers/Microsoft.Maintenance/publicMaintenanceConfigurations/SQL_{Region}_{MaintenanceConfigName}
   ```

-----

## <a name="configure-maintenance-window-for-existing-databases"></a>Настройка периода обслуживания для существующих баз данных


При применении выбранного периода обслуживания к базе данных в некоторых случаях может возникнуть небольшая перенастройка (несколько секунд), так как Azure применяет необходимые изменения.

# <a name="portal"></a>[Портал](#tab/azure-portal)

Следующие шаги задают период обслуживания в существующей базе данных, эластичном пуле или управляемом экземпляре с помощью портал Azure:


## <a name="set-the-maintenance-window-for-an-existing-database-or-elastic-pool"></a>Задание периода обслуживания для существующей базы данных или эластичного пула

1. Перейдите в базу данных SQL или эластичный пул, для которого необходимо задать период обслуживания.
1. В меню **Параметры** выберите **обслуживание**, а затем выберите нужное окно обслуживания.

   :::image type="content" source="media/maintenance-window-configure/maintenance.png" alt-text="Страница обслуживания базы данных SQL":::


## <a name="set-the-maintenance-window-for-an-existing-managed-instance"></a>Задание периода обслуживания для существующего управляемого экземпляра

1. Перейдите к управляемому экземпляру, для которого необходимо задать период обслуживания.
1. В меню **Параметры** выберите **обслуживание**, а затем выберите нужное окно обслуживания.

   :::image type="content" source="media/maintenance-window-configure/maintenance-mi.png" alt-text="Страница обслуживания управляемого экземпляра SQL":::



# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

## <a name="set-the-maintenance-window-for-an-existing-database"></a>Задание периода обслуживания для существующей базы данных

В следующем примере задается окно обслуживания в существующей базе данных с помощью командлета [Set-азсклдатабасе](/powershell/module/az.sql/set-azsqldatabase) . `-MaintenanceConfigurationId`Должно быть задано допустимое значение для региона базы данных. Чтобы получить допустимые значения для вашего региона, см. раздел [обнаружение доступных периодов обслуживания](#discover-available-maintenance-windows).

   ```powershell-interactive
    # Select different maintenance window
    $maintenanceConfig = "SQL_EastUS2_DB_2"

    Write-host "Changing database maintenance window to ${maintenanceConfig} ..."
    $database = Set-AzSqlDatabase `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -DatabaseName $databaseName `
      -MaintenanceConfigurationId $maintenanceConfig
    $database
   ```

## <a name="set-the-maintenance-window-on-an-existing-elastic-pool"></a>Задание периода обслуживания в существующем пуле эластичных БД

В следующем примере задается период обслуживания в существующем пуле эластичных БД с помощью командлета [Set-азсклеластикпул](/powershell/module/az.sql/set-azsqlelasticpool) . Важно убедиться, что `$maintenanceConfig` значение является допустимым значением для региона пула.  Чтобы получить допустимые значения для региона, см. раздел [обнаружение доступных периодов обслуживания](#discover-available-maintenance-windows).

   ```powershell-interactive
    # Select different maintenance window
    $maintenanceConfig = "SQL_EastUS2_DB_1"
    
    Write-host "Changing pool maintenance window to ${maintenanceConfig} ..."
    $pool = Set-AzSqlElasticPool `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -ElasticPoolName $poolName `
      -MaintenanceConfigurationId $maintenanceConfig
    $pool
   ```



## <a name="set-the-maintenance-window-on-an-existing-managed-instance"></a>Задание периода обслуживания для существующего управляемого экземпляра

В следующем примере задается период обслуживания для существующего управляемого экземпляра с помощью командлета [Set-азсклинстанце](/powershell/module/az.sql/set-azsqlinstance) . Важно убедиться, что `$maintenanceConfig` значение должно быть допустимым значением для региона вашего экземпляра.  Чтобы получить допустимые значения для региона, см. раздел [обнаружение доступных периодов обслуживания](#discover-available-maintenance-windows).


   ```powershell-interactive
   Set-AzSqlInstance -Name "your_mi_name" `
     -ResourceGroupName "your_resource_group_name" `
     -MaintenanceConfigurationId "/subscriptions/{SubID}/providers/Microsoft.Maintenance/publicMaintenanceConfigurations/SQL_{Region}_{MaintenanceConfigName}" `
     -AsJob
   ```

# <a name="cli"></a>[CLI](#tab/azure-cli)

В следующих примерах показано, как настроить период обслуживания с помощью Azure CLI. Можно [установить Azure CLI](/cli/azure/install-azure-cli)или использовать Azure Cloud Shell.

## <a name="set-the-maintenance-window-for-an-existing-database"></a>Задание периода обслуживания для существующей базы данных

В следующем примере задается окно обслуживания в существующей базе данных с помощью команды [AZ SQL DB Update](/cli/azure/sql/db#az_sql_db_update) . Параметру `--maint-config-id` (или `-m` ) должно быть присвоено допустимое значение для региона базы данных. Чтобы получить допустимые значения для вашего региона, см. раздел [обнаружение доступных периодов обслуживания](#discover-available-maintenance-windows).

   ```azurecli
    # Select different maintenance window
    maintenanceConfig="SQL_EastUS2_DB_2"

    # Update database
    az sql db update \
      --resource-group $resourceGroupName \
      --server $serverName \
      --name $databaseName \
      --maint-config-id $maintenanceConfig
   ```

## <a name="set-the-maintenance-window-on-an-existing-elastic-pool"></a>Задание периода обслуживания в существующем пуле эластичных БД

В следующем примере задается период обслуживания в существующем пуле эластичных БД с помощью команды [AZ SQL эластичного пула Update](/cli/azure/sql/elastic-pool#az_sql_elastic_pool_update) . Важно убедиться, что `maintenanceConfig` значение является допустимым значением для региона пула.  Чтобы получить допустимые значения для региона, см. раздел [обнаружение доступных периодов обслуживания](#discover-available-maintenance-windows).

   ```azurecli
    # Select different maintenance window
    maintenanceConfig="SQL_EastUS2_DB_1"

    # Update pool
    az sql elastic-pool update \
      --resource-group $resourceGroupName \
      --server $serverName \
      --name $poolName \
      --maint-config-id $maintenanceConfig
   ```

## <a name="set-the-maintenance-window-on-an-existing-managed-instance"></a>Задание периода обслуживания для существующего управляемого экземпляра

В следующем примере задается окно обслуживания с помощью команды [AZ SQL MI Update](/cli/azure/sql/mi#az_sql_mi_update). Окно обслуживания устанавливается на экземпляре, поэтому все базы данных в экземпляре имеют Расписание окна обслуживания экземпляра. Для `-MaintenanceConfigurationId` параметр *маинтенанцеконфигнаме* должен быть допустимым значением для региона вашего экземпляра. Чтобы получить допустимые значения для вашего региона, см. раздел [обнаружение доступных периодов обслуживания](#discover-available-maintenance-windows).

   ```azurecli
   az sql mi update -g mygroup  -n myinstance -m /subscriptions/{SubID}/providers/Microsoft.Maintenance/publicMaintenanceConfigurations/SQL_{Region}_{MainteanceConfigName}
   ```

-----

## <a name="cleanup-resources"></a>Очистка ресурсов

Не забудьте удалить ненужные ресурсы после завершения работы с ними, чтобы избежать ненужных расходов.

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. Перейдите к базе данных SQL или эластичному пулу, которые больше не нужны.
1. В меню **Обзор** выберите параметр, чтобы удалить ресурс.


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

   ```powershell-interactive
   # Delete database
   Remove-AzSqlDatabase `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -DatabaseName $databaseName

   # Delete elastic pool
   Remove-AzSqlElasticPool `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -ElasticPoolName $poolName
   ```

# <a name="cli"></a>[CLI](#tab/azure-cli)

   ```azurecli
   az sql db delete \
      --resource-group $resourceGroupName \
      --server $serverName \
      --name $databaseName

   az sql elastic-pool delete \
      --resource-group $resourceGroupName \
      --server $serverName \
      --name $poolName
   ```

-----

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о периоде обслуживания см. в разделе [период обслуживания (Предварительная версия)](maintenance-window.md).
- Дополнительные сведения см. в разделе [часто задаваемые вопросы об окне обслуживания](maintenance-window-faq.yml).
- Дополнительные сведения о оптимизации производительности см. [в статье мониторинг и настройка производительности в базе данных SQL Azure и управляемый экземпляр Azure SQL](monitor-tune-overview.md).
