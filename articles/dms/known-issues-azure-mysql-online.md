---
title: 'Известные проблемы: оперативная миграция в базу данных Azure для MySQL'
titleSuffix: Azure Database Migration Service
description: Сведения об известных проблемах и ограничениях миграции при оперативной миграции в базу данных Azure для MySQL при использовании Azure Database Migration Service.
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
ms.openlocfilehash: 11659bcbdf77d04c0f4e6f8bc7aca30c716fc924
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97606895"
---
# <a name="online-migration-issues--limitations-to-azure-db-for-mysql-with-azure-database-migration-service"></a>Проблемы оперативной миграции & ограничения для базы данных Azure для MySQL с Azure Database Migration Service

В следующих разделах описываются известные проблемы и ограничения, связанные с сетевыми миграциями из MySQL в Базу данных Azure для MySQL.

## <a name="online-migration-configuration"></a>Настройка сетевой миграции


- На исходном сервере должна быть MySQL версии 5.6.35, 5.7.18 или более поздней.
- База данных Azure для MySQL поддерживает:
  - MySQL Community Edition;
  - подсистему InnoDB.
- Миграция между одинаковыми версиями. Миграция с MySQL 5.6 в Базу данных Azure для MySQL 5.7 не поддерживается. Миграция в версию MySQL 8.0 или из нее не поддерживается.
- Включите ведение двоичных журналов в файле my.ini (Windows) или my.cnf (Unix).
  - Установите для значения Server_id любое число, которое больше или равно 1, например, Server_id = 1 (только для MySQL 5.6).
  - Установите log-bin = \<path> (только для MySQL 5.6).
  - Установите binlog_format = row.
  - Установите Expire_logs_days = 5 ( рекомендуется только для MySQL 5.6).
- Пользователю должна быть назначена роль ReplicationAdmin.
- Параметры сортировки для исходной базы данных MySQL такие же, как те, что определены в целевой базе данных службы "База данных Azure для MySQL".
- Схемы исходной базы данных MySQL и целевой базы данных в Базе данных Azure для MySQL должны соответствовать.
- Схема целевой базы данных службы "База данных Azure для MySQL" не должна содержать внешних ключей. Чтобы удалить внешние ключи, используйте следующий запрос:
    ```sql
    SET group_concat_max_len = 8192;
    SELECT SchemaName, GROUP_CONCAT(DropQuery SEPARATOR ';\n') as DropQuery, GROUP_CONCAT(AddQuery SEPARATOR ';\n') as AddQuery
    FROM
    (SELECT 
    KCU.REFERENCED_TABLE_SCHEMA as SchemaName, KCU.TABLE_NAME, KCU.COLUMN_NAME,
      CONCAT('ALTER TABLE ', KCU.TABLE_NAME, ' DROP FOREIGN KEY ', KCU.CONSTRAINT_NAME) AS DropQuery,
        CONCAT('ALTER TABLE ', KCU.TABLE_NAME, ' ADD CONSTRAINT ', KCU.CONSTRAINT_NAME, ' FOREIGN KEY (`', KCU.COLUMN_NAME, '`) REFERENCES `', KCU.REFERENCED_TABLE_NAME, '` (`', KCU.REFERENCED_COLUMN_NAME, '`) ON UPDATE ',RC.UPDATE_RULE, ' ON DELETE ',RC.DELETE_RULE) AS AddQuery
        FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE KCU, information_schema.REFERENTIAL_CONSTRAINTS RC
        WHERE
        KCU.CONSTRAINT_NAME = RC.CONSTRAINT_NAME
        AND KCU.REFERENCED_TABLE_SCHEMA = RC.UNIQUE_CONSTRAINT_SCHEMA
      AND KCU.REFERENCED_TABLE_SCHEMA = ['schema_name']) Queries
      GROUP BY SchemaName;
    ```

    Запустите сценарий удаления внешнего ключа (второй столбец) в результате запроса.
- Схема целевой базы данных службы "База данных Azure для MySQL" не должна содержать триггеров. Чтобы удалить триггеры в целевой базе данных:
    ```
    SELECT Concat('DROP TRIGGER ', Trigger_Name, ';') FROM  information_schema.TRIGGERS WHERE TRIGGER_SCHEMA = 'your_schema';
    ```

