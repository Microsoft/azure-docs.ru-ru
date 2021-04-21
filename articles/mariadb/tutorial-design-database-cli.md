---
title: Руководство по проектированию Базы данных Azure для MariaDB с помощью Azure CLI
description: В этом руководстве описано, как создать и администрировать сервер и Базу данных Azure для MariaDB и с помощью Azure CLI 2.0 из командной строки.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 3/18/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 6b3a65d6b7fbf98d53b756b7f089e06f8a359ab3
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107789957"
---
# <a name="tutorial-design-an-azure-database-for-mariadb-using-azure-cli"></a>Руководство по Проектирование Базы данных Azure для MariaDB с помощью Azure CLI

База данных Azure для MariaDB — это служба реляционной базы данных в среде Microsoft Cloud на основе ядра СУБД MariaDB Community Edition. Из этого руководства вы узнаете, как с помощью Azure CLI (интерфейса командной строки) и других служебных программ выполнять следующие операции:

> [!div class="checklist"]
> * создание Базы данных Azure для MariaDB;
> * настройка брандмауэра сервера;
> * использование [программы командной строки MySQL](https://dev.mysql.com/doc/refman/5.7/en/mysql.html) для создания базы данных.
> * Загрузка примера данных
> * Данные запросов
> * Обновление данных
> * восстановление данных.

Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/) Azure, прежде чем начинать работу.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена. 

