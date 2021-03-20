---
title: Настройка существующей виртуальной сети
titleSuffix: Azure SQL Managed Instance
description: В этой статье описывается, как настроить существующую виртуальную сеть и подсеть, в которых можно развернуть Управляемый экземпляр Azure SQL.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein, bonova
ms.date: 03/17/2020
ms.openlocfilehash: 461acc07ee2217a38f7bb59805d4c7e0de4a1e22
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91617661"
---
# <a name="configure-an-existing-virtual-network-for-azure-sql-managed-instance"></a>Настройка имеющейся виртуальной сети для Управляемого экземпляра SQL Azure
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Управляемый экземпляр Azure SQL необходимо развернуть в [виртуальной сети](../../virtual-network/virtual-networks-overview.md) Azure и подсети, выделенной только для управляемых экземпляров. Вы можете использовать существующую виртуальную сеть и подсеть, если они настроены в соответствии с [требованиями к виртуальной сети SQL управляемый экземпляр](connectivity-architecture-overview.md#network-requirements).

Если к вам применим один из следующих вариантов, вы можете проверить и изменить свою сеть с помощью сценария, описанного в этой статье:

- У вас есть новая подсеть, которая все еще не настроена.
- Вы не уверены, что подсеть соответствует [требованиям](connectivity-architecture-overview.md#network-requirements).
- Вы хотите убедиться, что подсеть по-прежнему соответствует [требованиям к сети](connectivity-architecture-overview.md#network-requirements) после внесения изменений.

> [!Note]
> Управляемый экземпляр можно создать только в виртуальных сетях, созданных с помощью модели развертывания Azure Resource Manager. Пиринг между виртуальными сетями, созданными с помощью классической модели развертывания, невозможен. Вычислите размер подсети, следуя указаниям в статье [Определение размера подсети для управляемый экземпляр SQL](vnet-subnet-determine-size.md) . Вы не можете изменить размер подсети после того, как развернете в ней ресурсы.
>
> После создания управляемого экземпляра перемещение экземпляра или виртуальной сети в другую группу ресурсов или подписку не поддерживается.

## <a name="validate-and-modify-an-existing-virtual-network"></a>Проверка и изменение имеющейся виртуальной сети

Если вы хотите создать управляемый экземпляр в существующей подсети, для подготовки подсети рекомендуется использовать следующий сценарий PowerShell:

```powershell
$scriptUrlBase = 'https://raw.githubusercontent.com/Microsoft/sql-server-samples/master/samples/manage/azure-sql-db-managed-instance/delegate-subnet'

$parameters = @{
    subscriptionId = '<subscriptionId>'
    resourceGroupName = '<resourceGroupName>'
    virtualNetworkName = '<virtualNetworkName>'
    subnetName = '<subnetName>'
    }

Invoke-Command -ScriptBlock ([Scriptblock]::Create((iwr ($scriptUrlBase+'/delegateSubnet.ps1?t='+ [DateTime]::Now.Ticks)).Content)) -ArgumentList $parameters
```

Сценарий подготавливает подсеть в три этапа:

1. Проверка: проверяет выбранную виртуальную сеть и подсеть на предмет требований к сети SQL Управляемый экземпляр.
2. Подтверждение: в нем отображается набор изменений, которые необходимо внести для подготовки подсети к развертыванию Управляемый экземпляр SQL. Кроме того, в процессе выдается запрос на согласие.
3. Подготовка. Виртуальная сеть и подсеть настраиваются надлежащим образом.

## <a name="next-steps"></a>Дальнейшие действия

- Общие сведения см. в разделе [что такое SQL управляемый экземпляр?](sql-managed-instance-paas-overview.md).
- Руководство, в котором показано, как создать виртуальную сеть, создать управляемый экземпляр и восстановить базу данных из резервной копии базы данных, см. в разделе [Создание управляемого экземпляра](instance-create-quickstart.md).
- Во избежание проблем при настройке DNS ознакомьтесь со сведениями в статье [Настройка пользовательской службы DNS для Управляемого экземпляра Базы данных SQL Azure](custom-dns-configure.md).
