---
title: Изучение данных в кластере Hadoop в процессе обработки и анализа данных группы
description: Применение процесса обработки и анализа данных группы в комплексном сценарии, включающем в себя использование кластера HDInsight Hadoop для создания и развертывания модели.
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
ms.openlocfilehash: 53f50e98bcec4b8ace342808f0bcfd96770834b0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96002227"
---
# <a name="the-team-data-science-process-in-action-use-azure-hdinsight-hadoop-clusters"></a>Процесс обработки и анализа данных группы на практике: использование кластеров Azure HDInsight Hadoop
В этом пошаговом руководстве показано комплексное использование [процесса обработки и анализа данных группы (TDSP)](overview.md). Мы используем кластер [Azure HDInsight Hadoop](https://azure.microsoft.com/services/hdinsight/) для хранения и просмотра данных из общедоступного набора данных [Поездки такси Нью-Йорка](https://www.andresmh.com/nyctaxitrips/), реконструирования их характеристик и сокращения их выборки. Для обработки двоичных и мультиклассовых классификационных и регрессионных прогнозных задач мы создаем модели данных с помощью Машинного обучения Azure. 

Пошаговое руководство, в котором показано, как обрабатывать больший набор данных, см. в разделе [процесс обработки и анализа данных группы — использование Azure HDInsight Hadoop кластеров в наборе данных объемом 1 ТБ](hive-criteo-walkthrough.md).

Вы также можете использовать записную книжку IPython Notebook для выполнения задач, представленных в пошаговом руководстве, в котором используется набор данных объемом 1 ТБ. Дополнительные сведения см. в статье [Criteo walkthrough using a Hive ODBC connection](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/iPythonNotebooks/machine-Learning-data-science-process-hive-walkthrough-criteo.ipynb) (Пошаговое руководство Criteo по использованию подключения Hive ODBC).

## <a name="nyc-taxi-trips-dataset-description"></a><a name="dataset"></a>Описание набора данных "Поездки такси Нью-Йорка"
Сведения о поездках на такси в Нью-Йорке помещены в сжатые файлы данных с разделителями-запятыми (CSV) объемом около 20 ГБ (примерно 48 ГБ без сжатия). Они содержат данные о 173 миллионах поездок и о стоимости каждой из них. Каждая запись о поездке содержит время и место посадки и высадки, анонимизированный номер лицензии водителя и номер медальона (уникальный идентификатор такси). Данные включают в себя все поездки за 2013 год и предоставляются в виде указанных ниже двух наборов данных за каждый месяц.

- Trip_data CSV-файлы содержат сведения о поездках: количество пассажиров, выборка и высадки баллов, длительность поездки и продолжительность поездки. Вот несколько примеров записей:

  `medallion,hack_license,vendor_id,rate_code,store_and_fwd_flag,pickup_datetime,dropoff_datetime,passenger_count,trip_time_in_secs,trip_distance,pickup_longitude,pickup_latitude,dropoff_longitude,dropoff_latitude`

  `89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,1,N,2013-01-01 15:11:48,2013-01-01 15:18:10,4,382,1.00,-73.978165,40.757977,-73.989838,40.751171`

  `0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-06 00:18:35,2013-01-06 00:22:54,1,259,1.50,-74.006683,40.731781,-73.994499,40.75066`

  `0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-05 18:49:41,2013-01-05 18:54:23,1,282,1.10,-74.004707,40.73777,-74.009834,40.726002`

  `DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:54:15,2013-01-07 23:58:20,2,244,.70,-73.974602,40.759945,-73.984734,40.759388`

  `DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:25:03,2013-01-07 23:34:24,1,560,2.10,-73.97625,40.748528,-74.002586,40.747868`

- Trip_fare CSV-файлы содержат сведения о FARE, оплаченные за каждый поездку: тип оплаты, сумма FARE, доплата и налоги, советы и тарифы, а также общая сумма оплаты. Вот несколько примеров записей:

  `medallion, hack_license, vendor_id, pickup_datetime, payment_type, fare_amount, surcharge, mta_tax, tip_amount, tolls_amount, total_amount`

  `89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,2013-01-01 15:11:48,CSH,6.5,0,0.5,0,0,7`

  `0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-06 00:18:35,CSH,6,0.5,0.5,0,0,7`

  `0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-05 18:49:41,CSH,5.5,1,0.5,0,0,7`

  `DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:54:15,CSH,5,0.5,0.5,0,0,6`

  `DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:25:03,CSH,9.5,0.5,0.5,0,0,10.5`

Уникальный ключ для объединения trip\_data и trip\_fare состоит из полей medallion, hack\_license и pickup\_datetime. Чтобы получить все данные, относящиеся к определенной поездке, достаточно выполнить объединение с этими тремя ключами.

## <a name="examples-of-prediction-tasks"></a><a name="mltasks"></a>Примеры задач прогнозирования
Определите тип прогнозов, которые необходимо сделать на основе анализа данных, чтобы помочь уточнить необходимые задачи процесса. Ниже приведены три примера проблем прогнозирования, которые мы будем решать в этом пошаговом руководстве, на основе *\_ суммы TIP*:

- **Двоичная классификация.** Позволяет спрогнозировать, были ли выплачены чаевые за поездку. Т. е. значение *tip\_amount* больше 0 $ — это положительный пример, а значение *tip\_amount* 0 $ — отрицательный пример.

  - Class 0: tip_amount = $0
  - Класс 1: tip_amount > $0

- **Мультиклассовая классификация.** Позволяет спрогнозировать диапазон суммы чаевых за поездку. Мы разделяем *tip\_amount* на пять классов:

  - Class 0: tip_amount = $0
  - Class 1: tip_amount > $0 and tip_amount <= $5
  - Class 2: tip_amount > $5 and tip_amount <= $10
  - Class 3: tip_amount > $10 and tip_amount <= $20
  - Class 4: tip_amount > $20

- **Задача регрессии.** Позволяет спрогнозировать сумму чаевых, выплаченных за поездку.  

## <a name="set-up-an-hdinsight-hadoop-cluster-for-advanced-analytics"></a><a name="setup"></a>Настройка кластера HDInsight Hadoop для расширенной аналитики
> [!NOTE]
> Эту задачу обычно выполняет администратор.
> 
> 

Настроить среду Azure для использования расширенной аналитики, в которой задействуется кластер HDInsight, вы можете в три этапа.

1. [Создание учетной записи хранения](../../storage/common/storage-account-create.md). Эта учетная запись хранения используется для хранения данных в хранилище BLOB-объектов Azure. Здесь находятся также и данные, используемые в кластерах HDInsight.
2. [Настройка кластеров Azure HDInsight Hadoop для технологии и процесса расширенного анализа](../../hdinsight/spark/apache-spark-jupyter-spark-sql.md). На этом этапе создается кластер HDInsight Hadoop, на всех узлах которого установлена 64-разрядная версия Anaconda Python 2.7. Существует два важных действия, которые нужно не забыть выполнить при настройке кластера HDInsight.
   
   * Во время создания кластера HDInsight не забудьте связать учетную запись хранения, созданную на шаге 1, с этим кластером. Эта учетная запись хранения позволяет получить доступ к данным, которые обрабатываются в пределах кластера.
   * После создания кластера включите удаленный доступ к головному узлу кластера. Перейдите на вкладку **Конфигурация** и выберите **Включить удаленный доступ**. На этом шаге указываются учетные данные пользователя для удаленного входа.
3. [Создание рабочей области Машинного обучения Azure.](../classic/create-workspace.md) С помощью этой рабочей области можно создать модели машинного обучения. Эта задача решается после начального исследования данных и сокращения выборки с использованием кластера HDInsight.

## <a name="get-the-data-from-a-public-source"></a><a name="getdata"></a>Получение данных из общедоступного источника
> [!NOTE]
> Эту задачу обычно выполняет администратор.
> 
> 

Если вам нужно скопировать набор данных [Поездки такси Нью-Йорка](https://www.andresmh.com/nyctaxitrips/) из общедоступного расположения на компьютер, вы можете воспользоваться любым из методов, описанных в статье [Перемещение данных в хранилище BLOB-объектов Azure и из него](move-azure-blob.md).

Здесь рассматривается использование AzCopy для передачи файлов, содержащих данные. Чтобы скачать и установить AzCopy, следуйте инструкциям в статье [Приступая к работе со служебной программой командной строки AzCopy](../../storage/common/storage-use-azcopy-v10.md).

1. В окне командной строки выполните следующие команды AzCopy, заменив их на *\<path_to_data_folder>* нужное место назначения:

    ```console
    "C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy" /Source:https://nyctaxitrips.blob.core.windows.net/data /Dest:<path_to_data_folder> /S
    ```

1. По завершении копирования выбранная папка данных будет содержать в общей сложности 24 ZIP-файла. Распакуйте загруженные файлы в тот же каталог на локальном компьютере. Обратите внимание на папку, в которой располагаются распакованные файлы. Эта папка называется, как *\<path\_to\_unzipped_data\_files\>* показано ниже.

## <a name="upload-the-data-to-the-default-container-of-the-hdinsight-hadoop-cluster"></a><a name="upload"></a>Отправка данных в контейнер по умолчанию кластера HDInsight Hadoop
> [!NOTE]
> Эту задачу обычно выполняет администратор.
> 
> 

В следующих командах AzCopy замените приведенные ниже параметры фактическими значениями, указанными при создании кластера Hadoop и распаковке файлов данных.

* ***\<path_to_data_folder>*** Каталог (вместе с путем) на компьютере, который содержит распакованные файлы данных.  
* ***\<storage account name of Hadoop cluster>*** Учетная запись хранения, связанная с кластером HDInsight.
* ***\<default container of Hadoop cluster>*** Контейнер по умолчанию, используемый кластером. Имя контейнера по умолчанию обычно совпадает с именем самого кластера. Например, если кластер называется abc123.azurehdinsight.net, контейнером по умолчанию будет abc123.
* ***\<storage account key>*** Ключ для учетной записи хранения, используемой кластером.

В командной строке или окне Windows PowerShell выполните две приведенные ниже команды AzCopy.

Эта команда передает данные о поездке в каталог ***nyctaxitripraw*** в контейнере по умолчанию кластера Hadoop.

```console
"C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy" /Source:<path_to_unzipped_data_files> /Dest:https://<storage account name of Hadoop cluster>.blob.core.windows.net/<default container of Hadoop cluster>/nyctaxitripraw /DestKey:<storage account key> /S /Pattern:trip_data_*.csv
```

Эта команда передает данные о тарифе в каталог ***nyctaxifareraw*** в контейнере по умолчанию кластера Hadoop.

```console
"C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy" /Source:<path_to_unzipped_data_files> /Dest:https://<storage account name of Hadoop cluster>.blob.core.windows.net/<default container of Hadoop cluster>/nyctaxifareraw /DestKey:<storage account key> /S /Pattern:trip_fare_*.csv
```

Теперь данные должны находиться в хранилище BLOB-объектов и быть готовы к использованию в кластере HDInsight.

## <a name="sign-in-to-the-head-node-of-hadoop-cluster-and-prepare-for-exploratory-data-analysis"></a><a name="#download-hql-files"></a>Вход в головной узел кластера Hadoop и подготовка к исследовательскому анализу данных
> [!NOTE]
> Эту задачу обычно выполняет администратор.
> 
> 

Чтобы получить доступ к головному узлу кластера в целях исследовательского анализа данных и сокращения их выборки, выполните процедуру, описанную в статье [Создание кластера Apache Spark в Azure HDInsight](../../hdinsight/spark/apache-spark-jupyter-spark-sql.md).

Для предварительных исследований данных мы в этом пошаговом руководстве в основном используем запросы, написанные на языке запросов [Hive](https://hive.apache.org/)(он похож на язык SQL). Запросы Hive хранятся в файлах ". HQL". Затем мы сокращаем выборку этих данных, которые будут использоваться в машинном обучении для создания моделей.

Чтобы подготовить кластер для анализа исследовательских данных, Скачайте файлы ". HQL", содержащие соответствующие скрипты Hive, из [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/DataScienceProcess/DataScienceScripts) в локальный каталог (C:\temp) на головном узле. Откройте командную строку в головном узле кластера и выполните следующие две команды:

```console
set script='https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/DataScienceProcess/DataScienceScripts/Download_DataScience_Scripts.ps1'

@powershell -NoProfile -ExecutionPolicy unrestricted -Command "iex ((new-object net.webclient).DownloadString(%script%))"
```

Эти две команды скачивают все файлы ". HQL", необходимые в этом пошаговом руководстве, в локальный каталог ***C:\temp&#92;*** на головном узле.

## <a name="create-hive-database-and-tables-partitioned-by-month"></a><a name="#hive-db-tables"></a>Создание базы данных Hive и таблиц, секционированных по месяцам
> [!NOTE]
> Обычно эта задача предназначена для администратора.
> 
> 

Теперь вы готовы создать таблицы Hive для набора данных о такси Нью-Йорка.
На рабочем столе головного узла кластера Hadoop откройте командную строку Hadoop. Войдите в каталог Hive с помощью следующей команды:

```console
cd %hive_home%\bin
```

> [!NOTE]
> Выполняйте все команды Hive в этом пошаговом руководстве из командной строки каталога bin/ Hive. Таким образом вы сможете автоматически избежать любых проблем с путем. Термины "командная строка каталога Hive", "командная строка Hive bin/directory" и "командная строка Hadoop" в этом руководстве являются взаимозаменяемыми.
> 
> 

В строке каталога Hive выполните следующую команду в командной строке Hadoop головного узла, который создает базу данных и таблицы Hive:

```console
hive -f "C:\temp\sample_hive_create_db_and_tables.hql"
```

Ниже приведено содержимое файла **файла c:\temp\Sample \_ Hive \_ Create \_ DB \_ и \_ Tables. HQL** , в котором создается база данных Hive **nyctaxidb**, а также таблицы для **прохода** и **FARE**.

```hiveql
create database if not exists nyctaxidb;

create external table if not exists nyctaxidb.trip
(
    medallion string,
    hack_license string,
    vendor_id string,
    rate_code string,
    store_and_fwd_flag string,
    pickup_datetime string,
    dropoff_datetime string,
    passenger_count int,
    trip_time_in_secs double,
    trip_distance double,
    pickup_longitude double,
    pickup_latitude double,
    dropoff_longitude double,
    dropoff_latitude double)  
PARTITIONED BY (month int)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' lines terminated by '\n'
STORED AS TEXTFILE LOCATION 'wasb:///nyctaxidbdata/trip' TBLPROPERTIES('skip.header.line.count'='1');

create external table if not exists nyctaxidb.fare
(
    medallion string,
    hack_license string,
    vendor_id string,
    pickup_datetime string,
    payment_type string,
    fare_amount double,
    surcharge double,
    mta_tax double,
    tip_amount double,
    tolls_amount double,
    total_amount double)
PARTITIONED BY (month int)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' lines terminated by '\n'
STORED AS TEXTFILE LOCATION 'wasb:///nyctaxidbdata/fare' TBLPROPERTIES('skip.header.line.count'='1');
```

Этот сценарий Hive создает две таблицы:

* В таблице **поездки** содержатся сведения о каждом выходе каждого из них (сведения о драйвере, время сбора данных, расстояние поездок и время).
* Таблица **FARE** содержит сведения о FARE (сумма FARE, сумма TIP, сборы и платы).

Если вам требуется дополнительная помощь в работе с этими процедурами или вы хотите изучить альтернативы, см. раздел [Отправка запросов Hive](move-hive-tables.md#submit).

## <a name="load-data-to-hive-tables-by-partitions"></a><a name="#load-data"></a>Загрузка данных в таблицы Hive по разделам
> [!NOTE]
> Обычно эта задача предназначена для администратора.
> 
> 

Набор данных о такси Нью-Йорка содержит естественное секционирование по месяцам, которое используется для ускорения обработки и выполнения запросов. Приведенные ниже команды PowerShell (которые вводятся из каталога Hive с помощью командной строки Hadoop) загружают данные в таблицы Hive trip и fare, секционированные по месяцам.

```powershell
for /L %i IN (1,1,12) DO (hive -hiveconf MONTH=%i -f "C:\temp\sample_hive_load_data_by_partitions.hql")
```

**Пример \_ \_ данных загрузки Hive \_ \_ в файл \_ partitions. HQL** содержит следующие команды **Load** :

```hiveql
LOAD DATA INPATH 'wasb:///nyctaxitripraw/trip_data_${hiveconf:MONTH}.csv' INTO TABLE nyctaxidb.trip PARTITION (month=${hiveconf:MONTH});
LOAD DATA INPATH 'wasb:///nyctaxifareraw/trip_fare_${hiveconf:MONTH}.csv' INTO TABLE nyctaxidb.fare PARTITION (month=${hiveconf:MONTH});
```

Ряд запросов Hive, используемых в процессе исследования, подразумевает просмотр только одного или двух разделов. Однако эти запросы можно выполнять по всему набору данных.

### <a name="show-databases-in-the-hdinsight-hadoop-cluster"></a><a name="#show-db"></a>Отображение баз данных в кластере HDInsight Hadoop
Чтобы отобразить базы данных, созданные в кластере HDInsight Hadoop в окне командной строки Hadoop, выполните в командной строке Hadoop приведенную ниже команду.

```console
hive -e "show databases;"
```

### <a name="show-the-hive-tables-in-the-nyctaxidb-database"></a><a name="#show-tables"></a>Отображение таблиц Hive в базе данных **nyctaxidb**
Для отображения таблиц в базе данных **nyctaxidb** выполните в командной строке Hadoop следующую команду.

```console
hive -e "show tables in nyctaxidb;"
```

Мы можем убедиться, что таблицы секционированы, выполнив следующую команду.

```console
hive -e "show partitions nyctaxidb.trip;"
```

Вот, что мы должны были увидеть.

```output
month=1
month=10
month=11
month=12
month=2
month=3
month=4
month=5
month=6
month=7
month=8
month=9
Time taken: 2.075 seconds, Fetched: 12 row(s)
```

Аналогично мы можем убедиться в том, что таблица fare секционирована, выполнив следующую команду.

```console
hive -e "show partitions nyctaxidb.fare;"
```

Вот, что мы должны были увидеть.

```output
month=1
month=10
month=11
month=12
month=2
month=3
month=4
month=5
month=6
month=7
month=8
month=9
Time taken: 1.887 seconds, Fetched: 12 row(s)
```

## <a name="data-exploration-and-feature-engineering-in-hive"></a><a name="#explore-hive"></a>Исследование данных и проектирование признаков в Hive
> [!NOTE]
> Обычно эту задачу выполняет специалист по обработке и анализу данных.
> 
> 

С помощью запросов Hive можно выполнить задачи, которые связаны с просмотром данных и созданием функций и которые относятся к данным, загруженным в таблицы Hive. Ниже приведены примеры таких задач.

* Просмотр 10 верхних записей в обеих таблицах.
* Просмотрим распределение данных по нескольким полям в различных временных окнах.
* Исследуем качество данных по полям долготы и широты.
* Создадим метки двоичной и мультиклассовой классификации на основе суммы чаевых.
* Создание функций путем вычисления дальности поездки.

### <a name="exploration-view-the-top-10-records-in-table-trip"></a>Исследование: просмотр 10 верхних записей в таблице trip
> [!NOTE]
> Обычно эту задачу выполняет специалист по обработке и анализу данных.
> 
> 

Чтобы увидеть, как выглядят данные, изучите по 10 записей из каждой таблицы. Чтобы просмотреть записи, выполните приведенные ниже два запроса по отдельности в консоли командной строки Hadoop из каталога Hive.

Чтобы получить первые 10 записей в таблице trip за первый месяц, выполните следующий запрос:

```console
hive -e "select * from nyctaxidb.trip where month=1 limit 10;"
```

Чтобы получить первые 10 записей в таблице fare за первый месяц, выполните следующий запрос:

```console
hive -e "select * from nyctaxidb.fare where month=1 limit 10;"
```

Можно сохранить записи в файл для удобного просмотра с помощью небольшого изменения предыдущего запроса:

```console
hive -e "select * from nyctaxidb.fare where month=1 limit 10;" > C:\temp\testoutput
```

### <a name="exploration-view-the-number-of-records-in-each-of-the-12-partitions"></a>Исследование: просмотр количества записей в каждом из 12 разделов
> [!NOTE]
> Обычно эту задачу выполняет специалист по обработке и анализу данных.
> 
> 

Интерес представляет варьирование числа поездок в течение календарного года. Группирование по месяцам отображает распределение поездок.

```console
hive -e "select month, count(*) from nyctaxidb.trip group by month;"
```

Эта команда создает следующие выходные данные:

```output
1       14776615
2       13990176
3       15749228
4       15100468
5       15285049
6       14385456
7       13823840
8       12597109
9       14107693
10      15004556
11      14388451
12      13971118
Time taken: 283.406 seconds, Fetched: 12 row(s)
```

Здесь первый столбец обозначает месяц, а второй содержит количество поездок за этот месяц.

Кроме того, мы можем подсчитать общее число записей в нашем наборе данных о поездках, выполнив следующую команду в командной строке каталога Hive.

```console
hive -e "select count(*) from nyctaxidb.trip;"
```

Эта команда возвращает:

```output
173179759
Time taken: 284.017 seconds, Fetched: 1 row(s)
```

Чтобы проверить количество записей, мы можем с помощью команд, подобных показанным для набора данных о поездках, выполнить запросы Hive в командной строке каталога Hive для набора данных о тарифах.

```console
hive -e "select month, count(*) from nyctaxidb.fare group by month;"
```

Эта команда создает следующие выходные данные:

```output
1       14776615
2       13990176
3       15749228
4       15100468
5       15285049
6       14385456
7       13823840
8       12597109
9       14107693
10      15004556
11      14388451
12      13971118
Time taken: 253.955 seconds, Fetched: 12 row(s)
```

Для обоих наборов данных возвращается точно такое же количество обращений в месяц, что обеспечивает первую проверку того, что данные были загружены правильно.

Вы можете подсчитать общее число записей в наборе данных о поездках, выполнив следующую команду в командной строке каталога Hive.

```console
hive -e "select count(*) from nyctaxidb.fare;"
```

Эта команда возвращает:

```output
173179759
Time taken: 186.683 seconds, Fetched: 1 row(s)
```

Общее число записей в обеих таблицах также одинаково, что обеспечивает вторую проверку правильности загрузки данных.

### <a name="exploration-trip-distribution-by-medallion"></a>Просмотр: распределение поездок по параметру medallion
> [!NOTE]
> Этот анализ обычно является задачей анализу данных.
> 
> 

В этом примере определяются медальоны (номера такси) с более чем 100 поездками за заданный период времени. Этот запрос выигрывает от доступа к секционированной таблице, так как в нем содержится условие, заданное переменной раздела **month**. Результаты запроса записываются в локальный файл **queryoutput. tsv** в `C:\temp` на головном узле.

```console
hive -f "C:\temp\sample_hive_trip_count_by_medallion.hql" > C:\temp\queryoutput.tsv
```

Вот содержимое файла **sample\_hive\_trip\_count\_by\_medallion.hql** для проверки.

```hiveql
SELECT medallion, COUNT(*) as med_count
FROM nyctaxidb.fare
WHERE month<=3
GROUP BY medallion
HAVING med_count > 100
ORDER BY med_count desc;
```

Медальон в наборе данных о такси Нью-Йорка идентифицирует уникальный автомобиль такси. Вы можете определить, какие автомобили сравнительно востребованы, запросив, какие из них выполнили более определенного количества поездок за определенный период времени. В указанном ниже примере идентифицируются автомобили, которые выполнили более ста поездок за первые три месяца, и результаты запроса сохраняются в локальном файле **C:\temp\queryoutput.tsv**.

Вот содержимое файла **sample\_hive\_trip\_count\_by\_medallion.hql** для проверки.

```hiveql
SELECT medallion, COUNT(*) as med_count
FROM nyctaxidb.fare
WHERE month<=3
GROUP BY medallion
HAVING med_count > 100
ORDER BY med_count desc;
```

В командной строке каталога Hive выполните следующую команду.

```console
hive -f "C:\temp\sample_hive_trip_count_by_medallion.hql" > C:\temp\queryoutput.tsv
```

### <a name="exploration-trip-distribution-by-medallion-and-hack-license"></a>Изучение: распределение поездок по медальону и номеру лицензии водителя
> [!NOTE]
> Обычно эта задача предназначена для анализу данных.
> 
> 

При изучении набора данных часто требуется изучить распределения групп значений. В этом разделе приведен пример того, как выполнить этот анализ для CAB и драйверов.

Файл **sample\_hive\_trip\_count\_by\_medallion\_license.hql** группирует набор данных о тарифах по параметрам **medallion** и **hack_license** и возвращает количество вхождений каждой из комбинаций. Вот его содержимое:

```hiveql
SELECT medallion, hack_license, COUNT(*) as trip_count
FROM nyctaxidb.fare
WHERE month=1
GROUP BY medallion, hack_license
HAVING trip_count > 100
ORDER BY trip_count desc;
```

Этот запрос возвращает комбинации автомобиля и водителя, упорядоченные по убыванию количества поездок.

В командной строке каталога Hive выполните приведенную ниже команду.

```console
hive -f "C:\temp\sample_hive_trip_count_by_medallion_license.hql" > C:\temp\queryoutput.tsv
```

Результаты запроса записываются в локальный файл **C:\temp\queryoutput.tsv**.

### <a name="exploration-assessing-data-quality-by-checking-for-invalid-longitude-or-latitude-records"></a>Исследование: оценка качества данных через проверку наличия недопустимых записей долготы или широты
> [!NOTE]
> Обычно эту задачу выполняет специалист по обработке и анализу данных.
> 
> 

Общей целью исследовательского анализа данных является отсеивание недопустимых или неверных записей. Пример в этом разделе определяет, содержатся ли в полях долготы (longitude) и широты (latitude) значения, указывающие на местоположения далеко за пределами региона Нью-Йорка. Так как вероятно, что в таких записях будут содержаться ошибочные значения долготы и широты, нам следует устранить их из любых данных, которые будут использоваться для моделирования.

Вот содержимое файла **sample\_hive\_quality\_assessment.hql** для проверки.

```hiveql
    SELECT COUNT(*) FROM nyctaxidb.trip
    WHERE month=1
    AND  (CAST(pickup_longitude AS float) NOT BETWEEN -90 AND -30
    OR    CAST(pickup_latitude AS float) NOT BETWEEN 30 AND 90
    OR    CAST(dropoff_longitude AS float) NOT BETWEEN -90 AND -30
    OR    CAST(dropoff_latitude AS float) NOT BETWEEN 30 AND 90);
```

В командной строке каталога Hive выполните приведенную ниже команду.

```console
hive -S -f "C:\temp\sample_hive_quality_assessment.hql"
```

Аргумент *-S* этой команды скрывает окно состояния задач Hive Map/Reduce. Эта команда полезна, так как делает вывод на экран вывода запросов Hive более читаемым.

### <a name="exploration-binary-class-distributions-of-trip-tips"></a>Исследования: двоичные классовые распределения чаевых за поездки
> [!NOTE]
> Обычно эту задачу выполняет специалист по обработке и анализу данных.
> 
> 

Касательно проблемы двоичной классификации, изложенной в разделе [Примеры задач прогнозирования](hive-walkthrough.md#mltasks) , полезно знать, были ли выплачены чаевые за поездку. Распределение чаевых является двоичным:

* чаевые оставлены (класс 1, tip\_amount > $0);  
* чаевые не оставлены (класс 0, tip\_amount = $0).

В следующем **примере \_ файла \_ HQL с \_ невысоких частотой использования Hive** показана команда для запуска:

```hiveql
SELECT tipped, COUNT(*) AS tip_freq
FROM
(
    SELECT if(tip_amount > 0, 1, 0) as tipped, tip_amount
    FROM nyctaxidb.fare
)tc
GROUP BY tipped;
```

В командной строке каталога Hive выполните приведенную ниже команду.

```console
hive -f "C:\temp\sample_hive_tipped_frequencies.hql"
```


### <a name="exploration-class-distributions-in-the-multiclass-setting"></a>Исследование: классовые распределения при мультиклассовой настройке
> [!NOTE]
> Обычно эту задачу выполняет специалист по обработке и анализу данных.
> 
> 

Для проблемы мультиклассовой классификации, изложенной в разделе [Примеры задач прогнозирования](hive-walkthrough.md#mltasks), этот набор данных тоже поддается естественной классификации, если нужно спрогнозировать сумму выплачиваемых чаевых. Для определения диапазонов чаевых в запросе можно использовать ячейки. Для получения классовых распределений различных диапазонов чаевых используйте файл **sample\_hive\_tip\_range\_frequencies.hql**. Вот его содержимое:

```hiveql
SELECT tip_class, COUNT(*) AS tip_freq
FROM
(
    SELECT if(tip_amount=0, 0,
        if(tip_amount>0 and tip_amount<=5, 1,
        if(tip_amount>5 and tip_amount<=10, 2,
        if(tip_amount>10 and tip_amount<=20, 3, 4)))) as tip_class, tip_amount
    FROM nyctaxidb.fare
)tc
GROUP BY tip_class;
```

В консоли командной строки Hadoop выполните приведенную ниже команду.

```console
hive -f "C:\temp\sample_hive_tip_range_frequencies.hql"
```

### <a name="exploration-compute-the-direct-distance-between-two-longitude-latitude-locations"></a>Исследование: вычисление расстояния по прямой между двумя местоположениями, заданными долготой и широтой
> [!NOTE]
> Обычно эту задачу выполняет специалист по обработке и анализу данных.
> 
> 

Вам может потребоваться выяснить, имеется ли расхождение в расстоянии между двумя расположениями и фактическом расстоянии поездки такси. Пассажир с меньшей вероятностью выплатит чаевые, если определит, что водитель намеренно вез его намного более длинным маршрутом.

Для просмотра сравнения между фактическим расстоянием поездки и [расстоянием гаверсинуса](https://en.wikipedia.org/wiki/Haversine_formula) между двумя точками долготы и широты (расстоянием по "большому кругу") можете использовать доступные в Hive тригонометрические функции.

```hiveql
set R=3959;
set pi=radians(180);

insert overwrite directory 'wasb:///queryoutputdir'

select pickup_longitude, pickup_latitude, dropoff_longitude, dropoff_latitude, trip_distance, trip_time_in_secs,
${hiveconf:R}*2*2*atan((1-sqrt(1-pow(sin((dropoff_latitude-pickup_latitude)
 *${hiveconf:pi}/180/2),2)-cos(pickup_latitude*${hiveconf:pi}/180)
 *cos(dropoff_latitude*${hiveconf:pi}/180)*pow(sin((dropoff_longitude-pickup_longitude)*${hiveconf:pi}/180/2),2)))
 /sqrt(pow(sin((dropoff_latitude-pickup_latitude)*${hiveconf:pi}/180/2),2)
 +cos(pickup_latitude*${hiveconf:pi}/180)*cos(dropoff_latitude*${hiveconf:pi}/180)*
 pow(sin((dropoff_longitude-pickup_longitude)*${hiveconf:pi}/180/2),2))) as direct_distance
from nyctaxidb.trip
where month=1
and pickup_longitude between -90 and -30
and pickup_latitude between 30 and 90
and dropoff_longitude between -90 and -30
and dropoff_latitude between 30 and 90;
```

В предыдущем запросе R — это радиус Земли в милях, а число пи (pi) преобразуется в радианы. Точки долготы и широты фильтруются для удаления значений, наявляющихся далеко от области Нью.

В этом случае мы запишем результаты в каталог с именем **queryoutputdir**. Последовательность команд, приведенных ниже, сначала создает этот выходной каталог, а затем выполняет команду Hive.

В командной строке каталога Hive выполните приведенную ниже команду.

```hiveql
hdfs dfs -mkdir wasb:///queryoutputdir

hive -f "C:\temp\sample_hive_trip_direct_distance.hql"
```

Результаты запроса записываются в девять больших двоичных объектов Azure 9 (с **queryoutputdir/000000\_0** по **queryoutputdir/000008\_0**) в используемом по умолчанию контейнере кластера Hadoop.

Чтобы просмотреть размер отдельных больших двоичных объектов, выполните приведенную ниже команду в командной строке каталога Hive:

```hiveql
hdfs dfs -ls wasb:///queryoutputdir
```

Чтобы просмотреть содержимое заданного файла, скажем **000000\_0**, выполните команду Hadoop `copyToLocal`.

```hiveql
hdfs dfs -copyToLocal wasb:///queryoutputdir/000000_0 C:\temp\tempfile
```

> [!WARNING]
> `copyToLocal` может выполняться очень медленно для больших файлов, и использовать его с ними не рекомендуется.  
> 
> 

Ключевое преимущество размещения этих данных в большом двоичном объекте Azure заключается в том, что мы можем исследовать данные в машинном обучении с помощью модуля [Импорт данных][import-data].

## <a name="down-sample-data-and-build-models-in-machine-learning"></a><a name="#downsample"></a>Сокращение выборки и создание моделей в машинном обучении
> [!NOTE]
> Обычно эту задачу выполняет специалист по обработке и анализу данных.
> 
> 

После исследовательской фазы анализа данных мы готовы к сокращению их выборки для создания моделей в машинном обучении. В этом разделе показано, как использовать запрос Hive, чтобы уменьшить выборку данных. Вы можете получить доступ к ней с помощью модуля [Импорт данных][import-data] в машинном обучении.

### <a name="down-sampling-the-data"></a>Сокращение выборки данных
Эта процедура состоит из двух шагов. Сначала мы соединяем таблицы **nyctaxidb. поездку** и **nyctaxidb. FARE** на трех ключах, которые находятся во всех записях: **Medallion**, **\_ Лицензия хакера** и **Дата и \_ время отправки**. Затем мы создаем метку двоичной классификации **tipped** и метку мультиклассовой классификации **tip\_class**.

Чтобы использовать данные сокращенной выборки непосредственно из модуля [Импорт данных][import-data] в машинном обучении, необходимо сохранить результаты приведенного выше запроса во внутренней таблице Hive. В дальнейшем мы создаем внутреннюю таблицу Hive и заполняем ее содержимое объединенными данными сокращенной выборки.

Запрос применяет стандартные функции Hive напрямую для создания следующих параметров времени из поля **\_ даты отправки** .
- час дня;
- неделя года;
- будний день ("1" означает понедельник, а "7" означает воскресенье)

Запрос также формирует расстояние по прямой между местоположениями посадки и высадки. Полный список определяемых пользователем функций см. на странице [LanguageManual UDF](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF) (Руководство по языку: определяемые пользователем функции).

Затем запрос сокращает выборку данных так, чтобы результаты запроса можно было загрузить в Студию машинного обучения Azure. В студию импортируется только около 1 процента исходного набора данных.

Далее приведено содержимое файла **sample\_hive\_prepare\_for\_aml\_full.hql**, подготавливающего данные к созданию моделей в машинном обучении.

```hiveql
set R = 3959;
set pi=radians(180);

create table if not exists nyctaxidb.nyctaxi_downsampled_dataset (

medallion string,
hack_license string,
vendor_id string,
rate_code string,
store_and_fwd_flag string,
pickup_datetime string,
dropoff_datetime string,
pickup_hour string,
pickup_week string,
weekday string,
passenger_count int,
trip_time_in_secs double,
trip_distance double,
pickup_longitude double,
pickup_latitude double,
dropoff_longitude double,
dropoff_latitude double,
direct_distance double,
payment_type string,
fare_amount double,
surcharge double,
mta_tax double,
tip_amount double,
tolls_amount double,
total_amount double,
tipped string,
tip_class string
)
row format delimited fields terminated by ','
lines terminated by '\n'
stored as textfile;

--- now insert contents of the join into the above internal table

insert overwrite table nyctaxidb.nyctaxi_downsampled_dataset
select
t.medallion,
t.hack_license,
t.vendor_id,
t.rate_code,
t.store_and_fwd_flag,
t.pickup_datetime,
t.dropoff_datetime,
hour(t.pickup_datetime) as pickup_hour,
weekofyear(t.pickup_datetime) as pickup_week,
from_unixtime(unix_timestamp(t.pickup_datetime, 'yyyy-MM-dd HH:mm:ss'),'u') as weekday,
t.passenger_count,
t.trip_time_in_secs,
t.trip_distance,
t.pickup_longitude,
t.pickup_latitude,
t.dropoff_longitude,
t.dropoff_latitude,
t.direct_distance,
f.payment_type,
f.fare_amount,
f.surcharge,
f.mta_tax,
f.tip_amount,
f.tolls_amount,
f.total_amount,
if(tip_amount>0,1,0) as tipped,
if(tip_amount=0,0,
if(tip_amount>0 and tip_amount<=5,1,
if(tip_amount>5 and tip_amount<=10,2,
if(tip_amount>10 and tip_amount<=20,3,4)))) as tip_class

from
(
select
medallion,
hack_license,
vendor_id,
rate_code,
store_and_fwd_flag,
pickup_datetime,
dropoff_datetime,
passenger_count,
trip_time_in_secs,
trip_distance,
pickup_longitude,
pickup_latitude,
dropoff_longitude,
dropoff_latitude,
${hiveconf:R}*2*2*atan((1-sqrt(1-pow(sin((dropoff_latitude-pickup_latitude)
*${hiveconf:pi}/180/2),2)-cos(pickup_latitude*${hiveconf:pi}/180)
*cos(dropoff_latitude*${hiveconf:pi}/180)*pow(sin((dropoff_longitude-pickup_longitude)*${hiveconf:pi}/180/2),2)))
/sqrt(pow(sin((dropoff_latitude-pickup_latitude)*${hiveconf:pi}/180/2),2)
+cos(pickup_latitude*${hiveconf:pi}/180)*cos(dropoff_latitude*${hiveconf:pi}/180)*pow(sin((dropoff_longitude-pickup_longitude)*${hiveconf:pi}/180/2),2))) as direct_distance,
rand() as sample_key

from nyctaxidb.trip
where pickup_latitude between 30 and 90
    and pickup_longitude between -90 and -30
    and dropoff_latitude between 30 and 90
    and dropoff_longitude between -90 and -30
)t
join
(
select
medallion,
hack_license,
vendor_id,
pickup_datetime,
payment_type,
fare_amount,
surcharge,
mta_tax,
tip_amount,
tolls_amount,
total_amount
from nyctaxidb.fare
)f
on t.medallion=f.medallion and t.hack_license=f.hack_license and t.pickup_datetime=f.pickup_datetime
where t.sample_key<=0.01
```

Выполнение этого запроса в командной строке каталога Hive:

```console
hive -f "C:\temp\sample_hive_prepare_for_aml_full.hql"
```

Теперь у нас есть внутренняя таблица **nyctaxidb.nyctaxi_downsampled_dataset**, к которой можно получить доступ с помощью модуля [Импорт данных][import-data] в машинном обучении. Кроме того, мы можем использовать этот набор данных для создания моделей машинного обучения.  

### <a name="use-the-import-data-module-in-machine-learning-to-access-the-down-sampled-data"></a>Использование модуля "Импорт данных" в машинном обучении для доступа к данным сокращенной выборки
Чтобы выполнять запросы Hive в модуле [Импорт данных][import-data] машинного обучения, необходимо получить доступ к рабочей области машинного обучения. Кроме того, вам понадобится доступ к учетным данным кластера и связанной с ним учетной записи хранения.

Далее приведены некоторые сведения о модуле [Импорт данных][import-data] и входных параметрах:

**URI сервера HCatalog**: Если имя кластера — **abc123**, используйте: https: \/ /abc123.azurehdinsight.NET.

**Имя учетной записи пользователя Hadoop**: имя пользователя, выбранное для кластера (не имя пользователя удаленного доступа).

**Пароль учетной записи пользователя Hadoop**— пароль, выбранный для кластера (а не пароль удаленного доступа).

**Расположение выходных данных**: выбрано в качестве Azure.

**Имя учетной записи хранения Azure**: имя учетной записи хранения по умолчанию, связанной с кластером.

**Имя контейнера Azure**. имя контейнера по умолчанию для кластера и обычно совпадает с именем кластера. Для кластера с именем **abc123** имя — abc123.

> [!IMPORTANT]
> Любая таблица, к которой нам требуется отправить запрос с помощью модуля [Импорт данных][import-data] в машинном обучении, должна быть внутренней таблицей.
> 
> 

Чтобы определить, является ли таблица **T** в базе данных **D.db** внутренней таблицей, выполните приведенные ниже действия. В командной строке каталога Hive выполните следующую команду.

```hiveql
hdfs dfs -ls wasb:///D.db/T
```

Если таблица является внутренней и заполнена, ее содержимое должно отобразиться здесь.

Другой способ определить, является ли таблица внутренней, — использовать Обозреватель службы хранилища Azure. С его помощью перейдите к используемому по умолчанию имени контейнера кластера, а затем выполните фильтрацию по имени таблицы. Если таблица и ее содержимое отображаются, это подтверждает, что таблица является внутренней.

Далее приведен снимок экрана запроса Hive и модуля [Импорт данных][import-data].

![Снимок экрана: запрос Hive для модуля "Импорт данных"](./media/hive-walkthrough/1eTYf52.png)

Так как данные пониженной выборки находятся в контейнере по умолчанию, полученный запрос Hive из Машинное обучение прост. Это **SELECT * FROM nyctaxidb.nyctaxi\_downsampled\_data**.

Теперь набор данных можно использовать в качестве отправной точки для создания моделей машинного обучения.

### <a name="build-models-in-machine-learning"></a><a name="mlmodel"></a>Создание моделей в компоненте машинного обучения
Теперь вы можете перейти к созданию и развертыванию модели в [компоненте машинного обучения](https://studio.azureml.net). Данные готовы к использованию для решения задач прогнозирования, определенных ранее.

- **Двоичная классификация**: для прогнозирования того, оплачивается ли TIP для поездки.

  **Используемое средство обучения:** двухклассовая логистическая регрессия.

  а. Для задачи целевой меткой (меткой класса) будет **tipped** (чаевые выплачены). В исходном наборе данных сокращенной выборки есть несколько столбцов, содержащих целевые утечки сведений для данного эксперимента по классификации. В частности, **\_ класс TIP**, **\_ значение TIP** и **Общая \_ Сумма** раскрывают сведения о целевой метке, недоступные во время тестирования. Мы удаляем эти столбцы из рассмотрения с помощью модуля [Select Columns in Dataset][select-columns] (Выбор столбцов в наборе данных).

  На схеме ниже показан наш эксперимент по прогнозированию выплаты чаевых за эту поездку.

  ![Схема эксперимента по прогнозированию выплаты чаевых](./media/hive-walkthrough/QGxRz5A.png)

  b. Для этого эксперимента распределение наших целевых меток составляло примерно 1:1.

   На схеме ниже показано распределение меток класса чаевых для задачи двоичной классификации.

  ![Схема распределения меток класса чаевых](./media/hive-walkthrough/9mM4jlD.png)

    В результате мы получаем значения для области под кривой (AUC) величиной 0,987, как показано на рисунке ниже.

  ![Схема значения AUC](./media/hive-walkthrough/8JDT0F8.png)

- **Мультиклассовая классификация**: прогнозирование диапазона сумм чаевых за поездку с использованием заранее определенных классов.

  **Используемое средство обучения:** мультиклассовая логистическая регрессия.

  а. Для этой проблемы наша конечная метка (или класс) является **\_ классом TIP**, который может принимать одно из пяти значений (0, 1, 2, 3, 4). Как и в случае двоичной классификации, у нас есть несколько столбцов с утечкой целевой информации для этого эксперимента. В частности, **tipped**, **tip\_amount** и **total\_amount** раскрывают информацию о целевой метке, которая недоступна во время тестирования. Мы удаляем эти столбцы с помощью модуля [Select Columns in Dataset][select-columns] (Выбор столбцов в наборе данных).

  На схеме ниже показан наш эксперимент по прогнозированию ячейки, в которую вероятнее всего попадет сумма чаевых. К этим ячейкам относятся: класс 0: чаевые = 0 $, класс 1: чаевые > 0 $ и чаевые <= 5 $, класс 2: чаевые > 5 $ и чаевые <= 10 $, класс 3: чаевые > 10 $ и чаевые <= 20 $, класс 4: чаевые > 20 $.

  ![Схема эксперимента по прогнозированию ячеек чаевых](./media/hive-walkthrough/5ztv0n0.png)

  Теперь мы покажем, как выглядит наше фактическое тестовое классовое распределение. Классы 0 и 1 являются наиболее распространенными, а другие классы редки.

  ![Схема тестового классового распределения](./media/hive-walkthrough/Vy1FUKa.png)

  b. Для этого эксперимента мы используем матрицу путаницы для просмотра точности прогнозов, как показано ниже:

  ![Матрица неточностей](./media/hive-walkthrough/cxFmErM.png)

  Хотя классные классы в самых распространенных классах хорошо подходят, модель не выполняет хорошее задание "обучения" в классах реже.

- **Регрессионная задача**: прогнозирование количества подсказок, оплаченных за поездку.

  **Используемое средство обучения:** повышенное дерево принятия решений.

  а. Для задачи целевой меткой (меткой класса) будет **tip\_amount** (сумма чаевых). Целевые утечки в этом случае: **tipped**, **tip\_class** и **total\_amount**. Все эти переменные раскрывают информацию о сумме чаевых, которая обычно недоступна во время тестирования. Мы удаляем эти столбцы с помощью модуля [Select Columns in Dataset][select-columns] (Выбор столбцов в наборе данных).

  На схеме ниже показан эксперимент по прогнозированию суммы выплаченных чаевых.

  ![Схема эксперимента по прогнозированию суммы чаевых](./media/hive-walkthrough/11TZWgV.png)

  b. Для задач регрессии мы измеряем величины точности прогноза, рассматривая квадратичную ошибку в прогнозах и коэффициент детерминации.

  ![Снимок экрана: статистические данные прогноза](./media/hive-walkthrough/Jat9mrz.png)

  Здесь коэффициент детерминации равен 0,709, что подразумевает, что коэффициенты модели объясняют примерно 71 процент дисперсии.

> [!IMPORTANT]
> Чтобы узнать больше о службе машинного обучения, о доступе к ней и ее использовании, см. статью [Введение в машинное обучение в облаке Azure](../classic/index.yml). Кроме того, [коллекция решений ИИ Azure](https://gallery.cortanaintelligence.com/) охватывает спектр экспериментов и служит подробным введением в диапазон возможностей машинного обучения.
> 
> 

## <a name="license-information"></a>Сведения о лицензии
Доступ к этому учебнику и включенным в него скриптам предоставляет корпорация Майкрософт на основании лицензии свободного ПО. Дополнительные сведения см. в файле **LICENSE.txt** в каталоге примера кода на сайте GitHub.

## <a name="references"></a>Ссылки
•    [Страница Андреса Монройя для скачивания данных о поездках в такси Нью-Йорка](https://www.andresmh.com/nyctaxitrips/)  
•    [Данные о поездках в такси получение Нью от Крис whong)](https://chriswhong.com/open-data/foil_nyc_taxi/)   
•    [Исследование и статистика Нью такси и лимузинов](https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

[2]: ./media/hive-walkthrough/output-hive-results-3.png
[11]: ./media/hive-walkthrough/hive-reader-properties.png
[12]: ./media/hive-walkthrough/binary-classification-training.png
[13]: ./media/hive-walkthrough/create-scoring-experiment.png
[14]: ./media/hive-walkthrough/binary-classification-scoring.png
[15]: ./media/hive-walkthrough/amlreader.png

<!-- Module References -->
[select-columns]: /azure/machine-learning/studio-module-reference/select-columns-in-dataset
[import-data]: /azure/machine-learning/studio-module-reference/import-data