---
title: Совместимость драйверов и средств — база данных Azure для MySQL
description: В этой статье описываются драйверы и инструменты управления MySQL, совместимые со службой "База данных Azure для MySQL".
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 3/18/2020
ms.openlocfilehash: c2faf8e8b6a814989fa9e1693fddeedb525df13a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100006859"
---
# <a name="mysql-drivers-and-management-tools-compatible-with-azure-database-for-mysql"></a>Совместимость драйверов и инструментов управления MySQL с базой данных Azure для MySQL.
В этой статье описываются драйверы и инструменты управления, совместимые с Базой данных Azure для MySQL.

## <a name="mysql-drivers"></a>Драйверы MySQL
База данных Azure для MySQL использует самый популярный выпуск Community Edition базы данных MySQL. Таким образом, она совместима с самыми разнообразными языками программирования и драйверами. Цель этого руководства — обеспечить поддержку трех последних версий драйверов MySQL и взаимодействие с сообществом разработчиков открытого кода, чтобы постоянно улучшать функциональность и удобство использования драйверов MySQL. Следующая таблица содержит список драйверов, которые были протестированы и которые совместимы с базой данных Azure для MySQL 5.6 и 5.7.

| **Язык программирования** | **Driver** | **Ссылки** | **Совместимые версии** | **Несовместимые версии** | **Примечания** |
| :----------------------- | :--------- | :-------- | :---------------------- | :------------------------ | :-------- |
| PHP | mysqli, pdo_mysql, мисклнд | https://secure.php.net/downloads.php | 5.5, 5.6, 7.x | 5,3 | Для подключения PHP 7.0 с SSL MySQLi добавьте MYSQLI_CLIENT_SSL_DONT_VERIFY_SERVER_CERT в строке подключения. <br> ```mysqli_real_connect($conn, $host, $username, $password, $db_name, 3306, NULL, MYSQLI_CLIENT_SSL_DONT_VERIFY_SERVER_CERT);```<br> Набор PDO: параметр ```PDO::MYSQL_ATTR_SSL_VERIFY_SERVER_CERT``` имеет значение False.|
| .NET | Асинхронный соединитель MySQL для .NET | https://github.com/mysql-net/MySqlConnector <br> [Пакет установки из Nuget](https://www.nuget.org/packages/MySqlConnector/) | Версия 0.27 и более поздние версии | Версия 0.26.5 и предыдущие версии | |
| .NET | MySQL Connector/NET | https://github.com/mysql/mysql-connector-net | 6.6.3, 7,0, 8,0 |  | Ошибка кодирования может привести к сбою подключения в некоторых системах Windows, не поддерживающих UTF-8. |
| Node.js | mysqljs | https://github.com/mysqljs/mysql/ <br> Пакет установки из NPM:<br> Запустите команду `npm install mysql` из NPM | 2.15 | Версия 2.14.1 и предыдущие версии | |
| Node.js | node-mysql2 | https://github.com/sidorares/node-mysql2 | 1.3.4 + | | |
| Go | Переход к драйверу MySQL | https://github.com/go-sql-driver/mysql/releases | 1,3, 1,4 | Версия 1.2 и предыдущие версии | Используйте `allowNativePasswords=true` в строке подключения для версии 1,3. Версия 1,4 содержит исправление и `allowNativePasswords=true` больше не требуется. |
| Python | Соединитель MySQL/Python | https://pypi.python.org/pypi/mysql-connector-python | 1.2.3, 2,0, 2,1, 2,2, использование 8.0.16 + с MySQL 8,0  | Версия 1.2.2 и предыдущие версии | |
| Python | пимискл | https://pypi.org/project/PyMySQL/ | 0.7.11, 0.8.0, 0.8.1, 0.9.3 + | 0.9.0-0.9.2 (регрессия в web2py) | |
| Java | Соединитель MariaDB/J | https://downloads.mariadb.org/connector-java/ | 2.1, 2.0, 1.6 | Версия 1.5.5 и предыдущие версии | | 
| Java | MySQL Connector/J | https://github.com/mysql/mysql-connector-j | 5.1.21 +, использование 8.0.17 + с MySQL 8,0 | 5.1.20 и ниже | |
| C | Соединитель MySQL/C (либмисклклиент) | https://dev.mysql.com/doc/refman/5.7/en/c-api-implementations.html | 6.0.2 + | | |
| C | Соединитель MySQL/ODBC (мйодбк) | https://github.com/mysql/mysql-connector-odbc | 3.51.29 + | | |
| C++ | Соединитель MySQL/C++ | https://github.com/mysql/mysql-connector-cpp | 1.1.9 + | 1.1.3 и ниже | | 
| C++ | MySQL + +| https://github.com/tangentsoft/mysqlpp | 3.2.3 + | | |
| Ruby | mysql2 | https://github.com/brianmario/mysql2 | 0.4.10 + | | |
| R | RMySQL | https://github.com/rstats-db/RMySQL | 0.10.16 + | | |
| Swift | MySQL — SWIFT | https://github.com/novi/mysql-swift | 0.7.2 + | | |
| Swift | Вапор и MySQL | https://github.com/vapor/mysql-kit | 2.0.1 + | | |

## <a name="management-tools"></a>Средства управления
Преимущества совместимости также распространяются на инструменты управления базой данных. Имеющиеся инструменты должны продолжать работать с базой данных Azure для MySQL, пока обработка базы данных выполняется в пределах разрешений пользователя. Следующая таблица содержит три распространенных инструмента управления базой данных, которые были протестированы и которые совместимы с базой данных Azure для MySQL 5.6 и 5.7.

|                                     | **MySQL Workbench 6.x и более поздней версии** | **Navicat 12** | **PHPMyAdmin 4.x и более поздней версии** | **Дбфорже Studio для MySQL 9,0** |
| :---------------------------------- | :----------------------------- | :------------- | :-------------------------| :------------------------------- |
| **Создание, обновление, чтение, запись и удаление** | X | X | X | X |
| **SSL-соединение** | X | X | X | X |
| **Автозавершение запроса SQL** | X | X |  | X |
| **Импорт и экспорт данных** | X | X | X | X |
| **Экспорт в несколько форматов** | X | X | X | X |
| **Резервное копирование и восстановление** |  | X |  | X |
| **Отображение параметров сервера** | X | X | X | X |
| **Отображение клиентских подключений** | X | X | X | X |

## <a name="next-steps"></a>Дальнейшие действия

- [Узнайте, как устранить проблемы с подключением к Базе данных Azure для MySQL](howto-troubleshoot-common-connection-issues.md)
