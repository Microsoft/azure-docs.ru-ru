---
title: Использование аналитики API в службе управления API Azure | Документация Майкрософт
description: Используйте аналитику в службе управления API Azure, чтобы понять и классифицировать использование API-интерфейсов и производительности API.
author: dlepow
ms.service: api-management
ms.topic: article
ms.date: 11/24/2020
ms.author: apimpm
ms.openlocfilehash: ca7bd70bbf99a6d0079717a7a02328b11528d2e0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96841511"
---
# <a name="get-api-analytics-in-azure-api-management"></a>Получение аналитики API в службе управления API Azure

Служба управления API Azure предоставляет встроенную аналитику для API-интерфейсов. Анализируйте использование и производительность API-интерфейсов в экземпляре управления API в нескольких измерениях, в том числе:

* время;
* Географический регион
* Программные интерфейсы
* Операции API
* Продукты
* Подписки
* Пользователи
* Requests

:::image type="content" source="media/howto-use-analytics/analytics-report-portal.png" alt-text="Аналитика временной шкалы на портале":::

Используйте аналитику для высокого уровня мониторинга и устранения неполадок ваших интерфейсов API. Дополнительные функции мониторинга, включая метрики практически в реальном времени и журналы ресурсов для диагностики и аудита, см. в разделе [учебник. Мониторинг опубликованных API](api-management-howto-use-azure-monitor.md).

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="analytics---portal"></a>Аналитика — портал

Используйте портал Azure, чтобы быстро просмотреть аналитические данные об экземпляре службы управления API.

1. Перейдите к экземпляру Управления API на [портале Azure](https://portal.azure.com). 
1. В меню слева в разделе **мониторинг** выберите **аналитика**.

    :::image type="content" source="media/howto-use-analytics/monitoring-menu-analytics.png" alt-text="Выбор аналитики для экземпляра службы управления API на портале":::  
1. Выберите диапазон времени для данных или введите настраиваемый диапазон времени.
1. Выберите категорию отчета для данных аналитики, например " **временная шкала**", " **География**" и т. д.
1. При необходимости можно отфильтровать отчет по одной или нескольким дополнительным категориям.

## <a name="analytics---rest-api"></a>Аналитика — REST API

Используйте операции с [отчетами](/rest/api/apimanagement/2019-12-01/reports) в REST API управления API для получения и фильтрации данных аналитики для экземпляра службы управления API.

Доступные операции возвращают записи отчетов по API, географию, операциям API, продукту, запросу, подписке, времени или пользователю.

## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения о Azure Monitor функциях в управлении API см. в разделе [учебник. Мониторинг опубликованных API-интерфейсов.](api-management-howto-use-azure-monitor.md)
* Подробные сведения о ведении журналов и мониторинге HTTP см. в статье [мониторинг API-интерфейсов с помощью управления API Azure, концентраторов событий и моесиф](api-management-log-to-eventhub-sample.md).
* Сведения об интеграции [управления API Azure с azure Application Insights](api-management-howto-app-insights.md).