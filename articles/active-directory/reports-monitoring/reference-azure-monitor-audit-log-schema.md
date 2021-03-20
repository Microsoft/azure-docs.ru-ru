---
title: Интерпретировать Azure Active Directory схему журнала аудита в Azure Monitor | Документация Майкрософт
description: Описание схемы журнала аудита Azure AD для использования в Azure Monitor
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 4b18127b-d1d0-4bdc-8f9c-6a4c991c5f75
ms.service: active-directory
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/18/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: b4d32e0b939731d64f3ef2495ab606b5dc0780e4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "100592291"
---
# <a name="interpret-the-azure-ad-audit-logs-schema-in-azure-monitor-preview"></a>Интерпретация схемы журналов аудита Azure AD в Azure Monitor (предварительная версия)

В этой статье описывается схема журналов аудита Azure Active Directory (Azure AD) в Azure Monitor. Каждая отдельная запись журнала хранится как текст в формате большого двоичного объекта JSON, как показано в двух примерах ниже. 

```json
{ 
    "records": [ 
    { 
        "time": "2018-03-17T00:14:31.2585575Z", 
        "operationName": "Change password (self-service)",
        "operationVersion": "1.0",
        "category": "Audit", 
        "tenantId": "bf85dc9d-cb43-44a4-80c4-469e8c58249e", 
        "resultType": "Success", 
        "resultSignature": "-1", 
        "resultDescription": "None", 
        "durationMs": "-1", 
        "correlationId": "60d5e89a-b890-413f-9e25-a047734afe9f", 
        "identity": "sreens@wingtiptoysonline.com", 
        "Level": "Informational", 
        "location": "WUS", 
        "properties": { 
            "identityType": "UPN", 
            "operationType": "Update", 
            "additionalDetails": "None", 
            "additionalTargets": "", 
            "targetUpdatedProperties": "", 
            "targetResourceType": "UPN__TenantContextID__PUID__ObjectID__ObjectClass", 
            "targetResourceName": "sreens@wingtiptoysonline.com__bf85dc9d-cb43-44a4-80c4-469e8c58249e__1003BFFD9FEB17DB__7a408bdd-7d97-4574-8511-dd747b56465d__User", 
            "auditEventCategory": "UserManagement" 
        } 
    } 
    ] 
} 
```

```json
{ 
    "records": [ 
    { 
        "time": "2018-03-18T19:47:43.0368859Z", 
        "operationName": "Update service principal.", 
        "operationVersion": "1.0", 
        "category": "Audit", 
        "tenantId": "bf85dc9d-cb43-44a4-80c4-469e8c58249e", 
        "resultType": "Success", 
        "resultSignature": "-1", 
        "durationMs": "-1", 
        "callerIpAddress": "<null>", 
        "correlationId": "14916c7a-5a7d-44e8-9b06-74b49efb08ee", 
        "identity": "NA", 
        "Level": "Informational", 
        "properties": { 
            "identityType": "NA", 
            "operationType": "Update", 
            "additionalDetails": {}, 
            "additionalTargets": "", 
            "targetUpdatedProperties": [ 
            { 
                "Name": "Included Updated Properties", 
                "OldValue": null, 
                "NewValue": "" 
            }, 
            { 
                "Name": "TargetId.ServicePrincipalNames", 
                "OldValue": null, 
                "NewValue": "http://adapplicationregistry.onmicrosoft.com/salesforce.com/primary;cd3ed3de-93ee-400b-8b19-b61ef44a0f29" 
            } 
            ], 
        "targetResourceType": "Other__ObjectID__ObjectClass__Name__AppId__SPN", 
        "targetResourceName": "ServicePrincipal_ea70a262-4da3-440a-b396-9734ddfd9df2__ea70a262-4da3-440a-b396-9734ddfd9df2__ServicePrincipal__Salesforce__cd3ed3de-93ee-400b-8b19-b61ef44a0f29__http://adapplicationregistry.onmicrosoft.com/salesforce.com/primary;cd3ed3de-93ee-400b-8b19-b61ef44a0f29", 
        "auditEventCategory": "ApplicationManagement" 
      } 
    } 
    ] 
} 
```

