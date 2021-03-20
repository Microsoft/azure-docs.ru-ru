---
title: Использование кластера Azure HDInsight Hadoop в наборе данных объемом 1 ТБ — процесс обработки и анализа данных группы
description: Применение процесса обработки и анализа данных группы в комплексном сценарии, включающем в себя использование кластера HDInsight Hadoop для создания и развертывания модели на основе общедоступного набора данных большого объема (1 ТБ).
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: e66bd0a4e56f63185d8361355d6cf8e0e29bc30b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93305932"
---
# <a name="the-team-data-science-process-in-action---using-an-azure-hdinsight-hadoop-cluster-on-a-1-tb-dataset"></a>Процесс обработки и анализа данных группы в действии — использование кластера Azure HDInsight Hadoop в наборе данных объемом 1 ТБ

В этом пошаговом руководстве показано как использовать комплексный сценарий процесса обработки и анализа данных группы на примере [кластера Azure HDInsight Hadoop](https://azure.microsoft.com/services/hdinsight/) для хранения, изучения, проектирования признаков и снижения частоты выборки данных в одном из общедоступных наборов данных [Criteo](https://labs.criteo.com/downloads/download-terabyte-click-logs/). Мы построим модель двоичной классификации этих данных, используя Машинное обучение Azure, и покажем, как опубликовать одну из этих моделей в качестве веб-службы.

Для выполнения заданий, представленных в этом пошаговом руководстве, можно также использовать iPython Notebook. Пользователям, которые хотят применить этот подход, следует просмотреть статью [Criteo walkthrough using a Hive ODBC connection](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/iPythonNotebooks/machine-Learning-data-science-process-hive-walkthrough-criteo.ipynb) (Пошаговое руководство Criteo по использованию подключения Hive ODBC).

## <a name="criteo-dataset-description"></a><a name="dataset"></a>Описание набора данных Criteo
Данные Criteo — это прогнозирующий набор данных, который составляет 370 ГБ сжатых TSV-файлов gzip (~ 1,3 ТБ без сжатия), включающих более 4 300 000 000 записей. Эти данные получены на основе данных о переходах по ссылкам за 24 дня, предоставленных [Criteo](https://labs.criteo.com/downloads/download-terabyte-click-logs/). Для удобства специалистов по обработке и анализу данных данные, доступные для экспериментирования, были разархивированы.

Каждая запись в этом наборе данных содержит 40 столбцов:

* первый столбец — столбец с меткой, указывающий, щелкнул пользователь **рекламное объявление** (значение 1) или не щелкнул (значение 0);
* следующие 13 столбцов являются числовыми;
* последние 26 столбцов категориальные.

Столбцы являются анонимными, и в них используется ряд перечисляемых имен — от Col1 (столбец с меткой) до Col40 (последний категориальный столбец).

Ниже приведен фрагмент первых 20 столбцов из двух наблюдений (строк) этого набора данных:

> `Col1    Col2    Col3    Col4    Col5    Col6    Col7    Col8    Col9    Col10    Col11    Col12    Col13    Col14    Col15            Col16            Col17            Col18            Col19        Col20`
>
> `0       40      42      2       54      3       0       0       2       16      0       1       4448    4       1acfe1ee        1b2ff61f        2e8b2631        6faef306        c6fc10d3    6fcd6dcb`
>
> `0               24              27      5               0       2       1               3       10064           9a8cb066        7a06385f        417e6103        2170fc56        acf676aa    6fcd6dcb`

В этом наборе данных как в числовых столбцах, так и в категориальных столбцах есть недостающие значения. Простой способ обработки недостающих значений приведен ниже. Дополнительные сведения о данных предоставляются при их сохранении в таблицах Hive.

**Определение:** *Частота* переходов. Эта метрика представляет собой процент щелчков в данных. В этом наборе Criteo коэффициент составляет около 3,3 % или 0,033.

## <a name="examples-of-prediction-tasks"></a><a name="mltasks"></a>Примеры задач прогнозирования
В этом пошаговом руководстве рассмотрены два примера задач по прогнозированию:

1. **Двоичная классификация**— прогнозирует, щелкнет ли пользователь рекламное объявление:

   * класс 0 — не щелкнул;
   * класс 1 — щелкнул.
2. **Регрессия**— прогнозирует вероятность щелчка рекламного объявления по признакам пользователя.

## <a name="set-up-an-hdinsight-hadoop-cluster-for-data-science"></a><a name="setup"></a>Настройка кластера HDInsight Hadoop для обработки и анализа данных
> [!NOTE]
> Этот шаг обычно является задачей **администратора** .

Настройте среду обработки и анализа данных Azure, чтобы создавать решения для прогнозной аналитики с помощью кластеров HDInsight, выполнив три шага:

1. [Создание учетной записи хранения.](../../storage/common/storage-account-create.md) Эта учетная запись хранения используется для хранения данных в хранилище BLOB-объектов Azure. Здесь хранятся данные, используемые в кластерах HDInsight.
2. [Настройка кластеров Azure HDInsight Hadoop для обработки и анализа данных.](../../hdinsight/spark/apache-spark-jupyter-spark-sql.md) На этом шаге создается кластер Azure HDInsight Hadoop и на всех узлах устанавливается 64-разрядный дистрибутив Anaconda Python 2.7. При настройке кластера HDInsight следует выполнить два важных шага (описанных в этом разделе).

   * Свяжите учетную запись хранения, созданную на шаге 1, с кластером HDInsight при его создании. Эта учетная запись хранения используется для доступа к данным, которые можно обработать в пределах кластера.
   * После создания кластера включите удаленный доступ к головному узлу кластера. Запомните указанные здесь учетные данные для удаленного доступа (отличные от учетных данных, указанных при создании кластера). выполните следующие процедуры.
3. [Создание рабочей области машинное обучение Azure Studio (классической)](../classic/create-workspace.md). Эта машинное обучение Azure Рабочая область используется для создания моделей машинного обучения после первоначального изучения и понижения выборки данных в кластере HDInsight.

## <a name="get-and-consume-data-from-a-public-source"></a><a name="getdata"></a>Получение и использование данных из общедоступного источника
Доступ к набору данных [Criteo](https://labs.criteo.com/downloads/download-terabyte-click-logs/) можно получить, щелкнув ссылку, приняв условия использования и указав имя. Ниже показан моментальный снимок.

![Принятие условий использования Criteo](./media/hive-criteo-walkthrough/hLxfI2E.png)

Нажмите кнопку **Continue to Download** (Продолжить для скачивания), чтобы ознакомиться с дополнительными сведениями о наборе данных и его доступности.

Данные находятся в расположении [хранилища BLOB-объектов Azure](../../storage/blobs/storage-quickstart-blobs-dotnet.md) : wasb://criteo@azuremlsampleexperiments.blob.core.windows.net/raw/ . wasb относится к расположению хранилища больших двоичных объектов Azure.

1. Данные в этом хранилище BLOB-объектов Azure состоят из трех вложенных папок распакованных данных.

   1. Вложенная папка *raw/count/* содержит данные за первые 21 день — от day\_00 до day\_20.
   2. Вложенная папка *raw/train/* содержит данные за один день — day\_21.
   3. Вложенная папка *raw/test/* содержит данные за два дня — day\_22 и day\_23.
2. Необработанные данные gzip также доступны в основной папке *RAW/* AS day_NN. gz, где nn — от 00 до 23.

Альтернативный способ получения доступа к этим данным, их просмотра и моделирования, который не требует скачивания локальных файлов, описан далее в этом пошаговом руководстве на этапе создания таблиц Hive.

## <a name="log-in-to-the-cluster-headnode"></a><a name="login"></a>Вход в головной узел кластера
Чтобы войти в головной узел кластера, узнайте его расположение, используя [портал Azure](https://ms.portal.azure.com) . Щелкните значок HDInsight в виде слона в левой части портала, а затем дважды щелкните имя кластера. Перейдите на вкладку **Конфигурация** , дважды щелкните значок подключения в нижней части страницы и введите учетные данные удаленного доступа при появлении запроса на головного узла кластера.

Вот как обычно первое имя входа в кластер головного узла выглядит следующим образом:

![Вход в кластер](./media/hive-criteo-walkthrough/Yys9Vvm.png)

В левой части виден значок "Командная строка Hadoop". С помощью этой строки выполняется просмотр данных. Обратите внимание на два полезных URL-адреса — "Состояние Hadoop Yarn" и "Узел имен Hadoop". URL-адрес состояния Yarn показывает ход выполнения задания, а URL-адрес узла имен предоставляет дополнительную информацию о конфигурации кластера.

Теперь все настроено и готово для начала первой части пошагового руководства — просмотр данных с помощью Hive и подготовка данных для Машинного обучения Azure.

## <a name="create-hive-database-and-tables"></a><a name="hive-db-tables"></a> Создание базы данных и таблиц Hive
Чтобы создать таблицы Hive для нашего набора данных Criteo, откройте ***командную строку Hadoop*** на рабочем столе головного узла и введите имя каталога Hive с помощью следующей команды:

```console
cd %hive_home%\bin
```

> [!NOTE]
> Выполняйте все команды Hive в этом пошаговом руководстве из командной строки каталога bin/ Hive. Таким образом вы сможете автоматически избежать любых проблем с путем. Вы можете использовать термины "командная строка каталога Hive", "командная строка каталога bin/ Hive" и "командная строка Hadoop" на взаимозаменяемой основе.
>
> [!NOTE]
> Любой запрос Hive всегда можно выполнить с помощью следующих команд:
>
> `cd %hive_home%\bin`
> `hive`

После появления командной строки Hive REPL с символом hive > просто скопируйте и вставьте запрос, чтобы выполнить его.

Следующий код создает базу данных «Criteo», а затем создает четыре таблицы:

* *таблицу для создания счетчиков*, созданную для дней с day\_00 до day\_20;
* *таблицу для использования в качестве набора данных*, созданную для дня day\_21;
* две *таблицы для использования в качестве наборов данных теста*, созданные для дней day\_22 и day\_23 соответственно.

Разделите тестовый набор данных на две различные таблицы, так как один из дней — праздник. Нам необходимо определить, может ли модель определить разницу между праздником и другим днем по коэффициенту выбора рекламного объявления.

Ниже для удобства приведен скрипт [sample&#95;hive&#95;create&#95;criteo&#95;database&#95;and&#95;tables.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_create_criteo_database_and_tables.hql):

```hiveql
CREATE DATABASE IF NOT EXISTS criteo;
DROP TABLE IF EXISTS criteo.criteo_count;
CREATE TABLE criteo.criteo_count (
col1 string,col2 double,col3 double,col4 double,col5 double,col6 double,col7 double,col8 double,col9 double,col10 double,col11 double,col12 double,col13 double,col14 double,col15 string,col16 string,col17 string,col18 string,col19 string,col20 string,col21 string,col22 string,col23 string,col24 string,col25 string,col26 string,col27 string,col28 string,col29 string,col30 string,col31 string,col32 string,col33 string,col34 string,col35 string,col36 string,col37 string,col38 string,col39 string,col40 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
STORED AS TEXTFILE LOCATION 'wasb://criteo@azuremlsampleexperiments.blob.core.windows.net/raw/count';

DROP TABLE IF EXISTS criteo.criteo_train;
CREATE TABLE criteo.criteo_train (
col1 string,col2 double,col3 double,col4 double,col5 double,col6 double,col7 double,col8 double,col9 double,col10 double,col11 double,col12 double,col13 double,col14 double,col15 string,col16 string,col17 string,col18 string,col19 string,col20 string,col21 string,col22 string,col23 string,col24 string,col25 string,col26 string,col27 string,col28 string,col29 string,col30 string,col31 string,col32 string,col33 string,col34 string,col35 string,col36 string,col37 string,col38 string,col39 string,col40 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
STORED AS TEXTFILE LOCATION 'wasb://criteo@azuremlsampleexperiments.blob.core.windows.net/raw/train';

DROP TABLE IF EXISTS criteo.criteo_test_day_22;
CREATE TABLE criteo.criteo_test_day_22 (
col1 string,col2 double,col3 double,col4 double,col5 double,col6 double,col7 double,col8 double,col9 double,col10 double,col11 double,col12 double,col13 double,col14 double,col15 string,col16 string,col17 string,col18 string,col19 string,col20 string,col21 string,col22 string,col23 string,col24 string,col25 string,col26 string,col27 string,col28 string,col29 string,col30 string,col31 string,col32 string,col33 string,col34 string,col35 string,col36 string,col37 string,col38 string,col39 string,col40 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
STORED AS TEXTFILE LOCATION 'wasb://criteo@azuremlsampleexperiments.blob.core.windows.net/raw/test/day_22';

DROP TABLE IF EXISTS criteo.criteo_test_day_23;
CREATE TABLE criteo.criteo_test_day_23 (
col1 string,col2 double,col3 double,col4 double,col5 double,col6 double,col7 double,col8 double,col9 double,col10 double,col11 double,col12 double,col13 double,col14 double,col15 string,col16 string,col17 string,col18 string,col19 string,col20 string,col21 string,col22 string,col23 string,col24 string,col25 string,col26 string,col27 string,col28 string,col29 string,col30 string,col31 string,col32 string,col33 string,col34 string,col35 string,col36 string,col37 string,col38 string,col39 string,col40 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
STORED AS TEXTFILE LOCATION 'wasb://criteo@azuremlsampleexperiments.blob.core.windows.net/raw/test/day_23';
```

Все эти таблицы являются внешними, поэтому вы можете указать их расположения в хранилище BLOB-объектов Azure (wasb).

**ЛЮБОЙ запрос Hive можно выполнить двумя способами:**

* **С помощью командной строки Hive REPL**: сначала выполните команду Hive, а затем скопируйте и вставьте запрос в командной строке Hive REPL:

  ```console
  cd %hive_home%\bin
  hive
  ```

     Теперь в командной строке REPL выполняется вырезание и вставление запроса.
* **Сохранение запросов в файл и выполнение команды**. Второй способ — сохранить запросы в файл ". HQL" ([пример&#95;hive&#95;создать&#95;criteo&#95;database&#95;и&#95;Tables. HQL](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_create_criteo_database_and_tables.hql)), а затем выполнить следующую команду для выполнения запроса:

  ```console
  hive -f C:\temp\sample_hive_create_criteo_database_and_tables.hql
  ```

### <a name="confirm-database-and-table-creation"></a>Подтверждение создания базы данных и таблицы
Далее подтвердите создание базы данных, выполнив команду в командной строке каталога bin/ Hive.

```console
hive -e "show databases;"
```

Результат выполнения команды:

```output
criteo
default
Time taken: 1.25 seconds, Fetched: 2 row(s)
```

Таким образом подтверждается создание новой базы данных criteo.

Чтобы увидеть, какие таблицы были созданы, просто выполните команду в командной строке каталога bin/ Hive.

```output
hive -e "show tables in criteo;"
```

После этого вы должны увидеть следующие выходные данные:

```ouput
criteo_count
criteo_test_day_22
criteo_test_day_23
criteo_train
Time taken: 1.437 seconds, Fetched: 4 row(s)
```

## <a name="data-exploration-in-hive"></a><a name="exploration"></a> Просмотр данных в Hive
Теперь можно приступить к базовому просмотру данных в Hive. Начните с подсчета примеров в таблицах данных для обучения и тестирования.

### <a name="number-of-train-examples"></a>Количество примеров для обучения
Ниже показано содержимое скрипта [sample&#95;hive&#95;count&#95;train&#95;table&#95;examples.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_count_train_table_examples.hql):

```hiveql
SELECT COUNT(*) FROM criteo.criteo_train;
```

Мы получаем такой результат:

```output
192215183
Time taken: 264.154 seconds, Fetched: 1 row(s)
```

Кроме того, в командной строке каталога bin/ Hive можно выполнить следующую команду:

```console
hive -f C:\temp\sample_hive_count_criteo_train_table_examples.hql
```

### <a name="number-of-test-examples-in-the-two-test-datasets"></a>Количество тестовых примеров в двух тестовых наборах данных
Теперь подсчитайте количество примеров в двух тестовых наборах данных. Ниже показано содержимое скрипта [sample&#95;hive&#95;count&#95;criteo&#95;test&#95;day&#95;22&#95;table&#95;examples.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_count_criteo_test_day_22_table_examples.hql):

```hiveql
SELECT COUNT(*) FROM criteo.criteo_test_day_22;
```

Мы получаем такой результат:

```output
189747893
Time taken: 267.968 seconds, Fetched: 1 row(s)
```

Как правило, вы также можете вызвать скрипт в командной строке каталога bin/ Hive, используя следующую команду:

```console
hive -f C:\temp\sample_hive_count_criteo_test_day_22_table_examples.hql
```

Наконец, можно просмотреть количество тестовых примеров в тестовом наборе данных на основе показателя day\_23.

Команда для этого похожа на показанную (см [. пример&#95;hive&#95;количество&#95;criteo&#95;test&#95;день&#95;23&#95;examples. HQL](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_count_criteo_test_day_23_examples.hql)):

```hiveql
SELECT COUNT(*) FROM criteo.criteo_test_day_23;
```

Результат выполнения команды:

```output
178274637
Time taken: 253.089 seconds, Fetched: 1 row(s)
```

### <a name="label-distribution-in-the-train-dataset"></a>Распределение меток в наборе данных для обучения
Необходимо определить распределение меток в наборе данных для обучения. Чтобы увидеть его, покажите содержимое скрипта [sample_hive_criteo_label_distribution_train_table.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_label_distribution_train_table.hql):

```hiveql
SELECT Col1, COUNT(*) AS CT FROM criteo.criteo_train GROUP BY Col1;
```

Мы получаем такое распределение меток:

```output
1       6292903
0       185922280
Time taken: 459.435 seconds, Fetched: 2 row(s)
```

Процент положительных меток составляет примерно 3,3% (в соответствии с исходным набором данных).

### <a name="histogram-distributions-of-some-numeric-variables-in-the-train-dataset"></a>Распределения гистограммы некоторых числовых переменных в наборе данных для обучения
Чтобы узнать, как выглядит распределение числовых переменных, вы можете использовать функцию Hive histogram\_numeric. Ниже представлено содержимое скрипта [sample&#95;hive&#95;criteo&#95;histogram&#95;numeric.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_histogram_numeric.hql):

```hiveql
SELECT CAST(hist.x as int) as bin_center, CAST(hist.y as bigint) as bin_height FROM
    (SELECT
    histogram_numeric(col2, 20) as col2_hist
    FROM
    criteo.criteo_train
    ) a
    LATERAL VIEW explode(col2_hist) exploded_table as hist;
```

Мы получаем следующий результат:

```output
26      155878415
2606    92753
6755    22086
11202   6922
14432   4163
17815   2488
21072   1901
24113   1283
27429   1225
30818   906
34512   723
38026   387
41007   290
43417   312
45797   571
49819   428
53505   328
56853   527
61004   160
65510   3446
Time taken: 317.851 seconds, Fetched: 20 row(s)
```

Сочетание параметров LATERAL VIEW и explode в Hive служит для вывода выходных данных, подобных SQL, вместо обычного списка. В этой таблице первый столбец соответствует центру ячейки, а второй — к частоте bin.

### <a name="approximate-percentiles-of-some-numeric-variables-in-the-train-dataset"></a>Приблизительные процентили для некоторых числовых переменных в наборе данных для обучения
Кроме того, нам необходимо вычислить приблизительные процентили для числовых переменных. Функция Hive percentile\_approx делает это автоматически. Ниже представлено содержимое скрипта [sample&#95;hive&#95;criteo&#95;approximate&#95;percentiles.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_approximate_percentiles.hql):

```hiveql
SELECT MIN(Col2) AS Col2_min, PERCENTILE_APPROX(Col2, 0.1) AS Col2_01, PERCENTILE_APPROX(Col2, 0.3) AS Col2_03, PERCENTILE_APPROX(Col2, 0.5) AS Col2_median, PERCENTILE_APPROX(Col2, 0.8) AS Col2_08, MAX(Col2) AS Col2_max FROM criteo.criteo_train;
```

Мы получаем такой результат:

```output
1.0     2.1418600917169246      2.1418600917169246    6.21887086390288 27.53454893115633       65535.0
Time taken: 564.953 seconds, Fetched: 1 row(s)
```

Обычно распределение процентилей тесно связано с распределением гистограммы для любой числовой переменной.

### <a name="find-number-of-unique-values-for-some-categorical-columns-in-the-train-dataset"></a>Поиск количества уникальных значений для некоторых категориальных столбцов в наборе данных для обучения
Мы продолжаем просматривать данные, и теперь нам нужно найти количество уникальных значений, принимаемых некоторыми категориальными столбцами. Для этого следует показать содержимое скрипта [sample_hive_criteo_unique_values_categoricals.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_unique_values_categoricals.hql):

```hiveql
SELECT COUNT(DISTINCT(Col15)) AS num_uniques FROM criteo.criteo_train;
```

Мы получаем такой результат:

```output
19011825
Time taken: 448.116 seconds, Fetched: 1 row(s)
```

Col15 имеет 19M уникальные значения! Использование упрощенных методов, таких как "прямое кодирование" для кодирования таких многомерных категориальных переменных, невозможно. Для эффективного решения этой проблемы объясняется и демонстрируется эффективный и надежный метод, называемый [обучение с использованием счетчиков](/archive/blogs/machinelearning/big-learning-made-easy-with-counts).

Наконец, определите количество уникальных значений для других категориальных столбцов. Ниже приведено содержимое скрипта [sample&#95;hive&#95;criteo&#95;unique&#95;values&#95;multiple&#95;categoricals.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_unique_values_multiple_categoricals.hql):

```hiveql
SELECT COUNT(DISTINCT(Col16)), COUNT(DISTINCT(Col17)),
COUNT(DISTINCT(Col18), COUNT(DISTINCT(Col19), COUNT(DISTINCT(Col20))
FROM criteo.criteo_train;
```

Мы получаем такой результат:

```output
30935   15200   7349    20067   3
Time taken: 1933.883 seconds, Fetched: 1 row(s)
```

Снова отметим, что во всех столбцах, за исключением Col20, содержится множество уникальных значений.

### <a name="co-occurrence-counts-of-pairs-of-categorical-variables-in-the-train-dataset"></a>Подсчет совместного вхождения пар категориальных переменных в наборе данных для обучения

Также интересны распределения количества пар переменных категории. Это можно определить с помощью кода в скрипте [sample&#95;hive&#95;criteo&#95;paired&#95;categorical&#95;counts.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_paired_categorical_counts.hql):

```hiveql
SELECT Col15, Col16, COUNT(*) AS paired_count FROM criteo.criteo_train GROUP BY Col15, Col16 ORDER BY paired_count DESC LIMIT 15;
```

В этом случае количество приведено в обратном порядке по порядку вхождения и рассмотрено 15 первых значений. Результат выполнения команды:

```output
ad98e872        cea68cd3        8964458
ad98e872        3dbb483e        8444762
ad98e872        43ced263        3082503
ad98e872        420acc05        2694489
ad98e872        ac4c5591        2559535
ad98e872        fb1e95da        2227216
ad98e872        8af1edc8        1794955
ad98e872        e56937ee        1643550
ad98e872        d1fade1c        1348719
ad98e872        977b4431        1115528
e5f3fd8d        a15d1051        959252
ad98e872        dd86c04a        872975
349b3fec        a52ef97d        821062
e5f3fd8d        a0aaffa6        792250
265366bf        6f5c7c41        782142
Time taken: 560.22 seconds, Fetched: 15 row(s)
```

## <a name="down-sample-the-datasets-for-azure-machine-learning"></a><a name="downsample"></a> Снижение частоты выборки в наборе данных для Машинного обучения Azure
Просмотрев наборы данных и продемонстрировав, как выполнить этот тип просмотра для любых переменных (включая сочетания переменных), приступим к снижению частоты выборки в наборах данных для создания моделей в Машинном обучении Azure. Помните, что рассматривается следующая задача: при заданном наборе примеров атрибутов (значений признаков от Col2 до Col40) нужно спрогнозировать, какое значение примет Col1 — 0 (без щелчка) или 1 (щелчок).

Чтобы снизить частоту выборки в наших наборах данных для обучения и тестовых наборах данных до 1 % от исходного размера, будет использована функция Hive RAND(). Следующий скрипт [sample&#95;hive&#95;criteo&#95;downsample&#95;train&#95;dataset.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_downsample_train_dataset.hql) выполняет это действие для набора данных для обучения:

```hiveql
CREATE TABLE criteo.criteo_train_downsample_1perc (
col1 string,col2 double,col3 double,col4 double,col5 double,col6 double,col7 double,col8 double,col9 double,col10 double,col11 double,col12 double,col13 double,col14 double,col15 string,col16 string,col17 string,col18 string,col19 string,col20 string,col21 string,col22 string,col23 string,col24 string,col25 string,col26 string,col27 string,col28 string,col29 string,col30 string,col31 string,col32 string,col33 string,col34 string,col35 string,col36 string,col37 string,col38 string,col39 string,col40 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
STORED AS TEXTFILE;

---Now downsample and store in this table

INSERT OVERWRITE TABLE criteo.criteo_train_downsample_1perc SELECT * FROM criteo.criteo_train WHERE RAND() <= 0.01;
```

Мы получаем такой результат:

```output
Time taken: 12.22 seconds
Time taken: 298.98 seconds
```

Скрипт [sample&#95;hive&#95;criteo&#95;downsample&#95;test&#95;day&#95;22&#95;dataset.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_downsample_test_day_22_dataset.hql) выполняет это действие для тестовых данных за день day\_22:

```hiveql
--- Now for test data (day_22)

CREATE TABLE criteo.criteo_test_day_22_downsample_1perc (
col1 string,col2 double,col3 double,col4 double,col5 double,col6 double,col7 double,col8 double,col9 double,col10 double,col11 double,col12 double,col13 double,col14 double,col15 string,col16 string,col17 string,col18 string,col19 string,col20 string,col21 string,col22 string,col23 string,col24 string,col25 string,col26 string,col27 string,col28 string,col29 string,col30 string,col31 string,col32 string,col33 string,col34 string,col35 string,col36 string,col37 string,col38 string,col39 string,col40 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
STORED AS TEXTFILE;

INSERT OVERWRITE TABLE criteo.criteo_test_day_22_downsample_1perc SELECT * FROM criteo.criteo_test_day_22 WHERE RAND() <= 0.01;
```

Мы получаем такой результат:

```output
Time taken: 1.22 seconds
Time taken: 317.66 seconds
```

Скрипт [sample&#95;hive&#95;criteo&#95;downsample&#95;test&#95;day&#95;23&#95;dataset.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_downsample_test_day_23_dataset.hql) выполняет это действие для тестовых данных за день day\_23:

```hiveql
--- Finally test data day_23
CREATE TABLE criteo.criteo_test_day_23_downsample_1perc (
col1 string,col2 double,col3 double,col4 double,col5 double,col6 double,col7 double,col8 double,col9 double,col10 double,col11 double,col12 double,col13 double,col14 double,col15 string,col16 string,col17 string,col18 string,col19 string,col20 string,col21 string,col22 string,col23 string,col24 string,col25 string,col26 string,col27 string,col28 string,col29 string,col30 string,col31 string,col32 string,col33 string,col34 string,col35 string,col36 string,col37 string,col38 string,col39 string,col40 srical feature; tring)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
STORED AS TEXTFILE;

INSERT OVERWRITE TABLE criteo.criteo_test_day_23_downsample_1perc SELECT * FROM criteo.criteo_test_day_23 WHERE RAND() <= 0.01;
```

Мы получаем такой результат:

```output
Time taken: 1.86 seconds
Time taken: 300.02 seconds
```

Такой результат позволяет вам использовать наборы данных для обучения и тестовые наборы данных со сниженной частотой выборки для создания моделей в Машинном обучении Azure.

Прежде чем перейти к машинному обучению Azure, следует обсудить последний важный компонент, касающийся таблицы счетчиков. В следующем подразделе таблица счетчиков обсуждается подробно.

## <a name="a-brief-discussion-on-the-count-table"></a><a name="count"></a> Краткое описание таблицы счетчиков
Как вы видели, несколько переменных категории имеют высокую размерность. В нашем пошаговом руководстве представлен метод под названием [обучение с использованием счетчиков](/archive/blogs/machinelearning/big-learning-made-easy-with-counts) для кодирования этих переменных эффективным и надежным способом. Дополнительную информацию об этом методе можно получить, перейдя по указанной ссылке.

>[!NOTE]
>В этом пошаговом руководстве рассматривается использование таблиц счетчиков для создания компактных представлений многомерных категориальных признаков. Это не единственный способ кодирования категориальных признаков. Дополнительные сведения о других методах заинтересованные пользователи могут найти в статьях по [прямому кодированию](https://en.wikipedia.org/wiki/One-hot) и [хэшированию признаков](https://en.wikipedia.org/wiki/Feature_hashing).
>

Чтобы создать таблицы счетчиков на основе счетных данных, используйте данные в папке raw/count. В разделе о моделировании пользователям показано, как создавать эти таблицы счетчиков для категориальных признаков с нуля или использовать предварительно созданную таблицу счетчиков для работы с данными. В следующих разделах термин "предварительно созданные таблицы счетчиков" употребляется в значении предоставляемых таблиц счетчиков. Подробные инструкции о том, как получить доступ к этим таблицам, приведены в следующем разделе.

## <a name="build-a-model-with-azure-machine-learning"></a><a name="aml"></a> Создание моделей с помощью Машинного обучения Azure
Процесс создания моделей в Машинном обучении Azure состоит из следующих шагов:

1. [Получение данных из таблиц Hive и их добавление в Машинное обучение Azure](#step1)
2. [Создайте эксперимент: Очистите данные и сделайте его функцией с таблицами подсчета](#step2)
3. [Создание, обучение и оценка модели](#step3)
4. [Анализ модели](#step4)
5. [Публикация модели в качестве веб-службы](#step5)

Теперь можно приступить к созданию моделей в Студии машинного обучения Azure. Наши данные со сниженной частотой выборки сохраняются в кластере в виде таблиц Hive. Чтобы считать эти данные, используйте модуль **Импорт данных** Машинного обучения Azure. Учетные данные для доступа к учетной записи хранения этого кластера указаны далее.

### <a name="step-1-get-data-from-hive-tables-into-azure-machine-learning-using-the-import-data-module-and-select-it-for-a-machine-learning-experiment"></a><a name="step1"></a> Шаг 1. Извлечение данных из таблиц Hive в Машинное обучение Azure с помощью модуля "Импорт данных" и выбор этих данных для эксперимента машинного обучения
Для начала выберите **Новый**  ->    ->  **пустой эксперимент** эксперимента. Затем в поле **Поиск** в верхнем левом углу экрана введите запрос "Импорт данных". Перетащите модуль **Импорт данных** на холст эксперимента (в средней части экрана), чтобы использовать его для доступа к данным.

Вот как выглядит модуль **Импорт данных** при получении данных из таблицы Hive:

![Модуль "Импорт данных" получает данные](./media/hive-criteo-walkthrough/i3zRaoj.png)

Указанные выше параметры модуля **Импорт данных** представляют собой пример обязательных значений. Ниже представлены общие указания по заполнению набора параметров для модуля **Импорт данных** .

1. В поле **Источник данных**
2. В поле **Hive database query** (Запрос базы данных Hive) достаточно просто ввести SELECT * FROM <имя\_вашей\_базы_данных.имя\_вашей\_таблицы>.
3. **URI сервера Hcatalog**: Если кластер имеет значение ABC, это просто: https: \/ /abc.azurehdinsight.NET
4. **Имя учетной записи пользователя Hadoop** — имя пользователя, выбранное во время введения кластера в эксплуатацию. (НЕ имя пользователя для удаленного доступа).
5. **Пароль учетной записи пользователя Hadoop**. Пароль для имени пользователя, выбранный во время введения кластера в эксплуатацию (НЕ пароль для удаленного доступа).
6. **Расположение выходных данных**: выберите Azure.
7. **Имя учетной записи хранения Azure**: учетная запись хранения, связанная с кластером.
8. **Ключ учетной записи хранения Azure**— ключ учетной записи хранения, связанной с кластером.
9. **Имя контейнера Azure**: если используется имя кластера abc, нужно просто ввести abc.

После того как модуль **Импорт данных** завершит получение данных (возле модуля отобразится зеленая галочка), сохраните эти данные в качестве набора данных (с соответствующим именем). Это выглядит так:

![Модуль "Импорт данных" сохраняет данные](./media/hive-criteo-walkthrough/oxM73Np.png)

Щелкните правой кнопкой мыши порт вывода модуля **Импорт данных** . Отобразятся параметры **Сохранение набора данных** и **Визуализировать**. Если щелкнуть параметр **Визуализировать** , отобразится 100 строк с данными, а также правая панель, использующаяся для получения сводных статистических данных. Чтобы сохранить данные, выберите **Сохранение набора данных** и следуйте указаниям.

Чтобы выбрать сохраненный набор данных, который будет использоваться в эксперименте машинного обучения, найдите его с помощью поля **Поиск** (на рисунке ниже). Затем введите часть имени набора данных, чтобы получить к нему доступ, и перетащите набор данных на главную панель. При этом набор данных автоматически выбирается для использования в моделировании машинного обучения.

![Перетащите набор данных на главную панель](./media/hive-criteo-walkthrough/cl5tpGw.png)

> [!NOTE]
> Выполните эту процедуру для набора данных для обучения и тестового набора данных. Кроме того, не забывайте использовать имя базы данных и имена таблиц, которые вы присвоили для этой цели. Показанные в примере значения приводятся только для справки.\*\*
>
>

### <a name="step-2-create-an-experiment-in-azure-machine-learning-to-predict-clicks--no-clicks"></a><a name="step2"></a> Шаг 2. Создание эксперимента в Машинное обучение Azure для прогнозирования щелчков/без щелчков
Наш эксперимент Машинное обучение Azure Studio (классический) выглядит следующим образом:

![Эксперимент машинного обучения](./media/hive-criteo-walkthrough/xRpVfrY.png)

Теперь рассмотрим основные компоненты этого эксперимента. Сначала перетащите наши сохраненные наборы данных Training и Test на холст эксперимента.

#### <a name="clean-missing-data"></a>Очистка недостающих данных
Модуль **Clean Missing Data** (Очистка недостающих данных) выполняет одноименное действие, т. е. очищает недостающие данные, применяя для этого способ, указанный пользователем. Просмотрите этот модуль, чтобы увидеть следующее:

![Очистка недостающих данных](./media/hive-criteo-walkthrough/0ycXod6.png)

Здесь можно заменить все отсутствующие значения на 0. Есть и другие варианты, которые можно увидеть, просмотрев раскрывающиеся списки в модуле.

#### <a name="feature-engineering-on-the-data"></a>Проектирование компонентов на основе данных
Некоторые категориальные функции крупных наборов данных могут иметь миллионы уникальных значений. Использовать для представления таких многомерных функций упрощенные методы, например метод "прямого кодирования", нецелесообразно. В этом пошаговом руководстве мы покажем, как с помощью встроенных модулей машинного обучения Azure применять функции счетчиков для создания компактных представлений многомерных категориальных переменных. Конечным результатом является меньший размер модели, более быстрое время обучения и метрики производительности, сравнимые с использованием других методов.

##### <a name="building-counting-transforms"></a>Построение преобразований счетчиков
Для создания функции счетчика используйте модуль **Построение преобразования счетчика**, доступный в Машинном обучении Azure. Модуль выглядит следующим образом:

![Свойства модуля "Построение преобразования счетчика"](./media/hive-criteo-walkthrough/e0eqKtZ.png)
![Модуль "Построение преобразования счетчика"](./media/hive-criteo-walkthrough/OdDN0vw.png)

> [!IMPORTANT]
> В поле **Count columns** (Число столбцов) введите столбцы, к которым нужно применить счетчик. Обычно это (как уже говорилось) — многомерные категориальные столбцы. Помните, что набор данных Criteo содержит 26 категориальных столбцов, от Col15 до Col40. Теперь применим счетчики ко всем этим столбцам и присвоим им индексы (от 15 до 40, разделенные запятыми).
>

Для использования модуля в режиме MapReduce (подходит для больших наборов данных) требуется доступ к кластеру HDInsight Hadoop (можно использовать тот же кластер, который применялся для просмотра компонентов) и его учетные данные. На предыдущих снимках экрана показано, как выглядят заполненные поля значений (замените их на значения, соответствующие вашему случаю).

![Параметры модуля](./media/hive-criteo-walkthrough/05IqySf.png)

На предыдущем снимке экрана показано, как указать расположение входного большого двоичного объекта. В этом расположении содержатся данные, зарезервированные под таблицы счетчиков.

Когда выполнение этого модуля будет завершено, сохраните преобразование для последующего использования, щелкнув модуль правой кнопкой мыши и выбрав параметр **Save as Transform** (Сохранить как преобразование):

![Параметр Save as Transform (Сохранить как преобразование)](./media/hive-criteo-walkthrough/IcVgvHR.png)

В представленной выше экспериментальной архитектуре сохраненному преобразованию счетчика точно соответствует набор данных ytransform2. Для остальной части эксперимента предполагается, что читатель, применяет модуль **Построение преобразования счетчика** к некоторым данным для формирования счетчиков, а затем может использовать эти счетчики для создания функций счетчиков, применяемых к наборам данных для обучения и тестовым наборам данных.

##### <a name="choosing-what-count-features-to-include-as-part-of-the-train-and-test-datasets"></a>Выбор функций счетчика для включения в наборы данных для обучения и тестовые наборы данных
После преобразования счетчика можно выбрать функции для включения в наборы данных для обучения и тестовые наборы данных, используя для этого модуль **Изменение параметров таблицы счетчиков**. Этот модуль показан здесь для полноты информации, но в целях упрощения в данном эксперименте он фактически не используется.

![Modify Count Table Parameters (Изменение параметров таблицы счетчиков)](./media/hive-criteo-walkthrough/PfCHkVg.png)

Как уже было показано, в данном случае будут использоваться только логарифмы отношения вероятностей и будет игнорироваться столбец задержки. Вы также можете задать такие параметры, как пороговое значение ячейки сборки мусора, количество псевдослучайных предыдущих примеров для добавления с целью сглаживания и использование лапласовского шума. Все это — дополнительные возможности, и следует отметить, что пользователям, не знакомым с процессом создания функций подобного типа, рекомендуется начинать со значений по умолчанию.

##### <a name="data-transformation-before-generating-the-count-features"></a>Преобразование данных перед созданием функций счетчика
Теперь основное внимание уделяется такому важному моменту, как преобразование данных для обучения и тестовых данных перед созданием функций счетчика. Перед применением преобразования «счетчик» к нашим данным используются два модуля « **выполнение сценария R** ».

![Модули "Выполнить сценарий R"](./media/hive-criteo-walkthrough/aF59wbc.png)

Первый скрипт R выглядит следующим образом:

![Первый сценарий R](./media/hive-criteo-walkthrough/3hkIoMx.png)

В этом скрипте R столбцы получают новые имена — от Col1 до Col40. Этот формат имен необходим для преобразования счетчика.

Второй скрипт R балансирует распределение между положительными и отрицательными классами (классы 1 и 0 соответственно). Для этого используется понижающая дискретизация отрицательного класса. Приведенный ниже скрипт R показывает, как ее выполнить.

![Второй сценарий R](./media/hive-criteo-walkthrough/91wvcwN.png)

В этом простом скрипте R для установки интервала баланса между положительным и отрицательным классами используется выражение "pos\_neg\_ratio". Это действие важно, так как устранение дисбаланса между классами положительно влияет на скорость выполнения задач классификации со скошенным распределением классов (напомним, что в этом случае у вас на положительный класс приходится 3,3 %, а на отрицательный — 96,7 %).

##### <a name="applying-the-count-transformation-on-our-data"></a>Применение преобразования счетчика к данным
Наконец, вы можете использовать модуль **Применить преобразование** и применить преобразования счетчика к наборам данных для обучения и тестовым наборам данных. Этот модуль воспринимает сохраненное преобразование счетчика как один набор входных данных, а наборы данных для обучения и тестовые наборы данных — как другой набор входных данных и возвращает данные с функциями счетчиков. Вот как это работает:

![Модуль применения преобразования](./media/hive-criteo-walkthrough/xnQvsYf.png)

##### <a name="an-excerpt-of-what-the-count-features-look-like"></a>Фрагмент вида функций счетчика
Внешний вид функций счетчика в нашем случае очень информативен. Ниже приведен его фрагмент:

![Функции счетчика](./media/hive-criteo-walkthrough/FO1nNfw.png)

В этом фрагменте показано, что для инвентаризированных столбцов вы получаете как соответствующие задержки, так и счетчики и логарифмы отношения вероятностей.

Теперь вы готовы к созданию модели машинного обучения Azure, используя преобразованный набор данных. В следующем разделе показано, как это сделать.

### <a name="step-3-build-train-and-score-the-model"></a><a name="step3"></a> Шаг 3. Создание, обучение и оценка модели

#### <a name="choice-of-learner"></a>Выбор ученика
В первую очередь вам необходимо выбрать ученика. Используйте двухклассовое увеличивающееся дерево принятия решений в качестве ученика. По умолчанию для этого ученика используются следующие параметры:

![Параметры двухклассового увеличивающегося дерева принятия решений](./media/hive-criteo-walkthrough/bH3ST2z.png)

Выберите значения по умолчанию для эксперимента. Значения по умолчанию являются понятными и хорошим способом быстро получить базовые показатели производительности. Построив базу, можно повысить производительность, скорректировав желаемые параметры.

#### <a name="train-the-model"></a>Обучение модели
Для обучения просто вызовите модуль **Обучение модели**. Он использует два набора входных данных — двухклассовое увеличивающееся дерево принятия решений (наш ученик) и набор данных для обучения. Вот как это работает:

![Модуль "Обучение модели"](./media/hive-criteo-walkthrough/2bZDZTy.png)

#### <a name="score-the-model"></a>Оценка модели
Обучив модель, вы можете оценить ее производительность с помощью тестового набора данных. Чтобы это сделать, используйте модуль **Score Model** (Оценка модели) на следующем рисунке вместе с модулем **Evaluate Model** (Анализ модели):

![Модуль "Оценка модели"](./media/hive-criteo-walkthrough/fydcv6u.png)

### <a name="step-4-evaluate-the-model"></a><a name="step4"></a> Шаг 4. Анализ модели
Наконец следует проанализировать производительность модели. Как правило, для задач двухклассовой (двоичной) классификации хорошо подходит мера AUC. Чтобы визуализировать эту кривую, подключите модуль **Оценка модели** к модулю **Анализ модели** . Если щелкнуть **Визуализировать** в модуле **Evaluate Model** (Анализ модели), отобразится подобная схема:

![Модуль «Анализ модели» и «Увеличивающееся дерево принятия решений»](./media/hive-criteo-walkthrough/0Tl0cdg.png)

В задачах двоичной (или двухклассовой) классификации хорошо подходит такая мера прогнозирования точности, как AUC (площадь под кривой). В следующих разделах продемонстрированы результаты использования этой модели на основе нашего тестового набора данных. Щелкните правой кнопкой мыши порт вывода модуля **Анализ модели** и выберите **визуализировать**.

![Визуализация модуля «Анализ модели»](./media/hive-criteo-walkthrough/IRfc7fH.png)

### <a name="step-5-publish-the-model-as-a-web-service"></a><a name="step5"></a> Шаг 5. Публикация модели в качестве веб-службы
Публикация модели машинного обучения Azure как веб-службы требует минимальных усилий и позволяет сделать ее доступной для других пользователей. После публикации любой сможет вызвать веб-службу, передав ей исходные данные, по которым требуется прогноз, а веб-служба — выдать прогноз, используя эту модель.

Сначала сохраните обученную модель как объект обученной модели, щелкнув модуль **обучение модели** правой кнопкой мыши и выбрав команду **Сохранить как обученную модель** .

Теперь создайте входные и выходные порты для веб-службы:

* входной порт принимает данные в том же виде, что и данные, для которых вам требуется прогноз;
* выходной порт вывода выдает метки оценки и связанные с ними значения вероятности.

#### <a name="select-a-few-rows-of-data-for-the-input-port"></a>Выбор нескольких строк данных для порта ввода
Чтобы выбрать всего 10 строк для использования в качестве данных порта ввода, можно воспользоваться модулем **Применение преобразования SQL** . Выберите для входного порта только эти строки данных с помощью показанного ниже SQL-запроса.

![Данные порта ввода](./media/hive-criteo-walkthrough/XqVtSxu.png)

#### <a name="web-service"></a>Веб-служба
Теперь можно приступить к выполнению небольшого эксперимента, который можно использовать для публикации веб-службы.

#### <a name="generate-input-data-for-webservice"></a>Создание входных данных для веб-службы
Так как таблица счетчиков имеет большой размер, на начальном этапе возьмите несколько строк тестовых данных и создайте из них выходные данные с признаками счетчиков. Эти выходные данные могут использоваться в качестве формата входных данных для нашей WebService, как показано ниже:

![Создание входных данных модуля «Увеличивающееся дерево принятия решений»](./media/hive-criteo-walkthrough/OEJMmst.png)

> [!NOTE]
> В качестве формата входных данных используйте выходные данные модуля **Count Featurizer** (Характеризатор счетчиков). После завершения выполнения этого эксперимента сохраните выходные данные модуля **Характеризатор счетчиков** в качестве набора данных. Этот набор данных используется для входных данных в веб-службе.
>
>

#### <a name="scoring-experiment-for-publishing-webservice"></a>Оценка эксперимента для публикации веб-службы
Во-первых, необходимая структура — это модуль **оценки модели** , который принимает наш обученный объект модели и несколько строк входных данных, созданных в предыдущих шагах с помощью модуля **Count характеризатора** . Используйте модуль "Выбор столбцов в наборе данных", чтобы отобразить расчетные метки и значения вероятности оценки.

![Выбор столбцов в наборе данных](./media/hive-criteo-walkthrough/kRHrIbe.png)

Обратите внимание, что модуль **Выбор столбцов в наборе данных** можно использовать для фильтрации данных из набора данных. Содержимое показано ниже:

![Фильтрация с помощью модуля "Выбор столбцов в наборе данных"](./media/hive-criteo-walkthrough/oVUJC9K.png)

Чтобы порты ввода и вывода стали синими, просто щелкните значок **Подготовить веб-службу** в нижнем правом углу. Выполнение этого эксперимента также позволяет опубликовать веб-службу, щелкнув значок **Publish web service** (Опубликовать веб-службу) в нижнем правом углу, показанный ниже.

![Publish web service](./media/hive-criteo-walkthrough/WO0nens.png)

После публикации веб-службы перейдите на страницу, которая выглядит таким образом:

![Панель мониторинга веб-службы](./media/hive-criteo-walkthrough/YKzxAA5.png)

Обратите внимание на две ссылки для веб-служб слева:

* Служба **Запрос — ответ** (или RRS), предназначенная для разовых прогнозов. Именно она будет использоваться на этом семинаре.
* служба **ПАКЕТНОГО ВЫПОЛНЕНИЯ** (BES) используется для пакетных прогнозов и требует, чтобы входные данные, для которых нужно создать прогноз, находились в большом двоичном объекте Azure.

Если щелкнуть ссылку **Запрос — ответ**, вы перейдете на страницу, которая содержит код на C#, Python и R, поставляемый изготовителем. Этот код можно использовать для вызова веб-службы. Ключ API на этой странице необходимо использовать для проверки подлинности.

Удобно скопировать этот код Рython и вставить его в новую ячейку в iPython Notebook.

Ниже приведен фрагмент кода Python с правильным ключом API.

![Код Python](./media/hive-criteo-walkthrough/f8N4L4g.png)

Ключ API по умолчанию был заменен ключом API WebService. Щелкнув **Запуск** в данной ячейке в iPython Notebook, вы получите следующий ответ:

![Ответ iPython](./media/hive-criteo-walkthrough/KSxmia2.png)

Для двух примеров тестов, о которых потребовалось в JSON Script Framework, вы получаете ответы в форме "оцененные метки, оцененные вероятности". В этом случае значения по умолчанию были выбраны для кода, поставляемого изготовителем (значение 0 для всех числовых столбцов, строка "Значение" для всех категориальных столбцов).

В заключение показано, как управлять большим набором данных с помощью Машинное обучение Azure. Вы начали работу с терабайтом данных, построили модель прогнозирования и развернули ее в облаке как веб-службу.