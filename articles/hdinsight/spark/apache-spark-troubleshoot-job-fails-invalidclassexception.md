---
title: Инвалидклассексцептион ошибка из Apache Spark — Azure HDInsight
description: Сбой задания Apache Spark с Инвалидклассексцептион, несоответствие версий класса в Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 07/29/2019
ms.openlocfilehash: 0b0889ac1e71ce33406e89ead62370a0c0168763
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98929253"
---
# <a name="apache-spark-job-fails-with-invalidclassexception-class-version-mismatch-in-azure-hdinsight"></a>Сбой задания Apache Spark с Инвалидклассексцептион, несоответствие версий класса в Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы исправления проблем, возникающих при использовании компонентов Apache Spark в кластерах Azure HDInsight.

## <a name="issue"></a>Проблема

Вы пытаетесь создать задание Apache Spark в кластере Spark 2. x. Она завершается ошибкой, как показано ниже.

```
18/09/18 09:32:26 WARN TaskSetManager: Lost task 0.0 in stage 1.0 (TID 1, wn7-dev-co.2zyfbddadfih0xdq0cdja4g.ax.internal.cloudapp.net, executor 4): java.io.InvalidClassException:
org.apache.commons.lang3.time.FastDateFormat; local class incompatible: stream classdesc serialVersionUID = 2, local class serialVersionUID = 1
        at java.io.ObjectStreamClass.initNonProxy(ObjectStreamClass.java:699)
        at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:1885)
        at java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1751)
        at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2042)
        at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1573)
```

## <a name="cause"></a>Причина

Эта ошибка может быть вызвана добавлением дополнительного JAR `spark.yarn.jars` -файла в конфигурацию, в частности, затененного JAR, включающего в себя другую версию `commons-lang3` пакета и представляющего несоответствие класса. По умолчанию в Spark 2.1/2/3 используется версия 3,5 `commons-lang3` .

> [!TIP]
> Чтобы затениить библиотеку, необходимо разместить ее содержимое в собственном JAR-файле, изменив его пакет. Это отличается от упаковки библиотеки, которая помещает библиотеку в собственный JAR-файл без переупаковки.

## <a name="resolution"></a>Решение

Удалите JAR-файл или выполните повторную компиляцию настроенного JAR (Азурелогаппендер) и используйте [подключаемый модуль Maven-Shader](https://maven.apache.org/plugins/maven-shade-plugin/examples/class-relocation.html) для перемещения классов.

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]