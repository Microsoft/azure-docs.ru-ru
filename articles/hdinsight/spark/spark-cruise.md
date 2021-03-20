---
title: Ускорение Apache Spark запросов с помощью Спарккруисе в Azure HDInsight
description: Узнайте, как использовать платформу оптимизации Спарккруисе для повышения эффективности Apache Sparkных запросов.
ms.service: hdinsight
ms.topic: how-to
ms.date: 07/27/2020
ms.openlocfilehash: f07ea3007d2e72c179abe38cca56178b564bc231
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98928274"
---
# <a name="sparkcruise-on-azure-hdinsight"></a>Спарккруисе в Azure HDInsight

В этом документе рассматривается функция Azure HDInsight *спарккруисе*, которая автоматически использует вычисления Apache Spark для повышения эффективности запросов.

## <a name="overview"></a>Обзор

Запросы, выполняемые на платформе аналитики, например Apache Spark, разбиваются на план запроса, который содержит небольшие вложенные запросы. Эти вложенные запросы могут многократно отображаться в планах запросов для нескольких запросов. Каждый раз, когда они происходят, они повторно выполняются для возврата результатов. Однако повторное выполнение того же запроса может оказаться неэффективным и привести к ненужным затратам на вычисления.

*Спарккруисе* — это платформа оптимизации рабочей нагрузки, которая может повторно использовать общие вычисления, уменьшая общее время выполнения запроса и затраты на передачу данных. Платформа использует концепцию *материализованных представлений*, которые представляют собой запрос, результаты которого хранятся в предварительно вычисленной форме. Эти результаты можно использовать повторно, когда сам запрос снова отобразится позже, вместо повторного вычисления результатов.

## <a name="setup-and-installation"></a>Настройка и установка

Спарккруисе доступен во всех кластерах HDInsight 4,0 с Spark 2,3 или 2,4. Файлы библиотеки Спарккруисе устанавливаются в каталог в `/opt/peregrine/` кластере HDInsight. Для правильной работы *спарккруисе* требуются следующие свойства конфигурации, которые устанавливаются по умолчанию.

* `spark.sql.queryExecutionListeners` имеет значение `com.microsoft.peregrine.spark.listeners.PlanLogListener` , которое позволяет вести журнал планов запросов.
* `spark.sql.extensions` имеет значение `com.microsoft.peregrine.spark.extensions.SparkExtensionsHdi` , которое включает правила оптимизатора для интерактивного материализации и повторного использования.

## <a name="computation-reuse-in-spark-sql"></a>Повторное использование вычислений в Spark SQL

В следующем примере сценария показано, как использовать *спарккруисе* для оптимизации запросов Apache Spark. 

1. Подключитесь по протоколу SSH к головному узлу кластера Spark.
1. Введите `spark-shell`.
1. Выполните следующий фрагмент кода, в котором выполняется несколько базовых запросов с использованием демонстрационных данных в кластере.

    ```scala
    spark.sql("select count(*) from hivesampletable").collect
    spark.sql("select count(*) from hivesampletable").collect
    spark.sql("select distinct market from hivesampletable where querytime like '11%'").show
    spark.sql("select distinct state, country from hivesampletable where querytime like '11%'").show
    :quit
    ```
1. Используйте средство анализа запросов *спарккруисе* для анализа планов запросов предыдущих запросов, которые хранятся в журналах приложений Spark. 

    ```
    sudo /opt/peregrine/analyze/peregrine.sh analyze views
    ```

1. Просмотрите выходные данные процесса анализа, который является файлом обратной связи. Этот файл обратной связи содержит заметки для будущих запросов Spark SQL. 

    ```
    sudo /opt/peregrine/analyze/peregrine.sh show
    ```

