---
title: Настройка производительности с помощью материализованных представлений
description: Рекомендации и вопросы по материализованным представлениям для повышения производительности запросов.
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: xiaoyul
ms.reviewer: nibruno; jrasnick
ms.openlocfilehash: e6c3987e2de7f9592a1f7f6086657592e1bf0c16
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101676596"
---
# <a name="performance-tuning-with-materialized-views-using-dedicated-sql-pool-in-azure-synapse-analytics"></a>Настройка производительности с материализованными представлениями с помощью выделенного пула SQL в Azure синапсе Analytics

В выделенном пуле SQL материализованные представления обеспечивают низкий уровень обслуживания сложных аналитических запросов, что позволяет добиться высокой производительности без изменения запроса. В этой статье рассматриваются общие рекомендации по использованию материализованных представлений.

## <a name="materialized-views-vs-standard-views"></a>Материализованные представления и стандартные представления

Пул SQL поддерживает как стандартные, так и материализованные представления.  И те, и другие являются виртуальными таблицами, созданными с помощью выражений SELECT и представленными для запросов в виде логических таблиц.  Представления показывают сложность общих вычислений данных и добавляют уровень абстракции к вычислительным изменениям, поэтому нет необходимости переписывать запросы.  

Стандартное представление рассчитывает данные при каждом использовании представления.  Данные не хранятся на диске. Обычно пользователи используют стандартные представления в качестве средства, помогающего организовать логические объекты и запросы в базе данных.  Чтобы использовать стандартное представление, запрос должен явно сослаться на него.

Материализованные представления предварительно вычисляются, сохраняют и сохраняют свои данные в выделенном пуле SQL так же, как и таблица.  При использовании материализованных представлений повторное вычисление не требуется.  Именно поэтому запросы, использующие все или подмножество данных в материализованных представлениях, могут повысить производительность.  Более того, запросы могут использовать материализованные представления, не используя явные ссылки на них, поэтому нет необходимости изменять код приложения.  

Большинство требований к стандартным представлениям применимо и к материализованным представлениям. Дополнительные сведения о синтаксисе материализованных представлений и других требованиях см. в статье о выражении [CREATE MATERIALIZED VIEW AS SELECT](/sql/t-sql/statements/create-materialized-view-as-select-transact-sql?view=azure-sqldw-latest&preserve-view=true).

| Сравнение                     | Представление                                         | Материализованное представление
|:-------------------------------|:---------------------------------------------|:--------------------------------------------------------------|
|Просмотр определений                 | Хранится в хранилище данных Azure.              | Хранится в хранилище данных Azure.
|Содержимое представления                    | Создается каждый раз при использовании представления.   | Обрабатывается предварительно и сохраняется в хранилище данных Azure при создании представления. Обновляется по мере добавления данных в базовые таблицы.
|Обновление данных                    | Всегда актуальны                               | Всегда актуальны
|Скорость получения данных представления из сложных запросов     | Медленно                                         | быстрый;  
|Дополнительный объем хранилища                   | Нет                                           | Да
|Синтаксис                          | CREATE VIEW                                  | CREATE MATERIALIZED VIEW AS SELECT

## <a name="benefits-of-materialized-views"></a>Преимущества материализованных представлений

Правильно спроектированное материализованное представление предоставляет следующие преимущества.

- Сокращение времени выполнения сложных запросов с операторами JOIN и агрегатными функциями. Чем сложнее запрос, тем потенциально больше времени удастся сэкономить. Наибольшее преимущество достигается, когда вычислительные затраты запроса высоки, а результирующий набор данных мал.  

- Оптимизатор запросов в выделенном пуле SQL может автоматически использовать развернутые материализованные представления для улучшения планов выполнения запросов.  Этот процесс является прозрачным для пользователей, обеспечивает более высокую производительность запросов и не требует указания прямой ссылки на материализованные представления.

