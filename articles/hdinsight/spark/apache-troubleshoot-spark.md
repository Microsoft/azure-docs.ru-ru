---
title: Устранение неполадок Apache Spark в Azure HDInsight
description: Получите ответы на распространенные вопросы о работе с Apache Spark и Azure HDInsight.
ms.service: hdinsight
ms.reviewer: jasonh
ms.topic: troubleshooting
ms.date: 08/22/2019
ms.custom: seodec18
ms.openlocfilehash: b54b9d932505ada890ac21c1b8de3178ad2f0042
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104867514"
---
# <a name="troubleshoot-apache-spark-by-using-azure-hdinsight"></a>Устранение неполадок в Apache Spark с помощью Azure HDInsight

Узнайте о главных проблемах и их разрешениях при работе с полезными данными Apache Spark в [Apache Ambari](https://ambari.apache.org/).

## <a name="how-do-i-configure-an-apache-spark-application-by-using-apache-ambari-on-clusters"></a>Как настроить приложение Apache Spark с помощью Apache Ambari в кластерах?

Можно настроить значения конфигурации Spark, чтобы избежать исключения Apache Spark приложения `OutofMemoryError` . Следующие шаги демонстрируют значения конфигурации Spark по умолчанию в Azure HDInsight.

1. Войдите в Ambari `https://CLUSTERNAME.azurehdidnsight.net` с учетными данными кластера. На начальном экране отображается панель мониторинга обзора. Между HDInsight 3,6 и 4,0 существуют небольшие косметические различия.

1. Перейдите к **Spark2**  >  **configs**.

    :::image type="content" source="./media/apache-troubleshoot-spark/apache-spark-ambari-config2.png" alt-text="Выбор вкладки &quot;Конфигурации&quot;" border="true":::

1. В списке конфигураций выберите и разверните пользовательский параметр **-spark2 — значения по умолчанию**.

1. Найдите параметр значения, который необходимо настроить, например **spark.executor.memory**. В этом случае значение **9728m** слишком велико.

    :::image type="content" source="./media/apache-troubleshoot-spark/apache-spark-ambari-config4.png" alt-text="Выбор конфигурации custom-spark-defaults" border="true":::

1. Задайте для этого параметра рекомендуемое значение. Рекомендуется использовать значение **2048m**.

1. Сохраните это значение, а затем сохраните конфигурацию. Щелкните **Сохранить**.

    :::image type="content" source="./media/apache-troubleshoot-spark/apache-spark-ambari-config6a.png" alt-text="Изменение значения на 2048m" border="true":::

    Запишите примечание об изменениях конфигурации, а затем нажмите кнопку **Сохранить**.

    :::image type="content" source="./media/apache-troubleshoot-spark/apache-spark-ambari-config6c.png" alt-text="Примечание об изменениях" border="true":::

    Если нужно будет пересмотреть какую-либо конфигурацию, вы получите оповещение. Проверьте элементы, а затем нажмите кнопку **Proceed Anyway** (Продолжить).

    :::image type="content" source="./media/apache-troubleshoot-spark/apache-spark-ambari-config6b.png" alt-text="Нажатие кнопки &quot;Proceed Anyway&quot; (Продолжить)" border="true":::

1. При каждом сохранении конфигурации вам будет предложено перезапустить службу. Нажмите кнопку **Перезапустить**.

    :::image type="content" source="./media/apache-troubleshoot-spark/apache-spark-ambari-config7a.png" alt-text="Выберите перезапустить" border="true":::

    Подтвердите перезапуск.

    :::image type="content" source="./media/apache-troubleshoot-spark/apache-spark-ambari-config7b.png" alt-text="Нажатие кнопки &quot;Confirm Restart All&quot; (Подтвердить перезапуск всех)" border="true":::

    Вы можете просмотреть запущенные процессы.

    :::image type="content" source="./media/apache-troubleshoot-spark/apache-spark-ambari-config7c.png" alt-text="Просмотр запущенных процессов" border="true":::

1. Вы можете добавить конфигурации. В списке конфигураций выберите **Custom-spark2-defaults**, а затем щелкните **Добавить свойство**.

    :::image type="content" source="./media/apache-troubleshoot-spark/apache-spark-ambari-config8.png" alt-text="Выбор &quot;Добавить свойство&quot;" border="true":::

1. Определите новое свойство. Вы можете определить отдельное свойство с помощью диалогового окна для определенных параметров, например тип данных. Или вы можете определить несколько свойств с одним определением на строку.

    В этом примере свойство **spark.driver.memory** определяется со значением **4 ГБ**.

    :::image type="content" source="./media/apache-troubleshoot-spark/apache-spark-ambari-config9.png" alt-text="Определение нового свойства" border="true":::

1. Сохраните конфигурацию и перезапустите службу, как описано на шагах 6 и 7.

Эти изменения применяются на уровне кластера, но их можно переопределить при отправке задания Spark.

## <a name="how-do-i-configure-an-apache-spark-application-by-using-a-jupyter-notebook-on-clusters"></a>Разделы справки настроить Apache Spark приложение с помощью Jupyter Notebook в кластерах?

В первой ячейке Jupyter Notebook после директивы **%% Configure** Укажите конфигурации Spark в допустимом формате JSON. При необходимости измените фактические значения:

:::image type="content" source="./media/apache-troubleshoot-spark/add-configuration-cell.png" alt-text="Добавление конфигурации" border="true":::

## <a name="how-do-i-configure-an-apache-spark-application-by-using-apache-livy-on-clusters"></a>Как настроить приложение Apache Spark с помощью Apache Livy в кластерах?

Отправьте приложение Spark в Livy с помощью клиента REST, например cURL. Используйте команду, аналогичную приведенной ниже. При необходимости измените фактические значения:

```apache
curl -k --user 'username:password' -v -H 'Content-Type: application/json' -X POST -d '{ "file":"wasb://container@storageaccountname.blob.core.windows.net/example/jars/sparkapplication.jar", "className":"com.microsoft.spark.application", "numExecutors":4, "executorMemory":"4g", "executorCores":2, "driverMemory":"8g", "driverCores":4}'  
```

## <a name="how-do-i-configure-an-apache-spark-application-by-using-spark-submit-on-clusters"></a>Как настроить приложение Apache Spark с помощью spark-submit в кластерах?

Запустите оболочку Spark с помощью команды, аналогичной приведенной ниже. При необходимости измените фактические значения конфигураций:

```apache
spark-submit --master yarn-cluster --class com.microsoft.spark.application --num-executors 4 --executor-memory 4g --executor-cores 2 --driver-memory 8g --driver-cores 4 /home/user/spark/sparkapplication.jar
```

### <a name="additional-reading"></a>Дополнительные материалы

[Отправка заданий Apache Spark в кластерах HDInsight](https://web.archive.org/web/20190112152841/https://blogs.msdn.microsoft.com/azuredatalake/2017/01/06/spark-job-submission-on-hdinsight-101/)

## <a name="next-steps"></a>Дальнейшие действия

Если вы не видите своего варианта проблемы или вам не удается ее устранить, дополнительные сведения можно получить, посетив один из следующих каналов.

* [Общие сведения об управлении памятью Spark](https://spark.apache.org/docs/latest/tuning.html#memory-management-overview).

* [Отладка приложения Spark в кластерах HDInsight](/archive/blogs/azuredatalake/spark-debugging-101).

* Получите ответы специалистов Azure на [сайте поддержки сообщества пользователей Azure](https://azure.microsoft.com/support/community/).

* Подпишитесь на [@AzureSupport](https://twitter.com/azuresupport) — официальный канал Microsoft Azure для работы с клиентами. Вступайте в сообщество Azure для получения нужных ресурсов: ответов, поддержки и советов экспертов.

* Если вам нужна дополнительная помощь, отправьте запрос в службу поддержки на [портале Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Выберите **Поддержка** в строке меню или откройте центр **Справка и поддержка**. Дополнительные сведения см. в статье [Создание запроса на поддержку Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). Доступ к управлению подписками и поддержкой выставления счетов уже включен в вашу подписку Microsoft Azure, а техническая поддержка предоставляется в рамках одного из [планов Службы поддержки Azure](https://azure.microsoft.com/support/plans/).