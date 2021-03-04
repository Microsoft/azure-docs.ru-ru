---
title: Как активировать сложные действия с помощью оповещений Azure Monitor
description: Сведения о том, как создать действие приложения логики для обработки оповещений Azure Monitor.
author: dkamstra
ms.author: dukek
ms.topic: conceptual
ms.date: 02/19/2021
ms.openlocfilehash: a1371e00a6d4c5db609466e25c9d94aad5e73398
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102045723"
---
# <a name="how-to-trigger-complex-actions-with-azure-monitor-alerts"></a>Как активировать сложные действия с помощью оповещений Azure Monitor

Эта статья описывает, как настроить и активировать приложение логики, чтобы создать беседу в Microsoft Teams при срабатывании оповещения.

## <a name="overview"></a>Обзор

Когда активируется оповещение Azure Monitor, оно вызывает [группу действий](./action-groups.md). Группы действий позволяют активировать одно или несколько действий для уведомления других лиц об оповещении и устранения его причины.

Общий процесс приведен ниже:

-   Создание приложения логики для соответствующего типа оповещений.

-   Импортируйте пример полезных данных для соответствующего типа оповещений в приложение логики.

-   Определение поведения приложения логики.

-   Копирование конечной точки HTTP приложения логики в группу действий Azure.

Если требуется, чтобы приложение логики выполняло другое действие, процесс настройки аналогичен.

## <a name="create-an-activity-log-alert-administrative"></a>Создание оповещения журнала действий на портале администрирования

1. [Создание приложения логики](~/articles/logic-apps/quickstart-create-first-logic-app-workflow.md)

2.  Выберите триггер **При получении HTTP-запроса**.

1. В диалоговом окне **При получении HTTP-запроса** выберите **Использовать пример полезной нагрузки, чтобы создать схему**.

    ![Снимок экрана: диалоговое окно When an HTTP request is received (При получении HTTP-запроса) и выбор параметра "Использовать пример полезной нагрузки, чтобы создать схему". ](~/articles/app-service/media/tutorial-send-email/generate-schema-with-payload.png)

3.  Скопируйте и вставьте приведенный ниже пример полезных данных в диалоговое окно.

    ```json
        {
            "schemaId": "Microsoft.Insights/activityLogs",
            "data": {
                "status": "Activated",
                "context": {
                "activityLog": {
                    "authorization": {
                    "action": "microsoft.insights/activityLogAlerts/write",
                    "scope": "/subscriptions/…"
                    },
                    "channels": "Operation",
                    "claims": "…",
                    "caller": "logicappdemo@contoso.com",
                    "correlationId": "91ad2bac-1afa-4932-a2ce-2f8efd6765a3",
                    "description": "",
                    "eventSource": "Administrative",
                    "eventTimestamp": "2018-04-03T22:33:11.762469+00:00",
                    "eventDataId": "ec74c4a2-d7ae-48c3-a4d0-2684a1611ca0",
                    "level": "Informational",
                    "operationName": "microsoft.insights/activityLogAlerts/write",
                    "operationId": "61f59fc8-1442-4c74-9f5f-937392a9723c",
                    "resourceId": "/subscriptions/…",
                    "resourceGroupName": "LOGICAPP-DEMO",
                    "resourceProviderName": "microsoft.insights",
                    "status": "Succeeded",
                    "subStatus": "",
                    "subscriptionId": "…",
                    "submissionTimestamp": "2018-04-03T22:33:36.1068742+00:00",
                    "resourceType": "microsoft.insights/activityLogAlerts"
                }
                },
                "properties": {}
            }
        }
    ```

9. Отображается всплывающее окно **Конструктор приложений логики** с напоминанием о том, что запрос, отправляемый в приложение логики, должен задавать в заголовке **Content-Type** значение **application/json**. Закройте всплывающее окно. Оповещение Azure Monitor задает заголовок.

    ![Задание заголовка Content-Type](media/action-groups-logic-app/content-type-header.png "Задание заголовка Content-Type")

10. Выберите **+** **новый шаг** и нажмите кнопку **Добавить действие**.

    ![Добавление действия](media/action-groups-logic-app/add-action.png "Добавление действия")

11. Найдите и выберите соединитель Microsoft Teams. Выберите действие **Microsoft Teams — отправить сообщение** .

    ![Действия Microsoft Teams](media/action-groups-logic-app/microsoft-teams-actions.png "Действия Microsoft Teams")

12. Настройте действие Microsoft Teams. **Конструктор Logic Apps** предлагает выполнить аутентификацию в рабочей или учебной учетной записи. Выберите значения параметров **Идентификатор команды** и **Идентификатор канала** для отправки сообщения.