- Требует минимального обслуживания представлений.  Материализованное представление сохраняет данные в двух местах: кластеризованный индекс columnstore для начальных данных при создании представления и разностное хранилище для добавочных изменений данных.  Все изменения данных из базовых таблиц автоматически добавляются в разностное хранилище синхронным образом.  Фоновый процесс (перемещатель кортежей) периодически перемещает данные из разностного хранилища в индекс columnstore представления.  Такая схема позволяет запросам из материализованных представлений возвращать те же данные, что и при непосредственном обращении к базовым таблицам.
- Данные в материализованном представлении могут распределяться иначе, чем в базовых таблицах.  
- Данные в материализованных представлениях получают те же преимущества высокой доступности и устойчивости, что и данные в обычных таблицах.  

По сравнению с другими поставщиками хранилища данных материализованные представления, реализованные в выделенном пуле SQL, также предоставляют следующие дополнительные преимущества:

- Автоматическое и синхронное обновление данных при их изменении в базовых таблицах. Вмешательство пользователя не требуется.
- Широкая поддержка агрегатных функций. См. статью [CREATE MATERIALIZED VIEW AS SELECT (Transact-SQL)](/sql/t-sql/statements/create-materialized-view-as-select-transact-sql?view=azure-sqldw-latest&preserve-view=true).
- Поддержка рекомендаций по материализованным представлениям для конкретных запросов.  См. статью [EXPLAIN (Transact-SQL)](/sql/t-sql/queries/explain-transact-sql?view=azure-sqldw-latest&preserve-view=true).

## <a name="common-scenarios"></a>Распространенные сценарии  

Материализованные представления обычно используются в следующих сценариях.

**Требуется повышение производительности сложных аналитических запросов к большим объемам данных**

Сложные аналитические запросы обычно используют больше агрегатных функций и соединений таблиц, что приводит при выполнении запроса к большим объемам вычислительных операций, таких как перемешивание и соединение.  Именно поэтому выполнение таких запросов занимает больше времени, особенно на больших таблицах.  

Пользователи могут создавать материализованные представления для данных, возвращаемых из общих вычислений запросов, поэтому нет необходимости выполнять перерасчеты, если эти данные понадобятся для запросов, что позволяет снизить вычислительные затраты и уменьшить отклики на запросы.

**Требуется более высокая производительность при отсутствии или минимальных изменениях запроса**

Изменения схемы и запросов в хранилищах данных обычно хранятся как минимум для поддержки обычных операций ETL и создания отчетов.  Пользователи могут использовать материализованные представления для настройки производительности запросов, если затраты, связанные с представлениями, могут быть компенсированы за счет повышения производительности запросов.

В сравнении с другими вариантами настройки, такими как масштабирование и управление статистикой, создание и поддержание материализованного представления требует гораздо меньших изменений рабочей среды, обеспечивая при этом больший потенциальный прирост производительности.

- Создание и обслуживание материализованных представлений не влияет на запросы, выполняемые к базовым таблицам.
- Оптимизатор запросов может автоматически использовать развернутые материализованные представления без прямых ссылок на них в запросе. Эта возможность снижает необходимость изменений запроса при настройке производительности.

**Требуется другая стратегия распределения данных для повышения производительности запросов**

Хранилище данных Azure является распределенной системой с массовой параллельной обработкой (MPP).  

Synapse SQL — это система распределенных запросов, которая позволяет предприятиям реализовать сценарии хранения и виртуализации данных с помощью стандартных возможностей T-SQL, знакомых специалистам по работе с данными. Кроме того, решение расширяет возможности SQL для использования в сценариях потоковой передачи данных и машинного обучения. Данные в таблице хранилища данных распределяются между 60 узлами, используя одну из трех [стратегий распределения](../sql-data-warehouse/sql-data-warehouse-tables-distribute.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) (хэширование, циклический перебор или репликация).  

