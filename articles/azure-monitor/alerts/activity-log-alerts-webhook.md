---
title: Общие сведения о схеме веб-перехватчика, используемой в оповещениях журнала действий
description: Дополнительные сведения о схеме JSON, отправляемой по URL-адресу веб-перехватчика при активации оповещения журнала действий.
ms.topic: conceptual
ms.date: 03/31/2017
ms.openlocfilehash: 31b9f4b41d741475a031efd4392c7df2fd2260c4
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102034342"
---
# <a name="webhooks-for-azure-activity-log-alerts"></a>Веб-перехватчики для оповещений журнала действий Azure
В определении группы действий можно настроить конечные точки веб-перехватчика для получения уведомлений об оповещениях журнала действий. С помощью веб-перехватчика можно направлять эти уведомления в другие системы для последующей обработки или выполнения настраиваемых действий. В этой статье показано, как выглядят полезные данные HTTP POST для webhook.

Дополнительные сведения об оповещениях журнала действий см. в статье [Создание оповещений журнала действий](./activity-log-alerts.md).

Сведения о группах действия см. в разделе [Создание групп действий и управление ими на портале Azure](./action-groups.md).

> [!NOTE]
> Вы также можете использовать [общую схему предупреждений](./alerts-common-schema.md), которая предоставляет преимущества единого расширяемого и унифицированного набора полезных данных оповещений во всех службах предупреждений в Azure Monitor для интеграции веб-перехватчика. [Сведения об общих определениях схемы предупреждений.](./alerts-common-schema-definitions.md)


## <a name="authenticate-the-webhook"></a>Аутентификация веб-перехватчика
При необходимости веб-перехватчик может использовать авторизацию на основе токенов для аутентификации. URI веб-перехватчика сохраняется вместе с идентификатором токена, например `https://mysamplealert/webcallback?tokenid=sometokenid&someparameter=somevalue`.

## <a name="payload-schema"></a>Схема полезных данных
Полезные данные JSON, содержащихся в операции POST, могут быть различны в зависимости от поля data.context.activityLog.eventSource.

> [!NOTE]
> В настоящее время описание, которое является частью события журнала действий, копируется в вызванное свойство **"Описание предупреждения"** .
>
> Чтобы выстроить полезные данные журнала действий с другими типами оповещений, начиная с 1 апреля, 2021 в свойстве **"Описание"** запущенного свойства предупреждения будет содержаться описание правила генерации оповещений.
>
> При подготовке к этому изменению мы создали новое свойство **"описание события журнала действий"** для оповещения, которое активировал журнал действий. Это новое свойство будет заполнено свойством **"Description"** , которое уже доступно для использования. Это означает, что новое поле **"описание события журнала действий"** будет содержать описание, которое является частью события журнала действий.
>
> Проверьте правила генерации оповещений, правила действий, веб-перехватчики, приложение логики или другие конфигурации, в которых вы можете использовать свойство **"Description"** из выданного предупреждения, и замените его свойством **"описание события журнала действий"** .
>
> Если ваше условие (в правилах действий, веб-перехватчиков, в приложении логики или в других конфигурациях) основано на свойстве **"Description"** для оповещений журнала действий, то, возможно, потребуется изменить свойство так, чтобы оно основывалось на свойстве **"описание события журнала действий"** .
>
> Чтобы заполнить новое свойство **"Description"** , можно добавить описание в определении правила генерации оповещений.
> ![Запущенные оповещения журнала действий](media/activity-log-alerts-webhook/activity-log-alert-fired.png)

### <a name="common"></a>Распространенные

```json
{
    "schemaId": "Microsoft.Insights/activityLogs",
    "data": {
        "status": "Activated",
        "context": {
            "activityLog": {
                "channels": "Operation",
                "correlationId": "6ac88262-43be-4adf-a11c-bd2179852898",
                "eventSource": "Administrative",
                "eventTimestamp": "2017-03-29T15:43:08.0019532+00:00",
                "eventDataId": "8195a56a-85de-4663-943e-1a2bf401ad94",
                "level": "Informational",
                "operationName": "Microsoft.Insights/actionGroups/write",
                "operationId": "6ac88262-43be-4adf-a11c-bd2179852898",
                "status": "Started",
                "subStatus": "",
                "subscriptionId": "52c65f65-0518-4d37-9719-7dbbfc68c57a",
                "submissionTimestamp": "2017-03-29T15:43:20.3863637+00:00",
                ...
            }
        },
        "properties": {}
    }
}
```

### <a name="administrative"></a>Административный

