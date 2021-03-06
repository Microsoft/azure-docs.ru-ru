---
title: Планирование управления затратами для Azure ExpressRoute
description: Узнайте, как планировать и управлять затратами для Azure ExpressRoute с помощью анализа затрат в портал Azure.
author: duongau
ms.author: duau
ms.custom: subject-cost-optimization
ms.service: expressroute
ms.topic: how-to
ms.date: 11/30/2020
ms.openlocfilehash: de8405477611d62b8a46e8b6b645887cc4d30099
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98784247"
---
# <a name="plan-and-manage-costs-for-azure-expressroute"></a>Планирование и управление затратами для Azure ExpressRoute

В этой статье описывается планирование и управление затратами для ExpressRoute. Сначала вы используете калькулятор цен Azure, чтобы помочь спланировать затраты ExpressRoute перед добавлением ресурсов для службы для оценки затрат. После добавления ресурсов Azure ознакомьтесь с оценочными затратами. 

После начала использования ресурсов ExpressRoute используйте функции управления затратами для настройки бюджетов и отслеживания затрат. Вы также можете просматривать прогнозные затраты и выявление тенденций расходов для выявления областей, в которых может потребоваться выполнить действия. 

Помните, что стоимостью ExpressRoute является только часть ежемесячных затрат в счете Azure. Хотя в этой статье объясняется, как спланировать и управлять затратами ExpressRoute, вы оплачиваете все службы и ресурсы Azure, используемые в вашей подписке Azure, включая сторонние службы.

## <a name="prerequisites"></a>Предварительные условия

Анализ затрат в службе "Управление затратами" поддерживает большинство типов учетных записей Azure, но не все из них. Полный список поддерживаемых типов учетных записей см. в статье [Understand Cost Management data](../cost-management-billing/costs/understand-cost-mgt-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) (Интерпретация данных службы "Управление затратами"). Для просмотра данных о затратах требуется по крайней мере доступ на чтение для учетной записи Azure. 

