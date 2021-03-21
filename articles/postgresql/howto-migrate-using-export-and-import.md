---
title: Миграция базы данных Azure для PostgreSQL — один сервер
description: Описывается, как извлечь базу данных PostgreSQL в файл сценария и импортировать данные из этого файла в целевую базу данных.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: d0f1824a1e4754dddea26484c836df79e95246d5
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104580986"
---
# <a name="migrate-your-postgresql-database-using-export-and-import"></a>Перенос базы данных PostgreSQL с помощью экспорта и импорта
[!INCLUDE[applies-to-postgres-single-flexible-server](includes/applies-to-postgres-single-flexible-server.md)]

Можно извлечь базу данных PostgreSQL в файл сценария с помощью [pg_dump](https://www.postgresql.org/docs/current/static/app-pgdump.html) и импортировать данные из этого файла в целевую базу данных с помощью [psql](https://www.postgresql.org/docs/current/static/app-psql.html).

## <a name="prerequisites"></a>Предварительные требования
Прежде чем приступить к выполнению этого руководства, необходимы следующие компоненты:
- [сервер базы данных Azure для PostgreSQL](quickstart-create-server-database-portal.md) с правилами брандмауэра, разрешающими доступ к этом серверу и его базам данных;
- установленная программа командной строки [pg_dump](https://www.postgresql.org/docs/current/static/app-pgdump.html);
- установленная программа командной строки [psql](https://www.postgresql.org/docs/current/static/app-psql.html).

Выполните приведенные ниже действия, чтобы экспортировать и импортировать базу данных PostgreSQL.

## <a name="create-a-script-file-using-pg_dump-that-contains-the-data-to-be-loaded"></a>Создание файла сценария, содержащего загружаемые данные, с помощью pg_dump
Чтобы экспортировать имеющуюся базу данных PostgreSQL в локальную среду или на виртуальную машину в виде файла сценария SQL, выполните следующую команду:

```bash
pg_dump –-host=<host> --username=<name> --dbname=<database name> --file=<database>.sql
```
Например, если в нем есть локальный сервер и база данных с именем **TestDB** :
```bash
pg_dump --host=localhost --username=masterlogin --dbname=testdb --file=testdb.sql
```

## <a name="import-the-data-on-target-azure-database-for-postgresql"></a>Импорт данных в целевую базу данных Azure для PostrgeSQL
Вы можете использовать командную строку psql с параметром --dbname (-d), чтобы импортировать данные в базу данных Azure для сервера PostrgeSQL и загрузить данные из SQL-файла.

```bash
psql --file=<database>.sql --host=<server name> --port=5432 --username=<user> --dbname=<target database name>
```
В этом примере используется программа psql и файл сценария **testdb.sql** из предыдущего шага, чтобы импортировать данные в базу данных **mypgsqldb** на целевом сервере **mydemoserver.postgres.database.azure.com**.

Для **одного сервера** используйте эту команду. 
```bash
psql --file=testdb.sql --host=mydemoserver.database.windows.net --port=5432 --username=mylogin@mydemoserver --dbname=mypgsqldb
```

Для **гибкого сервера** используйте эту команду.  
```bash
psql --file=testdb.sql --host=mydemoserver.database.windows.net --port=5432 --username=mylogin --dbname=mypgsqldb
```



## <a name="next-steps"></a>Дальнейшие действия
- Сведения о миграции базы данных PostgreSQL с помощью dump и RESTORE см. в статье [Перенос базы данных PostgreSQL с помощью функции Dump и восстановления](howto-migrate-using-dump-and-restore.md).
- Дополнительные сведения о переносе баз данных в службу "База данных Azure для PostgreSQL" см. в [этой статье](https://aka.ms/datamigration).
