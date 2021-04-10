---
title: Краткое руководство. Подключение к гибкому серверу Базы данных Azure PostgreSQL с помощью Azure CLI
description: В этом кратком руководстве показано несколько способов подключения к гибкому серверу Базы данных Azure PostgreSQL с помощью Azure CLI.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.custom: mvc
ms.topic: quickstart
ms.date: 03/06/2021
ms.openlocfilehash: f4eec89aadee1966271286b9280916af973e4b1c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102614349"
---
# <a name="quickstart-connect-and-query-with-azure-cli--with-azure-database-for-postgresql---flexible-server"></a>Краткое руководство. Подключение и выполнение запроса к гибкому серверу Базы данных Azure PostgreSQL с помощью Azure CLI

> [!IMPORTANT]
> Гибкий сервер Базы данных Azure PostgreSQL сейчас находится на этапе общедоступной предварительной версии.

В этом кратком руководстве объясняется, как подключиться к гибкому серверу Базы данных Azure PostgreSQL с помощью команды ```az postgres flexible-server connect``` Azure CLI. С помощью этой команды можно проверить подключение к серверу базы данных и выполнить запросы. Вы также можете выполнить несколько запросов в интерактивном режиме. 

## <a name="prerequisites"></a>Предварительные требования
- Учетная запись Azure. Если у вас ее нет, [получите бесплатную пробную версию](https://azure.microsoft.com/free/).
- Установите последнюю версию [Azure CLI](/cli/azure/install-azure-cli) (2.20.0 или более позднюю).
- Войдите с помощью Azure CLI, выполнив команду ```az login```. 
- Включите функцию сохраняемости параметров, выполнив команду ```az config param-persist on```. Функция сохраняемости параметров позволит использовать локальный контекст без необходимости повторения большого количества аргументов, таких как группа ресурсов или расположение.

## <a name="create-an-postgresql-flexible-server"></a>Создание гибкого сервера PostgreSQL

Прежде всего мы создадим управляемый сервер PostgreSQL. В [Azure Cloud Shell](https://shell.azure.com/) выполните следующий скрипт и запишите **имя сервера**, **имя пользователя** и **пароль**, созданные с помощью этой команды.

```azurecli
az postgres flexible-server create --public-access <your-ip-address>
```
Чтобы настроить эту команду, вы можете указать дополнительные аргументы. Ознакомьтесь со всеми аргументами для команды [az postgres flexible-server create](/cli/azure/postgres/flexible-server#az_postgres_flexible_server_create).

## <a name="view-all-the-arguments"></a>Просмотр всех аргументов
Вы можете просмотреть все аргументы для этой команды с помощью аргумента ```--help```. 

```azurecli
az postgresql flexible-server connect --help
```

## <a name="test-database-server-connection"></a>Проверка подключения к серверу базы данных
Вы можете протестировать и проверить подключение к базе данных из окружения разработки, выполнив следующую команду:

```azurecli
az postgres flexible-server connect -n <servername> -u <username> -p "<password>" -d <databasename>
```
**Пример**. 
```azurecli
az postgres flexible-server connect -n postgresdemoserver -u dbuser -p "dbpassword" -d postgres
```
Если подключение установлено успешно, отобразятся выходные данные.
```output
Command group 'postgres flexible-server' is in preview and under development. Reference and support levels: https://aka.ms/CLI_refstatus
Successfully connected to postgresdemoserver.
Local context is turned on. Its information is saved in working directory C:\mydir. You can run `az local-context off` to turn it off.
Your preference of  are now saved to local context. To learn more, type in `az local-context --help`
```

Если установить подключение не удалось, воспользуйтесь приведенными ниже решениями:
- Проверьте, открыт ли порт 5432 на клиентском компьютере.
- Проверьте, правильно ли указаны имя пользователя и пароль администратора сервера.
- Проверьте, настроено ли правило брандмауэра для клиентского компьютера.
- Если вы настроили сервер с закрытым доступом в виртуальной сети, убедитесь, что клиентский компьютер находится в той же виртуальной сети.

## <a name="run-single-query"></a>Выполнение одного запроса
Чтобы выполнить один запрос, используйте следующую команду с аргументом ```--querytext```, ```-q```.

```azurecli
az postgres flexible-server connect -n <server-name> -u <username> -p "<password>" -d <database-name> -q "<query-text>"
```

**Пример**. 
```azurecli
az postgresql flexible-server connect -n postgresdemoserver -u dbuser -p "dbpassword" -d flexibleserverdb -q "select * from table1;" --output table
```

Результат выполнения команды будет таким:

```output
Command group 'postgres flexible-server' is in preview and under development. Reference and support levels: https://aka.ms/CLI_refstatus
Successfully connected to postgresdemoserver.
Ran Database Query: 'select * from table1;'
Retrieving first 30 rows of query output, if applicable.
Closed the connection postgresdemoserver.
Local context is turned on. Its information is saved in working directory C:\mydir. You can run `az local-context off` to turn it off.
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
az postgres flexible-server connect -n <servername> -u <username> -p "<password>" -d <databasename>
```

**Пример**.

```azurecli
az postgresql flexible-server connect -n postgresdemoserver -u dbuser -p "dbpassword" -d flexibleserverdb --interactive
```

Отобразится приведенная ниже оболочка **PSQL**:

```bash
Command group 'postgres flexible-server' is in preview and under development. Reference and support levels: https://aka.ms/CLI_refstatus
Password for earthyTurtle7:
Server: PostgreSQL 12.5
Version: 3.0.0
Chat: https://gitter.im/dbcli/pgcli
Home: http://pgcli.com
postgres> create database pollsdb;
CREATE DATABASE
Time: 0.308s
postgres> exit
Goodbye!
Local context is turned on. Its information is saved in working directory C:\sunitha. You can run `az local-context off` to turn it off.
Your preference of  are now saved to local context. To learn more, type in `az local-context --help`
```


## <a name="next-steps"></a>Next Steps

> [!div class="nextstepaction"]
> [Управление сервером](./how-to-manage-server-cli.md)
