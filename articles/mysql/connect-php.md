---
title: Краткое руководство. Подключение к Базе данных Azure для MySQL с помощью PHP
description: В этом кратком руководстве представлены примеры кода PHP, которые можно использовать для подключения к базе данных Azure для MySQL и запроса данных из нее.
author: savjani
ms.author: pariks
ms.service: mysql
ms.custom: mvc
ms.topic: quickstart
ms.date: 10/28/2020
ms.openlocfilehash: bf62eca26d846cc529df43b519b1377f88c8aede
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98132908"
---
# <a name="quickstart-use-php-to-connect-and-query-data-in-azure-database-for-mysql"></a>Краткое руководство. Подключение к Базе данных Azure для MySQL и запрос данных с помощью PHP
В этом кратком руководстве объясняется, как подключиться к базе данных Azure для MySQL с помощью приложения [PHP](https://secure.php.net/manual/intro-whatis.php). Здесь также показано, как использовать инструкции SQL для запроса, вставки, обновления и удаления данных в базе данных.

## <a name="prerequisites"></a>Предварительные требования
Для целей этого краткого руководства понадобится:

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free) бесплатно.
- Создание отдельного сервера Базы данных Azure для MySQL с помощью [портала Azure](./quickstart-create-mysql-server-database-using-azure-portal.md) <br/> или с помощью [Azure CLI](./quickstart-create-mysql-server-database-using-azure-cli.md), если он еще не создан.
- Выполнить **ОДНО** из действий (в зависимости от того, пользуетесь вы общим или частным доступом), чтобы настроить возможность подключения.

    |Действие| Метод подключения|Практическое руководство|
    |:--------- |:--------- |:--------- |
    | **Настройка правил брандмауэра** | Общие | [Портал](./howto-manage-firewall-using-portal.md) <br/> [CLI](./howto-manage-firewall-using-cli.md)|
    | **Настройка конечной точки службы** | Общие | [Портал](./howto-manage-vnet-using-portal.md) <br/> [CLI](./howto-manage-vnet-using-cli.md)|
    | **Настройка приватного канала** | Private | [Портал](./howto-configure-privatelink-portal.md) <br/> [CLI](./howto-configure-privatelink-cli.md) |

