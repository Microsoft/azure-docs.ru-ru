---
title: Логическое декодирование — база данных Azure для PostgreSQL — один сервер
description: Описывает логическое декодирование и wal2json для отслеживания измененных данных в базе данных Azure для PostgreSQL-Single Server.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: conceptual
ms.date: 12/09/2020
ms.openlocfilehash: 0ea58050c5dc952392df56b4fb556a0998eef165
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96938908"
---
# <a name="logical-decoding"></a>Логическое декодирование

[Логическое декодирование в PostgreSQL](https://www.postgresql.org/docs/current/logicaldecoding.html) позволяет передавать изменения данных внешним потребителям. Логическое декодирование часто используется для потоковой передачи событий и сценариев отслеживания измененных данных.

При логическом декодировании используется подключаемый модуль вывода для преобразования журнала упреждающего ввода postgres (WAL) в доступный для чтения формат. Служба "база данных Azure для PostgreSQL" предоставляет выходные подключаемые модули [wal2json](https://github.com/eulerto/wal2json), [test_decoding](https://www.postgresql.org/docs/current/test-decoding.html) и пгаутпут. пгаутпут становится доступным для PostgreSQL из PostgreSQL версии 10 и выше.

Общие сведения о том, как работает логическое декодирование postgres, см. в [нашем блоге](https://techcommunity.microsoft.com/t5/azure-database-for-postgresql/change-data-capture-in-postgres-how-to-use-logical-decoding-and/ba-p/1396421). 

> [!NOTE]
> Логическая репликация с использованием публикации или подписки PostgreSQL не поддерживается в базе данных Azure для PostgreSQL-Single Server.


## <a name="set-up-your-server"></a>Настройка сервера 
Логическая декодирование и [Чтение реплик](concepts-read-replicas.md) зависят от postgres журнала упреждающего записи (WAL) для получения сведений. Для этих двух функций требуется разный уровень ведения журнала от postgres. Для логического декодирования требуется более высокий уровень ведения журнала, чем считывание реплик.

Чтобы настроить правильный уровень ведения журнала, используйте параметр поддержки репликации Azure. Поддержка репликации в Azure имеет три параметра:

* **Off** — накладывает наименьшую информацию в Wal. Этот параметр недоступен на большинстве серверов базы данных Azure для PostgreSQL.  
* **Реплика** — более подробная, чем **Off**. Это минимальный уровень ведения журнала, необходимый для работы [реплик чтения](concepts-read-replicas.md) . Этот параметр используется по умолчанию на большинстве серверов.
* **Логический** — более подробный, чем **реплика**. Это минимальный уровень ведения журнала для работы логического декодирования. Реплики чтения также работают с этим параметром.


### <a name="using-azure-cli"></a>Использование Azure CLI

1. Задайте для azure.replication_support значение `logical` .
   ```azurecli-interactive
   az postgres server configuration set --resource-group mygroup --server-name myserver --name azure.replication_support --value logical
   ``` 

2. Перезапустите сервер, чтобы применить изменение.
   ```azurecli-interactive
   az postgres server restart --resource-group mygroup --name myserver
   ```
3. Если вы используете postgres 9,5 или 9,6 и используете доступ к общедоступной сети, добавьте правило брандмауэра, чтобы включить общедоступный IP-адрес клиента, с которого будет выполняться логическая репликация. Имя правила брандмауэра должно включать **_replrule**. Например, *test_replrule*. Чтобы создать правило брандмауэра на сервере, выполните команду [az postgres server firewall-rule create](/cli/azure/postgres/server/firewall-rule). 

### <a name="using-azure-portal"></a>Использование портала Azure

1. Задайте для параметра Поддержка репликации Azure значение **логический**. Щелкните **Сохранить**.

   :::image type="content" source="./media/concepts-logical/replication-support.png" alt-text="База данных Azure для PostgreSQL — репликация — поддержка репликации Azure":::

2. Перезапустите сервер, чтобы применить изменение, выбрав **Да**.

   :::image type="content" source="./media/concepts-logical/confirm-restart.png" alt-text="База данных Azure для PostgreSQL-Replication-подтверждение перезапуска":::

3. Если вы используете postgres 9,5 или 9,6 и используете доступ к общедоступной сети, добавьте правило брандмауэра, чтобы включить общедоступный IP-адрес клиента, с которого будет выполняться логическая репликация. Имя правила брандмауэра должно включать **_replrule**. Например, *test_replrule*. Затем нажмите кнопку **Сохранить**.

   :::image type="content" source="./media/concepts-logical/client-replrule-firewall.png" alt-text="База данных Azure для PostgreSQL — репликация — Добавление правила брандмауэра":::

## <a name="start-logical-decoding"></a>Начать логическое декодирование

Логическое декодирование можно использовать с помощью протокола потоковой передачи или интерфейса SQL. Оба метода используют [слоты репликации](https://www.postgresql.org/docs/current/logicaldecoding-explanation.html#LOGICALDECODING-REPLICATION-SLOTS). Слот представляет поток изменений из одной базы данных.

Для использования слота репликации требуются права репликации postgres. В настоящее время привилегия репликации доступна только для пользователя-администратора сервера. 

### <a name="streaming-protocol"></a>Протокол потоковой передачи
Часто рекомендуется использовать изменения с помощью протокола потоковой передачи. Вы можете создать собственный потребитель или соединитель или использовать такой инструмент, как [дебезиум](https://debezium.io/). 

Обратитесь к документации по wal2json, чтобы получить [Пример использования протокола потоковой передачи с pg_recvlogical](https://github.com/eulerto/wal2json#pg_recvlogical).


### <a name="sql-interface"></a>Интерфейс SQL
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


## <a name="monitoring-slots"></a>Слоты мониторинга

Необходимо отслеживать логическое декодирование. Необходимо удалить любой неиспользуемый слот репликации. Слоты находятся в postgres WAL журналов и соответствующих системных каталогах, пока они не считаны потребителем. Если потребитель завершает работу или неправильно настроен, неиспользованные журналы будут занимать и заполнять хранилище. Кроме того, неиспользованные журналы увеличивают риск идентификатора транзакции врапараунд. В обоих случаях сервер может стать недоступным. Поэтому важно непрерывно потреблять логические слоты репликации. Если сегмент логической репликации больше не используется, немедленно удалите его.

Столбец "Active" в представлении pg_replication_slots будет указывать, есть ли потребитель, подключенный к слоту.
```SQL
SELECT * FROM pg_replication_slots;
```

[Настройте оповещения](howto-alert-on-metric.md) в *используемом хранилище* и *максимальную задержку* в метриках реплик, чтобы уведомить вас о том, что значения увеличиваются после обычных пороговых значений. 

> [!IMPORTANT]
> Необходимо удалить неиспользуемые слоты репликации. Если этого не сделать, это может привести к недоступности сервера.

## <a name="how-to-drop-a-slot"></a>Как удалить слот
Если вы не используете слот репликации, удалите его.

Удаление слота репликации с именем `test_slot` с помощью SQL:
```SQL
SELECT pg_drop_replication_slot('test_slot');
```

> [!IMPORTANT]
> Если вы прекращаете использовать логическое декодирование, измените azure.replication_support обратно на `replica` или `off` . Сведения о WAL, сохраняемые с помощью, `logical` являются более подробными и должны быть отключены, если не используется логическое декодирование. 

 
## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные [сведения о логическом декодировании](https://www.postgresql.org/docs/current/logicaldecoding-explanation.html)см. в документации по postgres.
* Если у вас возникли вопросы о логическом декодировании, обратитесь к [нашей команде](mailto:AskAzureDBforPostgreSQL@service.microsoft.com) .
* Дополнительные сведения о [репликах чтения](concepts-read-replicas.md).

