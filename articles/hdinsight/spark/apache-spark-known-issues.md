---
title: Устранение неполадок с кластером Apache Spark в Azure HDInsight
description: Сведения о проблемах, связанных с кластерами Apache Spark в Azure HDInsight, и их решении.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: troubleshooting
ms.date: 08/15/2019
ms.openlocfilehash: c92b55d3ac7f4476b7b74d25b40150a74c6ea1cf
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98930149"
---
# <a name="known-issues-for-apache-spark-cluster-on-hdinsight"></a>Известные проблемы в работе кластера Apache Spark в HDInsight

В этом документе отслеживаются все известные проблемы с общедоступной предварительной версией Spark HDInsight.  

## <a name="apache-livy-leaks-interactive-session"></a>Утечка интерактивного сеанса Apache Livy
При перезапуске [Apache Livy](https://livy.incubator.apache.org/) (из [Apache Ambari](https://ambari.apache.org/) или в связи с перезапуском виртуальной машины headnode 0) во время работы интерактивного сеанса происходит утечка сеанса интерактивного задания. В результате этого новые задания могут "зависнуть" в состоянии "Принято".

**Устранение рисков**

Для решения этой проблемы выполните указанные ниже действия:

1. Подключитесь к головному узлу по протоколу SSH. См. дополнительные сведения об [использовании SSH в HDInsight](../hdinsight-hadoop-linux-use-ssh-unix.md).

2. Выполните следующую команду, чтобы найти идентификаторы приложений для интерактивных заданий, запущенных из Livy:

   ```bash
   yarn application –list
   ```

    Если задания запущены с помощью интерактивного сеанса Livy без явно заданных имен, по умолчанию используются имена Livy. Для сеанса Livy, запущенного [Jupyter Notebook](https://jupyter.org/), имя задания начинается с `remotesparkmagics_*` .

3. Чтобы аннулировать эти задания, выполните следующую команду:

   ```bash
   yarn application –kill <Application ID>
   ```

После этого начинают выполняться новые задания.

## <a name="spark-history-server-not-started"></a>Сервер журналов Spark не запускается
Сервер журналов Spark не запускается автоматически после создания кластера.  

**Устранение рисков**

Вручную запустите сервер журналов из Ambari.

## <a name="permission-issue-in-spark-log-directory"></a>Проблема разрешений в каталоге журналов Spark
Отправляя задание с помощью команды spark-submit, hdiuser возвращает следующую ошибку:

```
java.io.FileNotFoundException: /var/log/spark/sparkdriver_hdiuser.log (Permission denied)
```

При этом журнал драйверов не ведется.

**Устранение рисков**

1. Добавьте hdiuser в группу Hadoop.
2. После создания кластера предоставьте разрешения 777 для каталога /var/log/spark.
3. В Ambari измените путь размещения журнала Spark на каталог с разрешениями 777.  
4. Выполните команду spark-submit как sudo.  

## <a name="spark-phoenix-connector-is-not-supported"></a>Соединитель Spark-Phoenix не поддерживается

Кластеры HDInsight Spark не поддерживают соединитель Spark-Phoenix.

**Устранение рисков**

Вместо него необходимо использовать соединитель Spark-HBase. Инструкции см. в записи блога об [использовании соединителя Spark-HBase](https://web.archive.org/web/20190112153146/https://blogs.msdn.microsoft.com/azuredatalake/2016/07/25/hdinsight-how-to-use-spark-hbase-connector/).

## <a name="issues-related-to-jupyter-notebooks"></a>Проблемы, связанные с записными книжками Jupyter

Ниже приведены некоторые известные проблемы, связанные с записными книжками Jupyter.

### <a name="notebooks-with-non-ascii-characters-in-filenames"></a>Записные книжки со знаками не из набора ASCII в именах файлов

Не используйте символы, отличные от ASCII, в Jupyter Notebook именах файлов. При попытке отправить через пользовательский интерфейс Jupyter файл, имя которого содержит символы не из набора ASCII, происходит сбой без сообщения об ошибке. Jupyter не позволяет отправить файл, но и не выдает явную ошибку.

### <a name="error-while-loading-notebooks-of-larger-sizes"></a>Ошибка при загрузке записных книжек большого размера

При **`Error loading notebook`** загрузке записных книжек большего размера может появиться сообщение об ошибке.  

**Устранение рисков**

Появление этой ошибки не означает, что данные повреждены или утрачены.  Записные книжки по-прежнему хранятся на диске в каталоге `/var/lib/jupyter`, и вы можете получить к ним доступ, подключившись к кластеру по протоколу SSH. См. дополнительные сведения об [использовании SSH в HDInsight](../hdinsight-hadoop-linux-use-ssh-unix.md).

После подключения к кластеру с помощью SSH во избежание потери важных данных записные книжки можно скопировать из кластера на локальный компьютер (с помощью SCP или WinSCP), тем самым создав резервную копию. Затем вы сможете использовать туннель SSH к головному узлу через порт 8001, чтобы получить доступ к Jupyter без прохождения через шлюз.  В Jupyter можно очистить выходные данные записной книжки и повторно сохранить ее, чтобы уменьшить ее размер.

Чтобы избежать этой ошибки в будущем, необходимо следовать некоторым рекомендациям.

* Записная книжка должна быть небольшого размера. В записной книжке сохраняются любые выходные данные заданий Spark, которые отправляются обратно в Jupyter.  Рекомендуется использовать Jupyter в целом, чтобы не запускать `.collect()` большие RDD или кадры данных. вместо этого, если вы хотите просмотреть содержимое RDD, рассмотрите возможность запуска `.take()` или `.sample()` , чтобы выходные данные не слишком большими.
* Кроме того, если при сохранении записной книжки очистить все ячейки выходных данных, это также поможет уменьшить размер.

### <a name="notebook-initial-startup-takes-longer-than-expected"></a>Начальная загрузка записной книжки загружается дольше ожидаемого

Первая инструкция Code в Jupyter Notebook с помощью функции Spark Magic может занять больше минуты.  

**Объяснение**

Это происходит из-за выполнения первой ячейки кода. В фоновом режиме инициируется конфигурация сеанса и задается контекст Spark, SQL и Hive. После настройки этих контекстов выполняется первая инструкция, и из-за этого кажется, что она выполняется долго.

### <a name="jupyter-notebook-timeout-in-creating-the-session"></a>Jupyter Notebook время ожидания при создании сеанса

Если в кластере Spark исчерпаны ресурсы, то при попытке создать сеанс в ядре Spark и PySpark в Jupyter Notebook будет превышено время ожидания.

**Способы их устранения**

1. Освободите часть ресурсов в кластере Spark.

   * Остановите другие записные книжки Spark в меню "Закрыть и остановить" или нажмите кнопку "Завершить" в обозревателе записных книжек.
   * Остановите другие приложения Spark из YARN.

2. Перезапустите записную книжку, которую вы пытались запустить. Теперь ресурсов должно быть достаточно для создания сеанса.

## <a name="see-also"></a>См. также раздел

* [Обзор: Spark в Azure HDInsight](apache-spark-overview.md)

### <a name="scenarios"></a>Сценарии

* [Использование Apache Spark со средствами бизнес-аналитики. Выполнение интерактивного анализа данных с использованием Spark в HDInsight с помощью средств бизнес-аналитики](apache-spark-use-bi-tools.md)
* [Apache Spark и Машинное обучение. Анализ температуры в здании на основе данных системы кондиционирования с помощью Spark в HDInsight](apache-spark-ipython-notebook-machine-learning.md)
* [Apache Spark и Машинное обучение. Прогнозирование результатов проверки пищевых продуктов с помощью Spark в HDInsight](apache-spark-machine-learning-mllib-ipython.md)
* [Анализ журналов веб-сайтов с помощью Apache Spark в HDInsight](apache-spark-custom-library-website-log-analysis.md)

### <a name="create-and-run-applications"></a>Создание и запуск приложений

* [Создание автономного приложения с использованием Scala](apache-spark-create-standalone-application.md)
* [Удаленный запуск заданий с помощью Apache Livy в кластере Apache Spark](apache-spark-livy-rest-interface.md)

### <a name="tools-and-extensions"></a>Инструменты и расширения

* [Использование подключаемого модуля средств HDInsight для IntelliJ IDEA для создания и отправки приложений Spark Scala](apache-spark-intellij-tool-plugin.md)
* [Удаленная отладка приложений Apache Spark с помощью подключаемого модуля средств HDInsight для IntelliJ IDEA](apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)
* [Использование записных книжек Zeppelin с кластером Apache Spark в Azure HDInsight](apache-spark-zeppelin-notebook.md)
* [Ядра, доступные для Jupyter Notebook в кластере Apache Spark для HDInsight](apache-spark-jupyter-notebook-kernels.md)
* [Использование внешних пакетов с записными книжками Jupyter](apache-spark-jupyter-notebook-use-external-packages.md)
* [Установка записной книжки Jupyter на компьютере и ее подключение к кластеру Apache Spark в Azure HDInsight (предварительная версия)](apache-spark-jupyter-notebook-install-locally.md)

### <a name="manage-resources"></a>Управление ресурсами

* [Управление ресурсами кластера Apache Spark в Azure HDInsight](apache-spark-resource-manager.md)
* [Отслеживание и отладка заданий в кластере Apache Spark в HDInsight на платформе Linux](apache-spark-job-debugging.md)
