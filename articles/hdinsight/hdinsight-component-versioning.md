---
title: Компоненты и версии Apache Hadoop в Azure HDInsight
description: Узнайте о компонентах и версиях Apache Hadoop в Azure HDInsight.
ms.service: hdinsight
ms.topic: conceptual
author: deshriva
ms.author: deshriva
ms.date: 02/08/2021
ms.openlocfilehash: 53ca2ac73fdec9d3b39ffc04cbb24aca707a72eb
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103490455"
---
# <a name="azure-hdinsight-versions"></a>Версии Azure HDInsight

HDInsight объединяет Apache Hadoop компоненты среды и платформу HDInsight в пакет, развернутый в кластере. Дополнительные сведения см. в статье [как работает управление версиями HDInsight](hdinsight-overview-versioning.md).

## <a name="supported-hdinsight-versions"></a>Поддерживаемые версии HDInsight

В этой таблице перечислены версии HDInsight, доступные в портал Azure и других методах развертывания, таких как PowerShell, CLI и пакет SDK для .NET.

| Версия HDInsight | ОС виртуальной машины | Дата выпуска | Дата прекращения поддержки | Дата вывода | Высокий уровень доступности |
| --- | --- | --- | --- | --- | --- |
| [HDInsight 4.0](hdinsight-40-component-versioning.md) |Ubuntu 16.0.4 LTS |24 сентября 2018 г. | | |Да |
| [HDInsight 3.6](hdinsight-36-component-versioning.md) |Ubuntu 16.0.4 LTS |4 апреля 2017 г.      | * 30 июня 2021 г. |30 июня 2021 г. |Да |

* Мы расширяем период поддержки для определенных типов кластеров HDInsight 3,6. См. раздел [версии компонентов HDInsight 3,6](hdinsight-36-component-versioning.md).

## <a name="release-notes"></a>Заметки о выпуске

Дополнительные заметки о выпусках последних версий HDInsight см. [здесь](hdinsight-release-notes.md).

## <a name="support-options-for-hdinsight-versions"></a>Варианты поддержки для версий HDInsight

HDInsight предлагает стандартную поддержку, которая определяется как период времени, в течение которого версия HDInsight поддерживается службой поддержки клиентов Майкрософт.

**Прекращение поддержки** означает, что корпорация Майкрософт больше не будет предоставлять поддержку для конкретной версии HDInsight. Эта версия больше не будет предоставляться на портале Azure для создания кластера.

**Прекращение использования** означает, что существующие кластеры с этой версией HDInsight продолжат работу обычным образом. Новые кластеры с такой версией создать будет нельзя, в том числе с помощью CLI и пакетов SDK. Другие функции плоскости управления, такие как ручное масштабирование и автоматическое масштабирование, не будут работать после даты выбытия. Поддержка версий, использование которых прекращено, также не осуществляется.

## <a name="versioning-considerations"></a>Вопросы управления версиями
- После развертывания кластера с образом этот кластер не будет автоматически обновлен до более новой версии образа. При создании новых кластеров будет развернута последняя версия образа.
- Клиенты должны протестировать и проверить, правильно ли работают приложения при использовании новой версии HDInsight.
- HDInsight оставляет за собой право изменить версию по умолчанию без предварительного уведомления. При наличии зависимости версий укажите версию HDInsight при создании кластеров.
- HDInsight может снять с учета версию компонента OSS до снятия версии HDInsight.

## <a name="next-steps"></a>Дальнейшие действия

- [Установка кластеров в HDInsight с использованием Hadoop, Spark, Kafka и других технологий](hdinsight-hadoop-provision-linux-clusters.md)
- [Пакет безопасности корпоративного уровня](./enterprise-security-package.md)
- [Работа в экосистеме Hadoop в HDInsight на компьютере с Windows](hdinsight-hadoop-windows-tools.md)