---
title: Переход от метрик Аналитика Службы хранилища к Azure Monitorным метрикам | Документация Майкрософт
description: Узнайте, как переходить от метрик Аналитика Службы хранилища (классические метрики) к метрикам в Azure Monitor.
author: normesta
ms.service: storage
ms.topic: conceptual
ms.date: 10/20/2020
ms.author: normesta
ms.reviewer: fryu
ms.subservice: common
ms.custom: monitoring
ms.openlocfilehash: ff4c3c5ae8629879fca6e9e683a9c77d0e2f144b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100596116"
---
# <a name="transition-to-metrics-in-azure-monitor"></a>Переход к метрикам в Azure Monitor

С **31 августа 2023** аналитика службы хранилища метрики, также называемые *классическими метриками* , будут прекращены. Дополнительные сведения см. в [официальном объявлении](https://azure.microsoft.com/updates/azure-storage-classic-metrics-will-be-retired-on-31-august-2023/). Если вы используете классические метрики, обязательно перейдите на использование метрик в Azure Monitor до этой даты. Эта статья поможет вам выполнить переход. 

## <a name="steps-to-complete-the-transition"></a>Шаги для завершения перехода

Для перехода на метрики в Azure Monitor рекомендуется следующий подход.

1. Узнайте о некоторых [основных различиях](#key-differences-between-classic-metrics-and-metrics-in-azure-monitor) между классическими метриками и метриками в Azure Monitor. 

2. Скомпилируйте список используемых в настоящее время классических метрик.

3. Укажите, [какие метрики в Azure Monitor](#metrics-mapping-between-old-metrics-and-new-metrics) предоставляют те же данные, что и используемые в настоящее время метрики. 
   
4. Создайте [диаграммы](/learn/modules/gather-metrics-blob-storage/2-viewing-blob-metrics-in-azure-portal) или [панели мониторинга](/learn/modules/gather-metrics-blob-storage/4-using-dashboards-in-the-azure-portal) для просмотра данных метрик.

   > [!NOTE]
   > Метрики в Azure Monitor включены по умолчанию, поэтому вам не нужно предпринимать никаких действий, чтобы начать сбор метрик. Однако для просмотра этих метрик необходимо создать диаграммы или панели мониторинга. 
 
5. Если вы создали правила генерации оповещений, основанные на классических метриках хранилища, [Создайте правила генерации оповещений](../../azure-monitor/alerts/alerts-overview.md) , основанные на метриках в Azure Monitor. 

6. После того как вы сможете увидеть все метрики в Azure Monitor, вы можете отключить классическое ведение журнала. 

<a id="key-differences-between-classic-metrics-and-metrics-in-azure-monitor"></a>

## <a name="classic-metrics-vs-metrics-in-azure-monitor"></a>Классические метрики и метрики в Azure Monitor

В этом разделе описаны некоторые ключевые различия между этими двумя платформами метрик.

Основное различие заключается в том, как управляются метрики. Классические метрики управляются службой хранилища Azure, а метрики в Azure Monitor управляются Azure Monitor. С помощью классических метрик служба хранилища Azure собирает значения метрик, объединяет их, а затем сохраняет в таблицах, расположенных в учетной записи хранения. Используя метрики в Azure Monitor, служба хранилища Azure отправляет данные метрик в серверную части Azure Monitor. Azure Monitor предоставляет единый интерфейс мониторинга, который включает данные из портал Azure, а также данные, которые принимаются. 

Классические метрики отправляются и хранятся в учетной записи хранения Azure. Метрики Azure Monitor можно отправлять в несколько расположений. Учетная запись хранения может быть одним из этих расположений, но она не требуется.  

Как только метрики поддерживаются, классические метрики предоставляют метрики **емкости** только для хранилища BLOB-объектов Azure. Метрики в Azure Monitor предоставляют метрики емкости для больших двоичных объектов, таблиц, файлов, очередей и хранилища класса Premium. Классические метрики предоставляют метрики **транзакций** для хранилища BLOB-объектов, таблиц, файлов Azure и очередей. Метрики в Azure Monitor добавить хранилище класса Premium в этот список.

Если действие в учетной записи не запускает метрику, для этой метрики в классической метрике будет отображаться нулевое значение (0). Метрики в Azure Monitor будут полностью опускать данные, что ведет к очистке отчетов. Например, при использовании классической метрики, если ошибки времени ожидания сервера не выводятся, `ServerTimeoutError` значение в таблице метрик задается равным 0. Azure Monitor не возвращает никаких данных при запросе значения метрики `Transactions` с измерением, `ResponseType` равным `ServerTimeoutError` . 

Дополнительные сведения о метриках в Azure Monitor см. в разделе [метрики в Azure Monitor](../../azure-monitor/essentials/data-platform-metrics.md).

<a id="metrics-mapping-between-old-metrics-and-new-metrics"></a>

## <a name="map-classic-metrics-to-metrics-in-azure-monitor"></a>Сопоставьте классические метрики с метриками в Azure Monitor

 Используйте эти таблицы, чтобы указать, какие метрики в Azure Monitor предоставляют те же данные, что и используемые в настоящее время метрики. 

**Метрики емкости**

| Классическая метрика | Метрика в Azure Monitor |
| ------------------- | ----------------- |
| `Capacity`            | `BlobCapacity` с измерением, `BlobType` равным `BlockBlob` или `PageBlob` |
| `ObjectCount`        | `BlobCount` с измерением, `BlobType` равным `BlockBlob` или `PageBlob` |
| `ContainerCount`      | `ContainerCount` |

> [!NOTE]
> Также есть несколько новых метрик емкости, которые не были доступны в качестве классических метрик. Полный список см. в разделе [метрики](../blobs/monitor-blob-storage-reference.md#metrics).

**Метрики транзакций**

| Классическая метрика | Метрика в Azure Monitor |
| ------------------- | ----------------- |
| `AnonymousAuthorizationError` | Транзакции с измерением, `ResponseType` равным `AuthorizationError` , и измерением, `Authentication` равным `Anonymous` |
| `AnonymousClientOtherError` | Транзакции с измерением, `ResponseType` равным `ClientOtherError` , и измерением, `Authentication` равным `Anonymous` |
| `AnonymousClientTimeoutError` | Транзакции с измерением, `ResponseType` равным `ClientTimeoutError` , и измерением, `Authentication` равным `Anonymous` |
| `AnonymousNetworkError` | Транзакции с измерением, `ResponseType` равным `NetworkError` , и измерением, `Authentication` равным `Anonymous` |
| `AnonymousServerOtherError` | Транзакции с измерением, `ResponseType` равным `ServerOtherError` , и измерением, `Authentication` равным `Anonymous` |
| `AnonymousServerTimeoutError` | Транзакции с измерением, `ResponseType` равным `ServerTimeoutError` , и измерением, `Authentication` равным `Anonymous` |
| `AnonymousSuccess` | Транзакции с измерением, `ResponseType` равным `Success` , и измерением, `Authentication` равным `Anonymous` |
| `AnonymousThrottlingError` | Транзакции с измерением, `ResponseType` равным `ClientThrottlingError` или `ServerBusyError` , и измерением, `Authentication` равным `Anonymous` |
| `AuthorizationError` | Транзакции с измерением, `ResponseType` равным `AuthorizationError` |
| `Availability` | `Availability` |
| `AverageE2ELatency` | `SuccessE2ELatency` |
| `AverageServerLatency` | `SuccessServerLatency` |
| `ClientOtherError` | Транзакции с измерением, `ResponseType` равным `ClientOtherError` |
| `ClientTimeoutError` | Транзакции с измерением, `ResponseType` равным `ClientTimeoutError` |
| `NetworkError` | Транзакции с измерением, `ResponseType` равным `NetworkError` |
| `PercentAuthorizationError` | Транзакции с измерением, `ResponseType` равным `AuthorizationError` |
| `PercentClientOtherError` | Транзакции с измерением, `ResponseType` равным `ClientOtherError` |
| `PercentNetworkError` | Транзакции с измерением, `ResponseType` равным `NetworkError` |
| `PercentServerOtherError` | Транзакции с измерением, `ResponseType` равным `ServerOtherError` |
| `PercentSuccess` | Транзакции с измерением, `ResponseType` равным `Success` |
| `PercentThrottlingError` | Транзакции с измерением, `ResponseType` равным `ClientThrottlingError` или `ServerBusyError` |
| `PercentTimeoutError` | Транзакции с измерением, `ResponseType` равным `ServerTimeoutError` или `ResponseType` равным `ClientTimeoutError` |
| `SASAuthorizationError` | Транзакции с измерением, `ResponseType` равным `AuthorizationError` , и измерением, `Authentication` равным `SAS` |
| `SASClientOtherError` | Транзакции с измерением, `ResponseType` равным `ClientOtherError` , и измерением, `Authentication` равным `SAS` |
| `SASClientTimeoutError` | Транзакции с измерением, `ResponseType` равным `ClientTimeoutError` , и измерением, `Authentication` равным `SAS` |
| `SASNetworkError` | Транзакции с измерением, `ResponseType` равным `NetworkError` , и измерением, `Authentication` равным `SAS` |
| `SASServerOtherError` | Транзакции с измерением, `ResponseType` равным `ServerOtherError` , и измерением, `Authentication` равным `SAS` |
| `SASServerTimeoutError` | Транзакции с измерением, `ResponseType` равным `ServerTimeoutError` , и измерением, `Authentication` равным `SAS` |
| `SASSuccess` | Транзакции с измерением, `ResponseType` равным `Success` , и измерением, `Authentication` равным `SAS` |
| `SASThrottlingError` | Транзакции с измерением, `ResponseType` равным `ClientThrottlingError` или `ServerBusyError` , и измерением, `Authentication` равным `SAS` |
| `ServerOtherError` | Транзакции с измерением, `ResponseType` равным `ServerOtherError` |
| `ServerTimeoutError` | Транзакции с измерением, `ResponseType` равным `ServerTimeoutError` |
| `Success` | Транзакции с измерением, `ResponseType` равным `Success` |
| `ThrottlingError` | `Transactions` с измерением, `ResponseType` равным `ClientThrottlingError` или `ServerBusyError`|
| `TotalBillableRequests` | `Transactions` |
| `TotalEgress` | `Egress` |
| `TotalIngress` | `Ingress` |
| `TotalRequests` | `Transactions` |

## <a name="next-steps"></a>Дальнейшие действия

* [Azure Monitor](../../azure-monitor/overview.md)