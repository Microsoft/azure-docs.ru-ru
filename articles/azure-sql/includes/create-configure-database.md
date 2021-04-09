---
author: jovanpop-msft
ms.service: sql-database
ms.subservice: operations
ms.topic: include
ms.date: 12/17/2020
ms.author: jovanpop
ms.openlocfilehash: d1422d029bc2c5d41330e4ac2d21ff14e4fa82ab
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97703978"
---
  Для создания и настройки базы данных можно использовать одно из этих кратких руководств.

  | Действие | База данных SQL | Управляемый экземпляр SQL | SQL Server на виртуальной машине Azure | Azure Synapse Analytics |
  |:--- |:--- |:---|:---|:---|
  | Создание| [Портал](../database/single-database-create-quickstart.md) | [Портал](../managed-instance/instance-create-quickstart.md) | [Портал](../virtual-machines/windows/sql-vm-create-portal-quickstart.md) | [Портал](../../synapse-analytics/quickstart-create-workspace.md) |
  || [CLI](../database/scripts/create-and-configure-database-cli.md) | | | [CLI](../../synapse-analytics/quickstart-create-workspace-cli.md) |
  || [PowerShell](../database/scripts/create-and-configure-database-powershell.md) | [PowerShell](../managed-instance/scripts/create-configure-managed-instance-powershell.md) | [PowerShell](../virtual-machines/windows/sql-vm-create-powershell-quickstart.md) | [PowerShell](../../synapse-analytics/quickstart-create-workspace-powershell.md) |
  || | | [Шаблон развертывания](../virtual-machines/windows/create-sql-vm-resource-manager-template.md) | [Шаблон развертывания](../../synapse-analytics/quickstart-deployment-template-workspaces.md) | 
  | Configure | [Правило брандмауэра для IP-адресов на уровне сервера](../database/firewall-create-server-level-portal-quickstart.md)| [Подключение из виртуальной машины](../managed-instance/connect-vm-instance-configure.md)| |
  |||[Подключение из локальной сети](../managed-instance/point-to-site-p2s-configure.md) | [Подключение к экземпляру SQL Server](../virtual-machines/windows/sql-vm-create-portal-quickstart.md) |
  | Получение сведений о подключении | [Azure SQL;](../database/connect-query-content-reference-guide.md#get-server-connection-information)|[Azure SQL;](../database/connect-query-content-reference-guide.md#get-server-connection-information)| [Виртуальная машина SQL](../virtual-machines/windows/sql-vm-create-portal-quickstart.md?#connect-to-sql-server)| [Synapse SQL](../../synapse-analytics/sql/connect-overview.md#find-your-server-name)|
  |||||
