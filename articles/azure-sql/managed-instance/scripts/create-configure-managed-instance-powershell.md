---
title: PowerShell. Создание управляемого экземпляра
titleSuffix: Azure SQL Managed Instance
description: Эта статья содержит пример скрипта Azure PowerShell для создания управляемого экземпляра.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: PowerShell
ms.topic: sample
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 03/25/2019
ms.openlocfilehash: fd091cedb67c07b7de2c7e8540e99c3e8daf7dcf
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91323867"
---
# <a name="use-powershell-to-create-a-managed-instance"></a>Создание управляемого экземпляра с помощью PowerShell

[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqlmi.md)]

Этот пример скрипта PowerShell создает управляемый экземпляр в выделенной подсети в новой виртуальной сети. Он также настраивает таблицу маршрутов и группу безопасности сети для виртуальной сети. После успешного выполнения скрипта доступ к управляемому экземпляру можно получить из виртуальной сети или из локальной среды. См. статью о [настройке виртуальной машины Azure для подключения к управляемому экземпляру Базы данных SQL Azure](../connect-vm-instance-configure.md), а также статью о [настройке подключения "точка — сеть" к управляемому экземпляру Базы данных SQL Azure из локальной среды](../point-to-site-p2s-configure.md).

> [!IMPORTANT]
> Сведения об ограничениях см. в разделах [Поддерживаемые регионы](../resource-limits.md#supported-regions) и [Поддерживаемые типы подписок](../resource-limits.md#supported-subscription-types).

[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Чтобы установить и использовать PowerShell локально, вам понадобится Azure PowerShell 1.4.0 или последующей версии для работы с этим руководством. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="sample-script"></a>Пример скрипта

[!code-powershell-interactive[main](../../../../powershell_scripts/sql-database/managed-instance/create-and-configure-managed-instance.ps1 "Create managed instance")]

## <a name="clean-up-deployment"></a>Очистка развертывания

Используйте следующую команду, чтобы удалить группу ресурсов и все связанные с ней ресурсы.

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourcegroupname
```

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт использует часть из следующих команд. Чтобы получить дополнительные сведения об используемых и других командах, включенных в таблицу ниже, используйте ссылки на документацию по конкретным командам.

| Get-Help | Примечания |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Создает группу ресурсов, в которой хранятся все ресурсы.
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Создает виртуальную сеть. |
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/Add-AzVirtualNetworkSubnetConfig) | Добавляет конфигурацию подсети в виртуальную сеть. |
| [Get-AzVirtualNetwork](/powershell/module/az.network/Get-AzVirtualNetwork) | Получает виртуальную сеть в группе ресурсов. |
| [Set-AzVirtualNetwork](/powershell/module/az.network/Set-AzVirtualNetwork) | Задает состояние цели для виртуальной сети. |
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/Get-AzVirtualNetworkSubnetConfig) | Получает подсеть в виртуальной сети. |
| [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/Set-AzVirtualNetworkSubnetConfig) | Настраивает состояние цели для конфигурации подсети в виртуальной сети. |
| [New-AzRouteTable](/powershell/module/az.network/New-AzRouteTable) | Создает таблицу маршрутизации. |
| [Get-AzRouteTable](/powershell/module/az.network/Get-AzRouteTable) | Получает таблицы маршрутов. |
| [Set-AzRouteTable](/powershell/module/az.network/Set-AzRouteTable) | Задает состояние цели для таблицы маршрутов. |
| [New-AzSqlInstance](/powershell/module/az.sql/New-AzSqlInstance) | Создает управляемый экземпляр |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов со всеми вложенными ресурсами. |
|||

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure PowerShell см. в [документации по Azure PowerShell](/powershell/azure/).

Дополнительные примеры скриптов PowerShell для управляемого экземпляра Базы данных SQL Azure можно найти [здесь](../../database/powershell-script-content-guide.md).
