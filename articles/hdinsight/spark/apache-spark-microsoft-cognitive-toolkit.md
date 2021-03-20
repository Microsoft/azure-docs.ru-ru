---
title: Microsoft Cognitive Toolkit с Apache Spark Azure HDInsight
description: Узнайте, как можно применить обученную модель обучения Microsoft Cognitive Toolkit к набору данных с помощью API Python для Spark в кластере Azure HDInsight Spark.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 01/14/2020
ms.openlocfilehash: cddbc4b6a5c7a2c787c8305fdf703e34543746f8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98929965"
---
# <a name="use-microsoft-cognitive-toolkit-deep-learning-model-with-azure-hdinsight-spark-cluster"></a>Использование модели глубокого обучения Microsoft Cognitive Toolkit в кластере Azure HDInsight Spark

Ниже перечислены действия, которые вы выполните в этой статье.

1. Запустите пользовательский скрипт для установки [Microsoft Cognitive Toolkit](/cognitive-toolkit/) в кластере Azure HDInsight Spark.

2. передача [Jupyter Notebook](https://jupyter.org/) в кластер [Apache Spark](https://spark.apache.org/) для применения обученной модели глубокого обучения Microsoft Cognitive Toolkit к файлам в учетной записи хранилища BLOB-объектов Azure с помощью [API Python для Spark (PySpark)](https://spark.apache.org/docs/latest/api/python/index.html).

## <a name="prerequisites"></a>Предварительные требования

* Кластер Apache Spark в HDInsight. Ознакомьтесь со статьей [Краткое руководство. Создание кластера Apache Spark в HDInsight с помощью шаблона](./apache-spark-jupyter-spark-sql-use-portal.md).

* Опыт работы с записными книжками Jupyter с Spark в HDInsight. Дополнительные сведения см. в статье [Руководство. Загрузка данных и выполнение запросов в кластере Apache Spark в Azure HDInsight](./apache-spark-load-data-run-query.md).

## <a name="how-does-this-solution-flow"></a>Как реализуется это решение?

Это решение делится на эту статью и Jupyter Notebook, которую вы отправляете в рамках этой статьи. Ниже перечислены действия, которые вы выполните в этой статье.

* Запуск действия сценария в кластере HDInsight Spark для установки Microsoft Cognitive Toolkit и пакетов Python.
* Загрузите Jupyter Notebook, который запускает решение, в кластер HDInsight Spark.

Следующие оставшиеся действия описаны в Jupyter Notebook.

* Загрузите образцы изображений в отказоустойчивый распределенный набор данных Spark или RDD.
  * Загрузка модулей и определение предустановок.
  * Скачивание набора данных локально в кластер Spark.
  * Преобразование набора данных в RDD.
* Оценка изображений с помощью обученной модели Cognitive Toolkit.
  * Скачивание обученной модели Cognitive Toolkit в кластер Spark.
  * Определение функций, используемых рабочими узлами.
  * Оценка изображений на рабочих узлах.
  * Анализ точности модели.

## <a name="install-microsoft-cognitive-toolkit"></a>Установка Microsoft Cognitive Toolkit

Microsoft Cognitive Toolkit в кластере Spark можно установить с помощью действия сценария. Действие скрипта использует пользовательские скрипты для установки компонентов в кластере, которые недоступны по умолчанию. Пользовательский скрипт можно использовать из портал Azure, с помощью пакета SDK для HDInsight .NET или с помощью Azure PowerShell. Этот сценарий можно также использовать для установки данного набора средств при создании кластера или после его подготовки и запуска.

В этой статье мы используем портал для установки набора средств после того, как кластер был создан. Другие способы выполнения пользовательского сценария описаны в разделе [Настройка кластеров HDInsight под управлением Linux с помощью действия сценария](../hdinsight-hadoop-customize-cluster-linux.md).

### <a name="using-the-azure-portal"></a>Использование портала Azure

Инструкции по использованию портал Azure для выполнения действия сценария см. в разделе [Настройка кластеров HDInsight с помощью действия сценария](../hdinsight-hadoop-customize-cluster-linux.md#script-action-during-cluster-creation). Обязательно укажите приведенные ниже данные для установки Microsoft Cognitive Toolkit. Используйте следующие значения для действия скрипта:

|Свойство. |Значение |
|---|---|
|Тип скрипта|- Custom|
|Имя| Установка MCT|
|URI bash-скрипта|`https://raw.githubusercontent.com/Azure-Samples/hdinsight-pyspark-cntk-integration/master/cntk-install.sh`|
|Типы узлов:|Головной, Рабочий|
|Параметры|Отсутствуют|

## <a name="upload-the-jupyter-notebook-to-azure-hdinsight-spark-cluster"></a>Отправка Jupyter Notebook в кластер Azure HDInsight Spark

Чтобы использовать Microsoft Cognitive Toolkit с кластером Azure HDInsight Spark, необходимо загрузить Jupyter Notebook **CNTK_model_scoring_on_Spark_walkthrough. ipynb** в кластер Azure HDInsight Spark. Эта записная книжка доступна на сайте GitHub по адресу [https://github.com/Azure-Samples/hdinsight-pyspark-cntk-integration](https://github.com/Azure-Samples/hdinsight-pyspark-cntk-integration) .

1. Скачайте и распакуйте [https://github.com/Azure-Samples/hdinsight-pyspark-cntk-integration](https://github.com/Azure-Samples/hdinsight-pyspark-cntk-integration) .

1. В веб-браузере перейдите на страницу `https://CLUSTERNAME.azurehdinsight.net/jupyter`, где `CLUSTERNAME` — это имя вашего кластера.

1. В Jupyter Notebook выберите **Отправить** в правом верхнем углу, а затем перейдите к файлу Download (скачать) и выберите File (файл) `CNTK_model_scoring_on_Spark_walkthrough.ipynb` .

    ![Отправка Jupyter Notebook в кластер Azure HDInsight Spark](./media/apache-spark-microsoft-cognitive-toolkit/hdinsight-microsoft-cognitive-toolkit-load-jupyter-notebook.png "Отправка Jupyter Notebook в кластер Azure HDInsight Spark")

1. Выберите **Отправить** еще раз.

1. После отправки записной книжки щелкните имя записной книжки, а затем следуйте инструкциям в самой записной книжке о том, как загрузить набор данных и выполнить статью.

## <a name="see-also"></a>См. также раздел

* [Обзор: Spark в Azure HDInsight](apache-spark-overview.md)

### <a name="scenarios"></a>Сценарии

* [Использование Apache Spark со средствами бизнес-аналитики. Выполнение интерактивного анализа данных с использованием Spark в HDInsight с помощью средств бизнес-аналитики](apache-spark-use-bi-tools.md)
* [Apache Spark и Машинное обучение. Анализ температуры в здании на основе данных системы кондиционирования с помощью Spark в HDInsight](apache-spark-ipython-notebook-machine-learning.md)
* [Apache Spark и Машинное обучение. Прогнозирование результатов проверки пищевых продуктов с помощью Spark в HDInsight](apache-spark-machine-learning-mllib-ipython.md)
* [Анализ журналов веб-сайтов с помощью Apache Spark в HDInsight](apache-spark-custom-library-website-log-analysis.md)
* [Анализ журналов телеметрии Application Insights с помощью Apache Spark в HDInsight](apache-spark-analyze-application-insight-logs.md)

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
