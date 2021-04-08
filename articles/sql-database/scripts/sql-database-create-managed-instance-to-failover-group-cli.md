---
title: 'Azure CLI: добавление управляемого экземпляра в группу отработки отказа'
description: Узнайте, как создать два управляемых экземпляра, добавить их в группу отработки отказа, а затем протестировать отработку отказа.
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.custom: devx-track-azurecli
ms.devlang: azurecli
ms.topic: sample
author: MashaMSFT
ms.author: mathoma
ms.reviewer: carlrab
ms.date: 07/16/2019
ms.openlocfilehash: afefc556bcac096d67051f3014c31e449dbbca32
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "91323572"
---
# <a name="use-cli-to-create-an-azure-sql-managed-instance-to-a-failover-group"></a>Создание Управляемого экземпляра SQL Azure в группе отработки отказа с помощью CLI

Этот пример скрипта для Azure CLI создает два управляемых экземпляра, добавляет их в группу отработки отказа, а затем проверяет отработку отказа с основного на дополнительный управляемый экземпляр.

Если вы решили установить и использовать интерфейс командной строки локально, для работы с этой статьей вам понадобится Azure CLI 2.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI](/cli/azure/install-azure-cli).

## <a name="sample-scripts"></a>Примеры сценариев

### <a name="sign-in-to-azure"></a>Вход в Azure

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

### <a name="run-the-script"></a>Выполнение скрипта

[!code-azurecli-interactive[main](../../../cli_scripts/sql-database/failover-groups/add-managed-instance-to-failover-group-az-cli.sh "Add managed instance to a failover group")]

### <a name="clean-up-deployment"></a>Очистка развертывания

Используйте приведенную ниже команду, чтобы удалить группу ресурсов и все связанные с ней ресурсы. Группу ресурсов необходимо будет удалить дважды. При первом удалении группы ресурсов будет удален управляемый экземпляр и виртуальные кластеры, но после этого будет выдаваться сообщение об ошибке `az group delete : Long running operation failed with status 'Conflict'.`. Выполните команду az group delete еще раз, чтобы удалить все оставшиеся ресурсы, а также группу ресурсов.

```azurecli-interactive
az group delete --name $resource
```

## <a name="sample-reference"></a>Примеры

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Описание |
|---|---|
| [az network vnet](/cli/azure/network/vnet) | Команды виртуальной сети.  |
| [az network vnet subnet](/cli/azure/network/vnet/subnet) | Команды подсети виртуальной сети. |
| [az network nsg](/cli/azure/network/nsg) | Команды группы безопасности сети. |
| [az network nsg rule](/cli/azure/network/nsg/rule)| Команды правила сетевой безопасности. |
| [az network route-table](/cli/azure/network/route-table) | Команды таблицы маршрутизации. |
| [az sql mi](/cli/azure/sql/mi) | Команды управляемого экземпляра базы данных SQL. |
| [az network public-ip](/cli/azure/network/public-ip) | Команды сетевого общедоступного IP-адреса. |
| [az network vnet-gateway](/cli/azure/network/vnet-gateway) | Команды шлюза виртуальной сети. |
| [az sql instance-failover-group](/cli/azure/sql/instance-failover-group) | Команды группы отработки отказа Управляемого экземпляра SQL. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры сценариев интерфейса командной строки для Базы данных SQL Azure см. в [документации по Базе данных SQL](../../azure-sql/database/az-cli-script-samples-content-guide.md).
