---
title: 'Azure CLI: Импорт BACPAC-файла в базу данных в Базе данных SQL Azure'
description: Пример сценария Azure CLI для импорта BACPAC-файла в базу данных в Базе данных SQL Azure
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: load & move data, devx-track-azurecli
ms.devlang: azurecli
ms.topic: sample
author: stevestein
ms.author: sstein
ms.reviewer: carlrab
ms.date: 05/24/2019
ms.openlocfilehash: a76a2e72533068f37613d801e39f9451098b89e5
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107786609"
---
# <a name="use-cli-to-import-a-bacpac-file-into-a-database-in-sql-database"></a>Использование CLI для импорта BACPAC-файла в базу данных в Базе данных SQL

Этот пример сценария Azure CLI импортирует базу данных из *BACPAC*-файла в базу данных в Базе данных SQL.  

Если вы решили установить и использовать интерфейс командной строки локально, для работы с этой статьей вам понадобится Azure CLI 2.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI](/cli/azure/install-azure-cli).

## <a name="sample-script"></a>Пример скрипта

### <a name="sign-in-to-azure"></a>Вход в Azure

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

### <a name="run-the-script"></a>Выполнение скрипта

[!code-azurecli-interactive[main](../../../cli_scripts/sql-database/import-from-bacpac/import-from-bacpac.sh "Create SQL Database")]

### <a name="clean-up-deployment"></a>Очистка развертывания

Используйте приведенную ниже команду, чтобы удалить группу ресурсов и все связанные с ней ресурсы.

```azurecli-interactive
az group delete --name $resource
```

## <a name="sample-reference"></a>Примеры

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Описание |
|---|---|
| [az sql server](/cli/azure/sql/server) | Команды сервера. |
| [az sql db import](/cli/azure/sql/db#az_sql_db_import) | Команда импорта базы данных. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры сценариев интерфейса командной строки для Базы данных SQL Azure см. в [документации по Базе данных SQL](../../azure-sql/database/az-cli-script-samples-content-guide.md).
