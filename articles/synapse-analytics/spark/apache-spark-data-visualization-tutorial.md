---
title: Визуализация данных с помощью Apache Spark
description: Создание разнообразных визуализаций данных с помощью Apache Spark и записных книжек Azure синапсе Analytics
services: synapse-analytics
author: midesa
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: machine-learning
ms.date: 10/20/2020
ms.author: midesa
ms.openlocfilehash: 56af49b6fa862c93822293056752182b534ac442
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98942277"
---
# <a name="analyze-data-with-apache-spark"></a>Анализ данных с помощью Apache Spark

В этом руководстве вы узнаете, как выполнять исследовательский анализ данных с помощью Azure Open DataSet и Apache Spark. Затем можно визуализировать результаты в записной книжке синапсе Studio в Azure синапсе Analytics.

В частности, мы проанализируем набор данных о [городе Нью-Йорка (Нью)](https://azure.microsoft.com/en-us/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/) . Данные доступны через открытые наборы данных Azure. Это подмножество набора данных содержит сведения о желтых поездках в такси: сведения о каждом путешествии, времени начала и окончания и расположениях, стоимости и других интересных атрибутах.
  
## <a name="before-you-begin"></a>Подготовка к работе
Создайте пул Apache Spark, выполнив инструкции из [этого руководства](../articles/../quickstart-create-apache-spark-pool-studio.md). 

## <a name="download-and-prepare-the-data"></a>Скачивание и подготовка данных
1. Создайте записную книжку, используя ядро PySpark. Инструкции см. в разделе [Создание записной книжки](../quickstart-apache-spark-notebook.md#create-a-notebook). 
   
   > [!Note]
   > Ядро PySpark позволяет не задавать контексты явным образом. Контекст Spark будет создан автоматически при выполнении первой ячейки кода.

2. В этом учебнике мы будем использовать несколько различных библиотек, которые помогут нам визуализировать набор данных. Чтобы выполнить этот анализ, импортируйте следующие библиотеки: 

   ```python
   import matplotlib.pyplot as plt
   import seaborn as sns
   import pandas as pd
   ```

3. Так как необработанные данные имеют формат Parquet, вы можете использовать контекст Spark, чтобы извлечь файл в память напрямую в виде кадра данных. Создайте DataFrame в Spark, получив данные с помощью API Открытых наборов данных. Здесь мы используем схему "кадры данных Spark" *для чтения* свойств, чтобы определить типы данных и схему.

   ```python
   from azureml.opendatasets import NycTlcYellow
   from datetime import datetime
   from dateutil import parser

   end_date = parser.parse('2018-06-06')
   start_date = parser.parse('2018-05-01')
   nyc_tlc = NycTlcYellow(start_date=start_date, end_date=end_date)
   df = nyc_tlc.to_spark_dataframe()
   ```

4. После считывания данных необходимо выполнить начальную фильтрацию для очистки набора данных. Мы можем удалить ненужные столбцы и добавить столбцы, которые извлекают важные сведения. Кроме того, мы отфильтрим аномалии в наборе данных.

   ```python
   # Filter the dataset 
   from pyspark.sql.functions import *

   filtered_df = df.select('vendorID', 'passengerCount', 'tripDistance','paymentType', 'fareAmount', 'tipAmount'\
                                   , date_format('tpepPickupDateTime', 'hh').alias('hour_of_day')\
                                   , dayofweek('tpepPickupDateTime').alias('day_of_week')\
                                   , dayofmonth(col('tpepPickupDateTime')).alias('day_of_month'))\
                               .filter((df.passengerCount > 0)\
                                   & (df.tipAmount >= 0)\
                                   & (df.fareAmount >= 1) & (df.fareAmount <= 250)\
                                   & (df.tripDistance > 0) & (df.tripDistance <= 200))

   filtered_df.createOrReplaceTempView("taxi_dataset")
   ```

## <a name="analyze-data"></a>Анализ данных
В качестве аналитика данных доступен широкий спектр средств, позволяющих извлекать ценные сведения из данных. В этой части руководства мы рассмотрим несколько полезных средств, доступных в записных книжках Azure синапсе Analytics. В этом анализе мы хотим понять факторы, которые выдают более высокие советы по использованию выбранного периода.

### <a name="apache-spark-sql-magic"></a>Apache Spark SQL Magic 
Во первых, мы выполним анализ исследовательских данных, Apache Spark команды SQL и Magic с помощью записной книжки Azure синапсе. После выполнения запроса мы будем визуализировать результаты с помощью встроенной ```chart options``` возможности. 

1. В записной книжке создайте новую ячейку и скопируйте следующий код. С помощью этого запроса мы хотим понять, как средние значения TIP были изменены в течение выбранного периода. Этот запрос также поможет нам определить другие полезные сведения, включая минимальное и максимальное количество советов в день и среднюю FARE сумму.
   
   ```sql
   %%sql
   SELECT 
       day_of_month
       , MIN(tipAmount) AS minTipAmount
       , MAX(tipAmount) AS maxTipAmount
       , AVG(tipAmount) AS avgTipAmount
       , AVG(fareAmount) as fareAmount
   FROM taxi_dataset 
   GROUP BY day_of_month
   ORDER BY day_of_month ASC
   ```

2. После завершения выполнения запроса можно визуализировать результаты, переключившись на представление диаграммы. В этом примере создается график путем указания поля в ```day_of_month``` качестве ключа и ```avgTipAmount``` значения. После выбора параметров нажмите кнопку **Применить** , чтобы обновить диаграмму. 
   
## <a name="visualize-data"></a>Визуализируйте данные
В дополнение к встроенным параметрам создания диаграмм для записных книжек вы можете использовать популярные библиотеки с открытым исходным кодом, чтобы создавать собственные визуализации. В следующих примерах мы будем использовать Сеаборн и Matplotlib. Они часто используются библиотеками Python для визуализации данных. 

> [!Note]
> По умолчанию каждый пул Apache Spark в Azure синапсе Analytics содержит набор часто используемых и стандартных библиотек. Полный список библиотек можно просмотреть в документации по [среде выполнения Azure синапсе](../spark/apache-spark-version-support.md) . Кроме того, чтобы сделать код стороннего разработчика или локального кода доступным для приложений, можно [установить библиотеку](../spark/apache-spark-azure-portal-add-libraries.md) в один из пулов Spark.

1. Чтобы упростить процесс разработки и снизить расходы, мы будем использовать этот набор данных. Мы будем использовать встроенную возможность выборки Apache Spark. Кроме того, для Сеаборн и Matplotlib требуется кадр данных Pandas или массив NumPy. Чтобы получить кадр данных Pandas, используйте команду, ```toPandas()``` чтобы преобразовать кадр данных.

   ```python
   # To make development easier, faster, and less expensive, downsample for now
   sampled_taxi_df = filtered_df.sample(True, 0.001, seed=1234)

   # The charting package needs a Pandas DataFrame or NumPy array to do the conversion
   sampled_taxi_pd_df = sampled_taxi_df.toPandas()
   ```

1. Мы хотим разобраться с распределением советов в нашем наборе данных. Мы будем использовать Matplotlib для создания гистограммы, показывающей распределение суммы и количества TIP. В зависимости от распределения мы видим, что советы переносятся в сторону сумм, меньших или равных $10.

   ```python
   # Look at a histogram of tips by count by using Matplotlib

   ax1 = sampled_taxi_pd_df['tipAmount'].plot(kind='hist', bins=25, facecolor='lightblue')
   ax1.set_title('Tip amount distribution')
   ax1.set_xlabel('Tip Amount ($)')
   ax1.set_ylabel('Counts')
   plt.suptitle('')
   plt.show()
   ```

   ![Гистограмма советов.](./media/apache-spark-machine-learning-mllib-notebook/histogram.png)

1. Далее мы хотим понять связь между советами по данному пути и днем недели. Используйте Сеаборн, чтобы создать блочную диаграмму, которая суммирует тенденции за каждый день недели. 

   ```python
   # View the distribution of tips by day of week using Seaborn
   ax = sns.boxplot(x="day_of_week", y="tipAmount",data=sampled_taxi_pd_df, showfliers = False)
   ax.set_title('Tip amount distribution per day')
   ax.set_xlabel('Day of Week')
   ax.set_ylabel('Tip Amount ($)')
   plt.show()

   ```
   ![График, отображающий распределение советов в день.](./media/apache-spark-data-viz/data-analyst-tutorial-per-day.png)

4. Другой гипотезой может быть то, что существует положительная связь между числом пассажиров и общей суммой советов по такси. Чтобы проверить эту связь, выполните следующий код, чтобы создать блочную диаграмму, которая иллюстрирует распределение советов для каждого пассажиров числа. 
   
   ```python
   # How many passengers tipped by various amounts 
   ax2 = sampled_taxi_pd_df.boxplot(column=['tipAmount'], by=['passengerCount'])
   ax2.set_title('Tip amount by Passenger count')
   ax2.set_xlabel('Passenger count')
   ax2.set_ylabel('Tip Amount ($)')
   ax2.set_ylim(0,30)
   plt.suptitle('')
   plt.show()
   ```
   ![Граф, на котором показана прямоугольная усами графика.](./media/apache-spark-machine-learning-mllib-notebook/box-whisker-plot.png)

1. Наконец, мы хотим понять отношение между суммой FARE и суммой TIP. На основе результатов мы видим, что существует несколько наблюдений, когда люди не подсказок. Однако мы также видим положительные отношения между общими суммами FARE и TIP.
   
   ```python
   # Look at the relationship between fare and tip amounts

   ax = sampled_taxi_pd_df.plot(kind='scatter', x= 'fareAmount', y = 'tipAmount', c='blue', alpha = 0.10, s=2.5*(sampled_taxi_pd_df['passengerCount']))
   ax.set_title('Tip amount by Fare amount')
   ax.set_xlabel('Fare Amount ($)')
   ax.set_ylabel('Tip Amount ($)')
   plt.axis([-2, 80, -2, 20])
   plt.suptitle('')
   plt.show()
   ```
   ![Точечная диаграмма с суммой советов.](./media/apache-spark-machine-learning-mllib-notebook/scatter.png)

## <a name="shut-down-the-spark-instance"></a>Завершение работы экземпляра Spark

После завершения работы приложения завершите работу записной книжки, чтобы освободить ресурсы. Либо закройте вкладку, либо выберите пункт **завершить сеанс** на панели состояния в нижней части записной книжки.

## <a name="see-also"></a>См. также раздел

- [Обзор: Apache Spark в Azure Synapse Analytics](apache-spark-overview.md)
- [Создание модели машинного обучения с помощью Apache SparkML](../spark/apache-spark-machine-learning-mllib-notebook.md)

## <a name="next-steps"></a>Дальнейшие действия

- [Azure Synapse Analytics](../index.yml)
- [Официальная документация по Apache Spark](https://spark.apache.org/docs/latest/)