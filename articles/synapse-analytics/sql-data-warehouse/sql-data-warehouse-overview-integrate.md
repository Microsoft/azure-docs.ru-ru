---
title: Создание интегрированных решений
description: Средства и партнеры решений, которые интегрируются с выделенным пулом SQL (ранее — ХРАНИЛИЩЕм данных SQL) в Azure синапсе Analytics.
services: synapse-analytics
author: mlee3gsd
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 04/17/2018
ms.author: martinle
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: 3e55ef054d5c305937f88d6ec5b2b4453cac6792
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98117764"
---
# <a name="integrate-other-services-with-a-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>Интегрируйте другие службы с выделенным пулом SQL (ранее — ХРАНИЛИЩЕм данных SQL) в Azure синапсе Analytics.

Выделенная функция пула SQL (прежнее название — SQL DW) в Azure синапсе Analytics позволяет пользователям интегрироваться с многими другими службами в Azure. Выделенный пул SQL может использовать несколько дополнительных служб, некоторые из которых включают:

* Power BI
* Фабрика данных Azure
* Машинное обучение Azure
* Azure Stream Analytics

Дополнительные сведения об интеграции разных служб Azure см. в статье [Партнеры по интеграции данных в Azure Synapse Analytics](sql-data-warehouse-partner-data-integration.md).

## <a name="power-bi"></a>Power BI

Интеграция с Power BI позволяет совмещать вычислительные ресурсы хранилища данных с возможностями создания динамических отчетов и визуализации в Power BI. На данный момент интеграция с Power BI включает:

* **Прямое подключение**: более расширенное подключение с использованием логического включение к хранилищу данных, подготовленное с помощью выделенного пула SQL (прежнее название — SQL DW). Это обеспечивает более быстрый анализ в большем масштабе.
* **Открыть в Power BI**: кнопка "Открыть в Power BI" передает сведения об экземпляре в Power BI, упрощая подключение.

Дополнительные сведения см. в статье [Использование Power BI с хранилищем данных SQL](/power-bi/connect-data/service-azure-sql-data-warehouse-with-direct-connect) или в [документации по Power BI](https://powerbi.microsoft.com/blog/exploring-azure-sql-data-warehouse-with-power-bi/).

## <a name="azure-data-factory"></a>Фабрика данных Azure

Фабрика данных Azure предоставляет пользователям управляемую платформу для создания сложных конвейеров извлечения и загрузки. Интеграция выделенного пула SQL (прежнее название — SQL DW) с фабрикой данных Azure включает:

* **Хранимые процедуры.** Оркестрация выполнения хранимых процедур.
* **Копирование**: используйте ADF для перемещения данных в выделенный пул SQL (ранее — хранилище SQL DW). В основе этой операции может использоваться стандартный механизм перемещения данных ADF или PolyBase.

Дополнительную информацию см. в статье [Визуализация данных с помощью Power BI](../../data-factory/load-azure-sql-data-warehouse.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).

## <a name="azure-machine-learning"></a>Машинное обучение Azure

Служба "Машинное обучение Azure" — это полностью управляемая аналитическая служба, позволяющая пользователям создавать сложные модели с использованием крупного набора прогнозируемых инструментов. Выделенный пул SQL (ранее — хранилище данных SQL) поддерживается в качестве источника и назначения для этих моделей и имеет следующие функциональные возможности.

* **Чтение данных:** Масштабирование моделей с помощью T-SQL для выделенного пула SQL (ранее — хранилища данных SQL).
* **Запись данных:** Зафиксируйте изменения из любой модели обратно в выделенный пул SQL (ранее — хранилище данных SQL).

Дополнительную информацию см. в статье [Анализ данных с помощью машинного обучения Azure](sql-data-warehouse-get-started-analyze-with-azure-machine-learning.md).

## <a name="azure-stream-analytics"></a>Azure Stream Analytics

Azure Stream Analytics — это сложная полностью управляемая инфраструктура для обработки и потребления данных о событиях, создаваемых концентратором событий Azure.  Интеграция с выделенным пулом SQL (прежнее название — хранилище данных SQL) позволяет эффективно обрабатывать потоковые данные и хранить их вместе с реляционными данными, обеспечивая более глубокий и расширенный анализ.  

* **Выходные данные задания:** Отправка выходных данных заданий Stream Analytics непосредственно в выделенный пул SQL (ранее — хранилище данных SQL).

Дополнительные сведения см. в статье [Работа со службой Azure Stream Analytics и хранилищем данных SQL](sql-data-warehouse-integrate-azure-stream-analytics.md).