13. Настройте сообщение, используя сочетание статического текста и ссылок на объект \<fields\> в динамическом содержимом. Скопируйте следующий текст и вставьте его в поле **Сообщение**:

    ```text
      Activity Log Alert: <eventSource>
      operationName: <operationName>
      status: <status>
      resourceId: <resourceId>
    ```

    Затем найдите и замените с помощью \<fields\> тегов динамического содержимого с тем же именем.

    > [!NOTE]
    > Имеются два динамических поля с именем **status**. Добавьте оба эти поля в сообщение. Используйте поле в контейнере свойств **activityLog** и удалите другое. Наведите указатель мыши на поле **status**, чтобы отобразить подсказку с полным именем поля, как показано на следующем снимке экрана:

    ![Действие Microsoft teams: публикация сообщения](media/action-groups-logic-app/teams-action-post-message.png "Действие Microsoft teams: публикация сообщения")

14. В верхней части окна **Конструктор Logic Apps** выберите **Сохранить**, чтобы сохранить приложение логики.

15. Откройте существующую группу действий и добавьте в нее действие, которое ссылается на приложение логики. Если у вас нет группы действий, см. раздел [Создание групп действий и управление ими в портал Azure](./action-groups.md) для ее создания. Обязательно сохраните изменения.

    ![Обновление группы действий](media/action-groups-logic-app/update-action-group.png "Обновление группы действий")

При следующем вызове группы действий оповещением будет вызвано ваше приложение логики.

## <a name="create-a-service-health-alert"></a>Создание оповещения о работоспособности служб

