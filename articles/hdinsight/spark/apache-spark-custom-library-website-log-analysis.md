---
title: Анализ журналов веб-сайтов с помощью библиотек Python в Spark в Azure
description: Эта записная книжка показывает, как анализировать данные журналов с помощью пользовательской библиотеки с кластером Spark в Azure HDInsight.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 12/27/2019
ms.openlocfilehash: d99bf914d3ef746fe328d2447426565fc15e52b4
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104866324"
---
# <a name="analyze-website-logs-using-a-custom-python-library-with-apache-spark-cluster-on-hdinsight"></a>Анализ журналов веб-сайтов с помощью пользовательской библиотеки Python и кластера Apache Spark в HDInsight

Данная записная книжка показывает, как анализировать данные журналов с помощью настраиваемой библиотеки с кластером Apache Spark в HDInsight. В качестве пользовательской библиотеки используется библиотека Python с именем **iislogparser.py**.

## <a name="prerequisites"></a>Предварительные требования

Кластер Apache Spark в HDInsight. Инструкции см. в статье [Начало работы. Создание кластера Apache Spark в HDInsight на платформе Linux и выполнение интерактивных запросов с помощью SQL Spark](apache-spark-jupyter-spark-sql.md).

## <a name="save-raw-data-as-an-rdd"></a>Сохранение необработанных данных в формате RDD