## <a name="datatype-limitations"></a>Ограничения типа данных

- **Ограничение**: миграция завершится ошибкой во время непрерывной синхронизации, если в исходной базе данных MySQL присутствует тип данных JSON.

    **Возможное решение**: измените тип данных JSON на средний или длинный текст в исходной базе данных MySQL.

- **Ограничение**: непрерывная синхронизация завершится ошибкой, если в таблицах нет первичного ключа.

    **Возможное решение**: временно задайте первичный ключ для перемещаемой таблицы, чтобы продолжить. После завершения перемещения данных этот ключ можно удалить.

## <a name="lob-limitations"></a>Ограничения больших объектов

Столбцы больших объектов (LOB) — это столбцы, которые могут увеличиваться в размере. Для MySQL, Medium Text, LONGTEXT, BLOB, Медиумблоб, Лонгблоб и т. д. — это некоторые типы данных LOB.

- **Ограничение**: миграция завершится ошибкой, если типы данных LOB используются в качестве первичных ключей.

    **Возможное решение**: замените первичный ключ другими типами данных или столбцами, которые не относятся к LOB.

- **Ограничение**: Если длина столбца больших объектов (LOB) превышает значение параметра "ограничить размер LOB" (не должно превышать 64 КБ), данные могут быть усечены в целевом объекте. Вы можете проверить длину столбца LOB с помощью этого запроса:
    ```
    SELECT max(length(description)) as LEN from catalog;
    ```

    **Решение**. Если у вас есть объект LOB, размер которого ПРЕВЫШАЕТ 64 КБ, используйте параметр "Разрешить неограниченный объем LOB". Обратите внимание, что при миграции с использованием параметра "Разрешить неограниченный размер LOB" будет выполняться медленнее миграции с использованием параметра "ограничить размер LOB".

## <a name="limitations-when-migrating-online-from-aws-rds-mysql"></a>Ограничения при миграции через Интернет из AWS RDS MySQL

При попытке выполнить миграцию через Интернет из AWS RDS MySQL в базу данных Azure для MySQL могут возникнуть следующие ошибки.

- **Ошибка:** В базе данных " {0} " есть внешние ключи. Исправьте целевой объект и запустите новую процедуру переноса данных. Выполните приведенный ниже скрипт на целевом объекте, чтобы вывести список внешних ключей.

  **Ограничение**. при наличии внешних ключей в схеме начальная нагрузка и непрерывная синхронизация миграции завершатся сбоем.
  Инструкции по **решению**: выполните следующий скрипт в MySQL Workbench, чтобы извлечь скрипт удаления внешнего ключа и добавить скрипт внешнего ключа:

  ```
  SET group_concat_max_len = 8192; SELECT SchemaName, GROUP_CONCAT(DropQuery SEPARATOR ';\n') as DropQuery, GROUP_CONCAT(AddQuery SEPARATOR ';\n') as AddQuery FROM (SELECT KCU.REFERENCED_TABLE_SCHEMA as SchemaName, KCU.TABLE_NAME, KCU.COLUMN_NAME, CONCAT('ALTER TABLE ', KCU.TABLE_NAME, ' DROP FOREIGN KEY ', KCU.CONSTRAINT_NAME) AS DropQuery, CONCAT('ALTER TABLE ', KCU.TABLE_NAME, ' ADD CONSTRAINT ', KCU.CONSTRAINT_NAME, ' FOREIGN KEY (`', KCU.COLUMN_NAME, '`) REFERENCES `', KCU.REFERENCED_TABLE_NAME, '` (`', KCU.REFERENCED_COLUMN_NAME, '`) ON UPDATE ',RC.UPDATE_RULE, ' ON DELETE ',RC.DELETE_RULE) AS AddQuery FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE KCU, information_schema.REFERENTIAL_CONSTRAINTS RC WHERE KCU.CONSTRAINT_NAME = RC.CONSTRAINT_NAME AND KCU.REFERENCED_TABLE_SCHEMA = RC.UNIQUE_CONSTRAINT_SCHEMA AND KCU.REFERENCED_TABLE_SCHEMA = 'SchemaName') Queries GROUP BY SchemaName;
  ```

