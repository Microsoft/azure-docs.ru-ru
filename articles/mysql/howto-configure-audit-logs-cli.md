---
title: Доступ к журналам аудита — Azure CLI — база данных Azure для MySQL
description: В этой статье описывается, как настроить и получить доступ к журналам аудита в базе данных Azure для MySQL из Azure CLI.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 6/24/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: db7ffabae785a589bacf349356079f6046039f9c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94542003"
---
# <a name="configure-and-access-audit-logs-in-the-azure-cli"></a>Настройка и доступ к журналам аудита в Azure CLI

Вы можете настроить [журналы аудита базы данных Azure для MySQL](concepts-audit-logs.md) из Azure CLI.

## <a name="prerequisites"></a>Предварительные условия

Для пошагового руководства выполните следующие действия.

- Необходим [сервер базы данных Azure для MySQL](quickstart-create-mysql-server-database-using-azure-portal.md).

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="configure-audit-logging"></a>Настройка ведения журнала аудита

>[!IMPORTANT]
> Рекомендуется регистрировать только те типы событий и пользователей, которые необходимы для целей аудита, чтобы гарантировать, что производительность сервера не окажется сильной.

Включите и настройте ведение журнала аудита, выполнив следующие действия.

1. Включите журналы аудита, установив для параметра **audit_logs_enabled** значение ON. 
    ```azurecli-interactive
    az mysql server configuration set --name audit_log_enabled --resource-group myresourcegroup --server mydemoserver --value ON
    ```

1. Выберите [типы событий](concepts-audit-logs.md#configure-audit-logging) для записи, обновив параметр **audit_log_events** .
    ```azurecli-interactive
    az mysql server configuration set --name audit_log_events --resource-group myresourcegroup --server mydemoserver --value "ADMIN,CONNECTION"
    ```

1. Добавьте пользователей MySQL, которые будут исключены из ведения журнала, путем обновления параметра **audit_log_exclude_users** . Укажите пользователей, указав имя пользователя MySQL.
    ```azurecli-interactive
    az mysql server configuration set --name audit_log_exclude_users --resource-group myresourcegroup --server mydemoserver --value "azure_superuser"
    ```

1. Добавьте конкретных пользователей MySQL, которые будут включаться в ведение журнала, обновив параметр **audit_log_include_users** . Укажите пользователей, указав имя пользователя MySQL.
    ```azurecli-interactive
    az mysql server configuration set --name audit_log_include_users --resource-group myresourcegroup --server mydemoserver --value "sampleuser"
    ```

## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения о [журналах аудита](concepts-audit-logs.md) в базе данных Azure для MySQL
- Узнайте, как настроить журналы аудита в [портал Azure](howto-configure-audit-logs-portal.md)
