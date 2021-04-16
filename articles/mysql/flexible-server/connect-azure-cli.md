---
title: Краткое руководство. Подключение к гибкому серверу Базы данных Azure для MySQL с помощью Azure CLI
description: В этом кратком руководстве показано несколько способов подключения к гибкому серверу Базы данных Azure для MySQL с помощью Azure CLI.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.custom: mvc
ms.topic: quickstart
ms.date: 03/01/2021
ms.openlocfilehash: d40dfa9c8a79625910414409ac3a6df7045c31f2
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106490919"
---
# <a name="quickstart-connect-and-query-with-azure-cli--with-azure-database-for-mysql---flexible-server"></a>Краткое руководство. Подключение и выполнение запроса к гибкому серверу Базы данных Azure для MySQL с помощью Azure CLI

> [!IMPORTANT]
> Сейчас предоставляется общедоступная предварительная версия Гибкого сервера Базы данных Azure для MySQL.

В этом кратком руководстве объясняется, как подключиться к гибкому серверу Базы данных Azure для MySQL с помощью команды ```az mysql flexible-server connect``` Azure CLI. С помощью этой команды можно проверить подключение к серверу базы данных и выполнить запросы непосредственно к серверу.  Вы можете также запустить команду в интерактивном режиме, чтобы выполнить несколько запросов.

## <a name="prerequisites"></a>Предварительные условия

- Учетная запись Azure. Если у вас ее нет, [получите бесплатную пробную версию](https://azure.microsoft.com/free/).
- Установите последнюю версию [Azure CLI](/cli/azure/install-azure-cli) (2.20.0 или более позднюю).
- Войдите с помощью Azure CLI, выполнив команду ```az login```. 
- Включите функцию сохраняемости параметров, выполнив команду ```az config param-persist on```. Функция сохраняемости параметров позволит использовать локальный контекст без необходимости повторения большого количества аргументов, таких как группа ресурсов, расположение и т. д.

## <a name="create-an-mysql-flexible-server"></a>Создание гибкого сервера MySQL

Первое, что мы создадим, — это управляемый сервер MySQL. В [Azure Cloud Shell](https://shell.azure.com/) выполните следующий скрипт и запишите **имя сервера**, **имя пользователя** и **пароль**, созданные с помощью этой команды.

```azurecli
az mysql flexible-server create --public-access <your-ip-address>
```

Чтобы настроить эту команду, вы можете указать дополнительные аргументы. Ознакомьтесь со всеми аргументами для команды [az mysql flexible-server create](/cli/azure/mysql/flexible-server#az_mysql_flexible_server_create).

## <a name="create-a-database"></a>Создание базы данных
Выполните следующую команду, чтобы создать базу данных **newdatabase**, если она еще не создана.

```azurecli
az mysql flexible-server db create -d newdatabase
```

## <a name="view-all-the-arguments"></a>Просмотр всех аргументов
Вы можете просмотреть все аргументы для этой команды с помощью аргумента ```--help```. 

```azurecli
az mysql flexible-server connect --help
```

## <a name="test-database-server-connection"></a>Проверка подключения к серверу базы данных
Выполните следующий скрипт, чтобы протестировать и проверить подключение к базе данных из окружения разработки.

```azurecli
az mysql flexible-server connect -n <servername> -u <username> -p <password> -d <databasename>
```

**Пример**.
```azurecli
az mysql flexible-server connect -n mysqldemoserver1 -u dbuser -p "dbpassword" -d newdatabase
```

Если подключение выполнено успешно, отобразятся следующие выходные данные:

```output
Command group 'mysql flexible-server' is in preview and under development. Reference and support levels: https://aka.ms/CLI_refstatus
Connecting to newdatabase database.
Successfully connected to mysqldemoserver1.
```
Если установить подключение не удалось, воспользуйтесь приведенными ниже решениями:
- Проверьте, открыт ли порт 3306 на клиентском компьютере.
- Проверьте, правильно ли указаны имя пользователя и пароль администратора сервера.
- Проверьте, настроено ли правило брандмауэра для клиентского компьютера.
- Если вы настроили сервер с закрытым доступом в виртуальной сети, убедитесь, что клиентский компьютер находится в той же виртуальной сети.

## <a name="run-single-query"></a>Выполнение одного запроса
Используйте следующую команду, чтобы выполнить один запрос с помощью аргумента ```--querytext```, ```-q```.

```azurecli
az mysql flexible-server connect -n <server-name> -u <username> -p "<password>" -d <database-name> --querytext "<query text>"
```

**Пример**.
```azurecli
az mysql flexible-server connect -n mysqldemoserver1 -u dbuser -p "dbpassword" -d newdatabase -q "select * from table1;" --output table
```

Результат выполнения команды будет таким:

```output
Command group 'mysql flexible-server' is in preview and under development. Reference and support levels: https://aka.ms/CLI_refstatus
Successfully connected to mysqldemoserver1.
Ran Database Query: 'select * from table1;'
Retrieving first 30 rows of query output, if applicable.
Closed the connection to mysqldemoserver1
Local context is turned on. Its information is saved in working directory C:\Users\sumuth. You can run `az local-context off` to turn it off.
Your preference of  are now saved to local context. To learn more, type in `az local-context --help`
Txt    Val
-----  -----
test   200
test   200
test   200
test   200
test   200
test   200
test   200
```

## <a name="run-multiple-queries-using-interactive-mode"></a>Выполнение нескольких запросов в интерактивном режиме
Вы можете выполнить несколько запросов в **интерактивном** режиме. Чтобы включить интерактивный режим, выполните следующую команду:

```azurecli
az mysql flexible-server connect -n <server-name> -u <username> -p <password> --interactive
```

**Пример**.
```azurecli
az mysql flexible-server connect -n mysqldemoserver1 -u dbuser -p "dbpassword" -d newdatabase --interactive
```

Отобразится приведенная ниже оболочка **MySQL**:

```bash
Command group 'mysql flexible-server' is in preview and under development. Reference and support levels: https://aka.ms/CLI_refstatus
Password:
mysql 5.7.29-log
mycli 1.22.2
Chat: https://gitter.im/dbcli/mycli
Mail: https://groups.google.com/forum/#!forum/mycli-users
Home: http://mycli.net
Thanks to the contributor - Martijn Engler
newdatabase> CREATE TABLE table1 (id int NOT NULL, val int,txt varchar(200));
Query OK, 0 rows affected
Time: 2.290s
newdatabase1> INSERT INTO table1 values (1,100,'text1');
Query OK, 1 row affected
Time: 0.199s
newdatabase1> SELECT * FROM table1;
+----+-----+-------+
| id | val | txt   |
+----+-----+-------+
| 1  | 100 | text1 |
+----+-----+-------+
1 row in set
Time: 0.149s
newdatabase>exit;
Goodbye!
Local context is turned on. Its information is saved in working directory C:\mydir. You can run `az local-context off` to turn it off.
Your preference of  are now saved to local context. To learn more, type in `az local-context --help`
```


## <a name="next-steps"></a>Next Steps

> [!div class="nextstepaction"]
* [Подключение к базе данных Azure для MySQL — гибкому серверу с зашифрованными подключениями](how-to-connect-tls-ssl.md)
* [Управление сервером](./how-to-manage-server-cli.md)

