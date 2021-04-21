---
title: Руководство по Добавление эластичного пула в группу отработки отказа
description: Добавьте эластичный пул Базы данных SQL Azure в группу отработки отказа с помощью портала Azure, PowerShell или Azure CLI.
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.custom: seo-lt-2019 sqldbrb=1, devx-track-azurecli
ms.devlang: ''
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 08/27/2019
ms.openlocfilehash: dd0532469cd4390d9a72900dcebc22994239325b
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107479174"
---
# <a name="tutorial-add-an-azure-sql-database-elastic-pool-to-a-failover-group"></a>Руководство по Добавление эластичного пула службы "База данных SQL Azure" в группу отработки отказа
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Настройте группу отработки отказа для эластичного пула Базы данных SQL Azure и протестируйте отработку отказа с помощью портала Azure.  В этом учебнике рассматривается следующее.

> [!div class="checklist"]
>
> - Создание отдельной базы данных.
> - Добавление базы данных в эластичный пул.
> - Создание [группы отработки отказа](auto-failover-group-overview.md) для двух эластичных пулов между двумя серверами.
> - Тестовая отработка отказа.

## <a name="prerequisites"></a>Предварительные требования

В рамках этого руководства вам потребуются:

- Подписка Azure. [Создайте бесплатную учетную запись](https://azure.microsoft.com/free/), если у вас ее еще нет.

## <a name="1---create-a-single-database"></a>1\. Создание отдельной базы данных

[!INCLUDE [sql-database-create-single-database](../includes/sql-database-create-single-database.md)]

## <a name="2---add-the-database-to-an-elastic-pool"></a>2\. Добавление базы данных в эластичный пул

На этом шаге вы создадите эластичный пул и добавите в него базу данных.

# <a name="portal"></a>[Портал](#tab/azure-portal)

Создайте эластичный пул с помощью портала Azure.

1. На портале Azure в меню слева выберите **Azure SQL**. Если **SQL Azure** отсутствует в списке, выберите **Все службы** и в поле поиска введите "SQL Azure". (Необязательно) Щелкните звезду рядом с **Azure SQL**, чтобы добавить этот элемент в избранное и область навигации слева.
1. Щелкните **+ Добавить**, чтобы открыть страницу **Выбор варианта развертывания SQL**. Чтобы просмотреть дополнительные сведения о различных базах данных, выберите Показать сведения на плитке Базы данных.
1. Выберите **Эластичный пул** из раскрывающегося списка **Тип ресурса** на плитке **Базы данных SQL**. Щелкните **Создать**, чтобы создать эластичный пул.

    ![Выбор эластичного пула](./media/failover-group-add-elastic-pool-tutorial/select-azure-sql-elastic-pool.png)

1. Настройте для эластичного пула следующие значения:
   - **Имя**: Укажите для эластичного пула уникальное имя, например `myElasticPool`.
   - **Подписка**: Выберите подписку в раскрывающемся списке.
   - **Группа ресурсов**: Выберите в раскрывающемся списке `myResourceGroup`, то есть созданную в разделе 1 группу ресурсов.
   - **Сервер**: В раскрывающемся списке выберите сервер, созданный в разделе 1.  

       ![Создание нового сервера для эластичного пула](./media/failover-group-add-elastic-pool-tutorial/use-existing-server-for-elastic-pool.png)

   - **Вычисления и хранилище**. Выберите **Настроить эластичный пул**, чтобы указать параметры вычислительной среды и хранилища, а также добавить в эластичный пул отдельную базу данных. На вкладке **Параметры пула** оставьте выбранный по умолчанию вариант 5-го поколения с двумя виртуальными ядрами и 32 ГБ памяти.

1. На странице **Настройка** выберите вкладку **Базы данных**, а затем щелкните **Добавить базу данных**. Выберите базу данных, созданную в разделе 1, а затем щелкните **Применить**, чтобы добавить ее в эластичный пул. Еще раз щелкните **Применить**, чтобы применить параметры эластичного пула, и закройте страницу **Настройка**.

    ![Добавление базы данных в эластичный пул](./media/failover-group-add-elastic-pool-tutorial/add-database-to-elastic-pool.png)

1. Выберите **Проверка и создание**, чтобы просмотреть параметры эластичного пула, а затем щелкните **Создать**, чтобы создать эластичный пул.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Создайте эластичные пулы и сервер-получатель с помощью PowerShell.

   ```powershell-interactive
   # Set variables for your server and database
   # $subscriptionId = '<SubscriptionID>'
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"
   # $location = "East US"
   # $adminLogin = "azureuser"
   # $password = "PWD27!"+(New-Guid).Guid
   # $serverName = "mysqlserver-$(Get-Random)"
   $poolName = "myElasticPool"
   $databaseName = "mySampleDatabase"
   $drLocation = "West US"
   $drServerName = "mysqlsecondary-$(Get-Random)"
   $failoverGroupName = "failovergrouptutorial-$(Get-Random)"

   # The ip address range that you want to allow to access your server
   # Leaving at 0.0.0.0 will prevent outside-of-azure connections
   # $startIp = "0.0.0.0"
   # $endIp = "0.0.0.0"

   # Show randomized variables
   Write-host "DR Server name is" $drServerName
   Write-host "Failover group name is" $failoverGroupName

   # Create primary Gen5 elastic 2 vCore pool
   Write-host "Creating elastic pool..."
   $elasticPool = New-AzSqlElasticPool -ResourceGroupName $resourceGroupName `
       -ServerName $serverName `
       -ElasticPoolName $poolName `
       -Edition "GeneralPurpose" `
       -vCore 2 `
       -ComputeGeneration Gen5
   $elasticPool

   # Add single db into elastic pool
   Write-host "Creating elastic pool..."
   $addDatabase = Set-AzSqlDatabase -ResourceGroupName $resourceGroupName `
       -ServerName $serverName `
       -DatabaseName $databaseName `
       -ElasticPoolName $poolName
   $addDatabase
   ```

В этом разделе учебника используются следующие командлеты PowerShell:

| Get-Help | Примечания |
|---|---|
| [New-AzSqlElasticPool](/powershell/module/az.sql/new-azsqlelasticpool) | Создает эластичный пул для Базы данных SQL Azure.|
| [Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase) | Определяет свойства базы данных или перемещает ее в эластичный пул. |

---

## <a name="3---create-the-failover-group"></a>3\. Создание группы отработки отказа

На этом шаге вы создадите [группу отработки отказа](auto-failover-group-overview.md) между существующим сервером и новым сервером в другом регионе. Затем вы добавите эластичный пул в группу отработки отказа.

# <a name="portal"></a>[Портал](#tab/azure-portal)

Создайте группу отработки отказа с помощью портала Azure.

1. На [портале Azure](https://portal.azure.com) в меню слева выберите **Azure SQL**. Если **Azure SQL** отсутствует в списке, выберите **Все службы** и в поле поиска введите Azure SQL. (Необязательно) Щелкните звезду рядом с **Azure SQL**, чтобы добавить этот элемент в избранное и область навигации слева.
1. Выберите эластичный пул, созданный в предыдущем разделе, например `myElasticPool`.
1. На панели **Обзор** выберите имя сервера в разделе **Имя сервера**, чтобы открыть параметры для этого сервера.
  
    ![Открытие сервера для эластичного пула](./media/failover-group-add-elastic-pool-tutorial/server-for-elastic-pool.png)

1. Выберите элемент **Группы отработки отказа** в области **Параметры**, а затем щелкните элемент **Добавить группу**, чтобы создать новую группу отработки отказа.

    ![Создание группы отработки отказа.](./media/failover-group-add-elastic-pool-tutorial/elastic-pool-failover-group.png)

1. На странице **Группа отработки отказа** введите или выберите следующие значения, а затем щелкните **Создать**:
    - **Имя группы отработки отказа.** Введите уникальное имя для группы отработки отказа, например `failovergrouptutorial`.
    - **Сервер-получатель.** Выберите вариант *настройки обязательных параметров*, а затем щелкните элемент **Создать сервер**. Кроме того, вы можете выбрать в качестве сервера-получателя уже существующий сервер. Введите указанные ниже значения для нового сервера-получателя и щелкните **Выбрать**.
        - **Имя сервера**: Введите уникальное имя для сервера-получателя, например `mysqlsecondary`.
        - **Имя для входа администратора сервера**. Введите `azureuser`.
        - **Пароль**. Введите сложный пароль, который соответствует требованиям к паролю.
        - **Расположение.** Выберите расположение из раскрывающегося списка, например `East US`. Это расположение не должно совпадать с расположением сервера-источника.

       > [!NOTE]
       > Имя для входа и параметры брандмауэра на этом сервере должны совпадать с параметрами сервера-источника.

       ![Создание сервера-получателя для группы отработки отказа](./media/failover-group-add-elastic-pool-tutorial/create-secondary-failover-server.png)

1. Выберите **Базы данных в группе**, затем выберите эластичный пул, созданный в разделе 2. Должно появиться предупреждение с предложением создать эластичный пул на сервере-получателе. Щелкните это предупреждение, а затем нажмите **ОК**, чтобы создать эластичный пул на сервере-получателе. 

   ![Добавление эластичного пула в группу отработки отказа](./media/failover-group-add-elastic-pool-tutorial/add-elastic-pool-to-failover-group.png)

1. Щелкните **Выбрать**, чтобы применить параметры эластичного пула к группе отработки отказа, а затем щелкните **Создать**, чтобы создать группу отработки отказа. Добавление эластичного пула в группу отработки отказа автоматически запускает процесс георепликации.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Создайте группу отработки отказа с помощью PowerShell.

   ```powershell-interactive
   # Set variables for your server and database
   # $subscriptionId = '<SubscriptionID>'
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"
   # $location = "East US"
   # $adminLogin = "azureuser"
   # $password = "PWD27!"+(New-Guid).Guid
   # $serverName = "mysqlserver-$(Get-Random)"
   # $poolName = "myElasticPool"
   # $databaseName = "mySampleDatabase"
   # $drLocation = "West US"
   # $drServerName = "mysqlsecondary-$(Get-Random)"
   $failoverGroupName = "failovergrouptutorial-$(Get-Random)"

   # Create a secondary server in the failover region
   Write-host "Creating a secondary server in the failover region..."
   New-AzSqlServer -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName `
      -Location $drLocation `
      -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential `
         -ArgumentList $adminlogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))
   Write-host "Secondary server =" $drServerName

   # Create a server firewall rule that allows access from the specified IP range
   Write-host "Configuring firewall for secondary server..."
   New-AzSqlServerFirewallRule -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName `
      -FirewallRuleName "AllowedIPs" -StartIpAddress $startIp -EndIpAddress $endIp
   Write-host "Firewall configured"

   # Create secondary Gen5 elastic 2 vCore pool
   Write-host "Creating secondary elastic pool..."
   $elasticPool = New-AzSqlElasticPool -ResourceGroupName $resourceGroupName `
       -ServerName $drServerName `
       -ElasticPoolName $poolName `
       -Edition "GeneralPurpose" `
       -vCore 2 `
       -ComputeGeneration Gen5
   $elasticPool

   # Create a failover group between the servers
   Write-host "Creating failover group..."
   New-AzSqlDatabaseFailoverGroup `
     â€“ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -PartnerServerName $drServerName  `
      â€“FailoverGroupName $failoverGroupName `
      â€“FailoverPolicy Automatic `
      -GracePeriodWithDataLossHours 2
   Write-host "Failover group created successfully."

   # Add elastic pool to the failover group
   Write-host "Enumerating databases in elastic pool...."
   $FailoverGroup = Get-AzSqlDatabaseFailoverGroup `
                    -ResourceGroupName $resourceGroupName `
                    -ServerName $serverName `
                    -FailoverGroupName $failoverGroupName
   $databases = Get-AzSqlElasticPoolDatabase `
                  -ResourceGroupName $resourceGroupName `
                  -ServerName $serverName `
                  -ElasticPoolName $poolName
   Write-host "Adding databases to failover group..."
   $failoverGroup = $failoverGroup | Add-AzSqlDatabaseToFailoverGroup `
                                     -Database $databases
   $failoverGroup
   ```

