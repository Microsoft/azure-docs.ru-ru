---
title: Параметр "синхронизировать DNS-серверы виртуальной сети" в виртуальном кластере SQL Управляемый экземпляр
description: Узнайте, как синхронизировать параметры DNS-серверов виртуальной сети в виртуальном кластере SQL Управляемый экземпляр.
services: sql-database
ms.service: sql-managed-instance
author: srdan-bozovic-msft
ms.author: srbozovi
ms.topic: how-to
ms.date: 01/17/2021
ms.openlocfilehash: b95afe513dba2f1da9556b27ec17bcccc9fe88e1
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102173557"
---
# <a name="synchronize-virtual-network-dns-servers-setting-on-sql-managed-instance-virtual-cluster"></a>Параметр "синхронизировать DNS-серверы виртуальной сети" в виртуальном кластере SQL Управляемый экземпляр
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

В этой статье объясняется, когда и как синхронизировать параметр DNS-серверов виртуальной сети в виртуальном кластере SQL Управляемый экземпляр.

## <a name="when-to-synchronize-the-dns-setting"></a>Время синхронизации параметра DNS

Существует несколько сценариев (например, Database Mail, связанные серверы с другими экземплярами SQL Server в облачной или гибридной среде), для которых требуется, чтобы имена частных узлов разрешались из Управляемого экземпляра SQL. В таком случае в Azure необходимо настроить пользовательскую службу DNS. Дополнительные сведения см. в статье [Настройка пользовательской службы DNS для Управляемого экземпляра SQL Azure](custom-dns-configure.md).

Если это изменение реализуется после создания Управляемый экземпляр размещения [виртуального кластера](connectivity-architecture-overview.md#virtual-cluster-connectivity-architecture) , необходимо синхронизировать параметр DNS-серверов в виртуальном кластере с конфигурацией виртуальной сети.

> [!IMPORTANT]
> Параметр Синхронизация DNS-серверов будет влиять на все управляемые экземпляры, размещенные в виртуальном кластере.

## <a name="how-to-synchronize-the-dns-setting"></a>Как синхронизировать параметр DNS

### <a name="azure-rbac-permissions-required"></a>Требуются разрешения RBAC Azure

Конфигурация DNS-сервера для синхронизации пользователей должна иметь одну из следующих ролей Azure:

- Роль владельца подписки или
- Роль участника Управляемый экземпляр или
- Пользовательская роль со следующим разрешением:
  - `Microsoft.Sql/virtualClusters/updateManagedInstanceDnsServers/action`

### <a name="use-azure-powershell"></a>Использование Azure PowerShell

Получение виртуальной сети, в которой были обновлены параметры DNS-серверов.

```PowerShell
$ResourceGroup = 'enter resource group of virtual network'
$VirtualNetworkName = 'enter virtual network name'
$virtualNetwork = Get-AzVirtualNetwork -ResourceGroup $ResourceGroup -Name $VirtualNetworkName
```
Используйте команду PowerShell [Invoke-азресаурцеактион](/powershell/module/az.resources/invoke-azresourceaction) для синхронизации конфигурации DNS-серверов для всех виртуальных кластеров в подсети.

```PowerShell
Get-AzSqlVirtualCluster `
    | where SubnetId -match $virtualNetwork.Id `
    | select Id `
    | Invoke-AzResourceAction -Action updateManagedInstanceDnsServers -Force
```
### <a name="use-the-azure-cli"></a>Использование Azure CLI

Получение виртуальной сети, в которой были обновлены параметры DNS-серверов.

```Azure CLI
resourceGroup="auto-failover-group"
virtualNetworkName="vnet-fog-eastus"
virtualNetwork=$(az network vnet show -g $resourceGroup -n $virtualNetworkName --query "id" -otsv)
```

Чтобы синхронизировать конфигурацию DNS-серверов для всех виртуальных кластеров в подсети, используйте команду Azure CLI [AZ Resource Invoke-Action](/cli/azure/resource#az_resource_invoke_action) .

```Azure CLI
az sql virtual-cluster list --query "[? contains(subnetId,'$virtualNetwork')].id" -o tsv \
    | az resource invoke-action --action updateManagedInstanceDnsServers --ids @-
```
## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о настройке настраиваемой DNS-службы [Настройте настраиваемую службу DNS для управляемый экземпляр Azure SQL](custom-dns-configure.md).
- Общие сведения см. в статье [что такое Azure SQL управляемый экземпляр?](sql-managed-instance-paas-overview.md).
