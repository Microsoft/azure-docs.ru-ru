---
title: Использование Apache Beeline с Apache Hive в Azure HDInsight
description: Сведения о выполнении запросов Hive с помощью Hadoop в HDInsight с использованием клиента Beeline. Beeline — это служебная программа для работы с HiveServer2 через JDBC.
ms.service: hdinsight
ms.topic: how-to
ms.date: 10/28/2020
ms.custom: contperf-fy21q1, contperf-fy21q2
ms.openlocfilehash: e8b7478ba64da0f99a9b7a710222ff2953795adf
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98943205"
---
# <a name="use-the-apache-beeline-client-with-apache-hive"></a>Использование клиента Apache Beeline с Apache Hive

В этой статье описывается, как использовать клиент [Apache Beeline](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients#HiveServer2Clients-Beeline–NewCommandLineShell) для создания и выполнения запросов Apache Hive через SSH-подключение.

## <a name="background"></a>Историческая справка

Beeline — это клиент Hive, установленный на головных узлах кластера HDInsight. Чтобы подключиться к клиенту Beeline, установленному в кластере HDInsight, или установить Beeline локально, см. раздел [Подключение к или установка Apache Beeline](connect-install-beeline.md). Он подключается к службе HiveServer2, размещенной на кластере HDInsight, с помощью JDBC. Beeline также позволяет удаленно подключаться к Hive в HDInsight через Интернет. В следующих примерах приведены наиболее распространенные строки подключения, используемые для подключения к HDInsight из Beeline.

## <a name="prerequisites-for-examples"></a>Предварительные требования для примеров

* Кластер Hadoop в HDInsight. Ознакомьтесь со статьей [Краткое руководство. Использование Apache Hadoop и Apache Hive в Azure HDInsight с шаблоном Resource Manager](./apache-hadoop-linux-tutorial-get-started.md).

* Обратите внимание на схему URI для основного хранилища кластера. Например,  `wasb://` для службы хранилища Azure, `abfs://` для Azure Data Lake Storage 2-го поколения или `adl://` для Azure Data Lake Storage 1-го поколения. Если для службы хранилища Azure включено безопасное перемещение, URI имеет значение `wasbs://` . Дополнительные сведения см. в разделе [безопасное перемещение](../../storage/common/storage-require-secure-transfer.md).

* Клиент SSH. Дополнительные сведения см. в руководстве по [подключению к HDInsight (Apache Hadoop) с помощью SSH](../hdinsight-hadoop-linux-use-ssh-unix.md). В большинстве действий, описанных в этом документе, предполагается, что вы используете Beeline из сеанса SSH в кластере. Вы также можете использовать локальный клиент Beeline, но эти действия не описаны в этой статье.

## <a name="run-a-hive-query"></a>Выполнение запроса Hive

Этот пример основан на использовании клиента Beeline из SSH-подключения.

1. Откройте SSH-подключение к кластеру, используя приведенный ниже код. Замените `sshuser` именем пользователя SSH для кластера, а `CLUSTERNAME` — именем кластера. При появлении запроса введите пароль для учетной записи пользователя SSH.

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

2. Подключитесь к HiveServer2 с помощью клиента Beeline из открытого сеанса SSH, введя следующую команду:

    ```bash
    beeline -u 'jdbc:hive2://headnodehost:10001/;transportMode=http'
    ```

3. Команды Beeline начинаются со знака `!`. Например, `!help` выводит справку. Однако в некоторых командах `!` можно опустить. Например, `help` также работает.

    `!sql`Для выполнения инструкций HiveQL используется. Однако эти инструкции настолько распространены, что `!sql`тоже можно опустить. Приведенные ниже две инструкции эквивалентны.

    ```hiveql
    !sql show tables;
    show tables;
    ```

    В новом кластере отображена только одна таблица, **hivesampletable**.

4. Используйте следующую команду, чтобы отобразить схему для таблицы hivesampletable.

    ```hiveql
    describe hivesampletable;
    ```

    Эта команда возвращает приведенные ниже сведения.

    ```output
    +-----------------------+------------+----------+--+
    |       col_name        | data_type  | comment  |
    +-----------------------+------------+----------+--+
    | clientid              | string     |          |
    | querytime             | string     |          |
    | market                | string     |          |
    | deviceplatform        | string     |          |
    | devicemake            | string     |          |
    | devicemodel           | string     |          |
    | state                 | string     |          |
    | country               | string     |          |
    | querydwelltime        | double     |          |
    | sessionid             | bigint     |          |
    | sessionpagevieworder  | bigint     |          |
    +-----------------------+------------+----------+--+
    ```

    Они описывают столбцы в таблице.

5. Введите следующие инструкции, чтобы создать таблицу с именем **log4jLogs** с помощью демонстрационных данных, предоставляемых в кластере HDInsight: (внесите необходимые изменения в соответствии со схемой URI.)

    ```hiveql
    DROP TABLE log4jLogs;
    CREATE EXTERNAL TABLE log4jLogs (
        t1 string,
        t2 string,
        t3 string,
        t4 string,
        t5 string,
        t6 string,
        t7 string)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
    STORED AS TEXTFILE LOCATION 'wasbs:///example/data/';
    SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs
        WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log'
        GROUP BY t4;
    ```

    Эти инструкции выполняют следующие действия:

    |Инструкция |Описание |
    |---|---|
    |DROP TABLE|Если таблица существует, она удаляется.|
    |CREATE EXTERNAL TABLE|Создает **внешнюю** таблицу в Hive. Внешние таблицы хранят только определения таблицы в Hive. Данные остаются в исходном расположении.|
    |ФОРМАТ СТРОКИ|Форматирование данных. В данном случае поля всех журналов разделены пробелом.|
    |ХРАНИТСЯ КАК РАСПОЛОЖЕНИЕ TEXTFILE|Место хранения данных и формат файла.|
    |SELECT|Выбирает количество строк, в которых столбец **T4** содержит значение **[Error]**. Этот запрос должен вернуть значение **3**, так как таблица содержит три строки с данным значением.|
    |INPUT__FILE__NAME, например "%. log"|Hive пытается применить схему ко всем файлам в каталоге. В этом случае каталог содержит файлы, которые не соответствуют схеме. Чтобы исключить лишние данные в результатах, эта инструкция указывает Hive возвращать данные только из файлов, заканчивающихся на .log.|

   > [!NOTE]  
   > Внешние таблицы следует использовать, если исходные данные должны обновляться с использованием внешних источников. Например, процессом автоматизированной передачи данных или другой операцией MapReduce.
   >
   > Удаление внешней таблицы **не** приводит к удалению данных, будет удалено только определение таблицы.

    После выполнения этой команды вы должны увидеть текст, аналогичный приведенному ниже.

    ```output
    INFO  : Tez session hasn't been created yet. Opening session
    INFO  :

    INFO  : Status: Running (Executing on YARN cluster with App id application_1443698635933_0001)

    INFO  : Map 1: -/-      Reducer 2: 0/1
    INFO  : Map 1: 0/1      Reducer 2: 0/1
    INFO  : Map 1: 0/1      Reducer 2: 0/1
    INFO  : Map 1: 0/1      Reducer 2: 0/1
    INFO  : Map 1: 0/1      Reducer 2: 0/1
    INFO  : Map 1: 0(+1)/1  Reducer 2: 0/1
    INFO  : Map 1: 0(+1)/1  Reducer 2: 0/1
    INFO  : Map 1: 1/1      Reducer 2: 0/1
    INFO  : Map 1: 1/1      Reducer 2: 0(+1)/1
    INFO  : Map 1: 1/1      Reducer 2: 1/1
    +----------+--------+--+
    |   sev    | count  |
    +----------+--------+--+
    | [ERROR]  | 3      |
    +----------+--------+--+
    1 row selected (47.351 seconds)
    ```

6. Выход из Beeline:

    ```bash
    !exit
    ```

## <a name="run-a-hiveql-file"></a>Запуск файла HiveQL.

Этот пример является продолжением из предыдущих примеров. Чтобы создать файл, а затем запустить его с помощью Beeline, выполните следующие действия.

1. Создайте файл **query.hql**, используя следующую команду.

    ```bash
    nano query.hql
    ```

1. В качестве содержимого файла добавьте следующий текст: Этот запрос создает "внутреннюю" таблицу **errorLogs**.

    ```hiveql
    CREATE TABLE IF NOT EXISTS errorLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) STORED AS ORC;
    INSERT OVERWRITE TABLE errorLogs SELECT t1, t2, t3, t4, t5, t6, t7 FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log';
    ```

    Эти инструкции выполняют следующие действия:

    |Инструкция |Описание |
    |---|---|
    |CREATE TABLE, ЕСЛИ НЕ СУЩЕСТВУЕТ|Если таблица еще не существует, она создается. Поскольку ключевое слово **External** не используется, эта инструкция создает внутреннюю таблицу. Внутренние таблицы хранятся в хранилище данных Hive и полностью управляются Hive.|
    |ХРАНИТСЯ В ВИДЕ ORC|Позволяет сохранить данные в формате ORC. Это высокооптимизированный и эффективный формат для хранения данных Hive.|
    |ВСТАВИТЬ ПЕРЕЗАПИСЬ... МЕТЬТЕ|выбирает строки из таблицы **log4jLogs**, которые содержат значение **[ERROR]**, а затем вставляет данные в таблицу **errorLogs**.|

    > [!NOTE]  
    > В отличие от внешних таблиц, удаление внутренней таблицы приводит к удалению базовых данных.

1. Чтобы сохранить файл, нажмите **клавиши CTRL** + **X**, введите **Y** и, наконец, **введите**.

1. Запустите файл с помощью Beeline, используя следующую команду:

    ```bash
    beeline -u 'jdbc:hive2://headnodehost:10001/;transportMode=http' -i query.hql
    ```

    > [!NOTE]  
    > Параметр `-i` запускает Beeline и выполняет инструкции в файле `query.hql`. После выполнения запроса отобразится командная строка `jdbc:hive2://headnodehost:10001/>`. Можно также выполнить файл с помощью параметра `-f`, который завершает работу Beeline после завершения выполнения запроса.

1. Чтобы убедиться, что таблица **errorLogs** создана, выполните приведенную ниже инструкцию (она выводит все строки из таблицы **errorLogs**).

    ```hiveql
    SELECT * from errorLogs;
    ```

    В результате операции должны быть возвращены три строки со значением **[ERROR]** в столбце t4.

    ```output
    +---------------+---------------+---------------+---------------+---------------+---------------+---------------+--+
    | errorlogs.t1  | errorlogs.t2  | errorlogs.t3  | errorlogs.t4  | errorlogs.t5  | errorlogs.t6  | errorlogs.t7  |
    +---------------+---------------+---------------+---------------+---------------+---------------+---------------+--+
    | 2012-02-03    | 18:35:34      | SampleClass0  | [ERROR]       | incorrect     | id            |               |
    | 2012-02-03    | 18:55:54      | SampleClass1  | [ERROR]       | incorrect     | id            |               |
    | 2012-02-03    | 19:25:27      | SampleClass4  | [ERROR]       | incorrect     | id            |               |
    +---------------+---------------+---------------+---------------+---------------+---------------+---------------+--+
    3 rows selected (0.813 seconds)
    ```

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные общие сведения о Hive в HDInsight см. в статье [использование Apache Hive с Apache Hadoop в hdinsight](hdinsight-use-hive.md) .

* Дополнительные сведения о других способах работы с Hadoop в HDInsight см. в статье [Использование MapReduce с Apache Hadoop в hdinsight](hdinsight-use-mapreduce.md) .
