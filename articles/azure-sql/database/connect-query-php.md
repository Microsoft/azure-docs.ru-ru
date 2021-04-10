---
title: Использование PHP для отправки запросов
description: Как с помощью PHP создать программу, которая подключается к базе данных в службе "База данных SQL Azure" или Управляемому экземпляру SQL Azure, и запрашивать из них данные с использованием инструкций T-SQL.
titleSuffix: Azure SQL Database & SQL Managed Instance
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.devlang: php
ms.topic: quickstart
author: stevestein
ms.author: sstein
ms.reviewer: v-masebo
ms.date: 05/29/2020
ms.custom: sqldbrb=2
ms.openlocfilehash: e0ad8e9e71a0ee2c23412d535fe79955edb39ba8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99576656"
---
# <a name="quickstart-use-php-to-query-a-database-in-azure-sql-database"></a>Краткое руководство. Создание запросов к базе данных в службе "База данных SQL Azure" с использованием PHP
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

В этой статье показано, как использовать [PHP](https://php.net/manual/en/intro-whatis.php) для подключения к базе данных в службе "База данных SQL Azure" или Управляемому экземпляру SQL Azure. Затем можно запросить данные с использованием инструкций T-SQL.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим кратким руководством вам понадобится:

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.
- База данных в службе "База данных SQL Azure" или Управляемый экземпляр SQL Azure. Для создания и настройки базы данных можно использовать одно из этих кратких руководств.

  | Действие | База данных SQL | Управляемый экземпляр SQL | SQL Server на виртуальной машине Azure |
  |:--- |:--- |:---|:---|
  | Создание| [Портал](single-database-create-quickstart.md) | [Портал](../managed-instance/instance-create-quickstart.md) | [Портал](../virtual-machines/windows/sql-vm-create-portal-quickstart.md)
  || [CLI](scripts/create-and-configure-database-cli.md) | [CLI](https://medium.com/azure-sqldb-managed-instance/working-with-sql-managed-instance-using-azure-cli-611795fe0b44) |
  || [PowerShell](scripts/create-and-configure-database-powershell.md) | [PowerShell](../managed-instance/scripts/create-configure-managed-instance-powershell.md) | [PowerShell](../virtual-machines/windows/sql-vm-create-powershell-quickstart.md)
  | Configure | [Правило брандмауэра для IP-адресов на уровне сервера](firewall-create-server-level-portal-quickstart.md)| [Подключение из виртуальной машины](../managed-instance/connect-vm-instance-configure.md)|
  |||[Подключение из локальной сети](../managed-instance/point-to-site-p2s-configure.md) | [Подключение к экземпляру SQL Server](../virtual-machines/windows/sql-vm-create-portal-quickstart.md)
  |Загрузка данных|База данных Adventure Works, загруженная для краткого руководства|[Восстановление базы данных Wide World Importers](../managed-instance/restore-sample-database-quickstart.md) | [Восстановление базы данных Wide World Importers](../managed-instance/restore-sample-database-quickstart.md) |
  |||Восстановление или импорт Adventure Works из файла [BACPAC](database-import.md), размещенного на [GitHub](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/adventure-works)| Восстановление или импорт Adventure Works из файла [BACPAC](database-import.md), размещенного на [GitHub](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/adventure-works)|
  |||




  > [!IMPORTANT]
  > Скрипты в этой статье предназначены для использования базы данных Adventure Works. Используя Управляемый экземпляр SQL, необходимо импортировать базу данных Adventure Works в базу данных экземпляра или изменить скрипты в этой статье для использования базы данных Wide World Importers.

- Установленное программное обеспечение, связанное с PHP, для используемой операционной системы:

  - **macOS.** Установите PHP, драйвер ODBC, а затем драйвер PHP для SQL Server. Сведения см. разделах [Шаг 1, 2 и 3](/sql/connect/php/installation-tutorial-linux-mac).

  - **Linux.** Установите, PHP, драйвер ODBC, а затем драйвер PHP для SQL Server. Сведения см. разделах [Шаг 1, 2 и 3](/sql/connect/php/installation-tutorial-linux-mac).

  - **Windows.** Установите PHP и драйверы PHP, а затем установите драйвер ODBC и SQLCMD. Ознакомьтесь с шагами 1.2 и 1.3 в [этом руководстве](https://www.microsoft.com/sql-server/developer-get-started/php/windows/).

## <a name="get-server-connection-information"></a>Получение сведений о подключении к серверу

Получите сведения, необходимые для подключения к базе данных в составе Базы данных SQL Azure. Для дальнейших действий вам понадобится полное имя сервера или имя узла, имя базы данных и данные для входа.

1. Войдите на [портал Azure](https://portal.azure.com/).

2. Перейдите на страницу **Базы данных SQL** или **Управляемые экземпляры SQL**.

3. На странице **Обзор** просмотрите полное имя сервера рядом с полем **Имя сервера** для базы данных в службе "База данных SQL Azure" или полное имя сервера (либо IP-адрес) рядом с полем **Узел** для Управляемого экземпляра SQL Azure или SQL Server на виртуальной машине Azure. Чтобы скопировать имя сервера или имя узла, наведите на него указатель мыши и щелкните значок **копирования**.

> [!NOTE]
> Сведения о подключении SQL Server на виртуальной машине Azure см. в [этом разделе](../virtual-machines/windows/sql-vm-create-portal-quickstart.md#connect-to-sql-server).

## <a name="add-code-to-query-the-database"></a>Добавление кода для создания запроса к базе данных

1. Создайте файл *sqltest.php* в предпочитаемом текстовом редакторе.  

1. Замените содержимое файла приведенным ниже кодом. Затем добавьте соответствующие значения для сервера, базы данных, пользователя и пароля.

   ```PHP
   <?php
       $serverName = "your_server.database.windows.net"; // update me
       $connectionOptions = array(
           "Database" => "your_database", // update me
           "Uid" => "your_username", // update me
           "PWD" => "your_password" // update me
       );
       //Establishes the connection
       $conn = sqlsrv_connect($serverName, $connectionOptions);
       $tsql= "SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName
            FROM [SalesLT].[ProductCategory] pc
            JOIN [SalesLT].[Product] p
            ON pc.productcategoryid = p.productcategoryid";
       $getResults= sqlsrv_query($conn, $tsql);
       echo ("Reading data from table" . PHP_EOL);
       if ($getResults == FALSE)
           echo (sqlsrv_errors());
       while ($row = sqlsrv_fetch_array($getResults, SQLSRV_FETCH_ASSOC)) {
        echo ($row['CategoryName'] . " " . $row['ProductName'] . PHP_EOL);
       }
       sqlsrv_free_stmt($getResults);
   ?>
   ```

## <a name="run-the-code"></a>Выполнение кода

1. Запустите приложение в командной строке.

   ```bash
   php sqltest.php
   ```

1. Убедитесь, что возвращены первые 20 строк, и закройте окно приложения.

## <a name="next-steps"></a>Дальнейшие действия

- [Учебник. Проектирование реляционной базы данных в службе "База данных SQL Azure" с помощью SSMS](design-first-database-tutorial.md)
- [Microsoft PHP Drivers for SQL Server](https://github.com/Microsoft/msphpsql/) (Драйверы Microsoft PHP для SQL Server)
- [Сообщите о проблемах или задайте вопросы](https://github.com/Microsoft/msphpsql/issues)
- [Шаг 4. Выполнение устойчивого подключения к SQL с помощью PHP](/sql/connect/php/step-4-connect-resiliently-to-sql-with-php)