```json
{
    "schemaId": "Microsoft.Insights/activityLogs",
    "data": {
        "status": "Activated",
        "context": {
            "activityLog": {
                "authorization": {
                    "action": "Microsoft.Insights/actionGroups/write",
                    "scope": "/subscriptions/52c65f65-0518-4d37-9719-7dbbfc68c57b/resourceGroups/CONTOSO-TEST/providers/Microsoft.Insights/actionGroups/IncidentActions"
                },
                "claims": "{...}",
                "caller": "me@contoso.com",
                "description": "",
                "httpRequest": "{...}",
                "resourceId": "/subscriptions/52c65f65-0518-4d37-9719-7dbbfc68c57b/resourceGroups/CONTOSO-TEST/providers/Microsoft.Insights/actionGroups/IncidentActions",
                "resourceGroupName": "CONTOSO-TEST",
                "resourceProviderName": "Microsoft.Insights",
                "resourceType": "Microsoft.Insights/actionGroups"
            }
        },
        "properties": {}
    }
}
```

### <a name="security"></a>Безопасность

```json
{
    "schemaId":"Microsoft.Insights/activityLogs",
    "data":{"status":"Activated",
        "context":{
            "activityLog":{
                "channels":"Operation",
                "correlationId":"2518408115673929999",
                "description":"Failed SSH brute force attack. Failed brute force attacks were detected from the following attackers: [\"IP Address: 01.02.03.04\"].  Attackers were trying to access the host with the following user names: [\"root\"].",
                "eventSource":"Security",
                "eventTimestamp":"2017-06-25T19:00:32.607+00:00",
                "eventDataId":"Sec-07f2-4d74-aaf0-03d2f53d5a33",
                "level":"Informational",
                "operationName":"Microsoft.Security/locations/alerts/activate/action",
                "operationId":"Sec-07f2-4d74-aaf0-03d2f53d5a33",
                "properties":{
                    "attackers":"[\"IP Address: 01.02.03.04\"]",
                    "numberOfFailedAuthenticationAttemptsToHost":"456",
                    "accountsUsedOnFailedSignInToHostAttempts":"[\"root\"]",
                    "wasSSHSessionInitiated":"No","endTimeUTC":"06/25/2017 19:59:39",
                    "actionTaken":"Detected",
                    "resourceType":"Virtual Machine",
                    "severity":"Medium",
                    "compromisedEntity":"LinuxVM1",
                    "remediationSteps":"[In case this is an Azure virtual machine, add the source IP to NSG block list for 24 hours (see https://azure.microsoft.com/documentation/articles/virtual-networks-nsg/)]",
                    "attackedResourceType":"Virtual Machine"
                },
                "resourceId":"/subscriptions/12345-5645-123a-9867-123b45a6789/resourceGroups/contoso/providers/Microsoft.Security/locations/centralus/alerts/Sec-07f2-4d74-aaf0-03d2f53d5a33",
                "resourceGroupName":"contoso",
                "resourceProviderName":"Microsoft.Security",
                "status":"Active",
                "subscriptionId":"12345-5645-123a-9867-123b45a6789",
                "submissionTimestamp":"2017-06-25T20:23:04.9743772+00:00",
                "resourceType":"MICROSOFT.SECURITY/LOCATIONS/ALERTS"
            }
        },
        "properties":{}
    }
}
```

### <a name="recommendation"></a>Рекомендация

```json
{
    "schemaId":"Microsoft.Insights/activityLogs",
    "data":{
        "status":"Activated",
        "context":{
            "activityLog":{
                "channels":"Operation",
                "claims":"{\"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress\":\"Microsoft.Advisor\"}",
                "caller":"Microsoft.Advisor",
                "correlationId":"123b4c54-11bb-3d65-89f1-0678da7891bd",
                "description":"A new recommendation is available.",
                "eventSource":"Recommendation",
                "eventTimestamp":"2017-06-29T13:52:33.2742943+00:00",
                "httpRequest":"{\"clientIpAddress\":\"0.0.0.0\"}",
                "eventDataId":"1bf234ef-e45f-4567-8bba-fb9b0ee1dbcb",
                "level":"Informational",
                "operationName":"Microsoft.Advisor/recommendations/available/action",
                "properties":{
                    "recommendationSchemaVersion":"1.0",
                    "recommendationCategory":"HighAvailability",
                    "recommendationImpact":"Medium",
                    "recommendationName":"Enable Soft Delete to protect your blob data",
                    "recommendationResourceLink":"https://portal.azure.com/#blade/Microsoft_Azure_Expert/RecommendationListBlade/recommendationTypeId/12dbf883-5e4b-4f56-7da8-123b45c4b6e6",
                    "recommendationType":"12dbf883-5e4b-4f56-7da8-123b45c4b6e6"
                },
                "resourceId":"/subscriptions/12345-5645-123a-9867-123b45a6789/resourceGroups/contoso/providers/microsoft.storage/storageaccounts/contosoStore",
                "resourceGroupName":"CONTOSO",
                "resourceProviderName":"MICROSOFT.STORAGE",
                "status":"Active",
                "subStatus":"",
                "subscriptionId":"12345-5645-123a-9867-123b45a6789",
                "submissionTimestamp":"2017-06-29T13:52:33.2742943+00:00",
                "resourceType":"MICROSOFT.STORAGE/STORAGEACCOUNTS"
            }
        },
        "properties":{}
    }
}
```

