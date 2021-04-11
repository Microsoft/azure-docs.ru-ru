---
title: Скрипт Azure CLI. Скачивание журналов сервера в Базе данных Azure для PostgreSQL
description: Этот пример сценария Azure CLI демонстрирует, как включить и скачать журналы сервера базы данных Azure для PostgreSQL.
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.devlang: azurecli
ms.topic: sample
ms.custom: mvc, devx-track-azurecli
ms.date: 02/28/2018
ms.openlocfilehash: 3011182794df1c6d60fe286e48c3173dfcfcbe24
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105605252"
---
# <a name="enable-and-download-server-slow-query-logs-of-an-azure-database-for-postgresql-server-using-azure-cli"></a>Включение и загрузка журналов медленных запросов для сервера базы данных Azure для PostgreSQL с помощью Azure CLI
Этот пример сценария интерфейса командной строки позволяет скачать журналы медленных запросов на одном сервере базы данных Azure для PostgreSQL.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="sample-script"></a>Пример скрипта
В этом примере скрипта отредактируйте выделенные строки, чтобы изменить имя пользователя и пароль администратора на собственные. Замените &lt;log_file_name&gt; в командах `az monitor` выбранным именем для файла журнала сервера.
[!code-azurecli-interactive[main](../../../cli_scripts/postgresql/server-logs/server-logs.sh?highlight=15-16 "Manipulate with server logs.")]

## <a name="clean-up-deployment"></a>Очистка развертывания
После выполнения примера скрипта можно удалить группу ресурсов и все связанные с ней ресурсы, выполнив следующую команду. 
[!code-azurecli-interactive[main](../../../cli_scripts/postgresql/server-logs/delete-postgresql.sh  "Delete the resource group.")]

## <a name="script-explanation"></a>Описание скрипта
Этот скрипт использует команды, описанные в следующей таблице:

| **Command** | **Примечания** |
|---|---|
| [az group create](/cli/azure/group) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az postgres server create](/cli/azure/postgres/server) | Создает сервер PostgreSQL, на котором размещены базы данных. |
| [az postgres server configuration list](/cli/azure/postgres/server/configuration) | Перечисление значений конфигурации для сервера. |
| [az postgres server configuration set](/cli/azure/postgres/server/configuration) | Обновление конфигурации сервера. |
| [az postgres server-logs list](/cli/azure/postgres/server-logs) | Выводит список файлов журнала для сервера. |
| [az postgres server-logs download](/cli/azure/postgres/server-logs) | Скачивание файлов журнала. |
| [az group delete](/cli/azure/group) | Удаляет группу ресурсов со всеми вложенными ресурсами. |

## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).
- Дополнительные скрипты — [Примеры Azure CLI для базы данных Azure для PostgreSQL](../sample-scripts-azure-cli.md)
- [Настройка журналов сервера и получение к ним доступа с помощью портала Azure](../howto-configure-server-logs-in-portal.md)
