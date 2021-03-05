---
title: Руководство по Добавление базы данных в группу отработки отказа
description: Добавление базы данных в службе "База данных SQL Azure" в группу автоматической отработки отказа с помощью портала Azure, PowerShell или Azure CLI.
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.custom: sqldbrb=1, devx-track-azurecli
ms.devlang: ''
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 06/19/2019
ms.openlocfilehash: 68fa089713c3dd89b4699011ded7d667bca6f73f
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102178091"
---
# <a name="tutorial-add-an-azure-sql-database-to-an-autofailover-group"></a>Руководство по добавлению Базы данных SQL Azure в группу автоматической отработки отказа
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

[Группа отработки отказа](auto-failover-group-overview.md) — это декларативный слой абстракции, в котором группируются несколько геореплицированных баз данных. Узнайте, как настроить группу отработки отказа для Базы данных SQL Azure и протестировать отработку отказа с помощью портала Azure, PowerShell или Azure CLI.  В этом руководстве вы узнаете, как:

> [!div class="checklist"]
>
> - Создание базы данных в службе "База данных SQL Azure"
> - создание для базы данных группы отработки отказа между двумя серверами;
> - выполнить тестовую отработку отказа.

## <a name="prerequisites"></a>Предварительные требования

# <a name="the-portal"></a>[Портал](#tab/azure-portal)

В рамках этого руководства вам потребуются:

