---
title: Строки подключения — база данных Azure для MariaDB
description: В этом документе перечислены все поддерживаемые в настоящее время строки подключения приложений к базе данных Azure для MariaDB, включая ADO.NET (C#), JDBC, Node.js, ODBC, PHP, Python и Ruby.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: how-to
ms.date: 3/18/2020
ms.custom: devx-track-python, devx-track-js, devx-track-csharp
ms.openlocfilehash: 8119f0bfd1f9007cab0df93ad2c7ef22b4db2199
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98662183"
---
# <a name="how-to-connect-applications-to-azure-database-for-mariadb"></a>Подключение приложений к базе данных Azure для MariaDB
В этой статье перечислены типы строк подключения, поддерживаемые базой данных Azure для MariaDB, а также шаблоны и примеры. Строка подключения может содержать различные параметры и настройки.

- Сведения о получении сертификата см. в статье [Настройка SSL-подключений в приложении для безопасного подключения к базе данных Azure для MySQL](./howto-configure-ssl.md).
- {ваш_узел} = [имя_сервера].mariadb.database.azure.com
- {ваш_пользователь}@{имя_сервера} = формат userID для правильной аутентификации.  Если использовать только userID, аутентификация завершится ошибкой.

## <a name="adonet"></a>ADO.NET
```csharp
Server={your_host}; Port=3306; Database={your_database}; Uid={username@servername}; Pwd={your_password}; SslMode=Preferred;
```

В этом примере имя сервера — `mydemoserver`, имя базы данных — `wpdb`, имя пользователя — `WPAdmin`, а пароль — `mypassword!2`. В результате строка подключения будет следующей:

```csharp
Server= "mydemoserver.mariadb.database.azure.com"; Port=3306; Database= "wpdb"; Uid= "WPAdmin@mydemoserver"; Pwd="mypassword!2"; SslMode=Required;
```

## <a name="jdbc"></a>JDBC
```java
String url ="jdbc:mariadb://{your_host}:3306/{your_database}?useSSL=true&trustServerCertificate=true"; myDbConn = DriverManager.getConnection(url, "{username@servername}", {your_password});
```

## <a name="nodejs"></a>Node.js
```javascript
var conn = mysql.createConnection({host: "{your_host}", user: "{your_username}", password: {your_password}, database: {your_database}, port: 3306, ssl:{ca:fs.readFileSync({ca-cert filename})}});
```

## <a name="odbc"></a>ODBC
```cpp
DRIVER={MARIADB ODBC 3.0 Driver}; Server="{your_host}"; Port=3306; Database={your_database}; Uid="{username@servername}"; Pwd={your_password}; sslca={ca-cert filename}; sslverify=1;
```

## <a name="php"></a>PHP
```php
$con=mysqli_init(); mysqli_ssl_set($con, NULL, NULL, {ca-cert filename}, NULL, NULL); mysqli_real_connect($con, "{your_host}", "{username@servername}", {your_password}, {your_database}, 3306);
```

## <a name="python"></a>Python
```python
cnx = mysql.connector.connect(user="{username@servername}", password={your_password}, host="{your_host}", port=3306, database={your_database}, ssl_ca={ca-cert filename}, ssl_verify_cert=true)
```

## <a name="ruby"></a>Ruby
```ruby
client = Mysql2::Client.new(username: "{username@servername}", password: {your_password}, database: {your_database}, host: "{your_host}", port: 3306, sslca:{ca-cert filename}, sslverify:false, sslcipher:'AES256-SHA')
```

## <a name="get-the-connection-string-details-from-the-azure-portal"></a>Получение сведений о строке подключения на портале Azure
На [портале Microsoft Azure](https://portal.azure.com) перейдите к своему серверу базы данных Azure для MariaDB и щелкните **Строки подключения**, чтобы получить список строк для своего экземпляра: ![область строк подключения на портале Microsoft Azure](./media/howto-connection-strings/connection-strings-on-portal.png)

Строка содержит такие сведения, как драйвер, сервер и другие параметры подключения к базе данных. Измените эти примеры, чтобы использовать собственные параметры, такие как имя базы данных, пароль и т. д. Тогда вы сможете использовать эту строку для подключения к серверу из своего кода и приложений.

<!-- 
## Next steps
- For more information about connection libraries, see [Concepts - Connection libraries](./concepts-connection-libraries.md).
- -->
