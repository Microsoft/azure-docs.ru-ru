---
title: Перенос рабочих нагрузок Apache Kafka в Azure HDInsight 4,0
description: Узнайте, как перенести рабочие нагрузки Apache Kafka в HDInsight 3,6 в HDInsight 4,0.
ms.service: hdinsight
ms.topic: conceptual
ms.date: 12/18/2019
ms.openlocfilehash: e15ebb13aee0e5dd814688ae77edaded667d54ac
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104864131"
---
# <a name="migrate-apache-kafka-workloads-to-azure-hdinsight-40"></a>Перенос рабочих нагрузок Apache Kafka в Azure HDInsight 4,0

Azure HDInsight 4,0 предлагает новейшие компоненты с открытым исходным кодом, значительно повышая производительность, возможности подключения и безопасности. В этом документе объясняется, как перенести рабочие нагрузки Apache Kafka в HDInsight 3,6 в HDInsight 4,0. После переноса рабочих нагрузок в HDInsight 4,0 можно использовать многие из новых функций, которые недоступны в HDInsight 3,6.

## <a name="hdinsight-36-kafka-migration-paths"></a>Пути миграции HDInsight 3,6 Kafka

HDInsight 3,6 поддерживает две версии Kafka: 1.0.0 и 1.1.0. HDInsight 4,0 поддерживает версии 1.1.0 и 2.1.0. В зависимости от версии Kafka и версии HDInsight, которую вы хотите запустить, существует несколько поддерживаемых путей миграции. Эти пути описаны ниже и показаны на следующей схеме.

* **Запуск Kafka и hdinsight в последних версиях (рекомендуется)**: миграция приложения hdinsight 3,6 и Kafka 1.0.0 или 1.1.0 в hdinsight 4,0 с Kafka 2.1.0 (пути D и E ниже).
* **Запустите HDInsight в последней версии, но Kafka только в более поздней версии**: перенос приложения hdinsight 3,6 и Kafka 1.0.0 в hdinsight 4,0 с Kafka 1.1.0 (путь б ниже).
* **Запустите HDInsight в последней версии, оставьте версию Kafka**: миграция приложения hdinsight 3,6 и Kafka 1.1.0 в hdinsight 4,0 с Kafka 1.1.0 (путь C ниже).
* **Запустите Kafka в более новой версии, оставьте версию hdinsight**: перенесите приложение Kafka 1.0.0 на 1.1.0 и оставайтесь на HDInsight 3,6 (путь а ниже). Обратите внимание, что этот параметр по-прежнему потребуется для развертывания нового кластера. Обновление версии Kafka в существующем кластере не поддерживается. После создания кластера с требуемой версией перенесите клиенты Kafka для использования нового кластера.

:::image type="content" source="./media/upgrade-threesix-to-four/apache-kafka-upgrade-path.png" alt-text="Варианты обновления для Apache Kafka на 3,6" border="false":::

## <a name="apache-kafka-versions"></a>Версии Apache Kafka

### <a name="kafka-110"></a>Kafka 1.1.0
  
При миграции с Kafka 1.0.0 на 1.1.0 можно воспользоваться следующими новыми функциями.

