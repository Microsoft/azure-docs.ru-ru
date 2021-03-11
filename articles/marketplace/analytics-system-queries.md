---
title: Список системных запросов
description: Сведения о системных запросах, которые можно использовать для программного получения аналитических данных о ваших предложениях в коммерческом магазине Майкрософт.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: sayantanroy83
ms.author: sroy
ms.date: 3/08/2021
ms.openlocfilehash: f2b5f7eb559e349947f88067a3d2ea53d99b7cbf
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102584006"
---
# <a name="list-of-system-queries"></a>Список системных запросов

Следующие системные запросы можно использовать в [API создания отчета](analytics-programmatic-access.md#create-report-api) непосредственно с помощью `QueryId` . Системные запросы подобны отчетам об экспорте в центре партнеров в течение шести месяцев (6 мин) вычислительного периода.

Дополнительные сведения о именах, атрибутах и описании столбцов см. в следующих статьях:

- [Панель мониторинга клиентов в коммерческой аналитике Marketplace](customer-dashboard.md#customer-details-table)
- [Панель мониторинга заказов в коммерческой аналитике Marketplace](orders-dashboard.md#orders-details-table)
- [Панель мониторинга использования в коммерческой аналитике Marketplace](usage-dashboard.md#usage-details-table)
- [Панель мониторинга Marketplace Insights в коммерческой аналитике Marketplace](insights-dashboard.md#marketplace-insights-details-table)

В следующих разделах представлены запросы отчетов для различных отчетов.

## <a name="customers-report-query"></a>Запрос отчета о клиентах

**Описание отчета**: отчет о клиентах за последние 6 мин

**QueryID**:  `b9df4929-073f-4795-b0cb-a2c81b11e28d`

**Запрос отчета**:

`SELECT MarketplaceSubscriptionId,DateAcquired,DateLost,ProviderName,ProviderEmail,FirstName,LastName,Email,CustomerCompanyName,CustomerCity,CustomerPostalCode,CustomerCommunicationCulture,CustomerCountryRegion,AzureLicenseType,PromotionalCustomers,CustomerState,CommerceRootCustomer,CustomerId,BillingAccountId,ID  FROM ISVCustomer TIMESPAN LAST_6_MONTHS`

## <a name="orders-report-query"></a>Запрос отчета о заказах

Отчет **Описание**: отчет о заказах для последнего 6 мин

**QueryID**:  `fd0f299c-5a1c-4929-9f48-bfc6cc44355d`

**Запрос отчета**:

`SELECT MarketplaceSubscriptionId,MonthStartDate,OfferType,AzureLicenseType,MarketplaceLicenseType,Sku,CustomerCountry,IsPreviewSKU,OrderId,OrderQuantity,CloudInstanceName,IsNewCustomer,OrderStatus,OrderCancelDate,CustomerCompanyName,CustomerName,OrderPurchaseDate,OfferName,TrialEndDate,CustomerId,BillingAccountId FROM ISVOrder TIMESPAN LAST_6_MONTHS`

## <a name="usage-report-queries"></a>Запросы отчетов об использовании

**Описание отчета**: нормализованный отчет об использовании виртуальной машины за последние 6 мин

**QueryID**:  `2c6f384b-ad52-4aed-965f-32bfa09b3778`

**Запрос отчета**:

`SELECT MarketplaceSubscriptionId,MonthStartDate,OfferType,AzureLicenseType,MarketplaceLicenseType,SKU,CustomerCountry,IsPreviewSKU,SKUBillingType,IsInternal,VMSize,CloudInstanceName,ServicePlanName,OfferName,DeploymentMethod,CustomerName,CustomerCompanyName,UsageDate,IsNewCustomer,CoreSize,TrialEndDate,CustomerCurrencyCC,PriceCC,PayoutCurrencyPC,EstimatedPricePC,UsageReference,UsageUnit,CustomerId,BillingAccountId,NormalizedUsage,EstimatedExtendedChargeCC,EstimatedExtendedChargePC FROM ISVUsage WHERE OfferType IN ('vm core image', 'Virtual Machine Licenses', 'multisolution') TIMESPAN LAST_6_MONTHS`

**Описание отчета**: необработанный отчет об использовании виртуальной машины за последние 6 мин

**QueryID**:  `3f19fb95-5bc4-4ee0-872e-cedd22578512`

**Запрос отчета**:

`SELECT MarketplaceSubscriptionId,MonthStartDate,OfferType,AzureLicenseType,MarketplaceLicenseType,SKU,CustomerCountry,IsPreviewSKU,SKUBillingType,IsInternal,VMSize,CloudInstanceName,ServicePlanName,OfferName,DeploymentMethod,CustomerName,CustomerCompanyName,UsageDate,IsNewCustomer,CoreSize,TrialEndDate,CustomerCurrencyCC,PriceCC,PayoutCurrencyPC,EstimatedPricePC,UsageReference,UsageUnit,CustomerId,BillingAccountId,RawUsage,EstimatedExtendedChargeCC,EstimatedExtendedChargePC FROM ISVUsage WHERE OfferType IN ('vm core image', 'Virtual Machine Licenses', 'multisolution') TIMESPAN LAST_6_MONTHS`

**Описание отчета**: отчет о лимитном использовании для последней 6 мин

**QueryID**:  `f0c4927f-1f23-4c99-be4a-1371a5a9a086`

**Запрос отчета**:

`SELECT MarketplaceSubscriptionId,MonthStartDate,OfferType,AzureLicenseType,MarketplaceLicenseType,SKU,CustomerCountry,IsPreviewSKU,SKUBillingType,IsInternal,VMSize,CloudInstanceName,ServicePlanName,OfferName,DeploymentMethod,CustomerName,CustomerCompanyName,UsageDate,IsNewCustomer,CoreSize,TrialEndDate,CustomerCurrencyCC,PriceCC,PayoutCurrencyPC,EstimatedPricePC,UsageReference,UsageUnit,CustomerId,BillingAccountId,MeteredUsage,EstimatedExtendedChargeCC,EstimatedExtendedChargePC FROM ISVUsage WHERE OfferType IN ('SaaS', 'Azure Applications') TIMESPAN LAST_6_MONTHS`

## <a name="marketplace-insights-report-query"></a>Запрос к отчету Marketplace Insights

**Описание отчета**: отчет Marketplace Insights за последние 6 мин

**QueryID**:  `6fd7624b-aa9f-42df-a61d-67d42fd00e92`

**Запрос отчета**:

`Date,OfferName,ReferralDomain,CountryName,PageVisits,GetItNow,ContactMe,TestDrive,FreeTrial FROM ISVMarketplaceInsights TIMESPAN LAST_6_MONTHS`

## <a name="next-steps"></a>Дальнейшие действия

- [Интерфейсы API для доступа к данным анализа коммерческих рынков](analytics-available-apis.md)
- [Пример приложения для доступа к данным анализа коммерческих рынков](analytics-sample-application.md)
