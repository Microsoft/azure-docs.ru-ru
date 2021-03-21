---
title: Общие сведения о схеме Azure AD и пользовательских выражениях
description: В этой статье описывается схема Azure AD, атрибуты, появляющиеся в потоке агента подготовки, а также пользовательские выражения.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/18/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 750bc2f4f535238dbb757afc8b615fa71063d74d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98614365"
---
# <a name="understand-the-azure-ad-schema"></a>Общие сведения о схеме Azure AD
Объект в Azure Active Directory (Azure AD), как и любой каталог, представляет собой программную конструкцию высокого уровня данных, представляющую такие вещи, как пользователи, группы и контакты. При создании нового пользователя или контакта в Azure AD создается новый экземпляр этого объекта. Эти экземпляры можно отличить в зависимости от их свойств.

Свойства в Azure AD — это элементы, отвечающие за хранение сведений об экземпляре объекта в Azure AD.

Схема Azure AD определяет правила, для которых свойства могут использоваться в записи, виды значений, которые могут иметь эти свойства, и способ взаимодействия пользователей с этими значениями. 

В Azure AD есть два типа свойств:
- **Встроенные свойства**. свойства, предопределенные для схемы Azure AD. Эти свойства предоставляют различные варианты использования и могут быть недоступны.
- **Расширения каталогов**: предоставляемые свойства, которые позволяют настроить Azure AD для собственного использования. Например, если вы расширили локальную Active Directory с определенным атрибутом и хотите поработать с этим атрибутом, можно использовать одно из предоставленных пользовательских свойств. 

## <a name="attributes-and-expressions"></a>Атрибуты и выражения
При подготовке объекта, такого как пользователь, в Azure AD создается новый экземпляр объекта пользователя. Это создание включает свойства этого объекта, которые также называются атрибутами. Изначально созданный объект имеет атрибуты, для которых заданы значения, определяемые правилами синхронизации. Эти атрибуты затем сохраняются в актуальном виде с помощью агента подготовки облака.

![Подготовка объектов](media/concept-attributes/attribute-1.png)

Например, пользователь может быть частью отдела маркетинга. Атрибут отдела Azure AD изначально создается при подготовке, а значение — в маркетинговое. За шесть месяцев позже, если они изменятся на Sales, то атрибут «отдел Active Directory» изменяется на «продажи». Это изменение синхронизируется с Azure AD и отражается в объекте пользователя Azure AD.

Синхронизация атрибутов может быть прямой, где значение в Azure AD напрямую устанавливается в значение атрибута локальный. Или же программное выражение может справиться с синхронизацией. Программное выражение требуется в тех случаях, когда для заполнения значения необходимо выполнить некоторую логику или определение.

Например, если у вас есть атрибут mail john.smith@contoso.com и требуется удалить @contoso.com часть «» и передать только значение «Джон. Иванов», то вы будете использовать нечто вроде этого:

`Replace([mail], "@contoso.com", , ,"", ,)`  

**Пример ввода и вывода:** <br>

* **Входные данные** (mail): "john.smith@contoso.com"
* **Выходные данные**: John. Smith

Дополнительные сведения о написании пользовательских выражений и синтаксиса см. в разделе [Написание выражений для сопоставления атрибутов в Azure Active Directory](../app-provisioning/functions-for-customizing-application-data.md).

В следующей таблице перечислены общие атрибуты и их синхронизация с Azure AD.


|Локальная служба Active Directory|Тип сопоставления|Azure AD|
|-----|-----|-----|
|cn|Прямой доступ|commonName
|countryCode|Прямой доступ|countryCode|
|displayName|Прямой доступ|displayName|
|givenName|Expression|givenName|
|objectGUID|Прямой доступ|sourceAnchorBinary|  
|userprincipalName|Прямой доступ|userPrincipalName|
|проксядресс|Прямой доступ|ProxyAddress|

## <a name="view-the-schema"></a>Просмотр схемы
> [!WARNING]
> Конфигурация синхронизации в облаке создает субъект-службу. Субъект-служба отображается в портал Azure. Не следует изменять сопоставления атрибутов с помощью интерфейса субъекта-службы в портал Azure.  Такой способ связывания не поддерживается.

