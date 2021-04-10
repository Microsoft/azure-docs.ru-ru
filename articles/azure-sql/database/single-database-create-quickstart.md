---
title: Создание отдельной базы данных
description: Создание отдельной базы данных в службе "База данных SQL Azure" с помощью портала Azure, PowerShell или Azure CLI.
services: sql-database
ms.service: sql-database
ms.subservice: single-database
ms.custom: contperf-fy21q1, devx-track-azurecli
ms.devlang: ''
ms.topic: quickstart
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/27/2021
ms.openlocfilehash: c0e86cad252da1b5484944c12351d4ee1e39edbe
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101656319"
---
# <a name="quickstart-create-an-azure-sql-database-single-database"></a>Краткое руководство. Создание отдельной базы данных в Базе данных SQL Azure

В рамках этого краткого руководства вы создадите [отдельную базу данных](single-database-overview.md) в службе "База данных SQL Azure" с помощью портала Azure, скрипта PowerShell или Azure CLI. Затем вы отправите запросы к этой базе данных через **редактор запросов** на портале Azure.



## <a name="prerequisite"></a>Предварительные требования

- Активная подписка Azure. Если ее нет, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/).

## <a name="create-a-single-database"></a>Создание отдельной базы данных

В этом кратком руководстве описано, как создать отдельную базу данных на [уровне бессерверных вычислений](serverless-tier-overview.md).

# <a name="portal"></a>[Портал](#tab/azure-portal)

Чтобы создать отдельную базу данных на портале Azure с помощью этого краткого руководства, перейдите на страницу SQL Azure.