Распределение данных задается в момент создания таблицы и остается неизменным до ее удаления. Материализованные представления, будучи виртуальными таблицами на диске, поддерживают распределение данных хэшированием и циклическим перебором.  Пользователи могут выбрать распределение данных, отличное от базовых таблиц, но оптимально подходящее для производительности запросов, которые часто используют представления.  

## <a name="design-guidance"></a>Руководство по проектированию

Ниже приведены общие рекомендации по использованию материализованных представлений для повышения производительности запросов.

**Проектируйте для своей рабочей нагрузки**

Прежде чем приступать к созданию материализованных представлений, важно иметь глубокое понимание рабочей нагрузки с точки зрения шаблонов запросов, важности, частоты и размера результирующих данных.  

Пользователи могут выполнять выражения EXPLAIN WITH_RECOMMENDATIONS <выражение_SQL> для материализованных представлений, рекомендованных оптимизатором запросов.  Так как эти рекомендации относятся к конкретным запросам, материализованное представление, полезное для одного запроса, может быть неоптимальным для других запросов в той же рабочей нагрузке.  

Оценивайте эти рекомендации с учетом потребностей вашей рабочей нагрузки.  Идеальными материализованными представлениями являются те, которые приводят к повышению производительности данной рабочей нагрузки.  

**Не забывайте о компромиссе между скоростью выполнения запросов и затратами**

Каждое материализованное представление связано с затратами на хранение данных и обслуживание представления.  При изменении данных в базовых таблицах размер материализованных представлений увеличивается, а также изменяется их физическая структура.  

Чтобы избежать снижения производительности запросов, каждое материализованное представление обслуживается подсистемой хранилища данных отдельно, включая перемещение строк из разностного хранилища в сегменты индекса columnstore и консолидацию изменений данных.  

Рабочая нагрузка обслуживания увеличивается при увеличении числа материализованных представлений и изменений базовой таблицы.   Пользователи должны проверить, могут ли затраты на все материализованные представления быть компенсированы повышением производительности запросов.  

Для получения списка материализованных представлений в базе данных можно выполнить следующий запрос.

```sql
SELECT V.name as materialized_view, V.object_id
FROM sys.views V
JOIN sys.indexes I ON V.object_id= I.object_id AND I.index_id < 2;
```

Способы сокращения числа материализованных представлений.

- Выявите общие наборы данных, часто используемые сложными запросами в рабочей нагрузке.  Создайте материализованные представления для хранения этих наборов данных, чтобы оптимизатор мог использовать их в качестве стандартных блоков при создании планов выполнения.  

- Удалите материализованные представления, которые редко используются или больше не нужны.  Отключенные материализованные представления не обслуживаются, но по-прежнему требуют затрат на хранение.  

- Объедините материализованные представления, созданные на основе тех же или аналогичных базовых таблиц, даже если их данные не пересекаются.  Объединение материализованных представлений может привести к большему размеру представления, чем сумма отдельных представлений, однако затраты на обслуживание представления должны сократиться.  Пример:

```sql
-- Query 1 would benefit from having a materialized view created with this SELECT statement
SELECT A, SUM(B)
FROM T
GROUP BY A

-- Query 2 would benefit from having a materialized view created with this SELECT statement
SELECT C, SUM(D)
FROM T
GROUP BY C

-- You could create a single mateiralized view of this form
SELECT A, C, SUM(B), SUM(D)
FROM T
GROUP BY A, C

```

**Не все настройки производительности требуют изменения запроса**

Оптимизатор хранилища данных может автоматически использовать развернутые материализованные представления для повышения производительности запросов.  Эта поддержка прозрачна для запросов, которые не ссылаются на представления, и запросов, использующих статистические выражения, неподдерживаемые при создании материализованных представлений.  Изменение запроса не требуется. Вы можете просмотреть предполагаемый план выполнения запроса, чтобы проверить использование материализованных представлений.  

**Мониторинг материализованных представлений**