Дополнительные сведения о назначении доступа к данным службы "Управление затратами" Azure см. в [этой статье](../cost-management-billing/costs/assign-access-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

## <a name="local-vs-standard-vs-premium"></a>Сравнение локальных и стандартных и расширенных возможностей

ExpressRoute имеет три разных SKU канала: [*Local*](./expressroute-faqs.md#expressroute-local), *Standard* и [*Premium*](./expressroute-faqs.md#expressroute-premium). Способ оплаты за использование ExpressRoute зависит от трех типов SKU. При использовании локального номера SKU вы автоматически платите с неограниченным тарифным планом. При использовании SKU "Стандартный" и "Премиум" можно выбрать план данных с лимитом или без ограничений. Все данные входящих данных предоставляются бесплатно, за исключением случаев использования надстройки Global Reach. Важно понимать, какие типы SKU и план данных лучше подходят для вашей рабочей нагрузки, чтобы оптимизировать затраты и бюджет.

## <a name="estimate-costs"></a>Оценка затрат

Используйте [Калькулятор цен Azure](https://azure.microsoft.com/pricing/calculator/) для оценки затрат перед созданием канала ExpressRoute. 

1. Слева выберите **сеть**, а затем выберите **Azure ExpressRoute** , чтобы начать работу. 

1. Выберите соответствующую *зону* в зависимости от расположения пиринга. Чтобы выбрать соответствующую *зону* в раскрывающемся списке, обратитесь к [поставщикам услуг подключения ExpressRoute](./expressroute-locations-providers.md#partners) . 

1. Затем выберите *SKU*, *скорость канала* и *план данных* , который требуется оценить. 

1. В поле *дополнительной передачи исходящих данных* введите оценку объема исходящих данных, которые можно использовать в течение месяца, в ГБ. 

1. Наконец, можно добавить *Global REACH надстройку* в оценку.

На следующем снимке экрана показана оценка затрат с помощью калькулятора:

:::image type="content" source="media/plan-manage-cost/capacity-calculator-cost-estimate.png" alt-text="Оценка затрат ExpressRoute в калькуляторе Azure":::

Дополнительные сведения см. на странице [цен на Azure ExpressRoute](https://azure.microsoft.com/pricing/details/expressroute/).

### <a name="expressroute-gateway-estimated-cost"></a>Оценочная стоимость шлюза ExpressRoute

Если вы используете шлюз ExpressRoute, чтобы связать виртуальную сеть с каналом ExpressRoute, выполните следующие действия, чтобы оценить затраты на использование шлюза.

1. В левой части экрана выберите **сети**, а затем — **VPN-шлюз** , чтобы начать. 

1. Выберите *регион* для шлюза, а затем измените *Тип* на **шлюзы ExpressRoute**.

1. Выберите *тип шлюза* из раскрывающегося списка.

1. Введите *часы шлюза*. (720 часов = 30 дней)

## <a name="understand-the-full-billing-model-for-expressroute"></a>Сведения о модели полного выставления счетов для ExpressRoute

ExpressRoute работает в инфраструктуре Azure, которая при развертывании нового ресурса выполняет накопление затрат вместе с ExpressRoute. Важно понимать, что дополнительная инфраструктура может начислять затраты. Необходимо управлять этими затратами при внесении изменений в развернутые ресурсы. 

### <a name="costs-that-typically-accrue-with-expressroute"></a>Затраты, которые обычно начисляются с помощью ExpressRoute

При создании канала ExpressRoute вы можете создать шлюз ExpressRoute, чтобы связать виртуальную сеть с каналом. Плата за шлюзы Почасовая ставка плюс стоимость канала ExpressRoute. См. раздел [цены на expressroute](https://azure.microsoft.com/en-us/pricing/details/expressroute) и выберите шлюзы expressroute, чтобы увидеть ставки для разных номеров SKU шлюзов.
 
### <a name="costs-might-accrue-after-resource-deletion"></a>Затраты могут начисляться после удаления ресурса

Если после удаления канала ExpressRoute имеется шлюз ExpressRoute, плата по-прежнему взимается за затраты, пока вы не удалите ее.

### <a name="using-azure-prepayment-credit"></a>Использование предварительной оплаты Azure

Вы можете платить за использование ExpressRoute с предоплатой Azure (ранее называемой денежным обязательством). Однако вы не можете использовать предварительную оплату Azure для оплаты за продукты и услуги сторонних производителей, в том числе и из Azure Marketplace.

## <a name="monitor-costs"></a>Мониторинг затрат

При использовании ресурсов Azure с ExpressRoute взимается плата. Затраты на единицу использования ресурсов Azure зависят от временных интервалов (секунды, минуты, часы и дни) или использования единиц измерения (байт, мегабайт и т. д.). Как только начнется использование ExpressRoute, затраты будут затратны, и вы сможете просмотреть затраты в [анализе затрат](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

При использовании анализа затрат вы просматриваете затраты канала ExpressRoute в графиках и таблицах на различные интервалы времени. Некоторые примеры относятся к дню, текущему и предыдущему месяцу и году. Вы также можете просматривать затраты по бюджету и прогнозируемым затратам. Переключение на более длительные представления с течением времени может помочь определить тенденции расходов. И вы видите, где может произойти чрезмерная трата. Если вы создали бюджеты, можно легко увидеть, где они превышаются.

Для просмотра затрат ExpressRoute в анализе затрат:

1. Войдите на портал Azure.

1. Перейдите к разделу **подписки**, выберите подписку в списке, а затем в меню выберите  **анализ затрат** . Выберите **область** , чтобы переключиться на другую область в анализе затрат. По умолчанию стоимость служб показана на первой кольцевой диаграмме.

    Фактические месячные затраты отображаются при первоначальном открытии анализа затрат. Ниже приведен пример, показывающий все ежемесячные затраты на использование.

    :::image type="content" source="media/plan-manage-cost/cost-analysis-pane.png" alt-text="Пример, демонстрирующий накопленные затраты для подписки":::
    

1.  Чтобы уменьшить затраты на одну службу, например Expressroute, выберите **Добавить фильтр** , а затем — **имя службы**. Затем выберите **ExpressRoute**.

    Ниже приведен пример, показывающий затраты только для ExpressRoute.

    :::image type="content" source="media/plan-manage-cost/cost-analysis-pane-expressroute.png" alt-text="Пример, демонстрирующий накопленные затраты для ExpressRoute":::

В предыдущем примере вы видите текущие затраты на службу. Также показаны затраты на регионы Azure (расположение) и стоимость ExpressRoute по группам ресурсов. Здесь вы можете исследовать затраты самостоятельно.

## <a name="create-budgets-and-alerts"></a>Создание бюджетов и оповещений

Вы можете создавать [бюджеты](../cost-management-billing/costs/tutorial-acm-create-budgets.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) для управления затратами и создавать [оповещения](../cost-management-billing/costs/cost-mgt-alerts-monitor-usage-spending.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) , которые автоматически уведомляют заинтересованных лиц о трате аномалий и чрезмерной трате рисков. Оповещения основываются на расходах по сравнению с пороговыми значениями бюджета и затрат. Бюджеты и оповещения создаются для подписок Azure и групп ресурсов, поэтому они полезны в рамках общей стратегии мониторинга затрат. 

Бюджеты можно создавать с помощью фильтров для конкретных ресурсов или служб в Azure, если требуется дополнительная детализация в мониторинге. Фильтры помогают предотвратить случайное создание новых ресурсов, которые избавляет вас от дополнительных денег. Дополнительные сведения о параметрах фильтра при создании бюджета см. в разделе [Параметры группы и фильтра](../cost-management-billing/costs/group-filter.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

## <a name="export-cost-data"></a>Экспорт данных о затратах

Вы также можете [экспортировать данные о затратах](../cost-management-billing/costs/tutorial-export-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) в учетную запись хранения. Это полезно, если вам или другим пользователям требуется выполнить дополнительный анализ данных за затраты. Например, Группа финансов может анализировать данные с помощью Excel или Power BI. Вы можете экспортировать затраты на ежедневное, еженедельное или ежемесячное расписание и задать настраиваемый диапазон дат. Для извлечения наборов данных о затратах рекомендуется экспортировать данные о затратах.

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше о том, как цены работают с Azure ExpressRoute. См. [Общие сведения о ценах на Azure ExpressRoute](https://azure.microsoft.com/en-us/pricing/details/expressroute/).
- Узнайте [, как оптимизировать инвестиции в облако с помощью службы "Управление затратами Azure](../cost-management-billing/costs/cost-mgt-best-practices.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)".
- Дополнительные сведения об управлении затратами с помощью [анализа затрат](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Узнайте, как [предотвратить непредвиденные затраты](../cost-management-billing/cost-management-billing-overview.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
- Пройдите курс обучения по [управлению затратами](/learn/paths/control-spending-manage-bills?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) .