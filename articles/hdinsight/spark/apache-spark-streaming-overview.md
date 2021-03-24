---
title: Потоковая передача Spark в Azure HDInsight
description: Использование Apache Spark потоковых приложений в кластерах HDInsight Spark.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 04/23/2020
ms.openlocfilehash: e8a9f771827b870f493d6b0d7590feee7fc52b20
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104870251"
---
# <a name="overview-of-apache-spark-streaming"></a>Общие сведения о потоковой передаче Apache Spark

[Apache Spark](https://spark.apache.org/) Потоковая передача обеспечивает обработку потока данных в кластерах HDInsight Spark. С гарантией того, что любое событие ввода обрабатывается ровно один раз, даже если происходит сбой узла. Поток Spark — это длительно выполняемое задание, которое получает входные данные из широкого спектра источников, включая концентраторы событий Azure. Кроме того, центр Интернета вещей Azure, Apache Kafka, Apache Flume, Twitter, `ZeroMQ` необработанные сокеты TCP или от мониторинга Apache Hadoop YARN FileSystem. В отличие от отдельного процесса, управляемого событиями, потоковая передача данных из потока Spark выполняется в окнах времени. Например, 2-секундный срез, а затем преобразует каждый пакет данных с помощью операций Map, reduce, Join и Extract. Затем поток Spark записывает преобразованные данные в файловые системы, базы данных, панели мониторинга и консоль.

:::image type="content" source="./media/apache-spark-streaming-overview/hdinsight-spark-streaming.png" alt-text="Потоковая обработка с помощью HDInsight и потоковой передачи Spark" border="false":::

Приложения потоковой передачи Spark должны ожидать доли секунды, чтобы получить все `micro-batch` события перед отправкой этого пакета для обработки. В свою очередь, приложение на основе событий обрабатывает каждое событие сразу же. Задержка при потоковой передаче Spark, как правило, не превышает несколько секунд. Преимущества подхода с использованием микропакетов заключаются в более эффективной обработке данных и более простых статистических вычислениях.

## <a name="introducing-the-dstream"></a>Общие сведения о DStream

Потоковая передача Spark представляет непрерывный поток входящих данных с использованием *дискретизированного потока* под названием DStream. DStream можно создать из источников входных данных, таких как концентраторы событий или Kafka. Или путем применения преобразований для другого DStream.

Поток DStream обеспечивает уровень абстракции поверх необработанных данных событий.

Начните с одиночного события, например с чтения температуры с подключенного термостата. Когда это событие поступает в приложение потоковой передачи Spark, событие сохраняется в надежном виде, где оно реплицируется на нескольких узлах. Эта отказоустойчивость гарантирует, что сбой какого-либо отдельного узла не приведет к утрате события. В ядре Spark используется структура данных, которая распределяет данные между несколькими узлами в кластере. Каждый узел обычно хранит собственные данные в памяти для лучшей производительности. Эта структура данных называется *устойчивым распределенным набором данных* (RDD).

Каждый набор RDD содержит события, собранные за определяемый пользователем период времени, называемый *интервалом пакетной обработки*. По окончании каждого интервала пакетов создается новый набор RDD, содержащий все данные из этого интервала. Непрерывный набор RDD собираются в DStream. Например, если интервал пакетов составляет одну секунду, ваш поток DStream каждую секунду выдает пакет, содержащий один набор RDD со всеми данными, полученными на протяжении этой секунды. При обработке потока DStream событие температуры появляется в одном из этих пакетов. Приложение потоковой передачи Spark обрабатывает пакеты, содержащие события, и в конечном счете работает с данными, хранящимися в каждом наборе RDD.

:::image type="content" source="./media/apache-spark-streaming-overview/hdinsight-spark-streaming-example.png" alt-text="Пример потока DStream с событиями температуры" border="false":::

## <a name="structure-of-a-spark-streaming-application"></a>Структура приложения потоковой передачи Spark

Приложение потоковой передачи Spark — это длительно выполняемое приложение, которое получает данные из источников приема. Применяет преобразования для обработки данных, а затем передает их в одно или несколько назначений. Структура приложения потоковой передачи Spark состоит из статической и динамической частей. Статическая часть определяет, откуда берутся данные, какая обработка данных выполняется. И где должны идти результаты. Динамическая часть запускает приложение на неопределенный срок, ожидая сигнал об остановке.

Например, следующее простое приложение получает строку текста через TCP-сокет и подсчитывает количество появлений каждого слова.

### <a name="define-the-application"></a>Определение приложения

Определение логики приложения состоит из четырех шагов.

1. Создание объекта StreamingContext.
2. Создание потока DStream из StreamingContext.
3. Применение преобразований к потоку DStream.
4. Вывод результатов.

Это определение является статическим, обработка данных начинается только после запуска приложения.

#### <a name="create-a-streamingcontext"></a>Создание объекта StreamingContext

Создайте объект StreamingContext из объекта SparkContext, указывающего на ваш кластер. При создании объекта StreamingContext необходимо указать размер пакета в секундах, например:  

```
import org.apache.spark._
import org.apache.spark.streaming._

val ssc = new StreamingContext(sc, Seconds(1))
```

#### <a name="create-a-dstream"></a>Создание потока DStream

С помощью экземпляра StreamingContext создайте входной поток DStream для источника входных данных. В этом случае приложение наблюдает за внешним видом новых файлов в подключенном хранилище по умолчанию.

```
val lines = ssc.textFileStream("/uploads/Test/")
```

#### <a name="apply-transformations"></a>Применение преобразований

Обработка реализовывается путем применения преобразований к потоку DStream. Это приложение получает одну строку текста за раз из файла, разбивает каждую строку на слова. А затем использует шаблон Map-reduce для подсчета числа появлений каждого слова.

```
val words = lines.flatMap(_.split(" "))
val pairs = words.map(word => (word, 1))
val wordCounts = pairs.reduceByKey(_ + _)
```

#### <a name="output-results"></a>Вывод результатов

Отправьте результаты преобразования в системы назначения, применив операции вывода. В этом случае результат каждого выполнения путем вычисления выводится в выходных данных консоли.

```
wordCounts.print()
```

### <a name="run-the-application"></a>Выполнение приложения

Запустите приложение потоковой передачи и продолжайте его работу, пока не будет получен сигнал завершения.

```
ssc.start()
ssc.awaitTermination()
```

Дополнительные сведения об API потока Spark см. в статье [Apache Spark Пошаговое программирование для потоков](https://people.apache.org/~pwendell/spark-releases/latest/streaming-programming-guide.html).

Следующий пример приложения является автономным, поэтому его можно запустить в [блокноте Jupyter](apache-spark-jupyter-notebook-kernels.md). В этом примере создается макет источника данных в классе DummySource, который выводит значение счетчика и текущее время в миллисекундах каждые пять секунд. Для нового объекта StreamingContext интервал пакетной обработки составляет 30 секунд. При каждом создании пакета приложение потоковой передачи проверяет созданный RDD. Затем преобразует RDD в кадр данных Spark и создает временную таблицу в кадре данных.

```
class DummySource extends org.apache.spark.streaming.receiver.Receiver[(Int, Long)](org.apache.spark.storage.StorageLevel.MEMORY_AND_DISK_2) {

    /** Start the thread that simulates receiving data */
    def onStart() {
        new Thread("Dummy Source") { override def run() { receive() } }.start()
    }

    def onStop() {  }

    /** Periodically generate a random number from 0 to 9, and the timestamp */
    private def receive() {
        var counter = 0  
        while(!isStopped()) {
            store(Iterator((counter, System.currentTimeMillis)))
            counter += 1
            Thread.sleep(5000)
        }
    }
}

// A batch is created every 30 seconds
val ssc = new org.apache.spark.streaming.StreamingContext(spark.sparkContext, org.apache.spark.streaming.Seconds(30))

// Set the active SQLContext so that we can access it statically within the foreachRDD
org.apache.spark.sql.SQLContext.setActive(spark.sqlContext)

// Create the stream
val stream = ssc.receiverStream(new DummySource())

// Process RDDs in the batch
stream.foreachRDD { rdd =>

    // Access the SQLContext and create a table called demo_numbers we can query
    val _sqlContext = org.apache.spark.sql.SQLContext.getOrCreate(rdd.sparkContext)
    _sqlContext.createDataFrame(rdd).toDF("value", "time")
        .registerTempTable("demo_numbers")
}

// Start the stream processing
ssc.start()
```

Подождите около 30 секунд после запуска приложения выше.  Затем можно периодически выполнять запрос к кадру данных, чтобы увидеть текущий набор значений, представленных в пакете, например с помощью запроса SQL:

```sql
%%sql
SELECT * FROM demo_numbers
```

Итоговые выходные данные выглядят следующим образом:

| значение | time |
| --- | --- |
|10 | 1497314465256 |
|11 | 1497314470272 |
|12 | 1497314475289 |
|13 | 1497314480310 |
|14 | 1497314485327 |
|15 | 1497314490346 |

Выводятся шесть значений, так как DummySource создает значение каждые 5 секунд, а пакет выпускается приложением каждые 30 секунд.

## <a name="sliding-windows"></a>Скользящие окна

Для выполнения статистических вычислений в DStream за некоторый период времени, например для получения средней температуры за последние две секунды, используйте `sliding window` операции, включенные в потоковую передачу Spark. Скользящее окно имеет продолжительность и интервал, во время которого вычисляется содержимое окна (интервал скольжения).

Скользящие окна могут перекрывать друг друга. Например, можно определить окно с продолжительностью две секунды и интервалом скольжения в одну секунду. Это действие означает при каждом выполнении статистического вычисления, в окне будут содержаться данные из последней секунды предыдущего окна. И все новые данные в следующей секунде.

:::image type="content" source="./media/apache-spark-streaming-overview/hdinsight-spark-streaming-window-01.png" alt-text="Пример начального окна с событиями температуры" border="false":::

:::image type="content" source="./media/apache-spark-streaming-overview/hdinsight-spark-streaming-window-02.png" alt-text="Пример окна с событиями температуры после изменения слайда" border="false":::

В следующем примере обновляется код, который использует DummySource для сбора пакетов в окно длительностью в одну минуту и интервалом скольжения в одну минуту.

```
class DummySource extends org.apache.spark.streaming.receiver.Receiver[(Int, Long)](org.apache.spark.storage.StorageLevel.MEMORY_AND_DISK_2) {

    /** Start the thread that simulates receiving data */
    def onStart() {
        new Thread("Dummy Source") { override def run() { receive() } }.start()
    }

    def onStop() {  }

    /** Periodically generate a random number from 0 to 9, and the timestamp */
    private def receive() {
        var counter = 0  
        while(!isStopped()) {
            store(Iterator((counter, System.currentTimeMillis)))
            counter += 1
            Thread.sleep(5000)
        }
    }
}

// A batch is created every 30 seconds
val ssc = new org.apache.spark.streaming.StreamingContext(spark.sparkContext, org.apache.spark.streaming.Seconds(30))

// Set the active SQLContext so that we can access it statically within the foreachRDD
org.apache.spark.sql.SQLContext.setActive(spark.sqlContext)

// Create the stream
val stream = ssc.receiverStream(new DummySource())

// Process batches in 1 minute windows
stream.window(org.apache.spark.streaming.Minutes(1)).foreachRDD { rdd =>

    // Access the SQLContext and create a table called demo_numbers we can query
    val _sqlContext = org.apache.spark.sql.SQLContext.getOrCreate(rdd.sparkContext)
    _sqlContext.createDataFrame(rdd).toDF("value", "time")
    .registerTempTable("demo_numbers")
}

// Start the stream processing
ssc.start()
```

После истечения первой минуты будут получены 12 записей — по шесть записей из каждого из двух пакетов, собранных в окне.

| значение | time |
| --- | --- |
| 1 | 1497316294139 |
| 2 | 1497316299158
| 3 | 1497316304178
| 4 | 1497316309204
| 5 | 1497316314224
| 6 | 1497316319243
| 7 | 1497316324260
| 8 | 1497316329278
| 9 | 1497316334293
| 10 | 1497316339314
| 11 | 1497316344339
| 12 | 1497316349361

В функции скользящего окна, доступные в API потоковой передачи Spark, входит window, countByWindow, reduceByWindow и countByValueAndWindow. Дополнительные сведения об этих функциях см. в разделе [Transformations on DStreams](https://people.apache.org/~pwendell/spark-releases/latest/streaming-programming-guide.html#transformations-on-dstreams) (Преобразования в потоках DStream).

## <a name="checkpointing"></a>Назначение контрольных точек

Для обеспечения отказоустойчивости в потоковой передаче Spark используются контрольные точки, позволяющие гарантировать, что потоки обрабатываются непрерывно даже при сбоях узлов. Spark создает контрольные точки в долговременном хранилище (хранилище Azure или Data Lake Storage). Эти контрольные точки хранят метаданные приложения потоковой передачи, такие как конфигурация, и операции, определенные приложением. Кроме того, все пакеты, которые были поставлены в очередь, но еще не обработаны. Иногда контрольные точки также включают сохранение данных в RDD для более быстрого перестроения состояния данных из RDD, управляемого Spark.

## <a name="deploying-spark-streaming-applications"></a>Развертывание приложений потоковой передачи Spark

Обычно приложение потоковой передачи Spark создается локально в JAR-файле. Затем разверните его в Spark в HDInsight, скопировав JAR-файл в подключенное по умолчанию хранилище. Можно запустить приложение с использованием интерфейсов REST API LIVY, к которым можно получить доступ в кластере с помощью операции POST. Текст записи включает документ JSON, который предоставляет путь к JAR-файлу. И имя класса, главный метод которого определяет и запускает приложение потоковой передачи, а также при необходимости требования к ресурсам задания (например, число исполнителей, память и ядра). Кроме того, все параметры конфигурации, необходимые для кода приложения.

:::image type="content" source="./media/apache-spark-streaming-overview/hdinsight-spark-streaming-livy.png" alt-text="Развертывание приложения потоковой передачи Spark" border="false":::

Состояние всех приложений можно также проверить с помощью запроса GET к конечной точке LIVY. Наконец, можно завершить работу приложения, выполнив запрос DELETE к конечной точке LIVY. Дополнительные сведения об API LIVY см. в статье [Удаленная отправка заданий Spark в кластер Azure HDInsight с помощью Apache Spark REST API](apache-spark-livy-rest-interface.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Создание кластеров под управлением Linux в HDInsight с помощью портала Azure](../hdinsight-hadoop-create-linux-clusters-portal.md)
* [Руководство по программированию потоковой передачи Apache Spark](https://people.apache.org/~pwendell/spark-releases/latest/streaming-programming-guide.html)
* [Overview of Apache Spark Structured Streaming](apache-spark-structured-streaming-overview.md) (Обзор структурированной потоковой передачи Apache Spark)
