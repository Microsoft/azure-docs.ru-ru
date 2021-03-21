---
title: Оптимизация кластеров с помощью Apache Ambari в Azure HDInsight
description: Используйте веб-интерфейс Apache Ambari для настройки и оптимизации кластеров Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 05/04/2020
ms.openlocfilehash: 14268e58296554cedc9488e48b41719421797a57
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98933284"
---
# <a name="optimize-clusters-with-apache-ambari-in-azure-hdinsight"></a>Оптимизация кластеров с помощью Apache Ambari в Azure HDInsight

HDInsight предоставляет кластеры Apache Hadoop для приложений крупномасштабной обработки данных. Управление, мониторинг и оптимизация этих сложных кластеров с несколькими узлами может быть непростой задачей. Apache Ambari — это веб-интерфейс для мониторинга кластеров HDInsight под управлением Linux и управления ими.

Основные сведения о пользовательском веб-интерфейсе Ambari см. в статье [Управление кластерами HDInsight с помощью веб-интерфейса Ambari](hdinsight-hadoop-manage-ambari.md).

Войдите в Ambari по адресу `https://CLUSTERNAME.azurehdidnsight.net` с помощью учетных данных кластера. Начальный экран отображает панель мониторинга с общими сведениями.

![Отображаемая панель мониторинга пользователя Apache Ambari](./media/hdinsight-changing-configs-via-ambari/apache-ambari-dashboard.png)

Пользовательский веб-интерфейс Ambari используется для управления узлами, службами, оповещениями, конфигурациями и представлениями. Ambari нельзя использовать для создания кластера HDInsight или обновления служб. Также не может управлять стеками и версиями, высписанием или перекомиссионными узлами или добавлением служб в кластер.

## <a name="manage-your-clusters-configuration"></a>Управление конфигурацией кластера

Параметры конфигурации помогают настроить определенную службу. Чтобы изменить параметры конфигурации службы, выберите ее на боковой панели **службы** (слева). Затем перейдите на вкладку **configs (конфигурации** ) на странице сведений о службе.

![Боковая панель служб Apache Ambari](./media/hdinsight-changing-configs-via-ambari/ambari-services-sidebar.png)

## <a name="modify-namenode-java-heap-size"></a>Изменение размера кучи NameNode Java

Размер кучи NameNode Java зависит от многих факторов, таких как нагрузка на кластер. Кроме того, число файлов и число блоков. По умолчанию ее размер составляет 1 ГБ, что подходит для большинства кластеров, хотя для некоторых рабочих нагрузок может потребоваться больше или меньше памяти.

Вот как можно изменить размер кучи NameNode Java.

1. Выберите **HDFS** на боковой панели "Services" (Службы) и перейдите на вкладку **Configs** (Конфигурации).

    ![Конфигурация Apache Ambari HDFS](./media/hdinsight-changing-configs-via-ambari/ambari-apache-hdfs-config.png)

1. Найдите параметр **NameNode Java heap size** (Размер кучи NameNode Java). Можно также использовать текстовое поле **фильтра**, чтобы ввести и найти конкретное значение. Щелкните значок **пера** рядом с именем параметра.

    ![Размер кучи Apache Ambari NameNode Java](./media/hdinsight-changing-configs-via-ambari/ambari-java-heap-size.png)

1. Введите новое значение в текстовом поле и нажмите клавишу **ВВОД**, чтобы сохранить изменения.

    ![Ambari Edit NameNode Size1 куча Java](./media/hdinsight-changing-configs-via-ambari/java-heap-size-edit1.png)

1. Размер кучи Java NameNode изменяется на 1 ГБ с 2 ГБ.

    ![Измененная куча NameNode Java size2](./media/hdinsight-changing-configs-via-ambari/java-heap-size-edited.png)

1. Сохраните изменения, нажав зеленую кнопку **Save** (Сохранить) в верхней части экрана конфигурации.

    !["Apache Ambari Save Configurations"](./media/hdinsight-changing-configs-via-ambari/ambari-save-changes1.png)

## <a name="next-steps"></a>Дальнейшие действия

* [Управление кластерами HDInsight с помощью веб-интерфейса Ambari](hdinsight-hadoop-manage-ambari.md)
* [Управление кластерами HDInsight с помощью REST API Ambari](hdinsight-hadoop-manage-ambari-rest-api.md)
* [Оптимизация Apache HBase](./optimize-hbase-ambari.md)
* [Оптимизация Apache Hive](./optimize-hive-ambari.md)
* [Optimize Apache Pig](./optimize-pig-ambari.md)
