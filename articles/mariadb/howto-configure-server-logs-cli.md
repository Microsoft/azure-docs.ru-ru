---
title: Доступ к журналам запросов с высокой производительностью. Azure CLI — база данных Azure для MariaDB
description: В этой статье описывается, как получить доступ к журналам ожидания в базе данных Azure для MariaDB с помощью служебной программы командной строки Azure CLI.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.devlang: azurecli
ms.topic: how-to
ms.date: 4/13/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: cea1571a48afe00350dde247c3c10e222a446247
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98662395"
---
# <a name="configure-and-access-azure-database-for-maria-db-slow-query-logs-by-using-azure-cli"></a>Настройка журналов запросов базы данных Azure для Марии и доступ к ним с помощью Azure CLI

Вы можете скачать базу данных Azure для MariaDBных журналов запросов с помощью Azure CLI, служебной программы командной строки Azure.

## <a name="prerequisites"></a>Предварительные требования
Прежде чем приступить к выполнению этого руководства, необходимы следующие компоненты:
- [Сервер базы данных Azure для MariaDB](quickstart-create-mariadb-server-database-using-azure-cli.md)
- [Azure CLI](/cli/azure/install-azure-cli) или Azure Cloud Shell в браузере.

## <a name="configure-logging"></a>Настройка журнала
Можно настроить на сервере доступ к журналу медленных запросов MySQL, выполнив следующие действия:
1. Включите ведение журнала для запросов, задав для **параметра \_ \_ журнала скорости запроса** значение ON.
2. Выберите место вывода журналов для использования **\_ выходных данных журнала**. Чтобы отправить журналы в локальное хранилище и Azure Monitor журналы диагностики, выберите **файл**. Чтобы отправить журналы только в Azure Monitor журналы, выберите **нет** .
3. Настройте другие параметры, такие как **long\_query\_time** и **log\_slow\_admin\_statements**.

Ознакомьтесь со статьей [Настройка параметров конфигурации сервера с помощью Azure CLI](howto-configure-server-parameters-cli.md), чтобы узнать, как задать значение этих параметров с помощью Azure CLI.

Например, приведенная ниже команда интерфейса командной строки включает журнал медленных запросов, задает длительность запроса в 10 секунд и отключает ведение журнала медленных инструкций администрирования. Наконец, она выводит параметры конфигурации, чтобы вы могли их просмотреть.
```azurecli-interactive
az mariadb server configuration set --name slow_query_log --resource-group myresourcegroup --server mydemoserver --value ON
az mariadb server configuration set --name log_output --resource-group myresourcegroup --server mydemoserver --value FILE
az mariadb server configuration set --name long_query_time --resource-group myresourcegroup --server mydemoserver --value 10
az mariadb server configuration set --name log_slow_admin_statements --resource-group myresourcegroup --server mydemoserver --value OFF
az mariadb server configuration list --resource-group myresourcegroup --server mydemoserver
```

## <a name="list-logs-for-azure-database-for-mariadb-server"></a>Получение списка журналов сервера в Базе данных Azure для MariaDB
Если **log_output** настроена на "файл", доступ к журналам можно получить непосредственно из локального хранилища сервера. Чтобы получить список доступных файлов журнала запросов с высокой запятыми для сервера, выполните команду [AZ MariaDB Server-Logs List](/cli/azure/mariadb/server-logs#az-mariadb-server-logs-list) .

Вы можете вывести список файлов журнала для сервера **mydemoserver.mariadb.database.azure.com** в группе ресурсов **myresourcegroup**. Затем направьте список файлов журнала в текстовый файл с именем **log\_files\_list.txt**.
```azurecli-interactive
az mariadb server-logs list --resource-group myresourcegroup --server mydemoserver > log_files_list.txt
```
## <a name="download-logs-from-the-server"></a>Скачивание журналов с сервера
Если **log_output** настроен на "файл", можно загрузить отдельные файлы журнала с сервера с помощью команды [AZ MariaDB Server-Logs download](/cli/azure/mariadb/server-logs#az-mariadb-server-logs-download) .

В следующем примере в локальную среду скачивается определенный файл журнала для сервера **mydemoserver.mariadb.database.azure.com** в группе ресурсов **myresourcegroup**.
```azurecli-interactive
az mariadb server-logs download --name mysql-slow-mydemoserver-2018110800.log --resource-group myresourcegroup --server mydemoserver
```

## <a name="next-steps"></a>Дальнейшие действия
- Сведения о [очень низких журналах запросов в базе данных Azure для MariaDB](concepts-server-logs.md).
