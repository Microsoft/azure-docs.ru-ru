---
title: 'Известные проблемы: оперативная миграция с PostgreSQL на базу данных Azure для PostgreSQL'
titleSuffix: Azure Database Migration Service
description: Сведения об известных проблемах и ограничениях миграции с помощью оперативной миграции из PostgreSQL в базу данных Azure для PostgreSQL с использованием Azure Database Migration Service.
services: database-migration
author: arunkumarthiags
ms.author: arthiaga
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom:
- seo-lt-2019
- seo-dt-2019
ms.topic: troubleshooting
ms.date: 02/20/2020
ms.openlocfilehash: fdefcabdda64402610f115832976ec9f7af81b80
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99258835"
---
# <a name="known-issuesmigration-limitations-with-online-migrations-from-postgresql-to-azure-db-for-postgresql"></a>Известные проблемы и ограничения миграции при оперативной миграции из PostgreSQL в базу данных Azure для PostgreSQL

В следующих разделах описываются известные проблемы и ограничения, связанные с сетевыми миграциями из PostgreSQL в базу данных Azure для PostgreSQL.

## <a name="online-migration-configuration"></a>Настройка сетевой миграции

- Исходный сервер PostgreSQL должен работать под управлением версии 9,4, 9,5, 9,6, 10 или 11. Дополнительные сведения см. в статье [Поддерживаемые версии базы данных PostgreSQL](../postgresql/concepts-supported-versions.md).
- Поддерживаются только миграции на ту же или более позднюю версию. Например, миграция PostgreSQL 9,5 в базу данных Azure для PostgreSQL 9,6 или 10 поддерживается, но миграция с PostgreSQL 11 на PostgreSQL 9,6 не поддерживается.
- Чтобы включить логическую репликацию в **исходном файле PostgreSQL postgresql.conf**, задайте следующие параметры:
  - **wal_level** = логическое
  - **max_replication_slots** = [по крайней мере максимальное число баз данных для миграции]; Если вы хотите перенести четыре базы данных, задайте значение как минимум 4.
  - **max_wal_senders** = [количество одновременно выполняющихся баз данных]; рекомендуемое значение — 10
- Добавить IP-адрес агента DMS в исходный PostgreSQL pg_hba. conf
  1. Запишите IP-адрес DMS после завершения подготовки экземпляра Azure Database Migration Service.
  2. Добавьте IP-адрес в файл pg_hba.conf, как показано ниже.

      ```
          host    all    172.16.136.18/10    md5
          host    replication postgres    172.16.136.18/10     md5
      ```

- Пользователь должен иметь роль репликации на сервере, на котором размещается база данных источника.
- Исходные и целевые схемы базы данных должны совпадать.
- Схема в целевой базе данных Azure для PostgreSQL-Single Server не должна содержать внешние ключи. Чтобы удалить внешние ключи, используйте следующий запрос:

    ```
                  SELECT Queries.tablename
           ,concat('alter table ', Queries.tablename, ' ', STRING_AGG(concat('DROP CONSTRAINT ', Queries.foreignkey), ',')) as DropQuery
                ,concat('alter table ', Queries.tablename, ' ', 
                                                STRING_AGG(concat('ADD CONSTRAINT ', Queries.foreignkey, ' FOREIGN KEY (', column_name, ')', 'REFERENCES ', foreign_table_name, '(', foreign_column_name, ')' ), ',')) as AddQuery
        FROM
        (SELECT
        tc.table_schema, 
        tc.constraint_name as foreignkey, 
        tc.table_name as tableName, 
        kcu.column_name, 
        ccu.table_schema AS foreign_table_schema,
        ccu.table_name AS foreign_table_name,
        ccu.column_name AS foreign_column_name 
    FROM 
        information_schema.table_constraints AS tc 
        JOIN information_schema.key_column_usage AS kcu
          ON tc.constraint_name = kcu.constraint_name
          AND tc.table_schema = kcu.table_schema
        JOIN information_schema.constraint_column_usage AS ccu
          ON ccu.constraint_name = tc.constraint_name
          AND ccu.table_schema = tc.table_schema
    WHERE constraint_type = 'FOREIGN KEY') Queries
      GROUP BY Queries.tablename;
    
    ```

    Запустите сценарий удаления внешнего ключа (второй столбец) в результате запроса.

- Схема в целевой базе данных Azure для PostgreSQL-Single Server не должна иметь триггеров. Чтобы отключить триггеры в целевой базе данных, используйте следующий запрос:

     ```
    SELECT Concat('DROP TRIGGER ', Trigger_Name, ';') FROM  information_schema.TRIGGERS WHERE TRIGGER_SCHEMA = 'your_schema';
     ```