- **Ошибка:** База данных " {0} " не существует на сервере. На указанном исходном сервере MySQL учитывается регистр. Проверьте имя базы данных.

  **Ограничение**. при миграции базы данных MySQL в Azure с помощью интерфейса командной строки (CLI) пользователи могут столкнуться с этой ошибкой. Службе не удалось найти базу данных на исходном сервере, так как возможно, вы указали неправильное имя базы данных или база данных не существует на указанном сервере. Обратите внимание, что в именах баз данных учитывается регистр.

  **Обходное решение**. Укажите точное имя базы данных и повторите попытку.

- **Ошибка:** В базе данных "{Database}" есть таблицы с одинаковыми именами. В Базе данных Azure для MySQL не учитывается регистр в именах таблиц.

  **Ограничение**. Эта ошибка возникает, если в базе данных источника есть две таблицы с одинаковыми именами. База данных Azure для MySQL не поддерживает таблицы с учетом регистра.

  **Решение**. Обновите имена таблиц, чтобы они были уникальными, а затем повторите операцию.

- **Ошибка:** Целевая база данных {Database} пуста. Перенесите схему.

  **Ограничение**. Эта ошибка возникает, если целевая база данных Azure для базы данных MySQL не имеет требуемой схемы. Миграция схемы необходима для обеспечения миграции данных в целевой объект.

  **Решение**. [перенесите схему](./tutorial-mysql-azure-mysql-online.md#migrate-the-sample-schema) из базы данных источника в целевую базу данных.

## <a name="other-limitations"></a>Другие ограничения

- Строка пароля, которая в начале и конце содержит открывающие и закрывающие фигурные скобки {  }, не поддерживается. Это ограничение применяется при подключении как к исходной базе данных MySQL, так и к целевой базе данных Azure для MySQL.
- Следующие инструкции DDL не поддерживаются:
  - все инструкции DDL, обрабатывающие разделы;
  - Удаление таблицы
  - Переименование таблицы
- Использование инструкции *alter table <имя_таблицы> add column <имя_столбца>* для добавления столбцов в начало или в середину таблицы не поддерживается. Инструкция *alter table <имя_таблицы> add column <имя_столбца>* добавляет столбец в конец таблицы.
- Индексы, созданные только для части данных столбца, не поддерживаются. Приведенная ниже инструкция представляет пример создания индекса, используя только часть данных столбца:

    ``` 
    CREATE INDEX partial_name ON customer (name(10));
    ```

- В Azure Database Migration Service предел переносимых баз данных в одном действии миграции равен четырем.

- Azure Database Migration Service не поддерживает ссылочное действие CASCADE, которое позволяет автоматически удалить или обновить совпадающую строку в дочерней таблице при удалении или обновлении строки в родительской таблице. Дополнительные сведения см. в статье [Ограничения для внешних ключей](https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html) в документации по MySQL. В Azure Database Migration Service нужно снять ограничения для внешних ключей на сервере целевой базы данных во время начальной загрузки данных. При этом нельзя использовать ссылочные действия. Если ваша рабочая нагрузка зависит от обновлений связанной дочерней таблицы, выполняемых с помощью ссылочного действия, рекомендуется переносить базу данных с помощью [дампа и восстановления](../mysql/concepts-migrate-dump-restore.md). 

- **Ошибка:** Размер строки слишком велик (> 8126). Изменение некоторых столбцов на текст или большой двоичный объект может помочь. В текущем формате строки префикс большого двоичного объекта в 0 байт хранится в строке.

  **Ограничение**. Эта ошибка возникает при переносе в базу данных Azure для MySQL с помощью подсистемы хранилища InnoDB и слишком большого размера строки таблицы (>8126 байт).

  **Обходное решение**. обновите схему таблицы, которая имеет размер строки более 8126 байт. Мы не рекомендуем изменять режим "в режиме", так как данные будут обрезаны. Изменение page_size не поддерживается.