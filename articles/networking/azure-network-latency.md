---
title: Статистика задержки приема-передачи для сети Azure | Документация Майкрософт
description: Сведения о статистике задержки приема-передачи между регионами Azure.
services: networking
author: nayak-mahesh
ms.service: virtual-network
ms.topic: article
ms.date: 12/07/2020
ms.author: kumud
ms.openlocfilehash: bc2d7bb7ba17a4a47fecf2144157f79f5367fca7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98059179"
---
# <a name="azure-network-round-trip-latency-statistics"></a>Статистика задержки приема-передачи для сети Azure

Azure постоянно отслеживает задержку (скорость) основных областей своей сети с помощью внутренних средств мониторинга, а также измерений, собираемых [ThousandEyes](https://thousandeyes.com), сторонней службы искусственного мониторинга.

## <a name="how-are-the-measurements-collected"></a>Как собираются измерения?

Измерения задержки собираются от агентов ThousandEyes, размещенных в облачных регионах Azure по всему миру, которые постоянно отправляют проверки сети между собой через 1-минутные интервалы. Статистика месячной задержки определяется усреднением собранных выборок за месяц.

## <a name="december-2020-round-trip-latency-figures"></a>Декабрь 2020. цифры задержки приема-передачи

Ниже приведено ежемесячное среднее время приема-передачи между регионами Azure за последние 30 дней (заканчивается 31 декабря 2020). [ThousandEyes](https://thousandeyes.com)поддерживают следующие измерения.

[![Статистика задержки между регионами Azure](media/azure-network-latency/azure-network-latency.png)](media/azure-network-latency/azure-network-latency.png#lightbox)

## <a name="next-steps"></a>Дальнейшие действия

Сведения о [регионах Azure](https://azure.microsoft.com/global-infrastructure/regions/).
