---
title: Доступ к журналам аудита — Azure CLI — база данных Azure для MariaDB
description: В этой статье описывается, как настроить и получить доступ к журналам аудита в базе данных Azure для MariaDB из Azure CLI.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: how-to
ms.date: 6/24/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: c1d446d8ee2863077ad84c361876758336f5a3cb
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98665180"
---
# <a name="configure-and-access-azure-database-for-maria-db-audit-logs-in-the-azure-cli"></a>Настройка и доступ к журналам аудита базы данных Azure для Марии в Azure CLI

Вы можете настроить [журналы аудита базы данных Azure для MariaDB](concepts-audit-logs.md) из Azure CLI.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные условия

Для работы с этим руководством выполните следующие действия.

- Необходим [сервер базы данных Azure для MariaDB](quickstart-create-mariadb-server-database-using-azure-portal.md).

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="configure-audit-logging"></a>Настройка ведения журнала аудита

>[!IMPORTANT]
> Рекомендуется регистрировать только те типы событий и пользователей, которые необходимы для целей аудита, чтобы гарантировать, что производительность сервера не окажется сильной.

Включите и настройте ведение журнала аудита, выполнив следующие действия. 

1. Включите журналы аудита, установив для параметра **audit_logs_enabled** значение ON. 
    ```azurecli-interactive
    az mariadb server configuration set --name audit_log_enabled --resource-group myresourcegroup --server mydemoserver --value ON
    ```

1. Выберите [типы событий](concepts-audit-logs.md#configure-audit-logging) для записи, обновив параметр **audit_log_events** .
    ```azurecli-interactive
    az mariadb server configuration set --name audit_log_events --resource-group myresourcegroup --server mydemoserver --value "ADMIN,CONNECTION"
    ```

1. Добавьте всех пользователей MariaDB, которые будут исключены из ведения журнала, путем обновления параметра **audit_log_exclude_users** . Укажите пользователей, указав имя пользователя MariaDB.
    ```azurecli-interactive
    az mariadb server configuration set --name audit_log_exclude_users --resource-group myresourcegroup --server mydemoserver --value "azure_superuser"
    ```

1. Добавьте всех конкретных пользователей MariaDB, включаемых для ведения журнала, путем обновления параметра **audit_log_include_users** . Укажите пользователей, указав имя пользователя MariaDB.
    ```azurecli-interactive
    az mariadb server configuration set --name audit_log_include_users --resource-group myresourcegroup --server mydemoserver --value "sampleuser"
    ```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [журналах аудита](concepts-audit-logs.md) в базе данных Azure для MariaDB
- Узнайте, как настроить журналы аудита в [портал Azure](howto-configure-audit-logs-portal.md)
