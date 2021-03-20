---
title: Параметры сервера — масштабирование (Цитус) — база данных Azure для PostgreSQL
description: Параметры в API SQL для масштабирования (Цитус)
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: reference
ms.date: 08/10/2020
ms.openlocfilehash: 07f966c7b0be542f848f1a0a4eaf2b5549735b4b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91336247"
---
# <a name="server-parameters"></a>Параметры сервера

Существуют различные параметры сервера, влияющие на поведение функции "масштабирование" (Цитус), как от стандартных PostgreSQL, так и от особенностей масштабирования (Цитус).
Эти параметры можно задать в портал Azure для группы серверов с горизонтальным масштабированием (Цитус). В категории **Параметры** выберите **Параметры узла рабочей роли** или **Параметры узла координатора**. Эти страницы позволяют задавать параметры для всех рабочих узлов или только для узла координатора.

## <a name="hyperscale-citus-parameters"></a>Параметры масштабирования (Цитус)

> [!NOTE]
>
> Группы серверов Цитус, работающие под управлением старых версий подсистемы Цитус, могут не предлагать все указанные ниже параметры.

### <a name="general-configuration"></a>Общая конфигурация

#### <a name="citususe_secondary_nodes-enum"></a>Цитус. Использование \_ вторичных \_ узлов (enum)

