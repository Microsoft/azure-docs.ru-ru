---
title: Мониторинг запросов SQL в синапсе Studio
description: Узнайте, как отслеживать запросы SQL с помощью синапсе Studio.
services: synapse-analytics
author: matt1883
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: monitoring
ms.date: 11/30/2020
ms.author: mahi
ms.reviewer: mahi
ms.openlocfilehash: 7296fec91decaf1bd80829f9f3a743a518fbb7a7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96467402"
---
# <a name="use-synapse-studio-to-monitor-your-sql-requests"></a>Использование синапсе Studio для мониторинга запросов SQL

С помощью синапсе Studio можно выполнять скрипты SQL в пулах SQL в рабочей области.

В этой статье объясняется, как отслеживать запросы SQL, что позволяет следить за состоянием выполняющихся запросов и находить подробные сведения о исторических запросах.

## <a name="access-sql-requests-list"></a>Список запросов SQL Access

Чтобы просмотреть список запросов SQL в рабочей области, сначала [откройте синапсе Studio](https://web.azuresynapse.net/) и выберите рабочую область.

![Войти в рабочую область](./media/common/login-workspace.png)

Открыв рабочую область, выберите раздел **монитор** слева.

![Выбор концентратора мониторинга](./media/common/left-nav.png)

Выберите **SQL запросы** , чтобы просмотреть список запросов SQL.

 ![Выбор запросов SQL](./media/how-to-monitor-sql-requests/monitor-hub-nav-sql-requests.png)

## <a name="select-a-sql-pool"></a>Выберите пул SQL

По умолчанию в этом представлении отображается журнал запросов SQL для встроенного несерверного пула SQL. Вы можете выбрать один из выделенных пулов SQL, чтобы просмотреть журнал запросов SQL этого пула.

![Выбор пула SQL](./media/how-to-monitor-sql-requests/select-pool.png)

## <a name="filter-your-sql-requests"></a>Фильтрация запросов SQL

Список запросов SQL можно отфильтровать по интересующим вас запросам. Фильтры в верхней части экрана позволяют указать поле, по которому вы хотите выполнить фильтрацию.

Например, можно отфильтровать представление, чтобы просмотреть только запросы SQL, отправленные `maria@contoso.com` :

![Образец фильтра](./media/how-to-monitor-sql-requests/filter-example.png)

## <a name="view-details-about-a-specific-sql-request"></a>Просмотр сведений об определенном запросе SQL

Чтобы просмотреть сведения об одном из запросов SQL, откройте запрос SQL для перехода к подробному представлению. Для сложных запросов, выполняющихся на выделенных пулах SQL, можно отслеживать ход выполнения.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о выполнении конвейера мониторинга см. в статье [конвейер мониторинга, выполняемый в синапсе Studio](how-to-monitor-pipeline-runs.md) . 

Дополнительные сведения о мониторинге Apache Spark приложений см. в статье [мониторинг Apache Spark приложений в синапсе Studio](how-to-monitor-spark-applications.md) .