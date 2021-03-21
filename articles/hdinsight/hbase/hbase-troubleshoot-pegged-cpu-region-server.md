---
title: Фиксированный ЦП в кластере Apache HBase — Azure HDInsight
description: Устранение неполадок фиксированного ЦП на сервере регионов в кластере Apache HBase в Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 08/01/2019
ms.openlocfilehash: 7d0cf139f06bb296b486d2932d95b53fc1167a5d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98937016"
---
# <a name="scenario-pegged-cpu-on-region-server-in-apache-hbase-cluster-in-azure-hdinsight"></a>Сценарий: фиксированный ЦП на сервере региона в кластере Apache HBase в Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы решения проблем при взаимодействии с кластерами Azure HDInsight.

## <a name="issue"></a>Проблема

Процесс сервера Apache HBase в регионе начинается с близкого к 200% ЦП, что приводит к срабатыванию предупреждений в HBase Master процессе и кластере для неполной работы.

## <a name="cause"></a>Причина

Если вы используете кластер HBase версии 3.4, возможно, возникла потенциальная ошибка, вызванная обновлением JDK до Version 1.7.0 _151. Симптомом, который мы видим, это начало процесса сервера региона, который занимает около 200% ЦП (для проверки выполнения `top` команды; если существует процесс, который занимает около 200% ресурсов ЦП, получите идентификатор процесса и подтвердите, что он является процессом сервера региона, выполнив команду `ps -aux | grep` ).

## <a name="resolution"></a>Решение

1. Установите JDK 1,8 на всех узлах кластера, как показано ниже.

    * Выполните действие скрипта `https://raw.githubusercontent.com/Azure/hbase-utils/master/scripts/upgradetojdk18allnodes.sh` . Не забудьте выбрать параметр для запуска на всех узлах.

    * Кроме того, можно войти в каждый отдельный узел и выполнить команду `sudo add-apt-repository ppa:openjdk-r/ppa -y && sudo apt-get -y update && sudo apt-get install -y openjdk-8-jdk` .

1. Перейдите в пользовательский интерфейс Ambari `https://<clusterdnsname>.azurehdinsight.net` .

1. Перейдите к **HBase->configs — >Advanced->Advanced** `hbase-env configs` и измените значение переменной `JAVA_HOME` на `export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64` . Сохраните изменения конфигурации.

1. [Необязательно, но рекомендуется] [Очистить все таблицы в кластере](/archive/blogs/azuredatalake/hdinsight-hbase-how-to-improve-hbase-cluster-restart-time-by-flushing-tables).

1. В пользовательском интерфейсе Ambari перезапустите все службы HBase, требующие перезагрузки.

1. В зависимости от данных в кластере для достижения стабильного состояния кластера может потребоваться от нескольких минут до часа. Способ подтверждения кластера в стабильном состоянии заключается в проверке пользовательского интерфейса HMaster (все серверы регионов должны быть активны) из Ambari (обновление) или из головного узла запустите оболочку HBase, а затем выполните команду status.

Чтобы убедиться, что обновление прошло успешно, убедитесь, что соответствующие процессы HBase запущены с использованием соответствующей версии Java — например, для сервера региона:

```
ps -aux | grep regionserver, and verify the version like '''/usr/lib/jvm/java-8-openjdk-amd64/bin/java
```

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]