В этом разделе мы используем записную книжку [Jupyter](https://jupyter.org) , связанную с кластером Apache Spark в HDInsight, для выполнения заданий, обрабатывающих необработанные образцы данных, и сохранения их в виде таблицы Hive. В качестве демонстрационных данных выступает CSV-файл (hvac.csv), доступный на всех кластерах по умолчанию.

После сохранения данных в виде таблицы Apache Hive в следующем разделе мы будем подключаться к таблице Hive с помощью средств бизнес-аналитики, таких как Power BI и Tableau.

1. В веб-браузере перейдите на страницу `https://CLUSTERNAME.azurehdinsight.net/jupyter`, где `CLUSTERNAME` — это имя вашего кластера.

1. Создайте новую записную книжку. Выберите **создать**, а затем **PySpark**.

    :::image type="content" source="./media/apache-spark-custom-library-website-log-analysis/hdinsight-create-jupyter-notebook.png " alt-text="Создание нового Jupyter Notebook Apache" border="true"::: Записная книжка "border =" true ":::

1. Будет создана и открыта записная книжка с именем Untitled.pynb. Выберите имя записной книжки вверху и введите понятное имя.

    :::image type="content" source="./media/apache-spark-custom-library-website-log-analysis/hdinsight-name-jupyter-notebook.png " alt-text="Укажите имя записной книжки" border="true":::тебук "border =" true ":::

1. Так как вы создали записную книжку с помощью ядра PySpark, вам не нужно явно создавать контексты. Контексты Spark и Hive будут созданы автоматически при выполнении первой ячейки кода. Можно начать с импорта различных типов, необходимых для этого сценария. Вставьте следующий фрагмент кода в пустую ячейку и нажмите клавиши **SHIFT + ВВОД**.

    ```pyspark
    from pyspark.sql import Row
    from pyspark.sql.types import *
    ```

1. Создайте RDD, используя пример данных журнала, уже доступных в кластере. Доступ к данным в учетной записи хранения по умолчанию, связанной с кластером, можно получить по адресу `\HdiSamples\HdiSamples\WebsiteLogSampleData\SampleLog\909f2b.log` . Выполните следующий код.

    ```pyspark
    logs = sc.textFile('wasbs:///HdiSamples/HdiSamples/WebsiteLogSampleData/SampleLog/909f2b.log')
    ```

1. Извлеките пример набора журналов и убедитесь в том, что описанный выше шаг успешно выполнен.

    ```pyspark
    logs.take(5)
    ```

    Вы должны увидеть результат, аналогичный приведенному ниже:

    ```output
    [u'#Software: Microsoft Internet Information Services 8.0',
    u'#Fields: date time s-sitename cs-method cs-uri-stem cs-uri-query s-port cs-username c-ip cs(User-Agent) cs(Cookie) cs(Referer) cs-host sc-status sc-substatus sc-win32-status sc-bytes cs-bytes time-taken',
    u'2014-01-01 02:01:09 SAMPLEWEBSITE GET /blogposts/mvc4/step2.png X-ARR-LOG-ID=2ec4b8ad-3cf0-4442-93ab-837317ece6a1 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 53175 871 46',
    u'2014-01-01 02:01:09 SAMPLEWEBSITE GET /blogposts/mvc4/step3.png X-ARR-LOG-ID=9eace870-2f49-4efd-b204-0d170da46b4a 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 51237 871 32',
    u'2014-01-01 02:01:09 SAMPLEWEBSITE GET /blogposts/mvc4/step4.png X-ARR-LOG-ID=4bea5b3d-8ac9-46c9-9b8c-ec3e9500cbea 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 72177 871 47']
    ```

## <a name="analyze-log-data-using-a-custom-python-library"></a>Анализ данных журнала с помощью пользовательской библиотеки Python

1. В приведенном выше примере выходных данных первые несколько строк содержат сведения о заголовке, а все последующие строки соответствуют схеме, описанной в этом заголовке. Анализ таких журналов может быть сложным, поэтому мы используем настраиваемую библиотеку Python (**iislogparser.py**), которая делает эту задачу намного проще. По умолчанию эта библиотека входит в кластер Spark в HDInsight по адресу `/HdiSamples/HdiSamples/WebsiteLogSampleData/iislogparser.py` .

    Однако эта библиотека отсутствует в, `PYTHONPATH` поэтому мы не можем использовать ее с помощью оператора import, например `import iislogparser` . Чтобы использовать эту библиотеку, необходимо распространить ее на все рабочие узлы. Выполните следующий фрагмент кода.

    ```pyspark
    sc.addPyFile('wasbs:///HdiSamples/HdiSamples/WebsiteLogSampleData/iislogparser.py')
    ```

1. `iislogparser` предоставляет функцию `parse_log_line`, которая возвращает `None`, если строка журнала является строкой заголовка, или экземпляр класса `LogLine` при обнаружении строки журнала. Класс `LogLine` позволяет извлечь только строки журнала из RDD:

    ```pyspark
    def parse_line(l):
        import iislogparser
        return iislogparser.parse_log_line(l)
    logLines = logs.map(parse_line).filter(lambda p: p is not None).cache()
    ```

1. Выведите на экран несколько извлеченных строк журнала и убедитесь в том, что это действие выполнено успешно.

    ```pyspark
    logLines.take(2)
    ```

   Выходные данные должны быть похожи на следующий текст:

    ```output
    [2014-01-01 02:01:09 SAMPLEWEBSITE GET /blogposts/mvc4/step2.png X-ARR-LOG-ID=2ec4b8ad-3cf0-4442-93ab-837317ece6a1 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 53175 871 46,
    2014-01-01 02:01:09 SAMPLEWEBSITE GET /blogposts/mvc4/step3.png X-ARR-LOG-ID=9eace870-2f49-4efd-b204-0d170da46b4a 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 51237 871 32]
    ```

1. В свою очередь, класс `LogLine` включает несколько полезных методов, например метод `is_error()`, который возвращается, если запись журнала содержит код ошибки. Этот класс используется для вычислить количество ошибок в извлеченных строках журнала, а затем регистрировать все ошибки в другом файле.

    ```pyspark
    errors = logLines.filter(lambda p: p.is_error())
    numLines = logLines.count()
    numErrors = errors.count()
    print 'There are', numErrors, 'errors and', numLines, 'log entries'
    errors.map(lambda p: str(p)).saveAsTextFile('wasbs:///HdiSamples/HdiSamples/WebsiteLogSampleData/SampleLog/909f2b-2.log')
    ```

    Выходные данные должны иметь состояние `There are 30 errors and 646 log entries` .

1. Кроме того, для визуализации данных вы можете использовать **Matplotlib** . Например, если вы хотите установить причину длительного выполнения некоторых запросов, найдите файлы с наибольшим средним временем обработки. Код в представленном ниже фрагменте выдает первые 25 ресурсов с максимальным временем обработки запросов.

    ```pyspark
    def avgTimeTakenByKey(rdd):
        return rdd.combineByKey(lambda line: (line.time_taken, 1),
                                lambda x, line: (x[0] + line.time_taken, x[1] + 1),
                                lambda x, y: (x[0] + y[0], x[1] + y[1]))\
                    .map(lambda x: (x[0], float(x[1][0]) / float(x[1][1])))

    avgTimeTakenByKey(logLines.map(lambda p: (p.cs_uri_stem, p))).top(25, lambda x: x[1])
    ```

   Вы должны увидеть выходные данные, как в следующем тексте:

    ```output
    [(u'/blogposts/mvc4/step13.png', 197.5),
    (u'/blogposts/mvc2/step10.jpg', 179.5),
    (u'/blogposts/extractusercontrol/step5.png', 170.0),
    (u'/blogposts/mvc4/step8.png', 159.0),
    (u'/blogposts/mvcrouting/step22.jpg', 155.0),
    (u'/blogposts/mvcrouting/step3.jpg', 152.0),
    (u'/blogposts/linqsproc1/step16.jpg', 138.75),
    (u'/blogposts/linqsproc1/step26.jpg', 137.33333333333334),
    (u'/blogposts/vs2008javascript/step10.jpg', 127.0),
    (u'/blogposts/nested/step2.jpg', 126.0),
    (u'/blogposts/adminpack/step1.png', 124.0),
    (u'/BlogPosts/datalistpaging/step2.png', 118.0),
    (u'/blogposts/mvc4/step35.png', 117.0),
    (u'/blogposts/mvcrouting/step2.jpg', 116.5),
    (u'/blogposts/aboutme/basketball.jpg', 109.0),
    (u'/blogposts/anonymoustypes/step11.jpg', 109.0),
    (u'/blogposts/mvc4/step12.png', 106.0),
    (u'/blogposts/linq8/step0.jpg', 105.5),
    (u'/blogposts/mvc2/step18.jpg', 104.0),
    (u'/blogposts/mvc2/step11.jpg', 104.0),
    (u'/blogposts/mvcrouting/step1.jpg', 104.0),
    (u'/blogposts/extractusercontrol/step1.png', 103.0),
    (u'/blogposts/sqlvideos/sqlvideos.jpg', 102.0),
    (u'/blogposts/mvcrouting/step21.jpg', 101.0),
    (u'/blogposts/mvc4/step1.png', 98.0)]
    ```

1. Кроме того, эти сведения вы можете представить в виде графика. Чтобы создать график, для начала создадим временную таблицу **AverageTime**. Таблица группирует журналы по времени и показывает все необычные пики задержек за определенный период времени.

    ```pyspark
    avgTimeTakenByMinute = avgTimeTakenByKey(logLines.map(lambda p: (p.datetime.minute, p))).sortByKey()
    schema = StructType([StructField('Minutes', IntegerType(), True),
                        StructField('Time', FloatType(), True)])

    avgTimeTakenByMinuteDF = sqlContext.createDataFrame(avgTimeTakenByMinute, schema)
    avgTimeTakenByMinuteDF.registerTempTable('AverageTime')
    ```

1. После этого можно выполнить следующий запрос SQL, чтобы получить все записи таблицы **AverageTime** .

    ```pyspark
    %%sql -o averagetime
    SELECT * FROM AverageTime
    ```

   Волшебное слово `%%sql`, за которым следует `-o averagetime`, гарантирует, что вывод запроса сохраняется локально на сервере Jupyter (обычно это головной узел кластера). Выходные данные сохраняются в кадре данных [Pandas](https://pandas.pydata.org/) с именем **averagetime**.

   Вы должны увидеть выходные данные, как на следующем рисунке:

   выходные данные запроса SQL :::image type="content" source="./media/apache-spark-custom-library-website-log-analysis/hdinsight-jupyter-sql-qyery-output.png " alt-text="hdinsight jupyter" border="true":::ИТЕР SQL "border =" true ":::

   Дополнительные сведения о магической команде `%%sql` см. в разделе [Параметры, поддерживаемые волшебной командой %%sql](apache-spark-jupyter-notebook-kernels.md#parameters-supported-with-the-sql-magic).

1. Теперь можно создать график с помощью Matplotlib, библиотеки, используемой для визуализации данных. Так как диаграмма должна создаваться из локально сохраненного кадра данных **averagetime**, фрагмент кода должен начинаться с волшебного слова `%%local`. Это гарантирует, что код будет выполняться локально на сервере Jupyter.

    ```pyspark
    %%local
    %matplotlib inline
    import matplotlib.pyplot as plt

    plt.plot(averagetime['Minutes'], averagetime['Time'], marker='o', linestyle='--')
    plt.xlabel('Time (min)')
    plt.ylabel('Average time taken for request (ms)')
    ```

   Вы должны увидеть выходные данные, как на следующем рисунке:

   :::image type="content" source="./media/apache-spark-custom-library-website-log-analysis/hdinsight-apache-spark-web-log-analysis-plot.png " alt-text="Анализ журнала веб-журнала Apache Spark" border="true":::с отчетом EB диаграмма анализа журналов "border =" true ":::

1. Завершив работу с приложением, следует закрыть записную книжку, чтобы освободить ресурсы. Для этого в меню **File** (Файл) записной книжки выберите пункт **Close and Halt** (Закрыть и остановить). Это действие приведет к завершению работы и закрытию записной книжки.

## <a name="next-steps"></a>Дальнейшие действия

Изучите следующие статьи:

* [Обзор: Spark в Azure HDInsight](apache-spark-overview.md)
* [Использование внешних пакетов с записными книжками Jupyter](apache-spark-jupyter-notebook-use-external-packages.md)
* [Установка записной книжки Jupyter на компьютере и ее подключение к кластеру Apache Spark в Azure HDInsight (предварительная версия)](apache-spark-jupyter-notebook-install-locally.md)
