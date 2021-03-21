---
title: Межсервисный запрос между Azure Monitor и обозреватель данных Azure (Предварительная версия)
description: Запросите данные Azure обозреватель данных с помощью средств Log Analytics Azure, наоборот, чтобы присоединить и проанализировать все данные в одном месте.
author: osalzberg
ms.author: bwren
ms.reviewer: bwren
ms.topic: conceptual
ms.date: 06/12/2020
ms.openlocfilehash: 7a259af17943643e722633592e53f219726c4437
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102031214"
---
# <a name="cross-service-query---azure-monitor-and-azure-data-explorer-preview"></a>Кросс-Сервисный запрос — Azure Monitor и Azure обозреватель данных (Предварительная версия)
Создавайте запросы между службами между [Azure обозреватель данных](/azure/data-explorer/), [Application Insights](../app/app-insights-overview.md)и [log Analytics](../logs/data-platform-logs.md).
## <a name="azure-monitor-and-azure-data-explorer-cross-service-querying"></a>Azure Monitor и Azure обозреватель данных запросы между службами
Этот интерфейс позволяет [создавать запросы между службами между Azure обозреватель данных и Azure Monitor](/azure/data-explorer/query-monitor-data) , а также [создавать запросы между службами между Azure Monitor и обозреватель данных Azure](./azure-monitor-data-explorer-proxy.md).

Например, (запрос обозреватель данных Azure из Log Analytics):
```kusto
CustomEvents | where aField == 1
| join (adx("Help/Samples").TableName | where secondField == 3) on $left.Key == $right.key
```
Где внешний запрос запрашивает таблицу в рабочей области, а затем присоединяется к другой таблице в кластере Azure обозреватель данных (в данном случае — имя_кластера = Help, DatabaseName = Samples) с помощью новой функции "ADX ()", например, как можно сделать то же самое для запроса другой рабочей области из текста запроса.

> [!NOTE]
> * Возможность запрашивать Azure Monitor данных из Azure обозреватель данных или напрямую из клиентских средств Azure обозреватель данных или косвенно, выполняя запрос в кластере Azure обозреватель данных, находится в режиме предварительного просмотра.
> * Обратитесь к группе по [запросам обслуживания](mailto:adxproxy@microsoft.com) с любыми вопросами.

## <a name="query-exported-log-analytics-data-from-azure-blob-storage-account"></a>Запрос, экспортированный Log Analytics данных из учетной записи хранения BLOB-объектов Azure

Экспорт данных из Azure Monitor в учетную запись хранения Azure обеспечивает недорогое хранение и возможность перераспределения журналов в разные регионы.

Используйте обозреватель данных Azure для запроса данных, экспортированных из рабочих областей Log Analytics. После настройки поддерживаемые таблицы, отправленные из рабочих областей в учетную запись хранения Azure, будут доступны в качестве источника данных для Azure обозреватель данных. [Запрос экспортированных данных из Azure Monitor с помощью обозреватель данных Azure (Предварительная версия)](../logs/azure-data-explorer-query-storage.md).

[Запрос обозреватель данных Azure из потока хранилища](media\azure-data-explorer-query-storage\exported-data-query.png)

>[!tip] 
> * Чтобы экспортировать все данные из рабочей области Log Analytics в учетную запись хранения Azure или концентратор событий, используйте функцию экспорта данных рабочей области Log Analytics в журналах Azure Monitor. [См. раздел Экспорт данных рабочей области log Analytics в Azure Monitor (Предварительная версия)](/azure/data-explorer/query-monitor-data).

## <a name="next-steps"></a>Дальнейшие действия
См. также:
* [создавайте запросы между службами между Azure обозреватель данных и Azure Monitor](/azure/data-explorer/query-monitor-data). Запрос Azure Monitor данных из Azure обозреватель данных
* [создавайте запросы между службами между Azure Monitor и обозреватель данных Azure](./azure-monitor-data-explorer-proxy.md). Запрос данных обозреватель данных Azure от Azure Monitor
* [Log Analytics экспорт данных рабочей области в Azure Monitor (Предварительная версия)](/azure/data-explorer/query-monitor-data). Свяжите и запросите учетную запись хранилища BLOB-объектов Azure с Log Analytics экспортированными данными.