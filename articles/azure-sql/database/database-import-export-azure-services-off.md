---
title: Импортируйте или экспортируйте базу данных SQL Azure, не разрешая службам Azure доступ к серверу.
description: Импортируйте или экспортируйте базу данных SQL Azure, не разрешая службам Azure доступ к серверу.
services: sql-database
ms.service: sql-database
ms.subservice: migration
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/08/2020
ms.openlocfilehash: 3a02876234d43df2e98a3a4e60453fc3f1f74ef6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98724175"
---
# <a name="import-or-export-an-azure-sql-database-without-allowing-azure-services-to-access-the-server"></a>Импорт или экспорт базы данных SQL Azure без предоставления службам Azure доступа к серверу
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

В этой статье показано, как импортировать или экспортировать базу данных SQL Azure, если для параметра *Разрешить службы Azure* на сервере задано значение *выкл* . Рабочий процесс использует виртуальную машину Azure для запуска SqlPackage для выполнения операции импорта или экспорта.

## <a name="sign-in-to-the-azure-portal"></a>Вход на портал Azure

Войдите на [портал Azure](https://portal.azure.com/).

## <a name="create-the-azure-virtual-machine"></a>Создание виртуальной машины Azure

Создайте виртуальную машину Azure, нажав кнопку **развернуть в Azure** .

Этот шаблон позволяет развернуть простую виртуальную машину Windows, используя несколько различных параметров для версии Windows, используя последнюю исправленную версию. Это позволит развернуть виртуальную машину размера a2 в расположении группы ресурсов и вернуть полное доменное имя виртуальной машины.
<br><br>

[![Изображение с кнопкой "Развернуть в Azure".](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-vm-simple-windows%2Fazuredeploy.json)

Дополнительные сведения см. в статье [очень простое развертывание виртуальной машины Windows](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows).

## <a name="connect-to-the-virtual-machine"></a>Подключение к виртуальной машине

Ниже описано, как подключиться к виртуальной машине с помощью подключения к удаленному рабочему столу.

1. После завершения развертывания перейдите в ресурс виртуальной машины.

   ![На снимке экрана показана страница обзора виртуальной машины с кнопкой подключить.](./media/database-import-export-azure-services-off/vm.png)  

2. Выберите **Подключиться**.

   Для виртуальной машины отобразится форма файла протокола удаленного рабочего стола (RDP-файл), а также общедоступный IP-адрес и номер порта.

   ![Форма RDP](./media/database-import-export-azure-services-off/rdp.png)  

3. Выберите **Скачать RDP-файл**.

   > [!NOTE]
   > Вы также можете подключаться к виртуальной машине по протоколу SSH.

4. Закройте форму **Connect to virtual machine** (Подключение к виртуальной машине).
5. Чтобы подключиться к виртуальной машине, откройте скачанный RDP-файл.
6. При появлении запроса выберите **Подключиться**. На компьютере Mac вам понадобится клиент RDP, например [Remote Desktop Client](https://apps.apple.com/app/microsoft-remote-desktop-10/id1295203466?mt=12) из Mac App Store.

7. Введите имя пользователя и пароль, указанные при создании виртуальной машины, и нажмите кнопку **ОК**.

8. При входе в систему может появиться предупреждение о сертификате. Чтобы продолжить процесс подключения, выберите **Да** или **Продолжить**.

## <a name="install-sqlpackage"></a>Установка SqlPackage

[Скачайте и установите последнюю версию SqlPackage](/sql/tools/sqlpackage-download).

Дополнительные сведения см. в разделе [SqlPackage.exe](/sql/tools/sqlpackage).

## <a name="create-a-firewall-rule-to-allow-the-vm-access-to-the-database"></a>Создание правила брандмауэра, разрешающего доступ виртуальной машины к базе данных

Добавьте общедоступный IP-адрес виртуальной машины в брандмауэр сервера.

Ниже описано, как создать правило брандмауэра IP на уровне сервера для общедоступного IP-адреса виртуальной машины и включить подключение с виртуальной машины.

1. Выберите **базы данных SQL** в меню слева, а затем выберите свою базу данных на странице **базы данных SQL** . Откроется страница обзора для базы данных, в которой отображается полное имя сервера (например, **ServerName.Database.Windows.NET**) и приводятся параметры для дальнейшей настройки.

2. Скопируйте полное имя сервера для использования при подключении к серверу и его базам данных.

   ![имя сервера](./media/database-import-export-azure-services-off/server-name.png)

3. На панели инструментов щелкните **Настройка брандмауэра для сервера**. Откроется страница **Параметры брандмауэра** сервера.

   ![Правило брандмауэра для IP-адресов на уровне сервера](./media/database-import-export-azure-services-off/server-firewall-rule.png)

4. Чтобы добавить общедоступный IP-адрес виртуальной машины в новое правило брандмауэра IP-адреса на уровне сервера, на панели инструментов выберите **Добавить IP-клиент** . С использованием правила брандмауэра для IP-адресов на уровне сервера вы можете открыть порт 1433 для одного IP-адреса или диапазона IP-адресов.

5. Щелкните **Сохранить**. Правило брандмауэра IP на уровне сервера создается для общедоступного IP-адреса виртуальной машины, открывающего порт 1433 на сервере.

6. Закройте страницу **Параметры брандмауэра**.

## <a name="export-a-database-using-sqlpackage"></a>Экспорт базы данных с помощью SqlPackage

Сведения о том, как экспортировать базу данных SQL Azure с помощью служебной программы командной строки [SqlPackage](/sql/tools/sqlpackage) , см. в разделе [Экспорт параметров и свойств](/sql/tools/sqlpackage#export-parameters-and-properties). Служебная программа SqlPackage поставляется с последними версиями [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) и [SQL Server Data Tools](/sql/ssdt/download-sql-server-data-tools-ssdt)или можно скачать последнюю версию [SqlPackage](/sql/tools/sqlpackage-download).

Мы рекомендуем использовать служебную программу SqlPackage для масштабирования и производительности в большинстве рабочих сред. Сведения о миграции из SQL Server в Базу данных SQL Azure с использованием BACPAC-файлов см. в [блоге группы консультирования клиентов SQL Server](/archive/blogs/sqlcat/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files).

В этом примере показано, как экспортировать базу данных с помощью SqlPackage.exe с Active Directoryой универсальной аутентификации. Замените на значения, характерные для вашей среды.

```cmd
SqlPackage.exe /a:Export /tf:testExport.bacpac /scs:"Data Source=<servername>.database.windows.net;Initial Catalog=MyDB;" /ua:True /tid:"apptest.onmicrosoft.com"
```

## <a name="import-a-database-using-sqlpackage"></a>Импорт базы данных с помощью SqlPackage

Импорт базы данных SQL Server с помощью программы командной строки [SqlPackage](/sql/tools/sqlpackage) описывается в разделе [Параметры и свойства импорта](/sql/tools/sqlpackage#import-parameters-and-properties). SqlPackage содержит последние [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) и [SQL Server Data Tools](/sql/ssdt/download-sql-server-data-tools-ssdt). Вы также можете скачать последнюю версию [SqlPackage](/sql/tools/sqlpackage-download).

Вместо портала Azure рекомендуется использовать SqlPackage для масштабирования и обеспечения производительности в большинстве рабочих сред. Сведения о миграции из SQL Server в Базу данных SQL Azure с помощью файлов `BACPAC` см. в блоге команды Помощника по Azure для пользователей SQL Server [здесь](/archive/blogs/sqlcat/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files).

Следующая команда SqlPackage импортирует базу данных **AdventureWorks2017** из локального хранилища в базу данных SQL Azure. Эта команда создает базу данных **myMigratedDatabase** с уровнем служб **Премиум** и целевым уровнем служб **P6**. Подставьте соответствующие значения для своей среды.

```cmd
sqlpackage.exe /a:import /tcs:"Data Source=<serverName>.database.windows.net;Initial Catalog=myMigratedDatabase>;User Id=<userId>;Password=<password>" /sf:AdventureWorks2017.bacpac /p:DatabaseEdition=Premium /p:DatabaseServiceObjective=P6
```

> [!IMPORTANT]
> Чтобы подключиться к базе данных SQL Тазуре из корпоративного брандмауэра, в брандмауэре должен быть открыт порт 1433.

В этом примере показано, как импортировать базу данных с помощью SqlPackage, используя универсальную проверку подлинности Active Directory.

```cmd
sqlpackage.exe /a:Import /sf:testExport.bacpac /tdn:NewDacFX /tsn:apptestserver.database.windows.net /ua:True /tid:"apptest.onmicrosoft.com"
```

## <a name="performance-considerations"></a>Вопросы производительности

Скорость экспорта зависит от многих факторов (например, формы данных), поэтому невозможно предсказать, какая скорость должна быть ожидаемой. SqlPackage может занять значительное время, особенно для больших баз данных.

Чтобы добиться максимальной производительности, можно воспользоваться следующими стратегиями.

1. Убедитесь, что в базе данных не запущена другая рабочая нагрузка. Создание копии перед экспортом может быть лучшим решением для того, чтобы другие рабочие нагрузки не выполнялись.
2. Увеличьте цель уровня обслуживания базы данных, чтобы лучше обрабатывали рабочую нагрузку экспорта (в первую очередь операции чтения ввода-вывода). Если база данных в настоящее время GP_Gen5_4, то, возможно, критически важный для бизнесаный уровень будет полезен при чтении рабочей нагрузки.
3. Убедитесь, что имеются кластеризованные индексы, особенно для больших таблиц.
4. Виртуальные машины должны находиться в том же регионе, что и база данных, чтобы предотвратить ограничения сети.
5. Виртуальные машины должны иметь SSD с достаточным размером для создания временных артефактов перед отправкой в хранилище BLOB-объектов.
6. Виртуальные машины должны иметь достаточную базовую конфигурацию и память для конкретной базы данных.

## <a name="store-the-imported-or-exported-bacpac-file"></a>Сохраните импортированный или экспортированный объект. BACPAC-файл

Тот. BACPAC-файл можно хранить в [больших двоичных объектах Azure](../../storage/blobs/storage-blobs-overview.md)или в [службе файлов Azure](../../storage/files/storage-files-introduction.md).

Для достижения максимальной производительности используйте службу файлов Azure. SqlPackage работает с файловой системой, чтобы она могла обращаться непосредственно к файлам Azure.

Чтобы снизить затраты, используйте большие двоичные объекты Azure, стоимость которых меньше, чем общая папка Azure уровня "Премиум". Однако для этого потребуется скопировать [. BACPAC-файл](/sql/relational-databases/data-tier-applications/data-tier-applications#bacpac) между большим двоичным объектом и локальной файловой системой перед операцией импорта или экспорта. В результате процесс займет больше времени.

Для отправки или загрузки. BACPAC-файлы см. в разделе [Перенос данных с помощью AzCopy и хранилища BLOB-объектов](../../storage/common/storage-use-azcopy-v10.md#transfer-data)и [Перенос данных с помощью AzCopy и хранилища файлов](../../storage/common/storage-use-azcopy-files.md).

В зависимости от среды может потребоваться [настроить брандмауэры службы хранилища Azure и виртуальные сети](../../storage/common/storage-network-security.md).

## <a name="next-steps"></a>Дальнейшие действия

- Сведения о подключении к импортированной базе данных SQL и выполнении запросов к ней см. в разделе [Краткое руководство по базе данных SQL Azure: использование SQL Server Management Studio для подключения и запроса данных](connect-query-ssms.md).
- Сведения о миграции из SQL Server в Базу данных SQL Azure с использованием BACPAC-файлов см. в [блоге группы консультирования клиентов SQL Server](https://techcommunity.microsoft.com/t5/DataCAT/Migrating-from-SQL-Server-to-Azure-SQL-Database-using-Bacpac/ba-p/305407).
- Описание процесса миграции базы данных SQL Server в базу данных SQL Azure, включая рекомендации по его использованию, см. в [этой статье](migrate-to-database-from-sql-server.md).
- Чтобы узнать, как безопасно управлять ключами к хранилищу данных и подписанными URL-адресами и совместно их использовать, ознакомьтесь с [руководством по безопасности службы хранилища Azure](../../storage/blobs/security-recommendations.md).