Записи работоспособности служб Azure являются частью журнала действий. Процедура создания оповещения похожа на [создание предупреждения журнала действий](#create-an-activity-log-alert-administrative), но с некоторыми отличиями:

- Шаги с 1 по 3 одинаковы.
- Для шага 4 используйте следующий пример полезных данных для триггера HTTP-запроса:

    ```json
    {
        "schemaId": "Microsoft.Insights/activityLogs",
        "data": {
            "status": "Activated",
            "context": {
                "activityLog": {
                    "channels": "Admin",
                    "correlationId": "e416ed3c-8874-4ec8-bc6b-54e3c92a24d4",
                    "description": "…",
                    "eventSource": "ServiceHealth",
                    "eventTimestamp": "2018-04-03T22:44:43.7467716+00:00",
                    "eventDataId": "9ce152f5-d435-ee31-2dce-104228486a6d",
                    "level": "Informational",
                    "operationName": "Microsoft.ServiceHealth/incident/action",
                    "operationId": "e416ed3c-8874-4ec8-bc6b-54e3c92a24d4",
                    "properties": {
                        "title": "...",
                        "service": "...",
                        "region": "Global",
                        "communication": "...",
                        "incidentType": "Incident",
                        "trackingId": "...",
                        "impactStartTime": "2018-03-22T21:40:00.0000000Z",
                        "impactMitigationTime": "2018-03-22T21:41:00.0000000Z",
                        "impactedServices": "[{\"ImpactedRegions\"}]",
                        "defaultLanguageTitle": "...",
                        "defaultLanguageContent": "...",
                        "stage": "Active",
                        "communicationId": "11000001466525",
                        "version": "0.1.1"
                    },
                    "status": "Active",
                    "subscriptionId": "...",
                    "submissionTimestamp": "2018-04-03T22:44:50.8013523+00:00"
                }
            },
            "properties": {}
        }
    }
    ```

-  Шаги 5 и 6 одинаковы.
-  Для шагов с 7 по 11 используйте следующую процедуру.

   1. Выберите **+** **новый шаг** и нажмите кнопку **Добавить условие**. Задайте следующие условия, чтобы приложение логики выполнялось, только когда входные данные соответствуют указанным ниже значениям.  При вводе значения версии в текстовое поле поместите его в кавычки ("0.1.1"), чтобы гарантировать, что оно будет вычисляться как строка, а не числовой тип.  Система не отображает кавычки, если вы вернетесь к странице, однако базовый код сохраняет строковый тип.   
       - `schemaId == Microsoft.Insights/activityLogs`
       - `eventSource == ServiceHealth`
       - `version == "0.1.1"`

      !["Условие полезных данных работоспособности службы"](media/action-groups-logic-app/service-health-payload-condition.png "Условие для полезных данных работоспособности службы")

   1. В условии **Если истинно** выполните инструкции для шагов с 11 по 13 из раздела [Создание оповещения журнала действий](#create-an-activity-log-alert-administrative), чтобы добавить действие Microsoft Teams.

   1. Определите сообщение, используя сочетание кода HTML и динамического содержимого. Скопируйте приведенный ниже текст и вставьте его в поле **Сообщение**. Замените поля `[incidentType]`, `[trackingID]`, `[title]` и `[communication]` тегами динамического содержимого с таким же именем:

       ```html
       <p>
       <b>Alert Type:</b>&nbsp;<strong>[incidentType]</strong>&nbsp;
       <strong>Tracking ID:</strong>&nbsp;[trackingId]&nbsp;
       <strong>Title:</strong>&nbsp;[title]</p>
       <p>
       <a href="https://ms.portal.azure.com/#blade/Microsoft_Azure_Health/AzureHealthBrowseBlade/serviceIssues">For details, log in to the Azure Service Health dashboard.</a>
       </p>
       <p>[communication]</p>
       ```

       !["Действие после истинного условия работоспособности службы"](media/action-groups-logic-app/service-health-true-condition-post-action.png "Действие после истинного условия работоспособности службы")

   1. Для условия **Если ложно** введите имеющее смысл сообщение:

       ```html
       <p><strong>Service Health Alert</strong></p>
       <p><b>Unrecognized alert schema</b></p>
       <p><a href="https://ms.portal.azure.com/#blade/Microsoft_Azure_Health/AzureHealthBrowseBlade/serviceIssues">For details, log in to the Azure Service Health dashboard.\</a></p>
       ```

       !["Действие после ложного состояния службы"](media/action-groups-logic-app/service-health-false-condition-post-action.png "Действие записи условия работоспособности службы")

- Шаг 15 остается без изменений. Следуйте инструкциям, чтобы сохранить приложение логики и обновить группу действий.

## <a name="create-a-metric-alert"></a>Создание оповещения для метрики

Процедура создания оповещения для метрики похожа на [создание предупреждения для журнала действий](#create-an-activity-log-alert-administrative), но с некоторыми отличиями:

- Шаги с 1 по 3 одинаковы.
- Для шага 4 используйте следующий пример полезных данных для триггера HTTP-запроса:

    ```json
    {
    "schemaId": "AzureMonitorMetricAlert",
    "data": {
        "version": "2.0",
        "status": "Activated",
        "context": {
        "timestamp": "2018-04-09T19:00:07.7461615Z",
        "id": "...",
        "name": "TEST-VM CPU Utilization",
        "description": "",
        "conditionType": "SingleResourceMultipleMetricCriteria",
        "condition": {
            "windowSize": "PT15M",
            "allOf": [
                {
                    "metricName": "Percentage CPU",
                    "dimensions": [
                    {
                        "name": "ResourceId",
                        "value": "d92fc5cb-06cf-4309-8c9a-538eea6a17a6"
                    }
                ],
                "operator": "GreaterThan",
                "threshold": "5",
                "timeAggregation": "PT15M",
                "metricValue": 1.0
            }
            ]
        },
        "subscriptionId": "...",
        "resourceGroupName": "TEST",
        "resourceName": "test-vm",
        "resourceType": "Microsoft.Compute/virtualMachines",
        "resourceId": "...",
        "portalLink": "..."
        },
        "properties": {}
    }
    }
    ```

- Шаги 5 и 6 одинаковы.
- Для шагов с 7 по 11 используйте следующую процедуру.

  1. Выберите **+** **новый шаг** и нажмите кнопку **Добавить условие**. Задайте следующие условия, чтобы приложение логики выполнялось, только когда входные данные соответствуют указанным ниже значениям. При вводе значения версии в текстовое поле поместите его в кавычки ("2.0"), чтобы гарантировать, что оно будет вычисляться как строка, а не числовой тип.  Система не отображает кавычки, если вы вернетесь к странице, однако базовый код сохраняет строковый тип. 
     - `schemaId == AzureMonitorMetricAlert`
     - `version == "2.0"`
       
       !["Условие полезных данных оповещений метрик"](media/action-groups-logic-app/metric-alert-payload-condition.png "Условие полезных данных оповещений метрик")

  1. В условии **Если истинно** добавьте цикл **For each** и действие Microsoft Teams. Определите сообщение, используя сочетание кода HTML и динамического содержимого.

      !["Действие после истинного условия оповещения метрики"](media/action-groups-logic-app/metric-alert-true-condition-post-action.png "Действие записи состояния истинного оповещения метрики")

  1. В условии **If false** Определите действие Microsoft Teams, чтобы сообщить, что оповещение метрики не соответствует ожиданиям приложения логики. Включите полезные данные JSON. Обратите внимание на то, как указываются ссылки на динамическое содержимое `triggerBody` в выражении `json()`.

      !["Действие записи условия оповещения метрики"](media/action-groups-logic-app/metric-alert-false-condition-post-action.png "Действие записи состояния "ложное оповещение метрики"")

- Шаг 15 остается без изменений. Следуйте инструкциям, чтобы сохранить приложение логики и обновить группу действий.

## <a name="calling-other-applications-besides-microsoft-teams"></a>Вызов других приложений помимо Microsoft Teams
Служба Logic Apps имеет ряд различных соединителей, которые позволяют активировать действия в самых разных приложениях и базах данных. К таким приложениям, например, относятся Slack, SQL Server, Oracle, Salesforce. Дополнительные сведения о соединителях Azure Logic Apps см. в [этой статье](../../connectors/apis-list.md).  

## <a name="next-steps"></a>Дальнейшие действия
* Изучите [обзор оповещений журнала действий Azure](./alerts-overview.md) и узнайте, как получать оповещения.  
* Узнайте, как [настроить оповещения при поступлении уведомлений о Работоспособности служб Azure](../../service-health/alerts-activity-log-service-notifications-portal.md).
* Дополнительные сведения о группах действий см. в статье [Создание групп действий и управление ими на портале Azure](./action-groups.md).