Если вы используете несколько подписок, выберите соответствующую подписку, в которой находится ресурс либо в которой за него взимается плата. Выберите конкретный идентификатор подписки вашей учетной записи, выполнив команду [az account set](/cli/azure/account#az_account_set).
```azurecli-interactive
az account set --subscription 00000000-0000-0000-0000-000000000000
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов
Создайте [группу ресурсов Azure](../azure-resource-manager/management/overview.md), выполнив команду [az group create](/cli/azure/group#az_group_create). Группа ресурсов — это логический контейнер, в котором ресурсы Azure развертываются и администрируются как группа.

В следующем примере создается группа ресурсов с именем `myresourcegroup` в расположении именем `westus`.

```azurecli-interactive
az group create --name myresourcegroup --location westus
```

## <a name="create-an-azure-database-for-mariadb-server"></a>Создание сервера Базы данных Azure для MariaDB
Создайте сервер базы данных Azure для MariaDB с помощью команды `az mariadb server create`. Сервер может управлять несколькими базами данных. Как правило, для каждого проекта и для каждого пользователя используется отдельная база данных.

В следующем примере создается сервер Базы данных Azure для MariaDB сервера, расположенный в `westus` в группе ресурсов `myresourcegroup` с именем `mydemoserver`. Для сервера указано имя администратора для входа `myadmin`. Это сервер общего назначения 5-го поколения с 2 виртуальными ядрами. Замените `<server_admin_password>` собственным значением.

```azurecli-interactive
az mariadb server create --resource-group myresourcegroup --name mydemoserver --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen5_2 --version 10.2
```
Значение параметра sku-name соответствует соглашению {ценовая категория}\_{поколение вычислительных ресурсов}\_{количество виртуальных ядер}, как показано в примерах ниже:
+ `--sku-name B_Gen5_4` — ценовая категория "Базовый", поколение 5, 4 виртуальных ядра;
+ `--sku-name GP_Gen5_32` — "Общего назначения", поколение 5, 32 виртуальных ядра;
+ `--sku-name MO_Gen5_2` — "Оптимизированная для операций в памяти", поколение 5, 2 виртуальных ядра.

Допустимые значения для каждого региона и каждого уровня указаны в документации по [ценовым категориям](./concepts-pricing-tiers.md).

> [!IMPORTANT]
> Указанные здесь учетные данные и пароль администратора сервера понадобятся позже в этом руководстве, чтобы войти на сервер и в его базу данных. Запомните или запишите эту информацию для последующего использования.


## <a name="configure-firewall-rule"></a>Настройка правила брандмауэра
Создайте правило брандмауэра на уровне сервера Базы данных Azure для MariaDB с помощью команды `az mariadb server firewall-rule create`. Правило брандмауэра на уровне сервера позволяет внешним приложениям, таким как программа командной строки **mysql** или MySQL Workbench, подключаться к серверу через брандмауэр службы Azure MariaDB.

В приведенном ниже примере создается правило брандмауэра с именем `AllowMyIP`, которое разрешает подключения с определенного IP-адреса — 192.168.0.1. Подставьте IP-адрес или диапазон IP-адресов, которые соответствуют расположению, из которого будет устанавливаться подключение.

```azurecli-interactive
az mariadb server firewall-rule create --resource-group myresourcegroup --server mydemoserver --name AllowMyIP --start-ip-address 192.168.0.1 --end-ip-address 192.168.0.1
```

## <a name="get-the-connection-information"></a>Получение сведений о подключении

Чтобы подключиться к серверу, необходимо указать сведения об узле и учетные данные для доступа.
```azurecli-interactive
az mariadb server show --resource-group myresourcegroup --name mydemoserver
```

Результаты выводятся в формате JSON. Запишите значения **fullyQualifiedDomainName** и **administratorLogin**.
```json
{
  "administratorLogin": "myadmin",
  "administratorLoginPassword": null,
  "fullyQualifiedDomainName": "mydemoserver.mariadb.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforMariaDB/servers/mydemoserver",
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
  "type": "Microsoft.DBforMariaDB/servers",
  "userVisibleState": "Ready",
  "version": "10.2"
}
```

## <a name="connect-to-the-server-using-mysql"></a>Подключение к серверу с помощью MySQL
Используйте [программу командной строки MySQL](https://dev.mysql.com/doc/refman/5.7/en/mysql.html), чтобы подключиться к серверу Базы данных Azure для MariaDB. В этом примере используется следующая команда:
```cmd
mysql -h mydemoserver.database.windows.net -u myadmin@mydemoserver -p
```

## <a name="create-a-blank-database"></a>Создание пустой базы данных
Подключившись к серверу, создайте пустую базу данных.
```sql
mysql> CREATE DATABASE mysampledb;
```

Выполните следующую команду в командной строке, чтобы подключиться к созданной базе данных:
```sql
mysql> USE mysampledb;
```

## <a name="create-tables-in-the-database"></a>Создание таблиц в базе данных
Теперь, когда вы знаете, как подключиться к базе данных Azure для MariaDB, выполните основные задачи.

Сначала создайте таблицу и заполните ее некоторыми данными. Давайте создадим таблицу, в которой хранятся данные инвентаризации.
```sql
CREATE TABLE inventory (
    id serial PRIMARY KEY, 
    name VARCHAR(50), 
    quantity INTEGER
);
```

## <a name="load-data-into-the-tables"></a>Загрузка данных в таблицу
Теперь, когда таблица создана, мы можем вставить в нее данные. Чтобы вставить некоторые строки данных, в открытом окне командной строки выполните следующий запрос:
```sql
INSERT INTO inventory (id, name, quantity) VALUES (1, 'banana', 150); 
INSERT INTO inventory (id, name, quantity) VALUES (2, 'orange', 154);
```

Итак, в созданной ранее таблице содержится две строки данных.

## <a name="query-and-update-the-data-in-the-tables"></a>Запрос и обновление данных в таблицах
Чтобы извлечь сведения из таблицы базы данных, выполните приведенный ниже запрос.
```sql
SELECT * FROM inventory;
```

Вы можете также обновить данные в таблицах, выполнив следующую команду:
```sql
UPDATE inventory SET quantity = 200 WHERE name = 'banana';
```

При извлечении данных строка будет обновляться соответствующим образом.
```sql
SELECT * FROM inventory;
```

## <a name="restore-a-database-to-a-previous-point-in-time"></a>Восстановление базы данных до предыдущей точки во времени
Представьте, что вы случайно удалили таблицу. Восстановить ее будет не просто. База данных Azure для MariaDB позволяет вернуться в любой момент времени в течение последних 35 дней и восстановить данные на определенный момент времени на новом сервере. Вы можете восстановить удаленные данные с помощью нового сервера. Указанные ниже шаги позволяют восстановить сервер до точки во времени, когда была создана таблица.

Чтобы восстановить данные, вам потребуются следующие сведения:

- Точка восстановления. Выберите время до того момента, когда был изменен сервер. Значение точки восстановления должно быть не меньше значения самой старой резервной копии исходной базы данных.
- Целевой сервер. Укажите новое имя сервера, который нужно восстановить.
- Исходный сервер. Укажите имя сервера, из которого требуется выполнить восстановление.
- Расположение. Вы не сможете выбрать регион, по умолчанию он совпадает с исходным сервером.

```azurecli-interactive
az mariadb server restore --resource-group myresourcegroup --name mydemoserver-restored --restore-point-in-time "2017-05-4 03:10" --source-server-name mydemoserver
```

Для команды `az mariadb server restore` необходимо настроить следующие параметры:

| Параметр | Рекомендуемое значение | Описание  |
| --- | --- | --- |
| resource-group |  myresourcegroup |  Группа ресурсов, в которой находится исходный сервер.  |
| name | mydemoserver-restored | Имя нового сервера, созданного командой restore. |
| restore-point-in-time | 2017-04-13T13:59:00Z | Выберите точку во времени, до которой необходимо выполнить восстановление. Значения даты и времени должны находиться в пределах срока хранения резервной копии исходного сервера. Используйте формат даты и времени ISO8601. Например, вы можете использовать свой местный часовой пояс, например `2017-04-13T05:59:00-08:00`, или использовать формат UTC Zulu `2017-04-13T13:59:00Z`. |
| source-server | mydemoserver | Имя или идентификатор исходного сервера, с которого необходимо выполнить восстановление. |

При восстановлении сервера до определенной точки во времени создается новый сервер путем копирования той точки во времени исходного сервера, которую вы задали. Значения расположения и ценовой категории для восстановленного сервера совпадают со значениями исходного сервера.

Команда выполняется в синхронном режиме и будет возвращена после восстановления сервера. После завершения восстановления найдите созданный сервер. Убедитесь, что данные восстановлены надлежащим образом.

## <a name="next-steps"></a>Дальнейшие действия
Из этого руководства вы узнали, как выполнять следующие операции:
> [!div class="checklist"]
> * Создание сервера Базы данных Azure для MariaDB
> * настройка брандмауэра сервера;
> * использование [программы командной строки MySQL](https://dev.mysql.com/doc/refman/5.7/en/mysql.html) для создания базы данных.
> * Загрузка примера данных
> * Данные запросов
> * Обновление данных
> * восстановление данных.