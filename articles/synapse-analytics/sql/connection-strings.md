---
title: Строки подключения для Synapse SQL
description: Строки подключения для Synapse SQL
author: azaricstefan
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/15/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.custom: devx-track-csharp
ms.openlocfilehash: fb68a15bb64640fd8aa18d2ee01fd4a03d809125
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107565276"
---
# <a name="connection-strings-for-synapse-sql"></a>Строки подключения для Synapse SQL

К Synapse SQL можно подключиться с помощью различных протоколов приложений, таких как [ADO.NET](/dotnet/framework/data/adonet/), [ODBC](/sql/connect/odbc/windows/microsoft-odbc-driver-for-sql-server-on-windows), [PHP](/sql/connect/php/overview-of-the-php-sql-driver?f=255&MSPPError=-2147217396) и [JDBC](/sql/connect/jdbc/microsoft-jdbc-driver-for-sql-server). Ниже приведены некоторые примеры строк подключения для каждого протокола. 

Кроме того, для формирования строки подключения можно использовать портал Azure.  Чтобы создать строку подключения с помощью портала Azure, перейдите к колонке своей базы данных и в разделе *Основное* выберите *Показать строки подключения к базам данных*.

## <a name="sample-adonet-connection-string"></a>Пример строки подключения по протоколу ADO.NET

```csharp
Server=tcp:{your_server}.sql.azuresynapse.net,1433;Database={your_database};User ID={your_user_name};Password={your_password_here};Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
```

## <a name="sample-odbc-connection-string"></a>Пример строки подключения по протоколу ODBC

```csharp
Driver={SQL Server Native Client 11.0};Server=tcp:{your_server}.sql.azuresynapse.net,1433;Database={your_database};Uid={your_user_name};Pwd={your_password_here};Encrypt=yes;TrustServerCertificate=no;Connection Timeout=30;
```

## <a name="sample-php-connection-string"></a>Пример строки подключения по протоколу PHP

```PHP
Server: {your_server}.sql.azuresynapse.net,1433 \r\nSQL Database: {your_database}\r\nUser Name: {your_user_name}\r\n\r\nPHP Data Objects(PDO) Sample Code:\r\n\r\ntry {\r\n   $conn = new PDO ( \"sqlsrv:server = tcp:{your_server}.sql.azuresynapse.net,1433; Database = {your_database}\", \"{your_user_name}\", \"{your_password_here}\");\r\n    $conn->setAttribute( PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION );\r\n}\r\ncatch ( PDOException $e ) {\r\n   print( \"Error connecting to SQL Server.\" );\r\n   die(print_r($e));\r\n}\r\n\rSQL Server Extension Sample Code:\r\n\r\n$connectionInfo = array(\"UID\" => \"{your_user_name}\", \"pwd\" => \"{your_password_here}\", \"Database\" => \"{your_database}\", \"LoginTimeout\" => 30, \"Encrypt\" => 1, \"TrustServerCertificate\" => 0);\r\n$serverName = \"tcp:{your_server}.sql.azuresynapse.net,1433\";\r\n$conn = sqlsrv_connect($serverName, $connectionInfo);
```

## <a name="sample-jdbc-connection-string"></a>Пример строки подключения по протоколу JDBC

```Java
jdbc:sqlserver://yourserver.sql.azuresynapse.net:1433;database=yourdatabase;user={your_user_name};password={your_password_here};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.sql.azuresynapse.net;loginTimeout=30;
```

> [!NOTE]
> Рекомендуем задать время ожидания подключения, равное 300 секундам, чтобы подключение могло выдерживать короткие периоды недоступности.

## <a name="recommendations"></a>Рекомендации

Для выполнения запросов **бессерверного пула SQL** мы рекомендуем использовать [Azure Data Studio](get-started-azure-data-studio.md) и Azure Synapse Studio.

## <a name="next-steps"></a>Дальнейшие действия

Чтобы приступить к отправке запросов к данным аналитики с помощью Visual Studio и других приложений, см. статью [Connect to Azure Synapse Analytics with Visual Studio and SSDT](../sql-data-warehouse/sql-data-warehouse-query-visual-studio.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) (Подключение к Azure Synapse Analytics с помощью Visual Studio и SSDT).