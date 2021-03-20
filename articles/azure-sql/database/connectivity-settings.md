---
title: Параметры подключения для базы данных SQL Azure и Azure синапсе Analytics
description: В этой статье объясняется, как выбрать версию протокола TLS и как прокси-сервер, так и параметры перенаправления для базы данных SQL Azure и Azure синапсе Analytics.
services: sql-database
ms.service: sql-database
titleSuffix: Azure SQL Database and Azure Synapse Analytics
ms.topic: how-to
author: rohitnayakmsft
ms.author: rohitna
ms.reviewer: sstein, vanto
ms.date: 07/06/2020
ms.openlocfilehash: 84fdca96f2ce42c608e7def98f6a3400964cfe46
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101691932"
---
# <a name="azure-sql-connectivity-settings"></a>Параметры подключения к SQL Azure
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]

В этой статье описываются параметры, управляющие подключением к серверу для базы данных SQL Azure и Azure синапсе Analytics. Эти параметры применяются ко всем базам данных SQL и базе данных Azure синапсе Analytics, связанным с сервером.

> [!IMPORTANT]
> Эта статья не распространяется на Управляемый экземпляр Azure SQL.

Параметры подключения доступны на экране **брандмауэры и виртуальные сети** , как показано на следующем снимке экрана:

 ![Снимок экрана: окно "Параметры подключения".][1]

> [!NOTE]
> Эти параметры вступают в силу сразу же после применения. Ваши клиенты могут столкнуться с потерей подключения, если они не соответствуют требованиям для каждого параметра.

## <a name="deny-public-network-access"></a>Запрет доступа к общедоступной сети

Если для параметра **запретить доступ к общедоступной сети** задано значение **Да**, разрешены только подключения через частные конечные точки. Если этот параметр имеет значение **нет** (по умолчанию), клиенты могут подключаться с помощью общедоступных конечных точек (с правилами брандмауэра на основе IP-адресов или правил брандмауэра на основе виртуальной сети) или частных конечных точек (с помощью частного канала Azure), как описано в статье [Общие сведения о доступе к сети](network-access-controls-overview.md).

 ![Схема, показывающая подключение, если параметр запрет доступа к общедоступной сети имеет значение Да, а параметр запретить доступ к общедоступной сети имеет значение нет.][2]

Любые попытки установить **запрет доступа к общедоступной сети** к **Да** без каких бы то ни было существующих частных конечных точек на логическом сервере будут завершаться с сообщением об ошибке следующего вида:  

```output
Error 42102
Unable to set Deny Public Network Access to Yes since there is no private endpoint enabled to access the server.
Please set up private endpoints and retry the operation.
```

> [!NOTE]
> Чтобы определить правила брандмауэра виртуальной сети на логическом сервере, для которого уже настроены частные конечные точки, установите для параметра **запретить доступ к общедоступной сети** значение **нет**.

Если для параметра **запретить доступ к общедоступной сети** задано значение **Да**, разрешены только подключения через частные конечные точки. Все подключения через общедоступные конечные точки будут отвергнуты следующим сообщением об ошибке:  

```output
Error 47073
An instance-specific error occurred while establishing a connection to SQL Server. 
The public network interface on this server is not accessible. 
To connect to this server, use the Private Endpoint from inside your virtual network.
```

Если для параметра **запретить доступ к общедоступной сети** задано значение **Да**, любые попытки добавления или обновления правил брандмауэра будут отклонены с сообщением об ошибке следующего вида:

```output
Error 42101
Unable to create or modify firewall rules when public network interface for the server is disabled. 
To manage server or database level firewall rules, please enable the public network interface.
```

## <a name="change-public-network-access-via-powershell"></a>Изменение доступа к общедоступной сети с помощью PowerShell