- Подписка Azure. [Создайте бесплатную учетную запись](https://azure.microsoft.com/free/), если у вас ее еще нет.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Для работы с этим учебником требуется следующее:

- Подписка Azure. [Создайте бесплатную учетную запись](https://azure.microsoft.com/free/), если у вас ее еще нет.
- [Azure PowerShell](/powershell/azure/)

# <a name="the-azure-cli"></a>[CLI Azure.](#tab/azure-cli)

Для работы с этим руководством требуется следующее:

- Подписка Azure. [Создайте бесплатную учетную запись](https://azure.microsoft.com/free/), если у вас ее еще нет.
- Последняя версия [Azure CLI](/cli/azure/install-azure-cli).

---

## <a name="1---create-a-database"></a>1\. Создание базы данных.

[!INCLUDE [sql-database-create-single-database](../includes/sql-database-create-single-database.md)]

## <a name="2---create-the-failover-group"></a>2\. Создание группы отработки отказа.

На этом этапе вы создадите [группу отработки отказа](auto-failover-group-overview.md) между существующим сервером и новым сервером в другом регионе. Затем добавьте пример базы данных в группу отработки отказа.

# <a name="the-portal"></a>[Портал](#tab/azure-portal)

Создайте группу отработки отказа и добавьте в нее базу данных с помощью портала Azure.

1. На [портале Azure](https://portal.azure.com) в меню слева выберите **Azure SQL**. Если **Azure SQL** нет в списке, выберите элемент **Все службы** и введите "Azure SQL" в поле поиска. (Необязательно) Щелкните звезду рядом с **Azure SQL**, чтобы добавить этот элемент в избранное и область навигации слева.
1. Выберите базу данных, созданную в разделе 1, например `mySampleDatabase`.
1. Группы отработки отказа можно настраивать на уровне сервера. Выберите имя сервера в разделе **Имя сервера**, чтобы открыть параметры для этого сервера.

   ![Открытие сервера для базы данных](./media/failover-group-add-single-database-tutorial/open-sql-db-server.png)

1. Выберите элемент **Группы отработки отказа** в области **Параметры**, а затем щелкните элемент **Добавить группу**, чтобы создать новую группу отработки отказа.

   ![Создание группы отработки отказа.](./media/failover-group-add-single-database-tutorial/sqldb-add-new-failover-group.png)

1. На странице **Группа отработки отказа** введите или выберите следующие значения, а затем щелкните **Создать**:

   - **Имя группы отработки отказа.** Введите уникальное имя для группы отработки отказа, например `failovergrouptutorial`.
   - **Сервер-получатель.** Выберите вариант *настройки обязательных параметров*, а затем щелкните элемент **Создать сервер**. Кроме того, вы можете выбрать в качестве сервера-получателя уже существующий сервер. После ввода указанных ниже значений щелкните элемент **Выбрать**.
      - **Имя сервера**: Введите уникальное имя для сервера-получателя, например `mysqlsecondary`.
      - **Имя для входа администратора сервера**. Введите `azureuser`.
      - **Пароль**. Введите сложный пароль, который соответствует требованиям к паролю.
      - **Расположение.** Выберите расположение из раскрывающегося списка, например `East US`. Это расположение не может совпадать с расположением основного сервера.

     > [!NOTE]
     > Имя для входа и параметры брандмауэра на этом сервере должны совпадать с параметрами сервера-источника.

     ![Создание сервера-получателя для группы отработки отказа](./media/failover-group-add-single-database-tutorial/create-secondary-failover-server.png)

   - **База данных в группе.** Этот параметр будет разблокирован после выбора сервера-получателя. Щелкните его, чтобы **выбрать базы данных для добавления**, а затем выберите созданную в разделе 1 базу данных. Добавление базы данных в группу отработки отказа автоматически активирует георепликацию.

   ![Добавление Базы данных SQL в группу отработки отказа](./media/failover-group-add-single-database-tutorial/add-sqldb-to-failover-group.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Создайте группу отработки отказа и добавьте в нее базу данных с помощью PowerShell.

   > [!NOTE]
   > Имя для входа и параметры брандмауэра на этом сервере должны совпадать с параметрами сервера-источника.

   ```powershell-interactive
   # $subscriptionId = '<SubscriptionID>'
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"
   # $location = "West US"
   # $adminLogin = "azureuser"
   # $password = "PWD27!"+(New-Guid).Guid
   # $serverName = "mysqlserver-$(Get-Random)"
   # $databaseName = "mySampleDatabase"
   $drLocation = "East US"
   $drServerName = "mysqlsecondary-$(Get-Random)"
   $failoverGroupName = "failovergrouptutorial-$(Get-Random)"

   # The ip address range that you want to allow to access your server
   # (leaving at 0.0.0.0 will prevent outside-of-azure connections to your DB)
   $startIp = "0.0.0.0"
   $endIp = "0.0.0.0"

   # Show randomized variables
   Write-host "DR Server name is" $drServerName
   Write-host "Failover group name is" $failoverGroupName

   # Create a secondary server in the failover region
   Write-host "Creating a secondary server in the failover region..."
   $drServer = New-AzSqlServer -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName `
      -Location $drLocation `
      -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential `
         -ArgumentList $adminlogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))
   $drServer

   # Create a server firewall rule that allows access from the specified IP range
   Write-host "Configuring firewall for secondary server..."
   $serverFirewallRule = New-AzSqlServerFirewallRule -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName `
      -FirewallRuleName "AllowedIPs" -StartIpAddress $startIp -EndIpAddress $endIp
   $serverFirewallRule

   # Create a failover group between the servers
   $failovergroup = Write-host "Creating a failover group between the primary and secondary server..."
   New-AzSqlDatabaseFailoverGroup `
      –ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -PartnerServerName $drServerName  `
      –FailoverGroupName $failoverGroupName `
      –FailoverPolicy Automatic `
      -GracePeriodWithDataLossHours 2
   $failovergroup

   # Add the database to the failover group
   Write-host "Adding the database to the failover group..."
   Get-AzSqlDatabase `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -DatabaseName $databaseName | `
   Add-AzSqlDatabaseToFailoverGroup `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -FailoverGroupName $failoverGroupName
   Write-host "Successfully added the database to the failover group..."
   ```

В этом разделе руководства используются следующие командлеты PowerShell:

| Get-Help | Примечания |
|---|---|
| [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver) | Отвечает за создание сервера в Базе данных SQL Azure, на котором размещены отдельные базы данных и эластичные пулы. |
| [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule) | Отвечает за создание правила брандмауэра для сервера в Базе данных SQL. |
| [New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase) | Отвечает за создание отдельной базы данных в Базе данных SQL Azure. |
| [New-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/new-azsqldatabasefailovergroup) | Отвечает за создание группы отработки отказа в Базе данных SQL Azure. |
| [Get-AzSqlDatabase](/powershell/module/az.sql/get-azsqldatabase) | Отвечает за получение одной или нескольких баз данных из Базы данных SQL Azure. |
| [Add-AzSqlDatabaseToFailoverGroup](/powershell/module/az.sql/add-azsqldatabasetofailovergroup) | Отвечает за добавление одной или нескольких баз данных в группу отработки отказа в Базе данных SQL Azure. |

# <a name="the-azure-cli"></a>[CLI Azure.](#tab/azure-cli)

Создайте группу отработки отказа и добавьте в нее базу данных с помощью Azure CLI.

   > [!NOTE]
   > Имя для входа и параметры брандмауэра на этом сервере должны совпадать с параметрами сервера-источника.

   ```azurecli-interactive
   #!/bin/bash
   # set variables
   $failoverLocation = "West US"
   $failoverServer = "failoverServer-$randomIdentifier"
   $failoverGroup = "failoverGroup-$randomIdentifier"

   echo "Creating a secondary server in the DR region..."
   az sql server create --name $failoverServer --resource-group $resourceGroup --location $failoverLocation --admin-user $login --admin-password $password

   echo "Creating a failover group between the two servers..."
   az sql failover-group create --name $failoverGroup --partner-server $failoverServer --resource-group $resourceGroup --server $server --add-db $database --failover-policy Automatic
   ```

В этом разделе руководства используются следующие командлеты Azure CLI:

| Get-Help | Примечания |
|---|---|
| [az sql server create](/cli/azure/sql/server#az-sql-server-create) | Создает сервер, на котором размещены базы данных и эластичные пулы. |
| [az sql server firewall-rule create](/cli/azure/sql/server/firewall-rule) | Создает правила брандмауэра сервера. |
| [az sql failover-group create](/cli/azure/sql/failover-group#az-sql-failover-group-create) | Создает группу отработки отказа. |

---

## <a name="3---test-failover"></a>3\. Тестовая отработка отказа

На этом этапе вы переключите группу отработки отказа на сервер-получатель, а затем восстановите размещение с помощью портала Azure.

# <a name="the-portal"></a>[Портал](#tab/azure-portal)

Выполните тестовую отработку отказа с помощью портала Azure

1. На [портале Azure](https://portal.azure.com) в меню слева выберите **Azure SQL**. Если **Azure SQL** нет в списке, выберите элемент **Все службы** и введите "Azure SQL" в поле поиска. (Необязательно) Щелкните звезду рядом с **Azure SQL**, чтобы добавить этот элемент в избранное и область навигации слева.
1. Выберите базу данных, созданную в разделе 2, например `mySampleDatbase`.
1. Выберите имя сервера в разделе **Имя сервера**, чтобы открыть параметры для этого сервера.

   ![Открытие сервера для базы данных](./media/failover-group-add-single-database-tutorial/open-sql-db-server.png)

1. Выберите элемент **Группы отработки отказа** в области **Параметры**, а затем выберите группу отработки отказа, созданную в разделе 2.
  
   ![Выбор группы отработки отказа на портале](./media/failover-group-add-single-database-tutorial/select-failover-group.png)

1. Проверьте, какой сервер является основным, а какой — сервером-получателем.
1. Щелкните элемент **Отработка отказа** в области задач, чтобы выполнить отработку отказа для группы отработки отказа с нашей тестовой базой данных.
1. Выберите ответ **Да** в предупреждении о том, что сеансы TDS будут отключены.

   ![Отработка отказа для группы отработки отказа, которая содержит базу данных](./media/failover-group-add-single-database-tutorial/failover-sql-db.png)

1. Проверьте, какой сервер теперь является основным, а какой — сервером-получателем. Если отработка отказа прошла успешно, роли этих двух серверов должны поменяться между собой.
1. Снова щелкните элемент **Отработка отказа**, чтобы вернуть серверам исходные роли.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Выполните тестовую отработку отказа с помощью PowerShell.

Проверьте роль вторичной реплики:

   ```powershell-interactive
   # Set variables
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"
   # $serverName = "mysqlserver-$(Get-Random)"
   # $failoverGroupName = "failovergrouptutorial-$(Get-Random)"

   # Check role of secondary replica
   Write-host "Confirming the secondary replica is secondary...."
   (Get-AzSqlDatabaseFailoverGroup `
      -FailoverGroupName $failoverGroupName `
      -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName).ReplicationRole
   ```

Выполните отработку отказа на сервер-получатель:

   ```powershell-interactive
   # Set variables
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"
   # $serverName = "mysqlserver-$(Get-Random)"
   # $failoverGroupName = "failovergrouptutorial-$(Get-Random)"

   # Failover to secondary server
   Write-host "Failing over failover group to the secondary..."
   Switch-AzSqlDatabaseFailoverGroup `
      -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName `
      -FailoverGroupName $failoverGroupName
   Write-host "Failed failover group successfully to" $drServerName
   ```

Восстановите группу отработки отказа на основной сервер:

   ```powershell-interactive
   # Set variables
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"
   # $serverName = "mysqlserver-$(Get-Random)"
   # $failoverGroupName = "failovergrouptutorial-$(Get-Random)"

   # Revert failover to primary server
   Write-host "Failing over failover group to the primary...."
   Switch-AzSqlDatabaseFailoverGroup `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -FailoverGroupName $failoverGroupName
   Write-host "Failed failover group successfully back to" $serverName
   ```

В этом разделе руководства используются следующие командлеты PowerShell:

| Get-Help | Примечания |
|---|---|
| [Get-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/get-azsqldatabasefailovergroup) | Возвращает или перечисляет группы отработки отказа для Базы данных SQL Azure. |
| [Switch-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/switch-azsqldatabasefailovergroup)| Выполняет отработку отказа для группы отработки отказа Базы данных SQL Azure. |

# <a name="the-azure-cli"></a>[CLI Azure.](#tab/azure-cli)

Выполните тестовую отработку отказа с помощью Azure CLI.

Определите, какой сервер является сервером-получателем:

   ```azurecli-interactive
   echo "Verifying which server is in the secondary role..."
   az sql failover-group list --server $server --resource-group $resourceGroup
   ```

Выполните отработку отказа на сервер-получатель:

   ```azurecli-interactive
   echo "Failing over group to the secondary server..."
   az sql failover-group set-primary --name $failoverGroup --resource-group $resourceGroup --server $failoverServer
   echo "Successfully failed failover group over to" $failoverServer
   ```

Восстановите группу отработки отказа на основной сервер:

   ```azurecli-interactive
   echo "Failing over group back to the primary server..."
   az sql failover-group set-primary --name $failoverGroup --resource-group $resourceGroup --server $server
   echo "Successfully failed failover group back to" $server
   ```

В этом разделе руководства используются следующие командлеты Azure CLI:

| Get-Help | Примечания |
|---|---|
| [az sql failover-group list](/cli/azure/sql/failover-group#az-sql-failover-group-list) | Перечисляет группы отработки отказа на сервере. |
| [az sql failover-group set-primary](/cli/azure/sql/failover-group#az-sql-failover-group-set-primary) | Задает первичную точку группы отработки отказа. Для этого выполняется отработка отказа всех баз данных с текущего сервера-источника. |

---

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите ресурсы, удалив группу ресурсов.

# <a name="the-portal"></a>[Портал](#tab/azure-portal)

Удалите группу ресурсов с помощью портала Azure.

1. На [портале Azure](https://portal.azure.com) перейдите к используемой группе ресурсов.
1. Щелкните элемент **Удалить группу ресурсов**, чтобы удалить все ресурсы в этой группе ресурсов, а также саму группу.
1. Введите имя группы ресурсов (`myResourceGroup`) в соответствующем текстовом поле, а затем щелкните элемент **Удалить**, чтобы удалить группу ресурсов.  

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Удалите группу ресурсов с помощью PowerShell.

   ```powershell-interactive
   # Set variables
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"

   # Remove the resource group
   Write-host "Removing resource group..."
   Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
   Write-host "Resource group removed =" $resourceGroupName
   ```

В этом разделе руководства используются следующие командлеты PowerShell:

| Get-Help | Примечания |
|---|---|
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов. |

# <a name="the-azure-cli"></a>[CLI Azure.](#tab/azure-cli)

Удалите группу ресурсов с помощью Azure CLI.

   ```azurecli-interactive
   echo "Cleaning up resources by removing the resource group..."
   az group delete --name $resourceGroup
   echo "Successfully removed resource group" $resourceGroup
   ```

В этом разделе руководства используются следующие командлеты Azure CLI:

| Get-Help | Примечания |
|---|---|
| [az group delete](/cli/azure/vm/extension#az-vm-extension-set) | Удаляет группу ресурсов со всеми вложенными ресурсами. |

---

> [!IMPORTANT]
> Если вы намерены сохранить эту группу ресурсов, но база данных-получатель вам больше не нужна, удалите ее, сначала выполнив удаление из группы отработки отказа. Удаление базы данных-получателя без предварительного удаления ее из группы отработки отказа может привести к непредсказуемым последствиям.

## <a name="full-scripts"></a>Полный текст скриптов

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

[!code-powershell-interactive[main](../../../powershell_scripts/sql-database/failover-groups/add-single-db-to-failover-group-az-ps.ps1 "Add database to a failover group")]

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver) | Отвечает за создание в Базе данных SQL Azure сервера, на котором размещены отдельные базы данных и эластичные пулы. |
| [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule) | Отвечает за создание правила брандмауэра для сервера в Базе данных SQL. |
| [New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase) | Отвечает за создание базы данных в Базе данных SQL Azure. |
| [New-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/new-azsqldatabasefailovergroup) | Отвечает за создание группы отработки отказа в Базе данных SQL Azure. |
| [Get-AzSqlDatabase](/powershell/module/az.sql/get-azsqldatabase) | Отвечает за получение одной или нескольких баз данных из Базы данных SQL Azure. |
| [Add-AzSqlDatabaseToFailoverGroup](/powershell/module/az.sql/add-azsqldatabasetofailovergroup) | Отвечает за добавление одной или нескольких баз данных в группу отработки отказа в Базе данных SQL Azure. |
| [Get-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/get-azsqldatabasefailovergroup) | Отвечает за возврат или перечисление групп отработки отказа для Базы данных SQL Azure. |
| [Switch-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/switch-azsqldatabasefailovergroup)| Отвечает за отработку отказа для группы отработки отказа Базы данных SQL Azure. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Отвечает за удаление группы ресурсов из Базы данных SQL.|

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

[!code-azurecli-interactive[main](../../../cli_scripts/sql-database/failover-groups/add-single-db-to-failover-group-az-cli.sh "Add database to a failover group")]

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az account set](/cli/azure/account#az-account-set) | Позволяет указать подписку в качестве текущей активной подписки. |
| [az group create](/cli/azure/group#az-group-create) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az sql server create](/cli/azure/sql/server#az-sql-server-create) | Отвечает за создание в Базе данных SQL Azure сервера, на котором размещены отдельные базы данных и эластичные пулы. |
| [az sql server firewall-rule create](/cli/azure/sql/server/firewall-rule) | Отвечает за создание правил брандмауэра для IP-адресов на уровне сервера в Базе данных SQL. |
| [az sql db create](/cli/azure/sql/db) | Отвечает за создание базы данных в Базе данных SQL Azure. |
| [az sql failover-group create](/cli/azure/sql/failover-group#az-sql-failover-group-create) | Отвечает за создание группы отработки отказа в Базе данных SQL Azure. |
| [az sql failover-group list](/cli/azure/sql/failover-group#az-sql-failover-group-list) | Отвечает за перечисление групп отработки отказа на сервере в Базе данных SQL. |
| [az sql failover-group set-primary](/cli/azure/sql/failover-group#az-sql-failover-group-set-primary) | Задает первичную точку группы отработки отказа. Для этого выполняется отработка отказа всех баз данных с текущего сервера-источника. |
| [az group delete](/cli/azure/vm/extension#az-vm-extension-set) | Удаляет группу ресурсов со всеми вложенными ресурсами. |

# <a name="the-portal"></a>[Портал](#tab/azure-portal)

Скрипты для портала Azure не предоставляются.

---

Вы можете получить скрипты Базы данных SQL Azure для [Azure PowerShell](powershell-script-content-guide.md) и [Azure CLI](az-cli-script-samples-content-guide.md).

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого руководства вы добавили базу данных, размещенную в службе "База данных SQL Azure", в группу отработки отказа и проверили отработку отказа. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
>
> - Создание базы данных в службе "База данных SQL Azure"
> - создание для базы данных группы отработки отказа между двумя серверами;
> - тестовая отработка отказа.

Перейдите к следующему руководству, которое посвящено добавлению эластичного пула в группу отработки отказа.

> [!div class="nextstepaction"]
> [Руководство. Добавление эластичного пула службы "База данных SQL Azure" в группу отработки отказа](failover-group-add-elastic-pool-tutorial.md)