Задает политику, используемую при выборе узлов для запросов SELECT. Если задано значение "Always", планировщик будет запрашивать только те узлы, которые помечены как "вторичные" нодероле в [pg_dist_node](reference-hyperscale-metadata.md#worker-node-table).

Для этого перечисления поддерживаются следующие значения:

-   **никогда:** (по умолчанию) все операции чтения происходят на первичных узлах.
-   **всегда:** Вместо этого операции чтения выполняются на вторичных узлах, а инструкции INSERT и Update отключаются.

#### <a name="cituscluster_name-text"></a>Цитус. \_ имя кластера (текст)

Информирует планировщик узлов координатора о том, какие кластеры он координирует. После \_ задания имени кластера планировщик будет запрашивать рабочие узлы только в этом кластере.

#### <a name="citusenable_version_checks-boolean"></a>Цитус. включить \_ \_ проверки версий (логический)

Для обновления версии Цитус требуется перезагрузка сервера (для выбора новой общей библиотеки), а затем команду ALTER EXTENSION UPDATE.  Сбой выполнения обоих шагов может привести к ошибкам или сбоям.
Таким же путем Цитус проверяет версию кода и совпадение расширения и выведет ошибки, если они не \' t.

По умолчанию это значение равно true и эффективно для координатора. В редких случаях сложные процессы обновления могут потребовать установки этого параметра в значение false, что приведет к отключению проверки.

#### <a name="cituslog_distributed_deadlock_detection-boolean"></a>\_Обнаружение распределенной взаимоблокировки Цитус. log \_ \_ (логическое значение)

Следует ли записывать в журнал на сервере обработку, связанную с обнаружением распределенной взаимоблокировкой. По умолчанию используется значение false.

#### <a name="citusdistributed_deadlock_detection_factor-floating-point"></a>Цитус. \_ коэффициент обнаружения распределенной взаимоблокировки \_ \_ (с плавающей запятой)

Задает время ожидания перед проверкой распределенных взаимоблокировок. В частности, время ожидания будет умножено на значение \' параметра [ \_ времени ожидания взаимоблокировки](https://www.postgresql.org/docs/current/static/runtime-config-locks.html) PostgreSQL s. Значение по умолчанию — `2`. Значение `-1` Отключает обнаружение распределенной взаимоблокировки.

#### <a name="citusnode_connection_timeout-integer"></a>\_время ожидания подключения Цитус. Node \_ (целое число)

`citus.node_connection_timeout`Гук задает максимальную продолжительность (в миллисекундах) ожидания установки соединения. Если время ожидания истекло до установки хотя бы одного рабочего подключения, то выдается ошибка Цитус. Эта Гук влияет на подключения от координатора к работникам и на друг друга.

-   По умолчанию: пять секунд
-   Минимум: 10 миллисекунд
-   Максимум: один час

```postgresql
-- set to 30 seconds
ALTER DATABASE foo
SET citus.node_connection_timeout = 30000;
```

### <a name="query-statistics"></a>Статистика запросов

#### <a name="citusstat_statements_purge_interval-integer"></a>\_интервал очистки инструкций Цитус. stat \_ \_ (целое число)

Задает частоту, с которой управляющая программа удаляет записи из [citus_stat_statements](reference-hyperscale-metadata.md#query-statistics-table) , несовпадающие в `pg_stat_statements` . Это значение конфигурации задает интервал времени между очистками в секундах со значением по умолчанию 10. Значение 0 отключает очистку.

```psql
SET citus.stat_statements_purge_interval TO 5;
```

Этот параметр эффективен для координатора и может быть изменен во время выполнения.

### <a name="data-loading"></a>Загрузка данных

#### <a name="citusmulti_shard_commit_protocol-enum"></a>Цитус. \_ \_ протокол фиксации нескольких сегментов \_ (enum)

Задает протокол фиксации, используемый при выполнении операции копирования в распределенной хэш-таблице. В каждом отдельном размещении сегментов копирование выполняется в блоке транзакций, чтобы предотвратить прием данных при возникновении ошибки во время копирования. Однако существует определенный случай сбоя, при котором копирование выполняется успешно во всех местах, но сбой (оборудование) происходит до фиксации всех транзакций. Этот параметр можно использовать для предотвращения потери данных в этом случае путем выбора между следующими протоколами фиксации:

-   **2pc:** (по умолчанию) транзакции, в которых выполняется копирование на размещения сегментов, сначала подготавливаются с помощью PostgreSQL \' s двухфазной [фиксации](http://www.postgresql.org/docs/current/static/sql-prepare-transaction.html) , а затем зафиксированы. Фиксации с ошибками могут быть восстановлены вручную или прерваны с помощью инструкции COMMIT PREPAREDd или ROLLBACK PREPARED соответственно.
    При использовании 2pc [Максимальное количество \_ подготовленных \_ транзакций](http://www.postgresql.org/docs/current/static/runtime-config-resource.html) должно быть увеличено на всех рабочих ролях, как правило, до тех же значений, что и для максимального числа \_ подключений.
-   **1pc:** Транзакции, в которых выполняется копирование в размещениях сегментов, фиксируются в одном цикле. Данные могут быть потеряны, если фиксация завершается сбоем после завершения копирования во всех местах (редких случаях).

#### <a name="citusshard_replication_factor-integer"></a>Коэффициент репликации Цитус. сегментирования \_ \_ (целое число)

Задает коэффициент репликации для сегментов, т. е. количество узлов, на которых будут размещены сегменты, и значение по умолчанию 1. Этот параметр можно задать во время выполнения и эффективно использовать в координаторе. Оптимальное значение для этого параметра зависит от размера кластера и частоты сбоя узла.  Например, может потребоваться увеличить этот коэффициент репликации при запуске больших кластеров и последующем наблюдении за сбоями узлов.

#### <a name="citusshard_count-integer"></a>Цитус. Сегментирование \_ (целое число)

Задает число сегментов для таблиц с хэшированием и значение по умолчанию 32.  Это значение используется определяемой пользователем функцией [create_distributed_table](reference-hyperscale-functions.md#create_distributed_table) при создании таблиц с хэшированным секционированием. Этот параметр можно задать во время выполнения и эффективно использовать в координаторе.

#### <a name="citusshard_max_size-integer"></a>максимальный размер Цитус. сегментирования \_ \_ (Integer)

Задает максимальный размер, до которого будет расти сегмент, прежде чем его разбиение и значение по умолчанию составляет 1 ГБ. Если размер исходного файла \' (который используется для промежуточного хранения) для одного сегмента превышает это значение конфигурации, база данных обеспечивает создание нового сегмента. Этот параметр можно задать во время выполнения и эффективно использовать в координаторе.

### <a name="planner-configuration"></a>Конфигурация планировщика

#### <a name="cituslimit_clause_row_fetch_count-integer"></a>Цитус. Limit \_ , \_ число выборок строк в предложении \_ \_ (целое число)

Задает количество строк для выборки в каждой задаче для оптимизации предложения LIMIT.
В некоторых случаях для создания результатов в запросах SELECT с предложениями Limit может потребоваться выборка всех строк из каждой задачи. В таких случаях, когда приближение выдает значимые результаты, это значение конфигурации задает количество строк для выборки из каждого сегмента. По умолчанию ограничения по порогам отключены, а для этого параметра устанавливается значение-1. Это значение может быть задано во время выполнения и эффективно для координатора.

#### <a name="cituscount_distinct_error_rate-floating-point"></a>Частота ошибок Цитус. Count \_ Distinct \_ \_ (с плавающей запятой)

Функция Scale (Цитус) позволяет вычислить приблизительные значения Count (DISTINCT) с помощью расширения PostgreSQL-ХЛЛ. Эта запись конфигурации задает требуемую частоту ошибок при вычислении числа (DISTINCT). 0,0, который является значением по умолчанию, отключает приближения для Count (DISTINCT); и 1,0 не предоставляет никаких гарантий относительно точности результатов. Рекомендуется задать для этого параметра значение 0,005, чтобы получить наилучшие результаты. Это значение может быть задано во время выполнения и эффективно для координатора.

#### <a name="citustask_assignment_policy-enum"></a>Цитус. \_ политика назначения задач \_ (enum)

> [!NOTE]
> Этот Гук применяется только в том случае, если [shard_replication_factor](reference-hyperscale-parameters.md#citusshard_replication_factor-integer) больше одного или для запросов к [reference_tables](concepts-hyperscale-distributed-data.md#type-2-reference-tables).

Задает политику, используемую при назначении задач рабочим работникам. Координатор назначает задачи рабочим работникам на основе расположений сегментов. Это значение конфигурации указывает политику, используемую при выполнении этих назначений.
В настоящее время можно использовать три политики назначения задач.

-   **жадная:** Жадная политика является по умолчанию и нацелена на равномерное распределение задач между рабочими процессами.
-   циклический **перебор:** Политика циклического перебора назначает задачи работникам циклическим перебором между разными репликами. Эта политика обеспечивает лучшую загрузку кластера, если число сегментов для таблицы мало по сравнению с числом рабочих ролей.
-   **Первая реплика:** Политика первой реплики назначает задачи на основе порядка вставки (реплик) для сегментов. Иными словами, запрос фрагмента для сегмента назначается рабочему процессу, который имеет первую реплику этого сегмента.
    Этот метод позволяет иметь строгие гарантии того, какие сегменты будут использоваться на каких узлах (то есть, более надежные гарантии местонахождение памяти).

Этот параметр можно задать во время выполнения и эффективно использовать в координаторе.

### <a name="intermediate-data-transfer"></a>Промежуточный Передача данных

#### <a name="citusbinary_worker_copy_format-boolean"></a>Цитус. двоичный \_ \_ Формат копии рабочей роли \_ (логический)

Используйте формат двоичного копирования для перемещения промежуточных данных между рабочими процессами.
Во время соединений больших таблиц Цитус может потребоваться динамически пересекционировать и перемешать данные между разными рабочими процессами. По умолчанию эти данные передаются в текстовом формате. Включение этого параметра дает указание базе данных использовать формат двоичной сериализации PostgreSQL для передачи этих данных. Этот параметр эффективен для рабочих ролей и должен быть изменен в файле PostgreSQL. conf. После редактирования файла конфигурации пользователи могут отправить сигнал СИГХУП или перезапустить сервер, чтобы это изменение вступило в силу.

#### <a name="citusbinary_master_copy_format-boolean"></a>\_Формат главной копии Цитус. Binary \_ \_ (логический)

Используйте формат двоичного копирования для перемещения данных между координатором и рабочими процессами. При выполнении распределенных запросов работники передают свои промежуточные результаты в координатор для окончательного агрегирования. По умолчанию эти данные передаются в текстовом формате. Включение этого параметра дает указание базе данных использовать формат двоичной сериализации PostgreSQL для передачи этих данных.
Этот параметр можно задать во время выполнения и эффективно использовать в координаторе.

#### <a name="citusmax_intermediate_result_size-integer"></a>Цитус. Max \_ промежуточный \_ \_ Размер результата (целое число)

Максимальный размер промежуточных результатов для CTE, которые не удается передать на рабочие узлы для выполнения, и для сложных вложенных запросов. Значением по умолчанию является 1 ГБ, а значение-1 означает отсутствие ограничения.
Запросы, превышающие ограничение, будут отменены и выдают сообщение об ошибке.

### <a name="ddl"></a>DDL

#### <a name="citusenable_ddl_propagation-boolean"></a>Цитус. включить \_ \_ распространение DDL (логическое значение)

Указывает, следует ли автоматически распространять изменения DDL от координатора всем рабочим процессам. Значение по умолчанию — true. Поскольку для некоторых изменений схемы требуется монопольная блокировка доступа к таблицам, а автоматическое распространение применяется ко всем рабочим процессам последовательно, это может привести к временному снижению скорости реагирования кластера (Цитус). Вы можете отключить этот параметр и распространить изменения вручную.

### <a name="executor-configuration"></a>Конфигурация исполнителя

#### <a name="general"></a>Общие сведения

##### <a name="citusall_modifications_commutative"></a>Цитус. все \_ изменения \_ коммутативной

Цитус обеспечивает применение правил учитывает коммутативность и получение подходящих блокировок для операций изменения, чтобы обеспечить правильную работу. Например, предполагается, что инструкция INSERT находится в другой инструкции INSERT, но не с инструкцией UPDATE или DELETE. Аналогичным образом предполагается, что инструкция UPDATE или DELETE не работает с другой инструкцией UPDATE или DELETE. Эта мера предосторожности означает, что для получения более надежных блокировок для операций обновления и удаления требуется масштабирование (Цитус).

Если у вас есть инструкции UPDATE, коммутативной с вашими вставками или другими обновлениями, то эти учитывает коммутативность предположения можно ослабить, установив для этого параметра значение true. Если этот параметр имеет значение true, все команды считаются коммутативной и задают общую блокировку, что может повысить общую пропускную способность. Этот параметр можно задать во время выполнения и эффективно использовать в координаторе.

##### <a name="citusremote_task_check_interval-integer"></a>интервал проверки Цитус. Remote \_ Task \_ \_ Interval (целое число)

Задает частоту, с которой масштабирование (Цитус) проверяет состояние заданий, управляемых исполнителем средства записи задач. По умолчанию используется значение 10 мс. Координатор назначает задачи рабочим работникам, а затем регулярно проверяет их с учетом \' хода выполнения каждой задачи. Это значение конфигурации задает интервал времени между двумя проверками последующий. Этот параметр эффективен для координатора и может быть установлен во время выполнения.

##### <a name="citustask_executor_type-enum"></a>тип исполнителя Цитус. Task \_ \_ (enum)

В масштабировании (Цитус) имеется три типа исполнителей для выполнения распределенных запросов SELECT.  Чтобы выбрать нужный исполнитель, установите этот параметр конфигурации. Допустимые значения для этого параметра:

-   **адаптивный:** Значение по умолчанию. Это оптимально для быстрых ответов на запросы, включающие агрегаты и сопоставленные объединения, охватывающие несколько сегментов.
-   **средство регистрации задач:** Исполнитель средства записи задач хорошо подходит для длительных выполнения сложных запросов, требующих перетасовывание данных между рабочими узлами и эффективного управления ресурсами.
-   в **режиме реального времени** (не рекомендуется) используется аналогичная цель адаптивного исполнителя, но она менее гибкая и может привести к большему нажиму на рабочие узлы.

Этот параметр можно задать во время выполнения и эффективно использовать в координаторе.

##### <a name="citusmulti_task_query_log_level-enum-multi_task_logging"></a>\_уровень журнала запросов Цитус. Multi Task \_ \_ \_ (enum) {#multi_task_logging}

Задает уровень ведения журнала для любого запроса, который создает более одной задачи (то есть, которая имеет более одного сегмента). Ведение журнала полезно при переносе многоклиентского приложения, так как вы можете выбрать ошибку или предупреждать о таких запросах, чтобы найти их и добавить \_ к ним фильтр идентификаторов клиентов. Этот параметр можно задать во время выполнения и эффективно использовать в координаторе. Значение по умолчанию для этого параметра — \' Off \' .

Для этого перечисления поддерживаются следующие значения:

-   **выкл.:** Отключите ведение журнала всех запросов, создающих несколько задач (то есть, охватывающих несколько сегментов).
-   **Отладка:** Регистрирует инструкцию на уровне серьезности отладки.
-   **Журнал:** Регистрирует инструкцию на уровне серьезности журнала. В строке журнала будет содержаться выполненный запрос SQL.
-   **Обратите внимание:** Регистрирует инструкцию на уровне серьезности заметьте.
-   **Предупреждение:** Регистрирует инструкцию на уровне серьезности предупреждения.
-   **Ошибка:** Регистрирует инструкцию на уровне серьезности ошибки.

Это может быть полезно `error` при тестировании разработки и на более низком уровне ведения журнала, например `log` во время реального рабочего развертывания.
При выборе команды `log` запросы с несколькими задачами будут отображаться в журналах базы данных с самим запросом, показанным после \" инструкции.\"

```
LOG:  multi-task query about to be executed
HINT:  Queries are split to multiple tasks if they have to be split into several queries on the workers.
STATEMENT:  select * from foo;
```

##### <a name="citusenable_repartition_joins-boolean"></a>Цитус. Включение \_ ресекционных \_ соединений (логическое значение)

Обычно попытка выполнить повторное секционирование соединений с помощью адаптивного исполнителя завершится ошибкой с сообщением об ошибке.  Однако если задано `citus.enable_repartition_joins` значение true, средство масштабирования (Цитус) позволяет временно переключиться на исполнителя средства записи задач для выполнения объединения.  Значением по умолчанию является false.

#### <a name="task-tracker-executor-configuration"></a>Конфигурация исполнителя средства записи задач

##### <a name="citustask_tracker_delay-integer"></a>Цитус. задержка в средстве \_ записи мониторинга задач \_ (целое число)

Этот параметр задает время ожидания для средства мониторинга задач между циклами управления задачами и значениями по умолчанию 200 мс. Процесс регистрации задач регулярно выходит из спящего режима, проходит по всем назначенным задачам, планирует и выполняет эти задачи.  Затем средство регистрации задач находится в спящем режиме за период времени, прежде чем снова проанализировать эти задачи.
Это значение конфигурации определяет продолжительность периода ожидания. Этот параметр эффективен для рабочих ролей и должен быть изменен в файле PostgreSQL. conf. После редактирования файла конфигурации пользователи могут отправить сигнал СИГХУП или перезапустить сервер, чтобы изменения вступили в силу.

Этот параметр можно уменьшить, чтобы обрезать задержку, вызванную выпуском средства записи задач, уменьшая промежуток времени между циклами управления.
Уменьшение задержки полезно в случаях, когда запросы сегмента короткие, и поэтому регулярно обновляют их состояние.

##### <a name="citusmax_assign_task_batch_size-integer"></a>Цитус. Max \_ назначение \_ \_ размера пакета задач \_ (целое число)

Исполнитель планировщика задач в координаторе синхронно назначает задачи пакетами управляющей программы на рабочих ролях. Этот параметр задает максимальное число задач для назначения в одном пакете. Выбор большего размера пакета позволяет быстрее назначать задачи. Тем не менее, если число рабочих ролей велико, для получения задач всем работникам может потребоваться больше времени.  Этот параметр можно задать во время выполнения и эффективно использовать в координаторе.

##### <a name="citusmax_running_tasks_per_node-integer"></a>Цитус. Максимальное число \_ выполняемых \_ задач \_ на \_ узле (Integer)

Процесс "средство записи задач" планирует и выполняет назначенные ему задачи в соответствии с соответствующими задачами. Это значение конфигурации задает максимальное число задач, выполняемых одновременно на одном узле в любое заданное время, и значение по умолчанию — 8.

Ограничение гарантирует, что у вас \' не будет большого количества задач, которые прибегают к диску, и помогает избежать конфликтов дискового ввода-вывода. Если запросы обслуживаются из памяти или твердотельных накопителей, вы можете увеличить максимальное количество \_ выполняемых \_ задач \_ на \_ узле без существенной необходимости.

##### <a name="cituspartition_buffer_size-integer"></a>Размер буфера Цитус. Partition \_ \_ (целое число)

Задает размер буфера, используемый для операций с секциями, и значение по умолчанию — 8 МБ.
Цитус позволяет повторно секционировать табличные данные в несколько файлов при соединении двух больших таблиц. После заполнения этого буфера секционирования повторно секционированные данные сбрасываются в файлы на диске.  Эта запись конфигурации может быть задана во время выполнения и эффективна для рабочих ролей.

#### <a name="explain-output"></a>Объяснить выходные данные

##### <a name="citusexplain_all_tasks-boolean"></a>Цитус. объяснить \_ все \_ задачи (логические)

По умолчанию при выполнении команды "Цитус" в процессе [объяснения](http://www.postgresql.org/docs/current/static/sql-explain.html) в распределенном запросе отображаются выходные данные одной произвольной задачи. В большинстве случаев выходные данные объяснения будут похожи на задачи. Иногда некоторые задачи будут планироваться по-разному или более высокие значения времени выполнения. В таких случаях может быть полезно включить этот параметр, после чего выходные данные объяснения будут включать все задачи. Объяснение всех задач может привести к тому, что объяснение займет больше времени.

## <a name="postgresql-parameters"></a>Параметры PostgreSQL

* [Датестиле](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-DATETIME-OUTPUT) — задает формат вывода значений даты и времени.
* [Интервалстиле](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-INTERVAL-OUTPUT) — задает формат вывода значений интервала.
* [TimeZone](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-TIMEZONE) — задает часовой пояс для отображения и интерпретации меток времени.
* [APPLICATION_NAME](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-APPLICATION-NAME) — задает имя приложения для отчетов в статистике и журналах.
* [array_nulls](https://www.postgresql.org/docs/current/runtime-config-compatible.html#GUC-ARRAY-NULLS) — включает ввод элементов NULL в массивах
* [Автоочистка](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html#GUC-AUTOVACUUM) — запускает подпроцесс автоочистки
* [autovacuum_analyze_scale_factor](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html#GUC-AUTOVACUUM-ANALYZE-SCALE-FACTOR) — число вставок, обновлений или удалений кортежей перед анализом как часть релтуплес
* [autovacuum_analyze_threshold](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html#GUC-AUTOVACUUM-ANALYZE-THRESHOLD) — минимальное число вставок, обновлений или удалений кортежей перед анализом
* [autovacuum_naptime](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html#GUC-AUTOVACUUM-NAPTIME) время перехода на спящий режим между запусками автоочистки
* задержка затрат [autovacuum_vacuum_cost_delay](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html#GUC-AUTOVACUUM-VACUUM-COST-DELAY) очистки (в миллисекундах) для автоочистки
* сумма затрат [autovacuum_vacuum_cost_limit](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html#GUC-AUTOVACUUM-VACUUM-COST-LIMIT) очистки, доступная до Наппинг, для автоочистки
* [autovacuum_vacuum_scale_factor](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html#GUC-AUTOVACUUM-VACUUM-SCALE-FACTOR) — число обновлений или удалений кортежей до очистки в виде доли релтуплес
* [autovacuum_vacuum_threshold](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html#GUC-AUTOVACUUM-VACUUM-THRESHOLD) — минимальное число обновлений или удалений кортежей перед очисткой
* [autovacuum_work_mem](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-AUTOVACUUM-WORK-MEM) — задает максимальный объем памяти, используемый каждым рабочим процессом автоочистки.
* [backend_flush_after](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-BACKEND-FLUSH-AFTER) — число страниц, после которых ранее выполненные операции записи сбрасываются на диск
* [backslash_quote](https://www.postgresql.org/docs/current/runtime-config-compatible.html#GUC-BACKSLASH-QUOTE) — указывает \' , разрешено ли использование "" в строковых литералах
* [bgwriter_delay](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-BGWRITER-DELAY) — время ожидания в фоновом режиме для промежутка между циклами
* [bgwriter_flush_after](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-BGWRITER-FLUSH-AFTER) — число страниц, после которых ранее выполненные операции записи сбрасываются на диск
* [bgwriter_lru_maxpages](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-BGWRITER-LRU-MAXPAGES) — максимальное число страниц LRU для записи на диск в фоновом режиме для каждого цикла.
* [bgwriter_lru_multiplier](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-BGWRITER-LRU-MULTIPLIER) -кратное среднее использование буфера для освобождения в цикле
* [bytea_output](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-BYTEA-OUTPUT) — задает формат вывода для Byte
* [check_function_bodies](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-CHECK-FUNCTION-BODIES) — проверяет тела функций во время создания функции
* [checkpoint_completion_target](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-CHECKPOINT-COMPLETION-TARGET) время, затраченное на сброс "грязных" буферов во время контрольной точки, в качестве доли интервала контрольной точки
* [checkpoint_timeout](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-CHECKPOINT-TIMEOUT) — задает максимальное время между автоматическими контрольными точками Wal
* [checkpoint_warning](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-CHECKPOINT-WARNING) — включает предупреждения, если сегменты контрольных точек заполняются чаще, чем это
* [client_encoding](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-CLIENT-ENCODING) — задает кодировку набора символов клиента.
* [client_min_messages](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-CLIENT-MIN-MESSAGES) — задает уровни сообщений, отправляемые клиенту.
* [commit_delay](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-COMMIT-DELAY) — задает задержку в микросекундах между фиксацией транзакции и записью на диск Wal
* [commit_siblings](https://www.postgresql.org/docs/12/runtime-config-wal.html#GUC-COMMIT-SIBLINGS) — задает минимальное число параллельно открытых транзакций перед выполнением commit_delay
* [constraint_exclusion](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-CONSTRAINT-EXCLUSION) — позволяет планировщику использовать ограничения для оптимизации запросов.
* [cpu_index_tuple_cost](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-CPU-INDEX-TUPLE-COST) — задает оценку затрат на обработку каждой записи индекса при просмотре индекса в планировщике.
* [cpu_operator_cost](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-CPU-OPERATOR-COST) — задает оценку затрат на обработку каждого оператора или вызова функции в планировщике.
* [cpu_tuple_cost](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-CPU-TUPLE-COST) — задает оценку затрат на обработку каждого кортежа в планировщике (строка).
* [cursor_tuple_fraction](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-CURSOR-TUPLE-FRACTION) — задает оценку в планировщике доли строк курсора, которые будут извлечены
* [deadlock_timeout](https://www.postgresql.org/docs/current/runtime-config-locks.html#GUC-DEADLOCK-TIMEOUT) — задает время ожидания блокировки в миллисекундах перед проверкой на наличие взаимоблокировки.
* [debug_pretty_print](https://www.postgresql.org/docs/current/runtime-config-logging.html#id-1.6.6.11.5.2.3.1.3) -отступы синтаксический анализ и отображение дерева плана
* [debug_print_parse](https://www.postgresql.org/docs/current/runtime-config-logging.html#id-1.6.6.11.5.2.2.1.3) — записывает дерево синтаксического анализа каждого запроса
* [debug_print_plan](https://www.postgresql.org/docs/current/runtime-config-logging.html#id-1.6.6.11.5.2.2.1.3) — записывает в журнал план выполнения каждого запроса.
* [debug_print_rewritten](https://www.postgresql.org/docs/current/runtime-config-logging.html#id-1.6.6.11.5.2.2.1.3) — записывает дерево синтаксического анализа всех перезаписанных запросов
* [default_statistics_target](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-DEFAULT-STATISTICS-TARGET) — задает целевой объект статистики по умолчанию.
* [default_tablespace](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-DEFAULT-TABLESPACE) — задает табличное пространство по умолчанию для создания таблиц и индексов в
* [default_text_search_config](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-DEFAULT-TEXT-SEARCH-CONFIG) — задает конфигурацию текстового поиска по умолчанию
* [default_transaction_deferrable](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-DEFAULT-TRANSACTION-DEFERRABLE) — задает состояние задержку по умолчанию для новых транзакций.
* [default_transaction_isolation](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-DEFAULT-TRANSACTION-ISOLATION) — задает уровень изоляции транзакции для каждой новой транзакции.
* [default_transaction_read_only](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-DEFAULT-TRANSACTION-READ-ONLY) — задает состояние только для чтения новых транзакций по умолчанию.
* default_with_oids — создает новые таблицы с OID по умолчанию.
* [effective_cache_size](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-EFFECTIVE-CACHE-SIZE) — задает предположение планировщика относительно размера кэша диска.
* [enable_bitmapscan](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-BITMAPSCAN) — позволяет планировщику использовать планы растрового сканирования.
* [enable_gathermerge](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-GATHERMERGE) — включает использование планировщика для сбора планов слияния
* [enable_hashagg](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-HASHAGG) — включает использование планировщика для хэшированных планов агрегирования
* [enable_hashjoin](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-HASHJOIN) — позволяет планировщику использовать планы хэш-соединений.
* [enable_indexonlyscan](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-INDEXONLYSCAN) — включает использование планировщиком планов просмотра только индексов
* [enable_indexscan](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-INDEXSCAN) — включает использование планировщиком планов сканирования индекса.
* [enable_material](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-MATERIAL) — позволяет планировщику использовать материализация
* [enable_mergejoin](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-MERGEJOIN) — позволяет планировщику использовать планы соединений слиянием.
* [enable_nestloop](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-NESTLOOP) — позволяет планировщику использовать планы соединений вложенного цикла.
* [enable_seqscan](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-SEQSCAN) — позволяет планировщику использовать планы последовательного просмотра.
* [enable_sort](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-SORT) — позволяет планировщику использовать явные шаги сортировки.
* [enable_tidscan](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-TIDSCAN) — позволяет планировщику использовать планы сканирования TID
* [escape_string_warning](https://www.postgresql.org/docs/current/runtime-config-compatible.html#GUC-ESCAPE-STRING-WARNING) — предупреждает о escape-символах обратной косой черты в обычных строковых литералах
* [exit_on_error](https://www.postgresql.org/docs/current/runtime-config-error-handling.html#GUC-EXIT-ON-ERROR) — завершает сеанс при любой ошибке
* [extra_float_digits](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-EXTRA-FLOAT-DIGITS) — задает количество цифр, отображаемых для значений с плавающей запятой
* [force_parallel_mode](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-FORCE-PARALLEL-MODE) — принудительное использование средств параллельных запросов
* [from_collapse_limit](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-FROM-COLLAPSE-LIMIT) — ЗАДАЕТ размер из списка, после которого вложенные запросы не сворачиваются.
* [жеко](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-GEQO) — включает оптимизацию запросов генетических
* [geqo_effort](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-GEQO-EFFORT) -жеко: усилия используются для задания значений по умолчанию для других параметров жеко
* [geqo_generations](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-GEQO-GENERATIONS) -жеко: число итераций алгоритма
* [geqo_pool_size](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-GEQO-POOL-SIZE) -жеко: число пользователей в совокупности
* [geqo_seed](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-GEQO-SEED) -жеко: начальное значение для выбора случайного пути
* [geqo_selection_bias](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-GEQO-SELECTION-BIAS) -жеко: Выборочная нагрузка в пределах заполнения
* [geqo_threshold](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-GEQO-THRESHOLD) — задает пороговое значение для элементов, помимо которых используется жеко
* [gin_fuzzy_search_limit](https://www.postgresql.org/docs/current/runtime-config-client.html#id-1.6.6.14.5.2.2.1.3) — задает максимально допустимый результат для точного поиска по Ло.
* [gin_pending_list_limit](https://www.postgresql.org/docs/current/runtime-config-client.html#id-1.6.6.14.2.2.23.1.3) — задает максимальный размер ожидающего списка для индекса Ло.
* [idle_in_transaction_session_timeout](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-IDLE-IN-TRANSACTION-SESSION-TIMEOUT) — задает максимально допустимую длительность любой транзакции состояние простоя.
* [join_collapse_limit](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-JOIN-COLLAPSE-LIMIT) — ЗАДАЕТ размер из списка, за пределами которого не выполняется СВЕДЕНИЕ конструкций объединения
* [LC_MONETARY](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-LC-MONETARY) — задает языковой стандарт для форматирования денежных сумм
* [LC_NUMERIC](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-LC-NUMERIC) — задает языковой стандарт для форматирования чисел
* [lo_compat_privileges](https://www.postgresql.org/docs/current/runtime-config-compatible.html#GUC-LO-COMPAT-PRIVILEGES) — включает режим обратной совместимости для проверок прав доступа к большим объектам.
* [LOCK_TIMEOUT](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-LOCK-TIMEOUT) — задает максимально допустимую длительность (в миллисекундах) любого ожидания блокировки. 0 — отключение
* [log_autovacuum_min_duration](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html#) — задает минимальное время выполнения, по истечении которого будут регистрироваться действия автоочистки.
* [log_checkpoints](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-CHECKPOINTS) — записывает каждую контрольную точку.
* [log_connections](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-CONNECTIONS) — регистрирует все успешные подключения.
* [log_destination](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-DESTINATION) — задает назначение для вывода журнала сервера.
* [log_disconnections](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-DISCONNECTIONS) — записывает в журнал конец сеанса, включая длительность
* [log_duration](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-DURATION) — регистрирует длительность выполнения каждой выполненной инструкции SQL
* [log_error_verbosity](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-ERROR-VERBOSITY) — задает уровень детализации сообщений в журнале
* [log_lock_waits](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-LOCK-WAITS) — записывает задержки длительных блокировок
* [log_min_duration_statement](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-MIN-DURATION-STATEMENT) — задает минимальное время выполнения (в миллисекундах), по истечении которого будут регистрироваться инструкции. -1 отключает ведение журнала длительности выполнения инструкций
* [log_min_error_statement](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-MIN-ERROR-STATEMENT) — приводит к записи в журнал всех инструкций, создающих ошибку на этом уровне или выше.
* [log_min_messages](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-MIN-MESSAGES) — задает уровни сообщений, которые заносятся в журнал
* [log_replication_commands](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-REPLICATION-COMMANDS) — регистрирует каждую команду репликации.
* [log_statement](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-STATEMENT) — задает тип регистрируемых инструкций
* [log_statement_stats](https://www.postgresql.org/docs/current/runtime-config-statistics.html#id-1.6.6.12.3.2.1.1.3) — для каждого запроса записывает общую статистику производительности в журнал сервера.
* [log_temp_files](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-TEMP-FILES) — заносит в журнал использование временных файлов, размер которых превышает это значение (в килобайтах).
* [maintenance_work_mem](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-MAINTENANCE-WORK-MEM) — задает максимальный объем памяти, используемый для операций обслуживания.
* [max_parallel_workers](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-MAX-PARALLEL-WORKERS) — задает максимальное число параллельных рабочих ролей, которое может быть активно в один момент времени.
* [max_parallel_workers_per_gather](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-MAX-PARALLEL-WORKERS-PER-GATHER) — задает максимальное число параллельных процессов на один узел исполнителя.
* [max_pred_locks_per_page](https://www.postgresql.org/docs/current/runtime-config-locks.html#GUC-MAX-PRED-LOCKS-PER-PAGE) — задает максимальное число кортежей, заблокированных предикатом на страницу
* [max_pred_locks_per_relation](https://www.postgresql.org/docs/current/runtime-config-locks.html#GUC-MAX-PRED-LOCKS-PER-RELATION) — задает максимальное число страниц с блокировкой предиката и кортежей по связям.
* [max_standby_archive_delay](https://www.postgresql.org/docs/current/runtime-config-replication.html#GUC-MAX-STANDBY-ARCHIVE-DELAY) — задает максимальную задержку перед отменой запросов, когда сервер горячего резервирования обрабатывает архивные данные Wal
* [max_standby_streaming_delay](https://www.postgresql.org/docs/current/runtime-config-replication.html#GUC-MAX-STANDBY-STREAMING-DELAY) — задает максимальную задержку перед отменой запросов, когда сервер горячего резервирования обрабатывает потоковые данные Wal
* [max_sync_workers_per_subscription](https://www.postgresql.org/docs/current/runtime-config-replication.html#GUC-MAX-SYNC-WORKERS-PER-SUBSCRIPTION) -максимальное число рабочих ролей синхронизации таблиц на подписку
* [max_wal_size](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-MAX-WAL-SIZE) — ЗАДАЕТ размер Wal, который запускает контрольную точку.
* [min_parallel_index_scan_size](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-MIN-PARALLEL-INDEX-SCAN-SIZE) — задает минимальный объем данных индекса для параллельного сканирования.
* [min_wal_size](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-MIN-WAL-SIZE) — устанавливает минимальный размер для сжатия Wal до
* [operator_precedence_warning](https://www.postgresql.org/docs/current/runtime-config-compatible.html#GUC-OPERATOR-PRECEDENCE-WARNING) — выдает предупреждение для конструкций, которые изменили это значение с момента PostgreSQL 9,4
* [parallel_setup_cost](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-PARALLEL-SETUP-COST) — определяет оценку затрат на запуск рабочих процессов в планировщике для параллельных запросов
* [parallel_tuple_cost](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-PARALLEL-TUPLE-COST) — задает оценку стоимости планировщика для передачи каждого кортежа (строки) от рабочей роли до главной серверной части.
* [pg_stat_statements. Save](https://www.postgresql.org/docs/current/pgstatstatements.html#id-1.11.7.38.8) — сохранение статистики pg_stat_statements по завершении работы сервера
* [pg_stat_statements. Track](https://www.postgresql.org/docs/current/pgstatstatements.html#id-1.11.7.38.8) — выбор инструкций, которые отправляются pg_stat_statements
* [pg_stat_statements. track_utility](https://www.postgresql.org/docs/current/pgstatstatements.html#id-1.11.7.38.8) — выбирает, будут ли команды программы отслеживаниться с помощью pg_stat_statements
* [quote_all_identifiers](https://www.postgresql.org/docs/current/runtime-config-compatible.html#GUC-QUOTE-ALL-IDENTIFIERS) — при создании фрагментов SQL, в кавычках все идентификаторы
* [random_page_cost](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-RANDOM-PAGE-COST) — задает оценку стоимости планировщика для страницы непоследовательно извлекаемого диска
* [row_security](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-ROW-SECURITY) — обеспечивает безопасность строк
* [search_path](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-SEARCH-PATH) — задает порядок поиска в схеме для имен, которые не являются полными схемами
* [seq_page_cost](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-SEQ-PAGE-COST) — задает оценку затрат на последовательное получение диска в планировщике
* [session_replication_role](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-SESSION-REPLICATION-ROLE) — задает поведение сеанса для триггеров и правил перезаписи.
* [standard_conforming_strings](https://www.postgresql.org/docs/current/runtime-config-compatible.html#id-1.6.6.16.2.2.7.1.3) — причины "..." строки для интерпретации обратных косых черт буквально
* [statement_timeout](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-STATEMENT-TIMEOUT) — задает максимально допустимую длительность (в миллисекундах) любой инструкции. 0 — отключение
* [synchronize_seqscans](https://www.postgresql.org/docs/current/runtime-config-compatible.html#id-1.6.6.16.2.2.8.1.3) — включает синхронизированные последовательные Просмотры
* [SYNCHRONOUS_COMMIT](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-SYNCHRONOUS-COMMIT) — задает уровень синхронизации для текущей транзакции.
* [tcp_keepalives_count](https://www.postgresql.org/docs/current/runtime-config-connection.html#GUC-TCP-KEEPALIVES-COUNT) — максимальное число повторных подлинности TCP KeepAlive
* [tcp_keepalives_idle](https://www.postgresql.org/docs/current/runtime-config-connection.html#GUC-TCP-KEEPALIVES-IDLE) время между выдачей TCP-проверку активности
* время [tcp_keepalives_interval](https://www.postgresql.org/docs/current/runtime-config-connection.html#GUC-TCP-KEEPALIVES-INTERVAL) между повторными передачами проверки активности TCP
* [temp_buffers](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-TEMP-BUFFERS) — задает максимальное количество временных буферов, используемых каждым сеансом базы данных.
* [temp_tablespaces](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-TEMP-TABLESPACES) — задает табличные пространства, используемые для временных таблиц и файлов сортировки.
* [track_activities](https://www.postgresql.org/docs/current/runtime-config-statistics.html#GUC-TRACK-ACTIVITIES) — собирает сведения о выполняющихся командах
* [track_counts](https://www.postgresql.org/docs/current/runtime-config-statistics.html#GUC-TRACK-COUNTS) — собирает статистику по активности базы данных
* [track_functions](https://www.postgresql.org/docs/current/runtime-config-statistics.html#GUC-TRACK-FUNCTIONS) — собирает статистические данные об активности базы данных на уровне функций
* [track_io_timing](https://www.postgresql.org/docs/current/runtime-config-statistics.html#GUC-TRACK-IO-TIMING) — собирает статистику времени для операций ввода-вывода базы данных.
* [transform_null_equals](https://www.postgresql.org/docs/current/runtime-config-compatible.html#GUC-TRANSFORM-NULL-EQUALS) — рассматривает "expr = null" как "expr null"
* задержка затрат [vacuum_cost_delay](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-VACUUM-COST-DELAY) чистильщика в миллисекундах
* объем [vacuum_cost_limit](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-VACUUM-COST-LIMIT) -очистки, доступный до Наппинг
* затраты [vacuum_cost_page_dirty](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-VACUUM-COST-PAGE-DIRTY) на чистильщик для страницы, изменялся по вакууму
* Стоимость [vacuum_cost_page_hit](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-VACUUM-COST-PAGE-HIT) -чистильщика для страницы, найденной в буферном кэше
* [vacuum_cost_page_miss](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-VACUUM-COST-PAGE-MISS) -вакуумные затраты на страницу, не найденную в буферном кэше
* [vacuum_defer_cleanup_age](https://www.postgresql.org/docs/current/runtime-config-replication.html#GUC-VACUUM-DEFER-CLEANUP-AGE) — количество транзакций, по которым должна быть отложена очистка и горячая очистка, если они есть
* [vacuum_freeze_min_age](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-VACUUM-FREEZE-MIN-AGE) — минимальный возраст, по которому чистильщик должен заморозить строку таблицы
* [vacuum_freeze_table_age](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-VACUUM-FREEZE-TABLE-AGE) — возраст, по которому чистильщик должен сканировать всю таблицу для закрепления кортежей.
* [vacuum_multixact_freeze_min_age](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-VACUUM-MULTIXACT-FREEZE-MIN-AGE) — минимальный возраст, при котором чистильщик должен заморозить мултиксактид в строке таблицы
* [vacuum_multixact_freeze_table_age](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-VACUUM-MULTIXACT-FREEZE-TABLE-AGE) мултиксакт Age, при котором чистильщик должен сканировать всю таблицу для закрепления кортежей.
* [wal_receiver_status_interval](https://www.postgresql.org/docs/current/runtime-config-replication.html#GUC-WAL-RECEIVER-STATUS-INTERVAL) — задает максимальный интервал между отчетами о состоянии приемника Wal для основного
* [wal_writer_delay](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-WAL-WRITER-DELAY) времени между очистками Wal, выполненными в модуле записи Wal
* [wal_writer_flush_after](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-WAL-WRITER-FLUSH-AFTER) -объем Wal, записанный модулем записи Wal, который запускает сброс
* [work_mem](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-WORK-MEM) — задает объем памяти, используемый внутренними операциями сортировки и хэш-таблицами перед записью во временные файлы диска.
* [ксмлбинари](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-XMLBINARY) — задает способ кодирования двоичных значений в формате XML
* [ксмлоптион](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-XMLOPTION) — указывает, следует ли рассматривать XML-данные в операциях неявного анализа и сериализации как документы или фрагменты содержимого.

## <a name="next-steps"></a>Дальнейшие действия

* Другой формой конфигурации, помимо параметров сервера, являются [Параметры конфигурации](concepts-hyperscale-configuration-options.md) ресурсов в группе серверов Цитус.
* Базовая база данных PostgreSQL также имеет [Параметры конфигурации](http://www.postgresql.org/docs/current/static/runtime-config.html).
