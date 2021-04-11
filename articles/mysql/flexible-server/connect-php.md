---
title: Краткое руководство. Подключение к Базе данных Azure для MySQL (Гибкий сервер) с помощью PHP
description: В этом кратком руководстве представлены примеры кода PHP, которые можно использовать для подключения к Базе данных Azure для MySQL (Гибкий сервер) и запроса данных от него.
author: savjani
ms.author: pariks
ms.service: mysql
ms.custom: mvc
ms.topic: quickstart
ms.date: 9/21/2020
ms.openlocfilehash: baba4d373d4a79ab0c339aac00bb9ab48de9262b
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105109613"
---
# <a name="quickstart-use-php-to-connect-and-query-data-in-azure-database-for-mysql---flexible-server"></a>Краткое руководство. Подключение к Базе данных Azure для MySQL (Гибкий сервер) и запрос данных из нее с помощью PHP

> [!IMPORTANT]
> Сейчас предоставляется общедоступная предварительная версия Гибкого сервера Базы данных Azure для MySQL.

В этом кратком руководстве объясняется, как подключиться к Базе данных Azure для MySQL (Гибкий сервер) с помощью приложения [PHP](https://secure.php.net/manual/intro-whatis.php). Здесь также показано, как использовать инструкции SQL для запроса, вставки, обновления и удаления данных в базе данных. В этой статье предполагается, что у вас уже есть опыт разработки на языке PHP и вы только начали работу с Гибким сервером Базы данных Azure для MySQL.

## <a name="prerequisites"></a>Предварительные требования
В качестве отправной точки в этом кратком руководстве используются ресурсы, созданные в соответствии со следующими материалами:

- [Создание Базы данных Azure для MySQL (Гибкий сервер) с помощью портала Azure](./quickstart-create-server-portal.md)
- [Создание Базы данных Azure для MySQL (Гибкий сервер) с помощью Azure CLI](./quickstart-create-server-cli.md)

## <a name="preparing-your-client-workstation"></a>Подготовка клиентской рабочей станции
1. Если вы создали гибкий сервер в режиме *Закрытый доступ (интеграция с виртуальной сетью)* , к этому серверу придется подключаться с другого ресурса в той же виртуальной сети. Например, можно создать виртуальную машину и добавить ее в виртуальную сеть, созданную для гибкого сервера. Дополнительные сведения см. в статье [Создание виртуальной сети для Базы данных Azure для MySQL (Гибкий сервер) и управление ею с помощью Azure CLI](./how-to-manage-virtual-network-cli.md).

2. Если вы создали гибкий сервер в режиме *Открытый доступ (разрешенные IP-адреса)* , вы можете добавить локальный IP-адрес в список правил брандмауэра на этом сервере. Дополнительные сведения см. в статье [Создание правил брандмауэра для Базы данных Azure для MySQL (Гибкий сервер) и управление ими с помощью Azure CLI](./how-to-manage-firewall-cli.md).

### <a name="install-php"></a>Установка PHP

Установите PHP на своем сервере или создайте [веб-приложение](../../app-service/overview.md) Azure с PHP.  Дополнительные сведения см. в статье о [создании правил брандмауэра и управлении ими](./how-to-manage-firewall-portal.md).

#### <a name="macos"></a>MacOS

1. Скачайте [PHP версии 7.1.4](https://secure.php.net/downloads.php).
2. Установите PHP и выполните настройку согласно инструкциям в [руководстве по PHP](https://secure.php.net/manual/install.macosx.php).

#### <a name="linux-ubuntu"></a>Linux (Ubuntu)

1. Скачайте [PHP 7.1.4 (x64) непотокобезопасной версии](https://secure.php.net/downloads.php).
2. Установите PHP и выполните настройку согласно инструкциям в [руководстве по PHP](https://secure.php.net/manual/install.unix.php).

#### <a name="windows"></a>Windows

1. Скачайте [PHP 7.1.4 (x64) непотокобезопасной версии](https://windows.php.net/download#php-7.1).
2. Установите PHP и выполните настройку согласно инструкциям в [руководстве по PHP](https://secure.php.net/manual/install.windows.php).

## <a name="get-connection-information"></a>Получение сведений о подключении

Получите сведения о подключении, необходимые для подключения к Гибкому серверу Базы данных Azure для MySQL. Вам потребуется полное имя сервера и учетные данные для входа.

1. Войдите на [портал Azure](https://portal.azure.com/).
2. В меню слева на портале Azure выберите **Все ресурсы** и выполните поиск по имени созданного сервера (например, **mydemoserver**).
3. Выберите имя сервера.
4. Запишите **имя сервера** и **имя для входа администратора сервера** с панели сервера **Обзор**. Если вы забыли свой пароль, можно также сбросить пароль с помощью этой панели.
 <!---:::image type="content" source="./media/connect-php/1_server-overview-name-login.png" alt-text="Azure Database for MySQL Flexible Server name":::--->

## <a name="connecting-to-flexible-server-using-tlsssl-in-php"></a>Подключение к гибкому серверу с помощью TLS или SSL в PHP

Чтобы установить зашифрованное подключение приложения к гибкому серверу по протоколу TLS или SSL, изучите приведенные ниже примеры кода. Вы можете скачать сертификат, необходимый для обмена данными по протоколу TLS или SSL, из [https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem](https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem)

```php using SSL
$conn = mysqli_init();
mysqli_ssl_set($conn,NULL,NULL, "/var/www/html/DigiCertGlobalRootCA.crt.pem", NULL, NULL);
mysqli_real_connect($conn, 'mydemoserver.mysql.database.azure.com', 'myadmin', 'yourpassword', 'quickstartdb', 3306, MYSQLI_CLIENT_SSL);
if (mysqli_connect_errno($conn)) {
die('Failed to connect to MySQL: '.mysqli_connect_error());
}
```

## <a name="connect-and-create-a-table"></a>Подключение и создание таблицы

Используйте указанный ниже код с инструкцией SQL **CREATE TABLE** для подключения и создания таблицы.

В коде используется класс **улучшенного расширения MySQL** (mysqli), включенный в PHP. Код вызывает методы [mysqli_init](https://secure.php.net/manual/mysqli.init.php) и [mysqli_real_connect](https://secure.php.net/manual/mysqli.real-connect.php), чтобы подключиться к MySQL. Затем код вызывает метод [mysqli_query](https://secure.php.net/manual/mysqli.query.php) для выполнения запроса и метод [mysqli_close](https://secure.php.net/manual/mysqli.close.php), чтобы разорвать подключение.

Замените значения параметров host, username, password и db_name своими значениями.

```php
<?php
$host = 'mydemoserver.mysql.database.azure.com';
$username = 'myadmin';
$password = 'your_password';
$db_name = 'your_database';

//Establishes the connection
$conn = mysqli_init();
mysqli_real_connect($conn, $host, $username, $password, $db_name, 3306);
if (mysqli_connect_errno($conn)) {
die('Failed to connect to MySQL: '.mysqli_connect_error());
}

// Run the create table query
if (mysqli_query($conn, '
CREATE TABLE Products (
`Id` INT NOT NULL AUTO_INCREMENT ,
`ProductName` VARCHAR(200) NOT NULL ,
`Color` VARCHAR(50) NOT NULL ,
`Price` DOUBLE NOT NULL ,
PRIMARY KEY (`Id`)
);
')) {
printf("Table created\n");
}

//Close the connection
mysqli_close($conn);
?>
```

## <a name="insert-data"></a>Добавление данных

Используйте указанный ниже код с инструкцией SQL **INSERT** для подключения и вставки данных.

В коде используется класс **улучшенного расширения MySQL** (mysqli), включенный в PHP. Метод [mysqli_prepare](https://secure.php.net/manual/mysqli.prepare.php) используется для создания подготовленной инструкции INSERT, а затем с помощью метода [mysqli_stmt_bind_param](https://secure.php.net/manual/mysqli-stmt.bind-param.php) привязываются параметры для каждого вставленного значения столбца. Код выполняет инструкцию, используя метод [mysqli_stmt_execute](https://secure.php.net/manual/mysqli-stmt.execute.php), и закрывает ее с помощью метода [mysqli_stmt_close](https://secure.php.net/manual/mysqli-stmt.close.php).

Замените значения параметров host, username, password и db_name своими значениями.

```php
<?php
$host = 'mydemoserver.mysql.database.azure.com';
$username = 'myadmin';
$password = 'your_password';
$db_name = 'your_database';

//Establishes the connection
$conn = mysqli_init();
mysqli_real_connect($conn, $host, $username, $password, $db_name, 3306);
if (mysqli_connect_errno($conn)) {
die('Failed to connect to MySQL: '.mysqli_connect_error());
}

//Create an Insert prepared statement and run it
$product_name = 'BrandNewProduct';
$product_color = 'Blue';
$product_price = 15.5;
if ($stmt = mysqli_prepare($conn, "INSERT INTO Products (ProductName, Color, Price) VALUES (?, ?, ?)")) {
mysqli_stmt_bind_param($stmt, 'ssd', $product_name, $product_color, $product_price);
mysqli_stmt_execute($stmt);
printf("Insert: Affected %d rows\n", mysqli_stmt_affected_rows($stmt));
mysqli_stmt_close($stmt);
}

// Close the connection
mysqli_close($conn);
?>
```

## <a name="read-data"></a>Чтение данных

Используйте указанный ниже код с инструкцией SQL **SELECT** для подключения и чтения данных.  В коде используется класс **улучшенного расширения MySQL** (mysqli), включенный в PHP. В коде используется метод [mysqli_query](https://secure.php.net/manual/mysqli.query.php) для выполнения SQL-запроса и метод [mysqli_fetch_assoc](https://secure.php.net/manual/mysqli-result.fetch-assoc.php) для получения результирующих строк.

Замените значения параметров host, username, password и db_name своими значениями.

```php
<?php
$host = 'mydemoserver.mysql.database.azure.com';
$username = 'myadmin';
$password = 'your_password';
$db_name = 'your_database';

//Establishes the connection
$conn = mysqli_init();
mysqli_real_connect($conn, $host, $username, $password, $db_name, 3306);
if (mysqli_connect_errno($conn)) {
die('Failed to connect to MySQL: '.mysqli_connect_error());
}

//Run the Select query
printf("Reading data from table: \n");
$res = mysqli_query($conn, 'SELECT * FROM Products');
while ($row = mysqli_fetch_assoc($res)) {
var_dump($row);
}

//Close the connection
mysqli_close($conn);
?>
```

## <a name="update-data"></a>Обновление данных

Используйте указанный ниже код с инструкцией SQL **UPDATE** для подключения и обновления данных.

В коде используется класс **улучшенного расширения MySQL** (mysqli), включенный в PHP. Метод [mysqli_prepare](https://secure.php.net/manual/mysqli.prepare.php) используется для создания подготовленной инструкции UPDATE, а затем с помощью метода [mysqli_stmt_bind_param](https://secure.php.net/manual/mysqli-stmt.bind-param.php) привязываются параметры для каждого обновленного значения столбца. Код выполняет инструкцию, используя метод [mysqli_stmt_execute](https://secure.php.net/manual/mysqli-stmt.execute.php), и закрывает ее с помощью метода [mysqli_stmt_close](https://secure.php.net/manual/mysqli-stmt.close.php).

Замените значения параметров host, username, password и db_name своими значениями.

```php
<?php
$host = 'mydemoserver.mysql.database.azure.com';
$username = 'myadmin';
$password = 'your_password';
$db_name = 'your_database';

//Establishes the connection
$conn = mysqli_init();
mysqli_real_connect($conn, $host, $username, $password, $db_name, 3306);
if (mysqli_connect_errno($conn)) {
die('Failed to connect to MySQL: '.mysqli_connect_error());
}

//Run the Update statement
$product_name = 'BrandNewProduct';
$new_product_price = 15.1;
if ($stmt = mysqli_prepare($conn, "UPDATE Products SET Price = ? WHERE ProductName = ?")) {
mysqli_stmt_bind_param($stmt, 'ds', $new_product_price, $product_name);
mysqli_stmt_execute($stmt);
printf("Update: Affected %d rows\n", mysqli_stmt_affected_rows($stmt));

//Close the connection
mysqli_stmt_close($stmt);
}

mysqli_close($conn);
?>
```


## <a name="delete-data"></a>Удаление данных
Используйте следующий код с инструкцией SQL **DELETE** для подключения и чтения данных.

В коде используется класс **улучшенного расширения MySQL** (mysqli), включенный в PHP. Метод [mysqli_prepare](https://secure.php.net/manual/mysqli.prepare.php) используется для создания подготовленной инструкции DELETE, а затем с помощью метода [mysqli_stmt_bind_param](https://secure.php.net/manual/mysqli-stmt.bind-param.php) привязываются параметры для предложения WHERE в инструкции. Код выполняет инструкцию, используя метод [mysqli_stmt_execute](https://secure.php.net/manual/mysqli-stmt.execute.php), и закрывает ее с помощью метода [mysqli_stmt_close](https://secure.php.net/manual/mysqli-stmt.close.php).

Замените значения параметров host, username, password и db_name своими значениями.

```php
<?php
$host = 'mydemoserver.mysql.database.azure.com';
$username = 'myadmin';
$password = 'your_password';
$db_name = 'your_database';

//Establishes the connection
$conn = mysqli_init();
mysqli_real_connect($conn, $host, $username, $password, $db_name, 3306);
if (mysqli_connect_errno($conn)) {
die('Failed to connect to MySQL: '.mysqli_connect_error());
}

//Run the Delete statement
$product_name = 'BrandNewProduct';
if ($stmt = mysqli_prepare($conn, "DELETE FROM Products WHERE ProductName = ?")) {
mysqli_stmt_bind_param($stmt, 's', $product_name);
mysqli_stmt_execute($stmt);
printf("Delete: Affected %d rows\n", mysqli_stmt_affected_rows($stmt));
mysqli_stmt_close($stmt);
}

//Close the connection
mysqli_close($conn);
?>
```
## <a name="next-steps"></a>Дальнейшие действия
- [Зашифрованное подключение с использованием протокола TLS 1.2 к Базе данных Azure для MySQL (Гибкий сервер)](./how-to-connect-tls-ssl.md).
- Изучите дополнительные сведения о [сетевых подключениях к Гибкому серверу Базы данных Azure для MySQL](./concepts-networking.md).
- [Создание правил брандмауэра для Гибкого сервера Базы данных Azure для MySQL и управление ими с помощью портала Azure](./how-to-manage-firewall-portal.md).
- [Создание виртуальной сети для Гибкого сервера Базы данных Azure для MySQL и управление ею с помощью портала Azure](./how-to-manage-virtual-network-portal.md).