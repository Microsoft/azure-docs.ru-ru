---
title: Руководство по разработке базы данных в службе "База данных Azure для PostgreSQL — отдельный сервер" с помощью Azure CLI
description: Это руководство содержит сведения о создании и настройке первой базы данных в службе "База данных Azure для PostgreSQL — отдельный сервер", а также о выполнении запросов к ней с помощью Azure CLI.
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.custom: mvc, devx-track-azurecli
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 06/25/2019
ms.openlocfilehash: 478fc175304a24536d28807915f4a4ec05a7158e
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105605881"
---
# <a name="tutorial-design-an-azure-database-for-postgresql---single-server-using-azure-cli"></a>Руководство по Разработка базы данных в службе "База данных Azure для PostgreSQL — отдельный сервер" с помощью Azure CLI 
Из этого руководства вы узнаете, как с помощью Azure CLI (интерфейса командной строки) и других служебных программ выполнять следующие операции:
> [!div class="checklist"]
> * Создание сервера Базы данных Azure для PostgreSQL
> * настройка брандмауэра сервера;
> * использование служебной программы [**psql**](https://www.postgresql.org/docs/9.6/static/app-psql.html) для создания базы данных;
> * Загрузка примера данных
> * Данные запросов
> * Обновление данных
> * восстановление данных.

Вы можете использовать Azure Cloud Shell в браузере или [установить Azure CLI]( /cli/azure/install-azure-cli) на компьютере, чтобы запускать команды, присутствующие в этом руководстве.

## <a name="prerequisites"></a>Предварительные требования
Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

Если вы решили установить и использовать интерфейс командной строки локально, для работы с этой статьей вам понадобится Azure CLI 2.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0]( /cli/azure/install-azure-cli). 

