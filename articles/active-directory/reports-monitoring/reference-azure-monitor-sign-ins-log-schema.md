---
title: Схема журнала входа в Azure Monitor | Документация Майкрософт
description: Опишите схему журнала входа Azure AD для использования в Azure Monitor
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
ms.date: 03/12/2021
ms.author: markvi
ms.reviewer: besiler
ms.collection: M365-identity-device-management
ms.openlocfilehash: bad8ae86827144269e816a6c2e01d6af3f4d88ac
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103225424"
---
# <a name="interpret-the-azure-ad-sign-in-logs-schema-in-azure-monitor"></a>Анализ схемы журналов входа в Azure AD в Azure Monitor

В этой статье описывается схема журналов входа Azure Active Directory (Azure AD) в Azure Monitor. Большая часть информации об операциях входа предоставляется в атрибуте *Properties* объекта `records`.


```json
{ 
    "time": "2019-03-12T16:02:15.5522137Z", 
    "resourceId": "/tenants/<TENANT ID>/providers/Microsoft.aadiam",
    "operationName": "Sign-in activity", 
    "operationVersion": "1.0", 
    "category": "SignInLogs", 
    "tenantId": "<TENANT ID>", 
    "resultType": "50140", 
    "resultSignature": "None", 
    "resultDescription": "This error occurred due to 'Keep me signed in' interrupt when the user was signing-in.", 
    "durationMs": 0, 
    "callerIpAddress": "<CALLER IP ADDRESS>", 
    "correlationId": "a75a10bd-c126-486b-9742-c03110d36262", 
    "identity": "Timothy Perkins", 
    "Level": 4, 
    "location": "US", 
    "properties": 
        {
            "id":"0231f922-93fa-4005-bb11-b344eca03c01",
            "createdDateTime":"2019-03-12T16:02:15.5522137+00:00",
            "userDisplayName":"Timothy Perkins",
            "userPrincipalName":"<USER PRINCIPAL NAME>",
            "userId":"<USER ID>",
            "appId":"<APPLICATION ID>",
            "appDisplayName":"Azure Portal",
            "ipAddress":"<IP ADDRESS>",
            "status":
            {
                "errorCode":50140,
                "failureReason":"This error occurred due to 'Keep me signed in' interrupt when the user was signing-in."
            },
            "clientAppUsed":"Browser",
            "deviceDetail":
            {
                "operatingSystem":"Windows 10",
                "browser":"Chrome 72.0.3626"
            },
            "location":
                {
                    "city":"Bellevue",
                    "state":"Washington",
                    "countryOrRegion":"US",
                    "geoCoordinates":
                    {
                        "latitude":45,
                        "longitude":122
                    }
                },
            "correlationId":"a75a10bd-c126-486b-9742-c03110d36262",
            "conditionalAccessStatus":"notApplied",
            "appliedConditionalAccessPolicies":
            [
                {
                    "id":"ae11ffaa-9879-44e0-972c-7538fd5c4d1a",
                    "displayName":"Hr app access policy",
                    "enforcedGrantControls":
                    [
                        "Mfa"
                    ],
                    "enforcedSessionControls":
                    [
                    ],
                    "result":"notApplied"
                },
                {
                    "id":"b915a70b-2eee-47b6-85b6-ff4f4a66256d",
                    "displayName":"MFA for all but global support access",
                    "enforcedGrantControls":[],
                    "enforcedSessionControls":[],
                    "result":"notEnabled"
                },
                {
                    "id":"830f27fa-67a8-461f-8791-635b7225caf1",
                    "displayName":"Header Based Application Control",
                    "enforcedGrantControls":["Mfa"],
                    "enforcedSessionControls":[],
                    "result":"notApplied"
                },
                {
                    "id":"8ed8d7f7-0a2e-437b-b512-9e47bed562e6",
                    "displayName":"MFA for everyones",
                    "enforcedGrantControls":[],
                    "enforcedSessionControls":[],
                    "result":"notEnabled"
                },
                {
                    "id":"52924e0f-798b-4afd-8c42-49055c7d6395",
                    "displayName":"Device compliant",
                    "enforcedGrantControls":[],
                    "enforcedSessionControls":[],
                    "result":"notEnabled"
                },
             ],
            "isInteractive":true,
            "tokenIssuerType":"AzureAD",
            "authenticationProcessingDetails":[],
            "networkLocationDetails":[],
            "processingTimeInMilliseconds":0,
            "riskDetail":"hidden",
            "riskLevelAggregated":"hidden",
            "riskLevelDuringSignIn":"hidden",
            "riskState":"none",
            "riskEventTypes":[],
            "resourceDisplayName":"windows azure service management api",
            "resourceId":"797f4846-ba00-4fd7-ba43-dac1f8f63013",
            "authenticationMethodsUsed":[]
        }
}
```


