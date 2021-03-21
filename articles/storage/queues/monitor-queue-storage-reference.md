---
title: Справочник по данным мониторинга хранилища очередей Azure
description: Справочник по журналам и метрикам для мониторинга данных из хранилища очередей Azure.
author: normesta
services: azure-monitor
ms.author: normesta
ms.date: 10/02/2020
ms.topic: reference
ms.service: azure-monitor
ms.subservice: logs
ms.custom: monitoring
ms.openlocfilehash: 95f20737b044140fe12ea939e71cd2397cb4826d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100576684"
---
# <a name="azure-queue-storage-monitoring-data-reference"></a>Справочник по данным мониторинга хранилища очередей Azure

Сведения о сборе и анализе данных мониторинга для службы хранилища Azure см. на странице [Мониторинг службы хранилища Azure](monitor-queue-storage.md).

## <a name="metrics"></a>Метрики

В следующих таблицах перечислены метрики платформы, собранные для службы хранилища Azure:

### <a name="capacity-metrics"></a>Метрики емкости

Значения метрик емкости обновляются ежедневно (до 24 часов). Интервал времени определяет интервал, за который представлены значения метрик. Поддерживаемый интервал времени для всех метрик емкости — один час (PT1H).

Служба хранилища Azure предоставляет следующие метрики емкости в Azure Monitor.

#### <a name="account-level-capacity-metrics"></a>Метрики емкости на уровне учетной записи

[!INCLUDE [Account-level capacity metrics](../../../includes/azure-storage-account-capacity-metrics.md)]

#### <a name="queue-storage-metrics"></a>Метрики хранилища очередей

В этой таблице показаны [метрики хранилища очередей](../../azure-monitor/essentials/metrics-supported.md#microsoftstoragestorageaccountsqueueservices).

| Метрика | Описание |
| ------------------- | ----------------- |
| **QueueCapacity** | Объем хранилища очередей, используемого учетной записью хранения. <br><br> Модульных `Bytes` <br> Тип агрегирования: `Average` <br> Пример значения: `1024` |
| **QueueCount** | Количество очередей в учетной записи хранения. <br><br> Модульных `Count` <br> Тип агрегирования: `Average` <br> Пример значения: `1024` |
| **QueueMessageCount** | Приблизительное количество сообщений в очереди в учетной записи хранения. <br><br> Модульных `Count` <br> Тип агрегирования: `Average` <br> Пример значения: `1024` |

### <a name="transaction-metrics"></a>Метрики транзакций

Метрики транзакций создаются при каждом запросе к учетной записи хранения, отправляемом из службы хранилища Azure в Azure Monitor. Если учетная запись хранения неактивна, данные на основе метрик транзакций за этот период будут отсутствовать. Все метрики транзакций доступны на уровне службы хранилища учетной записи и очереди. Интервал времени определяет интервал, за который представлены значения метрик. Поддерживаемые интервалы времени для всех метрик транзакций — PT1H и PT1M.

[!INCLUDE [Transaction metrics](../../../includes/azure-storage-account-transaction-metrics.md)]

<a id="metrics-dimensions"></a>

## <a name="metrics-dimensions"></a>Измерения метрик

Служба хранилища Azure поддерживает следующие измерения метрик в Azure Monitor.

[!INCLUDE [Metrics dimensions](../../../includes/azure-storage-account-metrics-dimensions.md)]

## <a name="resource-logs-preview"></a>Журналы ресурсов (предварительная версия)

> [!NOTE]
> Журналы службы хранилища Azure в Azure Monitor предоставляются в общедоступной предварительной версии. Они также доступны для предварительного тестирования во всех регионах общедоступного облака. Эта предварительная версия включает журналы для больших двоичных объектов (в том числе Azure Data Lake Storage 2-го поколения), файлов, очередей, таблиц, учетных записей хранения категории "Премиум" общего назначения версии 1 и учетных записей хранения общего назначения версии 2. Классические учетные записи хранения не поддерживаются.

В следующей таблице перечислены свойства журналов ресурсов службы хранилища Azure при их сборе в журналах Azure Monitor или службе хранилища Azure. Свойства содержат сведения об операции, службе и типе авторизации, которые использовались для выполнения операции.

### <a name="fields-that-describe-the-operation"></a>Поля со сведениями об операции

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-logs-properties-operation.md)]

### <a name="fields-that-describe-how-the-operation-was-authenticated"></a>Поля, описывающие аутентификацию операции

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-logs-properties-authentication.md)]

### <a name="fields-that-describe-the-service"></a>Поля со сведениями о службе

[!INCLUDE [Account level capacity metrics](../../../includes/azure-storage-logs-properties-service.md)]

## <a name="see-also"></a>См. также раздел

- Описание мониторинга хранилища очередей Azure см. в статье [мониторинг хранилища очередей Azure](monitor-queue-storage.md) .
- Подробные сведения о мониторинге ресурсов Azure см. в статье [Мониторинг ресурсов Azure с помощью Azure Monitor](../../azure-monitor/essentials/monitor-azure-resource.md).