### <a name="servicehealth"></a>ServiceHealth

```json
{
    "schemaId": "Microsoft.Insights/activityLogs",
    "data": {
        "status": "Activated",
        "context": {
            "activityLog": {
            "channels": "Admin",
            "correlationId": "bbac944f-ddc0-4b4c-aa85-cc7dc5d5c1a6",
            "description": "Active: Virtual Machines - Australia East",
            "eventSource": "ServiceHealth",
            "eventTimestamp": "2017-10-18T23:49:25.3736084+00:00",
            "eventDataId": "6fa98c0f-334a-b066-1934-1a4b3d929856",
            "level": "Informational",
            "operationName": "Microsoft.ServiceHealth/incident/action",
            "operationId": "bbac944f-ddc0-4b4c-aa85-cc7dc5d5c1a6",
            "properties": {
                "title": "Virtual Machines - Australia East",
                "service": "Virtual Machines",
                "region": "Australia East",
                "communication": "Starting at 02:48 UTC on 18 Oct 2017 you have been identified as a customer using Virtual Machines in Australia East who may receive errors starting Dv2 Promo and DSv2 Promo Virtual Machines which are in a stopped &quot;deallocated&quot; or suspended state. Customers can still provision Dv1 and Dv2 series Virtual Machines or try deploying Virtual Machines in other regions, as a possible workaround. Engineers have identified a possible fix for the underlying cause, and are exploring implementation options. The next update will be provided as events warrant.",
                "incidentType": "Incident",
                "trackingId": "0NIH-U2O",
                "impactStartTime": "2017-10-18T02:48:00.0000000Z",
                "impactedServices": "[{\"ImpactedRegions\":[{\"RegionName\":\"Australia East\"}],\"ServiceName\":\"Virtual Machines\"}]",
                "defaultLanguageTitle": "Virtual Machines - Australia East",
                "defaultLanguageContent": "Starting at 02:48 UTC on 18 Oct 2017 you have been identified as a customer using Virtual Machines in Australia East who may receive errors starting Dv2 Promo and DSv2 Promo Virtual Machines which are in a stopped &quot;deallocated&quot; or suspended state. Customers can still provision Dv1 and Dv2 series Virtual Machines or try deploying Virtual Machines in other regions, as a possible workaround. Engineers have identified a possible fix for the underlying cause, and are exploring implementation options. The next update will be provided as events warrant.",
                "stage": "Active",
                "communicationId": "636439673646212912",
                "version": "0.1.1"
            },
            "status": "Active",
            "subscriptionId": "45529734-0ed9-4895-a0df-44b59a5a07f9",
            "submissionTimestamp": "2017-10-18T23:49:28.7864349+00:00"
        }
    },
    "properties": {}
    }
}
```

Сведения о конкретной схеме оповещений журнала действий для уведомлений о работоспособности службы см. в статье [Уведомления о работоспособности службы](../../service-health/service-notifications.md). Также узнайте, как [настроить уведомления веб-перехватчика о работоспособности службы с помощью существующих решений по управлению проблемами](../../service-health/service-health-alert-webhook-guide.md).

### <a name="resourcehealth"></a>ResourceHealth

```json
{
    "schemaId": "Microsoft.Insights/activityLogs",
    "data": {
        "status": "Activated",
        "context": {
            "activityLog": {
                "channels": "Admin, Operation",
                "correlationId": "a1be61fd-37ur-ba05-b827-cb874708babf",
                "eventSource": "ResourceHealth",
                "eventTimestamp": "2018-09-04T23:09:03.343+00:00",
                "eventDataId": "2b37e2d0-7bda-4de7-ur8c6-1447d02265b2",
                "level": "Informational",
                "operationName": "Microsoft.Resourcehealth/healthevent/Activated/action",
                "operationId": "2b37e2d0-7bda-489f-81c6-1447d02265b2",
                "properties": {
                    "title": "Virtual Machine health status changed to unavailable",
                    "details": "Virtual machine has experienced an unexpected event",
                    "currentHealthStatus": "Unavailable",
                    "previousHealthStatus": "Available",
                    "type": "Downtime",
                    "cause": "PlatformInitiated"
                },
                "resourceId": "/subscriptions/<subscription Id>/resourceGroups/<resource group>/providers/Microsoft.Compute/virtualMachines/<resource name>",
                "resourceGroupName": "<resource group>",
                "resourceProviderName": "Microsoft.Resourcehealth/healthevent/action",
                "status": "Active",
                "subscriptionId": "<subscription Id>",
                "submissionTimestamp": "2018-09-04T23:11:06.1607287+00:00",
                "resourceType": "Microsoft.Compute/virtualMachines"
            }
        }
    }
}
```

