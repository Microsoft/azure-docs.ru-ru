---
title: 'Интерфейс командной строки Azure: Создание отдельной базы данных'
description: Используйте этот пример сценария Azure CLI для создания отдельной базы данных.
services: sql-database
ms.service: sql-database
ms.subservice: single-database
ms.custom: sqldbrb=1, devx-track-azurecli
ms.devlang: azurecli
ms.topic: sample
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 06/25/2019
ms.openlocfilehash: 00346accbccd67f542a8496ac097b0236c56a08c
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107773825"
---
# <a name="use-the-azure-cli-to-create-a-single-database-and-configure-a-firewall-rule"></a>Создание отдельной базы данных и настройка правила брандмауэра с помощью Azure CLI

[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqldb.md)]

Этот пример сценария Azure CLI создает отдельную базу данных в службе "База данных SQL" и настраивает правило брандмауэра на уровне сервера. После успешного выполнения сценария доступ к базе данных можно получить из всех служб Azure, а также по настроенному IP-адресу.

Если вы решили установить и использовать интерфейс командной строки локально, для работы с этим руководством вам понадобится Azure CLI 2.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI]( /cli/azure/install-azure-cli).

## <a name="sample-script"></a>Пример скрипта

### <a name="sign-in-to-azure"></a>Вход в Azure

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

### <a name="run-the-script"></a>Выполнение скрипта

[!code-azurecli-interactive[main](../../../../cli_scripts/sql-database/create-and-configure-database/create-and-configure-database.sh "Create SQL Database")]

### <a name="clean-up-deployment"></a>Очистка развертывания

Используйте следующую команду, чтобы удалить группу ресурсов и все связанные с ней ресурсы.

```azurecli-interactive
az group delete --name $resource
```

## <a name="sample-reference"></a>Примеры

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Описание |
|---|---|
| [az sql server](/cli/azure/sql/server#az_sql_server_create) | Команды сервера. |
| [az sql server firewall](/cli/azure/sql/server/firewall-rule#az_sql_server_firewall_rule_create) | Команды брандмауэра для сервера. |
| [az sql db](/cli/azure/sql/db#az_sql_db_create) | Команды базы данных. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры сценариев интерфейса командной строки для Базы данных SQL Azure см. в [документации по Базе данных SQL](../az-cli-script-samples-content-guide.md).
