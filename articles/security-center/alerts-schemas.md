---
title: Схемы для оповещений центра безопасности Azure
description: В этой статье описываются различные схемы, используемые центром безопасности Azure для оповещений системы безопасности.
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/19/2020
ms.author: memildin
ms.openlocfilehash: 55f8d37d435aa8adeb4d97246ce7b2c7811140be
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102558004"
---
# <a name="security-alerts-schemas"></a>Схемы оповещений системы безопасности

Если в вашей подписке включен Защитник Azure, вы получите оповещения системы безопасности, когда центр безопасности обнаружит угрозы для их ресурсов.

Эти оповещения безопасности можно просмотреть на страницах **защиты от угроз** в центре безопасности Azure или с помощью внешних средств, таких как:

- [Sentinel Azure](../sentinel/index.yml) — собственная облачная SIEM Майкрософт. Соединитель Sentinel получает оповещения из центра безопасности Azure и отправляет их в [рабочую область log Analytics](../azure-monitor/logs/quick-create-workspace.md) для Azure Sentinel.
- Сторонние решения Siem — отправляйте данные в [концентраторы событий Azure](../event-hubs/index.yml). Затем интегрируйте данные концентратора событий со сторонними SIEM. Подробные сведения см. в статье [Потоковая передача оповещений в решение SIEM, SOAR или ITSM](export-to-siem.md).
- [REST API](/rest/api/securitycenter/) . Если вы используете REST API для доступа к оповещениям, см. [документацию по API интерактивных оповещений](/rest/api/securitycenter/alerts).

Если вы используете любые программные методы для использования оповещений, вам потребуется правильная схема, чтобы найти поля, имеющие отношение к вам. Кроме того, если вы экспортируете в концентратор событий или пытаетесь активировать автоматизацию рабочих процессов с универсальными соединителями HTTP, используйте схемы для правильного анализа объектов JSON.

>[!IMPORTANT]
> Эта схема немного отличается для каждого из этих сценариев, поэтому убедитесь, что вы выбрали соответствующую вкладку ниже.


## <a name="the-schemas"></a>Схемы 


### <a name="workflow-automation-and-continuous-export-to-event-hub"></a>[Автоматизация рабочих процессов и непрерывный экспорт в концентратор событий](#tab/schema-continuousexport)

### <a name="sample-json-for-alerts-sent-to-logic-apps-event-hub-and-third-party-siems"></a>Пример JSON для оповещений, отправленных в Logic Apps, концентратор событий и сторонние решения Siem

Ниже вы найдете схему событий предупреждений, передаваемых в:

- Экземпляры приложения логики Azure, которые были настроены в автоматизации рабочих процессов центра безопасности
- Концентратор событий Azure, использующий функцию непрерывного экспорта в центре безопасности

Дополнительные сведения о функции автоматизации рабочих процессов см. в разделе [Автоматизация ответов на триггеры центра безопасности](workflow-automation.md).

Дополнительные сведения о непрерывном экспорте см. в разделе [непрерывный экспорт данных центра безопасности](continuous-export.md).

[!INCLUDE [Workflow schema](../../includes/security-center-alerts-schema-workflow-automation.md)]




### <a name="azure-sentinel-and-log-analytics-workspaces"></a>[Sentinel и Log Analytics рабочих областей Azure](#tab/schema-sentinel)

Соединитель Sentinel получает оповещения из центра безопасности Azure и отправляет их в рабочую область Log Analytics для Azure Sentinel. 

Чтобы создать случай или инцидент с помощью оповещений центра безопасности, вам потребуется схема для этих оповещений, показанных ниже. 

Дополнительные сведения об Azure Sentinel см. [в документации](../sentinel/index.yml).

[!INCLUDE [Sentinel and workspace schema](../../includes/security-center-alerts-schema-log-analytics-workspace.md)]




### <a name="azure-activity-log"></a>[Журнал действий Azure](#tab/schema-activitylog)

Центр безопасности Azure выполняет аудит созданных оповещений системы безопасности в качестве событий в журнале действий Azure.

События оповещения системы безопасности в журнале действий можно просмотреть, выполнив поиск события активировать предупреждение, как показано ниже.