Чтобы просмотреть схему и проверить ее, выполните следующие действия.

1.  Перейдите в [проводник Graph](https://developer.microsoft.com/graph/graph-explorer).
1.  Войдите с помощью учетной записи глобального администратора.
1.  В левой части экрана выберите **изменить разрешения** и убедитесь, что параметр **Directory. ReadWrite. ALL** *имеет значение.*
1.  Выполните запрос `https://graph.microsoft.com/beta/serviceprincipals/?$filter=startswith(Displayname,'Active')` . Этот запрос возвращает отфильтрованный список субъектов-служб.
1.  Найдите `"appDisplayName": "Active Directory to Azure Active Directory Provisioning"` и запишите значение для параметра `"id"` .
    ```
    "value": [
            {
                "id": "00d41b14-7958-45ad-9d75-d52fa29e02a1",
                "deletedDateTime": null,
                "accountEnabled": true,
                "appDisplayName": "Active Directory to Azure Active Directory Provisioning",
                "appId": "1a4721b3-e57f-4451-ae87-ef078703ec94",
                "applicationTemplateId": null,
                "appOwnerOrganizationId": "47df5bb7-e6bc-4256-afb0-dd8c8e3c1ce8",
                "appRoleAssignmentRequired": false,
                "displayName": "Active Directory to Azure Active Directory Provisioning",
                "errorUrl": null,
                "homepage": "https://account.activedirectory.windowsazure.com:444/applications/default.aspx?metadata=AD2AADProvisioning|ISV9.1|primary|z",
                "loginUrl": null,
                "logoutUrl": null,
                "notificationEmailAddresses": [],
                "preferredSingleSignOnMode": null,
                "preferredTokenSigningKeyEndDateTime": null,
                "preferredTokenSigningKeyThumbprint": null,
                "publisherName": "Active Directory Application Registry",
                "replyUrls": [],
                "samlMetadataUrl": null,
                "samlSingleSignOnSettings": null,
                "servicePrincipalNames": [
                    "http://adapplicationregistry.onmicrosoft.com/adprovisioningtoaad/primary",
                    "1a4721b3-e57f-4451-ae87-ef078703ec94"
                ],
                "signInAudience": "AzureADMultipleOrgs",
                "tags": [
                    "WindowsAzureActiveDirectoryIntegratedApp"
                ],
                "addIns": [],
                "api": {
                    "resourceSpecificApplicationPermissions": []
                },
                "appRoles": [
                    {
                        "allowedMemberTypes": [
                            "User"
                        ],
                        "description": "msiam_access",
                        "displayName": "msiam_access",
                        "id": "a0326856-1f51-4311-8ae7-a034d168eedf",
                        "isEnabled": true,
                        "origin": "Application",
                        "value": null
                    }
                ],
                "info": {
                    "termsOfServiceUrl": null,
                    "supportUrl": null,
                    "privacyStatementUrl": null,
                    "marketingUrl": null,
                    "logoUrl": null
                },
                "keyCredentials": [],
                "publishedPermissionScopes": [
                    {
                        "adminConsentDescription": "Allow the application to access Active Directory to Azure Active Directory Provisioning on behalf of the signed-in user.",
                        "adminConsentDisplayName": "Access Active Directory to Azure Active Directory Provisioning",
                        "id": "d40ed463-646c-4efe-bb3e-3fa7d0006688",
                        "isEnabled": true,
                        "type": "User",
                        "userConsentDescription": "Allow the application to access Active Directory to Azure Active Directory Provisioning on your behalf.",
                        "userConsentDisplayName": "Access Active Directory to Azure Active Directory Provisioning",
                        "value": "user_impersonation"
                    }
                ],
                "passwordCredentials": []
            },
    ```
1. Замените `{Service Principal id}` на свое значение и выполните запрос `https://graph.microsoft.com/beta/serviceprincipals/{Service Principal id}/synchronization/jobs/` .
1. Найдите `"id": "AD2AADProvisioning.fd1c9b9e8077402c8bc03a7186c8f976"` и запишите значение для параметра `"id"` .
    ```
    {
                "id": "AD2AADProvisioning.fd1c9b9e8077402c8bc03a7186c8f976",
                "templateId": "AD2AADProvisioning",
                "schedule": {
                    "expiration": null,
                    "interval": "PT2M",
                    "state": "Active"
                },
                "status": {
                    "countSuccessiveCompleteFailures": 0,
                    "escrowsPruned": false,
                    "code": "Active",
                    "lastSuccessfulExecutionWithExports": null,
                    "quarantine": null,
                    "steadyStateFirstAchievedTime": "2019-11-08T15:48:05.7360238Z",
                    "steadyStateLastAchievedTime": "2019-11-20T16:17:24.7957721Z",
                    "troubleshootingUrl": "",
                    "lastExecution": {
                        "activityIdentifier": "2dea06a7-2960-420d-931e-f6c807ebda24",
                        "countEntitled": 0,
                        "countEntitledForProvisioning": 0,
                        "countEscrowed": 15,
                        "countEscrowedRaw": 15,
                        "countExported": 0,
                        "countExports": 0,
                        "countImported": 0,
                        "countImportedDeltas": 0,
                        "countImportedReferenceDeltas": 0,
                        "state": "Succeeded",
                        "error": null,
                        "timeBegan": "2019-11-20T16:15:21.116098Z",
                        "timeEnded": "2019-11-20T16:17:24.7488681Z"
                    },
                    "lastSuccessfulExecution": {
                        "activityIdentifier": null,
                        "countEntitled": 0,
                        "countEntitledForProvisioning": 0,
                        "countEscrowed": 0,
                        "countEscrowedRaw": 0,
                        "countExported": 5,
                        "countExports": 0,
                        "countImported": 0,
                        "countImportedDeltas": 0,
                        "countImportedReferenceDeltas": 0,
                        "state": "Succeeded",
                        "error": null,
                        "timeBegan": "0001-01-01T00:00:00Z",
                        "timeEnded": "2019-11-20T14:09:46.8855027Z"
                    },
                    "progress": [],
                    "synchronizedEntryCountByType": [
                        {
                            "key": "group to Group",
                            "value": 33
                        },
                        {
                            "key": "user to User",
                            "value": 3
                        }
                    ]
                },
                "synchronizationJobSettings": [
                    {
                        "name": "Domain",
                        "value": "{\"DomainFQDN\":\"contoso.com\",\"DomainNetBios\":\"CONTOSO\",\"ForestFQDN\":\"contoso.com\",\"ForestNetBios\":\"CONTOSO\"}"
                    },
                    {
                        "name": "DomainFQDN",
                        "value": "contoso.com"
                    },
                    {
                        "name": "DomainNetBios",
                        "value": "CONTOSO"
                    },
                    {
                        "name": "ForestFQDN",
                        "value": "contoso.com"
                    },
                    {
                        "name": "ForestNetBios",
                        "value": "CONTOSO"
                    },
                    {
                        "name": "QuarantineTooManyDeletesThreshold",
                        "value": "500"
                    }
                ]
            }
    ```
1. Теперь выполните запрос `https://graph.microsoft.com/beta/serviceprincipals/{Service Principal Id}/synchronization/jobs/{AD2AAD Provisioning id}/schema` .
 
    Например, https://graph.microsoft.com/beta/serviceprincipals/653c0018-51f4-4736-a3a3-94da5dcb6862/synchronization/jobs/AD2AADProvisioning.e9287a7367e444c88dc67a531c36d8ec/schema.

   Замените `{Service Principal Id}` и `{AD2ADD Provisioning Id}` собственными значениями.

1. Этот запрос возвращает схему.

   ![Возвращенная схема](media/concept-attributes/schema-1.png)
 
## <a name="next-steps"></a>Дальнейшие действия

- [Что собой представляет подготовка?](what-is-provisioning.md)
- [Что представляет собой облачная синхронизация Azure AD Connect?](what-is-cloud-sync.md)