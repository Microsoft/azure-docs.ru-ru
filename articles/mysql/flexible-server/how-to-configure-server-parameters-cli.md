---
title: Настройка параметров сервера — Azure CLI — гибкий сервер базы данных Azure для MySQL
description: В этой статье описывается, как настроить параметры службы в базе данных Azure для гибкого сервера MySQL с помощью служебной программы командной строки Azure CLI.
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.devlang: azurecli
ms.topic: how-to
ms.date: 11/10/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 58e7c024d6494aee745884997e42b527c51ab237
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94489545"
---
# <a name="configure-server-parameters-in-azure-database-for-mysql-flexible-server-using-the-azure-cli"></a>Настройка параметров сервера в базе данных Azure для гибкого сервера MySQL с помощью Azure CLI

> [!IMPORTANT] 
> Сейчас предоставляется общедоступная предварительная версия Гибкого сервера Базы данных Azure для MySQL.

Вы можете вывести список, просмотреть и обновить параметры для гибкого сервера базы данных Azure для MySQL с помощью Azure CLI, служебной программы командной строки Azure. При создании сервера для параметров сервера настраиваются значения по умолчанию и Рекомендуемые.  

В этой статье описывается, как вывести, отобразить и обновить параметры сервера с помощью Azure CLI.

>[!Note]
> Параметры сервера можно обновлять глобально на уровне сервера, использовать [Azure CLI](./how-to-configure-server-parameters-cli.md) или [портал Azure](./how-to-configure-server-parameters-portal.md)

## <a name="prerequisites"></a>Предварительные требования
Прежде чем приступить к выполнению этого руководства, необходимы следующие компоненты:
- [Гибкий сервер базы данных Azure для MySQL](quickstart-create-server-cli.md)
- Программа командной строки [Azure CLI](/cli/azure/install-azure-cli) (или используйте Azure Cloud Shell в браузере).

## <a name="list-server-parameters-for-azure-database-for-mysql-flexible-server"></a>Вывод списка параметров сервера для гибкого сервера базы данных Azure для MySQL
Чтобы получить список всех параметров на сервере и их значения, выполните команду [AZ MySQL гибкий-Server parameter list](/cli/azure/mysql/flexible-server/parameter) .

Вы можете вывести список параметров сервера для сервера **mydemoserver.MySQL.Database.Azure.com** в разделе **myresourcegroup** группы ресурсов.
```azurecli-interactive
az mysql flexible-server parameter list --resource-group myresourcegroup --server-name mydemoserver
```
Определение каждого из перечисленных параметров см. в разделе ссылок MySQL на странице[Server System Variables](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html) (Системные переменные сервера).

## <a name="show-server-parameter-details"></a>Отобразить сведения о параметрах сервера
Чтобы отобразить сведения о конкретном параметре для сервера, выполните команду [AZ MySQL гибкий-Server Parameter отобразить](/cli/azure/mysql/flexible-server/parameter) .

В этом примере показаны подробные сведения о параметре сервера **\_ \_ журнала** **mydemoserver.MySQL.Database.Azure.com** запросов для сервера в разделе **myresourcegroup** группы ресурсов.
```azurecli-interactive
az mysql flexible-server parameter show --name slow_query_log --resource-group myresourcegroup --server-name mydemoserver
```
## <a name="modify-a-server-parameter-value"></a>Изменение значения параметра сервера
Можно также изменить значение определенного параметра сервера, который обновляет базовое значение конфигурации для ядра сервера MySQL. Чтобы обновить параметр сервера, используйте команду [AZ MySQL гибкий-Server Parameter Set](/cli/azure/mysql/flexible-server/parameter) . 

Чтобы обновить параметр **сервера \_ \_ журнала запросов** **mydemoserver.MySQL.Database.Azure.com** сервера в разделе myresourcegroup группы ресурсов **.**
```azurecli-interactive
az mysql flexible-server parameter set --name slow_query_log --resource-group myresourcegroup --server-name mydemoserver --value ON
```
Если требуется сбросить значение параметра, опустите необязательный `--value` параметр, и служба применяет значение по умолчанию. В приведенном выше примере это может выглядеть следующим образом.
```azurecli-interactive
az mysql flexible-server parameter set --name slow_query_log --resource-group myresourcegroup --server-name mydemoserver
```
Этот код сбрасывает **\_ \_ Журнал задолгох запросов** до значения по умолчанию **Off**. 

