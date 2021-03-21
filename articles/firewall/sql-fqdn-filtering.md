---
title: Настройка правил приложений брандмауэра Azure с полными доменными именами SQL
description: В этой статье рассказывается о настройке полных доменных имен SQL в правилах приложений брандмауэра Azure.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: how-to
ms.date: 06/18/2020
ms.author: victorh
ms.openlocfilehash: c65f32cc3ce56ddf3fd235de8c002528e7a3cebd
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98791448"
---
# <a name="configure-azure-firewall-application-rules-with-sql-fqdns"></a>Настройка правил приложений брандмауэра Azure с полными доменными именами SQL

Теперь можно настраивать правила приложений брандмауэра Azure с полными доменными именами SQL. Это позволяет ограничить доступ из виртуальных сетей заданными экземплярами SQL Server.

Полные доменные имена SQL позволяют фильтровать трафик

- Из виртуальных сетей в базу данных SQL Azure или Azure синапсе Analytics. Пример: Разрешить только доступ к *sql-server1.database.windows.net*.
- Из локальной среды в управляемые экземпляры SQL Azure или IaaS SQL, выполняемую в ваших виртуальных сетях.
- Из туннеля spoke-to-spoke в управляемые экземпляры SQL Azure или IaaS SQL, выполняемую в ваших виртуальных сетях.

Фильтрация полных доменных имен SQL поддерживается только в [прокси-режиме](../azure-sql/database/connectivity-architecture.md#connection-policy) (порт 1433). При использовании SQL в режиме перенаправления по умолчанию можно фильтровать доступ, используя тег службы SQL в составе [сетевых правил](features.md#network-traffic-filtering-rules).
Если для трафика IaaS SQL используются порты не по умолчанию, можно настроить эти порты в правилах приложений брандмауэра.

## <a name="configure-using-azure-cli"></a>Настройка с использованием интерфейса командной строки Azure

1. Развертывание [брандмауэра Azure с использованием интерфейса командной строки Azure](deploy-cli.md).
2. Если вы фильтруете трафик в базу данных SQL Azure, Azure синапсе Analytics или SQL Управляемый экземпляр, убедитесь, что для режима подключения SQL задано значение **прокси**. Сведения об изменении режима подключения SQL см. в разделе [Настройки подключения SQL Azure](../azure-sql/database/connectivity-settings.md#change-the-connection-policy-via-the-azure-cli).

   > [!NOTE]
   > В *прокси-* режиме SQL может наблюдаться увеличенная задержка по сравнению с режимом *перенаправления*. Если вы хотите продолжить пользоваться режимом перенаправления, который применяется для клиентов, подключающихся внутри Azure, по умолчанию, можно фильтровать доступ, используя [тег службы](service-tags.md) SQL в [сетевых правилах](tutorial-firewall-deploy-portal.md#configure-a-network-rule) брандмауэра.

3. Создайте новую коллекцию правил с помощью правила приложения с полным доменным именем SQL, чтобы разрешить доступ к SQL Server.

   ```azurecli
    az extension add -n azure-firewall
    
    az network firewall application-rule create \ 
    -g FWRG \
    --f azfirewall \ 
    --c sqlRuleCollection \
    --priority 1000 \
    --action Allow \
    --name sqlRule \
    --protocols mssql=1433 \
    --source-addresses 10.0.0.0/24 \
    --target-fqdns sql-serv1.database.windows.net
   ```

## <a name="configure-using-azure-powershell"></a>Настройка с помощью Azure PowerShell

1. Разверните [брандмауэр Azure с помощью Azure PowerShell](deploy-ps.md).
2. Если вы фильтруете трафик в базу данных SQL Azure, Azure синапсе Analytics или SQL Управляемый экземпляр, убедитесь, что для режима подключения SQL задано значение **прокси**. Сведения об изменении режима подключения SQL см. в разделе [Настройки подключения SQL Azure](../azure-sql/database/connectivity-settings.md#change-the-connection-policy-via-the-azure-cli).

   > [!NOTE]
   > В *прокси-* режиме SQL может наблюдаться увеличенная задержка по сравнению с режимом *перенаправления*. Если вы хотите продолжить пользоваться режимом перенаправления, который применяется для клиентов, подключающихся внутри Azure, по умолчанию, можно фильтровать доступ, используя [тег службы](service-tags.md) SQL в [сетевых правилах](tutorial-firewall-deploy-portal.md#configure-a-network-rule) брандмауэра.

3. Создайте новую коллекцию правил с помощью правила приложения с полным доменным именем SQL, чтобы разрешить доступ к SQL Server.

   ```azurepowershell
   $AzFw = Get-AzFirewall -Name "azfirewall" -ResourceGroupName "FWRG"
    
   $sqlRule = @{
      Name          = "sqlRule"
      Protocol      = "mssql:1433" 
      TargetFqdn    = "sql-serv1.database.windows.net"
      SourceAddress = "10.0.0.0/24"
   }
    
   $rule = New-AzFirewallApplicationRule @sqlRule
    
   $sqlRuleCollection = @{
      Name       = "sqlRuleCollection" 
      Priority   = 1000 
      Rule       = $rule
      ActionType = "Allow"
   }
    
   $ruleCollection = New-AzFirewallApplicationRuleCollection @sqlRuleCollection
    
   $Azfw.ApplicationRuleCollections.Add($ruleCollection)    
   Set-AzFirewall -AzureFirewall $AzFw    
   ```

## <a name="configure-using-the-azure-portal"></a>Настройка с помощью портала Azure
1. Развертывание [брандмауэра Azure с использованием интерфейса командной строки Azure](deploy-cli.md).
2. Если вы фильтруете трафик в базу данных SQL Azure, Azure синапсе Analytics или SQL Управляемый экземпляр, убедитесь, что для режима подключения SQL задано значение **прокси**. Сведения об изменении режима подключения SQL см. в разделе [Настройки подключения SQL Azure](../azure-sql/database/connectivity-settings.md#change-the-connection-policy-via-the-azure-cli).  

   > [!NOTE]
   > В *прокси-* режиме SQL может наблюдаться увеличенная задержка по сравнению с режимом *перенаправления*. Если вы хотите продолжить пользоваться режимом перенаправления, который применяется для клиентов, подключающихся внутри Azure, по умолчанию, можно фильтровать доступ, используя [тег службы](service-tags.md) SQL в [сетевых правилах](tutorial-firewall-deploy-portal.md#configure-a-network-rule) брандмауэра.
3. Добавьте правило приложения с соответствующим протоколом, портом и полным доменным именем SQL, а затем нажмите **Сохранить**.
   ![Правило приложения с полным доменным именем SQL](media/sql-fqdn-filtering/application-rule-sql.png)
4. Осуществляйте доступ к SQL с виртуальной машины в виртуальной сети, которая фильтрует трафик с использованием брандмауэра. 
5. Убедитесь, что в [журналах брандмауэра Azure](./firewall-workbook.md) указано, что трафик разрешен.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о прокси-сервере SQL и режимах перенаправления см. в статье [Архитектура подключения к базе данных SQL Azure](../azure-sql/database/connectivity-architecture.md).