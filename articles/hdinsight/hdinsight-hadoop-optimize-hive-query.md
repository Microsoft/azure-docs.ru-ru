---
title: Оптимизация запросов Hive в Azure HDInsight
description: В этой статье описывается, как оптимизировать запросы Apache Hive в Azure HDInsight.
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 10/28/2020
ms.openlocfilehash: a15c3e0fb3550c6e50b3fba2279611fdba25bc84
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98945564"
---
# <a name="optimize-apache-hive-queries-in-azure-hdinsight"></a>Оптимизация запросов Apache в Hive в Azure HDInsight

В этой статье описываются некоторые из наиболее распространенных способов оптимизации производительности, которые можно использовать для повышения производительности запросов Apache Hive.

## <a name="cluster-type-selection"></a>Выбор типа кластера

В Azure HDInsight можно выполнять запросы Apache Hive на нескольких различных типах кластеров. 

Выберите подходящий тип кластера, чтобы оптимизировать производительность для потребностей рабочей нагрузки.

* Выберите тип кластера **интерактивных запросов** , чтобы оптимизировать для `ad hoc` интерактивных запросов. 
* Выберите тип кластера Apache **Hadoop**, чтобы оптимизировать его для запросов Hive, используемых в качестве пакетной обработки. 
* Типы кластеров **Spark** и **HBase** также могут выполнять запросы Hive и могут быть подходящими, если вы используете эти рабочие нагрузки. 

Дополнительные сведения о выполнении запросов Hive кластерами различных типов см. в статье [Обзор Apache Hive и HiveQL в Azure HDInsight](hadoop/hdinsight-use-hive.md).

## <a name="scale-out-worker-nodes"></a>Горизонтальное масштабирование рабочих узлов

Увеличение числа рабочих узлов в кластере HDInsight позволяет использовать больше средств сопоставления и модулей сжатия для параллельного выполнения. Горизонтальное масштабирование в HDInsight можно выполнить двумя способами:

* При создании кластера можно указать количество рабочих узлов, используя портал Azure, Azure PowerShell или интерфейс командной строки.  Дополнительные сведения см. в статье о [создании кластеров HDInsight](hdinsight-hadoop-provision-linux-clusters.md). На следующем снимке экрана показана рабочая конфигурация узла на портале Azure:
  
    ![Узлы размера кластера портал Azure](./media/hdinsight-hadoop-optimize-hive-query/azure-portal-cluster-configuration.png "scaleout_1")

* После создания кластера можно также изменить количество рабочих узлов, чтобы масштабировать кластер без необходимости его повторного создания.

    ![Размер кластера портал Azure Scale](./media/hdinsight-hadoop-optimize-hive-query/azure-portal-settings-nodes.png "scaleout_2")

Дополнительные сведения о масштабировании HDInsight см. в статье [Масштабирование кластеров HDInsight](hdinsight-scaling-best-practices.md).

## <a name="use-apache-tez-instead-of-map-reduce"></a>Использование Apache Tez вместо Map Reduce

