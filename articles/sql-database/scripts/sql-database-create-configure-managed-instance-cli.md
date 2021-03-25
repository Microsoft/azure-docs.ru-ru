---
title: 'Azure CLI: Создание управляемого экземпляра'
description: Пример скрипта Azure CLI для создания управляемого экземпляра в Базе данных SQL Azure
services: sql-database
ms.service: sql-database
ms.subservice: operations
ms.custom: devx-track-azurecli
ms.devlang: azurecli
ms.topic: sample
author: stevestein
ms.author: sstein
ms.reviewer: carlrab
ms.date: 03/25/2019
ms.openlocfilehash: 54e1aa993f177a4d3bc255287ae8c2fb14cf65af
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87497279"
---
# <a name="use-cli-to-create-an-azure-sql-managed-instance"></a>Создание управляемого экземпляра SQL Azure с помощью CLI

Этот пример скрипта Azure CLI создает управляемый экземпляр Базы данных SQL Azure в выделенной подсети в новой виртуальной сети. Он также настраивает таблицу маршрутов и группу безопасности сети для виртуальной сети. После успешного выполнения скрипта доступ к управляемому экземпляру можно получить из виртуальной сети или из локальной среды. См. статью о [настройке виртуальной машины Azure для подключения к управляемому экземпляру Базы данных SQL Azure]../../azure-sql/managed-instance/connect-vm-instance-configure.md), а также статью о [настройке подключения "точка — сеть" к управляемому экземпляру Базы данных SQL Azure из локальной среды](../../azure-sql/managed-instance/point-to-site-p2s-configure.md).

> [!IMPORTANT]
> Сведения об ограничениях см. в разделах [Поддерживаемые регионы](../../azure-sql/managed-instance/resource-limits.md#supported-regions) и [Поддерживаемые типы подписок](../../azure-sql/managed-instance/resource-limits.md#supported-subscription-types).

Если вы решили установить и использовать интерфейс командной строки локально, для работы с этой статьей вам понадобится Azure CLI 2.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI](/cli/azure/install-azure-cli).

## <a name="sample-script"></a>Пример скрипта

### <a name="sign-in-to-azure"></a>Вход в Azure

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

### <a name="run-the-script"></a>Выполнение скрипта

[!code-azurecli-interactive[main](../../../cli_scripts/sql-database/managed-instance/create-managed-instance.sh "Create managed instance")]

### <a name="clean-up-deployment"></a>Очистка развертывания

Используйте приведенную ниже команду, чтобы удалить группу ресурсов и все связанные с ней ресурсы.

```azurecli-interactive
az group delete --name $resource
```

## <a name="sample-reference"></a>Примеры

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Описание |
|---|---|
| [az network vnet](/cli/azure/network/vnet) | Команды виртуальной сети. |
| [az network vnet subnet](/cli/azure/network/vnet/subnet) | Команды подсети виртуальной сети. |
| [az network route-table](/cli/azure/network/route-table) | Команды таблицы маршрутизации. |
| [az sql mi](/cli/azure/sql/mi) | Команды управляемого экземпляра базы данных SQL. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры сценариев интерфейса командной строки для Базы данных SQL Azure см. в [документации по Базе данных SQL](../../azure-sql/database/az-cli-script-samples-content-guide.md).
