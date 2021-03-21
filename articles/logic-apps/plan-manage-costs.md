---
title: Планирование управления затратами на Azure Logic Apps
description: Узнайте, как планировать и управлять затратами на Azure Logic Apps с помощью анализа затрат в портал Azure
ms.service: logic-apps
ms.reviewer: estfan, logicappspm, azla
ms.topic: how-to
ms.custom: subject-cost-optimization
ms.date: 01/29/2021
ms.openlocfilehash: 44351497ed58c8d49404c094f6800b52186edabb
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101709637"
---
# <a name="plan-and-manage-costs-for-azure-logic-apps"></a>Планирование и управление затратами для Azure Logic Apps

Эта статья поможет вам в планировании и управлении затратами на Azure Logic Apps. Перед созданием или добавлением ресурсов с помощью этой службы оцените затраты с помощью калькулятора цен Azure. После начала использования ресурсов Logic Apps можно настроить бюджеты и отслеживать затраты с помощью службы " [Управление затратами Azure](../cost-management-billing/cost-management-billing-overview.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)". Для выявления областей, в которых может потребоваться действовать, можно также просмотреть прогнозируемые затраты и отслеживать тенденции расходов.

Помните, что затраты на Logic Apps являются частью ежемесячных затрат в счете Azure. Несмотря на то, что в этой статье объясняется, как оценить и управлять затратами на Logic Apps, вам будет выставлен счет за все службы и ресурсы Azure, используемые в подписке Azure, включая любые сторонние службы. После ознакомления с управлением затратами на Logic Apps можно применить аналогичные методы для управления затратами для всех служб Azure, используемых в вашей подписке.

## <a name="prerequisites"></a>Предварительные условия

<!--Note for Azure service writer: This section covers prerequisites for the Cost Management's Cost Analysis feature. Add other prerequisites needed for your service after the Cost Management prerequisites. -->

Служба " [Управление затратами Azure](../cost-management-billing/cost-management-billing-overview.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) " поддерживает большинство типов учетных записей Azure. Чтобы просмотреть все поддерживаемые типы учетных записей, см. раздел Общие сведения об [управлении затратами](../cost-management-billing/costs/understand-cost-mgt-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn). Чтобы просмотреть данные о затратах, нужен как минимум доступ на чтение для учетной записи Azure.

