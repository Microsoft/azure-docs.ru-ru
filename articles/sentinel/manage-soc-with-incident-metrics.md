---
title: Более эффективное управление SOC с помощью метрик инцидентов в Azure Sentinel | Документация Майкрософт
description: Используйте сведения на экране и книге метрик инцидента Azure Sentinel, чтобы управлять безопасностью центра обработки данных (SOC).
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/11/2020
ms.author: yelevin
ms.openlocfilehash: 408913fed864ee5f966b96c81afbfee4b2dc8678
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94660735"
---
# <a name="manage-your-soc-better-with-incident-metrics"></a>Улучшенное управление SOC с помощью метрик инцидентов

> [!IMPORTANT]
> Функции метрик инцидентов в настоящее время доступны в общедоступной предварительной версии.
> Эти функции предоставляются без соглашения об уровне обслуживания и не рекомендуются для рабочих нагрузок.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Как менеджер по работе с системой безопасности (SOC), вам необходимо получить общие метрики и меры эффективности, чтобы оценить производительность команды. Вы захотите просмотреть операции с инцидентами с течением времени, используя множество различных критериев, таких как серьезность, MITRE тактика, среднее время рассмотрения, среднее время разрешения и многое другое. Azure Sentinel теперь предоставляет доступ к этим данным в новой таблице и схеме **секуритинЦидент** в log Analytics и в сопутствующей книге **эффективности операций безопасности** . Вы сможете визуализировать производительность команды с течением времени и использовать эту аналитику для повышения эффективности. Вы также можете писать и использовать собственные запросы ККЛ к таблице инцидентов для создания настраиваемых книг, соответствующих конкретным потребностям аудита и ключевым показателям эффективности.

## <a name="use-the-security-incidents-table"></a>Использование таблицы "инциденты безопасности"

Таблица **секуритинЦидент** встроена в Azure Sentinel. Он будет найден вместе с другими таблицами в коллекции **секуритинсигхтс** в разделе **logs**. Вы можете запросить его как любую другую таблицу в Log Analytics.

:::image type="content" source="./media/manage-soc-with-incident-metrics/security-incident-table.png" alt-text="Таблица инцидентов безопасности":::

Каждый раз при создании или обновлении инцидента в таблицу будет добавлена новая запись журнала. Это позволяет относить изменения, внесенные в инциденты, и обеспечивает еще более эффективные метрики SOC, но при формировании запросов для этой таблицы необходимо быть учитыватьм, так как может потребоваться удалить дубликаты записей для инцидента (в зависимости от конкретного выполняемого запроса). 

Например, если требуется возвратить список всех инцидентов, отсортированных по номеру инцидента, но только последний журнал для каждого инцидента, это можно сделать с помощью [оператора суммирования](/azure/data-explorer/kusto/query/summarizeoperator) ККЛ с `arg_max()` [статистической функцией](/azure/data-explorer/kusto/query/arg-max-aggfunction).


```Kusto
SecurityIncident
| summarize arg_max(LastModifiedTime, *) by IncidentNumber
```
### <a name="more-sample-queries"></a>Дополнительные примеры запросов

Среднее время закрытия:
```Kusto
SecurityIncident
| summarize arg_max(TimeGenerated,*) by IncidentNumber 
| extend TimeToClosure =  (ClosedTime - CreatedTime)/1h
| summarize 5th_Percentile=percentile(TimeToClosure, 5),50th_Percentile=percentile(TimeToClosure, 50), 
  90th_Percentile=percentile(TimeToClosure, 90),99th_Percentile=percentile(TimeToClosure, 99)
```

Среднее время рассмотрения:
```Kusto
SecurityIncident
| summarize arg_max(TimeGenerated,*) by IncidentNumber 
| extend TimeToTriage =  (FirstModifiedTime - CreatedTime)/1h
| summarize 5th_Percentile=max_of(percentile(TimeToTriage, 5),0),50th_Percentile=percentile(TimeToTriage, 50), 
  90th_Percentile=percentile(TimeToTriage, 90),99th_Percentile=percentile(TimeToTriage, 99) 
```

## <a name="security-operations-efficiency-workbook"></a>Книга эффективности операций безопасности

Чтобы дополнить таблицу **секуритинЦидентс** , мы предоставили готовый шаблон **эффективного использования операций безопасности** , который можно использовать для мониторинга операций SoC. Книга содержит следующие метрики: 
- Инцидент создан с течением времени 
- Инциденты, созданные при закрытии классификации, серьезности, владельца и состояния 
- Среднее время рассмотрения 
- Среднее время до закрытия 
- Инциденты, созданные по серьезности, владельцу, состоянию, продукту и тактике с течением времени 
- Время на рассмотрение процентили 
- Время до закрытия процентили 
- Среднее время рассмотрения каждого владельца 
- Недавние действия 
- Недавние закрывающие классификации  

Этот новый шаблон книги можно найти, выбрав пункт **книги** в меню навигации меток Azure и выбрав вкладку **шаблоны** . Выберите в галерее **эффективность операций безопасности** и щелкните одну из кнопок **просмотреть сохраненную книгу** и **просмотреть шаблон** .

:::image type="content" source="./media/manage-soc-with-incident-metrics/security-incidents-workbooks-gallery.png" alt-text="Коллекция книг инцидентов безопасности":::

:::image type="content" source="./media/manage-soc-with-incident-metrics/security-operations-workbook-1.png" alt-text="Книга &quot;инциденты безопасности&quot; завершена":::

Шаблон можно использовать для создания собственных настраиваемых книг, соответствующих вашим потребностям.

## <a name="securityincidents-schema"></a>Схема СекуритинЦидентс

[!INCLUDE [SecurityIncidents schema](../../includes/sentinel-schema-security-incident.md)]

## <a name="next-steps"></a>Дальнейшие действия

- Чтобы начать работу с Azure Sentinel, вам нужна подписка на Microsoft Azure. Если у вас нет подписки, вы можете зарегистрироваться для получения [бесплатной пробной версии](https://azure.microsoft.com/free/).
- Узнайте, как [подключить данные к Azure Sentinel](quickstart-onboard.md), и [отслеживайте свои данные и потенциальные угрозы](quickstart-get-visibility.md).