---
title: Модель данных телеметрии Azure Application Insights — телеметрия событий | Документы Майкрософт
description: Модель данных Application Insights для телеметрии событий
ms.topic: conceptual
ms.date: 04/25/2017
ms.reviewer: sergkanz
ms.openlocfilehash: 69685afa14352a22b58bccbea342038e4273696e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87320618"
---
# <a name="event-telemetry-application-insights-data-model"></a>Телеметрия событий: модель данных Application Insights

Элементы телеметрии событий, которые можно создавать в [Application Insights](./app-insights-overview.md), представляют событие, произошедшее в приложении. Обычно это взаимодействие с пользователем, например нажатие кнопки или оформление заказа. Кроме того, это может быть событие жизненного цикла приложения, такое как инициализация или изменение конфигурации. 

Семантически события могут как коррелировать, так и не коррелировать с запросами. Однако при правильном использовании телеметрия событий важнее, чем запросы или трассировки. События представляют бизнес-данные телеметрии и должны быть предметом отдельной, менее агрессивной [выборки](./api-filtering-sampling.md).

## <a name="name"></a>Имя

Имя события. Чтобы обеспечить правильную группировку и значимость метрик, настройте в приложении создание небольшого количества имен отдельных событий. Например, не используйте отдельное имя для каждого созданного экземпляра события.

Максимальная длина: 512 символов

## <a name="custom-properties"></a>Пользовательские свойства

[!INCLUDE [application-insights-data-model-properties](../../../includes/application-insights-data-model-properties.md)]

## <a name="custom-measurements"></a>Пользовательские измерения

[!INCLUDE [application-insights-data-model-measurements](../../../includes/application-insights-data-model-measurements.md)]

## <a name="next-steps"></a>Дальнейшие действия

- В [этой статье](data-model.md) представлены типы данных и модель данных для Application Insights.
- [Написание пользовательской телеметрии событий](./api-custom-events-metrics.md#trackevent)
- Ознакомление с [платформами](./platforms.md), поддерживаемыми Application Insights.

