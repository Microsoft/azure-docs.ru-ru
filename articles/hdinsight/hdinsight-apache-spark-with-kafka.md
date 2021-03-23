---
title: Потоковая передача Apache Spark c Apache Kafka в Azure HDInsight
description: Узнайте об использовании Apache Spark для двунаправленного потокового обмена данными с Apache Kafka с помощью DStreams. В этом примере выполняется потоковая передача данных с помощью Jupyter Notebook из Spark в HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 11/21/2019
ms.openlocfilehash: 6ef11e9c7907f57b3b8de0a042e1035bce638cf4
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104863281"
---
# <a name="apache-spark-streaming-dstream-example-with-apache-kafka-on-hdinsight"></a>Пример потоковой передачи Apache Spark (DStream) с использованием Apache Kafka в HDInsight

Узнайте, как использовать [Apache Spark](https://spark.apache.org/) для потоковой передачи данных в [Apache Kafka](https://kafka.apache.org/) в HDInsight и из него с помощью [DStreams](https://spark.apache.org/docs/latest/api/java/org/apache/spark/streaming/dstream/DStream.html). В этом примере используется средство [Jupyter Notebook](https://jupyter.org/), которое выполняется на кластере Spark.

> [!NOTE]  
> Вы узнаете, как создать группу ресурсов Azure, которая содержит кластеры Spark и Kafka в HDInsight. Оба этих кластера находятся в виртуальной сети Azure, что позволяет кластеру Spark напрямую обмениваться данными с кластером Kafka.
>
> Выполнив инструкции, не забудьте удалить кластеры, чтобы избежать ненужных расходов.

> [!IMPORTANT]  
> В этом примере используется DStreams (старая технология потоковой передачи Spark). Пример, использующий новые функции потоковой передачи Spark, см. в статье [Использование структурированной потоковой передачи Spark с Apache Kafka в HDInsight](hdinsight-apache-kafka-spark-structured-streaming.md).

## <a name="create-the-clusters"></a>Создание кластеров

Apache Kafka в HDInsight не предоставляет доступ к брокерам Kafka через общедоступный сегмент Интернета. Все объекты, обращающиеся к Kafka, должны находиться в той же виртуальной сети Azure, что и узлы в кластере Kafka. В этом примере кластеры Kafka и Spark расположены в виртуальной сети Azure. На следующей схеме показано, как взаимодействуют кластеры.

:::image type="content" source="./media/hdinsight-apache-spark-with-kafka/apache-spark-kafka-vnet.png" alt-text="Схема кластеров Spark и Kafka в виртуальной сети Azure" border="false":::

> [!NOTE]  
> Хотя само решение Kafka ограничено связью в пределах виртуальной сети, другие службы в кластере, например SSH и Ambari, доступны через Интернет. Дополнительные сведения об общих портах, доступных в HDInsight, см. в статье [Порты и универсальные коды ресурсов (URI), используемые кластерами HDInsight](hdinsight-hadoop-port-settings-for-services.md).

Хотя виртуальную сеть Azure, а также кластеры Kafka и Spark можно создать вручную, проще использовать шаблон Azure Resource Manager. Выполните следующие действия, чтобы развернуть виртуальную сеть Azure, а также кластеры Kafka и Spark в подписке Azure.

1. Нажмите эту кнопку, чтобы войти в Azure и открыть шаблон на портале Azure.

   <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fhditutorialdata.blob.core.windows.net%2Farmtemplates%2Fcreate-linux-based-kafka-spark-cluster-in-vnet-v4.1.json" target="_blank"><img src="./media/hdinsight-apache-spark-with-kafka/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

   Шаблон Azure Resource Manager доступен по адресу **https://hditutorialdata.blob.core.windows.net/armtemplates/create-linux-based-kafka-spark-cluster-in-vnet-v4.1.json** .

   > [!WARNING]
   > Чтобы обеспечить доступность Kafka в HDInsight, кластер должен содержать не менее трех рабочих узлов. Этот шаблон создает кластер Kafka, содержащий три рабочих узла.

   Этот шаблон создает кластер HDInsight 3.6 для Kafka и Spark.

1. Используйте следующие сведения, чтобы заполнить раздел **Настраиваемое развертывание**:

   |Свойство |Значение |
   |---|---|
   |Группа ресурсов|Создайте новую группу или выберите существующую.|
   |Расположение|Выберите близкое к вам географическое расположение.|
   |Имя базового кластера|Это значение будет использоваться в качестве базового имени для кластеров Spark и Kafka. Например, если ввести **hdistreaming**, будет создан кластер Spark с именем __spark-hdistreaming__ и кластер Kafka с именем **kafka-hdistreaming**.|
   |Имя пользователя для входа в кластер|Имя администратора для кластеров Spark и Kafka.|
   |Пароль для входа в кластер|Пароль администратора для кластеров Spark и Kafka.|
   |Имя пользователя SSH|Создаваемый пользователь SSH для кластеров Spark и Kafka.|
   |Пароль SSH|Пароль пользователя SSH для кластеров Spark и Kafka.|

   :::image type="content" source="./media/hdinsight-apache-spark-with-kafka/hdinsight-parameters.png" alt-text="Параметры настраиваемого развертывания HDInsight":::

1. Прочтите **условия использования** и установите флажок **Я принимаю указанные выше условия**.

1. Наконец, щелкните **Приобрести**. Процесс создания кластеров занимает около 20 минут.

Когда ресурсы будут созданы, отобразится страница со сводными сведениями.

:::image type="content" source="./media/hdinsight-apache-spark-with-kafka/hdinsight-group-blade.png" alt-text="Сводные сведения о группе ресурсов для виртуальной сети и кластеров":::

> [!IMPORTANT]  
> Обратите внимание, что кластерам HDInsight присвоены имена **spark-BASENAME** и **kafka-BASENAME**, где BASENAME — имя, указанное в шаблоне. Эти имена будут использоваться позже при подключении к кластерам.

## <a name="use-the-notebooks"></a>Использование записных книжек

Код для примера, описанного в этом документе, доступен по адресу [https://github.com/Azure-Samples/hdinsight-spark-scala-kafka](https://github.com/Azure-Samples/hdinsight-spark-scala-kafka) .

## <a name="delete-the-cluster"></a>Удаление кластера

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

Выполнив описанные здесь инструкции, вы создадите два кластера в одной группе ресурсов Azure. Следовательно, вы можете удалить эту группу ресурсов на портале Azure. При этом будут удалены все созданные в рамках этого руководства и используемые в кластерах ресурсы, виртуальная сеть Azure и учетная запись хранения.

## <a name="next-steps"></a>Дальнейшие действия

В этом примере описано, как использовать Spark для чтения и записи данных в Kafka. Другие материалы, посвященные работе с Kafka, доступны по следующим ссылкам:

* [Get started with Apache Kafka on HDInsight (preview)](kafka/apache-kafka-get-started.md) (Приступая к работе с Apache Kafka в HDInsight (предварительная версия))
* [Репликация разделов Apache Kafka с помощью Kafka в HDInsight и MirrorMaker](kafka/apache-kafka-mirroring.md)
* [Использование Apache Storm с Apache Kafka в HDInsight](hdinsight-apache-storm-with-kafka.md)
