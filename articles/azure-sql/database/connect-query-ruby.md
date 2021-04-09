---
title: Использование Ruby для отправки запросов
description: В этой статье показано, как использовать Ruby для создания программы, которая подключается к базе данных, и создавать к ней запросы с помощью инструкций Transact-SQL.
titleSuffix: Azure SQL Database & SQL Managed Instance
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: sqldbrb=2
ms.devlang: ruby
ms.topic: quickstart
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 05/29/2020
ms.openlocfilehash: 0434a99d28b4f71594e0ca9ce312087dee5b0ee2
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92672533"
---
# <a name="quickstart-use-ruby-to-query-a-database-in-azure-sql-database-or-azure-sql-managed-instance"></a>Краткое руководство. Использование Ruby для отправки запросов к базе данных в службе "База данных SQL Azure" или Управляемом экземпляру SQL Azure
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

В этом кратком руководстве показано, как использовать [Ruby](https://www.ruby-lang.org) для подключения к базе данных и выполнения запроса данных с помощью инструкций Transact-SQL.

## <a name="prerequisites"></a>Предварительные требования

Для работы с данным руководством необходимо следующее:

- База данных. Для создания и настройки базы данных можно использовать одно из этих кратких руководств.

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
  
- Ruby и связанное программное обеспечение для используемой операционной системы:
  
  - **macOS**: установите Homebrew, rbenv и ruby-build, Ruby, FreeTDS и TinyTDS. См. шаги в разделе 1.2, 1.3, 1.4, 1.5 и 2.1 в статье [Create Ruby apps using SQL Server on macOS](https://www.microsoft.com/sql-server/developer-get-started/ruby/mac/) (Создание приложений Ruby с помощью SQL Server в macOS).
  
  - **Ubuntu**: установите необходимые компоненты для Ruby, rbenv и ruby-build, Ruby, FreeTDS и TinyTDS. См. шаги в разделе 1.2, 1.3, 1.4, 1.5 и 2.1 в статье о [Create Ruby apps using SQL Server on Ubuntu](https://www.microsoft.com/sql-server/developer-get-started/ruby/ubuntu/) (Создание приложений Ruby с помощью SQL Server в Ubuntu).
  
  - **Windows**: установите Ruby, Ruby Devkit и TinyTDS. См. [Configure development environment for Ruby development](/sql/connect/ruby/step-1-configure-development-environment-for-ruby-development) (Настройка среды разработки Ruby).

## <a name="get-server-connection-information"></a>Получение сведений о подключении к серверу

Получите сведения, необходимые для подключения к базе данных в службе "База данных SQL Azure". Для дальнейших действий вам понадобится полное имя сервера или имя узла, имя базы данных и данные для входа.

1. Войдите на [портал Azure](https://portal.azure.com/).

2. Перейдите на страницу **Базы данных SQL** или **Управляемые экземпляры SQL**.

3. На странице **Обзор** просмотрите полное имя сервера рядом с полем **Имя сервера** для базы данных в службе "База данных SQL Azure" или полное имя сервера (либо IP-адрес) рядом с полем **Узел** для Управляемого экземпляра SQL Azure или SQL Server на виртуальной машине Azure. Чтобы скопировать имя сервера или имя узла, наведите на него указатель мыши и щелкните значок **копирования**.

> [!NOTE]
> Сведения о подключении SQL Server на виртуальной машине Azure см. в [этом разделе](../virtual-machines/windows/sql-vm-create-portal-quickstart.md#connect-to-sql-server).

## <a name="create-code-to-query-your-database-in-azure-sql-database"></a>Создание кода для создания запроса к базе данных в службе "База данных SQL Azure"

1. Создайте файл *sqltest.rb* в текстовом редакторе и редакторе кода.

1. Добавьте следующий код. Замените значения из вашей базы данных в службе "База данных SQL Azure" на `<server>`, `<database>`, `<username>` и `<password>`.

   >[!IMPORTANT]
   >Данный пример кода использует данные AdventureWorksLT, которые можно выбрать в качестве источника при создании базы данных. Если в вашей базе данных содержатся другие данные, используйте таблицы из собственной базы данных в запросе SELECT. 

   ```ruby
   require 'tiny_tds'
   server = '<server>.database.windows.net'
   database = '<database>'
   username = '<username>'
   password = '<password>'
   client = TinyTds::Client.new username: username, password: password, 
       host: server, port: 1433, database: database, azure: true
   
   puts "Reading data from table"
   tsql = "SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName
           FROM [SalesLT].[ProductCategory] pc
           JOIN [SalesLT].[Product] p
           ON pc.productcategoryid = p.productcategoryid"
   result = client.execute(tsql)
   result.each do |row|
       puts row
   end
   ```

## <a name="run-the-code"></a>Выполнение кода

1. В командной строке выполните следующую команду:

   ```bash
   ruby sqltest.rb
   ```
   
1. Убедитесь, что первые 20 строк с продуктами в категории возвращены из вашей базы данных. 

## <a name="next-steps"></a>Дальнейшие действия
- [Учебник. Проектирование реляционной базы данных в службе "База данных SQL Azure" с помощью SSMS](design-first-database-tutorial.md)
- [Просмотрите репозиторий GitHub для TinyTDS](https://github.com/rails-sqlserver/tiny_tds).
- [Сообщите о проблемах или задайте вопросы по TinyTDS](https://github.com/rails-sqlserver/tiny_tds/issues)
- [Драйвер Ruby для SQL Server](/sql/connect/ruby/ruby-driver-for-sql-server/)