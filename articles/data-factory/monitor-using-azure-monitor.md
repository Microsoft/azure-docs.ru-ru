---
title: Мониторинг фабрик данных с помощью Azure Monitor
description: Узнайте, как использовать Azure Monitor для мониторинга конвейеров фабрики данных/Азуре, включив журналы диагностики со сведениями из фабрики данных.
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 07/13/2020
ms.openlocfilehash: 04aba71e49fedc8a4ca22aed576aaa978f733336
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104783648"
---
# <a name="monitor-and-alert-data-factory-by-using-azure-monitor"></a>Мониторинг и оповещение фабрики данных с помощью Azure Monitor

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Облачные приложения являются сложными и имеют много движущихся частей. Мониторы предоставляют данные, помогающие обеспечить работоспособность приложений. Мониторы также помогают избежать потенциальных проблем и устранять неполадки в прошлом. Данные мониторинга можно использовать для получения глубокой информации о приложениях. Эти знания помогут повысить производительность и удобство обслуживания приложений. Она также помогает автоматизировать действия, которые в противном случае требуется вмешательство вручную.

Azure Monitor предоставляет метрики и журналы инфраструктуры базового уровня для большинства служб Azure. Журналы диагностики Azure выдаются ресурсом. Они содержат подробные и своевременные данные о работе этого ресурса. Фабрика данных Azure (ADF) может записывать журналы диагностики в Azure Monitor. Уделите 7 минут своего времени, чтобы просмотреть следующее видео с кратким обзором и демонстрацией этой функции.

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Monitor-Data-Factory-pipelines-using-Operations-Management-Suite-OMS/player]

Дополнительные сведения см. в статье [Общие сведения о службе Azure Monitor](../azure-monitor/overview.md).

## <a name="keeping-azure-data-factory-metrics-and-pipeline-run-data"></a>Обеспечение метрик фабрики данных Azure и конвейера — запуск данных

В фабрике данных хранится конвейер — данные запускаются только в течение 45 дней. Используйте Azure Monitor, если хотите, чтобы эти данные оставались более длительное время. С помощью монитора можно направить журналы диагностики для анализа в несколько разных целевых объектов.

* **Учетная запись хранения**. сохраняйте журналы диагностики в учетную запись хранения для аудита или проверки вручную. Параметры диагностики можно использовать для указания времени хранения в днях.
* **Концентратор событий**: потоковая передача журналов в концентраторы событий Azure. Журналы становятся входными данными для партнерской службы или пользовательского аналитического решения, например Power BI.
* **Log Analytics**: Проанализируйте журналы с помощью log Analytics. Интеграция фабрики данных с Azure Monitor полезна в следующих сценариях:
  * Необходимо писать сложные запросы к широкому набору метрик, которые публикуются фабрикой данных для отслеживания. С помощью монитора можно создавать пользовательские предупреждения для этих запросов.
  * Требуется отслеживать фабрики данных. Вы можете направить данные из нескольких фабрик данных в одну рабочую область монитора.

Вы также можете использовать учетную запись хранения или пространство имен концентратора событий, не подписку на ресурс, который создает журналы. Пользователь, который настраивает этот параметр, должен иметь доступ к обеим подпискам с помощью управления доступом на основе ролей Azure (Azure RBAC).

## <a name="configure-diagnostic-settings-and-workspace"></a>Настройка параметров и рабочей области диагностики

Создайте или добавьте параметры диагностики для фабрики данных.

1. На портале перейдите к разделу монитор. Выберите **Параметры** параметры  >  **диагностики**.

1. Выберите фабрику данных, для которой необходимо задать параметр диагностики.

1. Если в выбранной фабрике данных не существует параметров, вам будет предложено создать параметр. Выберите **Включить диагностику**.

   ![Создать параметр диагностики, если параметры не существуют](media/data-factory-monitor-oms/monitor-oms-image1.png)

   Если в фабрике данных есть существующие параметры, отобразится список параметров, уже настроенных в фабрике данных. Выберите **Добавить параметр диагностики**.

   ![Добавить параметр диагностики, если параметры существуют](media/data-factory-monitor-oms/add-diagnostic-setting.png)

1. Присвойте параметру имя, выберите **отправить log Analytics**, а затем выберите рабочую область из **log Analytics рабочей области**.

    * В режиме _диагностики Azure_ журналы диагностики поступают в таблицу _AzureDiagnostics_ .

    * В режиме, _зависящем от ресурса_ , журналы диагностики из фабрики данных Azure поступают в следующие таблицы:
      - _адфактивитирун_
      - _адфпипелинерун_
      - _адфтригжеррун_
      - _ADFSSISIntegrationRuntimeLogs_
      - _ADFSSISPackageEventMessageContext_
      - _ADFSSISPackageEventMessages_
      - _ADFSSISPackageExecutableStatistics_
      - _ADFSSISPackageExecutionComponentPhases_
      - _ADFSSISPackageExecutionDataStatistics_

      Можно выбрать различные журналы, относящиеся к рабочим нагрузкам, для отправки в Log Analytics таблицы. Например, если вы не используете SQL Server Integration Services (SSIS) вообще, не нужно выбирать журналы служб SSIS. Если вы хотите вести журнал операций запуска, отмены или обслуживания служб SSIS Integration Runtime (IR), можно выбрать пункт журналы IR для служб SSIS. При вызове выполнения пакетов служб SSIS с помощью T-SQL на SQL Server Management Studio (SSMS), агент SQL Server или других назначенных средствах можно выбрать журналы пакетов служб SSIS. При вызове выполнения пакетов служб SSIS с помощью действий выполнить пакет служб SSIS в конвейерах ADF можно выбрать все журналы.

    * Если выбрать _аллметрикс_, будут доступны различные метрики ADF для мониторинга или создания оповещений, включая метрики для действия ADF, конвейера и запуска триггеров, а также для операций IR и выполнения пакетов служб SSIS.

   ![Назовите свои параметры и выберите рабочую область log-Analytics.](media/data-factory-monitor-oms/monitor-oms-image2.png)

    > [!NOTE]
    > Так как таблица журналов Azure не может содержать более 500 столбцов, мы **настоятельно рекомендуем** выбрать _режим для конкретного ресурса_. Дополнительные сведения см. в разделе [Справочник по журналам AzureDiagnostics](/azure-monitor/reference/tables/azurediagnostics#additionalfields-column).

1. Щелкните **Сохранить**.

Через несколько секунд в списке параметров для этой фабрики данных появится новый параметр. Журналы диагностики передаются в эту рабочую область, как только создаются новые данные о событиях. Между созданием события и его отображением в Log Analytics может пройти до 15 минут.

## <a name="install-azure-data-factory-analytics-solution-from-azure-marketplace"></a>Установка решения "аналитика фабрики данных Azure" из Azure Marketplace

Это решение предоставляет сводку по общей работоспособности фабрики данных с параметрами для детализации подробностей и устранения непредвиденных закономерностей поведения. Благодаря богатому расширению представлений можно получать аналитические сведения об обработке ключей, в том числе:

* В кратком обзоре конвейера фабрики данных выполняются действия и триггеры.
* Возможность детализации операций фабрики данных по типу
* Сводка по верхнему конвейеру фабрики данных, ошибок действий

1. Перейдите в **Azure Marketplace**, выберите **аналитический** фильтр и найдите **аналитику фабрики данных Azure (Предварительная версия)** .

   ![Перейдите в Azure Marketplace, введите "Analytics Filter" и выберите "аналитика фабрики данных Azure (Предварительная версия)".](media/data-factory-monitor-oms/monitor-oms-image3.png)

1. Сведения о **аналитике фабрики данных Azure (Предварительная версия)**

   ![Сведения о "аналитике фабрики данных Azure (Предварительная версия)"](media/data-factory-monitor-oms/monitor-oms-image4.png)

1. Выберите **создать** , а затем создайте или выберите **рабочую область log Analytics**.

   ![Создание нового решения](media/data-factory-monitor-oms/monitor-log-analytics-image-5.png)

### <a name="monitor-data-factory-metrics"></a>Мониторинг метрик фабрики данных

При установке этого решения создается набор представлений по умолчанию в разделе книги выбранной Log Analytics рабочей области. В результате становятся доступными следующие метрики:

* Запуски ADF — 1) запуски конвейеров фабрикой данных
* Запуски ADF-2) запуски действий по фабрике данных
* Запуски ADF-3) запуски триггеров фабрикой данных
* Ошибки ADF — 1) первые 10 ошибок конвейера по фабрике данных
* Ошибки ADF — 2) первые 10 запусков операций по фабрике данных
* Ошибки ADF-3) 10 основных ошибок триггера по фабрике данных
* Статистика ADF-1) выполняемые действия по типу
* Статистика ADF-2. запуски триггеров по типу
* Статистика ADF-3) длительность выполнения конвейера в максимальной

