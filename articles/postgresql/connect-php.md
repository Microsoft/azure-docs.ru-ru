---
title: Краткое руководство. Подключение к службе "База данных Azure для PostgreSQL — отдельный сервер" с помощью PHP
description: В этом кратком руководстве представлен пример кода PHP, который можно использовать для подключения к службе "База данных Azure для PostgreSQL — отдельный сервер" и запроса данных из нее.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.custom: mvc
ms.devlang: php
ms.topic: quickstart
ms.date: 2/28/2018
ms.openlocfilehash: 200fdd126e2ed95804f81c1dd36804ecc6c61d85
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98019692"
---
# <a name="quickstart-use-php-to-connect-and-query-data-in-azure-database-for-postgresql---single-server"></a>Краткое руководство. Подключение к службе "База данных Azure для PostgreSQL — отдельный сервер" и выполнение запроса данных с помощью PHP

В этом кратком руководстве объясняется, как подключиться к базе данных Azure для PostgreSQL с помощью приложения [PHP](https://secure.php.net/manual/intro-whatis.php). Здесь также показано, как использовать инструкции SQL для запроса, вставки, обновления и удаления данных в базе данных. В этой статье предполагается, что у вас уже есть опыт разработки на PHP и вы только начали работу с базой данных Azure для PostgreSQL.

## <a name="prerequisites"></a>Предварительные требования
В качестве отправной точки в этом кратком руководстве используются ресурсы, созданные в соответствии со следующими материалами:
- [Создание базы данных — портал](quickstart-create-server-database-portal.md)
- [Создание базы данных с помощью Azure CLI](quickstart-create-server-database-azure-cli.md)

## <a name="install-php"></a>Установка PHP
Установите PHP на своем сервере или создайте [веб-приложение](../app-service/overview.md) Azure с PHP.

### <a name="windows"></a>Windows
- Скачайте [PHP 7.1.4 (x64) непотокобезопасной версии](https://windows.php.net/download#php-7.1).
- Установите PHP и выполните настройку согласно инструкциям в [руководстве по PHP](https://secure.php.net/manual/install.windows.php).
- В коде используется класс **pgsql** (ext/php_pgsql.dll), который устанавливается с PHP. 
- Включите расширение **pgsql**, изменив файл конфигурации php.ini. Как правило, он находится в папке `C:\Program Files\PHP\v7.1\php.ini`. Файл конфигурации должен содержать строку с текстом `extension=php_pgsql.so`. Если текст не отображается, добавьте его и сохраните файл. Если текст есть, но закомментирован префиксом в виде точки с запятой, раскомментируйте его, удалив точку с запятой.

### <a name="linux-ubuntu"></a>Linux (Ubuntu)
- Скачайте [PHP 7.1.4 (x64) непотокобезопасной версии](https://secure.php.net/downloads.php). 
- Установите PHP и выполните настройку согласно инструкциям в [руководстве по PHP](https://secure.php.net/manual/install.unix.php).
- В коде используется класс **pgsql** (php_pgsql.so). Установите его, выполнив команду `sudo apt-get install php-pgsql`.
- Включите расширение **pgsql**, изменив файл конфигурации `/etc/php/7.0/mods-available/pgsql.ini`. Файл конфигурации должен содержать строку с текстом `extension=php_pgsql.so`. Если текст не отображается, добавьте его и сохраните файл. Если текст есть, но закомментирован префиксом в виде точки с запятой, раскомментируйте его, удалив точку с запятой.

### <a name="macos"></a>MacOS
- Скачайте [PHP версии 7.1.4](https://secure.php.net/downloads.php).
- Установите PHP и выполните настройку согласно инструкциям в [руководстве по PHP](https://secure.php.net/manual/install.macosx.php).

## <a name="get-connection-information"></a>Получение сведений о подключении
Получите сведения, необходимые для подключения к базе данных Azure.для PostgreSQL. Вам потребуется полное имя сервера и учетные данные для входа.

1. Войдите на [портал Azure](https://portal.azure.com/).
2. В меню слева на портале Azure щелкните **Все ресурсы** и выполните поиск по имени созданного сервера (например, **mydemoserver**).
3. Щелкните имя сервера.
4. Запишите **имя сервера** и **имя для входа администратора сервера** с панели сервера **Обзор**. Если вы забыли свой пароль, можно также сбросить пароль с помощью этой панели.
 :::image type="content" source="./media/connect-php/1-connection-string.png" alt-text="Имя сервера службы &quot;База данных Azure для PostgreSQL&quot;":::

## <a name="connect-and-create-a-table"></a>Подключение и создание таблицы
Используйте приведенный ниже код для подключения и создайте таблицу с помощью инструкции SQL **CREATE TABLE**. Добавьте строки в таблицу, применив инструкцию SQL **INSERT INTO**.

Код вызывает метод [pg_connect()](https://secure.php.net/manual/en/function.pg-connect.php), чтобы подключиться к базе данных Azure для PostgreSQL. Затем он вызывает метод [pg_query()](https://secure.php.net/manual/en/function.pg-query.php) несколько раз, чтобы выполнить несколько команд, и метод [pg_last_error()](https://secure.php.net/manual/en/function.pg-last-error.php), чтобы проверить сведения, если каждое выполнение завершилось ошибкой. После этого вызывается метод [mysqli_close](https://secure.php.net/manual/en/function.pg-close.php), чтобы разорвать подключение.

Замените значения параметров `$host`, `$database`, `$user` и `$password` своими значениями. 

```php
<?php
    // Initialize connection variables.
    $host = "mydemoserver.postgres.database.azure.com";
    $database = "mypgsqldb";
    $user = "mylogin@mydemoserver";
    $password = "<server_admin_password>";

    // Initialize connection object.
    $connection = pg_connect("host=$host dbname=$database user=$user password=$password") 
        or die("Failed to create connection to database: ". pg_last_error(). "<br/>");
    print "Successfully created connection to database.<br/>";

    // Drop previous table of same name if one exists.
    $query = "DROP TABLE IF EXISTS inventory;";
    pg_query($connection, $query) 
        or die("Encountered an error when executing given sql statement: ". pg_last_error(). "<br/>");
    print "Finished dropping table (if existed).<br/>";

    // Create table.
    $query = "CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);";
    pg_query($connection, $query) 
        or die("Encountered an error when executing given sql statement: ". pg_last_error(). "<br/>");
    print "Finished creating table.<br/>";

    // Insert some data into table.
    $name = '\'banana\'';
    $quantity = 150;
    $query = "INSERT INTO inventory (name, quantity) VALUES ($name, $quantity);";
    pg_query($connection, $query) 
        or die("Encountered an error when executing given sql statement: ". pg_last_error(). "<br/>");

    $name = '\'orange\'';
    $quantity = 154;
    $query = "INSERT INTO inventory (name, quantity) VALUES ($name, $quantity);";
    pg_query($connection, $query) 
        or die("Encountered an error when executing given sql statement: ". pg_last_error(). "<br/>");

    $name = '\'apple\'';
    $quantity = 100;
    $query = "INSERT INTO inventory (name, quantity) VALUES ($name, $quantity);";
    pg_query($connection, $query) 
        or die("Encountered an error when executing given sql statement: ". pg_last_error()). "<br/>";

    print "Inserted 3 rows of data.<br/>";

    // Closing connection
    pg_close($connection);
?>
```

## <a name="read-data"></a>Чтение данных
Используйте указанный ниже код с инструкцией SQL **SELECT** для подключения и чтения данных. 

 Код вызывает метод [pg_connect()](https://secure.php.net/manual/en/function.pg-connect.php), чтобы подключиться к базе данных Azure для PostgreSQL. Затем он вызывает метод [pg_query()](https://secure.php.net/manual/en/function.pg-query.php) для выполнения команды SELECT, сохраняя результаты в результирующем наборе, и метод [pg_last_error()](https://secure.php.net/manual/en/function.pg-last-error.php), чтобы проверить сведения, если произошла ошибка.  Чтобы считать данные результирующего набора, в цикле вызывается метод [pg_fetch_row()](https://secure.php.net/manual/en/function.pg-fetch-row.php) для каждой строки. Данные строки извлекаются в виде массива `$row` с одним значением данных для каждого столбца в каждой позиции массива.  Чтобы освободить результирующий набор, вызывается метод [pg_free_result()](https://secure.php.net/manual/en/function.pg-free-result.php). После этого вызывается метод [mysqli_close](https://secure.php.net/manual/en/function.pg-close.php), чтобы разорвать подключение.

Замените значения параметров `$host`, `$database`, `$user` и `$password` своими значениями. 

```php
<?php
    // Initialize connection variables.
    $host = "mydemoserver.postgres.database.azure.com";
    $database = "mypgsqldb";
    $user = "mylogin@mydemoserver";
    $password = "<server_admin_password>";
    
    // Initialize connection object.
    $connection = pg_connect("host=$host dbname=$database user=$user password=$password")
                or die("Failed to create connection to database: ". pg_last_error(). "<br/>");

    print "Successfully created connection to database. <br/>";

    // Perform some SQL queries over the connection.
    $query = "SELECT * from inventory";
    $result_set = pg_query($connection, $query) 
        or die("Encountered an error when executing given sql statement: ". pg_last_error(). "<br/>");
    while ($row = pg_fetch_row($result_set))
    {
        print "Data row = ($row[0], $row[1], $row[2]). <br/>";
    }

    // Free result_set
    pg_free_result($result_set);

    // Closing connection
    pg_close($connection);
?>
```

## <a name="update-data"></a>Обновление данных
Используйте указанный ниже код с инструкцией SQL **UPDATE** для подключения и обновления данных.

Код вызывает метод [pg_connect()](https://secure.php.net/manual/en/function.pg-connect.php), чтобы подключиться к базе данных Azure для PostgreSQL. Затем он вызывает метод [pg_query()](https://secure.php.net/manual/en/function.pg-query.php), чтобы выполнить команду, и метод [pg_last_error()](https://secure.php.net/manual/en/function.pg-last-error.php), чтобы проверить сведения, если произошла ошибка. После этого вызывается метод [mysqli_close](https://secure.php.net/manual/en/function.pg-close.php), чтобы разорвать подключение.

Замените значения параметров `$host`, `$database`, `$user` и `$password` своими значениями. 

```php
<?php
    // Initialize connection variables.
    $host = "mydemoserver.postgres.database.azure.com";
    $database = "mypgsqldb";
    $user = "mylogin@mydemoserver";
    $password = "<server_admin_password>";

    // Initialize connection object.
    $connection = pg_connect("host=$host dbname=$database user=$user password=$password")
                or die("Failed to create connection to database: ". pg_last_error(). ".<br/>");

    print "Successfully created connection to database. <br/>";

    // Modify some data in table.
    $new_quantity = 200;
    $name = '\'banana\'';
    $query = "UPDATE inventory SET quantity = $new_quantity WHERE name = $name;";
    pg_query($connection, $query) 
        or die("Encountered an error when executing given sql statement: ". pg_last_error(). ".<br/>");
    print "Updated 1 row of data. </br>";

    // Closing connection
    pg_close($connection);
?>
```


## <a name="delete-data"></a>Удаление данных
Используйте указанный ниже код с инструкцией SQL **DELETE** для подключения и чтения данных. 

 Код вызывает метод [pg_connect()](https://secure.php.net/manual/en/function.pg-connect.php), чтобы подключиться к базе данных Azure для PostgreSQL. Затем он вызывает метод [pg_query()](https://secure.php.net/manual/en/function.pg-query.php), чтобы выполнить команду, и метод [pg_last_error()](https://secure.php.net/manual/en/function.pg-last-error.php), чтобы проверить сведения, если произошла ошибка. После этого вызывается метод [mysqli_close](https://secure.php.net/manual/en/function.pg-close.php), чтобы разорвать подключение.

Замените значения параметров `$host`, `$database`, `$user` и `$password` своими значениями. 

```php
<?php
    // Initialize connection variables.
    $host = "mydemoserver.postgres.database.azure.com";
    $database = "mypgsqldb";
    $user = "mylogin@mydemoserver";
    $password = "<server_admin_password>";

    // Initialize connection object.
    $connection = pg_connect("host=$host dbname=$database user=$user password=$password")
            or die("Failed to create connection to database: ". pg_last_error(). ". </br>");

    print "Successfully created connection to database. <br/>";

    // Delete some data from table.
    $name = '\'orange\'';
    $query = "DELETE FROM inventory WHERE name = $name;";
    pg_query($connection, $query) 
        or die("Encountered an error when executing given sql statement: ". pg_last_error(). ". <br/>");
    print "Deleted 1 row of data. <br/>";

    // Closing connection
    pg_close($connection);
?>
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы очистить все ресурсы, используемые во время этого краткого руководства, удалите группу ресурсов с помощью следующей команды:

```azurecli
az group delete \
    --name $AZ_RESOURCE_GROUP \
    --yes
```

## <a name="next-steps"></a>Дальнейшие действия
> [!div class="nextstepaction"]
> [Перенос базы данных с помощью экспорта и импорта](./howto-migrate-using-export-and-import.md)
