---
title: Пример CLI по восстановлению геоизбыточной резервной копии (База данных SQL Azure)
description: Пример скрипта Azure CLI по восстановлению Управляемого экземпляра Базы данных SQL Azure из геоизбыточной резервной копии.
services: sql-database
ms.service: sql-database
ms.subservice: operations
ms.custom: ''
ms.devlang: azurecli
ms.topic: sample
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein
ms.date: 07/03/2019
ms.openlocfilehash: 5468cfc733b037f6287e8536c8c85994f0840161
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "86231261"
---
# <a name="use-cli-to-restore-a-managed-instance-database-to-another-geo-region"></a>Восстановление Управляемого экземпляра базы данных в другом географическом регионе с помощью CLI

Этот пример скрипта Azure CLI восстанавливает Управляемый экземпляр Базы данных SQL Azure из удаленного географического региона (геовосстановление).  

Если вы решили установить и использовать интерфейс командной строки локально, для работы с этой статьей вам понадобится Azure CLI 2.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI](/cli/azure/install-azure-cli).

## <a name="sample-script"></a>Пример скрипта

### <a name="prerequisites"></a>Предварительные требования

Наличие пары управляемых экземпляров. Ознакомьтесь со сведениями о [создании Управляемого экземпляра SQL Azure с помощью Azure CLI](sql-database-create-configure-managed-instance-cli.md).

### <a name="sign-in-to-azure"></a>Вход в Azure

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

### <a name="run-the-script"></a>Выполнение скрипта

```azurepowershell-interactive
#!/bin/bash
$instance = "<instanceId>" # add instance here
$targetInstance = "<targetInstanceId>" # add target instance here
$resource = "<resourceId>" # add resource here

$randomIdentifier = $(Get-Random)
$managedDatabase = "managedDatabase-$randomIdentifier"

echo "Creating $($managedDatabase) on $($instance)..."
az sql midb create -g $resource --mi $instance -n $managedDatabase

echo "Restoring $($managedDatabase) to $($targetInstance)..."
az sql midb restore -g $resource --mi $instance -n $managedDatabase --dest-name $targetInstance --time "2018-05-20T05:34:22"
```

## <a name="sample-reference"></a>Примеры

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Скрипт | Описание |
|---|---|
| [az sql midb](/cli/azure/sql/midb) | Команды базы данных управляемого экземпляра |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры сценариев интерфейса командной строки для Базы данных SQL Azure см. в [документации по Базе данных SQL](../../azure-sql/database/az-cli-script-samples-content-guide.md).
