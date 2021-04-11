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
ms.date: 03/24/2021
ms.openlocfilehash: 0becbbdb68f75072e10a51f5a2eae95291b9ed77
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105108338"
---
# <a name="analyze-with-apache-spark"></a>Анализ с помощью Apache Spark

В этом учебнике описываются основные шаги по загрузке и анализу данных с помощью Apache Spark для Azure Synapse.

## <a name="create-a-serverless-apache-spark-pool"></a>Создание бессерверного пула Apache Spark

1. В окне Synapse Studio слева выберите **Управление** > **Пулы Apache Spark**.
1. Выберите **Создать**. 
1. В поле **Имя пула Apache Spark** введите **Spark1**.
1. В поле **Размер узла** введите **Небольшой**.
1. В поле **Число узлов** задайте не меньше и не больше 3 узлов.
1. Выберите команду **Просмотреть и создать** > **Создать**. Ваш пул Apache Spark будет готов через несколько секунд.

## <a name="understanding-serverless-apache-spark-pools"></a>Общие сведения о бессерверных пулах Apache Spark

Бессерверный пул Spark позволяет определить для пользователя способ работы со Spark. Когда вы начинаете использовать пул, при необходимости создается сеанс Spark. Пул управляет тем, сколько ресурсов Spark будет использоваться в этом сеансе и как долго будет длиться сеанс перед автоматической приостановкой. Вы платите за ресурсы Spark, используемые в этом сеансе, а не в самом пуле. Таким образом, пул Spark позволяет работать со Spark без необходимости управлять кластерами. Это похоже на то, как работает бессерверный пул SQL.

## <a name="analyze-nyc-taxi-data-in-blob-storage-using-spark"></a>Анализ данных такси Нью-Йорка в Хранилище BLOB-объектов с помощью Spark

1. В Synapse Studio перейдите в центр **Разработка**.
2. Создайте новую записную книжку с языком по умолчанию **PySpark (Python)** .
3. Создайте новую ячейку кода и вставьте в нее следующий код.
    ```py
    %%pyspark
    from azureml.opendatasets import NycTlcYellow

    data = NycTlcYellow()
    df = data.to_spark_dataframe()
    # Display 10 rows
    display(df.limit(10))
    ```
1. В записной книжке в меню **Присоединить к** выберите созданный ранее бессерверный пул Spark **Spark1**.
1. В ячейке выберите элемент **Выполнить**. При необходимости Synapse запустит новый сеанс Spark для выполнения этой ячейки. Если потребуется новый сеанс Spark, он будет создан в течение двух секунд. 
1. Если вы просто хотите просмотреть схему кадра данных, выполните ячейку со следующим кодом:

    ```py
    df.printSchema()
    ```

## <a name="load-the-nyc-taxi-data-into-the-spark-nyctaxi-database"></a>Загрузка данных нью-йоркского такси в базу данных Spark "nyctaxi"

Данные доступны через кадр данных с именем **data**. Загрузите их в базу данных Spark с именем **nyctaxi**.

1. В записной книжке введите следующий код:

    ```py
    spark.sql("CREATE DATABASE IF NOT EXISTS nyctaxi")
    df.write.mode("overwrite").saveAsTable("nyctaxi.trip")
    ```
## <a name="analyze-the-nyc-taxi-data-using-spark-and-notebooks"></a>Анализ данных нью-йоркского такси с помощью Spark и записных книжек

1. Вернитесь к своей записной книжке.
1. Создайте новую ячейку кода и введите следующий код. 

   ```py
   %%pyspark
   df = spark.sql("SELECT * FROM nyctaxi.trip") 
   display(df)
   ```

1. Запустите ячейку, чтобы показать данные о такси Нью-Йорка, которые мы загрузили в базу данных Spark **nyctaxi**.
1. Создайте новую ячейку кода и введите следующий код. Мы будем анализировать эти данные и сохранять результаты в таблице **nyctaxi.passengercountstats**.

   ```py
   %%pyspark
   df = spark.sql("""
      SELECT PassengerCount,
          SUM(TripDistance) as SumTripDistance,
          AVG(TripDistance) as AvgTripDistance
      FROM nyctaxi.trip
      WHERE TripDistance > 0 AND PassengerCount > 0
      GROUP BY PassengerCount
      ORDER BY PassengerCount
   """) 
   display(df)
   df.write.saveAsTable("nyctaxi.passengercountstats")
   ```

1. В результатах ячейки выберите пункт **Диаграмма**, чтобы просмотреть визуализацию данных.


## <a name="next-steps"></a>Следующие шаги

> [!div class="nextstepaction"]
> [Анализ данных с помощью выделенных пулов SQL](get-started-analyze-sql-pool.md)
