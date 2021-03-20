---
title: Подключение к базе данных SQL с помощью C и C++
description: Используйте приведенный в этом кратком руководстве пример кода, чтобы с помощью базы данных SQL Azure разработать современное приложение на C++ на основе мощной облачной реляционной базы данных.
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: sqldbrb=1
ms.devlang: cpp
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 12/12/2018
ms.openlocfilehash: e891c5797c9ce93e6cab7a07d2f68de1a9157249
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92674760"
---
# <a name="connect-to-sql-database-using-c-and-c"></a>Подключение к базе данных SQL с помощью C и C++
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Эта запись предназначена для разработчиков C и C++, пытающихся подключиться к базе данных SQL Azure. Публикация содержит несколько разделов, что дает возможность переходить сразу к интересующей вас теме.

## <a name="prerequisites-for-the-cc-tutorial"></a>Предварительные требования для выполнения инструкций руководства по C/C++

Убедитесь, что у вас есть указанные ниже компоненты.

* Активная учетная запись Azure. Если у вас нет такой учетной записи, вы можете зарегистрироваться для использования [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).
* [Visual Studio](https://www.visualstudio.com/downloads/). Для разработки и запуска этого примера необходимо установить компоненты языка C++.
* [Инструменты разработки Visual Studio для Linux](/cpp/linux/?view=vs-2019). Если вы разрабатываете приложение на платформе Linux, необходимо также установить расширение Linux для Visual Studio.

## <a name="azure-sql-database-and-sql-server-on-virtual-machines"></a><a id="AzureSQL"></a>База данных SQL Azure и SQL Server на виртуальных машинах

База данных SQL Azure построена на Microsoft SQL Server и предназначена для обеспечения высокой доступности, высокопроизводительной и масштабируемой службы. Использование Azure SQL для собственной базы данных, работающей локально, имеет много преимуществ. При использовании Azure SQL вам не нужно устанавливать, настраивать и обслуживать базу данных, а также только содержимое и структуру базы данных. В нее встроены такие возможности, как отказоустойчивость и избыточность, которые так важны при работе с базами данных.

В настоящее время Azure имеет два варианта размещения рабочих нагрузок SQL Server: база данных SQL Azure, база данных как служба и SQL Server на виртуальных машинах (ВМ). Мы не будем подробно избавиться от различий между этими двумя, за исключением того, что база данных SQL Azure является лучшим элементом для новых облачных приложений, чтобы воспользоваться преимуществами экономии и оптимизации производительности, предоставляемых облачными службами. Если вы рассматриваете возможность переноса или расширения своих локальных приложений в облако, сервер SQL на виртуальной машине Azure может быть хорошим выбором. Чтобы упростить работу с этой статьей, создадим базу данных SQL Azure.

## <a name="data-access-technologies-odbc-and-ole-db"></a><a id="ODBC"></a>Технологии доступа к данным: ODBC и OLE DB

Подключение к базе данных SQL Azure не отличается. в настоящее время существует два способа подключения к базам данных: ODBC (Open Database Connectivity) и OLE DB (связывание объектов и внедрение базы данных). В последние годы корпорация Майкрософт поддерживает [ODBC для доступа к собственным реляционным данным](/archive/blogs/sqlnativeclient/microsoft-is-aligning-with-odbc-for-native-relational-data-access). Технология ODBC относительно проста и работает гораздо быстрее, чем OLE DB. Единственное предостережение — ODBC использует старый API в стиле C.

## <a name="step-1--creating-your-azure-sql-database"></a><a id="Create"></a>Шаг 1. Создание базы данных SQL Azure

Чтобы узнать, как создать образец базы данных, перейдите на страницу [Начало работы](single-database-create-quickstart.md) .  Кроме того, для создания базы данных SQL Azure с помощью портал Azure можно воспользоваться этим [коротким видео в двух минутах](https://azure.microsoft.com/documentation/videos/azure-sql-database-create-dbs-in-seconds/) .

## <a name="step-2--get-connection-string"></a><a id="ConnectionString"></a>Шаг 2. Получение строки подключения

После подготовки базы данных SQL Azure необходимо выполнить следующие действия, чтобы определить сведения о подключении и добавить клиентский IP-адрес для доступа к брандмауэру.

В [портал Azure](https://portal.azure.com/)перейдите к строке подключения ODBC базы данных SQL Azure с помощью команды **Показывать строки подключения к базе** данных, перечисленные в разделе Обзор для вашей базы данных.

![ODBCConnectionString](./media/develop-cplusplus-simple/azureportal.png)

![ODBCConnectionStringProps](./media/develop-cplusplus-simple/dbconnection.png)

Скопируйте содержимое строки **ODBC (включает Node.js) [проверка подлинности SQL]**. Оно будет использоваться позже для подключения из интерпретатора командной строки ODBC C++. Эта строка содержит такие сведения, как драйвер, сервер и другие параметры подключения к базе данных.

## <a name="step-3--add-your-ip-to-the-firewall"></a><a id="Firewall"></a>Шаг 3. Добавление IP-адреса в брандмауэр

Перейдите в раздел "брандмауэр" для своего сервера и добавьте [IP-адрес клиента в брандмауэр, выполнив следующие действия](firewall-configure.md) , чтобы убедиться, что мы можем установить успешное подключение:

![AddyourIPWindow](./media/develop-cplusplus-simple/ip.png)

На этом этапе вы настроили базу данных SQL Azure и готовы к подключению из кода C++.

## <a name="step-4-connecting-from-a-windows-cc-application"></a><a id="Windows"></a>Шаг 4. Подключение из приложения Windows C/C++

Вы можете легко подключиться к [базе данных SQL Azure с помощью ODBC в Windows, используя этот пример](https://github.com/Microsoft/VCSamples/tree/master/VC2015Samples/ODBC%20database%20sample%20%28windows%29) , который строится с помощью Visual Studio. В примере реализован интерпретатор командной строки ODBC, который можно использовать для подключения к базе данных SQL Azure. Данный пример принимает в качестве аргумента командной строки файл с именем базы данных-источника (DSN) или подробную строку подключения, скопированную на портале Azure ранее. Откройте страницу свойств для этого проекта и вставьте строку подключения в качестве аргумента команды, как показано ниже:

![DSN Propsfile](./media/develop-cplusplus-simple/props.png)

Убедитесь, что в строке подключения к базе данных указаны правильные сведения для проверки подлинности.

Запустите приложение, чтобы создать его. Должно появиться следующее окно, подтверждающее успешность подключения. Чтобы проверить подключение базы данных, можно выполнить базовые команды SQL, например **создать таблицу**:

![Команды SQL](./media/develop-cplusplus-simple/sqlcommands.png)

Кроме того, файл DSN можно создать при помощи мастера, который запускается, если не указаны аргументы командной строки. Рекомендуется попробовать и этот вариант. Этот файл DSN можно использовать для автоматизации и защиты параметров проверки подлинности:

![Создание файла DSN](./media/develop-cplusplus-simple/datasource.png)

Поздравляем! Вы успешно установили подключение к базе данных SQL Azure при помощи C++ и ODBC в Windows. Чтобы сделать то же самое для платформы Linux, см. сведения дальше в этой статье.

## <a name="step-5-connecting-from-a-linux-cc-application"></a><a id="Linux"></a>Шаг 5. Подключение из приложения Linux C/C++

Если вы еще не слышали новости, Visual Studio теперь позволяет разрабатывать приложения C++ Linux. Об этой новой возможности можно прочитать в записи блога [Visual C++ for Linux Development](https://blogs.msdn.microsoft.com/vcblog/20../../visual-c-for-linux-development/) (Разработка Visual C++ для Linux). Чтобы создавать приложения для Linux, необходим удаленный компьютер, на котором запущен дистрибутив Linux. Если у вас ее нет, вы можете быстро настроить ее с помощью [виртуальных машин Linux в Azure](../../virtual-machines/linux/quick-create-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

Для выполнения инструкций этого руководства предположим, что у вас настроен дистрибутив Linux Ubuntu 16.04. Описанные здесь действия также должны работать для Ubuntu 15.10, Red Hat 6 и Red Hat 7.

С помощью следующих действий устанавливаются библиотеки, необходимые вашему дистрибутиву для SQL и ODBC:

```console
    sudo su
    sh -c 'echo "deb [arch=amd64] https://apt-mo.trafficmanager.net/repos/mssql-ubuntu-test/ xenial main" > /etc/apt/sources.list.d/mssqlpreview.list'
    sudo apt-key adv --keyserver apt-mo.trafficmanager.net --recv-keys 417A0893
    apt-get update
    apt-get install msodbcsql
    apt-get install unixodbc-dev-utf16 #this step is optional but recommended*
```

Запустите Visual Studio. Последовательно выберите "Средства" -> "Параметры" -> "Кроссплатформенный" -> "Диспетчер соединений" и добавьте подключение к компьютеру под управлением Linux:

!["Средства" -> "Параметры"](./media/develop-cplusplus-simple/tools.png)

После того как установлено подключение по протоколу SSH, создайте шаблон пустого проекта (Linux):

![Шаблон нового проекта](./media/develop-cplusplus-simple/template.png)

Затем можно добавить [новый исходный файл C и заменить его этим содержимым](https://github.com/Microsoft/VCSamples/blob/master/VC2015Samples/ODBC%20database%20sample%20%28linux%29/odbcconnector/odbcconnector.c). Используя API ODBC SQLAllocHandle, SQLSetConnectAttr и SQLDriverConnect, можно инициализировать и установить подключение к базе данных.
Как и для образца Windows ODBC, необходимо заменить вызов SQLDriverConnect сведениями из параметров строки подключения к базе данных, ранее скопированными на портале Azure.

```c
     retcode = SQLDriverConnect(
        hdbc, NULL, "Driver=ODBC Driver 13 for SQL"
                    "Server;Server=<yourserver>;Uid=<yourusername>;Pwd=<"
                    "yourpassword>;database=<yourdatabase>",
        SQL_NTS, outstr, sizeof(outstr), &outstrlen, SQL_DRIVER_NOPROMPT);
```

Последнее что необходимо выполнить перед компиляцией — добавить **odbc** в качестве зависимости библиотеки:

![Добавление ODBC в качестве входной библиотеки](./media/develop-cplusplus-simple/lib.png)

Чтобы запустить приложение, откройте консоль Linux из меню **Отладка**:

![Консоль Linux](./media/develop-cplusplus-simple/linuxconsole.png)

Если подключение успешно, вы увидите имя текущей базы данных в консоли Linux:

![Выходные данные в окне консоли Linux](./media/develop-cplusplus-simple/linuxconsolewindow.png)

Поздравляем! Вы успешно завершили учебник и теперь можете подключиться к базе данных SQL Azure из C++ на платформах Windows и Linux.

## <a name="get-the-complete-cc-tutorial-solution"></a><a id="GetSolution"></a>Получение полного решения C/C++ для этого руководства

Решение GetStarted, содержащее все примеры из этой статьи, можно найти в таких разделах GitHub:

* [образец ODBC C++ Windows](https://github.com/Microsoft/VCSamples/tree/master/VC2015Samples/ODBC%20database%20sample%20%28windows%29) — загрузите образец ODBC C++ Windows для подключения к Azure SQL;
* [образец ODBC C++ Linux](https://github.com/Microsoft/VCSamples/tree/master/VC2015Samples/ODBC%20database%20sample%20%28linux%29) — загрузите образец ODBC C++ Linux для подключения к Azure SQL.

## <a name="next-steps"></a>Дальнейшие действия

* [Обзор разработки базы данных SQL](develop-overview.md)
* См. дополнительные сведения в [справочнике по API ODBC](/sql/odbc/reference/syntax/odbc-api-reference/)

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Шаблоны разработки для мультитенантных приложений SaaS с использованием Базы данных Azure SQL](saas-tenancy-app-design-patterns.md)
* Вы можете изучить все [возможности Базы данных SQL](https://azure.microsoft.com/services/sql-database/)