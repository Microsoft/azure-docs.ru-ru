---
title: Настройка параметров сервера — Azure CLI — база данных Azure для MariaDB
description: В этой статье описывается настройка параметров службы в Базе данных Azure для MariaDB с помощью служебной программы командной строки Azure CL.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.devlang: azurecli
ms.topic: how-to
ms.date: 10/1/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 4009d8047dae7bf8d9ba66566ff8797fa09a8878
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98662310"
---
# <a name="configure-server-parameters-in-azure-database-for-mariadb-using-the-azure-cli"></a>Настройка параметров сервера в базе данных Azure для MariaDB с помощью Azure CLI
С помощью служебной программы командной строки (Azure CLI) можно вывести список параметров конфигурации для сервера Базы данных Azure для MariaDB, а также отобразить и обновить их. Только подмножество конфигураций ядра предоставляется на уровне сервера и может быть изменено.

>[!Note]
> Параметры сервера можно обновлять глобально на уровне сервера с помощью [Azure CLI](./howto-configure-server-parameters-cli.md), [PowerShell](./howto-configure-server-parameters-using-powershell.md)или [портала Azure](./howto-server-parameters.md).

## <a name="prerequisites"></a>Предварительные требования
Прежде чем приступить к выполнению этого руководства, необходимы следующие компоненты:
- [Сервер Базы данных Azure для MariaDB](quickstart-create-mariadb-server-database-using-azure-cli.md)
- Программа командной строки [Azure CLI](/cli/azure/install-azure-cli) (или используйте Azure Cloud Shell в браузере).

## <a name="list-server-configuration-parameters-for-azure-database-for-mariadb-server"></a>Получение списка параметров конфигурации сервера Базы данных Azure для MariaDB
Чтобы перечислить все изменяемые параметры на сервере и их значения, выполните команду [az mariadb server configuration list](/cli/azure/mariadb/server/configuration#az-mariadb-server-configuration-list).

Например, можно вывести список параметров конфигурации сервера **mydemoserver.mariadb.database.azure.com** в группе ресурсов **myresourcegroup**.
```azurecli-interactive
az mariadb server configuration list --resource-group myresourcegroup --server mydemoserver
```

Определение каждого из перечисленных параметров см. в разделе ссылок MariaDB на странице [Server System Variables](https://mariadb.com/kb/en/library/server-system-variables/) (Системные переменные сервера).

## <a name="show-server-configuration-parameter-details"></a>Отображение сведений о параметре конфигурации сервера
Чтобы отобразить сведения об определенном параметре конфигурации для сервера, выполните команду [az mariadb server configuration show](/cli/azure/mariadb/server/configuration#az-mariadb-server-configuration-show).

Этот пример отображает сведения параметра конфигурации сервера **slow\_query\_log** для сервера **mydemoserver.mariadb.database.azure.com** в группе ресурсов **myresourcegroup.**
```azurecli-interactive
az mariadb server configuration show --name slow_query_log --resource-group myresourcegroup --server mydemoserver
```

## <a name="modify-a-server-configuration-parameter-value"></a>Изменение значения параметра конфигурации сервера
Вы также можете изменить значение определенного параметра конфигурации сервера. При этом обновляется базовое значение конфигурации для ядра СУБД сервера MariaDB. Чтобы обновить конфигурацию, выполните команду [az mariadb server configuration set](/cli/azure/mariadb/server/configuration#az-mariadb-server-configuration-set). 

Чтобы обновить параметры конфигурации **slow\_query\_log** для сервера **mydemoserver.mariadb.database.azure.com** в группе ресурсов **myresourcegroup**, используйте следующую команду:
```azurecli-interactive
az mariadb server configuration set --name slow_query_log --resource-group myresourcegroup --server mydemoserver --value ON
```

Если вы хотите сбросить значение параметра конфигурации, нужно просто опустить необязательный параметр `--value`, и служба применит значение по умолчанию. В приведенном выше примере это может выглядеть следующим образом.
```azurecli-interactive
az mariadb server configuration set --name slow_query_log --resource-group myresourcegroup --server mydemoserver
```

Этот код выполняет сброс конфигурации **slow\_query\_log** к значению по умолчанию **OFF**. 

## <a name="setting-parameters-not-listed"></a>Параметры настройки не указаны
Если параметр сервера, который требуется обновить, не указан в портал Azure, можно при необходимости задать параметр на уровне соединения с помощью `init_connect` . Это задает параметры сервера для каждого клиента, подключающегося к серверу. 

Обновите параметр конфигурации сервера **init \_ connect** сервера **mydemoserver.MariaDB.Database.Azure.com** в разделе **myresourcegroup** группы ресурсов, чтобы задать значения, такие как кодировка.
```azurecli-interactive
az mariadb server configuration set --name init_connect --resource-group myresourcegroup --server mydemoserver --value "SET character_set_client=utf8;SET character_set_database=utf8mb4;SET character_set_connection=latin1;SET character_set_results=latin1;"
```

## <a name="working-with-the-time-zone-parameter"></a>Работа с параметром часового пояса

### <a name="populating-the-time-zone-tables"></a>Заполнение таблиц часовых поясов

Таблицы часовых поясов на сервере можно заполнить, вызвав хранимую процедуру `mysql.az_load_timezone` с помощью такого инструмента, как командная строка MariaDB или MariaDB Workbench.

> [!NOTE]
> Если вы используете команду `mysql.az_load_timezone` в MariaDB Workbench, может потребоваться предварительно отключить режим безопасного обновления с помощью `SET SQL_SAFE_UPDATES=0;`.

```sql
CALL mysql.az_load_timezone();
```

> [!IMPORTANT]
> Необходимо перезапустить сервер, чтобы убедиться, что таблицы часовых поясов заполнены правильно. Чтобы перезапустить сервер, используйте [портал Azure](howto-restart-server-portal.md) или [CLI](howto-restart-server-cli.md).

Чтобы просмотреть доступные значения часового пояса, выполните следующую команду.

```sql
SELECT name FROM mysql.time_zone_name;
```

### <a name="setting-the-global-level-time-zone"></a>Установка часового пояса глобального уровня

Часовой пояс глобального уровня можно задать с помощью команды [az mariadb server configuration set](/cli/azure/mariadb/server/configuration#az-mariadb-server-configuration-set).

Чтобы обновить параметр конфигурации **time\_zone** для сервера **mydemoserver.mariadb.database.azure.com** в группе ресурсов **myresourcegroup** и задать для него значение **US/Pacific**, используйте следующую команду.

```azurecli-interactive
az mariadb server configuration set --name time_zone --resource-group myresourcegroup --server mydemoserver --value "US/Pacific"
```

### <a name="setting-the-session-level-time-zone"></a>Настройка часового пояса уровня сеанса

Часовой пояс уровня сеанса можно задать, выполнив команду `SET time_zone` в командной строке MariaDB или MariaDB Workbench. В приведенном ниже примере задается часовой пояс **US/Pacific** (США, Тихоокеанский регион).  

```sql
SET time_zone = 'US/Pacific';
```

Описание [Функций даты и времени](https://mariadb.com/kb/en/library/date-time-functions/) см. в документации по MariaDB.

## <a name="next-steps"></a>Дальнейшие действия

- Настройка [параметров сервера на портале Azure](howto-server-parameters.md)
