---
title: Подключение и запрос SQL Azure
description: Узнайте, как подключиться к Azure SQL и запросить его.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 07/25/2020
ms.openlocfilehash: b56b65261950e9cf534a3755d214229ef7d5bb1e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93395212"
---
# <a name="connect-and-query-azure-sql-edge"></a>Подключение и запрос SQL Azure

В Azure SQL ребро после развертывания контейнера можно подключиться к ядру СУБД из следующих расположений:

- Внутри контейнера
- Из другого контейнера DOCKER, работающего на том же узле
- С хост-компьютера
- С любой другой клиентской машины в сети

## <a name="tools-to-connect-to-azure-sql-edge"></a>Средства для подключения к SQL Azure для пограничных вычислений

Вы можете подключиться к экземпляру SQL Azure с помощью одного из следующих стандартных средств:

* [sqlcmd](/sql/linux/sql-server-linux-setup-tools): клиентские средства SQLCMD уже включены в образ контейнера Azure SQL. Если подключиться к запущенному контейнеру с помощью интерактивной оболочки bash, можно запускать программы локально. Клиентские средства SQL недоступны на платформе ARM64, так как они не включены в ARM64 версию контейнеров SQL. 
* [Среда SQL Server Management Studio](/sql/ssms/sql-server-management-studio-ssms)
* [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio)
* [Visual Studio Code](/sql/visual-studio-code/sql-server-develop-use-vscode)

Чтобы подключиться к ядру СУБД Azure SQL с сетевого компьютера, вам потребуется следующее:

- **IP-адрес или сетевое имя главного компьютера**: это ведущий компьютер, на котором работает контейнер Azure SQL.
- **Сопоставление портов узла контейнера Azure SQL**. это сопоставление для порта контейнера DOCKER с портом на узле. В контейнере Azure SQL ребро всегда сопоставляется с портом 1433. Вы можете изменить это значение, если хотите. Чтобы изменить номер порта, обновите **Параметры создания контейнера** для модуля Azure SQL в Azure IOT Edge. В следующем примере порт 1433 в контейнере сопоставляется с портом 1600 на узле.

    ```JSON
    {
        "PortBindings": {
          "1433/tcp": [
            {
              "HostPort": "1600"
            }
          ]
        }
    }
    ```

- **Пароль SA для экземпляра Azure SQL ребра**: это значение, указанное для `SA_PASSWORD` переменной среды во время развертывания Azure SQL ребро.

## <a name="connect-to-the-database-engine-from-within-the-container"></a>Подключение к ядру СУБД из контейнера

[Программы командной строки SQL Server](/sql/linux/sql-server-linux-setup-tools) включены в образ контейнера Azure SQL. Если вы подключаетесь к контейнеру с помощью интерактивной командной строки, вы можете запускать средства локально. Клиентские средства SQL недоступны на платформе ARM64, так как они не включены в ARM64 версию контейнеров SQL. 

1. Выполните команду `docker exec -it`, чтобы запустить интерактивную оболочку bash внутри запущенного контейнера. В следующем примере `e69e056c702d` — это идентификатор контейнера.

    ```bash
    docker exec -it <Azure SQL Edge container ID or name> /bin/bash
    ```

    > [!TIP]
    > Указывать идентификатор контейнера полностью во всех случаях не требуется. Достаточно указать количество символов, необходимое для его уникальной идентификации. Поэтому в этом примере может быть достаточно, чтобы использовать `e6` или `e69` , а не полный идентификатор.

2. Когда Вы находитесь внутри контейнера, подключитесь локально с помощью программы sqlcmd. По умолчанию программа sqlcmd не находится в пути, поэтому необходимо указать полный путь.

    ```bash
    /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P '<YourPassword>'
    ```

3. Завершив работу с sqlcmd, введите `exit` .

4. Закончив работу с интерактивной командной строкой, введите `exit` . Контейнер продолжит работать после выхода из интерактивной оболочки bash.

## <a name="connect-to-azure-sql-edge-from-another-container-on-the-same-host"></a>Подключение к Azure SQL Server из другого контейнера на том же узле

Поскольку два контейнера, работающие на одном узле, находятся в одной сети DOCKER, к ним можно легко получить доступ, используя имя контейнера и адрес порта для службы. Например, если вы подключаетесь к экземпляру Azure SQL Server из другого модуля Python (контейнера) на том же узле, можно использовать строку подключения, аналогичную приведенной ниже. (В этом примере предполагается, что Azure SQL ребр настроен для прослушивания порта по умолчанию.)

```python

import pyodbc
server = 'MySQLEdgeContainer' # Replace this with the actual name of your SQL Edge Docker container
username = 'sa' # SQL Server username
password = 'MyStrongestP@ssword' # Replace this with the actual SA password from your deployment
database = 'MyEdgeDatabase' # Replace this with the actual database name from your deployment. If you do not have a database created, you can use Master database.
db_connection_string = "Driver={ODBC Driver 17 for SQL Server};Server=" + server + ";Database=" + database + ";UID=" + username + ";PWD=" + password + ";"
conn = pyodbc.connect(db_connection_string, autocommit=True)

```

## <a name="connect-to-azure-sql-edge-from-another-network-machine"></a>Подключение к Azure SQL с другого сетевого компьютера

Возможно, вам потребуется подключиться к экземпляру Azure SQL Server с другого компьютера в сети. Для этого используйте IP-адрес узла DOCKER и порт узла, с которым сопоставлен контейнер Azure SQL. Например, если IP-адрес узла DOCKER — *xxx.xxx.xxx.XXX*, а контейнер Azure SQL — с портом узла *1600*, то адресом сервера для экземпляра Azure SQL ребро будет *xxx. xxx. xxx. XXX, 1600*. Обновленный скрипт Python:

```python

import pyodbc
server = 'xxx.xxx.xxx.xxx,1600' # Replace this with the actual name of your SQL Edge Docker container
username = 'sa' # SQL Server username
password = 'MyStrongestP@ssword' # Replace this with the actual SA password from your deployment
database = 'MyEdgeDatabase' # Replace this with the actual database name from your deployment. If you do not have a database created, you can use Master database.
db_connection_string = "Driver={ODBC Driver 17 for SQL Server};Server=" + server + ";Database=" + database + ";UID=" + username + ";PWD=" + password + ";"
conn = pyodbc.connect(db_connection_string, autocommit=True)

```

Чтобы подключиться к экземпляру Azure SQL с помощью SQL Server Management Studio, работающей на компьютере Windows, см. [SQL Server Management Studio](/sql/linux/sql-server-linux-manage-ssms).

Чтобы подключиться к экземпляру Azure SQL с помощью Visual Studio Code на компьютере под управлением Windows, Mac или Linux, см. раздел [Visual Studio Code](/sql/visual-studio-code/sql-server-develop-use-vscode).

Чтобы подключиться к экземпляру Azure SQL с помощью Azure Data Studio на компьютере под управлением Windows, Mac или Linux, см. раздел [Azure Data Studio](/sql/azure-data-studio/quickstart-sql-server).

## <a name="next-steps"></a>Дальнейшие действия

[Подключение и запрос](/sql/linux/sql-server-linux-configure-docker#connect-and-query)

[Установка средств SQL Server на Linux](/sql/linux/sql-server-linux-setup-tools)