- [Создание базы данных и пользователя без прав администратора](./howto-create-users.md?tabs=single-server).
- Установка последней версии PHP для вашей операционной системы:
    - [PHP в macOS](https://secure.php.net/manual/install.macosx.php);
    - [PHP в Linux](https://secure.php.net/manual/install.unix.php);
    - [PHP в Windows](https://secure.php.net/manual/install.windows.php)

> [!NOTE]
> При работе с этим кратким руководством мы используем библиотеку [MySQLi](https://www.php.net/manual/en/book.mysqli.php) для управления подключением и выполнения запросов к серверу.

## <a name="get-connection-information"></a>Получение сведений о подключении
Чтобы получить сведения о подключении сервера базы данных с портала Azure, выполните следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com/).
2. Перейдите на страницу Базы данных Azure для MySQL. Найдите и выберите **База данных Azure для MySQL**.
:::image type="content" source="./media/quickstart-create-mysql-server-database-using-azure-portal/find-azure-mysql-in-portal.png" alt-text="Поиск Базы данных Azure для MySQL":::

2. Выберите свой сервер MySQL (например, **mydemoserver**).
3. На странице **Обзор** скопируйте полное имя сервера рядом с полем **Имя сервера** и имя администратора рядом с полем **Имя для входа администратора сервера**. Чтобы скопировать имя сервера или имя узла, наведите на него указатель мыши и щелкните значок **копирования**.

> [!IMPORTANT]
> - Если вы забыли свой пароль, его можно [сбросить](./howto-create-manage-server-portal.md#update-admin-password).
> - Замените значения параметров **host, username, password** и **db_name** своими значениями**.

## <a name="step-1-connect-to-the-server"></a>Шаг 1. Подключение к серверу
Протокол SSL включен по умолчанию. Для подключения из локальной среды может потребоваться скачать [SSL-сертификат DigiCertGlobalRootG2](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem). Этот код вызывает:
- [mysqli_init](https://secure.php.net/manual/mysqli.init.php) для инициализации MySQLi.
- [mysqli_ssl_set](https://www.php.net/manual/en/mysqli.ssl-set.php) для указания пути SSL-сертификата. Это необходимо для локальной среды, но не требуется для веб-приложения Службы приложений или виртуальных машин Azure.
- [mysqli_real_connect](https://secure.php.net/manual/mysqli.real-connect.php) для подключения к MySQL.
- [mysqli_close](https://secure.php.net/manual/mysqli.close.php) для закрытия подключения.


```php
$host = 'mydemoserver.mysql.database.azure.com';
$username = 'myadmin@mydemoserver';
$password = 'your_password';
$db_name = 'your_database';

//Initializes MySQLi
$conn = mysqli_init();

mysqli_ssl_set($conn,NULL,NULL, "/var/www/html/DigiCertGlobalRootG2.crt.pem", NULL, NULL);

// Establish the connection
mysqli_real_connect($conn, 'mydemoserver.mysql.database.azure.com', 'myadmin@mydemoserver', 'yourpassword', 'quickstartdb', 3306, NULL, MYSQLI_CLIENT_SSL);

//If connection failed, show the error
if (mysqli_connect_errno($conn))
{
    die('Failed to connect to MySQL: '.mysqli_connect_error());
}
```
[Возникли проблемы? Сообщите нам об этом](https://aka.ms/mysql-doc-feedback)

## <a name="step-2-create-a-table"></a>Шаг 2. Создание таблицы
Для подключения используйте следующий код. Этот код вызывает:
- [mysqli_query](https://secure.php.net/manual/mysqli.query.php) для запуска запроса.
```php
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
```

## <a name="step-3-insert-data"></a>Шаг 3. Добавление данных
Используйте указанный ниже код с инструкцией SQL **INSERT** для вставки данных. Этот код использует методы:
- [mysqli_prepare](https://secure.php.net/manual/mysqli.prepare.php) для создания подготовленной инструкции INSERT.
- [mysqli_stmt_bind_param](https://secure.php.net/manual/mysqli-stmt.bind-param.php) для привязки параметров для каждого вставленного значения столбца.
- [mysqli_stmt_execute](https://secure.php.net/manual/mysqli-stmt.execute.php).
- [mysqli_stmt_close](https://secure.php.net/manual/mysqli-stmt.close.php) для закрытия инструкции с помощью метода.


```php
//Create an Insert prepared statement and run it
$product_name = 'BrandNewProduct';
$product_color = 'Blue';
$product_price = 15.5;
if ($stmt = mysqli_prepare($conn, "INSERT INTO Products (ProductName, Color, Price) VALUES (?, ?, ?)"))
{
    mysqli_stmt_bind_param($stmt, 'ssd', $product_name, $product_color, $product_price);
    mysqli_stmt_execute($stmt);
    printf("Insert: Affected %d rows\n", mysqli_stmt_affected_rows($stmt));
    mysqli_stmt_close($stmt);
}

```

## <a name="step-4-read-data"></a>Шаг 4. Чтение данных
Используйте указанный ниже код с инструкцией SQL **SELECT** для чтения данных.  Этот код использует методы:
- [mysqli_query](https://secure.php.net/manual/mysqli.query.php) для выполнения запроса **SELECT**.
- [mysqli_fetch_assoc](https://secure.php.net/manual/mysqli-result.fetch-assoc.php) для получения результирующих строк.

```php
//Run the Select query
printf("Reading data from table: \n");
$res = mysqli_query($conn, 'SELECT * FROM Products');
while ($row = mysqli_fetch_assoc($res))
 {
    var_dump($row);
 }

```


## <a name="step-5-delete-data"></a>Шаг 5. Удаление данных
Используйте указанный ниже код с инструкцией SQL **DELETE** для удаления строк. Этот код использует методы:
- [mysqli_prepare](https://secure.php.net/manual/mysqli.prepare.php) для создания подготовленной инструкции DELETE.
- [mysqli_stmt_bind_param](https://secure.php.net/manual/mysqli-stmt.bind-param.php) для привязки параметров.
- [mysqli_stmt_execute](https://secure.php.net/manual/mysqli-stmt.execute.php) для создания подготовленной инструкции DELETE.
- [mysqli_stmt_close](https://secure.php.net/manual/mysqli-stmt.close.php) для закрытия инструкции.

```php
//Run the Delete statement
$product_name = 'BrandNewProduct';
if ($stmt = mysqli_prepare($conn, "DELETE FROM Products WHERE ProductName = ?")) {
mysqli_stmt_bind_param($stmt, 's', $product_name);
mysqli_stmt_execute($stmt);
printf("Delete: Affected %d rows\n", mysqli_stmt_affected_rows($stmt));
mysqli_stmt_close($stmt);
}
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
> [Управление сервером службы "База данных Azure для MySQL" через портал](./howto-create-manage-server-portal.md)<br/>

> [!div class="nextstepaction"]
> [Управление сервером службы "База данных Azure для MySQL" с помощью CLI](./how-to-manage-single-server-cli.md)

[Не можете найти нужную информацию? Сообщите нам!](https://aka.ms/mysql-doc-feedback)