[![Поиск события активации предупреждения в журнале действий](media/alerts-schemas/sample-activity-log-alert.png)](media/alerts-schemas/sample-activity-log-alert.png#lightbox)


### <a name="sample-json-for-alerts-sent-to-azure-activity-log"></a>Пример JSON для оповещений, отправляемых в журнал действий Azure

```json
{
    "channels": "Operation",
    "correlationId": "2518250008431989649_e7313e05-edf4-466d-adfd-35974921aeff",
    "description": "PREVIEW - Role binding to the cluster-admin role detected. Kubernetes audit log analysis detected a new binding to the cluster-admin role which gives administrator privileges.\r\nUnnecessary administrator privileges might cause privilege escalation in the cluster.",
    "eventDataId": "2518250008431989649_e7313e05-edf4-466d-adfd-35974921aeff",
    "eventName": {
        "value": "PREVIEW - Role binding to the cluster-admin role detected",
        "localizedValue": "PREVIEW - Role binding to the cluster-admin role detected"
    },
    "category": {
        "value": "Security",
        "localizedValue": "Security"
    },
    "eventTimestamp": "2019-12-25T18:52:36.801035Z",
    "id": "/subscriptions/SUBSCRIPTION_ID/resourceGroups/RESOURCE_GROUP_NAME/providers/Microsoft.Security/locations/centralus/alerts/2518250008431989649_e7313e05-edf4-466d-adfd-35974921aeff/events/2518250008431989649_e7313e05-edf4-466d-adfd-35974921aeff/ticks/637128967568010350",
    "level": "Informational",
    "operationId": "2518250008431989649_e7313e05-edf4-466d-adfd-35974921aeff",
    "operationName": {
        "value": "Microsoft.Security/locations/alerts/activate/action",
        "localizedValue": "Activate Alert"
    },
    "resourceGroupName": "RESOURCE_GROUP_NAME",
    "resourceProviderName": {
        "value": "Microsoft.Security",
        "localizedValue": "Microsoft.Security"
    },
    "resourceType": {
        "value": "Microsoft.Security/locations/alerts",
        "localizedValue": "Microsoft.Security/locations/alerts"
    },
    "resourceId": "/subscriptions/SUBSCRIPTION_ID/resourceGroups/RESOURCE_GROUP_NAME/providers/Microsoft.Security/locations/centralus/alerts/2518250008431989649_e7313e05-edf4-466d-adfd-35974921aeff",
    "status": {
        "value": "Active",
        "localizedValue": "Active"
    },
    "subStatus": {
        "value": "",
        "localizedValue": ""
    },
    "submissionTimestamp": "2019-12-25T19:14:03.5507487Z",
    "subscriptionId": "SUBSCRIPTION_ID",
    "properties": {
        "clusterRoleBindingName": "cluster-admin-binding",
        "subjectName": "for-binding-test",
        "subjectKind": "ServiceAccount",
        "username": "masterclient",
        "actionTaken": "Detected",
        "resourceType": "Kubernetes Service",
        "severity": "Low",
        "intent": "[\"Persistence\"]",
        "compromisedEntity": "ASC-IGNITE-DEMO",
        "remediationSteps": "[\"Review the user in the alert details. If cluster-admin is unnecessary for this user, consider granting lower privileges to the user.\"]",
        "attackedResourceType": "Kubernetes Service"
    },
    "relatedEvents": []
}
```

### <a name="the-data-model-of-the-schema"></a>Модель данных схемы

|Поле|Описание|
|----|----|
|**канала**|Constant, "Operation"|
|**correlationId**|Идентификатор оповещения центра безопасности Azure|
|**description**|Описание оповещения.|
|**eventDataId**|См. correlationId|
|**eventName**|Подполя Value и localizedValue содержат отображаемое имя предупреждения.|
|**category**|Подполя Value и localizedValue являются константой "Security"|
|**евенттиместамп**|Метка времени в формате UTC для момента создания предупреждения|
|**id**|Полный идентификатор предупреждения|
|**level**|Constant, "информационный"|
|**operationId**|См. correlationId|
|**operationName**|Поле значения имеет значение Constant-"Microsoft. Security/Locations/Alerts/Activate/Action", а локализованное значение будет "активировать оповещение" (может быть локализовано как Национальная настройка пользователя)|
|**resourceGroupName**|Будет включать имя группы ресурсов.|
|**resourceProviderName**|Подполя Value и localizedValue являются константой "Microsoft. Security".|
|**resourceType**|Подполя Value и localizedValue являются константой "Microsoft. Security/Locations/Alerts"|
|**resourceId**|Полный идентификатор ресурса Azure|
|**status**|Подполя Value и localizedValue являются константой "Active"|
|**Подсостояние**|Подполя Value и localizedValue пусты|
|**submissionTimestamp**|Метка времени в формате UTC для отправки события в журнал действий|
|**subscriptionId**|Идентификатор подписки скомпрометированного ресурса|
|**properties**|Контейнер JSON дополнительных свойств, относящихся к предупреждению. Они могут переключиться с одного предупреждения на другое, однако следующие поля будут отображаться во всех оповещениях:<br>-Severity — серьезность атаки.<br>-Компромиседентити: имя скомпрометированного ресурса.<br>-Ремедиатионстепс: массив выполняемых действий по исправлению<br>-намерение: цель принудительной цепочки для предупреждения. Возможные варианты задокументированы в таблице « [намерения](alerts-reference.md#intentions) »|
|**relatedEvents**|Константа-пустой массив|
|||





### <a name="ms-graph-api"></a>[MS API Graph](#tab/schema-graphapi)

Microsoft Graph — это шлюз для данных и аналитики в Microsoft 365. Она предоставляет унифицированную модель программирования, которую можно использовать для доступа к огромному объему данных в Microsoft 365, Windows 10 и Enterprise Mobility + Security. Используйте множество данных в Microsoft Graph для создания приложений для организаций и потребителей, взаимодействующих с миллионами пользователей.

Схема и представление JSON для оповещений системы безопасности, отправляемых в MS Graph, доступны в [документации по Microsoft Graph](/graph/api/resources/alert).

---


## <a name="next-steps"></a>Дальнейшие действия

В этой статье описаны схемы, используемые средствами защиты от угроз центра безопасности Azure при отправке сведений об оповещениях системы безопасности.

Дополнительные сведения о способах доступа к оповещениям системы безопасности извне центра безопасности см. на следующих страницах:

- [Sentinel Azure](../sentinel/index.yml) — это облачная служба Microsoft Native SIEM
- [Концентраторы событий Azure](../event-hubs/index.yml) — полностью управляемая служба приема данных в режиме реального времени (Майкрософт)
- [Непрерывный экспорт данных Центра безопасности](continuous-export.md)
- [Log Analytics рабочие области](../azure-monitor/logs/quick-create-workspace.md) — Azure Monitor хранят данные журнала в log Analytics рабочей области, контейнере, который содержит данные и сведения о конфигурации.