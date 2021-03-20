---
title: Включение конфигурации подсети с автоматизированной службой для Управляемый экземпляр Azure SQL
description: Включение конфигурации подсети с автоматизированной службой для Управляемый экземпляр Azure SQL
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: srdan-bozovic-msft
ms.author: srbozovi
ms.date: 03/12/2020
ms.openlocfilehash: 67b398194d9094cd99fccaa85ed0df3be362ce2b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91618035"
---
# <a name="enabling-service-aided-subnet-configuration-for-azure-sql-managed-instance"></a>Включение конфигурации подсети с автоматизированной службой для Управляемый экземпляр Azure SQL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Конфигурация подсети с разавтоматизированной службой обеспечивает автоматическое управление конфигурацией сети для подсетей, в которых размещаются управляемые экземпляры. При использовании конфигурации подсети с разавтоматизированной службой пользователь имеет полный контроль над доступом к данным (потокам трафика TDS), а управляемый экземпляр обязан обеспечить непрерывный поток трафика управления для выполнения соглашения об уровне обслуживания.

Автоматически настроенные группы безопасности сети и правила таблицы маршрутов отображаются для клиента и записываются с префиксом _Microsoft. SQL-managedInstances_UseOnly_ _.

Автоматизированная конфигурация включается автоматически после включения [делегирования подсети](../../virtual-network/subnet-delegation-overview.md) для `Microsoft.Sql/managedInstances` поставщика ресурсов.

> [!IMPORTANT] 
> После включения делегирования подсети его нельзя отключить, пока не будет удален последний виртуальный кластер из подсети. Дополнительные сведения о том, как удалить виртуальный кластер, см. в следующей [статье](virtual-cluster-delete.md#delete-a-virtual-cluster-from-the-azure-portal).

> [!NOTE] 
> Так как конфигурация подсети, разворачивающаяся в службе, является важнейшей для обслуживания соглашения об уровне обслуживания, начиная с 1 мая 2020, невозможно будет развернуть управляемые экземпляры в подсетях, которые не делегированы поставщику ресурсов управляемого экземпляра. 1 июля 2020. все подсети, содержащие управляемые экземпляры, будут автоматически делегированы поставщику ресурсов управляемого экземпляра. 

## <a name="enabling-subnet-delegation-for-new-deployments"></a>Включение делегирования подсети для новых развертываний
Чтобы развернуть управляемый экземпляр в в пустой подсети, необходимо делегировать его `Microsoft.Sql/managedInstances` поставщику ресурсов, как описано в следующей [статье](../../virtual-network/manage-subnet-delegation.md). _Обратите внимание, что в указанной статье используется `Microsoft.DBforPostgreSQL/serversv2` поставщик ресурсов, например. `Microsoft.Sql/managedInstances` Вместо этого необходимо использовать поставщик ресурсов._

## <a name="enabling-subnet-delegation-for-existing-deployments"></a>Включение делегирования подсети для существующих развертываний

Чтобы включить делегирование подсети для существующего развертывания управляемого экземпляра, необходимо найти подсеть виртуальной сети, в которой она размещена. 

Чтобы узнать это, вы можете проверить `Virtual network/subnet` `Overview` колонку на портале управляемого экземпляра.

В качестве альтернативы можно выполнить следующие команды PowerShell, чтобы узнать это. Замените **Subscription-ID** своим идентификатором подписки. Также замените **RG-Name** группой ресурсов для управляемого экземпляра и замените **MI-Name** именем управляемого экземпляра.

```powershell
Install-Module -Name Az

Import-Module Az.Accounts
Import-Module Az.Sql

Connect-AzAccount

# Use your subscription ID in place of subscription-id below

Select-AzSubscription -SubscriptionId {subscription-id}

# Replace rg-name with the resource group for your managed instance, and replace mi-name with the name of your managed instance

$mi = Get-AzSqlInstance -ResourceGroupName {rg-name} -Name {mi-name}

$mi.SubnetId
```

После обнаружения подсети управляемого экземпляра необходимо делегировать ее `Microsoft.Sql/managedInstances` поставщику ресурсов, как описано в следующей [статье](../../virtual-network/manage-subnet-delegation.md). _Обратите внимание, что в указанной статье используется `Microsoft.DBforPostgreSQL/serversv2` поставщик ресурсов, например. `Microsoft.Sql/managedInstances` Вместо этого необходимо использовать поставщик ресурсов._


> [!IMPORTANT]
> Включение автоматизированной конфигурации не вызывает отработку отказа или прерывание связи для управляемых экземпляров, которые уже находятся в подсети.