* Улучшения в управлении скоростью работы контроллера Kafka, поэтому вы можете перезапустить брокеры и быстро устранить неполадки. 
* Усовершенствования в [логике фетчрекуестс](https://issues.apache.org/jira/browse/KAFKA-6254) , позволяющие иметь больше разделов (и, следовательно, больше разделов) в кластере. 
* Kafka Connect поддерживает [заголовки записей](https://issues.apache.org/jira/browse/KAFKA-5142) и [регулярные выражения](https://issues.apache.org/jira/browse/KAFKA-3073) для разделов. 

Полный список обновлений см. в разделе [Apache Kafka 1,1. заметки о выпуске](https://archive.apache.org/dist/kafka/1.1.0/RELEASE_NOTES.html).

### <a name="apache-kafka-210"></a>Apache Kafka 2.1.0

При переходе на Kafka 2,1 можно воспользоваться следующими возможностями.

* Улучшение устойчивости брокера из-за улучшенного протокола репликации.
* Новые функции в API Кафкаадминклиент.
* Настраиваемое управление квотами.
* Поддержка сжатия Зстандард.

Полный список обновлений см. в статье [заметки о выпуске Apache Kafka 2,0](https://archive.apache.org/dist/kafka/2.0.0/RELEASE_NOTES.html) и [Apache Kafka 2,1](https://archive.apache.org/dist/kafka/2.1.0/RELEASE_NOTES.html).

## <a name="kafka-client-compatibility"></a>Kafka совместимость клиента

Новые брокеры Kafka поддерживают более старые клиенты. [Пропустить-35-получение версии протокола](https://cwiki.apache.org/confluence/display/KAFKA/KIP-35+-+Retrieving+protocol+version) предоставила механизм для динамического определения функциональных возможностей брокера Kafka и [пропустить-97: Улучшенная политика совместимости RPC для клиента Kafka](https://cwiki.apache.org/confluence/display/KAFKA/KIP-97%3A+Improved+Kafka+Client+RPC+Compatibility+Policy) предоставила новую политику совместимости и гарантирует наличие клиента Java. Ранее клиенту Kafka пришлось взаимодействовать с брокером той же или более новой версии. Теперь новые версии клиентов Java и других клиентов, которые поддерживают пропустить-35, такие как, `librdkafka` могут возвращаться к старым типам запросов или выдавать соответствующие ошибки, если функциональность недоступна.

:::image type="content" source="./media/upgrade-threesix-to-four/apache-kafka-client-compatibility.png" alt-text="Обновление совместимости клиента Kafka" border="false":::

Обратите внимание, что это не означает, что клиент поддерживает более старые брокеры.  Дополнительные сведения см. в разделе [Таблица совместимости](https://cwiki.apache.org/confluence/display/KAFKA/Compatibility+Matrix).

## <a name="general-migration-process"></a>Общий процесс миграции

В следующих руководствах по миграции предполагается, что кластер Apache Kafka 1.0.0 или 1.1.0, развернутый в HDInsight 3,6, находится в одной виртуальной сети. У существующего брокера есть несколько разделов и активно используемых производителями и потребителями.

:::image type="content" source="./media/upgrade-threesix-to-four/apache-kafka-presumed-environment.png" alt-text="Текущая Kafka предположительно среда" border="false":::

Чтобы завершить миграцию, выполните следующие действия.

1. **Разверните новый кластер и клиенты HDInsight 4,0 для тестирования.** Разверните новый кластер HDInsight 4,0 Kafka. Если можно выбрать несколько версий кластера Kafka, рекомендуется выбрать последнюю версию. После развертывания задайте необходимые параметры и создайте раздел с тем же именем, что и у существующей среды. Кроме того, при необходимости задайте шифрование TLS и присвойте ему ключ (BYOK). Затем проверьте, правильно ли работает новый кластер.

    :::image type="content" source="./media/upgrade-threesix-to-four/deploy-new-hdinsight-clusters.png" alt-text="Развертывание новых кластеров HDInsight 4,0" border="false":::

1. **Переключите кластер для приложения производителя и подождите, пока все данные очереди будут использоваться текущими потребителями.** Когда новый кластер HDInsight 4,0 Kafka будет готов, переключите существующий приемник на новый кластер. Оставьте его так, чтобы в существующем приложении-потребителе не использовались все данные из существующего кластера.

    :::image type="content" source="./media/upgrade-threesix-to-four/switch-cluster-producer-app.png" alt-text="Переключение кластера для приложения производителя" border="false":::

1. **Переключите кластер в приложении-потребителе.** Убедившись, что существующее приложение-потребитель завершило использование всех данных из существующего кластера, переключитесь на подключение к новому кластеру.

    :::image type="content" source="./media/upgrade-threesix-to-four/switch-cluster-consumer-app.png" alt-text="Переключение кластера на приложение-потребитель" border="false":::

1. **При необходимости удалите старый кластер и тестовые приложения.** После завершения и правильной работы коммутатора удалите старый кластер HDInsight 3,6 Kafka, а также поставщиков и потребителей, которые использовались в тесте по мере необходимости.

## <a name="next-steps"></a>Дальнейшие действия

* [Оптимизация производительности для кластеров Apache Kafka HDInsight](apache-kafka-performance-tuning.md)
* [Краткое руководство. Создание кластера Apache Kafka в Azure HDInsight с помощью портала Azure](apache-kafka-get-started.md)
