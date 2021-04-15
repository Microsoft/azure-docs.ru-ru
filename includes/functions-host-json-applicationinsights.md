---
title: Включить файл
description: Включить файл
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 10/19/2018
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: 218e98e65c7c78272f32f75a0fdb93e4d87e6948
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92167731"
---
Управляет [функцией выборки в Application Insights](../articles/azure-functions/configure-monitoring.md#configure-sampling).

```json
{
    "applicationInsights": {
        "sampling": {
          "isEnabled": true,
          "maxTelemetryItemsPerSecond" : 5
        }
    }
}
```

|Свойство  |По умолчанию | Описание |
|---------|---------|---------| 
|isEnabled|true|Включает или отключает выборку.| 
|maxTelemetryItemsPerSecond|5|Пороговое значение, при котором начинается выборка.| 
