---
title: Подключение и выполнение запросов
description: Ссылки на краткие руководства по Базе данных SQL Azure в которых объясняется, как подключаться и создавать запросы к Базе данных SQL Azure и Управляемому экземпляру Базы данных SQL Azure.
titleSuffix: Azure SQL Database & SQL Managed Instance
services: sql-database
ms.service: sql-database
ms.subservice: service
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: guide
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 03/17/2021
ms.openlocfilehash: a8419ff2ba269b05d1fbf7e9d08a284085465fd8
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/09/2021
ms.locfileid: "107257421"
---
# <a name="azure-sql-database-and-azure-sql-managed-instance-connect-and-query-articles"></a>Статьи о подключении и создании запросов к Базам данных SQL Azure и Управляемым экземплярам SQL Azure
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

В этом документе представлены ссылки на примеры для Azure, в которых показано, как подключаться и создавать запросы к Базе данных SQL Azure и Управляемому экземпляру SQL Azure. Некоторые рекомендации, связанные с использованием протокола TLS, см. в разделе [Рекомендации по использованию протокола TLS для подключения к базе данных](#tls-considerations-for-database-connectivity).

## <a name="quickstarts"></a>Краткие руководства

| Краткое руководство | Описание |
|---|---|
|[Среда SQL Server Management Studio](connect-query-ssms.md)|В этом кратком руководстве показано, как использовать SSMS для подключения к базе данных, а затем с помощью инструкций Transact-SQL выполнить запрос, вставку, обновление и удаление данных в базе данных.|
|[Azure Data Studio](/sql/azure-data-studio/quickstart-sql-database?toc=%2fazure%2fsql-database%2ftoc.json)|В этом кратком руководстве показано, как использовать Azure Data Studio для подключения к базе данных, а затем с помощью инструкций Transact-SQL (T-SQL) создать базу данных TutorialDB, которая используется в руководствах по Azure Data Studio.|
|[Портал Azure](connect-query-portal.md)|В этом кратком руководстве показано, как использовать редактор запросов для подключения к базе данных (только к Базе данных SQL Azure), а затем с помощью инструкций Transact-SQL выполнить запрос, вставку, обновление и удаление данных в ней.|
|[Visual Studio Code](connect-query-vscode.md)|В этом кратком руководстве показано, как, используя Visual Studio Code, подключиться к базе данных, а затем с помощью инструкций Transact-SQL выполнить запрос, вставку, обновление и удаление данных в ней.|
|[.NET с Visual Studio](connect-query-dotnet-visual-studio.md)|В этом кратком руководстве показано, как использовать .NET .NET Framework для создания программы C# с помощью Visual Studio, которая подключается к базе данных, а затем с помощью инструкций Transact-SQL выполнить запрос данных.|
|[.NET Core](connect-query-dotnet-core.md)|В этом кратком руководстве показано, как использовать .NET Core в Windows, Linux и macOS для создания программы C#, которая подключается к базе данных, а затем с помощью инструкций Transact-SQL выполнить запрос данных.|
|[GO](connect-query-go.md)|В этом кратком руководстве показано, как использовать Go для подключения к базе данных. Также демонстрируется применение инструкций Transact-SQL для запроса и изменения данных.|
|[Java](connect-query-java.md)|В этом кратком руководстве показано, как использовать Java для подключения к базе данных, а затем с помощью инструкций Transact-SQL выполнить запрос данных.|
|[Node.js](connect-query-nodejs.md)|В этом кратком руководстве показано, как использовать Node.js для создания программы, которая подключается к базе данных, а затем с помощью инструкций Transact-SQL выполнить запрос данных.|
|[PHP](connect-query-php.md)|В этом кратком руководстве показано, как использовать PHP для создания программы, которая подключается к базе данных, а затем с помощью инструкций Transact-SQL выполнить запрос данных.|
|[Python](connect-query-python.md)|В этом кратком руководстве показано, как использовать Python для подключения к базе данных, а затем с помощью инструкций Transact-SQL выполнить запрос данных. |
|[Ruby](connect-query-ruby.md)|В этом кратком руководстве показано, как использовать Ruby для создания программы, которая подключается к базе данных, а затем с помощью инструкций Transact-SQL выполнить запрос данных.|
|||

## <a name="get-server-connection-information"></a>Получение сведений о подключении к серверу

Получите сведения, необходимые для подключения к базе данных в составе Базы данных SQL Azure. Для дальнейших действий вам понадобится полное имя сервера или имя узла, имя базы данных и данные для входа.

1. Войдите на [портал Azure](https://portal.azure.com/).

2. Перейдите на страницу **Базы данных SQL** или **Управляемые экземпляры SQL**.

3. На странице **Обзор** просмотрите полное имя сервера рядом с полем **Имя сервера** для базы данных в службе "База данных SQL Azure" или полное имя сервера (либо IP-адрес) рядом с полем **Узел** для Управляемого экземпляра SQL Azure или SQL Server на виртуальной машине Azure. Чтобы скопировать имя сервера или имя узла, наведите на него указатель мыши и щелкните значок **копирования**.

> [!NOTE]
> Сведения о подключении SQL Server на виртуальной машине Azure см. в [этом разделе](../virtual-machines/windows/sql-vm-create-portal-quickstart.md#connect-to-sql-server).

## <a name="get-adonet-connection-information-optional---sql-database-only"></a>Получение сведений о подключении к ADO.NET (необязательно — только для Базы данных SQL)

1. Перейдите к колонке базы данных на портале Azure и в разделе **Параметры** выберите **Строки подключений**.

2. Просмотрите полную строку подключения **ADO.NET**.

    ![Строка подключения по протоколу ADO.NET](./media/connect-query-dotnet-core/adonet-connection-string2.png)

3. Скопируйте строку подключения **ADO.NET**, если вы планируете использовать ее.

## <a name="tls-considerations-for-database-connectivity"></a>Рекомендации по использованию протокола TLS для подключения к базе данных

Протокол TLS (Transport Layer Security) использует все драйверы, предоставляемые или поддерживаемые корпорацией Майкрософт, для подключения к базе данных в службе "База данных SQL Azure" или в Управляемом экземпляре SQL Azure. Специальная настройка не требуется. Чтобы выполнить подключение к экземпляру SQL Server, базе данных в службе "База данных SQL Azure" или экземпляру Управляемого экземпляра SQL Azure, мы рекомендуем задать во всех приложениях следующие или подобные конфигурации:

- **Encrypt = On**;
- **TrustServerCertificate = Off**.

В некоторых системах используются другие, но подобные ключевые слова для конфигурации. Эти конфигурации гарантируют, что драйвер клиента проверяет подлинность сертификата TLS, полученного от сервера.

Также рекомендуется отключить протоколы TLS 1.1 и 1.0 в клиенте, если нужно обеспечить соответствие Стандарту безопасности данных в сфере платежных карт (PCI-DSS).

Сторонние драйверы могут не поддерживать TLS по умолчанию. Это может вызвать трудности при подключении к Базе данных SQL Azure или Управляемому экземпляру SQL Azure. Иногда в приложениях со встроенными драйверами не предусмотрено управление этими параметрами подключения. Мы рекомендуем проверять безопасность таких драйверов и приложений перед их использованием в системах, которые взаимодействуют с конфиденциальными данными.

## <a name="libraries"></a>Библиотеки

Для подключения к Базе данных SQL Azure или Управляемому экземпляру SQL Azure можно использовать различные библиотеки и платформы. Ознакомьтесь с нашими [руководствами по началу работы](https://aka.ms/sqldev), чтобы быстро приступить к программированию на таких языках, как C#, Java, Node.js, PHP и Python. Затем создайте приложение с помощью SQL Server в Linux или Windows либо Docker в macOS.

В следующей таблице перечислены библиотеки подключений или *драйверы*, которые клиентские приложения могут применять с помощью различных языков для подключения и использования сервера SQL Server, работающего локально или в облаке. Эти библиотеки и драйверы можно использовать в среде Linux, Windows или Docker, а также при подключении к Базе данных SQL Azure, Управляемому экземпляру SQL Azure и Azure Synapse Analytics.

| Язык | Платформа | Дополнительные ресурсы | Скачивание | Начало работы |
| :-- | :-- | :-- | :-- | :-- |
| C# | Windows, Linux, macOS | [Microsoft ADO.NET для SQL Server](/sql/connect/ado-net/microsoft-ado-net-sql-server) | [Загрузить](https://www.microsoft.com/net/download/) | [Начало работы](https://www.microsoft.com/sql-server/developer-get-started/csharp/ubuntu)
| Java | Windows, Linux, macOS | [Microsoft JDBC Driver для SQL Server](/sql/connect/jdbc/microsoft-jdbc-driver-for-sql-server/) | [Загрузить](/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server) |  [Начало работы](https://www.microsoft.com/sql-server/developer-get-started/java/ubuntu)
| PHP | Windows, Linux, macOS| [Драйвер PHP SQL для SQL Server](/sql/connect/php/microsoft-php-driver-for-sql-server) | [Загрузить](/sql/connect/php/download-drivers-php-sql-server) | [Начало работы](https://www.microsoft.com/sql-server/developer-get-started/php/ubuntu/)
| Node.js | Windows, Linux, macOS | [Драйвер Node.js для SQL Server](/sql/connect/node-js/node-js-driver-for-sql-server/) | [Установка](/sql/connect/node-js/step-1-configure-development-environment-for-node-js-development/) |  [Начало работы](https://www.microsoft.com/sql-server/developer-get-started/node/ubuntu)
| Python | Windows, Linux, macOS | [Драйвер Python SQL](/sql/connect/python/python-driver-for-sql-server/) | Варианты установки: <br/> \* [pymssql](/sql/connect/python/pymssql/step-1-configure-development-environment-for-pymssql-python-development/) <br/> \* [pyodbc](/sql/connect/python/pyodbc/step-1-configure-development-environment-for-pyodbc-python-development/) |  [Начало работы](https://www.microsoft.com/sql-server/developer-get-started/python/ubuntu)
| Ruby | Windows, Linux, macOS | [Драйвер Ruby для SQL Server](/sql/connect/ruby/ruby-driver-for-sql-server/) | [Установка](/sql/connect/ruby/step-1-configure-development-environment-for-ruby-development/) | [Начало работы](https://www.microsoft.com/sql-server/developer-get-started/ruby/ubuntu)
| C++ | Windows, Linux, macOS | [Microsoft ODBC Driver for SQL Server](/sql/connect/odbc/microsoft-odbc-driver-for-sql-server/) | [Загрузить](/sql/connect/odbc/microsoft-odbc-driver-for-sql-server/) |  

В следующей таблице перечислены примеры платформ объектно-реляционного сопоставления и веб-платформ, которые клиентские приложения могут использовать с сервером SQL Server, Базой данных SQL Azure, Управляемым экземпляром SQL Azure или Azure Synapse Analytics. Вы можете использовать платформы в Linux, Windows или Docker.

| Язык | Платформа | ORM |
| :-- | :-- | :-- |
| C# | Windows, Linux, macOS | [Entity Framework](/ef)<br>[Entity Framework Core](/ef/core/index) |
| Java | Windows, Linux, macOS |[Hibernate ORM](https://hibernate.org/orm)|
| PHP | Windows, Linux, macOS | [Laravel (Eloquent)](https://laravel.com/docs/eloquent)<br>[Doctrine](https://www.doctrine-project.org/projects/orm.html) |
| Node.js | Windows, Linux, macOS | [Sequelize ORM](https://sequelize.org/) |
| Python | Windows, Linux, macOS |[Django](https://www.djangoproject.com/) |
| Ruby | Windows, Linux, macOS | [Ruby on Rails](https://rubyonrails.org/) |
||||

## <a name="next-steps"></a>Дальнейшие действия

- Сведения об архитектуре подключения см. в статье [Архитектура подключений к базе данных SQL Azure](connectivity-architecture.md).
- Получите [драйверы SQL Server](/sql/connect/sql-connection-libraries/), которые используются для подключения из клиентских приложений.
- Сведения о подключении к Базе данных SQL Azure или Управляемому экземпляру SQL Azure:
  - [Подключение и выполнение запросов с помощью NET (C#)](connect-query-dotnet-core.md)
  - [Подключение и выполнение запросов с помощью PHP](connect-query-php.md)
  - [Подключение и выполнение запросов с помощью Node.js](connect-query-nodejs.md)
  - [Подключение и выполнение запросов с помощью Java](connect-query-java.md)
  - [Подключение и выполнение запросов с помощью Python](connect-query-python.md)
  - [Подключение и выполнение запросов с помощью Ruby](connect-query-ruby.md)
  - [Установка программ командной строки SQL Server sqlcmd и bcp в Linux](/sql/linux/sql-server-linux-setup-tools). Пользователям Linux рекомендуется подключиться к Базе данных SQL Azure или Управляемому экземпляру SQL с помощью [sqlcmd](/sql/ssms/scripting/sqlcmd-use-the-utility).
- Примеры кода логики повторов:
  - [Шаг 4. Выполнение устойчивого подключения к SQL с помощью ADO.NET][step-4-connect-resiliently-to-sql-with-ado-net-a78n]
  - [Шаг 4. Выполнение устойчивого подключения к SQL с помощью PHP][step-4-connect-resiliently-to-sql-with-php-p42h]

<!-- Link references. -->

[step-4-connect-resiliently-to-sql-with-ado-net-a78n]: /sql/connect/ado-net/step-4-connect-resiliently-sql-ado-net

[step-4-connect-resiliently-to-sql-with-php-p42h]: /sql/connect/php/step-4-connect-resiliently-to-sql-with-php