Если вы используете несколько подписок, выберите соответствующую подписку, в которой находится ресурс либо в которой за него взимается плата. Выберите конкретный идентификатор подписки вашей учетной записи, выполнив команду [az account set](/cli/azure/account).
```azurecli-interactive
az account set --subscription 00000000-0000-0000-0000-000000000000
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов
Создайте [группу ресурсов Azure](../azure-resource-manager/management/overview.md) с помощью команды [az group create](/cli/azure/group). Группа ресурсов — это логический контейнер, в котором ресурсы Azure развертываются и администрируются как группа. В следующем примере создается группа ресурсов с именем `myresourcegroup` в расположении именем `westus`.
```azurecli-interactive
az group create --name myresourcegroup --location westus
```

## <a name="create-an-azure-database-for-postgresql-server"></a>Создание сервера Базы данных Azure для PostgreSQL
Создайте [сервер базы данных Azure для PostgreSQL](overview.md), выполнив команду [az postgres server create](/cli/azure/postgres/server). Сервер содержит группу баз данных, которыми можно управлять как группой. 

В указанном ниже примере в группе ресурсов `myresourcegroup` создается сервер `mydemoserver` с именем для входа администратора сервера `myadmin`. Имя сервера сопоставляется с DNS-именем, и поэтому оно должно быть глобально уникальным в рамках Azure. Замените `<server_admin_password>` собственным значением. Это сервер общего назначения 5-го поколения с 2 виртуальными ядрами.
```azurecli-interactive
az postgres server create --resource-group myresourcegroup --name mydemoserver --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen5_2 --version 9.6
```
Значение параметра sku-name соответствует соглашению {ценовая категория}\_{поколение вычислительных ресурсов}\_{количество виртуальных ядер}, как показано в примерах ниже:
+ `--sku-name B_Gen5_2` — ценовая категория "Базовый", поколение 5, 2 виртуальных ядра;
+ `--sku-name GP_Gen5_32` — "Общего назначения", поколение 5, 32 виртуальных ядра;
+ `--sku-name MO_Gen5_2` — "Оптимизированная для операций в памяти", поколение 5, 2 виртуальных ядра.

Допустимые значения для каждого региона и каждого уровня указаны в документации по [ценовым категориям](./concepts-pricing-tiers.md).

> [!IMPORTANT]
> Указанные здесь учетные данные и пароль администратора сервера понадобятся позже в этом руководстве, чтобы войти на сервер и в его базу данных. Запомните или запишите эту информацию для последующего использования.

По умолчанию на сервере создается база данных **postgres**. База данных [postgres](https://www.postgresql.org/docs/9.6/static/app-initdb.html) — это база данных по умолчанию, предназначенная для использования пользователями, служебными программами и сторонними приложениями. 


## <a name="configure-a-server-level-firewall-rule"></a>Настройка правила брандмауэра на уровне сервера

Создайте правило брандмауэра на уровне сервера Azure PostgreSQL, выполнив команду [az postgres server firewall-rule create](/cli/azure/postgres/server/firewall-rule). Правило брандмауэра на уровне сервера позволяет внешним приложениям, таким как [psql](https://www.postgresql.org/docs/9.2/static/app-psql.html) или [PgAdmin](https://www.pgadmin.org/), подключаться к серверу через брандмауэр службы Azure PostgreSQL. 

Вы можете задать правило брандмауэра, охватывающее диапазон IP-адресов, чтобы иметь возможность подключаться из сети. В приведенном ниже примере используется команда [az postgres server firewall-rule create](/cli/azure/postgres/server/firewall-rule) для создания правила брандмауэра `AllowMyIP`, позволяющего подключаться с одного IP-адреса.

```azurecli-interactive
az postgres server firewall-rule create --resource-group myresourcegroup --server mydemoserver --name AllowMyIP --start-ip-address 192.168.0.1 --end-ip-address 192.168.0.1
```

Чтобы доступ к серверу PostgreSQL Azure был ограничен только вашей сетью, можно настроить правило брандмауэра таким образом, чтобы оно охватывало диапазон IP-адресов только вашей корпоративной сети.

> [!NOTE]
> Сервер PostgreSQL Azure обменивается данными через порт 5432. При попытках подключения из корпоративной сети может оказаться, что исходящий трафик через порт 5432 запрещен сетевым брандмауэром. Чтобы вы могли подключиться к серверу Базы данных SQL Azure, ваш ИТ-отдел должен открыть порт 5432.
>

## <a name="get-the-connection-information"></a>Получение сведений о подключении

Чтобы подключиться к серверу, необходимо указать сведения об узле и учетные данные для доступа.
```azurecli-interactive
az postgres server show --resource-group myresourcegroup --name mydemoserver
```

Результаты выводятся в формате JSON. Запишите **administratorLogin** и **fullyQualifiedDomainName**.
```json
{
  "administratorLogin": "myadmin",
  "earliestRestoreDate": null,
  "fullyQualifiedDomainName": "mydemoserver.postgres.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforPostgreSQL/servers/mydemoserver",
  "location": "westus",
  "name": "mydemoserver",
  "resourceGroup": "myresourcegroup",
  "sku": {
    "capacity": 2,
    "family": "Gen5",
    "name": "GP_Gen5_2",
    "size": null,
    "tier": "GeneralPurpose"
  },
  "sslEnforcement": "Enabled",
  "storageProfile": {
    "backupRetentionDays": 7,
    "geoRedundantBackup": "Disabled",
    "storageMb": 5120
  },
  "tags": null,
  "type": "Microsoft.DBforPostgreSQL/servers",
  "userVisibleState": "Ready",
  "version": "9.6"

}
```

## <a name="connect-to-azure-database-for-postgresql-database-using-psql"></a>Подключение к базе данных Azure для PostgreSQL с помощью psql
Если на клиентском компьютере установлено PostgreSQL, вы можете использовать локальный экземпляр [psql](https://www.postgresql.org/docs/9.6/static/app-psql.html) или консоль облачной службы Azure, чтобы подключиться к серверу Azure PostgreSQL. Теперь подключимся к серверу базы данных Azure для PostgreSQL с помощью служебной программы командной строки psql.

1. Чтобы подключиться к базе данных Azure для базы данных PostgreSQL, выполните следующую команду psql:
   ```
   psql --host=<servername> --port=<port> --username=<user@servername> --dbname=<dbname>
   ```

   Например, следующая команда устанавливает подключение к базе данных по умолчанию **postgres** на сервере PostgreSQL **mydemoserver.postgres.database.azure.com**, используя учетные данные для доступа. Введите `<server_admin_password>`, указанный при появлении запроса на ввод пароля.
  
   ```
   psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin@mydemoserver --dbname=postgres
   ```

   > [!TIP]
   > Если вы предпочитаете использовать URL-путь для подключения к Postgres, закодируйте с помощью URL-адреса знак @ в имени пользователя с использованием `%40`. Например, строка подключения для psql будет выглядеть так:
   > ```
   > psql postgresql://myadmin%40mydemoserver@mydemoserver.postgres.database.azure.com:5432/postgres
   > ```

2. Подключившись к серверу, создайте пустую базу данных с помощью командной строки:
   ```sql
   CREATE DATABASE mypgsqldb;
   ```

3. Чтобы подключиться к созданной базе данных **mypgsqldb**, выполните следующую команду в командной строке:
   ```sql
   \c mypgsqldb
   ```

## <a name="create-tables-in-the-database"></a>Создание таблиц в базе данных
Теперь, когда вы знаете, как подключиться к базе данных Azure для PostgreSQL, можно выполнить некоторые основные задачи.

Сначала создайте таблицу и заполните ее некоторыми данными. Например, создадим таблицу, с помощью которой можно отслеживать данные инвентаризации:
```sql
CREATE TABLE inventory (
    id serial PRIMARY KEY, 
    name VARCHAR(50), 
    quantity INTEGER
);
```

Вы можете просмотреть созданную таблицу в списке таблиц, введя:
```sql
\dt
```

## <a name="load-data-into-the-table"></a>Загрузка данных в таблицу
Теперь, когда таблица создана, вставьте в нее некоторые данные. Чтобы вставить некоторые строки данных, в открытом окне командной строки выполните следующий запрос:
```sql
INSERT INTO inventory (id, name, quantity) VALUES (1, 'banana', 150); 
INSERT INTO inventory (id, name, quantity) VALUES (2, 'orange', 154);
```

Итак, в созданной ранее таблице добавлено две строки демонстрационных данных.

## <a name="query-and-update-the-data-in-the-tables"></a>Запрос и обновление данных в таблицах
Чтобы извлечь сведения из таблицы inventory, выполните приведенный ниже запрос: 
```sql
SELECT * FROM inventory;
```

Вы можете также обновить данные в таблице inventory:
```sql
UPDATE inventory SET quantity = 200 WHERE name = 'banana';
```

При извлечении данных вы увидите обновленные значения:
```sql
SELECT * FROM inventory;
```

## <a name="restore-a-database-to-a-previous-point-in-time"></a>Восстановление базы данных до предыдущей точки во времени
Представьте, что вы случайно удалили таблицу. Восстановить ее будет не просто. База данных Azure для PostgreSQL позволяет вернуть состояние до любой точки времени при наличии резервных копий сервера (определяется по настроенному сроку хранения резервной копии) и восстановить данные до этой точки на новом сервере. Вы можете восстановить удаленные данные с помощью нового сервера. 

Указанная ниже команда позволяет восстановить демонстрационный сервер до точки во времени, когда таблица еще не была создана:
```azurecli-interactive
az postgres server restore --resource-group myresourcegroup --name mydemoserver-restored --restore-point-in-time 2017-04-13T13:59:00Z --source-server mydemoserver
```

Для команды `az postgres server restore` необходимо настроить следующие параметры:

| Параметр | Рекомендуемое значение | Описание  |
| --- | --- | --- |
| resource-group |  myresourcegroup |  Группа ресурсов, в которой находится исходный сервер.  |
| name | mydemoserver-restored | Имя нового сервера, созданного командой restore. |
| restore-point-in-time | 2017-04-13T13:59:00Z | Выберите точку во времени, до которой необходимо выполнить восстановление. Значения даты и времени должны находиться в пределах срока хранения резервной копии исходного сервера. Используйте формат даты и времени ISO8601. Например, вы можете использовать свой местный часовой пояс, например `2017-04-13T05:59:00-08:00`, или использовать формат UTC Zulu `2017-04-13T13:59:00Z`. |
| source-server | mydemoserver | Имя или идентификатор исходного сервера, с которого необходимо выполнить восстановление. |

При восстановлении сервера до определенной точки во времени создается новый сервер путем копирования той точки во времени исходного сервера, которую вы задали. Значения расположения и ценовой категории для восстановленного сервера совпадают со значениями исходного сервера.

Команда выполняется в синхронном режиме и будет возвращена после восстановления сервера. После завершения восстановления найдите созданный сервер. Убедитесь, что данные восстановлены надлежащим образом.

## <a name="clean-up-resources"></a>Очистка ресурсов

На предыдущих шагах вы создали ресурсы Azure в группе серверов. Если эти ресурсы вам больше не нужны, удалите группу серверов. Нажмите кнопку *Удалить* на странице *Обзор*, чтобы удалить группу серверов. При появлении запроса на всплывающей странице проверьте имя группы серверов и нажмите кнопку *Удалить*.


## <a name="next-steps"></a>Дальнейшие действия
Из этого руководства вы узнали, как с помощью Azure CLI (интерфейса командной строки) и других служебных программ выполнить следующие операции:
> [!div class="checklist"]
> * Создание сервера Базы данных Azure для PostgreSQL
> * настройка брандмауэра сервера;
> * использование служебной программы **psql** для создания базы данных;
> * Загрузка примера данных
> * Данные запросов
> * Обновление данных
> * восстановление данных.

> [!div class="nextstepaction"]
> [Руководство по проектированию службы "База данных Azure для PostgreSQL" с помощью портала Azure](tutorial-design-database-using-azure-portal.md)
