---
title: Время простоя, соглашение об уровне обслуживания и непростости — Application Insights
description: Вычислите и сообщайте об уровне обслуживания для веб-тестов с помощью одной области прозрачности по ресурсам Application Insights и подпискам Azure.
ms.topic: conceptual
ms.date: 02/8/2021
ms.openlocfilehash: d225627a27bffd9088956e5aee37ca543e528d4a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101714057"
---
# <a name="downtime-sla-and-outages-workbook"></a>Книга по простоям, сбоям и Соглашению об уровне обслуживания

Представляем простой способ вычислить и сообщить о соглашении об уровне обслуживания для веб-тестов через единую панель прозрачности по ресурсам Application Insights и подпискам Azure. Отчет о простоях и сбоях предоставляет мощные готовые запросы и визуализации данных для улучшения понимания возможностей подключения клиента, типичного времени ответа приложения и длительного времени.

Шаблон книги SLA доступен в коллекции книг в Application Insights ресурсе или на вкладке доступность, щелкнув **отчеты об уровне обслуживания** в верхней части страницы.
:::image type="content" source="./media/sla-report/availability.png" alt-text="Снимок экрана вкладки &quot;доступность&quot; с выделенными отчетами об уровне обслуживания." lightbox="./media/sla-report/availability.png":::

:::image type="content" source="./media/sla-report/workbook-gallery.png" alt-text="Снимок экрана коллекции книг с выделенной книгой &quot;простой&quot; и &quot;время простоя&quot;." lightbox ="./media/sla-report/workbook-gallery.png":::

## <a name="parameter-flexibility"></a>Гибкость параметров

Параметры, заданные в книге, влияют на остальную часть отчета.

:::image type="content" source="./media/sla-report/outages.png" alt-text=" Снимок экрана: вкладка &quot;параметры сбоя/обслуживания&quot; в книге &quot;время простоя&quot;." lightbox ="./media/sla-report/outages.png":::

`Subscriptions``App Insights Resources`Параметры, и `Web Test` определяют параметры ресурсов высокого уровня. Эти параметры основываются на запросах log Analytics и используются в каждом запросе отчета.

`Failure Threshold` и `Outage Window` позволяют определить собственные критерии для сбоя службы, например критерии для предупреждения о доступности App Insights на основе счетчика неудачных расположений за выбранный период. Типичное пороговое значение — три расположения в течение пяти минут.

`Maintenance Period` позволяет выбрать типичную частоту обслуживания и `Maintenance Window` является селектором DateTime для примера периода обслуживания. Все данные, происходящие за определенный период, будут игнорироваться в результатах.

`Availability Target 9s` Указывает целевую цель 9S из двух 9S на пять 9s.

## <a name="overview-page"></a>Страница "Обзор"

На странице Обзор содержатся общие сведения об общем уровне обслуживания (за исключением периодов обслуживания, если они определены), сквозных экземпляров сбоя и времени простоя приложений. Экземпляры сбоя определяются в том случае, если тест начинается со сбоя до тех пор, пока он не будет успешным в зависимости от параметров сбоя. Если тест начинается со сбоя в 8:00 AM и снова завершается с 10:00 AM, то весь период данных считается одним и тем же сбоем.

:::image type="content" source="./media/sla-report/overview.gif" alt-text=" GIF страницы обзора, на которой показана Обзорная таблица в ходе тестирования." lightbox="./media/sla-report/overview.gif":::

Вы также можете исследовать наиболее длительный сбой, произошедший за отчетный период.

Некоторые тесты можно связать с их Application Insightsным ресурсом для дальнейшего изучения, но это возможно только в [ресурсе Application Insights на основе рабочей области](create-workspace-resource.md).

## <a name="downtime-outages-and-failures"></a>Время простоя, простои и сбои

Вкладка простои **и простои** содержит сведения об общем количестве невыполненных экземпляров и общем времени, разбитом на тесты. На вкладке " **сбои по расположению** " имеется географическая схема неудачных расположений тестирования, помогающая определить потенциальные области подключения.

:::image type="content" source="./media/sla-report/outages-failures.gif" alt-text=" Вкладка &quot;GIF&quot; и &quot;время простоя&quot; и &quot;сбой&quot; по вкладке &quot;расположение&quot; в книге &quot;время простоя&quot;." lightbox="./media/sla-report/outages-failures.gif":::

## <a name="edit-the-report"></a>Редактирование отчета

Отчет можно изменить как любую другую [книгу Azure Monitor](../visualize/workbooks-overview.md). Вы можете настроить запросы или визуализации в соответствии с потребностями вашей команды.

:::image type="content" source="./media/sla-report/edit.gif" alt-text=" GIF, нажав кнопку изменить, чтобы изменить визуализацию на круговую диаграмму." lightbox="./media/sla-report/edit.gif":::

### <a name="log-analytics"></a>Log Analytics

Все запросы могут выполняться в [log Analytics](../logs/log-analytics-overview.md) и использоваться в других отчетах или на панелях мониторинга. Удалите ограничение параметра и повторно используйте базовый запрос.

:::image type="content" source="./media/sla-report/logs.gif" alt-text=" GIF-файл запроса журнала." lightbox="./media/sla-report/logs.gif":::

## <a name="access-and-sharing"></a>Доступ и общий доступ

Отчет может использоваться совместно с командами, лидерами или закрепленными на панели мониторинга для дальнейшего использования. Пользователь должен иметь разрешение на чтение или доступ к ресурсу Application Insights, в котором хранится фактическая книга.

:::image type="content" source="./media/sla-report/share.png" alt-text=" Снимок экрана: общий доступ к этому шаблону." lightbox= "./media/sla-report/share.png":::

## <a name="next-steps"></a>Дальнейшие действия

- [Log Analytics советы по оптимизации запросов](../logs/query-optimization.md).
- Узнайте, как [создать диаграмму в книгах](../visualize/workbooks-chart-visualizations.md).
- Узнайте, как отслеживать доступность веб-сайта с помощью [тестов доступности](monitor-web-app-availability.md).