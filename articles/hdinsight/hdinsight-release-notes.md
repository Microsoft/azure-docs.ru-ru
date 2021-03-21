---
title: Заметки о выпуске Azure HDInsight
description: Последние заметки о выпуске Azure HDInsight. Ознакомьтесь с советами по разработке и подробными сведениями о Hadoop, Spark, R Server, Hive и т. д.
ms.custom: hdinsightactive
ms.service: hdinsight
ms.topic: conceptual
ms.date: 02/08/2021
ms.openlocfilehash: 3ea5168641f233f2e9580612a2f245fd8fef860f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101699357"
---
# <a name="azure-hdinsight-release-notes"></a>Заметки о выпуске Azure HDInsight

Эта статья содержит сведения о **последних** обновлениях выпуска Azure HDInsight. Дополнительные сведения о предыдущих выпусках см. в статье [Заметки о выпуске для компонентов Hadoop в Azure HDInsight (архив)](hdinsight-release-notes-archive.md).

## <a name="summary"></a>Сводка

Azure HDInsight является одной из самых популярных среди корпоративных клиентов служб для аналитики с открытым кодом в Azure.

Если вы хотите подписываться на заметки о выпуске, просмотрите выпуски в [этом репозитории GitHub](https://github.com/hdinsight/release-notes/releases).

## <a name="release-date-02052021"></a>Дата выпуска: 02/05/2021

Этот выпуск относится как к HDInsight 3,6, так и к HDInsight 4,0. Выпуск HDInsight предоставляется для разных регионов на протяжении нескольких дней. Указанная здесь дата выпуска обозначает дату выпуска для первого из регионов. Если вы не видите приведенных ниже изменений, дождитесь выхода выпуска в вашем регионе в течение нескольких дней.

## <a name="new-features"></a>новые функции;
### <a name="dav4-series-support"></a>Поддержка серии Dav4
В этом выпуске добавлена поддержка Dav4 серии HDInsight. Дополнительные сведения о [серии Dav4](/azure/virtual-machines/dav4-dasv4-series)см. здесь.

### <a name="kafka-rest-proxy-ga"></a>Прокси-сервер Kafka RESTFUL, общедоступный 
Прокси-сервер Kafka RESTFUL позволяет взаимодействовать с кластером Kafka через REST API по протоколу HTTPS. Прокси-сервер Kafka RESTful является общедоступным, начиная с этого выпуска. Дополнительные сведения о [прокси-сервере KAFKA RESTful](/azure/hdinsight/kafka/rest-proxy)см. здесь.

### <a name="moving-to-azure-virtual-machine-scale-sets"></a>Переход на масштабируемые наборы виртуальных машин Azure
Сейчас HDInsight использует для подготовки кластера виртуальные машины Azure. Служба постепенно переносится в [масштабируемые наборы виртуальных машин Azure](../virtual-machine-scale-sets/overview.md). Весь процесс может занять несколько месяцев. После переноса регионов и подписок вновь созданные кластеры HDInsight будут работать в масштабируемых наборах виртуальных машин без действий клиента. Критические изменения не требуются.

## <a name="deprecation"></a>Устаревшее
### <a name="disabled-vm-sizes"></a>Размеры отключенных виртуальных машин
Начиная с 9 2021 января HDInsight все клиенты будут блокировать создание кластеров с помощью standand_A8, standand_A9, standand_A10 и standand_A11 размеров виртуальных машин. Существующие кластеры будут работать, как есть. Рассмотрите возможность перехода на HDInsight 4,0, чтобы избежать потенциальных прерываний системы и поддержки.

## <a name="behavior-changes"></a>Изменения в поведении
### <a name="default-cluster-vm-size-changes-to-ev3-series"></a>Изменение размера виртуальной машины кластера по умолчанию на Ev3 
Размеры виртуальных машин кластера по умолчанию будут изменены с серии D на Ev3. Это изменение относится к головным узлам и рабочим узлам. Чтобы избежать этого изменения, влияющего на Тестируемые рабочие процессы, укажите размеры виртуальных машин, которые вы хотите использовать в шаблоне ARM.

### <a name="network-interface-resource-not-visible-for-clusters-running-on-azure-virtual-machine-scale-sets"></a>Ресурс сетевого интерфейса не отображается для кластеров, работающих в масштабируемых наборах виртуальных машин Azure
Миграция HDInsight постепенно выполняется в масштабируемых наборах виртуальных машин Azure. Сетевые интерфейсы для виртуальных машин больше не видны клиентам кластеров, использующих масштабируемые наборы виртуальных машин Azure.


### <a name="breaking-change-for-net-for-apache-spark-100"></a>Критическое изменение для .NET для Apache Spark 1.0.0
В последнем выпуске HDInsight представляет собой первую официальную версию 1.0.0 библиотеки [".NET для Apache Spark"](https://github.com/dotnet/spark) . Он обеспечивает полноту API кадров данных для Spark 2.4. x и Spark 3.0. x вместе с [другими функциями](https://github.com/dotnet/spark/blob/master/docs/release-notes/1.0.0/release-1.0.0.md). В этой основной версии будут внесены критические изменения. Дополнительные сведения о шагах, необходимых для обновления кода и конвейеров, см. в [руководстве по миграции .NET для Apache Spark](https://github.com/dotnet/spark/blob/master/docs/migration-guide.md#upgrading-from-microsoftspark-0x-to-10) . Дополнительные сведения см. в руководстве по [.NET для Apache Spark v 1.0 в Azure HDInsight](/azure/hdinsight/spark/spark-dotnet-version-update#using-net-for-apache-spark-v10-in-hdinsight).


## <a name="upcoming-changes"></a>Предстоящие изменения
В предстоящих выпусках будут внесены описанные ниже изменения.

### <a name="default-cluster-version-will-be-changed-to-40"></a>Версия кластера по умолчанию будет изменена на 4,0
Начиная с 2021 февраля, версия кластера HDInsight по умолчанию будет изменена с 3,6 на 4,0. Дополнительные сведения о доступных версиях см. в разделе [Доступные версии](./hdinsight-component-versioning.md). Дополнительные сведения о новых возможностях в [HDInsight 4,0](./hdinsight-version-release.md).

### <a name="os-version-upgrade"></a>Обновление версии ОС
HDInsight обновляет версию ОС с Ubuntu 16,04 до 18,04. Обновление будет выполнено до 2021 апреля.

### <a name="hdinsight-36-end-of-support-on-june-30-2021"></a>Окончание поддержки HDInsight 3,6 в июне 30 2021
HDInsight 3,6 будет иметь окончание поддержки. Начиная с 30 2021 июня клиенты не могут создавать кластеры HDInsight 3,6. Существующие кластеры будут продолжать работу без изменений, но без поддержки со стороны корпорации Майкрософт. Рассмотрите возможность перехода на HDInsight 4,0, чтобы избежать потенциальных прерываний системы и поддержки.

## <a name="bug-fixes"></a>Исправления ошибок
HDInsight постоянно повышает надежность и производительность кластеров. 

## <a name="component-version-change"></a>Изменение версий компонентов
В этом выпуска не вносилось изменений в версии компонентов. Текущие версии компонентов для HDInsight 4,0 и HDInsight 3,6 можно найти в [этом документе](./hdinsight-component-versioning.md).
