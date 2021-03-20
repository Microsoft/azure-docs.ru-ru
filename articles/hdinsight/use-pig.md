---
title: Использование Apache Pig
titleSuffix: Azure HDInsight
description: Узнайте, как использовать Pig с Apache Hadoop в HDInsight.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 01/28/2020
ms.openlocfilehash: 7b74a41f7d6b636dddce0388d5ee0e0a12658d52
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98944617"
---
# <a name="use-apache-pig-with-apache-hadoop-on-hdinsight"></a>Использование Apache Pig с Apache Hadoop в HDInsight

Узнайте, как использовать [Apache Pig](https://pig.apache.org/) с HDInsight.

Apache Pig — это платформа, позволяющая создавать программы для Hadoop с помощью процедурного языка, известного как *Pig Latin*. Pig, который является альтернативой Java для создания решений *MapReduce* , входит в состав Azure HDInsight. Воспользуйтесь приведенной таблицей, чтобы узнать, как можно использовать Pig с HDInsight.

## <a name="why-use-apache-pig"></a><a id="why"></a>Причины использования Apache Pig

Одна из трудностей обработки данных с помощью MapReduce в Hadoop заключается в реализации логики обработки с использованием только функций map и reduce. Для сложных задач часто приходится разбивать обработку на несколько операций MapReduce, соединенных друг с другом, чтобы достичь желаемого результата.

Pig позволяет определить обработку как ряд преобразований, через которые проходят данные для получения желаемого результата.

Язык Pig Latin позволяет описывать поток данных так, чтобы из необработанных входных данных можно было получить нужный результата посредством одного или нескольких преобразований. Программы Pig Latin следуют общему шаблону:

* **Загрузка**: чтение данных для обработки из файловой системы.

* **Преобразование**: обработка данных.

* **Дамп или хранение**: вывод данных на экран или сохранение их для обработки.

### <a name="user-defined-functions"></a>Пользовательские функции

Pig Latin также поддерживает пользовательские функции, позволяющие вызывать внешние компоненты, реализующие логику, которую трудно смоделировать в Pig Latin.

Дополнительные сведения о Pig Latin см. в [справочном руководстве по Pig Latin 1](https://archive.cloudera.com/cdh/3/pig/piglatin_ref1.html) и [справочном руководстве по Pig Latin 2](https://archive.cloudera.com/cdh/3/pig/piglatin_ref2.html).

## <a name="example-data"></a><a id="data"></a>Пример данных

HDInsight предоставляет различные наборы демонстрационных данных, которые хранятся в каталогах `/example/data` и `/HdiSamples`. Эти каталоги находятся в хранилище по умолчанию для кластера. В примере Pig в этом документе используется файл *log4j* из `/example/data/sample.log`.

Каждый журнал в файле состоит из строки полей, содержащей поле `[LOG LEVEL]` для отображения типа и серьезности.

```output
2012-02-03 20:26:41 SampleClass3 [ERROR] verbose detail for id 1527353937
```

В предыдущем примере уровень журнала имеет значение ERROR ("ошибка").

> [!NOTE]  
> Вы также можете создать файл log4j с помощью средства ведения журнала [Apache Log4j](https://en.wikipedia.org/wiki/Log4j), а затем отправить этот файл в большой двоичный объект. Дополнительные сведения см. в статье [Отправка данных для заданий Hadoop в HDInsight](hdinsight-upload-data.md). Дополнительные сведения о том, каким образом большие двоичные объекты в службе хранилища Azure используются с HDInsight, см. в статье [Использование хранилища BLOB-объектов Azure с HDInsight](hdinsight-hadoop-use-blob-storage.md).

## <a name="example-job"></a><a id="job"></a>Пример задания

Следующее задание Pig Latin загружает файл `sample.log` из хранилища по умолчанию для кластера HDInsight. Затем оно выполняет ряд преобразований, которые позволяют подсчитать, сколько раз каждый из уровней ведения журнала был указан во входных данных. Результаты записываются в поток STDOUT.

```output
LOGS = LOAD 'wasb:///example/data/sample.log';
LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;
FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;
GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;
FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;
RESULT = order FREQUENCIES by COUNT desc;
DUMP RESULT;
```

На следующем рисунке показано, что каждое из преобразований делает с данным.

![Графическое представление преобразований][image-hdi-pig-data-transformation]

## <a name="run-the-pig-latin-job"></a><a id="run"></a>Запуск задания Pig Latin

HDInsight может запускать задания Pig Latin с помощью различных методов. Используйте следующую таблицу, чтобы решить, какой метод подходит вам, а затем перейдите по ссылке к пошаговому руководству.

## <a name="pig-and-sql-server-integration-services"></a>Использование Pig и SQL Server Integration Services

С помощью служб SQL Server Integration Services (SSIS) можно выполнить задание Pig. Пакет дополнительных компонентов Azure для служб SSIS предоставляет следующие компоненты, которые работают с заданиями Pig в HDInsight.

* [Задача Pig для Azure HDInsight][pigtask]

* [Диспетчер подключений подписки Azure][connectionmanager]

Узнать больше о пакете дополнительных компонентов Azure для служб SSIS можно [здесь][ssispack].

## <a name="next-steps"></a><a id="nextsteps"></a>Следующие шаги

Теперь, когда вы узнали, как использовать Pig в HDInsight, воспользуйтесь следующими ссылками, чтобы изучить другие способы работы с Azure HDInsight.

* [Отправка данных в HDInsight](hdinsight-upload-data.md)
* [Использование Hive и HiveQL с Hadoop в HDInsight для анализа примера файла Apache log4j](./hadoop/hdinsight-use-hive.md)
* [Использование Apache Sqoop с HDInsight](./hadoop/hdinsight-use-sqoop.md)
* [Использование заданий MapReduce с HDInsight](./hadoop/hdinsight-use-mapreduce.md)

[apachepig-home]: https://pig.apache.org/
[putty]: https://www.chiark.greenend.org.uk/~sgtatham/putty/download.html
[curl]: https://curl.haxx.se/
[pigtask]: /sql/integration-services/control-flow/azure-hdinsight-pig-task?viewFallbackFrom=sql-server-2014
[connectionmanager]: /sql/integration-services/connection-manager/azure-subscription-connection-manager?viewFallbackFrom=sql-server-2014
[ssispack]: /sql/integration-services/azure-feature-pack-for-integration-services-ssis?viewFallbackFrom=sql-server-2014
[hdinsight-admin-powershell]: hdinsight-administer-use-powershell.md

[hdinsight-use-hive]:../hdinsight-use-hive.md

[hdinsight-provision]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-submit-jobs]:submit-apache-hadoop-jobs-programmatically.md#mapreduce-sdk

[Powershell-install-configure]: /powershell/azure/

[powershell-start]: https://technet.microsoft.com/library/hh847889.aspx


[image-hdi-pig-data-transformation]: ./media/use-pig/hdi-data-transformation.gif