1. Перейдите на страницу [Выберите вариант развертывания SQL](https://portal.azure.com/#create/Microsoft.AzureSQL).
1. В разделе **Базы данных SQL** оставьте для параметра **Тип ресурса** значение **Отдельная база данных** и нажмите кнопку **Создать**.

   ![Добавление в Azure SQL](./media/single-database-create-quickstart/select-deployment.png)

1. На вкладке **Основные сведения** формы **Создание базы данных SQL** в разделе **Сведения о проекте** выберите подходящую **подписку Azure**.
1. В разделе **Группа ресурсов** выберите **Создать**, введите *myResourceGroup* и нажмите кнопку **OK**.
1. В поле **Имя базы данных** введите *mySampleDatabase*.
1. В группе **Сервер** выберите **Создать** и заполните форму **Новый сервер** следующим образом:
   - **Имя сервера**: Введите *mysqlserver* и добавьте еще символы, чтобы имя было уникальным. Мы не можем указать точное имя сервера, так как имена всех серверов в Azure должны быть глобально уникальными, а не только в рамках подписки. Поэтому введите, например, mysqlserver12345 и узнайте на портале доступно это имя или нет.
   - **Имя для входа администратора сервера**. Введите *azureuser*.
   - **Пароль**. Введите пароль, соответствующий требованиям, и введите его еще раз в поле **Подтверждение пароля**.
   - **Расположение.** Выберите расположение из раскрывающегося списка.

   Щелкните **ОК**.

1. Оставьте для параметра **Хотите использовать Эластичный пул SQL?** значение **Нет**.
1. В разделе **Вычисления и хранилище** выберите **Настроить базу данных**.
1. В этом кратком руководстве используется бессерверная база данных, поэтому выберите **Бессерверное**, а затем нажмите кнопку **Применить**. 

      ![Настройка бессерверной базы данных](./media/single-database-create-quickstart/configure-database.png)

1. По завершении выберите **Next: Networking** (Далее: сеть) в нижней части экрана.

   ![Новая база данных SQL — вкладка "Основные сведения"](./media/single-database-create-quickstart/new-sql-database-basics.png)

1. На вкладке **Сеть** в разделе **Метод подключения** выберите **Общедоступная конечная точка**.
1. В разделе **Правила брандмауэра** установите переключатель **Добавить текущий IP-адрес клиента** в положение **Да**. Оставьте значение **Нет** для параметра **Разрешить доступ к серверу службам и ресурсам Azure**.
1. По завершении выберите **Next: Additional settings** (Далее: дополнительные параметры) в нижней части страницы.

   ![Вкладка "Сеть"](./media/single-database-create-quickstart/networking.png)
  

1. На вкладке **Дополнительные параметры** в разделе **Источник данных** для параметра **Использовать существующие данные** выберите значение **Пример**. При этом создается образец базы данных AdventureWorksLT, где можно выполнять запросы к некоторым таблицам и данным и экспериментировать с ними в отличие от пустой базы данных.
1. При необходимости включите [Azure Defender для SQL](../database/azure-defender-for-sql.md).
1. Вы также можете задать [период обслуживания](../database/maintenance-window.md), чтобы плановое обслуживание базы данных выполнялось в оптимальное время.
1. В нижней части страницы нажмите кнопку **Просмотреть и создать**:

   ![Вкладка "Дополнительные параметры"](./media/single-database-create-quickstart/additional-settings.png)

1. На странице **Просмотр и создание** после проверки нажмите кнопку **Создать**.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

## <a name="launch-azure-cloud-shell"></a>Запуск Azure Cloud Shell

Azure Cloud Shell — это бесплатная интерактивная оболочка, с помощью которой можно выполнять действия, описанные в этой статье. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью. 

Чтобы открыть Cloud Shell, просто выберите **Попробовать** в правом верхнем углу блока кода. Cloud Shell можно также запустить в отдельной вкладке браузера, перейдя на страницу [https://shell.azure.com](https://shell.azure.com). Нажмите кнопку **Копировать**, чтобы скопировать блоки кода. Вставьте код в Cloud Shell и нажмите клавишу **ВВОД**, чтобы выполнить его.

## <a name="set-parameter-values"></a>Установка значений параметра

Следующие значения используются в последующих командах для создания базы данных и требуемых ресурсов. Имена серверов должны быть глобально уникальными в Azure, поэтому для создания имени сервера используется функция $RANDOM. Замените значения 0.0.0.0 в диапазоне IP-адресов в соответствии с конкретной средой.

```azurecli-interactive
# Set the resource group name and location for your server
resourceGroupName=myResourceGroup
location=eastus

# Set an admin login and password for your database
adminlogin=azureuser
password=Azure1234567!

# Set a server name that is unique to Azure DNS (<server_name>.database.windows.net)
serverName=server-$RANDOM

# Set the ip address range that can access your database
startip=0.0.0.0
endip=0.0.0.0
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli-interactive
az group create --name $resourceGroupName --location $location
```

## <a name="create-a-server"></a>Создание сервера

Создайте сервер с помощью команды [az sql server create](/cli/azure/sql/server).

```azurecli-interactive
az sql server create \
    --name $serverName \
    --resource-group $resourceGroupName \
    --location $location  \
    --admin-user $adminlogin \
    --admin-password $password
```


## <a name="configure-a-firewall-rule-for-the-server"></a>Настройка правила брандмауэра для сервера

Создайте правило брандмауэра с помощью команды [az sql server firewall-rule create](/cli/azure/sql/server/firewall-rule).

```azurecli-interactive
az sql server firewall-rule create \
    --resource-group $resourceGroupName \
    --server $serverName \
    -n AllowYourIp \
    --start-ip-address $startip \
    --end-ip-address $endip
```


## <a name="create-a-single-database-with-azure-cli"></a>Создание отдельной базы данных с помощью Azure CLI

Создайте базу данных с помощью команды [az sql db create](/cli/azure/sql/db). Следующий фрагмент кода создает базу данных:


```azurecli-interactive
az sql db create \
    --resource-group $resourceGroupName \
    --server $serverName \
    --name mySampleDatabase \
    --sample-name AdventureWorksLT \
    --edition GeneralPurpose \
    --compute-model Serverless \
    --family Gen5 \
    --capacity 2
```


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Создать группу ресурсов, сервер и отдельную базу данных можно с помощью Windows PowerShell.

## <a name="launch-azure-cloud-shell"></a>Запуск Azure Cloud Shell

Azure Cloud Shell — это бесплатная интерактивная оболочка, с помощью которой можно выполнять действия, описанные в этой статье. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью. 

Чтобы открыть Cloud Shell, просто выберите **Попробовать** в правом верхнем углу блока кода. Cloud Shell можно также запустить в отдельной вкладке браузера, перейдя на страницу [https://shell.azure.com](https://shell.azure.com). Нажмите кнопку **Копировать**, чтобы скопировать блоки кода. Вставьте код в Cloud Shell и нажмите клавишу **ВВОД**, чтобы выполнить его.

## <a name="set-parameter-values"></a>Установка значений параметра

Следующие значения используются в последующих командах для создания базы данных и требуемых ресурсов. Имена серверов должны быть глобально уникальными в Azure, поэтому для создания имени сервера используется командлет Get-Random. Замените значения 0.0.0.0 в диапазоне IP-адресов в соответствии с конкретной средой.

```azurepowershell-interactive
   # Set variables for your server and database
   $resourceGroupName = "myResourceGroup"
   $location = "eastus"
   $adminLogin = "azureuser"
   $password = "Azure1234567!"
   $serverName = "mysqlserver-$(Get-Random)"
   $databaseName = "mySampleDatabase"

   # The ip address range that you want to allow to access your server
   $startIp = "0.0.0.0"
   $endIp = "0.0.0.0"

   # Show randomized variables
   Write-host "Resource group name is" $resourceGroupName
   Write-host "Server name is" $serverName
```


## <a name="create-resource-group"></a>Создать группу ресурсов

Создайте группу ресурсов Azure с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими.

```azurepowershell-interactive
   Write-host "Creating resource group..."
   $resourceGroup = New-AzResourceGroup -Name $resourceGroupName -Location $location -Tag @{Owner="SQLDB-Samples"}
   $resourceGroup
```


## <a name="create-a-server"></a>Создание сервера

Создайте сервер с помощью командлета [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver).

```azurepowershell-interactive
  Write-host "Creating primary server..."
   $server = New-AzSqlServer -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -Location $location `
      -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential `
      -ArgumentList $adminLogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))
   $server
```

## <a name="create-a-firewall-rule"></a>Создание правила брандмауэра

Создайте правило брандмауэра сервера с помощью командлета [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule).

```azurepowershell-interactive
   Write-host "Configuring server firewall rule..."
   $serverFirewallRule = New-AzSqlServerFirewallRule -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -FirewallRuleName "AllowedIPs" -StartIpAddress $startIp -EndIpAddress $endIp
   $serverFirewallRule
```


## <a name="create-a-single-database-with-powershell"></a>Создание отдельной базы данных с помощью PowerShell

Создайте отдельную базу данных с помощью командлета [New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase).

```azurepowershell-interactive
   Write-host "Creating a gen5 2 vCore serverless database..."
   $database = New-AzSqlDatabase  -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -DatabaseName $databaseName `
      -Edition GeneralPurpose `
      -ComputeModel Serverless `
      -ComputeGeneration Gen5 `
      -VCore 2 `
      -MinimumCapacity 2 `
      -SampleName "AdventureWorksLT"
   $database
```

---



## <a name="query-the-database"></a>Выполнение запросов к базе данных

Завершив создание базы данных, вы сможете через **редактор запросов (предварительная версия)** на портале Azure подключиться к этой базе данных и выполнить запрос данных.

1. На портале найдите и выберите элемент **Базы данных SQL**, а затем в открывшемся списке выберите нужную базу данных.
1. На странице для этой базы данных найдите и выберите элемент **Редактор запросов (предварительная версия)** в меню слева.
1. Введите учетные данные администратора сервера и щелкните **ОК**.

   ![Вход в редактор запросов](./media/single-database-create-quickstart/query-editor-login.png)

1. Введите следующий запрос в области **Редактор запросов**.

   ```sql
   SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName
   FROM SalesLT.ProductCategory pc
   JOIN SalesLT.Product p
   ON pc.productcategoryid = p.productcategoryid;
   ```

1. Щелкните **Выполнить** и просмотрите результаты запроса в области **Результаты**.

   ![Результаты в редакторе запросов](./media/single-database-create-quickstart/query-editor-results.png)

1. Закройте страницу **редактора запросов** и щелкните **ОК**, когда увидите запрос на отмену несохраненных изменений.

## <a name="clean-up-resources"></a>Очистка ресурсов

Сохраните группу ресурсов, сервер и отдельную базу данных, если вы планируете перейти к следующим шагам и изучить разные методы подключения к базе данных и отправки запросов к ней.

Когда вы закончите работу с этими ресурсами, созданную группу ресурсов можно удалить вместе с размещенными в ней сервером и отдельной базой данных.

### <a name="portal"></a>[Портал](#tab/azure-portal)

Чтобы удалить **myResourceGroup** и все ее ресурсы с помощью портала Azure, сделайте следующее:

1. На портале Azure найдите и выберите элемент **Группы ресурсов**, а затем выберите в списке элемент **myResourceGroup**.
1. На странице группы ресурсов выберите **Удалить группу ресурсов**.
1. В поле **Введите имя группы ресурсов** введите *myResourceGroup*, а затем щелкните **Удалить**.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы удалить группу ресурсов и все ресурсы в ней, выполните следующую команду Azure CLI с именем удаляемой группы ресурсов.

```azurecli-interactive
az group delete --name $resourceGroupName
```

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы удалить группу ресурсов и все ресурсы в ней, выполните следующий командлет PowerShell с именем удаляемой группы ресурсов.

```azurepowershell-interactive
Remove-AzResourceGroup -Name $resourceGroupName
```

---

## <a name="next-steps"></a>Дальнейшие действия

[Создайте подключение и выполните запрос](connect-query-content-reference-guide.md) к базе данных с помощью разных средств и языков.
> [!div class="nextstepaction"]
> [подключиться и создать запрос с помощью SQL Server Management Studio](connect-query-ssms.md);
>
> [подключиться и создать запрос с помощью Azure Data Studio](/sql/azure-data-studio/quickstart-sql-database?toc=/azure/sql-database/toc.json).

Хотите оптимизировать и сократить ваши расходы на облако?

> [!div class="nextstepaction"]
> [Начните анализировать затраты с помощью службы "Управление затратами"](../../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)