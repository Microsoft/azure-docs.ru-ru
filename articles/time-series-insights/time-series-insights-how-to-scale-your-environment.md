---
title: Как масштабировать среду в Azure Time Series Insights | Документация Майкрософт
description: Узнайте, как масштабировать среду службы "аналитика временных рядов Azure" с помощью портал Azure.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 09/29/2020
ms.custom: seodec18
ms.openlocfilehash: 0dadf523c5d17ffb91f4fefa71b52d1d1855c978
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91570245"
---
# <a name="how-to-scale-your-azure-time-series-insights-gen1-environment"></a>Как масштабировать среду Gen1 службы "аналитика временных рядов Azure"

> [!CAUTION]
> Эта статья посвящена службе "Аналитика временных рядов Azure" 1-го поколения.

В этой статье описывается, как изменить емкость среды службы "аналитика временных рядов Azure" с помощью [портал Azure](https://portal.azure.com). Емкость — множитель, применяемый к скорости входящих данных, емкости хранилища и затратам, связанным с выбранным номером SKU.

Портал Azure можно использовать для увеличения или уменьшения емкости в пределах ценовой категории SKU.

Однако изменение ценовой категории SKU запрещено. Например, среду с ценовой категорией SKU S1 нельзя преобразовать в среду S2 или наоборот.

## <a name="ga-limits"></a>Ограничения в общедоступной версии

[!INCLUDE [Azure Time Series Insights GA limits](../../includes/time-series-insights-ga-limits.md)]

## <a name="change-the-capacity-of-your-environment"></a>Изменение емкости среды

1. В портал Azure выберите среду службы "аналитика временных рядов Azure".

1. В меню среды службы "аналитика временных рядов Azure" выберите **Конфигурация хранилища**.

   [![Настройка емкости службы "аналитика временных рядов Azure"](media/scale-your-environment/scale-your-environment-configure.png)](media/scale-your-environment/scale-your-environment-configure.png#lightbox)

1. Настройте ползунок **емкости** соответственно требованиям скорости приема и емкости хранилища. Обратите внимание, что **скорость приема**, **емкость хранилища** и **оценочная стоимость** обновляются динамически, чтобы показать эффект изменения.

   [![Настройка среды с помощью ползунка емкости](media/scale-your-environment/scale-your-environment-slider.png)](media/scale-your-environment/scale-your-environment-slider.png#lightbox)

   Кроме того, можно ввести число множителя емкости в текстовом поле справа от ползунка.

1. Выберите **Сохранить**, чтобы масштабировать среду. Индикатор хода выполнения будет отображаться, пока изменения не зафиксируются.

1. Убедитесь, что Новая емкость [достаточна для предотвращения регулирования](time-series-insights-diagnose-and-solve-problems.md).

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения см. в обзоре [хранения в службе "аналитика временных рядов Azure](time-series-insights-concepts-retention.md)".

- Узнайте, как [настроить хранение данных в Azure Time Series Insights](time-series-insights-how-to-configure-retention.md).

- Узнайте о [планировании среды](time-series-insights-environment-planning.md).
