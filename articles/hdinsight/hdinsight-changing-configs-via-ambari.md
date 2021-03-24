---
title: Оптимизация кластеров с помощью Apache Ambari в Azure HDInsight
description: Используйте веб-интерфейс Apache Ambari для настройки и оптимизации кластеров Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 05/04/2020
ms.openlocfilehash: 2146ccb0c4d7f263c3e1a69db9b172649fcd25ea
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104863502"
---
# <a name="optimize-clusters-with-apache-ambari-in-azure-hdinsight"></a>Оптимизация кластеров с помощью Apache Ambari в Azure HDInsight

HDInsight предоставляет кластеры Apache Hadoop для приложений крупномасштабной обработки данных. Управление, мониторинг и оптимизация этих сложных кластеров с несколькими узлами может быть непростой задачей. Apache Ambari — это веб-интерфейс для мониторинга кластеров HDInsight под управлением Linux и управления ими.

Основные сведения о пользовательском веб-интерфейсе Ambari см. в статье [Управление кластерами HDInsight с помощью веб-интерфейса Ambari](hdinsight-hadoop-manage-ambari.md).

Войдите в Ambari по адресу `https://CLUSTERNAME.azurehdidnsight.net` с помощью учетных данных кластера. Начальный экран отображает панель мониторинга с общими сведениями.

:::image type="content" source="./media/hdinsight-changing-configs-via-ambari/apache-ambari-dashboard.png" alt-text="Отображаемая панель мониторинга пользователя Apache Ambari":::

Пользовательский веб-интерфейс Ambari используется для управления узлами, службами, оповещениями, конфигурациями и представлениями. Ambari нельзя использовать для создания кластера HDInsight или обновления служб. Также не может управлять стеками и версиями, высписанием или перекомиссионными узлами или добавлением служб в кластер.

## <a name="manage-your-clusters-configuration"></a>Управление конфигурацией кластера

Параметры конфигурации помогают настроить определенную службу. Чтобы изменить параметры конфигурации службы, выберите ее на боковой панели **службы** (слева). Затем перейдите на вкладку **configs (конфигурации** ) на странице сведений о службе.

:::image type="content" source="./media/hdinsight-changing-configs-via-ambari/ambari-services-sidebar.png" alt-text="Боковая панель служб Apache Ambari":::

## <a name="modify-namenode-java-heap-size"></a>Изменение размера кучи NameNode Java

Размер кучи NameNode Java зависит от многих факторов, таких как нагрузка на кластер. Кроме того, число файлов и число блоков. По умолчанию ее размер составляет 1 ГБ, что подходит для большинства кластеров, хотя для некоторых рабочих нагрузок может потребоваться больше или меньше памяти.

Вот как можно изменить размер кучи NameNode Java.

1. Выберите **HDFS** на боковой панели "Services" (Службы) и перейдите на вкладку **Configs** (Конфигурации).

    :::image type="content" source="./media/hdinsight-changing-configs-via-ambari/ambari-apache-hdfs-config.png" alt-text="Конфигурация Apache Ambari HDFS":::

1. Найдите параметр **NameNode Java heap size** (Размер кучи NameNode Java). Можно также использовать текстовое поле **фильтра**, чтобы ввести и найти конкретное значение. Щелкните значок **пера** рядом с именем параметра.

    :::image type="content" source="./media/hdinsight-changing-configs-via-ambari/ambari-java-heap-size.png" alt-text="Размер кучи Apache Ambari NameNode Java":::

1. Введите новое значение в текстовом поле и нажмите клавишу **ВВОД**, чтобы сохранить изменения.

    :::image type="content" source="./media/hdinsight-changing-configs-via-ambari/java-heap-size-edit1.png" alt-text="Ambari Edit NameNode Size1 куча Java":::

1. Размер кучи Java NameNode изменяется на 1 ГБ с 2 ГБ.

    :::image type="content" source="./media/hdinsight-changing-configs-via-ambari/java-heap-size-edited.png" alt-text="Измененная куча NameNode Java size2":::

1. Сохраните изменения, нажав зеленую кнопку **Save** (Сохранить) в верхней части экрана конфигурации.

    :::image type="content" source="./media/hdinsight-changing-configs-via-ambari/ambari-save-changes1.png" alt-text="&quot;Apache Ambari Save Configurations&quot;":::

## <a name="next-steps"></a>Дальнейшие действия

* [Управление кластерами HDInsight с помощью веб-интерфейса Ambari](hdinsight-hadoop-manage-ambari.md)
* [Управление кластерами HDInsight с помощью REST API Ambari](hdinsight-hadoop-manage-ambari-rest-api.md)
* [Оптимизация Apache HBase](./optimize-hbase-ambari.md)
* [Оптимизация Apache Hive](./optimize-hive-ambari.md)
* [Optimize Apache Pig](./optimize-pig-ambari.md)