Материализованные представления хранятся в хранилище данных точно так же, как таблица с кластеризованным индексом columnstore (CCI).  Чтение данных из материализованных представлений включает сканирование индекса и применение изменений из разностного хранилища.  Если число строк в разностном хранилище слишком велико, разрешение запроса из материализованных представлений может занять больше времени, чем непосредственное выполнение запросов к базовым таблицам.  

Чтобы избежать снижения производительности запросов, рекомендуется выполнять запрос [DBCC PDW_SHOWMATERIALIZEDVIEWOVERHEAD](/sql/t-sql/database-console-commands/dbcc-pdw-showmaterializedviewoverhead-transact-sql?view=azure-sqldw-latest&preserve-view=true) для отслеживания коэффициента_дополнительных_затрат представления (строк_всего/строк_в_базовом_представлении).  Если коэффициент_дополнительных_затрат слишком высок, рассмотрите возможность перестроения материализованных представлений, чтобы все строки в разностном хранилище переместились в индекс columnstore.  

**Материализованные представления и кэширование результирующего набора**

Эти две функции представлены в выделенном пуле SQL примерно в одно и то же время для настройки производительности запросов. Кэширование результирующего набора используется для достижения высокой степени параллелизма и быстрого ответа на повторяющиеся запросы к статическим данным.  

Чтобы использовать кэшированный результат, форма запроса кэша должна совпадать с запросом, который создал кэш.  Кроме того, кэшированный результат должен применяться ко всему запросу.  

Материализованные представления поддерживают изменение данных в базовых таблицах.  Данные в материализованных представлениях могут быть применены к части запроса.  Эти преимущества позволяют использовать одни и те же материализованные представления для разных запросов, которые совместно используют некоторые вычисления для повышения производительности.

## <a name="example"></a>Пример

В этом примере используется TPCDS-подобный запрос, который ищет клиентов, тратящих больше денег через каталог, чем в магазинах. Он также определяет предпочтительных клиентов и их страну или регион происхождения.   Запрос включает выбор первых 100 записей из объединения трех вложенных инструкций SELECT, включающих SUM () и GROUP BY.

```sql
WITH year_total AS (
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(ss_ext_list_price-ss_ext_wholesale_cost-ss_ext_discount_amt+ss_ext_sales_price, 0)/2) year_total
       ,'s' sale_type
FROM customer
     ,store_sales
     ,date_dim
WHERE c_customer_sk = ss_customer_sk
   AND ss_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year
UNION ALL
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(cs_ext_list_price-cs_ext_wholesale_cost-cs_ext_discount_amt+cs_ext_sales_price, 0)/2) year_total
       ,'c' sale_type
FROM customer
     ,catalog_sales
     ,date_dim
WHERE c_customer_sk = cs_bill_customer_sk
   AND cs_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year
UNION ALL
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(ws_ext_list_price-ws_ext_wholesale_cost-ws_ext_discount_amt+ws_ext_sales_price, 0)/2) year_total
       ,'w' sale_type
FROM customer
     ,web_sales
     ,date_dim
WHERE c_customer_sk = ws_bill_customer_sk
   AND ws_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year
         )
  SELECT TOP 100
                  t_s_secyear.customer_id
                 ,t_s_secyear.customer_first_name
                 ,t_s_secyear.customer_last_name
                 ,t_s_secyear.customer_birth_country
FROM year_total t_s_firstyear
     ,year_total t_s_secyear
     ,year_total t_c_firstyear
     ,year_total t_c_secyear
     ,year_total t_w_firstyear
     ,year_total t_w_secyear
WHERE t_s_secyear.customer_id = t_s_firstyear.customer_id
   AND t_s_firstyear.customer_id = t_c_secyear.customer_id
   AND t_s_firstyear.customer_id = t_c_firstyear.customer_id
   AND t_s_firstyear.customer_id = t_w_firstyear.customer_id
   AND t_s_firstyear.customer_id = t_w_secyear.customer_id
   AND t_s_firstyear.sale_type = 's'
   AND t_c_firstyear.sale_type = 'c'
   AND t_w_firstyear.sale_type = 'w'
   AND t_s_secyear.sale_type = 's'
   AND t_c_secyear.sale_type = 'c'
   AND t_w_secyear.sale_type = 'w'
   AND t_s_firstyear.dyear+0 =  1999
   AND t_s_secyear.dyear+0 = 1999+1
   AND t_c_firstyear.dyear+0 =  1999
   AND t_c_secyear.dyear+0 =  1999+1
   AND t_w_firstyear.dyear+0 = 1999
   AND t_w_secyear.dyear+0 = 1999+1
   AND t_s_firstyear.year_total > 0
   AND t_c_firstyear.year_total > 0
   AND t_w_firstyear.year_total > 0
   AND CASE WHEN t_c_firstyear.year_total > 0 THEN t_c_secyear.year_total / t_c_firstyear.year_total ELSE NULL END
           > CASE WHEN t_s_firstyear.year_total > 0 THEN t_s_secyear.year_total / t_s_firstyear.year_total ELSE NULL END
   AND CASE WHEN t_c_firstyear.year_total > 0 THEN t_c_secyear.year_total / t_c_firstyear.year_total ELSE NULL END
           > CASE WHEN t_w_firstyear.year_total > 0 THEN t_w_secyear.year_total / t_w_firstyear.year_total ELSE NULL END
ORDER BY t_s_secyear.customer_id
         ,t_s_secyear.customer_first_name
         ,t_s_secyear.customer_last_name
         ,t_s_secyear.customer_birth_country
OPTION ( LABEL = 'Query04-af359846-253-3');
```

