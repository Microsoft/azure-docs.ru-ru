---
title: Мониторинг пулов SQL в синапсе Studio
description: Узнайте, как отслеживать пулы SQL с помощью синапсе Studio.
services: synapse-analytics
author: matt1883
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: monitoring
ms.date: 11/30/2020
ms.author: mahi
ms.reviewer: mahi
ms.openlocfilehash: 63b63526333435387c3ff5b5c9d5599ec851c1a8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96467423"
---
# <a name="use-synapse-studio-to-monitor-your-sql-pools"></a>Использование синапсе Studio для мониторинга пулов SQL

С помощью синапсе Studio можно выполнять скрипты SQL в пулах SQL в рабочей области.

В этой статье объясняется, как отслеживать пулы SQL, что позволяет следить за состоянием и действиями пулов.

## <a name="access-sql-pools-list"></a>Список пулов SQL Access

Чтобы просмотреть список пулов SQL в рабочей области, сначала [откройте синапсе Studio](https://web.azuresynapse.net/) и выберите рабочую область.

![Войти в рабочую область](./media/common/login-workspace.png)

Открыв рабочую область, выберите раздел **монитор** слева.

![Выбор концентратора мониторинга](./media/common/left-nav.png)

Выберите **Пулы SQL** , чтобы просмотреть список пулов SQL.

 ![Выбор пулов SQL](./media/how-to-monitor-sql-pools/monitor-hub-nav-sql-pools.png)

## <a name="filter-your-sql-pools"></a>Фильтрация пулов SQL

Список пулов SQL можно отфильтровать по интересующим вас категориям. Фильтры в верхней части экрана позволяют указать поле, по которому вы хотите выполнить фильтрацию.

Например, можно отфильтровать представление, чтобы увидеть только пулы SQL, содержащие имя «салесрекордс»:

![Образец фильтра](./media/how-to-monitor-sql-pools/filter-example.png)

## <a name="view-details-about-a-specific-sql-pool"></a>Просмотр сведений о конкретном пуле SQL

Чтобы просмотреть сведения об одном из пулов SQL, выберите пул SQL для просмотра сведений.

![Сведения о пуле SQL](./media/how-to-monitor-sql-pools/sql-pool-details.png)

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о выполнении конвейера мониторинга см. в статье [конвейер мониторинга, выполняемый в синапсе Studio](how-to-monitor-pipeline-runs.md) . 

Дополнительные сведения о мониторинге запросов SQL см. в статье [мониторинг запросов SQL в синапсе Studio](how-to-monitor-sql-requests.md) .