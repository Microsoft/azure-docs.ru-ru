---
title: Масштабирование HiveServer2 в Azure HDInsight
description: Горизонтальное масштабирование HiveServer2 в кластерах Azure HDInsight с использованием граничных узлов для повышения отказоустойчивости и доступности.
services: hdinsight
ms.service: hdinsight
ms.topic: conceptual
ms.reviewer: hrasheed-msft
ms.author: kecheung
author: kcheeeung
ms.date: 08/12/2020
ms.openlocfilehash: d0a0df4791492c1c9f0d600630d723024c46c1b8
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88227401"
---
# <a name="scale-hiveserver2-on-azure-hdinsight-clusters-for-high-availability"></a>Масштабирование HiveServer2 в кластерах Azure HDInsight для обеспечения высокой доступности

Узнайте, как развернуть дополнительные HiveServer2 в кластере, чтобы повысить доступность и распределение нагрузки. При увеличении размера головного узла можно также использовать граничные узлы для развертывания HiveServer2. 

> [!NOTE]
> В зависимости от используемого использования увеличение числа HiveServer2 может увеличить число подключений к хранилище метаданных Hive. Также убедитесь, что база данных SQL Azure имеет правильный размер.

## <a name="prerequisites"></a>Предварительные требования

Чтобы использовать это руководство, необходимо ознакомиться со следующей статьей:
- [Использование пустых граничных узлов в кластерах Apache Hadoop в HDInsight](hdinsight-apps-use-edge-node.md)

## <a name="install-hiveserver2"></a>Установка HiveServer2

В этом разделе вы установите дополнительный HiveServer2 на целевые узлы.

1. Откройте Ambari в браузере и щелкните целевой узел.

:::image type="content" source="media/hdinsight-apps-install-hiveserver2/hdinsight-install-hiveserver2-a.png" alt-text="Меню узлов Ambari.":::

2. Нажмите кнопку "Добавить" и выберите HiveServer2

:::image type="content" source="media/hdinsight-apps-install-hiveserver2/hdinsight-install-hiveserver2-b.png" alt-text="Добавьте HiveServer2 панель узла.":::

3. Подтвердите, и процесс будет выполнен. Повторите 1-3 для всех нужных узлов.

4. После завершения установки перезапустите все службы с устаревшими конфигурациями и запустите HiveServer2.

:::image type="content" source="media/hdinsight-apps-install-hiveserver2/hdinsight-install-hiveserver2-c.png" alt-text="Запустите панель HiveServer2.":::

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как установить HiveServer2 на кластер. Дополнительные сведения о граничных узлах и приложениях см. в следующих статьях:

* [Установка пограничной узла](hdinsight-apps-use-edge-node.md). Узнайте, как установить пограничным узел в кластер HDInsight.
* [Установка приложений в HDInsight](hdinsight-apps-install-applications.md). Узнайте, как установить приложения HDInsight в кластерах.
* [Ограничения подключения DTU SQL Azure](../azure-sql/database/resource-limits-dtu-single-databases.md): Узнайте об ограничениях базы данных SQL Azure с помощью DTU.
* [Ограничения подключения Виртуальное ядро для Azure SQL](../azure-sql/database/resource-limits-vcore-elastic-pools.md): сведения об ограничениях базы данных SQL Azure с помощью виртуальных ядер.
