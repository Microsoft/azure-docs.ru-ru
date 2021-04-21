---
title: Скрипт CLI. Скачивание журналов медленных запросов в Базе данных Azure для MySQL
description: Этот пример скрипта Azure CLI демонстрирует, как включить и скачать журналы сервера в службе "База данных Azure для MySQL".
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: azurecli
ms.topic: sample
ms.custom: mvc, devx-track-azurecli
ms.date: 12/02/2019
ms.openlocfilehash: 15d22ea3e0d16d377847ba85acfa41f5ae1043a6
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107791703"
---
# <a name="enable-and-download-server-slow-query-logs-of-an-azure-database-for-mysql-server-using-azure-cli"></a>Включение и загрузка журналов медленных запросов для сервера службы "База данных Azure для MySQL" с помощью Azure CLI
Этот пример скрипта CLI позволяет скачать журналы медленных запросов на одном сервере службы "База данных Azure для MySQL".

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена. 

## <a name="sample-script"></a>Пример скрипта
В этом примере скрипта отредактируйте выделенные строки, чтобы изменить имя пользователя и пароль администратора на собственные. Замените &lt;log_file_name&gt; в командах `az monitor` выбранным именем для файла журнала сервера.
[!code-azurecli-interactive[main](../../../cli_scripts/mysql/server-logs/server-logs.sh?highlight=15-16 "Manipulate with server logs.")]

## <a name="clean-up-deployment"></a>Очистка развертывания
После выполнения примера скрипта можно удалить группу ресурсов и все связанные с ней ресурсы, выполнив следующую команду. 
[!code-azurecli-interactive[main](../../../cli_scripts/mysql/server-logs/delete-mysql.sh  "Delete the resource group.")]

## <a name="script-explanation"></a>Описание скрипта
Этот скрипт использует команды, описанные в следующей таблице:

| **Command** | **Примечания** |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az mysql server create](/cli/azure/mysql/server#az_mysql_server_create) | Создает сервер MySQL, на котором размещены базы данных. |
| [az mysql server configuration list](/cli/azure/mysql/server/configuration#az_mysql_server_configuration_list) | Перечисление значений конфигурации для сервера. |
| [az mysql server configuration set](/cli/azure/mysql/server/configuration#az_mysql_server_configuration_set) | Обновление конфигурации сервера. |
| [az mysql server-logs list](/cli/azure/mysql/server-logs#az_mysql_server_logs_list) | Выводит список файлов журнала для сервера. |
| [az mysql server-logs download](/cli/azure/mysql/server-logs#az_mysql_server_logs_download) | Скачивание файлов журнала. |
| [az group delete](/cli/azure/group#az_group_delete) | Удаляет группу ресурсов со всеми вложенными ресурсами. |

## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).
- Дополнительные скрипты — [Примеры Azure CLI для базы данных Azure для MySQL](../sample-scripts-azure-cli.md)
