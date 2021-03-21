---
title: Доступ к журналам запросов с высокой производительностью. Azure CLI — база данных Azure для MySQL
description: В этой статье описывается, как получить доступ к журналам запросов к службе "база данных Azure для MySQL" с помощью Azure CLI.
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: azurecli
ms.topic: how-to
ms.date: 4/13/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 945a67f81010a61adf814f6f6f422eba5001b48d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95998555"
---
# <a name="configure-and-access-slow-query-logs-by-using-azure-cli"></a>Настройка и доступ к журналам запросов с высокой занесением с помощью Azure CLI
Вы можете скачать журналы запросов к базе данных Azure для MySQL с помощью Azure CLI, служебной программы командной строки Azure.

## <a name="prerequisites"></a>Предварительные требования
Прежде чем приступить к выполнению этого руководства, необходимы следующие компоненты:
- [Сервер базы данных Azure для MySQL](quickstart-create-mysql-server-database-using-azure-cli.md)
- [Azure CLI](/cli/azure/install-azure-cli) или Azure Cloud Shell в браузере.

## <a name="configure-logging"></a>Настройка журнала
Можно настроить на сервере доступ к журналу медленных запросов MySQL, выполнив следующие действия:
1. Включите ведение журнала для запросов, задав для **параметра \_ \_ журнала скорости запроса** значение ON.
2. Выберите место вывода журналов для использования **\_ выходных данных журнала**. Чтобы отправить журналы в локальное хранилище и Azure Monitor журналы диагностики, выберите **файл**. Чтобы отправить журналы только в Azure Monitor журналы, выберите **нет** .
3. Настройте другие параметры, такие как **long\_query\_time** и **log\_slow\_admin\_statements**.

Ознакомьтесь со статьей [Настройка параметров конфигурации сервера с помощью Azure CLI](howto-configure-server-parameters-using-cli.md), чтобы узнать, как задать значение этих параметров с помощью Azure CLI.

Например, приведенная ниже команда интерфейса командной строки включает журнал медленных запросов, задает длительность запроса в 10 секунд и отключает ведение журнала медленных инструкций администрирования. Наконец, она выводит параметры конфигурации, чтобы вы могли их просмотреть.
```azurecli-interactive
az mysql server configuration set --name slow_query_log --resource-group myresourcegroup --server mydemoserver --value ON
az mysql server configuration set --name log_output --resource-group myresourcegroup --server mydemoserver --value FILE
az mysql server configuration set --name long_query_time --resource-group myresourcegroup --server mydemoserver --value 10
az mysql server configuration set --name log_slow_admin_statements --resource-group myresourcegroup --server mydemoserver --value OFF
az mysql server configuration list --resource-group myresourcegroup --server mydemoserver
```

## <a name="list-logs-for-azure-database-for-mysql-server"></a>Получение списка журналов для сервера базы данных Azure для MySQL
Если **log_output** настроена на "файл", доступ к журналам можно получить непосредственно из локального хранилища сервера. Чтобы получить список доступных файлов журнала запросов с высокой запятыми для сервера, выполните команду [AZ MySQL Server-Logs List](/cli/azure/mysql/server-logs#az-mysql-server-logs-list) .

Вы можете вывести список файлов журнала для сервера **mydemoserver.mysql.database.azure.com** в группе ресурсов **myresourcegroup**. Затем направьте список файлов журнала в текстовый файл с именем **log\_files\_list.txt**.
```azurecli-interactive
az mysql server-logs list --resource-group myresourcegroup --server mydemoserver > log_files_list.txt
```
## <a name="download-logs-from-the-server"></a>Скачивание журналов с сервера
Если **log_output** настроена на "файл", можно скачать отдельные файлы журналов с сервера с помощью команды [AZ MySQL Server-Logs download](/cli/azure/mysql/server-logs#az-mysql-server-logs-download) .

В следующем примере в локальную среду скачивается определенный файл журнала для сервера **mydemoserver.mysql.database.azure.com** в группе ресурсов **myresourcegroup**.
```azurecli-interactive
az mysql server-logs download --name 20170414-mydemoserver-mysql.log --resource-group myresourcegroup --server mydemoserver
```

## <a name="next-steps"></a>Дальнейшие действия
- Сведения о [очень низких журналах запросов в базе данных Azure для MySQL](concepts-server-logs.md).
