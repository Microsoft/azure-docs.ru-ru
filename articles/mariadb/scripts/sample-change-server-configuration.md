---
title: Скрипт CLI для изменения параметров сервера Базы данных Azure для MariaDB
description: В этом примере сценария для интерфейса командной строки перечисляются все доступные конфигурации и обновления серверов Базы данных Azure для MariaDB.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.devlang: azurecli
ms.topic: sample
ms.custom: mvc, devx-track-azurecli
ms.date: 12/02/2019
ms.openlocfilehash: c7a46f98f74648ccae9f9f9f94c218d42056decb
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98664653"
---
# <a name="list-and-update-configurations-of-an-azure-database-for-mariadb-server-using-azure-cli"></a>Получение списка и обновление конфигураций сервера Базы данных Azure для MariaDB с помощью Azure CLI
В этом примере скрипта CLI выводится список всех доступных параметров конфигурации и их допустимые значения для сервера Базы данных Azure для MariaDB. Кроме того, здесь задается значение *innodb_lock_wait_timeout*, отличное от значения по умолчанию.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="sample-script"></a>Пример скрипта
В этом примере скрипта отредактируйте выделенные строки, чтобы изменить имя пользователя и пароль администратора на собственные.
[!code-azurecli-interactive[main](../../../cli_scripts/mariadb/change-server-configurations/change-server-configurations.sh?highlight=15-16 "List and update configurations of Azure Database for MariaDB.")]

## <a name="clean-up-deployment"></a>Очистка развертывания
После выполнения примера скрипта можно удалить группу ресурсов и все связанные с ней ресурсы, выполнив следующую команду.
[!code-azurecli-interactive[main](../../../cli_scripts/mariadb/change-server-configurations/delete-mariadb.sh  "Delete the resource group.")]

## <a name="script-explanation"></a>Описание скрипта
Этот скрипт использует команды, описанные в следующей таблице:

| **Command** | **Примечания** |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az mariadb server create](/cli/azure/mariadb/server#az-mariadb-server-create) | Создает сервер MariaDB, на котором размещены базы данных. |
| [az mariadb server configuration list](/cli/azure/mariadb/server/configuration#az-mariadb-server-configuration-list) | Позволяет получить список конфигураций сервера Базы данных Azure для MariaDB. |
| [az mariadb server configuration set](/cli/azure/mariadb/server/configuration#az-mariadb-server-configuration-set) | Обновляет конфигурацию сервера Базы данных Azure для MariaDB. |
| [az mariadb server configuration show](/cli/azure/mariadb/server/configuration#az-mariadb-server-configuration-show) | Обновляет конфигурацию сервера Базы данных Azure для MariaDB. |
| [az group delete](/cli/azure/group#az-group-delete) | Удаляет группу ресурсов со всеми вложенными ресурсами. |

## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).
- Дополнительные скрипты — [Примеры Azure CLI для Базы данных Azure для MariaDB](../sample-scripts-azure-cli.md)
- Дополнительные сведения о параметрах сервера см. в статье [Как настроить параметры сервера в базе данных Azure для MariaDB с помощью портала Azure](../howto-server-parameters.md).