| Имя элемента | Описание |
| --- | --- |
| status |Используется для оповещений на основе метрик. Всегда имеет значение activated для оповещений журнала действий. |
| контекст |Контекст события. |
| resourceProviderName |Поставщик ресурсов для затронутого ресурса. |
| conditionType |Всегда имеет значение Event. |
| name |Имя правила генерации оповещений. |
| идентификатор |Идентификатор ресурса для оповещения. |
| description; |Описание оповещения, которое задается при его создании. |
| subscriptionId |Идентификатор подписки Azure. |
| TIMESTAMP |Время создания события службой Azure, которая обработала запрос. |
| resourceId |Идентификатор ресурса для затронутого ресурса. |
| имя_группы_ресурсов |Имя группы ресурсов для затронутого ресурса. |
| properties |Набор пар `<Key, Value>` (например, `Dictionary<String, String>`), содержащий сведения о событии. |
| event |Элемент, содержащий метаданные о событии. |
| авторизация |Свойства управления доступом на основе ролей Azure для события. Обычно к ним относятся action, role и scope. |
| категория |Категория события. Поддерживаются следующие значения: Administrative, Alert, Security, ServiceHealth, Recommendation. |
| caller |Адрес электронной почты пользователя, который выполнил операцию, утверждение имени субъекта-службы или имени участника-пользователя в зависимости от доступности. Может иметь значение NULL для определенных системных вызовов. |
| correlationId |Обычно GUID в строковом формате. События с correlationId относятся к одному крупному действию и обычно совместно используют correlationId. |
| eventDescription |Статическое описание события в текстовом виде. |
| eventDataId |Уникальный идентификатор события. |
| eventSource |Имя инфраструктуры или службы Azure, которая создала событие. |
| httpRequest |Обычно запрос содержит clientRequestId, clientIpAddress и method (метод HTTP, например PUT). |
| уровень |Одно из таких значений: Critical, Error, Warning, Informational. |
| operationId |Обычно события, относящиеся к одной операции, совместно используют один GUID. |
| operationName |Имя операции. |
| properties |Свойства события. |
| status |Строка. Состояние операции. Обычные значения: Started, In Progress, Succeeded, Failed, Active, Resolved. |
| subStatus |Обычно содержит код состояния HTTP для соответствующего вызова REST. Может также включать другие строки, описывающие подсостояние. Обычные значения подсостояния: OK (код состояния HTTP: 200), Created (код состояния HTTP: 201), Accepted (код состояния HTTP: 202), No Content (код состояния HTTP: 204), Bad Request (код состояния HTTP: 400), Not Found (код состояния HTTP: 404), Conflict (код состояния HTTP: 409), Internal Server Error (код состояния HTTP: 500), Service Unavailable (код состояния HTTP: 503), Gateway Timeout (код состояния HTTP: 504). |

Сведения о схеме для остальных оповещений журнала действий см. в статье [Мониторинг действий подписки с помощью журнала действий Azure](../essentials/platform-logs-overview.md).

## <a name="next-steps"></a>Дальнейшие действия
* Дополнительные [сведения о журнале действий](../essentials/platform-logs-overview.md).
* [Выполнение скриптов службы автоматизации Azure (модулей Runbook) в оповещениях Azure](https://go.microsoft.com/fwlink/?LinkId=627081).
* [Logic app that sends a text message when an alert fires](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-text-message-with-logic-app) (Приложение логики, которое отправляет текстовое сообщение при возникновении предупреждения). Это пример для оповещений на основе метрик, но его можно изменить для работы с оповещениями журнала действий.
* [Logic app that posts a message to a slack channel when an alert fires](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-slack-with-logic-app) (Приложение логики, которое отправляет сообщение в канал Slack при возникновении предупреждения). Это пример для оповещений на основе метрик, но его можно изменить для работы с оповещениями журнала действий.
* [Logic app that adds an item to a queue when an alert fires](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-queue-with-logic-app) (Приложение логики, добавляющее элемент в очередь при возникновении предупреждения). Это пример для оповещений на основе метрик, но его можно изменить для работы с оповещениями журнала действий.