> [!IMPORTANT]
> База данных SQL Azure по-прежнему поддерживает модуль PowerShell Azure Resource Manager, но вся будущая разработка предназначена для модуля AZ. SQL. Сведения об этих командлетах см. в разделе [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Аргументы команд в модулях Az и AzureRm практически идентичны. Для следующего скрипта требуется [модуль Azure PowerShell](/powershell/azure/install-az-ps).

В следующем сценарии PowerShell показано, как `Get` и `Set` свойство **доступа к общедоступной сети** на уровне сервера.

```powershell
# Get the Public Network Access property
(Get-AzSqlServer -ServerName sql-server-name -ResourceGroupName sql-server-group).PublicNetworkAccess

# Update Public Network Access to Disabled
$SecureString = ConvertTo-SecureString "password" -AsPlainText -Force

Set-AzSqlServer -ServerName sql-server-name -ResourceGroupName sql-server-group -SqlAdministratorPassword $SecureString -PublicNetworkAccess "Disabled"
```

## <a name="change-public-network-access-via-cli"></a>Изменение доступа к общедоступной сети с помощью интерфейса командной строки

> [!IMPORTANT]
> Для всех скриптов в этом разделе требуется [Azure CLI](/cli/azure/install-azure-cli).

### <a name="azure-cli-in-a-bash-shell"></a>Azure CLI в оболочке bash

В следующем сценарии интерфейса командной строки показано, как изменить параметр **доступа к общедоступной сети** в оболочке bash:

```azurecli-interactive

# Get current setting for Public Network Access
az sql server show -n sql-server-name -g sql-server-group --query "publicNetworkAccess"

# Update setting for Public Network Access
az sql server update -n sql-server-name -g sql-server-group --set publicNetworkAccess="Disabled"
```

## <a name="minimal-tls-version"></a>Минимальная версия TLS 

Параметр "минимальная версия протокола [TLS](https://support.microsoft.com/help/3135244/tls-1-2-support-for-microsoft-sql-server) " позволяет клиентам выбрать версию TLS, используемую базой данных SQL.

В настоящее время поддерживаются протоколы TLS 1.0, 1.1 и 1.2. Указание минимальной версии TLS гарантирует, что будут поддерживаться более поздние версии TLS. Например, выбор протокола TLS версии 1,1 означает, что принимаются только подключения с протоколом TLS 1,1 и 1,2, а подключения с протоколом TLS 1,0 отклоняются. По завершении тестирования, призванного подтвердить, что приложения его поддерживают, рекомендуется установить минимальную версию TLS 1.2. Эта версия включает исправления уязвимостей, обнаруженных в предыдущих версиях, и является максимальной версией TLS, поддерживаемой в базе данных SQL Azure.

> [!IMPORTANT]
> Значение по умолчанию для минимальной версии TLS — разрешить все версии. После применения версии протокола TLS возврат к значениям по умолчанию будет невозможен.

Для заказчиков, использующих приложения, которые применяют более ранние версии TLS, рекомендуется указывать минимальную версию TLS в соответствии с требованиями конкретного приложения. Для клиентов, которые используют приложения для подключения с использованием незашифрованного соединения, рекомендуется не указывать минимальную версию TLS.

Дополнительные сведения см. в статье [рекомендации по TLS для подключения к базе данных SQL](connect-query-content-reference-guide.md#tls-considerations-for-database-connectivity).

После установки минимальной версии TLS попытки входа от клиентов, которые используют версию TLS ниже минимальной версии сервера TLS, будут завершаться со следующей ошибкой:

```output
Error 47072
Login failed with invalid TLS version
```

## <a name="set-the-minimal-tls-version-via-powershell"></a>Установка минимальной версии TLS с помощью PowerShell

> [!IMPORTANT]
> База данных SQL Azure по-прежнему поддерживает модуль PowerShell Azure Resource Manager, но вся будущая разработка предназначена для модуля AZ. SQL. Сведения об этих командлетах см. в разделе [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Аргументы команд в модулях Az и AzureRm практически идентичны. Для следующего скрипта требуется [модуль Azure PowerShell](/powershell/azure/install-az-ps).

В следующем сценарии PowerShell показано, как `Get` и `Set` свойство **минимальной версии TLS** на уровне логического сервера.

```powershell
# Get the Minimal TLS Version property
(Get-AzSqlServer -ServerName sql-server-name -ResourceGroupName sql-server-group).MinimalTlsVersion

# Update Minimal TLS Version to 1.2
$SecureString = ConvertTo-SecureString "password" -AsPlainText -Force

Set-AzSqlServer -ServerName sql-server-name -ResourceGroupName sql-server-group -SqlAdministratorPassword $SecureString  -MinimalTlsVersion "1.2"
```

## <a name="set-the-minimal-tls-version-via-the-azure-cli"></a>Задайте минимальную версию TLS с помощью Azure CLI

> [!IMPORTANT]
> Для всех скриптов в этом разделе требуется [Azure CLI](/cli/azure/install-azure-cli).

### <a name="azure-cli-in-a-bash-shell"></a>Azure CLI в оболочке bash

В следующем сценарии интерфейса командной строки показано, как изменить параметр **минимальной версии протокола TLS** в оболочке bash:

```azurecli-interactive
# Get current setting for Minimal TLS Version
az sql server show -n sql-server-name -g sql-server-group --query "minimalTlsVersion"

# Update setting for Minimal TLS Version
az sql server update -n sql-server-name -g sql-server-group --set minimalTlsVersion="1.2"
```

## <a name="change-the-connection-policy"></a>Изменение политики подключения

[Политика подключения](connectivity-architecture.md#connection-policy) определяет, как клиенты подключаются к базе данных SQL Azure.

## <a name="change-the-connection-policy-via-powershell"></a>Изменение политики подключения с помощью PowerShell

> [!IMPORTANT]
> База данных SQL Azure по-прежнему поддерживает модуль PowerShell Azure Resource Manager, но вся будущая разработка предназначена для модуля AZ. SQL. Сведения об этих командлетах см. в разделе [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Аргументы команд в модулях Az и AzureRm практически идентичны. Для следующего скрипта требуется [модуль Azure PowerShell](/powershell/azure/install-az-ps).

В следующем сценарии PowerShell показано, как изменить политику подключения с помощью PowerShell.

```powershell
# Get SQL Server ID
$sqlserverid=(Get-AzSqlServer -ServerName sql-server-name -ResourceGroupName sql-server-group).ResourceId

# Set URI
$id="$sqlserverid/connectionPolicies/Default"

# Get current connection policy
(Get-AzResource -ResourceId $id -ApiVersion 2014-04-01 -Verbose).Properties.ConnectionType

# Update connection policy
Set-AzResource -ResourceId $id -Properties @{"connectionType" = "Proxy"} -f
```

## <a name="change-the-connection-policy-via-the-azure-cli"></a>Измените политику подключения с помощью Azure CLI

> [!IMPORTANT]
> Для всех скриптов в этом разделе требуется [Azure CLI](/cli/azure/install-azure-cli).

### <a name="azure-cli-in-a-bash-shell"></a>Azure CLI в оболочке bash

В следующем сценарии интерфейса командной строки показано, как изменить политику подключения в оболочке bash:

```azurecli-interactive
# Get SQL Server ID
sqlserverid=$(az sql server show -n sql-server-name -g sql-server-group --query 'id' -o tsv)

# Set URI
ids="$sqlserverid/connectionPolicies/Default"

# Get current connection policy
az resource show --ids $ids

# Update connection policy
az resource update --ids $ids --set properties.connectionType=Proxy
```

### <a name="azure-cli-from-a-windows-command-prompt"></a>Azure CLI из командной строки Windows

В следующем сценарии интерфейса командной строки показано, как изменить политику подключения из командной строки Windows (с установленным Azure CLI):

```azurecli
# Get SQL Server ID and set URI
FOR /F "tokens=*" %g IN ('az sql server show --resource-group myResourceGroup-571418053 --name server-538465606 --query "id" -o tsv') do (SET sqlserverid=%g/connectionPolicies/Default)

# Get current connection policy
az resource show --ids %sqlserverid%

# Update connection policy
az resource update --ids %sqlserverid% --set properties.connectionType=Proxy
```

## <a name="next-steps"></a>Дальнейшие действия

- Общие сведения о том, как работает подключение в базе данных SQL Azure, см. в статье [Архитектура подключения](connectivity-architecture.md).
- Сведения о том, как изменить политику подключения для сервера, см. в разделе [conn-Policy](/cli/azure/sql/server/conn-policy).

<!--Image references-->
[1]: media/single-database-create-quickstart/manage-connectivity-settings.png
[2]: media/single-database-create-quickstart/manage-connectivity-flowchart.png