## <a name="setting-non-modifiable-server-parameters"></a>Задание неизменяемых параметров сервера

Если параметр сервера, который требуется обновить, не является изменяемым, можно при необходимости задать параметр на уровне соединения с помощью `init_connect` . Это задает параметры сервера для каждого клиента, подключающегося к серверу. 

Обновите параметр сервера **init \_ connect** сервера **mydemoserver.MySQL.Database.Azure.com** в группе ресурсов **myresourcegroup** , чтобы задать значения, такие как кодировка.
```azurecli-interactive
az mysql flexible-server parameter set --name init_connect --resource-group myresourcegroup --server-name mydemoserver --value "SET character_set_client=utf8;SET character_set_database=utf8mb4;SET character_set_connection=latin1;SET character_set_results=latin1;"
```
>[!Note]
> `init_connect` можно использовать для изменения параметров, которым не нужны привилегии SUPER на уровне сеанса. Чтобы проверить, можно ли задать параметр с помощью `init_connect`, выполните команду `set session parameter_name=YOUR_DESIRED_VALUE;`. Если отображается сообщение об ошибке **Access denied; you need SUPER privileges(s)** (Отказано в доступе. Для выполнения этого действия необходимо иметь привилегии SUPER.), то параметр нельзя задать с помощью init_connect.

## <a name="working-with-the-time-zone-parameter"></a>Работа с параметром часового пояса

### <a name="populating-the-time-zone-tables"></a>Заполнение таблиц часовых поясов

Таблицы часовых поясов на сервере можно заполнить, вызвав хранимую процедуру `mysql.az_load_timezone` с помощью такого инструмента, как командная строка MySQL или MySQL Workbench.

> [!NOTE]
> Если вы используете команду `mysql.az_load_timezone` в MySQL Workbench, может потребоваться предварительно отключить режим безопасного обновления с помощью `SET SQL_SAFE_UPDATES=0;`.

```sql
CALL mysql.az_load_timezone();
```

> [!IMPORTANT]
> Необходимо перезапустить сервер, чтобы убедиться, что таблицы часовых поясов заполнены правильно.<!-- fIX me To restart the server, use the [Azure portal](howto-restart-server-portal.md) or [CLI](howto-restart-server-cli.md). -->

Чтобы просмотреть доступные значения часового пояса, выполните следующую команду.

```sql
SELECT name FROM mysql.time_zone_name;
```

### <a name="setting-the-global-level-time-zone"></a>Установка часового пояса глобального уровня

Часовой пояс глобального уровня можно задать с помощью команды [AZ MySQL гибкий-Server Parameter Set](/cli/azure/mysql/flexible-server/parameter) .

Следующая команда обновляет параметр сервера **часового \_ пояса** сервера **mydemoserver.MySQL.Database.Azure.com** в разделе Resource Group **myresourcegroup** to **US/Тихоокеанского**.

```azurecli-interactive
az mysql flexible-server parameter set --name time_zone --resource-group myresourcegroup --server-name mydemoserver --value "US/Pacific"
```

### <a name="setting-the-session-level-time-zone"></a>Настройка часового пояса уровня сеанса

Часовой пояс уровня сеанса можно задать, выполнив команду `SET time_zone` в командной строке MySQL или MySQL Workbench. В приведенном ниже примере задается часовой пояс **US/Pacific** (США, Тихоокеанский регион).  

```sql
SET time_zone = 'US/Pacific';
```

Описание [функций даты и времени](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_convert-tz) можно прочитать в документации по MySQL.


## <a name="next-steps"></a>Дальнейшие действия

- Настройка [параметров сервера на портале Azure](./how-to-configure-server-parameters-portal.md)