## <a name="size-limitations"></a>Ограничения размера
- Вы можете перенести до 2 ТБ данных из PostgreSQL в Azure DB для PostgreSQL с помощью одной службы DMS.
## <a name="datatype-limitations"></a>Ограничения типа данных

  **Ограничение**: Если в таблицах нет первичного ключа, изменения могут быть не синхронизированы с целевой базой данных.

  **Возможное решение**: временно задайте первичный ключ для перемещаемой таблицы, чтобы продолжить. После завершения перемещения данных этот ключ можно удалить.

## <a name="limitations-when-migrating-online-from-aws-rds-postgresql"></a>Ограничения при миграции через Интернет из AWS RDS PostgreSQL

При попытке выполнить миграцию через Интернет из AWS RDS PostgreSQL в базу данных Azure для PostgreSQL могут возникать следующие ошибки.

- **Ошибка**. "The Default value of column '{column}' in table '{table}' in database '{database}' is different on source and target servers. It's '{value on source}' on source and '{value on target}' on target" (Значение по умолчанию столбца "{столбец}" в таблице "{таблица}" в базе данных "{база данных}" на исходном сервере не совпадает с таким же параметром на целевом сервере. Это значение "{значение на исходном сервере}" на исходном сервере и "{значение на целевом сервере}" на целевом).

  **Ограничение**. Эта ошибка возникает, когда значение по умолчанию для схемы столбца отличается между исходной и целевой базами данных.
  **Обходное решение**. Убедитесь, что схема на целевом объекте соответствует схеме в источнике. Дополнительные сведения о переносе схемы см. в [документации по миграции Azure PostgreSQL Online](./tutorial-postgresql-azure-postgresql-online.md#migrate-the-sample-schema).

- **Ошибка**. "Target database '{database}' has '{number of tables}' tables where as source database '{database}' has '{number of tables}' tables. The number of tables on source and target databases should match" (Количество таблиц в целевой базе данных "{база данных}": {количество таблиц}. Количество таблиц в базе данных-источнике "{база данных}": {количество таблиц}. Количество таблиц в базах данных-источнике и целевой базе данных должно совпадать).

  **Ограничение**. Эта ошибка возникает, когда число таблиц отличается между исходной и целевой базами данных.

  **Обходное решение**. Убедитесь, что схема на целевом объекте соответствует схеме в источнике. Дополнительные сведения о переносе схемы см. в [документации по миграции Azure PostgreSQL Online](./tutorial-postgresql-azure-postgresql-online.md#migrate-the-sample-schema).

- **Ошибка:** База данных-источник {Database} пуста.

  **Ограничение**: Эта ошибка возникает, когда база данных-источник пуста. Скорее всего, в качестве источника выбрана неверная база данных.

  **Решение**. Дважды проверьте базу данных-источник, выбранную для миграции, и повторите попытку.

- **Ошибка:** Целевая база данных {Database} пуста. Перенесите схему.

  **Ограничение**. Эта ошибка возникает, если в целевой базе данных нет схемы. Убедитесь, что схема на целевом объекте соответствует схеме в источнике.
  **Обходное решение**. Убедитесь, что схема на целевом объекте соответствует схеме в источнике. Дополнительные сведения о переносе схемы см. в [документации по миграции Azure PostgreSQL Online](./tutorial-postgresql-azure-postgresql-online.md#migrate-the-sample-schema).

## <a name="other-limitations"></a>Другие ограничения

- Имя базы данных не может содержать точку с запятой (;).
- Записываемая таблица должна иметь первичный ключ. Если в таблице нет первичного ключа, результат операций записи DELETE и UPDATE будет непредсказуемым.
- Обновление сегмента первичного ключа игнорируется. Применение такого обновления будет определяться целевой базой данных как обновление, которое не привело к изменению строк, в результате чего запись будет добавлена в таблицу исключений.
- Миграция нескольких таблиц с одинаковыми именами, различающимися только регистром символов (например, таблица1, ТАБЛИЦА1 и Таблица1), может привести к непредсказуемым результатам и поэтому не поддерживается.
- Изменение обработки [CREATE | ALTER | DROP | TRUNCATE] Таблица означающего не поддерживается.
- В Azure Database Migration Service одно действие миграции может поддерживать только до четырех баз данных.
- Миграция таблицы pg_largeobject не поддерживается. 
