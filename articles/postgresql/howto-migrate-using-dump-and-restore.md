---
title: Дамп и восстановление — база данных Azure для PostgreSQL — один сервер
description: Описывает, как извлечь базу данных PostgreSQL в файл дампа и выполнить восстановление из файла, созданного pg_dump в базе данных Azure для PostgreSQL-Single Server.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.subservice: migration-guide
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: 16166183b56b371fe8338894f83dbacf2e659c53
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103563561"
---
# <a name="migrate-your-postgresql-database-using-dump-and-restore"></a>Перенос базы данных PostgreSQL с помощью дампа и ее восстановление
[!INCLUDE[applies-to-postgres-single-flexible-server](includes/applies-to-postgres-single-flexible-server.md)]

Можно извлечь базу данных PostgreSQL в файл дампа с помощью [pg_dump](https://www.postgresql.org/docs/current/static/app-pgdump.html) и с помощью [pg_restore](https://www.postgresql.org/docs/current/static/app-pgrestore.html) восстановить базу данных PostgreSQL из файла архива, созданного pg_dump.

## <a name="prerequisites"></a>Предварительные требования
Прежде чем приступить к выполнению этого руководства, необходимы следующие компоненты:
- [сервер базы данных Azure для PostgreSQL](quickstart-create-server-database-portal.md) с правилами брандмауэра, разрешающими доступ к этом серверу и его базам данных;
- установленные программы командной строки [pg_dump](https://www.postgresql.org/docs/current/static/app-pgdump.html) и [pg_restore](https://www.postgresql.org/docs/current/static/app-pgrestore.html).

Выполните указанные ниже действия, чтобы создать дамп базы данных PostgreSQL и восстановить ее.

## <a name="create-a-dump-file-using-pg_dump-that-contains-the-data-to-be-loaded"></a>Создание файла дампа, содержащего загружаемые данные, с помощью pg_dump
Чтобы создать резервную копию базы данных PostgreSQL локально или на виртуальной машине, выполните следующую команду:
```bash
pg_dump -Fc -v --host=<host> --username=<name> --dbname=<database name> -f <database>.dump
```
Например, если имеется локальный сервер с базой данных **testdb**.
```bash
pg_dump -Fc -v --host=localhost --username=masterlogin --dbname=testdb -f testdb.dump
```


## <a name="restore-the-data-into-the-target-azure-database-for-postgresql-using-pg_restore"></a>Восстановите данные в целевую базу данных Azure для PostgreSQL с помощью pg_restore
После создания целевой базы данных можно воспользоваться командой pg_restore с параметром -d, --dbname, чтобы восстановить данные в целевую базу данных из файла дампа.
```bash
pg_restore -v --no-owner --host=<server name> --port=<port> --username=<user-name> --dbname=<target database name> <database>.dump
```

Если включить параметр --no-owner, все объекты, созданные во время восстановления, будут присвоены пользователю --username. Дополнительные сведения см. в официальной документации PostgreSQL по [pg_restore](https://www.postgresql.org/docs/9.6/static/app-pgrestore.html).

> [!NOTE]
> Если серверу PostgreSQL требуются подключения TLS/SSL (по умолчанию в базе данных Azure для серверов PostgreSQL), задайте переменную среды, `PGSSLMODE=require` чтобы pg_restore средство подключается к TLS. Без TLS ошибка может быть прочитана  `FATAL:  SSL connection is required. Please specify SSL options and retry.`
>
> В командной строке Windows выполните команду `SET PGSSLMODE=require` перед выполнением команды pg_restore. В Linux или Bash выполните команду `export PGSSLMODE=require` перед выполнением команды pg_restore.
>

В этом примере восстановите данные из файла дампа **testdb.dump** в базу данных **mypgsqldb** на целевом сервере **mydemoserver.postgres.database.azure.com**.

Ниже приведен пример использования этого **pg_restore** для **одного сервера**:

```bash
pg_restore -v --no-owner --host=mydemoserver.postgres.database.azure.com --port=5432 --username=mylogin@mydemoserver --dbname=mypgsqldb testdb.dump
```
Ниже приведен пример использования этого **pg_restore** для **гибкого сервера**:

```bash
pg_restore -v --no-owner --host=mydemoserver.postgres.database.azure.com --port=5432 --username=mylogin --dbname=mypgsqldb testdb.dump
```
---

## <a name="optimizing-the-migration-process"></a>Оптимизация процесса миграции

Один из способов миграции существующей базы данных PostgreSQL в службу "База данных Azure для PostgreSQL" — это резервное копирование базы данных в источнике и ее восстановление в Azure. Чтобы свести к минимуму время, необходимое для завершения миграции, можно использовать следующие параметры с командами резервного копирования и восстановления.

> [!NOTE]
> Подробные сведения о синтаксисе см. в статьях о [pg_dump](https://www.postgresql.org/docs/current/static/app-pgdump.html) и [pg_restore](https://www.postgresql.org/docs/current/static/app-pgrestore.html).
>

### <a name="for-the-backup"></a>Для резервного копирования
- Создайте резервную копию с использованием параметра -Fc, чтобы вы могли выполнять восстановление параллельно, что позволит ускорить его. Пример:

    ```bash
    pg_dump -h my-source-server-name -U source-server-username -Fc -d source-databasename -f Z:\Data\Backups\my-database-backup.dump
    ```

### <a name="for-the-restore"></a>Для восстановления
- Мы предлагаем переместить файл резервной копии на виртуальную машину Azure в том же регионе, где находится сервер Базы данных Azure для PostgreSQL, на который перемещается база данных, и выполнить команду pg_restore с этой виртуальной машины, чтобы уменьшить задержку сети. Мы также рекомендуем, чтобы виртуальная машина была создана с функцией [ускорения работы в сети](../virtual-network/create-vm-accelerated-networking-powershell.md).

- Это должно происходить по умолчанию, но откройте файл дампа, чтобы проверить, что инструкции создания индекса находятся после вставленных данных. Если это не так, переместите инструкции создания индекса после вставленных данных.

- Для параллелизации восстановления необходимо выполнить восстановление с помощью коммутаторов-FC и-j *#* . *#* число ядер на целевом сервере. Также можно попробовать *#* установить в два раза больше, чем количество ядер целевого сервера, чтобы увидеть влияние. Пример:

Ниже приведен пример использования этого **pg_restore** для **одного сервера**:
```bash
 pg_restore -h my-target-server.postgres.database.azure.com -U azure-postgres-username@my-target-server -Fc -j 4 -d my-target-databasename Z:\Data\Backups\my-database-backup.dump
```
Ниже приведен пример использования этого **pg_restore** для **гибкого сервера**:
```bash
 pg_restore -h my-target-server.postgres.database.azure.com -U azure-postgres-username@my-target-server -Fc -j 4 -d my-target-databasename Z:\Data\Backups\my-database-backup.dump
 ```

- Можно также изменить файл дампа, добавив команду *set synchronous_commit = off;* в начале и команду *set synchronous_commit = on;* в конце. Если не включить ее в конце, прежде чем приложения изменят данные, это может привести к последующей потере данных.

- Перед восстановлением рассмотрите возможность выполнения следующих действий на целевом сервере Базы данных Azure для PostgreSQL.
    - Отключите отслеживание производительности запросов, так как эти статистические данные не нужны во время миграции. Вы можете сделать это, задав для pg_stat_statements.track, pg_qs.query_capture_mode и pgms_wait_sampling.query_capture_mode значение NONE.

    - Используйте номер SKU с высоким объемом ресурсов вычисления и памяти, например номер с оптимизацией для операций в памяти с 32 виртуальными ядрами, чтобы ускорить миграцию. Вы можете легко вернуться к предпочитаемому номеру SKU после завершения восстановления. Чем выше номер SKU, тем большего параллелизма можно достичь, увеличив значение соответствующего параметра `-j` в команде pg_restore.

    - Увеличьте число операций ввода-вывода в секунду на целевом сервере. Это может улучшить производительность восстановления. Вы можете подготовить больше операций ввода-вывода в секунду, увеличив объем хранилища на сервере. Этот параметр необратим, но стоит принять во внимание, будет ли большее количество операций ввода-вывода в секунду полезным для вашей рабочей нагрузки в будущем.

Не забудьте проверить и протестировать эти команды в тестовой среде, прежде чем использовать их в рабочей среде.

## <a name="next-steps"></a>Следующие шаги
- Сведения о миграции базы данных PostgreSQL с помощью экспорта и импорта см. в статье [Перенос базы данных PostgreSQL с помощью экспорта и импорта](howto-migrate-using-export-and-import.md).
- Дополнительные сведения о переносе баз данных в службу "База данных Azure для PostgreSQL" см. в [этой статье](https://aka.ms/datamigration).
