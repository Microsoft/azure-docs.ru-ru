---
title: Мониторинг пулов Apache Spark в синапсе Studio
description: Узнайте, как отслеживать пулы Apache Spark с помощью синапсе Studio.
services: synapse-analytics
author: matt1883
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: monitoring
ms.date: 11/30/2020
ms.author: mahi
ms.reviewer: mahi
ms.openlocfilehash: 8d95897e0c2d58b2a3955918be945800eed9ba56
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96467242"
---
# <a name="use-synapse-studio-to-monitor-your-apache-spark-pools"></a>Использование синапсе Studio для мониторинга пулов Apache Spark

С помощью Azure синапсе Analytics вы можете использовать Spark для запуска записных книжек, заданий и других типов приложений в пулах Spark в рабочей области.

В этой статье объясняется, как отслеживать пулы Apache Spark, позволяя следить за состоянием пулов, включая количество виртуальных ядер, используемых разными пользователями рабочей области.

## <a name="access-spark-pools-list"></a>Доступ к списку пулов Spark

Чтобы просмотреть список пулов Apache Spark в рабочей области, сначала [откройте синапсе Studio](https://web.azuresynapse.net/) и выберите рабочую область.

![Войти в рабочую область](./media/common/login-workspace.png)

Открыв рабочую область, выберите раздел **монитор** слева.

![Выбор концентратора мониторинга](./media/common/left-nav.png)

Выберите **пулы Apache Spark** , чтобы просмотреть список пулов Apache Spark.

 ![Выбор пулов Apache Spark](./media/how-to-monitor-spark-pools/monitor-hub-nav-spark-pools.png)

## <a name="filter-your-spark-pools"></a>Фильтрация пулов Spark

Список пулов Spark можно отфильтровать по интересующим вас категориям. Фильтры в верхней части экрана позволяют указать поле, по которому вы хотите выполнить фильтрацию.

Например, можно отфильтровать представление, чтобы увидеть только пулы Spark, содержащие имя "Подготовка к просмотру":

![Образец фильтра](./media/how-to-monitor-spark-pools/filter-example.png)

## <a name="view-details-about-a-specific-spark-pool"></a>Просмотр сведений о конкретном пуле Spark

Чтобы просмотреть сведения об одном из пулов Spark, выберите пул Spark, чтобы просмотреть сведения.

![Сведения о пуле Apache Spark](./media/how-to-monitor-spark-pools/spark-pool-details.png)

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о выполнении конвейера мониторинга см. в статье [конвейер мониторинга, выполняемый в синапсе Studio](how-to-monitor-pipeline-runs.md) . 

Дополнительные сведения о мониторинге Apache Spark приложений см. в статье [мониторинг Apache Spark приложений в синапсе Studio](how-to-monitor-spark-applications.md) .