![Окно с выделенными книгами (Предварительная версия) и "Азуредатафакторяналитикс"](media/data-factory-monitor-oms/monitor-oms-image6.png)

Вы можете визуализировать предыдущие метрики, просматривать запросы, основанные на этих метриках, изменять запросы, создавать предупреждения и выполнять другие действия.

![Графическое представление выполнения конвейера по фабрике данных "](media/data-factory-monitor-oms/monitor-oms-image8.png)

> [!NOTE]
> Аналитика фабрики данных Azure (Предварительная версия) отправляет журналы диагностики в целевые таблицы для _конкретных ресурсов_ . Вы можете создавать запросы к следующим таблицам: _адфпипелинерун_, _адфтригжеррун_ и _адфактивитирун_.

## <a name="data-factory-metrics"></a>Метрики фабрики данных

С помощью монитора можно получить представление о производительности и работоспособности рабочих нагрузок Azure. Наиболее важным типом данных монитора является метрика, которая также называется счетчиком производительности. Метрики создаются большинством ресурсов Azure. Монитор предоставляет несколько способов настройки и использования этих метрик для мониторинга и устранения неполадок.

Ниже приведены некоторые метрики, созданные фабрикой данных Azure версии 2.

| **Метрика**                           | **Отображаемое имя метрики**                  | **Единица измерения** | **Тип агрегирования** | **Описание**                |
|--------------------------------------|------------------------------------------|----------|----------------------|--------------------------------|
| ActivityCancelledRuns                 | Метрики отмененных выполнений действий           | Count    | Итог                | Общее число выполнений действий, которые были отменены в течение минуты. |
| ActivityFailedRuns                   | Метрики неудачных выполнений действий.             | Count    | Итог                | Общее число выполнений действий, завершившихся сбоем, в течение минуты. |
| ActivitySucceededRuns                | Метрики успешных выполнений действий.          | Count    | Итог                | Общее число выполнений действий, выполненных в течение минуты. |
| PipelineCancelledRuns                 | Метрики отмененных выполнений конвейеров           | Count    | Итог                | Общее число запусков конвейера, которые были отменены в течение минуты. |
| PipelineFailedRuns                   | Метрики неудачных выполнений конвейеров.             | Count    | Итог                | Общее число запусков конвейера, завершившихся сбоем, в течение минуты. |
| PipelineSucceededRuns                | Метрики успешных выполнений конвейеров.          | Count    | Итог                | Общее количество выполнений конвейеров, выполненных в течение минуты. |
| TriggerCancelledRuns                  | Метрики отмененных запусков триггеров            | Count    | Итог                | Общее число запусков триггеров, которые были отменены в течение минуты. |
| TriggerFailedRuns                    | Метрики неудачных запусков триггеров.              | Count    | Итог                | Общее число выполнений триггеров в течение минуты. |
| TriggerSucceededRuns                 | Метрики успешных запусков триггеров.           | Count    | Итог                | Общее число запусков триггеров, выполненных в течение минуты. |
| ссисинтегратионрунтиместартканцеллед  | Начальные метрики среды выполнения интеграции служб SSIS отменены           | Count    | Итог                | Общее число запусков среды выполнения интеграции служб SSIS, которые были отменены в течение минуты. |
| ссисинтегратионрунтиместартфаилед    | Неудачные метрики запуска среды выполнения интеграции SSIS             | Count    | Итог                | Общее число запусков среды выполнения интеграции служб SSIS в течение минуты. |
| ссисинтегратионрунтиместартсукцеедед | Метрики запуска среды выполнения интеграции служб SSIS          | Count    | Итог                | Общее число запусков среды выполнения интеграции служб SSIS в течение минуты. |
| ссисинтегратионрунтиместопстукк      | Зависание метрик среды выполнения интеграции SSIS               | Count    | Итог                | Общее количество остановленных сред выполнения интеграции служб SSIS в течение минуты. |
| ссисинтегратионрунтиместопсукцеедед  | Метрики остановленной среды выполнения интеграции SSIS завершены           | Count    | Итог                | Общее число остановленных сред выполнения интеграции служб SSIS в течение минуты. |
| ссиспаккажеексекутионканцеллед         | Метрики выполнения пакета служб SSIS отменены  | Count    | Итог                | Общее число выполнений пакетов служб SSIS, которые были отменены в течение минуты. |
| ссиспаккажеексекутионфаилед           | Неудачные метрики выполнения пакетов служб SSIS    | Count    | Итог                | Общее число выполнений пакетов служб SSIS, завершившихся сбоем, в течение минуты. |
| ссиспаккажеексекутионсукцеедед        | Метрики выполнения пакета служб SSIS завершены | Count    | Итог                | Общее число выполнений пакетов служб SSIS, выполненных в течение минуты. |

Чтобы получить доступ к метрикам, выполните инструкции в разделе [Azure Monitor Data Platform](../azure-monitor/data-platform.md).

> [!NOTE]
> Создаются только события из завершенных, активированных действий и запусков конвейера. В ходе выполнения и запуски отладки **не** выдаются. С другой стороны, создаются события из **всех** выполнений пакетов служб SSIS, включая завершенные и выполняемые, независимо от их методов вызова. Например, можно вызывать выполнение пакетов в SQL Server Data Tools с поддержкой Azure (SSDT) с помощью T-SQL в SSMS, агент SQL Server или других назначенных средств, а также в качестве активированных или отладочных запусков действий по выполнению пакетов служб SSIS в конвейерах ADF.

## <a name="data-factory-alerts"></a>Оповещения фабрики данных

Войдите в портал Azure и выберите **Monitor**  >  **Alerts (монитор оповещений** ), чтобы создать оповещения.

![Оповещения в меню портала](media/monitor-using-azure-monitor/alerts_image3.png)

### <a name="create-alerts"></a>Создание предупреждений

1. Нажмите кнопку **+ Новое правило генерации оповещений**, чтобы создать оповещение.

    ![Новое правило генерации оповещений](media/monitor-using-azure-monitor/alerts_image4.png)

1. Определите условие оповещения.

    > [!NOTE]
    > Убедитесь, что в раскрывающемся списке **Фильтровать по типу ресурсов** выбрано значение **все** .

    !["Определение условия оповещения" > "Выбор целевого объекта", который открывает область "Выбор ресурса" ](media/monitor-using-azure-monitor/alerts_image5.png)

    !["Определение условия оповещения" > "добавить условие", которое открывает панель "Настройка логики сигнала"](media/monitor-using-azure-monitor/alerts_image6.png)

    ![Панель «Настройка типа сигнала»](media/monitor-using-azure-monitor/alerts_image7.png)

1. Указание сведений для оповещения.

    ![Сведения об оповещении](media/monitor-using-azure-monitor/alerts_image8.png)

1. Определите группу действий.

    ![Создать правило с выделенным пунктом "Новая группа действий"](media/monitor-using-azure-monitor/alerts_image9.png)

    ![Создание новой группы действий](media/monitor-using-azure-monitor/alerts_image10.png)

    ![Настройка электронной почты, SMS, push-уведомлений и речи](media/monitor-using-azure-monitor/alerts_image11.png)

    ![Определение группы действий](media/monitor-using-azure-monitor/alerts_image12.png)

## <a name="set-up-diagnostic-logs-via-the-azure-monitor-rest-api"></a>Настройка журналов диагностики с помощью Azure Monitor REST API

### <a name="diagnostic-settings"></a>Параметры диагностики

Используйте параметры диагностики, чтобы настроить журналы диагностики для нерасчетных ресурсов. Параметры для элемента управления ресурсами имеют следующие характеристики.

* Они указывают, куда отправляются журналы диагностики. Примеры включают учетную запись хранения Azure, концентратор событий Azure или журналы мониторинга.
* Они указывают, какие категории журналов отправляются.
* Они определяют, как долго каждая категория журнала должна храниться в учетной записи хранения.
* Срок хранения 0 дней означает, что журналы хранятся неограниченно долго. В противном случае значение может быть любым числом дней от 1 до 2 147 483 647.
* Если политики хранения заданы, но хранение журналов в учетной записи хранения отключено, политики хранения не будут действовать. Например, это может произойти, если выбраны только параметры концентраторов событий или мониторинга журналов.
* Политики хранения применяются в день. Граница между днями выполняется в полночь в формате UTC. В конце дня журналы удаляются из числа дней, которые выходят за пределы политики хранения. Например, если у вас есть политика хранения на один день, то в начале текущей версии журналы с до вчерашнего дня удаляются.

### <a name="enable-diagnostic-logs-via-the-azure-monitor-rest-api"></a>Включение журналов диагностики с помощью Azure Monitor REST API

#### <a name="create-or-update-a-diagnostics-setting-in-the-monitor-rest-api"></a>Создание или обновление параметра диагностики в мониторе REST API

##### <a name="request"></a>Запрос

```
PUT
https://management.azure.com/{resource-id}/providers/microsoft.insights/diagnosticSettings/service?api-version={api-version}
```

##### <a name="headers"></a>Заголовки

* Замените `{api-version}` на `2016-09-01`.
* Замените `{resource-id}` идентификатором ресурса, параметры диагностики которого необходимо изменить. Дополнительные сведения см. [в статье Использование групп ресурсов для управления ресурсами Azure](../azure-resource-manager/management/manage-resource-groups-portal.md).
* В качестве заголовка `Content-Type` установите `application/json`.
* Задайте заголовок авторизации для веб-токена JSON, полученного из Azure Active Directory (Azure AD). Дополнительные сведения см. в разделе [Проверка подлинности запросов](../active-directory/develop/authentication-vs-authorization.md).

##### <a name="body"></a>Текст

```json
{
    "properties": {
        "storageAccountId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>/providers/Microsoft.Storage/storageAccounts/<storageAccountName>",
        "serviceBusRuleId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>/providers/Microsoft.EventHub/namespaces/<eventHubName>/authorizationrules/RootManageSharedAccessKey",
        "workspaceId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<LogAnalyticsName>",
        "metrics": [
        ],
        "logs": [
                {
                    "category": "PipelineRuns",
                    "enabled": true,
                    "retentionPolicy": {
                        "enabled": false,
                        "days": 0
                    }
                },
                {
                    "category": "TriggerRuns",
                    "enabled": true,
                    "retentionPolicy": {
                        "enabled": false,
                        "days": 0
                    }
                },
                {
                    "category": "ActivityRuns",
                    "enabled": true,
                    "retentionPolicy": {
                        "enabled": false,
                        "days": 0
                    }
                }
            ]
    },
    "location": ""
}
```

| Свойство | Тип | Описание |
| --- | --- | --- |
| **storageAccountId** |Строка | Идентификатор ресурса учетной записи хранения, в которую необходимо отправить журналы диагностики. |
| **serviceBusRuleId** |Строка | Идентификатор правила служебной шины для пространства имен Service-Bus, в котором должны быть созданы концентраторы событий для потоковой передачи журналов диагностики. Идентификатор правила имеет формат `{service bus resource ID}/authorizationrules/{key name}` .|
| **workspaceId** | Сложный тип | Массив временных граней метрики и их политик хранения. Значение этого свойства пусто. |
|**метрики**| Значения параметров выполнения конвейера, которые должны быть переданы в вызываемый конвейер.| Объект JSON, который сопоставляет имена параметров со значениями аргументов. |
| **журналы**| Сложный тип| Имя категории журнала диагностики для типа ресурса. Чтобы получить список категорий журналов диагностики для ресурса, выполните операцию получения диагностической настройки. |
| **category**| Строка| Массив категорий журналов и их политик хранения. |
| **timegrain** | Строка | Гранулярность метрик, которые записываются в формате длительности ISO 8601. Значение свойства должно быть равно `PT1M` , что указывает одну минуту. |
| **доступной**| Логическое | Указывает, включен ли сбор метрики или категории журналов для этого ресурса. |
| **retentionPolicy**| Сложный тип| Описывает политику хранения метрики или категории журналов. Это свойство используется только для учетных записей хранения. |
|**недели**| Int| Количество дней, в течение которых хранятся метрики или журналы. Если значение свойства равно 0, журналы хранятся неограниченное время. Это свойство используется только для учетных записей хранения. |

##### <a name="response"></a>Ответ

200 OK.

```json
{
    "id": "/subscriptions/<subID>/resourcegroups/adf/providers/microsoft.datafactory/factories/shloadobetest2/providers/microsoft.insights/diagnosticSettings/service",
    "type": null,
    "name": "service",
    "location": null,
    "kind": null,
    "tags": null,
    "properties": {
        "storageAccountId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>//providers/Microsoft.Storage/storageAccounts/<storageAccountName>",
        "serviceBusRuleId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>//providers/Microsoft.EventHub/namespaces/<eventHubName>/authorizationrules/RootManageSharedAccessKey",
        "workspaceId": "/subscriptions/<subID>/resourceGroups/<resourceGroupName>//providers/Microsoft.OperationalInsights/workspaces/<LogAnalyticsName>",
        "eventHubAuthorizationRuleId": null,
        "eventHubName": null,
        "metrics": [],
        "logs": [
            {
                "category": "PipelineRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            },
            {
                "category": "TriggerRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            },
            {
                "category": "ActivityRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            }
        ]
    },
    "identity": null
}
```

#### <a name="get-information-about-diagnostics-settings-in-the-monitor-rest-api"></a>Получение сведений о параметрах диагностики на мониторе REST API

##### <a name="request"></a>Запрос

```
GET
https://management.azure.com/{resource-id}/providers/microsoft.insights/diagnosticSettings/service?api-version={api-version}
```

##### <a name="headers"></a>Заголовки

* Замените `{api-version}` на `2016-09-01`.
* Замените `{resource-id}` идентификатором ресурса, параметры диагностики которого необходимо изменить. Дополнительные сведения см. [в статье Использование групп ресурсов для управления ресурсами Azure](../azure-resource-manager/management/manage-resource-groups-portal.md).
* В качестве заголовка `Content-Type` установите `application/json`.
* Задайте заголовок авторизации для веб-токена JSON, полученного из Azure AD. Дополнительные сведения см. в разделе [Проверка подлинности запросов](../active-directory/develop/authentication-vs-authorization.md).

##### <a name="response"></a>Ответ

200 OK.

```json
{
    "id": "/subscriptions/<subID>/resourcegroups/adf/providers/microsoft.datafactory/factories/shloadobetest2/providers/microsoft.insights/diagnosticSettings/service",
    "type": null,
    "name": "service",
    "location": null,
    "kind": null,
    "tags": null,
    "properties": {
        "storageAccountId": "/subscriptions/<subID>/resourceGroups/shloprivate/providers/Microsoft.Storage/storageAccounts/azmonlogs",
        "serviceBusRuleId": "/subscriptions/<subID>/resourceGroups/shloprivate/providers/Microsoft.EventHub/namespaces/shloeventhub/authorizationrules/RootManageSharedAccessKey",
        "workspaceId": "/subscriptions/<subID>/resourceGroups/ADF/providers/Microsoft.OperationalInsights/workspaces/mihaipie",
        "eventHubAuthorizationRuleId": null,
        "eventHubName": null,
        "metrics": [],
        "logs": [
            {
                "category": "PipelineRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            },
            {
                "category": "TriggerRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            },
            {
                "category": "ActivityRuns",
                "enabled": true,
                "retentionPolicy": {
                    "enabled": false,
                    "days": 0
                }
            }
        ]
    },
    "identity": null
}
```
Дополнительные сведения см. в разделе [параметры диагностики](/rest/api/monitor/diagnosticsettings).

## <a name="schema-of-logs-and-events"></a>Схема журналов и событий

### <a name="monitor-schema"></a>Схема монитора

#### <a name="activity-run-log-attributes"></a>Действия — выполнение атрибутов журнала

```json
{
   "Level": "",
   "correlationId":"",
   "time":"",
   "activityRunId":"",
   "pipelineRunId":"",
   "resourceId":"",
   "category":"ActivityRuns",
   "level":"Informational",
   "operationName":"",
   "pipelineName":"",
   "activityName":"",
   "start":"",
   "end":"",
   "properties":
       {
          "Input": "{
              "source": {
                "type": "BlobSource"
              },
              "sink": {
                "type": "BlobSink"
              }
           }",
          "Output": "{"dataRead":121,"dataWritten":121,"copyDuration":5,
               "throughput":0.0236328132,"errors":[]}",
          "Error": "{
              "errorCode": "null",
              "message": "null",
              "failureType": "null",
              "target": "CopyBlobtoBlob"
        }
    }
}
```

| Свойство | Тип | Описание | Пример |
| --- | --- | --- | --- |
| **Уровень** |Строка | Уровень журналов диагностики. Для журналов выполнения действий задайте для свойства значение 4. | `4` |
| **correlationId** |Строка | Уникальный идентификатор для отслеживания конкретного запроса. | `319dc6b4-f348-405e-b8d7-aafc77b73e77` |
| **time** | Строка | Время события в формате UTC TimeSpan `YYYY-MM-DDTHH:MM:SS.00000Z` . | `2017-06-28T21:00:27.3534352Z` |
|**activityRunId**| Строка| Идентификатор выполнения действия. | `3a171e1f-b36e-4b80-8a54-5625394f4354` |
|**pipelineRunId**| Строка| Идентификатор выполнения конвейера. | `9f6069d6-e522-4608-9f99-21807bfc3c70` |
|**resourceId**| Строка | Идентификатор, связанный с ресурсом фабрики данных. | `/SUBSCRIPTIONS/<subID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |
|**category**| Строка | Категория журналов диагностики. В качестве значения свойства задайте `ActivityRuns`. | `ActivityRuns` |
|**level**| Строка | Уровень журналов диагностики. В качестве значения свойства задайте `Informational`. | `Informational` |
|**operationName**| Строка | Имя действия с его состоянием. Если действие является начальным пульсом, значение свойства равно `MyActivity -` . Если действие является конечным периодом пульса, свойство имеет значение `MyActivity - Succeeded` . | `MyActivity - Succeeded` |
|**пипелиненаме**| Строка | Имя конвейера. | `MyPipeline` |
|**activityName**| Строка | Имя действия. | `MyActivity` |
|**start**| Строка | Время начала действия выполняется в формате времени в формате UTC. | `2017-06-26T20:55:29.5007959Z`|
|**end**| Строка | Время окончания действия выполняется в формате времени в формате UTC. Если в журнале диагностики указано, что действие началось, но еще не завершено, свойство имеет значение `1601-01-01T00:00:00Z` . | `2017-06-26T20:55:29.5007959Z` |

#### <a name="pipeline-run-log-attributes"></a>Конвейер — выполнение атрибутов журнала

```json
{
   "Level": "",
   "correlationId":"",
   "time":"",
   "runId":"",
   "resourceId":"",
   "category":"PipelineRuns",
   "level":"Informational",
   "operationName":"",
   "pipelineName":"",
   "start":"",
   "end":"",
   "status":"",
   "properties":
    {
      "Parameters": {
        "<parameter1Name>": "<parameter1Value>"
      },
      "SystemParameters": {
        "ExecutionStart": "",
        "TriggerId": "",
        "SubscriptionId": ""
      }
    }
}
```

| Свойство | Тип | Описание | Пример |
| --- | --- | --- | --- |
| **Уровень** |Строка | Уровень журналов диагностики. Для журналов выполнения действий задайте для свойства значение 4. | `4` |
| **correlationId** |Строка | Уникальный идентификатор для отслеживания конкретного запроса. | `319dc6b4-f348-405e-b8d7-aafc77b73e77` |
| **time** | Строка | Время события в формате UTC TimeSpan `YYYY-MM-DDTHH:MM:SS.00000Z` . | `2017-06-28T21:00:27.3534352Z` |
|**runId**| Строка| Идентификатор выполнения конвейера. | `9f6069d6-e522-4608-9f99-21807bfc3c70` |
|**resourceId**| Строка | Идентификатор, связанный с ресурсом фабрики данных. | `/SUBSCRIPTIONS/<subID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |
|**category**| Строка | Категория журналов диагностики. В качестве значения свойства задайте `PipelineRuns`. | `PipelineRuns` |
|**level**| Строка | Уровень журналов диагностики. В качестве значения свойства задайте `Informational`. | `Informational` |
|**operationName**| Строка | Имя конвейера и его состояние. По завершении выполнения конвейера свойство имеет значение `Pipeline - Succeeded` . | `MyPipeline - Succeeded`. |
|**пипелиненаме**| Строка | Имя конвейера. | `MyPipeline` |
|**start**| Строка | Время начала действия выполняется в формате времени в формате UTC. | `2017-06-26T20:55:29.5007959Z`. |
|**end**| Строка | Время окончания действия выполняется в формате времени в формате UTC. Если в журнале диагностики указано, что действие началось, но еще не завершено, свойство имеет значение `1601-01-01T00:00:00Z` .  | `2017-06-26T20:55:29.5007959Z` |
|**status**| Строка | Конечное состояние выполнения конвейера. Возможные значения свойств: `Succeeded` и `Failed` . | `Succeeded`|

#### <a name="trigger-run-log-attributes"></a>Атрибуты журнала запуска триггера

```json
{
   "Level": "",
   "correlationId":"",
   "time":"",
   "triggerId":"",
   "resourceId":"",
   "category":"TriggerRuns",
   "level":"Informational",
   "operationName":"",
   "triggerName":"",
   "triggerType":"",
   "triggerEvent":"",
   "start":"",
   "status":"",
   "properties":
   {
      "Parameters": {
        "TriggerTime": "",
       "ScheduleTime": ""
      },
      "SystemParameters": {}
    }
}
```

| Свойство | Тип | Описание | Пример |
| --- | --- | --- | --- |
| **Уровень** |Строка | Уровень журналов диагностики. Для журналов выполнения действий задайте для свойства значение 4. | `4` |
| **correlationId** |Строка | Уникальный идентификатор для отслеживания конкретного запроса. | `319dc6b4-f348-405e-b8d7-aafc77b73e77` |
| **time** | Строка | Время события в формате UTC TimeSpan `YYYY-MM-DDTHH:MM:SS.00000Z` . | `2017-06-28T21:00:27.3534352Z` |
|**triggerId**| Строка| Идентификатор запуска триггера. | `08587023010602533858661257311` |
|**resourceId**| Строка | Идентификатор, связанный с ресурсом фабрики данных. | `/SUBSCRIPTIONS/<subID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |
|**category**| Строка | Категория журналов диагностики. В качестве значения свойства задайте `PipelineRuns`. | `PipelineRuns` |
|**level**| Строка | Уровень журналов диагностики. В качестве значения свойства задайте `Informational`. | `Informational` |
|**operationName**| Строка | Имя триггера с окончательным состоянием, которое указывает, успешно ли запущен триггер. Если пульс прошел успешно, значение свойства равно `MyTrigger - Succeeded` . | `MyTrigger - Succeeded` |
|**triggerName**| Строка | Имя триггера. | `MyTrigger` |
|**triggerType может принимать**| Строка | Тип триггера. Возможные значения свойств: `Manual Trigger` и `Schedule Trigger` . | `ScheduleTrigger` |
|**triggerEvent**| Строка | Событие триггера. | `ScheduleTime - 2017-07-06T01:50:25Z` |
|**start**| Строка | Время начала срабатывания триггера в формате TimeSpan UTC. | `2017-06-26T20:55:29.5007959Z`|
|**status**| Строка | Конечное состояние, показывающее, успешно ли вызван триггер. Возможные значения свойств: `Succeeded` и `Failed` . | `Succeeded`|

#### <a name="ssis-integration-runtime-log-attributes"></a>Атрибуты журнала среды выполнения интеграции SSIS

Ниже приведены атрибуты журнала операций запуска, отмены и обслуживания служб SSIS IR.

```json
{
   "time": "",
   "operationName": "",
   "category": "",
   "correlationId": "",
   "dataFactoryName": "",
   "integrationRuntimeName": "",
   "level": "",
   "resultType": "",
   "properties": {
      "message": ""
   },
   "resourceId": ""
}
```

| Свойство                   | Тип   | Описание                                                   | Пример                        |
| -------------------------- | ------ | ------------------------------------------------------------- | ------------------------------ |
| **time**                   | Строка | Время события в формате UTC: `YYYY-MM-DDTHH:MM:SS.00000Z` | `2017-06-28T21:00:27.3534352Z` |
| **operationName**          | Строка | Имя операции со IR для служб SSIS                            | `Start/Stop/Maintenance` |
| **category**               | Строка | Категория журналов диагностики                               | `SSISIntegrationRuntimeLogs` |
| **correlationId**          | Строка | Уникальный идентификатор для отслеживания определенной операции             | `f13b159b-515f-4885-9dfa-a664e949f785Deprovision0059035558` |
| **dataFactoryName**        | Строка | Имя ADF                                          | `MyADFv2` |
| **интегратионрунтименаме** | Строка | Имя службы SSIS IR                                      | `MySSISIR` |
| **level**                  | Строка | Уровень журналов диагностики                                  | `Informational` |
| **resultType**             | Строка | Результат работы службы SSIS IR                          | `Started/InProgress/Succeeded/Failed` |
| **message**                | Строка | Выходное сообщение для операции IR службы SSIS                  | `The stopping of your SSIS integration runtime has succeeded.` |
| **resourceId**             | Строка | Уникальный идентификатор ресурса ADF                            | `/SUBSCRIPTIONS/<subscriptionID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |

#### <a name="ssis-event-message-context-log-attributes"></a>Атрибуты журнала контекста сообщения о событиях служб SSIS

Ниже приведены атрибуты журнала условий, связанные с сообщениями о событиях, создаваемыми при выполнении пакетов служб SSIS в среде IR. Они сообщают подобную информацию в виде [таблицы или представления контекста сообщений о событиях в каталоге служб SSIS](/sql/integration-services/system-views/catalog-event-message-context) , отображающем значения времени выполнения многих свойств пакета служб SSIS. Они создаются при выборе `Basic/Verbose` уровня ведения журнала и удобного для отладки и проверки соответствия.

```json
{
   "time": "",
   "operationName": "",
   "category": "",
   "correlationId": "",
   "dataFactoryName": "",
   "integrationRuntimeName": "",
   "level": "",
   "properties": {
      "operationId": "",
      "contextDepth": "",
      "packagePath": "",
      "contextType": "",
      "contextSourceName": "",
      "contextSourceId": "",
      "propertyName": "",
      "propertyValue": ""
   },
   "resourceId": ""
}
```

| Свойство                   | Тип   | Описание                                                          | Пример                        |
| -------------------------- | ------ | -------------------------------------------------------------------- | ------------------------------ |
| **time**                   | Строка | Время события в формате UTC: `YYYY-MM-DDTHH:MM:SS.00000Z`        | `2017-06-28T21:00:27.3534352Z` |
| **operationName**          | Строка | Для этого параметра задается значение `YourSSISIRName-SSISPackageEventMessageContext`       | `mysqlmissisir-SSISPackageEventMessageContext` |
| **category**               | Строка | Категория журналов диагностики                                      | `SSISPackageEventMessageContext` |
| **correlationId**          | Строка | Уникальный идентификатор для отслеживания определенной операции                    | `e55700df-4caf-4e7c-bfb8-78ac7d2f28a0` |
| **dataFactoryName**        | Строка | Имя ADF                                                 | `MyADFv2` |
| **интегратионрунтименаме** | Строка | Имя службы SSIS IR                                             | `MySSISIR` |
| **level**                  | Строка | Уровень журналов диагностики                                         | `Informational` |
| **operationId**            | Строка | Уникальный идентификатор для отслеживания определенной операции в SSISDB          | `1` (1 означает операции, связанные с пакетами, которые **не** ХРАНЯТСЯ в SSISDB/Invoked через T-SQL) |
| **контекстдепс**           | Строка | Глубина контекста сообщения о событии                              | `0` (0 означает контекст перед началом выполнения пакета, 1 означает контекст при возникновении ошибки и увеличивается по мере того, как контекст находится дальше от ошибки). |
| **packagePath**            | Строка | Путь к объекту пакета в качестве источника контекста сообщения о событии      | `\Package` |
| **contextType**            | Строка | Тип объекта пакета в качестве источника контекста сообщения о событии      | `60`(см. [Дополнительные типы контекста](/sql/integration-services/system-views/catalog-event-message-context#remarks)) |
| **контекстсаурценаме**      | Строка | Имя объекта пакета в качестве источника контекста сообщения о событии      | `MyPackage` |
| **контекстсаурцеид**        | Строка | Уникальный идентификатор объекта пакета в качестве источника контекста сообщения о событии | `{E2CF27FB-EA48-41E9-AF6F-3FE938B4ADE1}` |
| **propertyName**           | Строка | Имя свойства пакета для источника контекста сообщения о событии   | `DelayValidation` |
| **propertyValue**          | Строка | Значение свойства Package для источника контекста сообщения о событии  | `False` |
| **resourceId**             | Строка | Уникальный идентификатор ресурса ADF                                   | `/SUBSCRIPTIONS/<subscriptionID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |

#### <a name="ssis-event-messages-log-attributes"></a>Атрибуты журнала сообщений о событиях служб SSIS

Ниже приведены атрибуты журнала сообщений о событиях, созданных при выполнении пакетов служб SSIS в среде IR служб SSIS. Они представляют аналогичные сведения в [таблице или представлении сообщений о событиях SSISDB](/sql/integration-services/system-views/catalog-event-messages) , в которых показан подробный текст и метаданные сообщений о событиях. Они создаются на любом уровне ведения журнала, за исключением `None` .

```json
{
   "time": "",
   "operationName": "",
   "category": "",
   "correlationId": "",
   "dataFactoryName": "",
   "integrationRuntimeName": "",
   "level": "",
   "properties": {
      "operationId": "",
      "messageTime": "",
      "messageType": "",
      "messageSourceType": "",
      "message": "",
      "packageName": "",
      "eventName": "",
      "messageSourceName": "",
      "messageSourceId": "",
      "subcomponentName": "",
      "packagePath": "",
      "executionPath": "",
      "threadId": ""
   }
}
```

| Свойство                   | Тип   | Описание                                                        | Пример                        |
| -------------------------- | ------ | ------------------------------------------------------------------ | ------------------------------ |
| **time**                   | Строка | Время события в формате UTC: `YYYY-MM-DDTHH:MM:SS.00000Z`      | `2017-06-28T21:00:27.3534352Z` |
| **operationName**          | Строка | Для этого параметра задается значение `YourSSISIRName-SSISPackageEventMessages`           | `mysqlmissisir-SSISPackageEventMessages` |
| **category**               | Строка | Категория журналов диагностики                                    | `SSISPackageEventMessages` |
| **correlationId**          | Строка | Уникальный идентификатор для отслеживания определенной операции                  | `e55700df-4caf-4e7c-bfb8-78ac7d2f28a0` |
| **dataFactoryName**        | Строка | Имя ADF                                               | `MyADFv2` |
| **интегратионрунтименаме** | Строка | Имя службы SSIS IR                                           | `MySSISIR` |
| **level**                  | Строка | Уровень журналов диагностики                                       | `Informational` |
| **operationId**            | Строка | Уникальный идентификатор для отслеживания определенной операции в SSISDB        | `1` (1 означает операции, связанные с пакетами, которые **не** ХРАНЯТСЯ в SSISDB/Invoked через T-SQL) |
| **мессажетиме**            | Строка | Время создания сообщения о событии в формате UTC          | `2017-06-28T21:00:27.3534352Z` |
| **messageType**            | Строка | Тип сообщения о событии                                     | `70`(см. [Дополнительные типы сообщений](/sql/integration-services/system-views/catalog-operation-messages-ssisdb-database#remarks)) |
| **messageSourceType**      | Строка | Тип источника сообщения о событии                              | `20`(см. [Дополнительные типы источников сообщений](/sql/integration-services/system-views/catalog-operation-messages-ssisdb-database#remarks)) |
| **message**                | Строка | Текст сообщения о событии                                     | `MyPackage:Validation has started.` |
| **packageName**            | Строка | Имя выполненного файла пакета                             | `MyPackage.dtsx` |
| **eventName**              | Строка | Имя связанного события времени выполнения                                 | `OnPreValidate` |
| **мессажесаурценаме**      | Строка | Имя компонента пакета в качестве источника сообщения о событии         | `Data Flow Task` |
| **мессажесаурцеид**        | Строка | Уникальный идентификатор компонента пакета в качестве источника сообщения о событии    | `{1a45a5a4-3df9-4f02-b818-ebf583829ad2}    ` |
| **субкомпонентнаме**       | Строка | Имя компонента потока данных в качестве источника сообщения о событии       | `SSIS.Pipeline` |
| **packagePath**            | Строка | Путь к объекту пакета в качестве источника сообщения о событии            | `\Package\Data Flow Task` |
| **ексекутионпас**          | Строка | Полный путь от родительского пакета к выполненному компоненту            | `\Transformation\Data Flow Task` (Этот путь также захватывает итерации компонента) |
| **Tидентификатор**               | Строка | Уникальный идентификатор потока, выполняемого при занесении в журнал сообщения о событии | `{1a45a5a4-3df9-4f02-b818-ebf583829ad2}    ` |

#### <a name="ssis-executable-statistics-log-attributes"></a>Атрибуты журнала исполняемой статистики служб SSIS

Ниже приведены атрибуты журнала исполняемой статистики, создаваемые пакетами служб SSIS для среды выполнения интеграции SSIS, где исполняемые объекты — это контейнеры или задачи в потоке управления пакетами. Они сообщают подобную информацию в виде [таблицы или представления для статистики исполняемого файла SSISDB](/sql/integration-services/system-views/catalog-executable-statistics) , которая показывает строку для каждого выполняемого исполняемого файла, включая его итерации. Они создаются на любом уровне ведения журнала `None` , кроме и полезны для выявления узких мест и сбоев на уровне задач.

```json
{
   "time": "",
   "operationName": "",
   "category": "",
   "correlationId": "",
   "dataFactoryName": "",
   "integrationRuntimeName": "",
   "level": "",
   "properties": {
      "executionId": "",
      "executionPath": "",
      "startTime": "",
      "endTime": "",
      "executionDuration": "",
      "executionResult": "",
      "executionValue": ""
   },
   "resourceId": ""
}
```

| Свойство                   | Тип   | Описание                                                      | Пример                        |
| -------------------------- | ------ | ---------------------------------------------------------------- | ------------------------------ |
| **time**                   | Строка | Время события в формате UTC: `YYYY-MM-DDTHH:MM:SS.00000Z`    | `2017-06-28T21:00:27.3534352Z` |
| **operationName**          | Строка | Для этого параметра задается значение `YourSSISIRName-SSISPackageExecutableStatistics`  | `mysqlmissisir-SSISPackageExecutableStatistics` |
| **category**               | Строка | Категория журналов диагностики                                  | `SSISPackageExecutableStatistics` |
| **correlationId**          | Строка | Уникальный идентификатор для отслеживания определенной операции                | `e55700df-4caf-4e7c-bfb8-78ac7d2f28a0` |
| **dataFactoryName**        | Строка | Имя ADF                                             | `MyADFv2` |
| **интегратионрунтименаме** | Строка | Имя службы SSIS IR                                         | `MySSISIR` |
| **level**                  | Строка | Уровень журналов диагностики                                     | `Informational` |
| **executionId**            | Строка | Уникальный идентификатор для отслеживания определенного выполнения в SSISDB      | `1` (1 обозначает выполнение, связанное с пакетами, которые **не** ХРАНЯТСЯ в SSISDB/Invoked через T-SQL) |
| **ексекутионпас**          | Строка | Полный путь от родительского пакета к выполненному компоненту          | `\Transformation\Data Flow Task` (Этот путь также захватывает итерации компонента) |
| **startTime**              | Строка | Время, когда исполняемый объект вводит этап перед выполнением в формате UTC  | `2017-06-28T21:00:27.3534352Z` |
| **Завершения**                | Строка | Время, когда исполняемый файл переходит в фазу после выполнения в формате UTC | `2017-06-28T21:00:27.3534352Z` |
| **ексекутиондуратион**      | Строка | Время выполнения исполняемого файла в миллисекундах                   | `1,125` |
| **executionResult**        | Строка | Результат выполнения исполняемого файла                                 | `0` (0 означает успех, 1 означает сбой, 2 означает завершение, а 3 означает отмену) |
| **executionValue**         | Строка | Определяемое пользователем значение, возвращаемое выполняемым исполняемым файлом            | `1` |
| **resourceId**             | Строка | Уникальный идентификатор ресурса ADF                               | `/SUBSCRIPTIONS/<subscriptionID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |

#### <a name="ssis-execution-component-phases-log-attributes"></a>Атрибуты журнала фаз компонента выполнения служб SSIS

Ниже приведены атрибуты журнала статистики времени выполнения для компонентов потока данных, созданных при выполнении пакетов служб SSIS в среде IR. Они представляют подобную информацию в виде [таблицы или представления этапов выполнения SSISDB](/sql/integration-services/system-views/catalog-execution-component-phases) , которые показывают время, затраченное компонентами потока данных на все этапы выполнения. Они создаются при выборе `Performance/Verbose` уровня ведения журнала и удобно для сбора статистики выполнения потока данных.

```json
{
   "time": "",
   "operationName": "",
   "category": "",
   "correlationId": "",
   "dataFactoryName": "",
   "integrationRuntimeName": "",
   "level": "",
   "properties": {
      "executionId": "",
      "packageName": "",
      "taskName": "",
      "subcomponentName": "",
      "phase": "",
      "startTime": "",
      "endTime": "",
      "executionPath": ""
   },
   "resourceId": ""
}
```

| Свойство                   | Тип   | Описание                                                         | Пример                        |
| -------------------------- | ------ | ------------------------------------------------------------------- | ------------------------------ |
| **time**                   | Строка | Время события в формате UTC: `YYYY-MM-DDTHH:MM:SS.00000Z`       | `2017-06-28T21:00:27.3534352Z` |
| **operationName**          | Строка | Для этого параметра задается значение `YourSSISIRName-SSISPackageExecutionComponentPhases` | `mysqlmissisir-SSISPackageExecutionComponentPhases` |
| **category**               | Строка | Категория журналов диагностики                                     | `SSISPackageExecutionComponentPhases` |
| **correlationId**          | Строка | Уникальный идентификатор для отслеживания определенной операции                   | `e55700df-4caf-4e7c-bfb8-78ac7d2f28a0` |
| **dataFactoryName**        | Строка | Имя ADF                                                | `MyADFv2` |
| **интегратионрунтименаме** | Строка | Имя службы SSIS IR                                            | `MySSISIR` |
| **level**                  | Строка | Уровень журналов диагностики                                        | `Informational` |
| **executionId**            | Строка | Уникальный идентификатор для отслеживания определенного выполнения в SSISDB         | `1` (1 обозначает выполнение, связанное с пакетами, которые **не** ХРАНЯТСЯ в SSISDB/Invoked через T-SQL) |
| **packageName**            | Строка | Имя выполненного файла пакета                              | `MyPackage.dtsx` |
| **имя_задания**               | Строка | Имя выполненной задачи потока данных                                 | `Data Flow Task` |
| **субкомпонентнаме**       | Строка | Имя компонента потока данных                                     | `Derived Column` |
| **этапах**                  | Строка | Имя фазы выполнения                                         | `AcquireConnections` |
| **startTime**              | Строка | Время начала фазы выполнения в формате UTC                  | `2017-06-28T21:00:27.3534352Z` |
| **Завершения**                | Строка | Время окончания фазы выполнения в формате UTC                    | `2017-06-28T21:00:27.3534352Z` |
| **ексекутионпас**          | Строка | Путь выполнения задачи потока данных                            | `\Transformation\Data Flow Task` |
| **resourceId**             | Строка | Уникальный идентификатор ресурса ADF                                  | `/SUBSCRIPTIONS/<subscriptionID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |

#### <a name="ssis-execution-data-statistics-log-attributes"></a>Атрибуты журнала статистики данных выполнения служб SSIS

Ниже приведены атрибуты журнала перемещений данных в каждом контейнере конвейеров потоков данных, от вышестоящего до нижестоящих компонентов, которые создаются пакетами служб SSIS в среде выполнения интеграции служб SSIS. Они сообщают подобную информацию в виде [таблицы или представления статистики данных выполнения SSISDB](/sql/integration-services/system-views/catalog-execution-data-statistics) , показывающего количество строк данных, перемещаемых с помощью задач потока данных. Они создаются при выборе `Verbose` уровня ведения журнала и удобного для вычисления пропускной способности потока данных.

```json
{
   "time": "",
   "operationName": "",
   "category": "",
   "correlationId": "",
   "dataFactoryName": "",
   "integrationRuntimeName": "",
   "level": "",
   "properties": {
      "executionId": "",
      "packageName": "",
      "taskName": "",
      "dataflowPathIdString": "",
      "dataflowPathName": "",
      "sourceComponentName": "",
      "destinationComponentName": "",
      "rowsSent": "",
      "createdTime": "",
      "executionPath": ""
   },
   "resourceId": ""
}
```

| Свойство                     | Тип   | Описание                                                        | Пример                        |
| ---------------------------- | ------ | ------------------------------------------------------------------ | ------------------------------ |
| **time**                     | Строка | Время события в формате UTC: `YYYY-MM-DDTHH:MM:SS.00000Z`      | `2017-06-28T21:00:27.3534352Z` |
| **operationName**            | Строка | Для этого параметра задается значение `YourSSISIRName-SSISPackageExecutionDataStatistics` | `mysqlmissisir-SSISPackageExecutionDataStatistics` |
| **category**                 | Строка | Категория журналов диагностики                                    | `SSISPackageExecutionDataStatistics` |
| **correlationId**            | Строка | Уникальный идентификатор для отслеживания определенной операции                  | `e55700df-4caf-4e7c-bfb8-78ac7d2f28a0` |
| **dataFactoryName**          | Строка | Имя ADF                                               | `MyADFv2` |
| **интегратионрунтименаме**   | Строка | Имя службы SSIS IR                                           | `MySSISIR` |
| **level**                    | Строка | Уровень журналов диагностики                                       | `Informational` |
| **executionId**              | Строка | Уникальный идентификатор для отслеживания определенного выполнения в SSISDB        | `1` (1 обозначает выполнение, связанное с пакетами, которые **не** ХРАНЯТСЯ в SSISDB/Invoked через T-SQL) |
| **packageName**              | Строка | Имя выполненного файла пакета                             | `MyPackage.dtsx` |
| **имя_задания**                 | Строка | Имя выполненной задачи потока данных                                | `Data Flow Task` |
| **датафловпасидстринг**     | Строка | Уникальный идентификатор для отслеживания пути потока данных                          | `Paths[SQLDB Table3.ADO NET Source Output]` |
| **датафловпаснаме**         | Строка | Имя пути потока данных                                         | `ADO NET Source Output` |
| **саурцекомпонентнаме**      | Строка | Имя компонента потока данных, который отправляет данные                    | `SQLDB Table3` |
| **дестинатионкомпонентнаме** | Строка | Имя компонента потока данных, который получает данные                 | `Derived Column` |
| **ровссент**                 | Строка | Число строк, отправленных компонентом источника                        | `500` |
| **Время создания**              | Строка | Время получения значений строк в формате UTC                | `2017-06-28T21:00:27.3534352Z` |
| **ексекутионпас**            | Строка | Путь выполнения задачи потока данных                           | `\Transformation\Data Flow Task` |
| **resourceId**               | Строка | Уникальный идентификатор ресурса ADF                                 | `/SUBSCRIPTIONS/<subscriptionID>/RESOURCEGROUPS/<resourceGroupName>/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/<dataFactoryName>` |

### <a name="log-analytics-schema"></a>Схема Log Analytics

Log Analytics наследует схему от Monitor со следующими исключениями:

* Первая буква в имени каждого столбца задается прописной буквой. Например, имя столбца «correlationId» в параметре Monitor имеет значение «CorrelationId» в Log Analytics.
* Нет столбца "Level".
* Динамический столбец "Properties" сохраняется как следующий динамический тип больших двоичных объектов JSON.

    | Azure Monitor столбец | Log Analytics столбец | Тип |
    | --- | --- | --- |
    | $. Properties. усерпропертиес | UserProperties | Динамический |
    | $. Properties. Примечания | Заметки | Динамический |
    | $. Properties. Входной | Входные данные | Динамический |
    | $. Properties. Проверки | Выходные данные | Динамический |
    | $. Properties. Ошибка. errorCode | ErrorCode | INT |
    | $. Properties. Ошибка. сообщение | ErrorMessage | строка |
    | $. Properties. План | Ошибка | Динамический |
    | $. Properties. Предшественников | Предшественников | Динамический |
    | $. Properties. Вход | Параметры | Динамический |
    | Темпараметерс $.properties.Sys | SystemParameters | Динамический |
    | $. Properties. Тэги | Теги | Динамический |

## <a name="monitor-ssis-operations-with-azure-monitor"></a>Мониторинг операций служб SSIS с помощью Azure Monitor

Чтобы претянуть & смены рабочих нагрузок служб SSIS, можно [подготавливать службы SSIS IR в ADF](./tutorial-deploy-ssis-packages-azure.md) , поддерживающем:

- выполнение пакетов, развернутых в каталоге SSIS (SSISDB), которые размещаются на сервере Базы данных SQL Azure или в Управляемом экземпляре (модель развертывания для проектов);
- выполнение пакетов, развернутых в файловой системе, службе "Файлы Azure" или базе данных SQL Server (MSDB), которые размещаются в Управляемом экземпляре SQL Azure (модель развертывания для пакетов).

После подготовки можно [проверить состояние работы служб SSIS IR с помощью Azure PowerShell или в центре **мониторинга** на портале ADF](./monitor-integration-runtime.md#azure-ssis-integration-runtime). При использовании модели развертывания проекта журналы выполнения пакетов служб SSIS хранятся в внутренних таблицах и представлениях SSISDB, что позволяет выполнять запросы, анализировать и визуально представлять их с помощью определенных средств, таких как SSMS. При использовании модели развертывания пакетов журналы выполнения пакетов служб SSIS могут храниться в файловой системе или в файлах Azure в виде CSV-файлов, которые необходимо проанализировать и обработать с помощью других назначенных средств, прежде чем можно будет выполнять запросы, анализировать и визуально представлять их.

Теперь с [Azure Monitorной](../azure-monitor/data-platform.md) интеграцией можно запрашивать, анализировать и визуально представлять все метрики и журналы, созданные с помощью IR-операций SSIS и выполнения пакетов служб ssis на портал Azure. Кроме того, на них можно создавать оповещения.

### <a name="configure-diagnostic-settings-and-workspace-for-ssis-operations"></a>Настройка параметров диагностики и рабочей области для операций служб SSIS

Чтобы отправить все метрики и журналы, созданные с помощью IR-операций служб SSIS и выполнения пакетов служб SSIS, в Azure Monitor, необходимо [настроить параметры диагностики и рабочую область для ADF](#configure-diagnostic-settings-and-workspace)-файла.

### <a name="ssis-operational-metrics"></a>Операционные метрики служб SSIS

Операционные [метрики](../azure-monitor/essentials/data-platform-metrics.md) служб SSIS — это счетчики производительности или числовые значения, описывающие состояние операций запуска и завершения IR служб SSIS, а также выполнение пакетов служб SSIS в определенный момент времени. Они входят в состав [метрик ADF в Azure Monitor](#data-factory-metrics).

Когда вы настраиваете параметры диагностики и рабочую область для ADF на Azure Monitor, Установка флажка _аллметрикс_ сделает операционные МЕТРИКИ служб SSIS доступными для [интерактивного анализа с помощью обозреватель метрик Azure](../azure-monitor/essentials/metrics-getting-started.md), [представления на панели мониторинга Azure](../azure-monitor/app/tutorial-app-dashboards.md)и [оповещений практически в реальном времени](../azure-monitor/alerts/alerts-metric.md).

![Назовите свои параметры и выберите рабочую область log-Analytics.](media/data-factory-monitor-oms/monitor-oms-image2.png)

### <a name="ssis-operational-alerts"></a>Операционные оповещения служб SSIS

Чтобы создавать оповещения для операционных метрик служб SSIS на портале ADF, [выберите страницу **оповещения & метрик** центра **мониторинга** ADF и следуйте](./monitor-visually.md#alerts)пошаговым инструкциям.

![Создание операционных оповещений служб SSIS на портале ADF](media/data-factory-monitor-oms/data-factory-monitor-alerts-ssis.png)

Чтобы создавать оповещения о рабочих метриках служб SSIS из портал Azure, [выберите страницу **оповещения** в центре **мониторинга** Azure и следуйте](#data-factory-alerts)пошаговым инструкциям.

![Создание операционных оповещений служб SSIS из портал Azure](media/data-factory-monitor-oms/azure-monitor-alerts-ssis.png)

### <a name="ssis-operational-logs"></a>Операционные журналы служб SSIS

Операционные [журналы](../azure-monitor/logs/data-platform-logs.md) служб SSIS — это события, СОЗДАВАЕМые IR-ОПЕРАЦИЯми служб SSIS и выполнения пакетов служб SSIS, которые обеспечивают достаточный контекст при обнаруженных проблемах и полезны для анализа основных причин. 

При настройке параметров диагностики и рабочей области для ADF на Azure Monitor можно выбрать соответствующие рабочие журналы служб SSIS и отправить их Log Analytics на основе обозреватель данных Azure. В нем они будут доступны для [анализа с помощью расширенного языка запросов](../azure-monitor/logs/log-query-overview.md), [представления на панели мониторинга Azure](../azure-monitor/app/tutorial-app-dashboards.md)и [оповещений практически в реальном времени](../azure-monitor/alerts/alerts-log.md).

![Назовите свои параметры и выберите рабочую область log-Analytics.](media/data-factory-monitor-oms/monitor-oms-image2.png)

Схемы и содержимое журналов выполнения пакетов служб SSIS в Azure Monitor и Log Analytics похожи на схемы внутренних таблиц и представлений SSISDB.

| Категории журналов Azure Monitor          | Log Analytics таблицы                     | Внутренние таблицы и представления SSISDB              |
| ------------------------------------- | ---------------------------------------- | ----------------------------------------- |
| `SSISIntegrationRuntimeLogs`          | `ADFSSISIntegrationRuntimeLogs`          |                                           |
| `SSISPackageEventMessageContext`      | `ADFSSISPackageEventMessageContext`      | `[internal].[event_message_context]`      |
| `SSISPackageEventMessages`            | `ADFSSISPackageEventMessages`            | `[internal].[event_messages]`             |
| `SSISPackageExecutableStatistics`     | `ADFSSISPackageExecutableStatistics`     | `[internal].[executable_statistics]`      |
| `SSISPackageExecutionComponentPhases` | `ADFSSISPackageExecutionComponentPhases` | `[internal].[execution_component_phases]` |
| `SSISPackageExecutionDataStatistics`  | `ADFSSISPackageExecutionDataStatistics`  | `[internal].[execution_data_statistics]`  |

Дополнительные сведения о атрибутах и свойствах журнала служб SSIS см. в разделе [схемы Azure Monitor и log Analytics для ADF](#schema-of-logs-and-events).

Выбранные журналы выполнения пакетов служб SSIS всегда отправляются на Log Analytics независимо от их методов вызова. Например, можно вызывать выполнение пакетов в SSDT с поддержкой Azure с помощью T-SQL в SSMS, агент SQL Server или других назначенных средств, а также в качестве активированных или отладочных запусков выполнения действий пакетов служб SSIS в конвейерах ADF.

При запросе журналов операций со IR для служб SSIS в журналах аналитики можно использовать свойства **OperationName** и **resultType** , для которых заданы значения `Start/Stop/Maintenance` и `Started/InProgress/Succeeded/Failed` соответственно. 

![Запрос журналов операций службы SSIS IR на Log Analytics](media/data-factory-monitor-oms/log-analytics-query.png)

При запросе журналов выполнения пакетов служб SSIS в службе журналов Analytics их можно **объединить с помощью** / свойств **ExecutionId** / **correlationId** . С идентификатором  / Для **ExecutionId** всегда задано значение `1` для всех операций и выполнений, связанных с пакетами, которые **не** хранятся в SSISDB или вызываются через T-SQL.

![Запрос журналов выполнения пакетов служб SSIS на Log Analytics](media/data-factory-monitor-oms/log-analytics-query2.png)

## <a name="next-steps"></a>Следующие шаги
[Программное отслеживание конвейеров и управление ими](monitor-programmatically.md)