В этом разделе учебника используются следующие командлеты PowerShell:

| Get-Help | Примечания |
|---|---|
| [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver) | Создает сервер, на котором размещены базы данных и эластичные пулы. |
| [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule) | Создает правило брандмауэра для сервера. |
| [New-AzSqlElasticPool](/powershell/module/az.sql/new-azsqlelasticpool) | Создает эластичный пул для Базы данных SQL Azure.|
| [New-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/new-azsqldatabasefailovergroup) | Создает группу отработки отказа. |
| [Add-AzSqlDatabaseToFailoverGroup](/powershell/module/az.sql/add-azsqldatabasetofailovergroup) | Добавляет одну или несколько баз данных SQL Azure в группу отработки отказа. |
| [Get-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/get-azsqldatabasefailovergroup) | Возвращает или перечисляет группы отработки отказа для Базы данных SQL Azure. |

---

## <a name="4---test-failover"></a>4\. Тестовая отработка отказа

На этом шаге вы переключите группу отработки отказа на сервер-получатель, а затем восстановите размещение с помощью портала Azure.

# <a name="portal"></a>[Портал](#tab/azure-portal)

Протестируйте отработку отказа для группы отработки отказа с помощью портала Azure.

1. На [портале Azure](https://portal.azure.com) в меню слева выберите **Azure SQL**. Если **Azure SQL** отсутствует в списке, выберите **Все службы** и в поле поиска введите Azure SQL. (Необязательно) Щелкните звезду рядом с **Azure SQL**, чтобы добавить этот элемент в избранное и область навигации слева.
1. Выберите эластичный пул, созданный в предыдущем разделе, например `myElasticPool`.
1. Выберите имя сервера в разделе **Имя сервера**, чтобы открыть параметры для этого сервера.

    ![Открытие сервера для эластичного пула](./media/failover-group-add-elastic-pool-tutorial/server-for-elastic-pool.png)

1. Выберите элемент **Группы отработки отказа** в области **Параметры**, а затем выберите группу отработки отказа, созданную в разделе 2.
  
   ![Выбор группы отработки отказа на портале](./media/failover-group-add-elastic-pool-tutorial/select-failover-group.png)

1. Проверьте, какой сервер является источником, а какой — получателем.
1. Щелкните **Отработка отказа** в области задач, чтобы выполнить отработку отказа для группы отработки отказа с вашим эластичным пулом.
1. Выберите ответ **Да** в предупреждении о том, что сеансы TDS будут отключены.

   ![Отработка отказа для группы отработки отказа, которая содержит базу данных](./media/failover-group-add-elastic-pool-tutorial/failover-sql-db.png)

1. Проверьте, какой сервер является источником, а какой — получателем. Если отработка отказа прошла успешно, роли этих двух серверов должны поменяться между собой.
1. Снова щелкните **Отработка отказа**, чтобы вернуть исходные параметры группы отработки отказа.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Протестируйте отработку отказа для группы отработки отказа с помощью PowerShell.

   ```powershell-interactive
   # Set variables for your server and database
   # $subscriptionId = '<SubscriptionID>'
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"
   # $location = "East US"
   # $adminLogin = "azureuser"
   # $password = "PWD27!"+(New-Guid).Guid
   # $serverName = "mysqlserver-$(Get-Random)"
   # $poolName = "myElasticPool"
   # $databaseName = "mySampleDatabase"
   # $drLocation = "West US"
   # $drServerName = "mysqlsecondary-$(Get-Random)"
   # $failoverGroupName = "failovergrouptutorial-$(Get-Random)"

   # Check role of secondary replica
   Write-host "Confirming the secondary server is secondary...."
   (Get-AzSqlDatabaseFailoverGroup `
      -FailoverGroupName $failoverGroupName `
      -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName).ReplicationRole

   # Failover to secondary server
   Write-host "Failing over failover group to the secondary..."
   Switch-AzSqlDatabaseFailoverGroup `
      -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName `
      -FailoverGroupName $failoverGroupName
   Write-host "Failover group failed over to" $drServerName
   ```

Переключите группу отработки отказа на сервер-получатель, а затем восстановите размещение с помощью PowerShell.

   ```powershell-interactive
   # Set variables for your server and database
   # $subscriptionId = '<SubscriptionID>'
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"
   # $location = "East US"
   # $adminLogin = "azureuser"
   # $password = "PWD27!"+(New-Guid).Guid
   # $serverName = "mysqlserver-$(Get-Random)"
   # $poolName = "myElasticPool"
   # $databaseName = "mySampleDatabase"
   # $drLocation = "West US"
   # $drServerName = "mysqlsecondary-$(Get-Random)"
   # $failoverGroupName = "failovergrouptutorial-$(Get-Random)"

   # Check role of secondary replica
   Write-host "Confirming the secondary server is now primary"
   (Get-AzSqlDatabaseFailoverGroup `
      -FailoverGroupName $failoverGroupName `
      -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName).ReplicationRole

   # Revert failover to primary server
   Write-host "Failing over failover group to the primary...."
   Switch-AzSqlDatabaseFailoverGroup `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -FailoverGroupName $failoverGroupName
   Write-host "Failover group failed over to" $serverName
   ```

В этом разделе учебника используются следующие командлеты PowerShell:

| Get-Help | Примечания |
|---|---|
| [Get-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/get-azsqldatabasefailovergroup) | Возвращает или перечисляет группы отработки отказа для Базы данных SQL Azure. |
| [Switch-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/switch-azsqldatabasefailovergroup)| Выполняет отработку отказа для группы отработки отказа Базы данных SQL Azure. |

---

## <a name="clean-up-resources"></a>Очистка ресурсов

Очищает ресурсы, удаляя группу ресурсов.

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. На [портале Azure](https://portal.azure.com) перейдите к используемой группе ресурсов.
1. Щелкните элемент **Удалить группу ресурсов**, чтобы удалить все ресурсы в этой группе ресурсов, а также саму группу.
1. Введите имя группы ресурсов (`myResourceGroup`) в соответствующем текстовом поле, а затем щелкните элемент **Удалить**, чтобы удалить группу ресурсов.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Очистите ресурсы с помощью PowerShell.

   ```powershell-interactive
   # Set variables for your server and database
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"

   # Clean up resources by removing the resource group
   Write-host "Removing resource group..."
   Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
   Write-host "Resource group removed =" $resourceGroupName
   ```

В этом разделе учебника используются следующие командлеты PowerShell:

| Get-Help | Примечания |
|---|---|
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов. |

---

> [!IMPORTANT]
> Если вы намерены сохранить эту группу ресурсов, но база данных-получатель вам больше не нужна, удалите ее, сначала выполнив удаление из группы отработки отказа. Удаление базы данных-получателя без предварительного удаления ее из группы отработки отказа может привести к непредсказуемым последствиям.

## <a name="full-script"></a>Полный сценарий

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

[!code-powershell-interactive[main](../../../powershell_scripts/sql-database/failover-groups/add-elastic-pool-to-failover-group-az-ps.ps1 "Add elastic pool to a failover group")]

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver) | Создает сервер, на котором размещены базы данных и эластичные пулы. |
| [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule) | Создает правило брандмауэра для сервера. |
| [New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase) | Создает базу данных. |
| [New-AzSqlElasticPool](/powershell/module/az.sql/new-azsqlelasticpool) | Создает эластичный пул для Базы данных SQL Azure.|
| [Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase) | Определяет свойства базы данных или перемещает ее в эластичный пул. |
| [New-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/new-azsqldatabasefailovergroup) | Создает группу отработки отказа. |
| [Get-AzSqlDatabase](/powershell/module/az.sql/get-azsqldatabase) | Получает одну или несколько баз данных из Базы данных SQL. |
| [Add-AzSqlDatabaseToFailoverGroup](/powershell/module/az.sql/add-azsqldatabasetofailovergroup) | Добавляет одну или несколько баз данных SQL Azure в группу отработки отказа. |
| [Get-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/get-azsqldatabasefailovergroup) | Возвращает или перечисляет группы отработки отказа для Базы данных SQL Azure. |
| [Switch-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/switch-azsqldatabasefailovergroup)| Выполняет отработку отказа для группы отработки отказа Базы данных SQL Azure. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов. |

# <a name="portal"></a>[Портал](#tab/azure-portal)

Скрипты для портала Azure не предоставляются.

---

## <a name="next-steps"></a>Дальнейшие действия

В этом учебнике вы добавили эластичный пул, размещенный в службе "База данных SQL Azure", в группу отработки отказа и проверили отработку отказа. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
>
> - Создание отдельной базы данных.
> - Добавление базы данных в эластичный пул.
> - Создание [группы отработки отказа](auto-failover-group-overview.md) для двух эластичных пулов между двумя серверами.
> - Тестовая отработка отказа.

Перейдите к следующему руководству о миграции с помощью DMS.

> [!div class="nextstepaction"]
> [Учебник. Миграция с SQL Server в базу данных в пуле с помощью DMS](../../dms/tutorial-sql-server-to-azure-sql.md?toc=/azure/sql-database/toc.json)
