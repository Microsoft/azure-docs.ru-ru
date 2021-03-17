---
title: Краткое руководство. Начало анализа с помощью Spark
description: Из этого руководства вы узнаете, как анализировать данные с помощью Apache Spark.
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.subservice: spark
ms.topic: tutorial
ms.date: 12/31/2020
ms.openlocfilehash: 6b3c1ac2ea3625a768e16a3465230a5386c98ddc
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102423719"
---
# <a name="analyze-with-apache-spark"></a>Анализ с помощью Apache Spark

В этом учебнике описываются основные шаги по загрузке и анализу данных с помощью Apache Spark для Azure Synapse.

## <a name="analyze-nyc-taxi-data-in-blob-storage-using-spark"></a>Анализ данных такси Нью-Йорка в Хранилище BLOB-объектов с помощью Spark

1. В центре **Данные** нажмите кнопку **+** , чтобы **добавить новый ресурс**, и щелкните **Обзор коллекции**. 
1. Найдите и выберите **NYC Taxi & Limousine Commission - yellow taxi trip records**. 
1. В нижней части страницы щелкните **Продолжить** и выберите **Добавить набор данных**. 
1. Через некоторое время в центре **Данные** в разделе **Связанный** щелкните правой кнопкой мыши **Хранилище BLOB-объектов Azure >> Примеры наборов данных >> nyc_tlc_yellow** и выберите элемент **Новая записная книжка**, а затем — **Загрузить в кадр данных**.
1. Будет создана записная книжка со следующим кодом:
    ```

    from azureml.opendatasets import NycTlcYellow

    data = NycTlcYellow()
    df = data.to_spark_dataframe()
    # Display 10 rows
    display(df.limit(10))
    ```
1. В записной книжке в меню **Присоединить к** выберите созданный ранее бессерверный пул Spark **Spark1**.
1. В ячейке выберите элемент **Выполнить**. При необходимости Synapse запустит новый сеанс Spark для выполнения этой ячейки. Если потребуется новый сеанс Spark, он будет создан в течение двух секунд. 
1. Если вы просто хотите просмотреть схему кадра данных, выполните ячейку со следующим кодом:
    ```

    df.printSchema()
    ```

## <a name="load-the-nyc-taxi-data-into-the-spark-nyctaxi-database"></a>Загрузка данных нью-йоркского такси в базу данных Spark "nyctaxi"

Данные доступны в таблице в пуле **SQLPOOL1**. Загрузите их в базу данных Spark с именем **nyctaxi**.

1. В Synapse Studio перейдите в центр **Разработка**.
1. Выберите **+**  > **Записная книжка**.
1. В верхней части записной книжки задайте для параметра **Присоединить к** значение **Spark1**.
1. Перейдите к первой ячейке кода новой записной книжки и введите следующий код:


    ```scala
    %%spark
    spark.sql("CREATE DATABASE IF NOT EXISTS nyctaxi")
    val df = spark.read.sqlanalytics("SQLPOOL1.dbo.Trip") 
    df.write.mode("overwrite").saveAsTable("nyctaxi.trip")
    ```


1. Выполните скрипт. Это может занять 2–3 минуты.
1. В центре **Данные** на вкладке **Рабочая область** щелкните правой кнопкой мыши элемент **Базы данных** и выберите команду **Обновить**. Теперь в списке должна отобразиться база данных **nyctaxi (Spark)** .


## <a name="analyze-the-nyc-taxi-data-using-spark-and-notebooks"></a>Анализ данных нью-йоркского такси с помощью Spark и записных книжек

1. Вернитесь к своей записной книжке.
1. Создайте новую ячейку кода и введите следующий код. 


   ```py
   %%pyspark
   df = spark.sql("SELECT * FROM nyctaxi.trip") 
   display(df)
   ```

1. Запустите ячейку, чтобы показать данные о такси Нью-Йорка, которые мы загрузили в базу данных Spark **nyctaxi**.
1. Создайте новую ячейку кода и введите следующий код. Затем запустите ячейку, чтобы провести тот же анализ, который мы выполняли ранее с помощью выделенного пула SQL **SQLPOOL1**. Этот код сохраняет результаты анализа в таблицу с именем **nyctaxi.passengercountstats** и выводит эти результаты.


   ```py
   %%pyspark
   df = spark.sql("""
      SELECT PassengerCount,
          SUM(TripDistanceMiles) as SumTripDistance,
          AVG(TripDistanceMiles) as AvgTripDistance
      FROM nyctaxi.trip
      WHERE TripDistanceMiles > 0 AND PassengerCount > 0
      GROUP BY PassengerCount
      ORDER BY PassengerCount
   """) 
   display(df)
   df.write.saveAsTable("nyctaxi.passengercountstats")
   ```

1. В результатах ячейки выберите пункт **Диаграмма**, чтобы просмотреть визуализацию данных.

## <a name="load-data-from-a-spark-table-into-a-dedicated-sql-pool-table"></a>Загрузка данных из таблицы Spark в таблицу выделенного пула SQL

Ранее мы скопировали данные из таблицы выделенного пула SQL **SQLPOOL1.dbo.Trip** в таблицу Spark **nyctaxi.trip**. Затем вы агрегировали данные в таблицу Spark **nyctaxi.passengercountstats**. Теперь скопируйте данные из **nyctaxi.passengercountstats** в таблицу выделенного пула SQL с именем **SQLPOOL1.dbo.PassengerCountStats**.

1. Создайте новую ячейку кода и введите следующий код. Выполните ячейку в записной книжке. Сводная таблица Spark копируется обратно в таблицу выделенного пула SQL.

```scala
%%spark
val df = spark.sql("SELECT * FROM nyctaxi.passengercountstats")
df.write.sqlanalytics("SQLPOOL1.dbo.PassengerCountStats", Constants.INTERNAL )
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Анализ данных с помощью бессерверного пула SQL](get-started-analyze-sql-on-demand.md)