## <a name="field-descriptions"></a>Описания полей

| Имя поля | Ключ | Описание |
| --- | --- | --- | 
| Time |  - | Дата и время в формате UTC. |
| ResourceId | - | Это значение не сопоставлено, и его можно игнорировать.  |
| OperationName | - | Для операций входа это поле всегда имеет значение *Sign-in activity*. |
| OperationVersion | - | Запрошенная клиентом версия REST API. |
| Category | - | Для операций входа это поле всегда имеет значение *SignIn*. | 
| TenantId | - | Связанный с журналами уникальный идентификатор (GUID) клиента. |
| ResultType | - | Результатом операции входа может быть *Success* или *Failure*. | 
| ResultSignature | - | Содержит код ошибки, которая возникла во время входа. |
| ResultDescription | Н/д или пусто | Содержит описание ошибки, которая возникла во время входа. |
| рискдетаил | рискдетаил | Предоставляет "причину" для определенного состояния рискованного пользователя, входа или обнаружения риска. Возможные значения: `none` , `adminGeneratedTemporaryPassword` , `userPerformedSecuredPasswordChange` , `userPerformedSecuredPasswordReset` , `adminConfirmedSigninSafe` , `aiConfirmedSigninSafe` , `userPassedMFADrivenByRiskBasedPolicy` , `adminDismissedAllRiskForUser` , `adminConfirmedSigninCompromised` , `unknownFutureValue` . Это значение `none` означает, что для пользователя или входа не было выполнено никаких действий. <br>**Примечание.** Для получения сведений об этом свойстве требуется лицензия Azure AD Premium P2. Другие лицензии возвращают значение `hidden` . |
| рискевенттипес | рискевенттипес | Типы обнаружения рисков, связанные с входом. Возможные значения: `unlikelyTravel` , `anonymizedIPAddress` , `maliciousIPAddress` , `unfamiliarFeatures` , `malwareInfectedIPAddress` , `suspiciousIPAddress` , `leakedCredentials` , `investigationsThreatIntelligence` ,  `generic` и `unknownFutureValue` . |
| ауспроцессингдетаилс | Библиотека проверки подлинности приложений Azure AD | Содержит сведения о семействе, библиотеке и платформе в формате: "Family: Библиотека ADAL: ADAL.JS 1.0.0 Platform: JS" |
| ауспроцессингдетаилс | искаетокен | Значения true или false |
| рисклевелаггрегатед | рисклевел | Агрегированный уровень риска. Возможные значения: `none` , `low` , `medium` , `high` , `hidden` и `unknownFutureValue` . Значение `hidden` означает, что пользователь или вход не был включен для Защита идентификации Azure AD. **Примечание.** Сведения для этого свойства доступны только для клиентов Azure AD Premium P2. Будут возвращены все остальные клиенты `hidden` . |
| рисклевелдурингсигнин | рисклевел | Уровень риска во время входа. Возможные значения: `none` , `low` , `medium` , `high` , `hidden` и `unknownFutureValue` . Значение `hidden` означает, что пользователь или вход не был включен для Защита идентификации Azure AD. **Примечание.** Сведения для этого свойства доступны только для клиентов Azure AD Premium P2. Будут возвращены все остальные клиенты `hidden` . |
| рискстате | рискстате | Сообщает состояние рискованного пользователя, входа в систему или обнаружения риска. Возможные значения: `none` , `confirmedSafe` , `remediated` , `dismissed` , `atRisk` , `confirmedCompromised` , `unknownFutureValue` . |
| DurationMs | - | Это значение не сопоставлено, и его можно игнорировать. |
| CallerIpAddress | - | IP-адрес отправившего запрос клиента. | 
| CorrelationId | - | Необязательный код GUID, который передал клиент. Это значение может помочь найти связь между операциями на стороне клиента и операциями на стороне сервера. Кроме того, оно может пригодиться при отслеживании журналов, в которые записываются данные о многих службах. |
| Идентификация | - | Удостоверение из маркера, предоставленное при выполнении запроса. Это может быть учетная запись пользователя, системная учетная запись или субъект-служба. |
| Level | - | Предоставляет тип сообщения. Для операций аудита это поле всегда имеет значение *Informational*. |
| Расположение | - | Содержит сведения о местонахождении, в котором выполнялась операция входа. |
| Свойства | - | Содержит все свойства, связанные с операциями входа.|

## <a name="next-steps"></a>Следующие шаги

* [Interpret the Azure AD audit logs schema in Azure Monitor (preview)](reference-azure-monitor-audit-log-schema.md) (Интерпретация схемы журналов аудита Azure Active Directory в Azure Monitor (предварительная версия))
* [Подробнее о журналах платформы Azure](../../azure-monitor/essentials/platform-logs-overview.md)