```json
{
    "records": [
    {
        "time": "2018-12-10T00:03:46.6161822Z",
        "resourceId": "/tenants/7918d4b5-0442-4a97-be2d-36f9f9962ece/providers/Microsoft.aadiam",
        "operationName": "Update policy",
        "operationVersion": "1.0",
        "category": "AuditLogs",
        "tenantId": "7918d4b5-0442-4a97-be2d-36f9f9962ece",
        "resultSignature": "None",
        "durationMs": 0,
        "callerIpAddress": "<null>",
        "correlationId": "192298c1-0994-4dd6-b05a-a6c5984c31cb",
        "identity": "MS-PIM",
        "level": "Informational",
        "properties": {
            "id": "Directory_VNXV4_28148892",
            "category": "Policy",
            "correlationId": "192298c1-0994-4dd6-b05a-a6c5984c31cb",
            "result": 0,
            "resultReason": "",
            "activityDisplayName": "Update policy",
            "activityDateTime": "2018-12-10T00:03:46.6161822+00:00",
            "loggedByService": "Core Directory",
            "operationType": "Update",
            "initiatedBy": {},
            "targetResources": [
            {
                "id": "5e7a8ae7-165d-44a4-a4f4-6141f8c8ef40",
                "displayName": "Default Policy",
                "type": "Policy",
                "modifiedProperties": []
            }
            ],
            "additionalDetails": []
        }
    }
    ]
}

```

## <a name="field-and-property-descriptions"></a>Описания свойств и полей

| Имя поля | Описание |
|------------|-------------|
| time       | Дата и время (в формате UTC). |
| operationName | Имя операции. |
| operationVersion | Запрошенная клиентом версия REST API. |
| категория | В настоящее время поддерживается только значение *Audit*. |
| tenantId | Связанный с журналами уникальный идентификатор (GUID) клиента. |
| resultType | Результат операции. Значение результата может быть *Success* или *Failure*. |
| resultSignature |  Это поле не сопоставлено, и его можно игнорировать. | 
| resultDescription | Дополнительное описание результата, если оно доступно. | 
| durationMs |  Это поле не сопоставлено, и его можно игнорировать. |
| callerIpAddress | IP-адрес отправившего запрос клиента. | 
| correlationId | Необязательный код GUID, который передал клиент. Это значение может помочь найти связь между операциями на стороне клиента и операциями на стороне сервера. Кроме того, оно может пригодиться при отслеживании журналов, в которые записываются данные о работе служб. |
| удостоверение | Удостоверение из маркера, предоставленное при выполнении запроса. Это может быть учетная запись пользователя, учетная запись системы или субъект-служба. |
| уровень | Тип сообщения. Для журналов аудита уровень всегда имеет значение *Informational*. |
| location | Расположение центра обработки данных. |
| properties | Список поддерживаемых свойств, относящихся к журналу аудита. Этот процесс описан в следующей таблице. | 

<br>

| Имя свойства | Описание |
|---------------|-------------|
| AuditEventCategory | Тип события аудита. Может иметь значение *User Management*, *Application Management* или другого типа.|
| Identity Type | Это может быть *Application* или *User*. |
| Тип операции | Может иметь значение *Add*, *Update* или *Delete* или *Other*. |
| Target Resource Type | Задает тип целевого ресурса, с которым выполнялась операция. Может иметь значение *Application*, *User*, *Role*, *Policy*. | 
| Target Resource Name | Имя целевого ресурса. Это может быть имя приложения, имя роли, имя участника-пользователя или имя субъекта-службы. |
| additionalTargets | Содержит список дополнительных свойств для конкретных операций. Например, для операции обновления старые значения и новые значения перечислены в разделе *таржетупдатедпропертиес*. | 

## <a name="next-steps"></a>Дальнейшие действия

* [Interpret the Azure AD sign-in logs schema in Azure Monitor (preview)](reference-azure-monitor-sign-ins-log-schema.md) (Интерпретация схемы журналов входа Azure Active Directory в Azure Monitor (предварительная версия))
* [Журналы диагностики Azure](../../azure-monitor/essentials/platform-logs-overview.md)
* [Часто задаваемые вопросы и известные проблемы](concept-activity-logs-azure-monitor.md#frequently-asked-questions)