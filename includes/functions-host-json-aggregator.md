---
title: Включить файл
description: Включить файл
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 10/19/2018
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: c63cee1c451918569e9b7aa2e76209e8069d86ac
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92167664"
---
Указывает, сколько вызовов функций обрабатывается при [расчете метрик для Application Insights](../articles/azure-functions/configure-monitoring.md#configure-the-aggregator). 

```json
{
    "aggregator": {
        "batchSize": 1000,
        "flushTimeout": "00:00:30"
    }
}
```

|Свойство |По умолчанию  | Описание |
|---------|---------|---------| 
|batchSize|1000|Максимальное количество запросов, которое необходимо обработать.| 
|flushTimeout|00:00:30|Максимальный период времени, который необходимо обработать.| 

Вызовы функций обрабатываются, когда достигается одно из этих двух ограничений.
