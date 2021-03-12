---
title: Примеры запросов для программной аналитики
description: Используйте эти примеры запросов для программного доступа к данным аналитики Microsoft для коммерческого рынка.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: sayantanroy83
ms.author: sroy
ms.date: 3/08/2021
ms.openlocfilehash: 7d788448fb3f8a849f79e43fcb0737898f4c9e15
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102584015"
---
# <a name="sample-queries-for-programmatic-analytics"></a>Примеры запросов для программной аналитики

В этой статье приводятся примеры запросов для заказов, использования и отчетов для коммерческих рынков корпорации Майкрософт. Эти запросы можно использовать путем вызова конечной точки API [создания запроса отчета](analytics-programmatic-access.md#create-report-query-api) . При необходимости можно изменить вызов API [создания запроса отчета](analytics-programmatic-access.md#create-report-query-api) , чтобы добавить дополнительные столбцы, настроить период вычисления и добавить условия фильтра. Поддерживаемые периоды времени: шесть месяцев (6 мин), 12 месяцев (12 млн) и настраиваемый период времени.

Дополнительные сведения о именах, атрибутах и описаниях столбцов см. в следующих таблицах:

- [Таблица сведений о клиенте](customer-dashboard.md#customer-details-table)
- [Таблица сведений о заказах](orders-dashboard.md#orders-details-table)
- [Таблица сведений об использовании](usage-dashboard.md#usage-details-table)

## <a name="customers-report-queries"></a>Запросы клиентов отчета

Эти примеры запросов применяются к отчету о клиентах.

| **Описание запроса** | **Образец запроса** |
| --- | --- |
| Активные клиенты партнера до выбранной даты | `SELECT DateAcquired,CustomerCompanyName,CustomerId FROM ISVCustomer WHERE IsActive = 1` |
| Обработанные клиенты партнера до выбранной даты | `SELECT DateAcquired,CustomerCompanyName,CustomerId FROM ISVCustomer WHERE IsActive = 0` |
| Список новых клиентов из определенного географического региона за последние шесть месяцев | `SELECT DateAcquired,CustomerCompanyName,CustomerId FROM ISVCustomer WHERE DateAcquired <= ‘2020-06-30’ AND CustomerCountryRegion = ‘United States’` |
|||

## <a name="usage-report-queries"></a>Запросы отчетов об использовании

Эти примеры запросов применяются к отчету об использовании.

| **Описание запроса** | **Образец запроса** |
| --- | --- |
| Нормализованное использование виртуальной машины для типа лицензии Marketplace с оплатой по Azure за последние 6 мин | `SELECT MonthStartDate, NormalizedUsage FROM ISVUsage WHERE MarketplaceLicenseType = ‘Billed Through Azure’ AND OfferType NOT IN (‘Azure Applications’, ‘SaaS’) TIMESPAN LAST_6_MONTHS` |
| Использование необработанных виртуальных машин для типа лицензии Marketplace с оплатой по Azure за последние 12 млн | `SELECT MonthStartDate, RawUsage FROM ISVUsage WHERE MarketplaceLicenseType = ‘Billed Through Azure’ AND OfferType NOT IN (‘Azure Applications’, ‘SaaS’) TIMESPAN LAST_1_YEAR` |
| Нормализованное использование виртуальной машины для типа лицензии Marketplace "получить собственную лицензию" для последней 6 мин | `SELECT MonthStartDate, NormalizedUsage FROM ISVUsage WHERE MarketplaceLicenseType = ‘Bring Your Own License’ AND OfferType NOT IN (‘Azure Applications’, ‘SaaS’) TIMESPAN LAST_6_MONTHS` |
| Использование необработанных виртуальных машин для типа лицензии Marketplace с лицензией "получить собственную лицензию" для последней 6 мин | `SELECT MonthStartDate, RawUsage FROM ISVUsage WHERE MarketplaceLicenseType = ‘Bring Your Own License’ AND OfferType NOT IN (‘Azure Applications’, ‘SaaS’) TIMESPAN LAST_6_MONTHS` |
| На основе даты использования, ежедневного общего нормализованного использования и "оценочных расширенных расходов (PC/CC)" для платных планов за прошлый месяц | `SELECT UsageDate, NormalizedUsage, EstimatedExtendedChargePC FROM ISVUsage WHERE SKUBillingType = ‘Paid’ ORDER BY UsageDate DESC TIMESPAN LAST_MONTH` |
| На основе даты использования, ежедневного общего использования и "оценочных расширенных расходов (PC/CC)" для платных планов за прошлый месяц | `SELECT UsageDate, RawUsage, EstimatedExtendedChargePC FROM ISVUsage WHERE SKUBillingType = ‘Paid’ ORDER BY UsageDate DESC TIMESPAN LAST\_MONTH` |
| Для указанного имени предложения нормализованное использование виртуальной машины для типа лицензии Marketplace "с оплатой по Azure" для последней 6 мин | `SELECT OfferName, NormalizedUsage FROM ISVUsage WHERE MarketplaceLicenseType = ‘Billed Through Azure’ AND OfferName = ‘Example Offer Name’ TIMESPAN LAST_6_MONTHS` |
| Для указанного имени предложения лимитное использование для последней 6 мин | `SELECT OfferName, MeteredUsage FROM ISVUsage WHERE OfferName = ‘Example Offer Name’ AND OfferType IN (‘SaaS’, ‘Azure Applications’) TIMESPAN LAST_6_MONTHS` |
|||

## <a name="orders-report-queries"></a>Запросы отчетов о заказах

Эти образцы запросов применяются к отчету Orders.

| **Описание запроса** | **Образец запроса** |
| --- | --- |
| Отчет о заказах для типа лицензии Azure в качестве "корпоративного" для последней 6 мин | `SELECT OrderId, OrderPurchaseDate FROM ISVOrder WHERE AzureLicenseType = ‘Enterprise’ TIMESPAN LAST\_6\_MONTHS` |
| Отчет о заказах для типа лицензии Azure с оплатой по мере использования за последние 6 мин | `SELECT OrderId, OrderPurchaseDate FROM ISVOrder WHERE AzureLicenseType = ‘Pay as You Go’ TIMESPAN LAST_6_MONTHS` |
| Отчет о заказах для указанного имени предложения за последние 6 мин | `SELECT OrderId, OrderPurchaseDate FROM ISVOrder WHERE OfferName = ‘Example Offer Name’ TIMESPAN LAST_6_MONTHS` |
| Отчет о заказах для активных заказов за последние 6 мин | `SELECT OrderId, OrderPurchaseDate FROM ISVOrder WHERE OrderStatus = ‘Active’ TIMESPAN LAST_6_MONTHS` |
| Отчет о заказах для отмененных заказов за последние 6 мин | `SELECT OrderId, OrderPurchaseDate FROM ISVOrder WHERE OrderStatus = ‘Cancelled’ TIMESPAN LAST_6_MONTHS` |
|||

## <a name="next-steps"></a>Дальнейшие действия

- [Интерфейсы API для доступа к данным анализа коммерческих рынков](analytics-available-apis.md)
