---
title: Apache Spark приложение потоковой передачи остановлено через 24 дня в Azure HDInsight
description: Приложение потоковой передачи Apache Spark останавливается после выполнения в течение 24 дней, и в файлах журнала ошибок нет.
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 07/29/2019
ms.openlocfilehash: b702cbf915e4991df4c202564677ea7e0a02f9c4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98929456"
---
# <a name="scenario-apache-spark-streaming-application-stops-after-executing-for-24-days-in-azure-hdinsight"></a>Сценарий: Apache Spark приложение потоковой передачи останавливается после выполнения в течение 24 дней в Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы исправления проблем, возникающих при использовании компонентов Apache Spark в кластерах Azure HDInsight.

## <a name="issue"></a>Проблема

Приложение потоковой передачи Apache Spark останавливается после выполнения в течение 24 дней, и в файлах журнала ошибок нет.

## <a name="cause"></a>Причина

`livy.server.session.timeout`Значение определяет, сколько времени Apache Livy должен ожидать завершения сеанса. Когда длина сеанса достигнет `session.timeout` значения, сеанс Livy и приложение будут автоматически уничтожены.

## <a name="resolution"></a>Решение

Для длительных заданий увеличьте значение `livy.server.session.timeout` с помощью пользовательского интерфейса Ambari. Вы можете получить доступ к конфигурации Livy из пользовательского интерфейса Ambari, используя URL-адрес `https://<yourclustername>.azurehdinsight.net/#/main/services/LIVY/configs` .

Замените `<yourclustername>` именем кластера HDInsight, как показано на портале.

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]