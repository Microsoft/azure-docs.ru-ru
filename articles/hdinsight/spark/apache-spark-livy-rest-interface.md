---
title: Отправка заданий в кластер Spark в Azure HDInsight с помощью Livy Spark
description: Узнайте, как выполнить удаленную отправку заданий Spark в кластер Azure HDInsight с помощью Apache Spark REST API.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017
ms.date: 02/28/2020
ms.openlocfilehash: ff63f4fbadd7cb9e7584e2aa045583a35e0363fd
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98930131"
---
# <a name="use-apache-spark-rest-api-to-submit-remote-jobs-to-an-hdinsight-spark-cluster"></a>Удаленная отправка заданий Spark в кластер Azure HDInsight с помощью Apache Spark REST API

Узнайте, как использовать [Apache Livy](https://livy.incubator.apache.org/), интерфейс Apache Spark REST API, который применяется для удаленной отправки заданий в кластер Azure HDInsight Spark. Подробную документацию см. на [Apache Livy](https://livy.incubator.apache.org/docs/latest/rest-api.html).

Вы можете использовать Livy для выполнения интерактивных оболочек Spark или отправки пакетных заданий для запуска в кластере Spark. В статье рассматривается использование Livy для отправки пакетных заданий. Во фрагментах кода в этой статье используется Curl для отправки вызовов REST API к конечной точке Livy Spark.

## <a name="prerequisites"></a>Предварительные требования

Кластер Apache Spark в HDInsight. Инструкции см. в статье [Начало работы. Создание кластера Apache Spark в HDInsight на платформе Linux и выполнение интерактивных запросов с помощью SQL Spark](apache-spark-jupyter-spark-sql.md).

## <a name="submit-an-apache-livy-spark-batch-job"></a>Отправка пакетного задания Apache Livy Spark

Перед отправкой пакетного задания необходимо загрузить JAR-файл приложения в хранилище кластеров, связанное с соответствующим кластером. Вы можете использовать для этого служебную программу командной строки [AzCopy](../../storage/common/storage-use-azcopy-v10.md). Кроме того, для отправки данных вы можете использовать множество других клиентов. Дополнительные сведения о них см. в статье [Отправка данных для заданий Apache Hadoop в HDInsight](../hdinsight-upload-data.md).

```cmd
curl -k --user "admin:password" -v -H "Content-Type: application/json" -X POST -d '{ "file":"<path to application jar>", "className":"<classname in jar>" }' 'https://<spark_cluster_name>.azurehdinsight.net/livy/batches' -H "X-Requested-By: admin"
```

### <a name="examples"></a>Примеры

* Если JAR-файл находится в хранилище кластера (WASBS)

    ```cmd  
    curl -k --user "admin:mypassword1!" -v -H "Content-Type: application/json" -X POST -d '{ "file":"wasbs://mycontainer@mystorageaccount.blob.core.windows.net/data/SparkSimpleTest.jar", "className":"com.microsoft.spark.test.SimpleFile" }' "https://mysparkcluster.azurehdinsight.net/livy/batches" -H "X-Requested-By: admin"
    ```

* Если имя JAR-файла и имя класса необходимо передать в составе входного файла (в данном примере input.txt).

    ```cmd
    curl -k  --user "admin:mypassword1!" -v -H "Content-Type: application/json" -X POST --data @C:\Temp\input.txt "https://mysparkcluster.azurehdinsight.net/livy/batches" -H "X-Requested-By: admin"
    ```

## <a name="get-information-on-livy-spark-batches-running-on-the-cluster"></a>Получение сведений о пакетах Livy Spark, выполняемых в кластере

Синтаксис:

```cmd
curl -k --user "admin:password" -v -X GET "https://<spark_cluster_name>.azurehdinsight.net/livy/batches"
```

### <a name="examples"></a>Примеры

* Если вы хотите получить все пакеты Livy Spark, выполняемые в кластере:

    ```cmd
    curl -k --user "admin:mypassword1!" -v -X GET "https://mysparkcluster.azurehdinsight.net/livy/batches"
    ```

* Если требуется получить конкретный пакет с заданным ИДЕНТИФИКАТОРом пакета

    ```cmd
    curl -k --user "admin:mypassword1!" -v -X GET "https://mysparkcluster.azurehdinsight.net/livy/batches/{batchId}"
    ```

## <a name="delete-a-livy-spark-batch-job"></a>Удаление пакетного задания Livy Spark

```cmd
curl -k --user "admin:mypassword1!" -v -X DELETE "https://<spark_cluster_name>.azurehdinsight.net/livy/batches/{batchId}"
```

### <a name="example"></a>Пример

Удаление пакетного задания с ИДЕНТИФИКАТОРом пакета `5` .

```cmd
curl -k --user "admin:mypassword1!" -v -X DELETE "https://mysparkcluster.azurehdinsight.net/livy/batches/5"
```

## <a name="livy-spark-and-high-availability"></a>Livy Spark и высокая доступность

Livy обеспечивает высокую доступность заданий Spark, выполняемых в кластере. Вот несколько примеров.

* Если после удаленной отправки задания в кластер Spark служба Livy перестает работать, задание продолжит выполняться в фоновом режиме. Выполняя резервное копирование, служба Livy восстанавливает состояние задания и отправляет отчет.
* Записные книжки Jupyter для HDInsight работают на базе Livy в серверной части. Если записная книжка выполняет задание Spark, а служба Livy перезапускается, записная книжка продолжит выполнять ячейки кода.

## <a name="show-me-an-example"></a>Показать пример

В этом разделе мы рассмотрим примеры использования Livy Spark для отправки пакетного задания, проследим за ходом выполнения задания, а затем удалим это задание. В примере используется приложение, разработанное в статье [Создание автономного приложения Scala для работы в кластере HDInsight Spark (Linux)](apache-spark-create-standalone-application.md). Здесь предполагается следующее:

* Вы уже скопировали JAR-файл приложения в учетную запись хранения, связанную с кластером.
* Вы установили фигурные скобки на компьютере, где вы пытаетесь выполнить эти действия.

Выполните следующие действия:

1. Для простоты использования задайте переменные среды. Этот пример основан на среде Windows, изменяя переменные по мере необходимости для вашей среды. Замените `CLUSTERNAME` и `PASSWORD` соответствующими значениями.

    ```cmd
    set clustername=CLUSTERNAME
    set password=PASSWORD
    ```

1. Убедитесь, что в кластере выполняется Livy Spark. Для этого получите список выполняемых пакетов. Если вы запускаете задание с помощью Livy в первый раз, выходные данные должны вернуть ноль.

    ```cmd
    curl -k --user "admin:%password%" -v -X GET "https://%clustername%.azurehdinsight.net/livy/batches"
    ```

    Должен отобразиться результат, аналогичный приведенному ниже фрагменту кода.

    ```output
    < HTTP/1.1 200 OK
    < Content-Type: application/json; charset=UTF-8
    < Server: Microsoft-IIS/8.5
    < X-Powered-By: ARR/2.5
    < X-Powered-By: ASP.NET
    < Date: Fri, 20 Nov 2015 23:47:53 GMT
    < Content-Length: 34
    <
    {"from":0,"total":0,"sessions":[]}* Connection #0 to host mysparkcluster.azurehdinsight.net left intact
    ```

    Обратите внимание на то, что последняя строка выходных данных содержит значение **total:0**, означающее, что никакие пакеты не выполняются.

1. Теперь отправим пакетное задание. В приведенном ниже фрагменте кода для передачи имени JAR-файла и имени класса в качестве параметров используется входной файл (input.txt). Если вы выполняете эти действия на компьютере Windows, рекомендуется использовать входной файл.

    ```cmd
    curl -k --user "admin:%password%" -v -H "Content-Type: application/json" -X POST --data @C:\Temp\input.txt "https://%clustername%.azurehdinsight.net/livy/batches" -H "X-Requested-By: admin"
    ```

    Параметры в файле **input.txt** определяются следующим образом:

    ```text
    { "file":"wasbs:///example/jars/SparkSimpleApp.jar", "className":"com.microsoft.spark.example.WasbIOTest" }
    ```

    Должен отобразиться результат, аналогичный приведенному ниже фрагменту кода.

    ```output
    < HTTP/1.1 201 Created
    < Content-Type: application/json; charset=UTF-8
    < Location: /0
    < Server: Microsoft-IIS/8.5
    < X-Powered-By: ARR/2.5
    < X-Powered-By: ASP.NET
    < Date: Fri, 20 Nov 2015 23:51:30 GMT
    < Content-Length: 36
    <
    {"id":0,"state":"starting","log":[]}* Connection #0 to host mysparkcluster.azurehdinsight.net left intact
    ```

    Обратите внимание на то, что последняя строка выходных данных содержит значение **state:starting**. Кроме того, она включает параметр **id:0** — **0** означает идентификатор пакета.

1. Теперь вы можете извлечь состояние конкретного пакета по идентификатору соответствующего пакета.

    ```cmd
    curl -k --user "admin:%password%" -v -X GET "https://%clustername%.azurehdinsight.net/livy/batches/0"
    ```

    Должен отобразиться результат, аналогичный приведенному ниже фрагменту кода.

    ```output
    < HTTP/1.1 200 OK
    < Content-Type: application/json; charset=UTF-8
    < Server: Microsoft-IIS/8.5
    < X-Powered-By: ARR/2.5
    < X-Powered-By: ASP.NET
    < Date: Fri, 20 Nov 2015 23:54:42 GMT
    < Content-Length: 509
    <
    {"id":0,"state":"success","log":["\t diagnostics: N/A","\t ApplicationMaster host: 10.0.0.4","\t ApplicationMaster RPC port: 0","\t queue: default","\t start time: 1448063505350","\t final status: SUCCEEDED","\t tracking URL: http://myspar.lpel.jx.internal.cloudapp.net:8088/proxy/application_1447984474852_0002/","\t user: root","15/11/20 23:52:47 INFO Utils: Shutdown hook called","15/11/20 23:52:47 INFO Utils: Deleting directory /tmp/spark-b72cd2bf-280b-4c57-8ceb-9e3e69ac7d0c"]}* Connection #0 to host mysparkcluster.azurehdinsight.net left intact
    ```

    Теперь выходные данные содержат параметр **state:success**, означающий, что задание выполнено.

1. При необходимости пакет можно удалить.

    ```cmd
    curl -k --user "admin:%password%" -v -X DELETE "https://%clustername%.azurehdinsight.net/livy/batches/0"
    ```

    Должен отобразиться результат, аналогичный приведенному ниже фрагменту кода.

    ```output
    < HTTP/1.1 200 OK
    < Content-Type: application/json; charset=UTF-8
    < Server: Microsoft-IIS/8.5
    < X-Powered-By: ARR/2.5
    < X-Powered-By: ASP.NET
    < Date: Sat, 21 Nov 2015 18:51:54 GMT
    < Content-Length: 17
    <
    {"msg":"deleted"}* Connection #0 to host mysparkcluster.azurehdinsight.net left intact
    ```

    Последняя строка выходных данных показывает, что пакет удален. Удаление задания во время его выполнения также ликвидирует задание. Если задание удаляется после завершения (независимо от успешности выполнения), удаляются все данные об этом задании.

## <a name="updates-to-livy-configuration-starting-with-hdinsight-35-version"></a>Обновления конфигурации Livy, начиная с версии HDInsight 3.5

Кластер HDInsight версии 3.5 и выше по умолчанию запрещает использование локальных путей для доступа к примерам файлов данных или JAR-файлам. Мы рекомендуем использовать путь `wasbs://` для доступа к JAR-файлам или примерам файлов данных из кластера.

## <a name="submitting-livy-jobs-for-a-cluster-within-an-azure-virtual-network"></a>Отправка задания Livy для кластера в пределах виртуальной сети Azure

При подключении к кластеру HDInsight Spark из виртуальной сети Azure можно напрямую подключаться к Livy в кластере. В этом случае URL-адрес для конечной точки Livy — `http://<IP address of the headnode>:8998/batches`. Здесь **8998** — это порт, на котором выполняется Livy на головном узле кластера. Дополнительные сведения о доступе к службам через порты, не являющиеся общедоступными, см. в разделе [Порты, используемые службами Hadoop в HDInsight](../hdinsight-hadoop-port-settings-for-services.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Документация по Apache Livy REST API](https://livy.incubator.apache.org/docs/latest/rest-api.html)
* [Управление ресурсами кластера Apache Spark в Azure HDInsight](apache-spark-resource-manager.md)
* [Отслеживание и отладка заданий в кластере Apache Spark в HDInsight на платформе Linux](apache-spark-job-debugging.md)