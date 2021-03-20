---
title: Логическая репликация и логическое декодирование — база данных Azure для PostgreSQL — гибкий сервер
description: Сведения об использовании логической репликации и логической декодирования в базе данных Azure для PostgreSQL-гибкого сервера
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/23/2020
ms.openlocfilehash: b6689220873aaeb65337ba480e346e5d2c8020ce
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91707869"
---
# <a name="logical-replication-and-logical-decoding-in-azure-database-for-postgresql---flexible-server"></a>Логическая репликация и логическая декодирование в базе данных Azure для PostgreSQL — гибкий сервер

> [!IMPORTANT]
> Гибкий сервер Базы данных Azure для PostgreSQL предоставляется в режиме предварительной версии

Логическая репликация и логические функции декодирования PostgreSQL поддерживаются в базе данных Azure для PostgreSQL-гибкого сервера, для postgres версии 11.

## <a name="comparing-logical-replication-and-logical-decoding"></a>Сравнение логической и логической декодирования
Логическая репликация и логическое декодирование имеют несколько сходств. Оба они
* разрешить репликацию данных из postgres
* использование [журнала упреждающего записи (WAL)](https://www.postgresql.org/docs/current/wal.html) в качестве источника изменений
* Используйте [слоты логической репликации](https://www.postgresql.org/docs/current/logicaldecoding-explanation.html#LOGICALDECODING-REPLICATION-SLOTS) для отправки данных. Слот представляет поток изменений.
* использование [свойства Identity реплики](https://www.postgresql.org/docs/current/sql-altertable.html#SQL-CREATETABLE-REPLICA-IDENTITY) таблицы для определения того, какие изменения могут быть отправлены
* не выполнять репликацию изменений DDL


Две технологии имеют свои отличия: логическая репликация 
* позволяет указать таблицу или набор таблиц для репликации.
* реплицирует данные между экземплярами PostgreSQL

Логическое декодирование 
* Извлекает изменения во всех таблицах базы данных 
* невозможно напрямую передавать данные между экземплярами PostgreSQL


## <a name="pre-requisites-for-logical-replication-and-logical-decoding"></a>Предварительные требования для логической репликации и логического декодирования

1. Задайте для параметра сервера `wal_level` значение `logical` .
2. Перезапустите сервер, чтобы применить `wal_level` изменение.
3. Убедитесь, что ваш экземпляр PostgreSQL разрешает сетевой трафик из подключенного ресурса.
4. Предоставьте администратору разрешения на репликацию пользователя.
   ```SQL
   ALTER ROLE <adminname> WITH REPLICATION;
   ```


## <a name="using-logical-replication-and-logical-decoding"></a>Использование логической репликации и логического декодирования

### <a name="logical-replication"></a>Логическая репликация
Логическая репликация использует термины "издатель" и "Подписчик". 
* Издатель — это база данных PostgreSQL, **из** которой вы отправляете данные. 
* Подписчик — это база данных PostgreSQL, **в** которую вы отправляете данные.

Ниже приведен пример кода, который можно использовать для пробного использования логической репликации.

1. Подключитесь к базе данных издателя. Создайте таблицу и добавьте данные.
   ```SQL
   CREATE TABLE basic(id SERIAL, name varchar(40));
   INSERT INTO basic(name) VALUES ('apple');
   INSERT INTO basic(name) VALUES ('banana');
   ```

2. Создайте публикацию для таблицы.
   ```SQL
   CREATE PUBLICATION pub FOR TABLE basic;
   ```

3. Подключитесь к базе данных подписчика. Создайте таблицу с той же схемой, что и на издателе.
   ```SQL
   CREATE TABLE basic(id SERIAL, name varchar(40));
   ```

4. Создайте подписку, которая будет подключаться к созданной ранее публикации.
   ```SQL
   CREATE SUBSCRIPTION sub CONNECTION 'host=<server>.postgres.database.azure.com user=<admin> dbname=<dbname> password=<password>' PUBLICATION pub;
   ```

5. Теперь можно выполнить запрос к таблице на подписчике. Вы увидите, что он получил данные от издателя.
   ```SQL
   SELECT * FROM basic;
   ```

Можно добавить дополнительные строки в таблицу издателя и просмотреть изменения на подписчике.

Дополнительные сведения о [логической репликации](https://www.postgresql.org/docs/current/logical-replication.html)см. в документации по PostgreSQL.

### <a name="logical-decoding"></a>Логическое декодирование
Логическое декодирование можно использовать через протокол потоковой передачи или интерфейс SQL. 

#### <a name="streaming-protocol"></a>Протокол потоковой передачи
Часто рекомендуется использовать изменения с помощью протокола потоковой передачи. Вы можете создать собственный потребитель или соединитель или использовать сторонние службы, такие как [дебезиум](https://debezium.io/). 

Обратитесь к документации по wal2json, чтобы получить [Пример использования протокола потоковой передачи с pg_recvlogical](https://github.com/eulerto/wal2json#pg_recvlogical).

#### <a name="sql-interface"></a>Интерфейс SQL 
В приведенном ниже примере мы используем интерфейс SQL с подключаемым модулем wal2json.
 
1. Создайте слот.
   ```SQL
   SELECT * FROM pg_create_logical_replication_slot('test_slot', 'wal2json');
   ```
 
2. Выдавать команды SQL. Пример:
   ```SQL
   CREATE TABLE a_table (
      id varchar(40) NOT NULL,
      item varchar(40),
      PRIMARY KEY (id)
   );
   
   INSERT INTO a_table (id, item) VALUES ('id1', 'item1');
   DELETE FROM a_table WHERE id='id1';
   ```

3. Использование изменений.
   ```SQL
   SELECT data FROM pg_logical_slot_get_changes('test_slot', NULL, NULL, 'pretty-print', '1');
   ```

   Результат будет выглядеть следующим образом:
   ```
   {
         "change": [
         ]
   }
   {
         "change": [
                  {
                           "kind": "insert",
                           "schema": "public",
                           "table": "a_table",
                           "columnnames": ["id", "item"],
                           "columntypes": ["character varying(40)", "character varying(40)"],
                           "columnvalues": ["id1", "item1"]
                  }
         ]
   }
   {
         "change": [
                  {
                           "kind": "delete",
                           "schema": "public",
                           "table": "a_table",
                           "oldkeys": {
                                 "keynames": ["id"],
                                 "keytypes": ["character varying(40)"],
                                 "keyvalues": ["id1"]
                           }
                  }
         ]
   }
   ```

4. Удалите слот после завершения его использования.
   ```SQL
   SELECT pg_drop_replication_slot('test_slot'); 
   ```

Дополнительные сведения о [логическом декодировании](https://www.postgresql.org/docs/current/logicaldecoding.html)см. в документации по PostgreSQL.


## <a name="monitoring"></a>Мониторинг
Необходимо отслеживать логическое декодирование. Необходимо удалить любой неиспользуемый слот репликации. Слоты postgres WAL и соответствующие системные каталоги, пока не считаны изменения. В случае сбоя или неправильной настройки подписчика или потребителя неиспользованные журналы будут занимать и заполнять хранилище. Кроме того, неиспользованные журналы увеличивают риск идентификатора транзакции врапараунд. В обоих случаях сервер может стать недоступным. Поэтому важно непрерывно потреблять логические слоты репликации. Если сегмент логической репликации больше не используется, немедленно удалите его.

Столбец "Active" в представлении pg_replication_slots будет указывать, есть ли потребитель, подключенный к слоту.
```SQL
SELECT * FROM pg_replication_slots;
```

[Настройте оповещения](howto-alert-on-metrics.md) для **максимального числа используемых идентификаторов транзакций** и хранилища, которые **используют** гибкие метрики сервера, чтобы уведомить вас о том, что значения увеличиваются после обычных пороговых значений. 

## <a name="limitations"></a>Ограничения
* **Чтение реплик** . в настоящее время реплики чтения базы данных Azure для PostgreSQL не поддерживаются для гибких серверов.
* **Слоты и отработка отказа с высокой** доступностью — разъемы логических репликаций на сервере-источнике недоступны на резервном сервере во вторичной базе данных AZ. Это относится к вам, если сервер использует параметр высокой доступности с избыточностью зоны. В случае отработки отказа на резервный сервер слоты логических репликаций не будут доступны в режиме ожидания.

## <a name="next-steps"></a>Дальнейшие действия
* Дополнительные сведения о [параметрах сети](concepts-networking.md)
* Дополнительные сведения о [расширениях](concepts-extensions.md) , доступных на гибком сервере
* Дополнительные сведения о [высокой доступности](concepts-high-availability.md)

