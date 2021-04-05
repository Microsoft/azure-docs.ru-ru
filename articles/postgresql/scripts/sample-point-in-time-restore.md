---
title: Скрипт Azure CLI. Восстановление сервера Базы данных Azure для PostgreSQL | Документация Майкрософт
description: Этот пример скрипта Azure CLI демонстрирует, как восстановить сервер службы "База данных Azure для PostgreSQL" и его базы данных до предыдущей точки во времени.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.devlang: azurecli
ms.topic: sample
ms.custom: mvc, devx-track-azurecli
ms.date: 02/28/2018
ms.openlocfilehash: 9af4a3d4a484ef22e147640add455741c533e9b7
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94660480"
---
# <a name="restore-an-azure-database-for-postgresql-server-using-azure-cli"></a>Восстановление сервера базы данных Azure для PostgreSQL с помощью Azure CLI
Этот пример сценария интерфейса командной строки позволяет восстановить отдельный сервер базы данных Azure для PostgreSQL до предыдущей точки во времени.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="sample-script"></a>Пример скрипта
В этом примере скрипта отредактируйте выделенные строки, чтобы изменить имя пользователя и пароль администратора на собственные. Замените идентификатор подписки, используемый в командах `az monitor`, собственным.
[!code-azurecli-interactive[main](../../../cli_scripts/postgresql/backup-restore/backup-restore.sh?highlight=15-16 "Restore Azure Database for PostgreSQL.")]

## <a name="clean-up-deployment"></a>Очистка развертывания
После выполнения примера скрипта можно удалить группу ресурсов и все связанные с ней ресурсы, выполнив следующую команду. 
[!code-azurecli-interactive[main](../../../cli_scripts/postgresql/backup-restore/delete-postgresql.sh  "Delete the resource group.")]

## <a name="script-explanation"></a>Описание скрипта
Этот скрипт использует команды, описанные в следующей таблице:

| **Command** | **Примечания** |
|---|---|
| [az group create](/cli/azure/group) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az postgresql server create](/cli/azure/postgres/server#az-postgres-server-create) | Создает сервер PostgreSQL, на котором размещены базы данных. |
| [az postgresql server restore](/cli/azure/postgres/server#az-postgres-server-restore) | Восстановление сервера из резервной копии. |
| [az group delete](/cli/azure/group) | Удаляет группу ресурсов со всеми вложенными ресурсами. |

## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).
- Дополнительные скрипты — [Примеры Azure CLI для базы данных Azure для PostgreSQL](../sample-scripts-azure-cli.md)
- [Как заархивировать и восстановить сервер в базе данных Azure для PostgreSQL с помощью портала Azure](../howto-restore-server-portal.md)
