---
title: включить файл
description: включить файл
services: digital-twins
ms.service: digital-twins
ms.topic: include
ms.date: 11/11/2020
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.custom: include file
ms.openlocfilehash: 016ad0e11f3378dba887e0a235f235fa91e3aa03
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98109391"
---
### <a name="property-limits"></a>Ограничения свойств

Ограничения для свойств "аналитика временных рядов Azure" увеличены до 1 000 для горячего хранения и не имеют ограничения на количество свойств для холодного хранения. Предоставленные свойства событий имеют соответствующие столбцы JSON, CSV и диаграммы, которые можно просмотреть в [обозревателе Gen2 временных рядов Azure](../articles/time-series-insights/quickstart-explore-tsi.md).

| SKU | Максимальное число свойств |
| --- | --- |
| Gen2 (L1) | 1 000. свойства (столбцы) для горячего хранения и неограниченные для холодного хранения|
| Gen1 (S1) | 600 свойства (столбцы) |
| Gen1 (S2) | 800 свойства (столбцы) |

### <a name="streaming-ingestion"></a>Потоковый прием

* Для каждой среды существует не более двух [источников событий](../articles/time-series-insights/concepts-streaming-ingestion-event-sources.md) .

* Рекомендации и общие рекомендации для источников событий можно найти [здесь](../articles/time-series-insights/concepts-streaming-ingestion-event-sources.md#streaming-ingestion-best-practices) .

* По умолчанию служба "аналитика временных рядов Azure" Gen2 может принимать входящие данные с частотой **до 1 МБ в секунду (Мбит/с) на среду Gen2 "аналитика временных рядов Azure"**. Существуют дополнительные ограничения [для каждой секции центра](../articles/time-series-insights/concepts-streaming-ingress-throughput-limits.md#hub-partitions-and-per-partition-limits). Для отправки запроса в службу поддержки через портал Azure можно использовать скорости до 2 Мбит/с. Дополнительные сведения см. в статье [ограничения пропускной способности приема потоковой передачи](../articles/time-series-insights/concepts-streaming-ingress-throughput-limits.md).

### <a name="api-limits"></a>Ограничения API

Ограничения REST API для Gen2 Azure Time Series Insights указаны в [справочной документации по REST API](/rest/api/time-series-insights/preview#limits-1).