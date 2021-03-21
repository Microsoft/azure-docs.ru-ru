---
title: Интерфейсы API для доступа к данным анализа коммерческих рынков
description: Используйте эти API для программного доступа к данным аналитики в центре партнеров.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: sayantanroy83
ms.author: sroy
ms.date: 3/08/2021
ms.openlocfilehash: ed6d658155267ab21fd4cdd28dd50fcbb258ee90
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102584155"
---
# <a name="apis-for-accessing-commercial-marketplace-analytics-data"></a>Интерфейсы API для доступа к данным анализа коммерческих рынков

Ниже приведен список интерфейсов API для доступа к данным анализа коммерческих рынков и связанным с ними функциональным возможностям.

- [API-интерфейсы извлечения набора данных](#dataset-pull-apis)
- [API управления запросами](#query-management-apis)
- [API-интерфейсы управления отчетами](#report-management-apis)
- [API-интерфейсы извлечения выполнения отчета](#report-execution-pull-apis)

## <a name="dataset-pull-apis"></a>API-интерфейсы извлечения набора данных

***Таблица 1. API-интерфейсы извлечения набора данных***

| **API** | **функциональное назначение;** |
| --- | --- |
| [Получение всех наборов данных](analytics-api-get-all-datasets.md) | Возвращает все доступные наборы данных. Наборы данных. список таблиц, столбцов, метрик и временных диапазонов. |
|||

## <a name="query-management-apis"></a>API управления запросами

***Таблица 2. API-интерфейсы управления запросами***

| **API** | **функциональное назначение;** |
| --- | --- |
| [Создание запроса отчета](analytics-programmatic-access.md#create-report-query-api) | Создает пользовательские запросы, определяющие набор данных, из которого необходимо экспортировать столбцы и метрики. |
| [ПОЛУЧЕНИЕ запросов отчетов](analytics-api-get-report-queries.md) | Возвращает все запросы, доступные для использования в отчетах. Получает все системные и пользовательские запросы по умолчанию. |
| [Удаление запросов отчета](analytics-api-delete-report-queries.md) | Удаляет определяемые пользователем запросы. |
|||

## <a name="report-management-apis"></a>API-интерфейсы управления отчетами

***Таблица 3. API-интерфейсы управления отчетами***

| **API** | **функциональное назначение;** |
| --- | --- |
| [Создавать отчет](analytics-programmatic-access.md#create-report-api) | Планирует выполнение запроса с регулярными интервалами. |
| [ПРОБные запросы отчетов](analytics-api-try-report-queries.md) | Выполняет инструкцию запроса отчета. Возвращает только 10 записей, которые партнер может использовать для проверки ожидаемых данных. |
| [Получить отчет](analytics-api-get-report.md) | Получение всех запланированных отчетов. |
| [Обновить отчет](analytics-api-update-report.md) | Изменение параметра отчета. |
| [Удалить отчет](analytics-api-delete-report.md) | Удаляет все записи отчета и выполнения отчета. |
| [Приостановить выполнение отчета](analytics-api-pause-report-executions.md) | Приостанавливает запланированное выполнение отчетов. |
| [Возобновление выполнения отчетов](analytics-api-resume-report-executions.md) | Возобновляет запланированное выполнение приостановленного отчета. |
|||

## <a name="report-execution-pull-apis"></a>API-интерфейсы извлечения выполнения отчета

***Таблица 4. API-интерфейсы извлечения для выполнения отчета***

| **API** | **функциональное назначение;** |
| --- | --- |
| [Получение выполнений отчетов](analytics-programmatic-access.md#get-report-executions-api) | Получение всех выполнений, произошедших для данного отчета. |
|||

## <a name="next-steps"></a>Дальнейшие действия

- Вы можете испытать интерфейсы API с помощью [URL-адреса API Swagger](https://api.partnercenter.microsoft.com/insights/v1/cmp/swagger/index.html).
