---
title: Использование Apache Ambari в Azure HDInsight
description: Обсуждение использования Apache Ambari в Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 01/12/2021
ms.openlocfilehash: ff83e559919a836208faae4eae4a5f992534b6cb
ms.sourcegitcommit: 431bf5709b433bb12ab1f2e591f1f61f6d87f66c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/12/2021
ms.locfileid: "98134166"
---
# <a name="apache-ambari-usage-in-azure-hdinsight"></a>Использование Apache Ambari в Azure HDInsight

HDInsight использует Apache Ambari для развертывания кластера и управления им. Агенты Ambari запускаются на каждом узле (головного узла, Рабочий узел, Zookeeper и граничном узле, если он существует). Сервер Ambari работает только в головного узла (hn0 или HN1). Одновременно должен выполняться только один экземпляр сервера Ambari. Это управляется контроллером отработки отказа HDInsight. Когда один из головных узлах не работает для перезагрузки или обслуживания, остальные головного узла становятся активными, и Ambari Server на второй головного узла будет запущен.

Все настройки кластера должны выполняться через [Пользовательский интерфейс Ambari](./hdinsight-hadoop-manage-ambari.md), любое локальное изменение будет перезаписано при перезапуске узла.

## <a name="failover-controller-services"></a>Службы отказоустойчивого контроллера

Контроллер отработки отказа HDInsight также отвечает за обновление IP-адреса узла головного узла, который указывает на текущий активный головной узел. Все агенты Ambari настроены для передачи состояния и пульса в узел головного узла. Контроллер отработки отказа — это набор служб, выполняющихся на каждом узле в кластере. Если они не выполняются, головного узланая отработка отказа может работать неправильно и при попытке доступа к серверу Ambari будет использоваться HTTP 502.

Чтобы проверить, какая головного узла активна, одним из способов является подключение SSH к одному из узлов в кластере, затем выполняется `ping headnodehost` и сравнивается IP-адрес с этими двумя головных узлах.

Если службы контроллера отработки отказа не работают, головного узланая отработка отказа может выполняться неправильно, что может привести к невозможности запуска Ambari Server. Чтобы проверить, выполняются ли службы отказоустойчивого контроллера, выполните:

```bash
ps -ef | grep failover
```

## <a name="logs"></a>Журналы

На активной головного узла можно проверить журналы сервера Ambari по адресу:

```
/var/log/ambari-server/ambari-server.log
/var/log/ambari-server/ambari-server-check-database.log
```

На любом узле кластера вы можете проверить журналы агента Ambari по адресу:

```bash
/var/log/ambari-agent/ambari-agent.log
```

## <a name="service-start-sequences"></a>Последовательности запусков службы

Это последовательность запуска службы во время загрузки:

1. Hdinsight — агент запускает службы отказоустойчивого контроллера.
1. Службы контроллера отработки отказа запускают агент Ambari на каждом узле и сервере Ambari на активном головного узла.

## <a name="ambari-database"></a>База данных Ambari

HDInsight создает базу данных в базе данных SQL, которая будет использоваться в качестве базы данных для сервера Ambari. Уровень служб по умолчанию [— S0](../azure-sql/database/elastic-pool-scale.md).

Для любого кластера с числом рабочих узлов больше 16 при создании кластера S2 — это уровень служб базы данных.

## <a name="takeaway-points"></a>Точки мысль

Никогда не запускайте и останавливаются службы ambari-Server или ambari-Agent, если только вы не пытаетесь перезапустить службу для устранения проблемы. Чтобы принудительно выполнить отработку отказа, можно перезапустить активный головного узла.

Никогда не изменяйте файлы конфигурации на любом узле кластера вручную, позвольте Ambari пользовательскому интерфейсу выполнять задание.

## <a name="property-values-in-esp-clusters"></a>Значения свойств в кластерах ESP

В HDInsight 4,0 Корпоративный пакет безопасности кластерах в `|` качестве разделителей переменных используйте каналы, а не запятые. Пример приведен ниже.

```
Property Key: hive.security.authorization.sqlstd.confwhitelist.append
Property Value: environment|env|dl_data_dt
```

## <a name="next-steps"></a>Дальнейшие действия

* [Управление кластерами HDInsight с помощью веб-интерфейса Apache Ambari](hdinsight-hadoop-manage-ambari.md)
* [Управление кластерами HDInsight с помощью Apache Ambari REST API](hdinsight-hadoop-manage-ambari-rest-api.md)

[!INCLUDE [troubleshooting next steps](../../includes/hdinsight-troubleshooting-next-steps.md)]
