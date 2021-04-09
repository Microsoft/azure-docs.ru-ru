---
title: PowerShell. Добавление управляемого экземпляра в группу автоматической отработки отказа
titleSuffix: Azure SQL Managed Instance
description: Пример сценария Azure PowerShell для создания управляемого экземпляра, добавления его в группу автоматической отработки отказа и проверки отработки отказа.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: high-availability
ms.custom: sqldbrb=1
ms.devlang: PowerShell
ms.topic: sample
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 07/16/2019
ms.openlocfilehash: af1d7110cccfd8b0617d6c79eb5373cc7c087159
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94594017"
---
# <a name="use-powershell-to-add-a-managed-instance-to-a-failover-group"></a>Добавление управляемого экземпляра в группу отработки отказа с помощью PowerShell 

[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqlmi.md)]

Этот пример сценария PowerShell создает два управляемых экземпляра, добавляет их в группу отработки отказа, а затем проверяет отработку отказа с основного на дополнительный управляемый экземпляр. 

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Если требуется установить и использовать PowerShell локально, для работы с этим руководством вам понадобится Azure PowerShell 1.4.0 или последующей версии. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="sample-scripts"></a>Примеры сценариев

[!code-powershell-interactive[main](../../../../powershell_scripts/sql-database/failover-groups/add-managed-instance-to-failover-group-az-ps.ps1 "Add managed instance to a failover group")]

## <a name="clean-up-deployment"></a>Очистка развертывания

Используйте следующую команду, чтобы удалить группу ресурсов и все связанные с ней ресурсы. Группу ресурсов необходимо будет удалить дважды. При первом удалении группы ресурсов будет удален управляемый экземпляр и виртуальные кластеры, но после этого будет выдаваться сообщение об ошибке `Remove-AzResourceGroup : Long running operation failed with status 'Conflict'`. Выполните команду Remove-AzResourceGroup еще раз, чтобы удалить все остаточные ресурсы, а также группу ресурсов.

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
```

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Создает группы ресурсов Azure.  |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Создает виртуальную сеть.  |
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) | Добавляет конфигурацию подсети в виртуальную сеть. | 
| [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) | Получает виртуальную сеть в группе ресурсов. | 
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) | Получает подсеть в виртуальной сети. | 
| [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) | Создает группу безопасности сети. | 
| [New-AzRouteTable](/powershell/module/az.network/new-azroutetable) | Создает таблицу маршрутизации. |
| [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) | Обновляет конфигурацию подсети для виртуальной сети.  |
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Обновляет виртуальную сеть.  |
| [Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup) | Возвращает группу безопасности сети. |
| [Add-AzNetworkSecurityRuleConfig](/powershell/module/az.network/add-aznetworksecurityruleconfig)| Добавляет в группу безопасности сети конфигурацию правила безопасности сети. |
| [Set-AzNetworkSecurityGroup](/powershell/module/az.network/set-aznetworksecuritygroup) | Обновляет группу безопасности сети.  | 
| [Add-AzRouteConfig](/powershell/module/az.network/add-azrouteconfig) | Добавляет маршрут в таблицу маршрутизации. |
| [Set-AzRouteTable](/powershell/module/az.network/set-azroutetable) | Обновляет таблицу маршрутизации.  |
| [New-AzSqlInstance](/powershell/module/az.sql/new-azsqlinstance) | Создает управляемый экземпляр  |
| [Get-AzSqlInstance](/powershell/module/az.sql/get-azsqlinstance)| Возвращает информацию об Управляемом экземпляре SQL Azure. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Создает общедоступный IP-адрес.  | 
| [New-AzVirtualNetworkGatewayIpConfig](/powershell/module/az.network/new-azvirtualnetworkgatewayipconfig) | Создает IP-конфигурацию для шлюза виртуальной сети. |
| [New-AzVirtualNetworkGateway](/powershell/module/az.network/new-azvirtualnetworkgateway) | Создание шлюза виртуальной сети |
| [New-AzVirtualNetworkGatewayConnection](/powershell/module/az.network/new-azvirtualnetworkgatewayconnection) | Создает подключение между двумя шлюзами виртуальной сети.   |
| [New-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/new-azsqldatabaseinstancefailovergroup)| Создает новую группу отработки отказа Управляемого экземпляра SQL Azure.  |
| [Get-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/get-azsqldatabaseinstancefailovergroup) | Возвращает или перечисляет группы отработки отказа Управляемого экземпляра SQL.| 
| [Switch-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/switch-azsqldatabaseinstancefailovergroup) | Выполняет отработку отказа для группы отработки отказа Управляемого экземпляра SQL. | 
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов. | 

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure PowerShell см. в [документации по Azure PowerShell](/powershell/azure/).

Дополнительные примеры сценариев PowerShell для Управляемого экземпляра SQL можно найти в статье [Сценарии PowerShell для Управляемого экземпляра SQL Azure](../../database/powershell-script-content-guide.md).