[Apache Tez](https://tez.apache.org/) — это альтернативный механизм выполнения для механизма MapReduce. При подготовке кластеров HDInsight под управлением Linux платформа Tez включена по умолчанию.

![Обзорная схема Apache Tez HDInsight](./media/hdinsight-hadoop-optimize-hive-query/hdinsight-tez-engine.png)

Tez работает быстрее, так как:

* **Выполняет задания, представленные в виде направленных ациклических графов (DAG), как единое целое при работе с платформой MapReduce.** DAG требует, чтобы набору модулей сопоставления соответствовал набор модулей сжатия. Это требование приводит к тому, что для каждого запроса Hive несколько заданий MapReduce будут отключены. Tez не имеет такого ограничения и может обрабатывать сложную DAG как одно задание свести к минимуму издержки на запуск заданий.
* **Позволяет избежать ненужных операций записи.** Чтобы обработать тот же запрос Hive в механизме MapReduce, используются несколько заданий. Выходные данные каждого задания MapReduce записываются в HDFS для промежуточных данных. Поскольку Tez уменьшает количество заданий для каждого запроса Hive, можно избежать ненужных операций записи.
* **Сводит к минимуму задержки при загрузке.** Платформа Tez способствует снижению задержки при запуске благодаря уменьшению количества модулей сопоставления, необходимых для запуска, а также путем оптимизации всего процесса.
* **Повторно использует контейнеры.** Всякий раз, когда возможно, TEZ будет повторно использовать контейнеры, чтобы избежать снижения задержек при запуске контейнеров.
* **Применяет методы непрерывной оптимизации.** Традиционно оптимизация выполняется на этапе компиляции. Однако по мере поступления дополнительных сведений о входных данных появляется возможность улучшить оптимизацию уже на этапе работы. Tez применяет методы непрерывной оптимизации, что позволяет производить дальнейшую оптимизацию плана на этапе работы.

Дополнительные сведения об этих понятиях см. в документе об [Apache Tez](https://tez.apache.org/).

Вы можете выполнить любой запрос Hive, используя платформу Tez, активировав ее с помощью префикса запроса с использованием следующей команды набора:

```hive
set hive.execution.engine=tez;
```

## <a name="hive-partitioning"></a>Секционирование данных в Hive

Операции ввода-вывода — это основные ограничивающие факторы производительности при выполнении запросов Hive. Производительность можно повысить, если удастся уменьшить объем данных, которые необходимо считать. По умолчанию запросы Hive просматривают все таблицы Hive. Однако для запросов, которым требуется просмотреть только небольшой объем данных (например, для запросов с фильтрацией), это поведение создает излишнюю нагрузку. Секционирование данных Hive предоставляет запросам Hive доступ только к необходимому объему данных в таблицах Hive.

Секционирование данных в Hive реализуется путем реорганизации необработанных данных в новые каталоги. Каждой секции присвоен собственный каталог файлов. Секционирование определяется пользователем. Следующая схема иллюстрирует секционирование таблицы Hive по столбцу *Год*. Для каждого года создается новый каталог.

![Секционирование Apache Hive HDInsight](./media/hdinsight-hadoop-optimize-hive-query/hdinsight-partitioning.png)

Некоторые рекомендации для выполнения секционирования:

* Не подразделайте **секционирование** в столбцах с несколькими значениями может привести к небольшому числу секций. Например, секционирование на полу создает только две создаваемые секции (пол и женщина), поэтому Сократите задержку до половины.
* **Не поверх секций** — с другой стороны, создание секции для столбца с уникальным значением (например, UserID) приводит к созданию нескольких секций. что увеличит нагрузку на узел имен кластера из-за необходимости обрабатывать множество каталогов.
* **Избегайте неравномерного распределения данных** — выбирайте ключ секционирования рационально, таким образом, чтобы все секции были одинакового размера. Например, секционирование в столбце *Штат* может исказить распределение типов данных. Так как в штате Калифорния в 30 раз больше населения, чем в штате Вермонт, размер секции потенциально искажен, а производительность может сильно различаться.

Чтобы выполнить секционирование таблицы, используйте команду *Секционирована по* :

```sql
CREATE TABLE lineitem_part
      (L_ORDERKEY INT, L_PARTKEY INT, L_SUPPKEY INT,L_LINENUMBER INT,
      L_QUANTITY DOUBLE, L_EXTENDEDPRICE DOUBLE, L_DISCOUNT DOUBLE,
      L_TAX DOUBLE, L_RETURNFLAG STRING, L_LINESTATUS STRING,
      L_SHIPDATE_PS STRING, L_COMMITDATE STRING, L_RECEIPTDATE STRING,
      L_SHIPINSTRUCT STRING, L_SHIPMODE STRING, L_COMMENT STRING)
PARTITIONED BY(L_SHIPDATE STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE;
```

Как только секционированная таблица будет создана, можно выполнить статическое или динамическое секционирование.

* **Статическое секционирование** означает, что сегментированные данные уже находятся в соответствующих каталогах. С помощью статических секций вы добавляете секции Hive вручную, исходя из расположения каталога. Просмотрите пример в следующем фрагменте кода:
  
   ```sql
   INSERT OVERWRITE TABLE lineitem_part
   PARTITION (L_SHIPDATE = '5/23/1996 12:00:00 AM')
   SELECT * FROM lineitem
   WHERE lineitem.L_SHIPDATE = '5/23/1996 12:00:00 AM'

   ALTER TABLE lineitem_part ADD PARTITION (L_SHIPDATE = '5/23/1996 12:00:00 AM')
   LOCATION 'wasb://sampledata@ignitedemo.blob.core.windows.net/partitions/5_23_1996/'
   ```

* **Динамическое секционирование** означает, что Hive автоматически создаст секции для вас. Так как вы уже создали таблицу секционирования из промежуточной таблицы, достаточно вставить данные в секционированную таблицу:
  
   ```hive
   SET hive.exec.dynamic.partition = true;
   SET hive.exec.dynamic.partition.mode = nonstrict;
   INSERT INTO TABLE lineitem_part
   PARTITION (L_SHIPDATE)
   SELECT L_ORDERKEY as L_ORDERKEY, L_PARTKEY as L_PARTKEY,
       L_SUPPKEY as L_SUPPKEY, L_LINENUMBER as L_LINENUMBER,
       L_QUANTITY as L_QUANTITY, L_EXTENDEDPRICE as L_EXTENDEDPRICE,
       L_DISCOUNT as L_DISCOUNT, L_TAX as L_TAX, L_RETURNFLAG as L_RETURNFLAG,
       L_LINESTATUS as L_LINESTATUS, L_SHIPDATE as L_SHIPDATE_PS,
       L_COMMITDATE as L_COMMITDATE, L_RECEIPTDATE as L_RECEIPTDATE,
       L_SHIPINSTRUCT as L_SHIPINSTRUCT, L_SHIPMODE as L_SHIPMODE,
       L_COMMENT as L_COMMENT, L_SHIPDATE as L_SHIPDATE FROM lineitem;
   ```

Дополнительные сведения см. в разделе о [секционированных таблицах](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-PartitionedTables).

## <a name="use-the-orcfile-format"></a>Использование формата ORC-файлов

Hive поддерживает различные форматы. Пример:

* **Текст**: формат файла по умолчанию работает с большинством сценариев.
* **Avro**: хорошо подходит для сценариев взаимодействия.
* **ORC/Parquet**: лучше всего подходит для повышения производительности.

Формат ORC (Optimized Row Columnar) — это очень эффективный способ хранения данных Hive. По сравнению с другими форматами ORC имеет следующие преимущества:

* поддержка сложных типов, в том числе даты и время и частично структурированные и сложные типы;
* обеспечивает показатель сжатия данных до 70 %;
* индексирует каждые 10 000 строк, что позволяет пропускать строки;
* обеспечивает существенное снижение времени выполнения запросов.

Чтобы активировать ORC формат, необходимо сначала создать таблицу командой *Хранение в виде ORC*:

```sql
CREATE TABLE lineitem_orc_part
      (L_ORDERKEY INT, L_PARTKEY INT,L_SUPPKEY INT, L_LINENUMBER INT,
      L_QUANTITY DOUBLE, L_EXTENDEDPRICE DOUBLE, L_DISCOUNT DOUBLE,
      L_TAX DOUBLE, L_RETURNFLAG STRING, L_LINESTATUS STRING,
      L_SHIPDATE_PS STRING, L_COMMITDATE STRING, L_RECEIPTDATE STRING,
      L_SHIPINSTRUCT STRING, L_SHIPMODE STRING, L_COMMENT      STRING)
PARTITIONED BY(L_SHIPDATE STRING)
STORED AS ORC;
```

Далее необходимо выполнить вставку данных в таблицу ORC из промежуточной таблицы. Пример:

```sql
INSERT INTO TABLE lineitem_orc
SELECT L_ORDERKEY as L_ORDERKEY,
         L_PARTKEY as L_PARTKEY ,
         L_SUPPKEY as L_SUPPKEY,
         L_LINENUMBER as L_LINENUMBER,
         L_QUANTITY as L_QUANTITY,
         L_EXTENDEDPRICE as L_EXTENDEDPRICE,
         L_DISCOUNT as L_DISCOUNT,
         L_TAX as L_TAX,
         L_RETURNFLAG as L_RETURNFLAG,
         L_LINESTATUS as L_LINESTATUS,
         L_SHIPDATE as L_SHIPDATE,
         L_COMMITDATE as L_COMMITDATE,
         L_RECEIPTDATE as L_RECEIPTDATE,
         L_SHIPINSTRUCT as L_SHIPINSTRUCT,
         L_SHIPMODE as L_SHIPMODE,
         L_COMMENT as L_COMMENT
FROM lineitem;
```

Вы можете прочесть подробнее о формате ORC на странице [руководства по языку Apache Hive](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC).

## <a name="vectorization"></a>Векторизация

Векторизация позволяет Hive обрабатывать пакеты из 1024 строк одновременно вместо обработки каждой строки отдельно. Это означает, что простые операции будут выполняться быстрее благодаря меньшему объему внутреннего кода, необходимого для запуска.

Чтобы активировать функцию векторизации, создайте запрос Hive со следующими параметрами:

```hive
set hive.vectorized.execution.enabled = true;
```

Для получения дополнительных сведений обратитесь к разделу [Выполнение векторизованного запроса](https://cwiki.apache.org/confluence/display/Hive/Vectorized+Query+Execution).

## <a name="other-optimization-methods"></a>Другие методы оптимизации

Существуют дополнительные методы оптимизации, которые также можно использовать, например:

* **Группирование Hive** — техника, которая позволяет сегментировать или объединять в кластеры большие наборы данных для оптимизации производительности запросов.
* **Оптимизация объединений** — это оптимизация выполнения запросов Hive с целью повышения эффективности объединений и сокращения действия пользователя. Для получения дополнительных сведений обратитесь к разделу [Оптимизация объединений](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+JoinOptimization#LanguageManualJoinOptimization-JoinOptimization).
* **Увеличьте модулей сжатия**.

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали некоторые распространенные методы оптимизации запросов Hive. Дополнительные сведения см. в следующих статьях:

* [Оптимизация Apache Hive](./optimize-hive-ambari.md)
* [Анализ данных о задержке рейсов с помощью интерактивного запроса в HDInsight](./interactive-query/interactive-query-tutorial-analyze-flight-data.md)
* [Анализ данных Twitter с помощью Apache Hive в HDInsight](hdinsight-analyze-twitter-data-linux.md)