`analyze`Команда анализирует планы запросов и создает табличное представление рабочей нагрузки. Эту таблицу рабочей нагрузки можно запросить с помощью записной книжки *ворклоадинсигхтс* , входящей в репозиторий [HDInsight спарккруисе Samples](https://github.com/Azure-Samples/azure-sparkcruise-samples) . Затем `views` команда определяет общие выражения вложенного плана и выбирает интересные выражения вложенного плана для будущей материализации и повторного использования. Выходные данные — это файл обратной связи, содержащий заметки для будущих запросов Spark SQL. 

`show`Команда отображает выходные данные, как в следующем тексте:

```bash
Feedback file -->

1593761760087311271 Materialize /peregrine/views/1593761760087311271
1593761760087311271 Reuse /peregrine/views/1593761760087311271
18446744073621796959 Materialize /peregrine/views/18446744073621796959
18446744073621796959 Reuse /peregrine/views/18446744073621796959
11259615723090744908 Materialize /peregrine/views/11259615723090744908
11259615723090744908 Reuse /peregrine/views/11259615723090744908
9409467400931056980 Materialize /peregrine/views/9409467400931056980
9409467400931056980 Reuse /peregrine/views/9409467400931056980

Materialized subexpressions -->

Found 4 items
-rw-r--r--   1 sshuser sshuser     113445 2020-07-24 16:46 /peregrine/views/logical_ir.csv
-rw-r--r--   1 sshuser sshuser     169458 2020-07-24 16:46 /peregrine/views/physical_ir.csv
-rw-r--r--   1 sshuser sshuser      25730 2020-07-24 16:46 /peregrine/views/views.csv
-rw-r--r--   1 sshuser sshuser        536 2020-07-24 16:46 /peregrine/views/views.stp
```

Файл обратной связи содержит записи в следующем формате: `subplan-identifier [Materialize|Reuse] input/path/to/action` . В этом примере есть две уникальные сигнатуры: одна представляет первые два повторяющихся запроса, а вторая — предикат фильтра в двух последних запросах. При использовании этого файла обратной связи следующие запросы при повторной отправке будут автоматически материализовать и повторно использовать общие вложенные планы. 

Чтобы протестировать оптимизацию, выполните еще один набор примеров запросов.

1. Введите `spark-shell`.
1. Выполните следующий фрагмент кода.

    ```scala
    spark.sql("select count(*) from hivesampletable").collect
    spark.sql("select count(*) from hivesampletable").collect
    spark.sql("select distinct state, country from hivesampletable where querytime like '12%'").show
    spark.sql("select distinct market from hivesampletable where querytime like '12%'").show
    :quit
    ```

1. Снова просмотрите файл обратной связи, чтобы просмотреть сигнатуры повторно используемых запросов.

    ```
    sudo /opt/peregrine/analyze/peregrine.sh show
    ```

`show`Команда выводит примерно следующий текст:

```bash
Feedback file -->

1593761760087311271 Materialize /peregrine/views/1593761760087311271
1593761760087311271 Reuse /peregrine/views/1593761760087311271
18446744073621796959 Materialize /peregrine/views/18446744073621796959
18446744073621796959 Reuse /peregrine/views/18446744073621796959
11259615723090744908 Materialize /peregrine/views/11259615723090744908
11259615723090744908 Reuse /peregrine/views/11259615723090744908
9409467400931056980 Materialize /peregrine/views/9409467400931056980
9409467400931056980 Reuse /peregrine/views/9409467400931056980

Materialized subexpressions -->

Found 8 items
drwxr-xr-x   - root root          0 2020-07-24 17:21 /peregrine/views/11259615723090744908
drwxr-xr-x   - root root          0 2020-07-24 17:21 /peregrine/views/1593761760087311271
drwxr-xr-x   - root root          0 2020-07-24 17:22 /peregrine/views/18446744073621796959
drwxr-xr-x   - root root          0 2020-07-24 17:21 /peregrine/views/9409467400931056980
-rw-r--r--   1 root root     113445 2020-07-24 16:46 /peregrine/views/logical_ir.csv
-rw-r--r--   1 root root     169458 2020-07-24 16:46 /peregrine/views/physical_ir.csv
-rw-r--r--   1 root root      25730 2020-07-24 16:46 /peregrine/views/views.csv
-rw-r--r--   1 root root        536 2020-07-24 16:46 /peregrine/views/views.stp

```

Хотя литеральное значение в запросе изменилось с `'11%'` на `'12%'` , *спарккруисе* может по-прежнему сопоставлять предыдущие запросы с новыми запросами с незначительными вариациями, такими как развитие литеральных значений и версий наборов данных. Если запрос содержит значительные изменения, можно повторно запустить средство анализа, чтобы узнать о новых запросах, которые можно повторно использовать.

В фоновом режиме *спарккруисе* активирует вложенный запрос для материализации выбранного вложенного плана из первого запроса, содержащего его. Последующие запросы могут напрямую считывать материализованные вложенные планы, а не перечислять их. В этой рабочей нагрузке вложенные планы будут материализоваться в оперативном режиме в результате первого и третьего запросов. Можно увидеть изменение плана запросов после материализации общих вложенных планов.

Теперь вы видите четыре новых материализованных выражения, которые можно использовать повторно в последующих запросах.

## <a name="clean-up"></a>Очистка

Файлы отзывов, материализованные вложенные планы и журналы запросов сохраняются в сеансах Spark. Чтобы удалить эти файлы, выполните следующую команду:

```bash
sudo /opt/peregrine/analyze/peregrine.sh clean
```

## <a name="next-steps"></a>Дальнейшие действия

* [Используйте записную книжку "аналитика рабочей нагрузки", чтобы определить преимущества Спарккруисе](https://github.com/Azure-Samples/azure-sparkcruise-samples/tree/main/SparkCruise)
* [Повышение производительности Apache Spark рабочих нагрузок с помощью кэша ввода-вывода Azure HDInsight](apache-spark-improve-performance-iocache.md)
* [Оптимизация заданий Apache Spark в Azure HDInsight](./apache-spark-perf.md)
* [Спарккруисе: повторное вычисление Хандсфри в Spark](https://people.cs.umass.edu/~aroy/sparkcruise-vldb19.pdf)
* [Рекомендации по Apache Spark в Azure HDInsight](./spark-best-practices.md)