Проверьте предполагаемый план выполнения запроса.  Там будет 18 операций перемешивания и 17 операций соединения, что требует больше времени на выполнение. Теперь давайте создадим одно материализованное представление для каждой из трех вложенных инструкций SELECT.

```sql
CREATE materialized view nbViewSS WITH (DISTRIBUTION=HASH(customer_id)) AS
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(ss_ext_list_price-ss_ext_wholesale_cost-ss_ext_discount_amt+ss_ext_sales_price, 0)/2) year_total
          , count_big(*) AS cb
FROM dbo.customer
     ,dbo.store_sales
     ,dbo.date_dim
WHERE c_customer_sk = ss_customer_sk
   AND ss_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year
GO
CREATE materialized view nbViewCS WITH (DISTRIBUTION=HASH(customer_id)) AS
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(cs_ext_list_price-cs_ext_wholesale_cost-cs_ext_discount_amt+cs_ext_sales_price, 0)/2) year_total
          , count_big(*) as cb
FROM dbo.customer
     ,dbo.catalog_sales
     ,dbo.date_dim
WHERE c_customer_sk = cs_bill_customer_sk
   AND cs_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year

GO
CREATE materialized view nbViewWS WITH (DISTRIBUTION=HASH(customer_id)) AS
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(ws_ext_list_price-ws_ext_wholesale_cost-ws_ext_discount_amt+ws_ext_sales_price, 0)/2) year_total
          , count_big(*) AS cb
FROM dbo.customer
     ,dbo.web_sales
     ,dbo.date_dim
WHERE c_customer_sk = ws_bill_customer_sk
   AND ws_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year

```

Проверьте еще раз план выполнения исходного запроса.  Теперь количество соединений изменилось с 17 до 5, перемешивания и вовсе отсутствуют.  Выберите значок операции фильтрации в плане. Список выходных данных показывает, что данные считываются из материализованных представлений, а не из базовых таблиц.  

 ![Список выходных данных плана с материализованными представлениями](./media/develop-materialized-view-performance-tuning/output-list.png)

При использовании материализованных представлений один и тот же запрос выполняется гораздо быстрее без изменения кода.  

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные советы по разработке приведены в [обзоре разработки Synapse SQL](develop-overview.md).
 