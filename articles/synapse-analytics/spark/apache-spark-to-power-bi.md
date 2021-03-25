---
title: Записные книжки Azure Synapse Studio
description: В этом руководстве приведены общие сведения о том, как создать панель мониторинга Power BI с помощью Apache Spark и бессерверного пула SQL.
services: synapse-analytics
author: midesa
ms.author: midesa
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.subservice: spark
ms.topic: tutorial
ms.date: 11/16/2020
ms.openlocfilehash: 53fb256d3a0fba1cc10cafb239ff7f7bb21ac4eb
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98121079"
---
# <a name="tutorial-create-a-power-bi-report-using-apache-spark-and-azure-synapse-analytics"></a>Руководство по созданию отчета Power BI с помощью Apache Spark и Azure Synapse Analytics

Организациям часто приходится обрабатывать большие объемы данных, прежде чем они смогут предоставить их основным заинтересованным лицам. Из этого руководства вы узнаете, как использовать интегрированные возможности Azure Synapse Analytics для обработки данных с помощью Apache Spark и как затем предоставлять результаты такой обработки пользователям через Power BI и бессерверный пул SQL.

## <a name="before-you-begin"></a>Перед началом
- [Рабочая область Azure Synapse Analytics](../quickstart-create-workspace.md) с учетной записью хранения ADLS 2-го поколения, настроенной в качестве хранилища по умолчанию. 
- Рабочая область Power BI и Power BI Desktop для визуализации данных. Дополнительные сведения см. в статьях о [создании рабочей области Power BI](/power-bi/service-create-the-new-workspaces) и [установке Power BI Desktop](https://powerbi.microsoft.com/downloads/).
- Связанная служба для подключения Azure Synapse Analytics и рабочих областей Power BI. Дополнительные сведения см. в статье о [связывании рабочей области Power BI](../quickstart-power-bi.md).
- Бессерверный пул Apache Spark в рабочей области Synapse Analytics. Дополнительные сведения см. в статье о [создании бессерверного пула Apache Spark](../quickstart-create-apache-spark-pool-studio.md).
  
## <a name="download-and-prepare-the-data"></a>Скачивание и подготовка данных
В этом примере вы с помощью Apache Spark выполните некоторый анализ данных о чаевых таксистам Нью-Йорка. Данные доступны через [открытые наборы данных Azure](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/). Это подмножество набора данных содержит сведения о поездках на такси, включая сведения о каждой поездке, времени начала и окончания и расположениях, стоимости и других интересных атрибутах.

1. Выполните следующие строки, чтобы создать кадр данных Spark, вставив код в новую ячейку. При этом данные извлекаются через API Открытых наборов данных. Получение всех этих данных приводит к созданию порядка 1 500 000 000 строк. В следующем примере кода используются параметры start_date и end_date для применения фильтра, который возвращает данные за один месяц.
   
   ```python
    from azureml.opendatasets import NycTlcYellow
    from dateutil import parser
    from datetime import datetime

    end_date = parser.parse('2018-06-06')
    start_date = parser.parse('2018-05-01')
    nyc_tlc = NycTlcYellow(start_date=start_date, end_date=end_date)
    filtered_df = nyc_tlc.to_spark_dataframe()
   ```
2. С помощью Apache Spark SQL мы создадим базу данных с именем NycTlcTutorial. Мы будем использовать эту базу данных для хранения результатов обработки данных.
   ```python
   %%pyspark
    spark.sql("CREATE DATABASE IF NOT EXISTS NycTlcTutorial")
   ```
3. Далее вы обработаем данные с помощью операций с кадрами данных в Spark. Используя приведенный ниже код, мы выполним следующие преобразования:
   1. Удаление ненужных столбцов.
   2. Удаление выбросов/неверных значений с помощью фильтрации.
   3. Создание новых признаков, например ```tripTimeSecs``` и ```tipped```, для дополнительного анализа.
    ```python
    from pyspark.sql.functions import unix_timestamp, date_format, col, when

    taxi_df = filtered_df.select('totalAmount', 'fareAmount', 'tipAmount', 'paymentType', 'rateCodeId', 'passengerCount'\
                                    , 'tripDistance', 'tpepPickupDateTime', 'tpepDropoffDateTime'\
                                    , date_format('tpepPickupDateTime', 'hh').alias('pickupHour')\
                                    , date_format('tpepPickupDateTime', 'EEEE').alias('weekdayString')\
                                    , (unix_timestamp(col('tpepDropoffDateTime')) - unix_timestamp(col('tpepPickupDateTime'))).alias('tripTimeSecs')\
                                    , (when(col('tipAmount') > 0, 1).otherwise(0)).alias('tipped')
                                    )\
                            .filter((filtered_df.passengerCount > 0) & (filtered_df.passengerCount < 8)\
                                    & (filtered_df.tipAmount >= 0) & (filtered_df.tipAmount <= 25)\
                                    & (filtered_df.fareAmount >= 1) & (filtered_df.fareAmount <= 250)\
                                    & (filtered_df.tipAmount < filtered_df.fareAmount)\
                                    & (filtered_df.tripDistance > 0) & (filtered_df.tripDistance <= 100)\
                                    & (filtered_df.rateCodeId <= 5)
                                    & (filtered_df.paymentType.isin({"1", "2"})))
    ```
4. Наконец, мы сохраним наш кадр данных с помощью метода Apache Spark ```saveAsTable```. Это позволит вам в дальнейшем отправлять запросы к таблице и подключаться к ней с помощью бессерверных пулов SQL.
  ```python
     taxi_df.write.mode("overwrite").saveAsTable("NycTlcTutorial.nyctaxi")
  ```
   
## <a name="query-data-using-serverless-sql-pools"></a>Запрос данных с помощью бессерверных пулов SQL
Azure Synapse Analytics позволяет различным вычислительным системам рабочих областей использовать базы данных и таблицы совместно с бессерверными пулами Apache Spark и бессерверным пулом SQL. Такая возможность обеспечивается [системой управления общими метаданными](../metadata/overview.md) в Synapse. В результате созданные с помощью Spark базы данных и их таблицы с поддержкой Parquet становятся видимыми в бессерверном пуле SQL рабочей области.

Чтобы отправить запрос к таблице Apache Spark с помощью бессерверного пула SQL, выполните следующие действия:
   1. После сохранения таблицы Apache Spark переключитесь на вкладку **данных**.
   
   2. В разделе **Рабочие области** найдите созданную вами таблицу Apache Spark и выберите **Создать скрипт SQL** и **Выбрать первые 100 строк**. 
      
      :::image type="content" source="../spark/media/apache-spark-power-bi/query-spark-table-with-sql.png" alt-text="SQL-запрос" border="true":::

   3. Вы можете отредактировать запрос или даже визуализировать результаты с помощью параметров построения диаграмм SQL.

## <a name="connect-to-power-bi"></a>Подключение к Power BI
Теперь мы подключим бессерверный пул SQL к рабочей области Power BI. После подключения рабочей области вы сможете создавать отчеты Power BI непосредственно из Azure Synapse Analytics и из Power BI Desktop.

>[!Note]
> Прежде чем начать, настройте связанную службу для [рабочей области Power BI](../quickstart-power-bi.md) и скачайте [Power BI Desktop](/power-bi/service-create-the-new-workspaces).  

Чтобы подключить бессерверный пул SQL к рабочей области Power BI, сделайте следующее:

1.  Перейдите на вкладку **Наборы данных Power BI** и выберите **+ Создать набор данных**. По запросу скачайте файл с расширением .pbids из базы данных SQL Analytics, которую вы хотите использовать в качестве источника данных. 
   :::image type="content" source="../spark/media/apache-spark-power-bi/power-bi-datasets.png" alt-text="Наборы данных Power BI" border="true":::

2.  Откройте файл в Power BI Desktop, чтобы создать набор данных. После открытия файла подключите базу данных SQL Server с использованием **учетной записи Майкрософт**, выбрав пункт **Импорт**. 
   

## <a name="create-a-power-bi-report"></a>Создание отчета Power BI
1. Из Power BI Desktop вы теперь можете добавить в свой отчет диаграмму с **ключевыми факторами влияния**.
   
   1. Перетащите столбец со средними значениями *tipAmount* на ось **Анализ**.
   
   2. Перетащите **weekdayString**, столбцы со средними значениями **tripDistance** и средними значениями **tripTimeSecs** на ось **Объяснение по**. 
   
   :::image type="content" source="../spark/media/apache-spark-power-bi/power-bi-desktop.png" alt-text="Power BI Desktop" border="true":::

2. На главной вкладке Power BI Desktop выберите **Опубликовать** и **сохраните** изменения. Введите имя файла и сохраните этот отчет в *рабочей области NycTaxiTutorial*.
   
3. Вы также можете создать визуализации Power BI из своей рабочей области Azure Synapse Analytics. Для этого перейдите на вкладку **Разработка** в рабочей области Azure Synapse и откройте вкладку Power BI. Здесь вы можете выбрать свой отчет и создать дополнительные визуализации. 
   
   :::image type="content" source="../spark/media/apache-spark-power-bi/power-bi-synapse.png" alt-text="Рабочая область Azure Synapse Analytics" border="true":::

Дополнительные сведения о создании набора данных через бессерверный пул SQL и о подключении к Power BI см. в [этом руководстве](../../synapse-analytics/sql/tutorial-connect-power-bi-desktop.md).

## <a name="next-steps"></a>Дальнейшие действия
Чтобы узнать больше о возможностях визуализации данных в Azure Synapse Analytics, изучите следующие документы и руководства:
   - [Визуализация данных с помощью бессерверных пулов Apache Spark](../spark/apache-spark-data-visualization-tutorial.md)
   - [Общие сведения о визуализации данных с помощью пулов Apache Spark](../spark/apache-spark-data-visualization.md)