Дополнительные сведения о назначении доступа к данным службы "Управление затратами" Azure см. в [этой статье](../cost-management-billing/costs/assign-access-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

<!--Note for Azure service writer: If you have other prerequisites for your service, add them here -->

<a name="understand-billing-model"></a>

## <a name="understand-the-billing-model"></a>Общие сведения о модели выставления счетов

Azure Logic Apps работает в инфраструктуре Azure, которая выполняет [накопление затрат](https://azure.microsoft.com/pricing/details/logic-apps/) при развертывании новых ресурсов. Убедитесь, что вы понимаете [модель выставления счетов для службы Logic Apps вместе с соответствующими ресурсами Azure](logic-apps-pricing.md), а также управляйте затратами из-за этих зависимостей при внесении изменений в развернутые ресурсы.

<a name="typical-costs"></a>

### <a name="costs-that-typically-accrue-with-azure-logic-apps"></a>Затраты, которые обычно начисляются с помощью Azure Logic Apps

Служба Logic Apps применяет различные модели ценообразования на основе ресурсов, которые вы создаете и используете.

* Ресурсы приложения логики, создаваемые и выполняемые в многоклиентской Logic Apps службе, используют [модель ценообразования потребления](../logic-apps/logic-apps-pricing.md#consumption-pricing).

* Ресурсы приложения логики, создаваемые и выполняемые в [среде службы интеграции (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) , используют [фиксированную модель ценообразования](../logic-apps/logic-apps-pricing.md#fixed-pricing).

Ниже приведены другие ресурсы, которые приводят к затратам при создании их для использования с приложениями логики.

* [Учетная запись интеграции](../logic-apps/logic-apps-pricing.md#integration-accounts) — это отдельный ресурс, который создается и связывается с приложениями логики для создания интеграции B2B. В учетных записях интеграции используется [фиксированная модель ценообразования](../logic-apps/logic-apps-pricing.md#integration-accounts) , в которой ставка зависит от используемого типа учетной записи интеграции или *уровня* .

* [Интегрированная среда сценариев](../logic-apps/logic-apps-pricing.md#fixed-pricing) — это отдельный ресурс, который создается в качестве расположения развертывания для приложений логики, которым требуется прямой доступ к ресурсам в виртуальной сети. Исес используйте [фиксированную модель ценообразования](../logic-apps/logic-apps-pricing.md#fixed-pricing) , где ставка основывается на СОЗДАВАЕМОМ номере SKU интегрированной среды сценариев и других параметрах.

* [Пользовательский соединитель](../logic-apps/logic-apps-pricing.md#consumption-pricing) — это отдельный ресурс, создаваемый для REST API, у которого нет предварительно подготовленного соединителя для использования в приложениях логики. Пользовательские выполнения соединителей используют [модель ценообразования потребления](../logic-apps/logic-apps-pricing.md#consumption-pricing) , за исключением случаев, когда они используются в интегрированной среде сценариев.

* В службе Logic Apps с несколькими клиентами [Хранение и использование хранилища данных](../logic-apps/logic-apps-pricing.md#data-retention) начисляются с использованием [фиксированной модели ценообразования](../logic-apps/logic-apps-pricing.md#fixed-pricing). Например, входные и выходные данные из журнала выполнения хранятся в фоновом хранилище, которое отличается от ресурсов хранилища, которые вы самостоятельно создаете, управляете и к которым обращаются из приложения логики.

  В интегрированной среде сценариев хранение данных и использование хранилища не требуют затрат.

<a name="costs-after-resource-deletion"></a>

### <a name="costs-might-accrue-after-resource-deletion"></a>Затраты могут начисляться после удаления ресурса

<!--Note to Azure service writer: You might need to sync with your product team to identify resources that continue to exist after those ones for your service are deleted. If you're certain that no resources can exist after those for your service are deleted, then omit this section. -->

После удаления приложения логики служба Logic Apps не будет создавать и выполнять новые экземпляры рабочих процессов. Однако все выполняющиеся и ожидающие выполнения продолжаются до завершения. В зависимости от числа таких запусков Этот процесс может занять некоторое время. Дополнительные сведения см. в статье [Управление приложениями логики](manage-logic-apps-with-azure-portal.md#delete-logic-apps).

При наличии этих ресурсов после удаления приложения логики эти ресурсы продолжают существовать и начисляются затраты, пока вы не удалите их:

* Ресурсы Azure, которые вы создаете и управляете независимо от приложения логики, которое подключается к этим ресурсам, например приложения функций Azure, концентраторы событий, службы "Сетка событий" и т. д.

* Учетные записи интеграции

* Среды службы интеграции (Исес)

  Если [Удалить интегрированную среду сценариев](ise-manage-integration-service-environment.md#delete-ise), связанная виртуальная сеть Azure, подсети и другие связанные ресурсы продолжают существовать. После удаления интегрированной среды сценариев может потребоваться подождать до определенного количества часов, прежде чем можно будет попытаться удалить виртуальную сеть или подсети.

### <a name="using-monetary-credit-with-azure-logic-apps"></a>Использование денежного кредита с Azure Logic Apps

Вы можете платить за Azure Logic Apps оплаты с кредитом по денежным обязательствам EA. Однако нельзя использовать денежные обязательства EA для оплаты за продукты и услуги сторонних производителей, включая данные из Azure Marketplace.

<a name="estimate-costs"></a>

## <a name="estimate-costs"></a>Оценка затрат

Прежде чем создавать ресурсы с Azure Logic Apps, оцените затраты с помощью [калькулятора цен Azure](https://azure.microsoft.com/pricing/calculator/). Дополнительные сведения см. в [модели ценообразования для Azure Logic Apps](../logic-apps/logic-apps-pricing.md).

1. На [странице калькулятор цен Azure](https://azure.microsoft.com/pricing/calculator/)в меню слева выберите **Integration**  >  **Azure Logic Apps**.

   ![Снимок экрана, на котором показан калькулятор цен Azure с выбранным "Azure Logic Apps".](./media/plan-manage-costs/add-azure-logic-apps-pricing-calculator.png)

1. Прокрутите страницу вниз, пока не сможете просмотреть калькулятор цен на Azure Logic Apps. В различных разделах, посвященных ресурсам Azure, которые напрямую связаны с Azure Logic Apps, введите количество ресурсов, которые планируется использовать, и число интервалов, по которым эти ресурсы можно использовать.

   На этом снимке экрана показан пример оценки затрат с помощью калькулятора:

   ![Пример, в котором показана оценочная стоимость в калькуляторе цен Azure](./media/plan-manage-costs/example-logic-apps-pricing-calculator.png)

1. Чтобы обновить оценки затрат по мере создания и использования новых связанных ресурсов, вернитесь к этому калькулятору и обновите эти ресурсы.

<a name="create-budgets-alerts"></a>

## <a name="create-budgets-and-alerts"></a>Создание бюджетов и оповещений

Чтобы упростить управление затратами для вашей учетной записи или подписки Azure, вы можете создавать [бюджеты](../cost-management-billing/costs/tutorial-acm-create-budgets.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) и [оповещения](../cost-management-billing/costs/cost-mgt-alerts-monitor-usage-spending.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) с помощью службы " [Управление затратами Azure" и "выставление счетов](../cost-management-billing/cost-management-billing-overview.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) " и возможности.  Бюджеты и оповещения создаются для подписок Azure и групп ресурсов, поэтому они полезны в рамках общей стратегии мониторинга затрат.

На основе расходов по сравнению с пороговыми значениями бюджета и стоимости оповещения автоматически уведомляют заинтересованные лица о трате аномалий и чрезмерном потреблении рисков. Если вы хотите повысить степень детализации в мониторинге, можно также создать бюджеты, использующие фильтры для конкретных ресурсов или служб в Azure. Фильтры помогают убедиться, что вы случайно не создаете новые ресурсы, которые изменяют дополнительные деньги. Дополнительные сведения о параметрах фильтра см. в разделе [Параметры группы и фильтра](../cost-management-billing/costs/group-filter.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).

<a name="monitor-costs"></a>

## <a name="monitor-costs"></a>Мониторинг затрат

Затраты на единицы использования ресурсов зависят от временных интервалов, таких как секунды, минуты, часы и дни, или использования единиц измерения, таких как байты, мегабайты и т. д. Некоторые примеры относятся к дню, текущему и предыдущему месяцу и году. Переключение на более длительные представления с течением времени может помочь определить тенденции расходов. При использовании функций анализа затрат можно просматривать затраты в виде графиков и таблиц за различные интервалы времени. Если вы создали бюджеты и прогнозы затрат, вы также можете легко найти, где превышены бюджеты и что может привести к чрезмерному расходу.

После начала периодических затрат для ресурсов, которые создают или начинают использовать в Azure, вы можете просматривать и отслеживать эти затраты следующим образом:

* [Мониторинг выполнения приложений логики и потребления хранилища](#monitor-billing-metrics) с помощью Azure Monitor

* Выполнение [анализа затрат](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) с помощью службы " [Управление затратами Azure" и выставления счетов](../cost-management-billing/cost-management-billing-overview.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)

<a name="monitor-billing-metrics"></a>

### <a name="monitor-logic-app-executions-and-storage-consumption"></a>Мониторинг выполнения приложений логики и использования хранилища

С помощью Azure Monitor можно просмотреть эти метрики для конкретного приложения логики:

* Оплачиваемые выполнения действий
* Оплачиваемые выполнения триггеров
* Использование выставления счетов для выполнения собственных операций
* Использование выставления счетов для стандартных выполнений соединителей
* Использование выставления счетов для использования хранилища
* Всего оплачиваемых выполнений

<a name="execution-storage-metrics"></a>

#### <a name="view-execution-and-storage-consumption-metrics"></a>Просмотр метрик выполнения и потребления хранилища

1. На портале Azure найдите и откройте нужное приложение логики. В меню приложения логики в разделе **мониторинг** выберите **метрики**.

1. В области справа в разделе **заголовок диаграммы** на панели метрика откройте список **метрик** и выберите нужную метрику.

   > [!NOTE]
   > Потребление хранилища измеряется как количество единиц хранения (в ГБ), которые использует приложение логики и оплачивается. Запуски, использующие менее 500 МБ памяти в хранилище, могут не отображаться в представлении мониторинга, но счета за них по-прежнему выставляются.

   ![Снимок экрана, на котором показана панель метрик с открытым списком "Метрика".](./media/logic-apps-pricing/select-metric.png)

1. В правом верхнем углу панели выберите нужный период времени.

1. Чтобы просмотреть другие данные о потреблении хранилища, в частности, размеры входных и выходных данных действия в журнале выполнения приложения логики, [выполните следующие действия](#view-input-output-sizes).

<a name="view-input-output-sizes"></a>

#### <a name="view-action-input-and-output-sizes-in-run-history"></a>Просмотр размеров входных и выходных данных действий в журнале выполнения

1. На портале Azure найдите и откройте нужное приложение логики.

1. В меню приложения логики выберите **Обзор**.

1. В области справа в разделе **журнал запусков** выберите Запуск с входными и выходными данными, которые необходимо просмотреть.

1. В разделе **Запуск приложения логики** выберите **сведения о выполнении**.

1. В области **сведения о запуске приложения логики** в таблице действия, в которой перечислены состояние и длительность каждого действия, выберите действие, которое требуется просмотреть.

1. В области **действие приложения логики** найдите размеры входных и выходных данных этого действия. В разделе **входные данные ссылки** и **выходные данные** найдите ссылки на эти входные и выходные данные.

   > [!NOTE]
   > Для циклов for только действия верхнего уровня показывают размеры входных и выходных данных. Для действий внутри вложенных циклов входные и выходные данные показывают нулевой размер и не имеют ссылок.

<a name="run-cost-analysis"></a>

### <a name="run-cost-analysis-by-using-azure-cost-management-and-billing"></a>Выполнение анализа затрат с помощью службы "Управление затратами Azure" и выставления счетов

Чтобы ознакомиться с затратами на Logic Appsную службу на основе определенной области, например подписки Azure, можно использовать возможности [анализа затрат](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) в [службе "Управление затратами Azure" и "выставление счетов](../cost-management-billing/cost-management-billing-overview.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)".

1. В портал Azure откройте нужную область, например подписку Azure. В меню слева в разделе **Управление затратами** выберите **анализ затрат**.

   При первом открытии панели анализ затрат в верхней диаграмме отображаются фактические и прогнозируемые затраты на использование всех служб в подписке за текущий месяц.

   ![Снимок экрана, на котором показана портал Azure и панель анализа затрат с примером для фактической и прогнозируемой затрат в подписке.](./media/plan-manage-costs/cost-analysis-total-actual-forecasted-costs.png)

   > [!TIP]
   > Чтобы изменить области, на панели " **анализ затрат** " на панели "фильтры" выберите фильтр **области** . На панели **Выбор области** перейдите к нужной области.

   В нижней диаграмме кольцевые диаграммы показывают текущие затраты на службы Azure, по регионам Azure (расположение) и по группе ресурсов.

   ![Снимок экрана, на котором показана портал Azure и панель анализа затрат с примерами кольцевых диаграмм для служб, регионов и групп ресурсов.](./media/plan-manage-costs/cost-analysis-donut-charts.png)

1. Чтобы отфильтровать диаграмму в определенной области, например в службе или ресурсе, на панели Фильтры выберите **Добавить фильтр**.

1. В списке слева выберите тип фильтра, например **имя службы**. В списке справа выберите фильтр, например **Logic Apps**. Когда все будет готово, установите флажок зеленая галочка.

   ![Снимок экрана, на котором показана портал Azure и панель анализа затрат с выбранными фильтрами.](./media/plan-manage-costs/cost-analysis-add-service-name-filter.png)

   Например, вот результат для службы Logic Apps:

   ![Снимок экрана, на котором показана портал Azure и панель анализа затрат с результатами, отфильтрованными по "Logic Apps".](./media/plan-manage-costs/cost-analysis-total-actual-costs-service.png)

### <a name="export-cost-data"></a>Экспорт данных о затратах

Если требуется выполнить дополнительные анализ данных по затратам, можно [экспортировать данные о затратах](../cost-management-billing/costs/tutorial-export-acm-data.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn) в учетную запись хранения. Например, Группа финансового отдела может анализировать эти данные с помощью Excel или Power BI. Вы можете экспортировать затраты на ежедневное, еженедельное или ежемесячное расписание, а также задать настраиваемый диапазон дат. Для извлечения наборов данных о затратах рекомендуется экспортировать данные о затратах.

## <a name="other-ways-to-manage-and-reduce-costs"></a>Другие способы управления и снижения затрат

<!-- Note to Azure service writer: This section is optional. Other than using the Cost Management methods above, your service probably has other specific ways to minimize costs. For example, you might have best practice advice or specific ways to reduce costs that are specific to your service. If so, try to add that guidance here or at least summarize key points. Try to be as prescriptive as possible. If you have more comprehensive content, add links to your other published articles or sections here.

Add a statement that discusses any recommended settings for your service that might help keep the charges minimal if a service isn't being actively used by the customer. For example: Will turning off a VM help to get no charges for the specific VM resource?

Otherwise, if no other cost-saving recommendations or best practices exist to reduce costs, cut this section.
-->

Чтобы сократить затраты на ТД логики и связанные ресурсы, попробуйте выполнить следующие действия:

* По возможности используйте [встроенные триггеры и действия](../connectors/apis-list.md#built-in), которые снижают затраты на выполнение для каждого выполнения, чем [управляемые триггеры и действия соединителя](../connectors/apis-list.md#managed-connectors).

  Например, вы можете снизить затраты при доступе к другим ресурсам с помощью [действия HTTP](../connectors/connectors-native-http.md) или путем вызова функции, созданной с помощью [службы функций Azure](../azure-functions/functions-overview.md) , и использования [встроенного действия функций Azure](../logic-apps/logic-apps-azure-functions.md). Тем не менее, использование функций Azure также влечет за собой затраты, поэтому необходимо сравнить варианты.

* [Укажите точные условия триггера](logic-apps-workflow-actions-triggers.md#trigger-conditions) для запуска рабочего процесса.

  Например, можно указать, что триггер срабатывает только в том случае, если целевой веб-сайт возвращает внутреннюю ошибку сервера. В определении JSON триггера используйте `conditions` свойство, чтобы указать условие, которое ссылается на код состояния триггера.

* Если у триггера есть версия для опроса и веб-перехватчик, попробуйте использовать версию веб-перехватчика, которая ожидает, пока заданное событие не порождается, а не регулярно проверяет событие.

* Вызовите приложение логики через другую службу, чтобы триггер срабатывает только при запуске рабочего процесса.

  Например, вы можете вызвать приложение логики из функции, созданной и выполняемой с помощью службы функций Azure. Например, см. статью [вызов или Активация приложений логики с помощью функций Azure и служебной шины Azure](logic-apps-scenario-function-sb-trigger.md).

* [Отключите приложения логики](manage-logic-apps-with-azure-portal.md#disable-or-enable-logic-apps) , которые не должны постоянно запускаться, или [удалите приложения логики](manage-logic-apps-with-azure-portal.md#delete-logic-apps) , которые больше не нужны. Если это возможно, отключите все ресурсы, которые не требуется постоянно активировать.

## <a name="next-steps"></a>Дальнейшие действия

* [Оптимизация инвестиций в облако с помощью Управления затратами Azure](../cost-management-billing/costs/cost-mgt-best-practices.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)
* [Управление затратами с помощью анализа затрат](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)
* [Предотвращение непредвиденных расходов](../cost-management-billing/understand/analyze-unexpected-charges.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)
* Пройдите курс обучения по [управлению затратами](/learn/paths/control-spending-manage-bills?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)