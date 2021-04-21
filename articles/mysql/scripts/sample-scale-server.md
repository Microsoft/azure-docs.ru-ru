---
title: Скрипт CLI. Масштабирование сервера службы "База данных Azure для MySQL"
description: Этот пример скрипта CLI масштабирует сервер базы данных Azure для MySQL до нужного уровня производительности после выполнения запроса к метрикам.
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: azurecli
ms.topic: sample
ms.custom: mvc, devx-track-azurecli
ms.date: 12/02/2019
ms.openlocfilehash: e7dea1cb99ea8ce941b16739bdbd258843877e89
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107791663"
---
# <a name="monitor-and-scale-an-azure-database-for-mysql-server-using-azure-cli"></a>Мониторинг и масштабирование сервера базы данных Azure для MySQL с помощью Azure CLI
Этот пример скрипта CLI масштабирует вычислительные ресурсы и хранилище для отдельного сервера Базы данных Azure для MySQL после выполнения запроса к метрикам. Вычислительные ресурсы можно масштабировать произвольно. Объем хранилища можно только наращивать.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена. 

## <a name="sample-script"></a>Пример скрипта
Укажите в скрипте идентификатор своей подписки.
[!code-azurecli-interactive[main](../../../cli_scripts/mysql/scale-mysql-server/scale-mysql-server.sh "Create and scale Azure Database for MySQL.")]

## <a name="clean-up-deployment"></a>Очистка развертывания
После выполнения примера скрипта можно удалить группу ресурсов и все связанные с ней ресурсы, выполнив следующую команду. 
[!code-azurecli-interactive[main](../../../cli_scripts/mysql/scale-mysql-server/delete-mysql.sh  "Delete the resource group.")]

## <a name="script-explanation"></a>Описание скрипта
Этот скрипт использует команды, описанные в следующей таблице:

| **Command** | **Примечания** |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az mysql server create](/cli/azure/mysql/server#az_mysql_server_create) | Создает сервер MySQL, на котором размещены базы данных. |
| [az mysql server update](/cli/azure/mysql/server#az_mysql_server_update) | Обновляет свойства сервера MySQL. |
| [az monitor metrics list](/cli/azure/monitor/metrics#az_monitor_metrics_list) | Выводит список значений метрики для ресурсов. |
| [az group delete](/cli/azure/group#az_group_delete) | Удаляет группу ресурсов со всеми вложенными ресурсами. |

## <a name="next-steps"></a>Дальнейшие действия
- См. о [вычислительных ресурсах и хранилище Базы данных Azure для MySQL](../concepts-pricing-tiers.md).
- Дополнительные скрипты — [Примеры Azure CLI для базы данных Azure для MySQL](../sample-scripts-azure-cli.md)
- См. об [Azure CLI](/cli/azure).
