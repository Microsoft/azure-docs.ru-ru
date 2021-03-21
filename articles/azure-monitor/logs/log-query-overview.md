---
title: Запросы журналов в Azure Monitor
description: Справочная информация о языке запросов Kusto, используемого в Azure Monitor. Статья включает дополнительные элементы, относящиеся к Azure Monitor, а также элементы, которые не поддерживаются в запросах журнала Azure Monitor.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/09/2020
ms.openlocfilehash: 529fc432bf8777ef7e2b527f08e9cb59e42bf156
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102041031"
---
# <a name="log-queries-in-azure-monitor"></a>Запросы журналов в Azure Monitor
Azure Monitor журналы основаны на обозреватель данныхах Azure, а запросы журнала записываются с помощью того же языка запросов Kusto (ККЛ). Это расширенный язык, который упрощает чтение и создание запросов, поэтому вы можете приступить к написанию запросов, опираясь на базовые рекомендации.

Области в Azure Monitor, где будут использоваться запросы, включают следующее:

- [Log Analytics](../logs/log-analytics-overview.md). Основное средство в портал Azure для редактирования запросов журналов и интерактивного анализа результатов. Даже если вы планируете использовать запрос журнала в другом месте Azure Monitor, вы обычно пишете и тестируете его в Log Analytics, прежде чем скопировать в конечное расположение.
- [Правила генерации оповещений журнала](../alerts/alerts-overview.md). Заблаговременно выявление проблем с данными в рабочей области.  Каждое правило генерации оповещений основано на запросе журнала, который автоматически выполняется через регулярные интервалы времени.  Результаты проверяются, чтобы определить, следует ли создавать оповещение.
- [Книги](../visualize/workbooks-overview.md). Включение результатов запросов журнала с помощью различных визуализаций в интерактивных визуальных отчетах в портал Azure.
- [Панели мониторинга Azure](../visualize/tutorial-logs-dashboards.md). Закрепите результаты любого запроса на панели мониторинга Azure, которые позволяют визуализировать данные журналов и метрик вместе и при необходимости предоставлять общий доступ другим пользователям Azure.
- [Logic Apps](../logs/logicapp-flow-connector.md).  Используйте результаты запроса журнала в автоматическом рабочем процессе, используя Logic Apps.
- [PowerShell.](/powershell/module/az.operationalinsights/get-azoperationalinsightssearchresult) Используйте результаты запроса журнала в скрипте PowerShell из командной строки или модуля Runbook службы автоматизации Azure, который использует Get-Азоператионалинсигхтссеарчресултс.
- [API Azure Monitor журналов](https://dev.loganalytics.io). Получение данных журнала из рабочей области из любого REST API клиента.  Запрос API включает запрос, который выполняется в Azure Monitor, чтобы определить извлекаемые данные.

## <a name="getting-started"></a>Начало работы
Лучший способ начать обучение написанию запросов к журналу с помощью ККЛ — это использование доступных руководств и примеров.

- [Log Analytics учебник](./log-analytics-tutorial.md) по использованию функций log Analytics, которые используются в портал Azure для редактирования и выполнения запросов. Он также позволяет создавать простые запросы без непосредственного использования языка запросов. Если вы еще не работали с Log Analytics, начните сейчас, чтобы изучить этот инструмент, который вы будете использовать в других учебниках и примерах.
- Руководство по [ККЛ](/azure/data-explorer/kusto/query/tutorial?pivots=azuremonitor) . пошаговые инструкции по основным концепциям ККЛ и общим операторам. Это лучшее место для начала работы, чтобы ускорить работу с самим языком и структурой запросов журнала. 
- [Примеры запросов](../logs/example-queries.md) . Описание примеров запросов, доступных в log Analytics. Вы можете использовать запросы без изменений или в качестве примеров для изучения KQL.
- [Примеры запросов](/azure/data-explorer/kusto/query/samples?pivots=azuremonitor) . Примеры запросов, иллюстрирующие различные концепции.



## <a name="reference-documentation"></a>Справочная документация
[Документация по ККЛ](/azure/data-explorer/kusto/query/) , включая Справочник по всем командам и операторам, доступна в документации по Azure обозреватель данных. Даже если вы хорошо знакомы с ККЛ, вы по-прежнему будете регулярно использовать ссылку для изучения новых команд и сценариев, которые раньше не использовались.


## <a name="language-differences"></a>Различия между языками
Хотя Azure Monitor использует те же ККЛ, что и обозреватель данных Azure, существуют некоторые различия. В документации по ККЛ будут указаны операторы, которые не поддерживаются Azure Monitor или с разными функциональными возможностями. Операторы, относящиеся к Azure Monitor, описаны в содержимом Azure Monitor. В следующих разделах приводится список различий между версиями языка для краткого справочника.

### <a name="statements-not-supported-in-azure-monitor"></a>Инструкции, которые не поддерживаются в Azure Monitor:

* [Псевдоним](/azure/kusto/query/aliasstatement)
* [параметры запроса](/azure/kusto/query/queryparametersstatement).

### <a name="functions-not-supported-in-azure-monitor"></a>Функции, которые не поддерживаются в Azure Monitor:

* [кластер ()](/azure/kusto/query/clusterfunction)
* [cursor_after()](/azure/kusto/query/cursorafterfunction)
* [cursor_before_or_at()](/azure/kusto/query/cursorbeforeoratfunction)
* [cursor_current(), current_cursor()](/azure/kusto/query/cursorcurrent);
* [база данных ()](/azure/kusto/query/databasefunction)
* [current_principal()](/azure/kusto/query/current-principalfunction)
* [extent_id()](/azure/kusto/query/extentidfunction)
* [extent_tags()](/azure/kusto/query/extenttagsfunction)

### <a name="operators-not-supported-in-azure-monitor"></a>Операторы, которые не поддерживаются в Azure Monitor:

* [Соединение между кластерами](/azure/kusto/query/joincrosscluster)

### <a name="plugins-not-supported-in-azure-monitor"></a>Подключаемые модули, которые не поддерживаются в Azure Monitor:

* [Подключаемый модуль Python](/azure/kusto/query/pythonplugin)
* [Подключаемый модуль sql_request](/azure/kusto/query/sqlrequestplugin)


### <a name="additional-operators-in-azure-monitor"></a>Дополнительные операторы в Azure Monitor
Следующие операторы поддерживают конкретные функции Azure Monitor и недоступны вне Azure Monitor.

* [приложение ()](../logs/app-expression.md)
* [ресурс ()](./resource-expression.md)
* [Рабочая область ()](../logs/workspace-expression.md)

## <a name="next-steps"></a>Дальнейшие действия
- Пошаговое [руководство по написанию запросов](/azure/data-explorer/kusto/query/tutorial?pivots=azuremonitor).
- Ознакомьтесь с полной [справочной документацией по языку запросов Kusto](/azure/kusto/query/).