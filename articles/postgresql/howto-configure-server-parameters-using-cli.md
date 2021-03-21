---
title: Настройка параметров — база данных Azure для PostgreSQL — один сервер
description: В этой статье описывается, как настроить параметры postgres в базе данных Azure для PostgreSQL-Single Server с помощью Azure CLI.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.devlang: azurecli
ms.topic: how-to
ms.date: 06/19/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 4231f348f99073406fcb6a5bef9bf0f84cacf2eb
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96005577"
---
# <a name="customize-server-configuration-parameters-for-azure-database-for-postgresql---single-server-using-azure-cli"></a>Настройка параметров конфигурации сервера для базы данных Azure для PostgreSQL — один сервер с помощью Azure CLI
С помощью интерфейса командной строки (Azure CLI) можно вывести список параметров конфигурации для сервера Azure PostgreSQL, а также отобразить и обновить их. Только подмножество конфигураций ядра предоставляется на уровне сервера и может быть изменено. 

## <a name="prerequisites"></a>Предварительные требования
Прежде чем приступить к выполнению этого руководства, необходимы следующие компоненты:
- создайте базу данных и сервер базы данных Azure для PostgreSQL, выполнив инструкции из раздела [Создание базы данных Azure для PostgreSQL](quickstart-create-server-database-azure-cli.md);
- установите интерфейс командной строки [Azure CLI](/cli/azure/install-azure-cli) на компьютере или используйте [Azure Cloud Shell](../cloud-shell/overview.md) на портале Azure с помощью браузера.

## <a name="list-server-configuration-parameters-for-azure-database-for-postgresql-server"></a>Получение списка параметров конфигурации сервера для базы данных Azure для сервера PostgreSQL
Чтобы перечислить все изменяемые параметры на сервере и их значения, выполните команду [az postgres server configuration list](/cli/azure/postgres/server/configuration).

Например, можно вывести список параметров конфигурации сервера для сервера **mydemoserver.postgres.database.azure.com** в группе ресурсов **myresourcegroup**.
```azurecli-interactive
az postgres server configuration list --resource-group myresourcegroup --server mydemoserver
```
## <a name="show-server-configuration-parameter-details"></a>Отображение сведений о параметре конфигурации сервера
Чтобы отобразить сведения об определенном параметре конфигурации для сервера, выполните команду [az postgres server configuration show](/cli/azure/postgres/server/configuration).

Этот пример отображает сведения параметра конфигурации сервера **log\_min\_messages** для сервера **mydemoserver.postgres.database.azure.com** в группе ресурсов **myresourcegroup.**
```azurecli-interactive
az postgres server configuration show --name log_min_messages --resource-group myresourcegroup --server mydemoserver
```
## <a name="modify-server-configuration-parameter-value"></a>Изменение значения параметра конфигурации сервера
Вы также можете изменить значение определенного параметра конфигурации сервера. При этом обновляется базовое значение конфигурации для ядра сервера. Чтобы обновить конфигурацию, выполните команду [az postgres server configuration set](/cli/azure/postgres/server/configuration). 

Обновите параметр конфигурации сервера **log\_min\_messages** для сервера **mydemoserver.postgres.database.azure.com** в группе ресурсов **myresourcegroup.**
```azurecli-interactive
az postgres server configuration set --name log_min_messages --resource-group myresourcegroup --server mydemoserver --value INFO
```
Если вы хотите сбросить значение параметра конфигурации, нужно просто убрать необязательный параметр `--value`, и служба применит значение по умолчанию. В приведенном выше примере это может выглядеть следующим образом.
```azurecli-interactive
az postgres server configuration set --name log_min_messages --resource-group myresourcegroup --server mydemoserver
```
Данная команда восстановит значение по умолчанию **WARNING** для параметра конфигурации **log\_min\_messages**. Дополнительные сведения о конфигурации сервера и допустимых значениях приведены в документации PostgreSQL по [конфигурации сервера](https://www.postgresql.org/docs/9.6/static/runtime-config.html).

## <a name="next-steps"></a>Дальнейшие действия
- [Сведения о перезагрузке сервера](howto-restart-server-cli.md)
- Чтобы настроить журналы сервера и получать к ним доступ, ознакомьтесь с разделом [Журналы сервера в базе данных Azure для PostgreSQL](concepts-server-logs.md).
