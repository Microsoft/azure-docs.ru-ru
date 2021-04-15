---
author: mrbullwinkle
ms.service: application-insights
ms.topic: include
ms.date: 11/09/2018
ms.author: mbullwin
ms.openlocfilehash: 91141e314bf276d9138157c8a9d85d5262ac5907
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "67185045"
---
Коллекция пользовательских измерений. Используйте эту коллекцию для создания отчета с именем измерения, связанного с элементом телеметрии. Типичные случаи использования:
- Размер полезных данных телеметрии зависимостей.
- Число элементов очереди, обрабатываемых телеметрией запросов.
- Время, требуемое клиенту для завершения шага в мастере телеметрии событий.

[Пользовательские измерения](https://analytics.applicationinsights.io/demo?q=H4sIAAAAAAAAA2WLOw6DMAyGd07hZoLeoRPqyMaGGAL8aiPhGCV2kKoeHsHK%2Bj1myyr8LoiaqfrT%2FkUCzRft4LMl8OUeL3LuLLIx%2BxR%2BIF8%2BtcoiNq2o78vgWuFthQaJ1AeGGxt6UlBwKxa1qQ6EpLhAfQAAAA%3D%3D&timespan=PT24H) можно запросить в аналитике приложений:

```
customEvents
| where customMeasurements != ""
| summarize avg(todouble(customMeasurements["Completion Time"]) * itemCount)
```

 > [!NOTE]
 > Пользовательские измерения связаны с элементом телеметрии, к которому они принадлежат. Они должны быть доступны для выборки с использованием элемента телеметрии, содержащего эти измерения. Чтобы отследить измерения со значением, которое не зависит от другого типа телеметрии, используйте [телеметрию метрик](../articles/azure-monitor/app/api-custom-events-metrics.md).

Максимальная длина ключа: 150
