---
title: Изменение типа проверки подлинности поддомена с помощью PowerShell и Graph-Azure Active Directory | Документация Майкрософт
description: Измените параметры проверки подлинности поддомены по умолчанию, унаследованные от параметров корневого домена в Azure Active Directory.
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 11/15/2020
ms.author: curtand
ms.reviewer: sumitp
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 734e6824f13e62ad080500eff18c4892e1f76807
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95503672"
---
# <a name="change-subdomain-authentication-type-in-azure-active-directory"></a>Изменение типа проверки подлинности поддомена в Azure Active Directory

После добавления корневого домена в Azure Active Directory (Azure AD) все последующие поддомены, добавленные в этот корневой каталог в вашей организации Azure AD, автоматически наследуют параметр проверки подлинности из корневого домена. Однако, если вы хотите управлять параметрами проверки подлинности домена независимо от параметров корневого домена, теперь можно использовать Microsoft Graph API. Например, если у вас есть федеративный корневой домен, например contoso.com, эта статья поможет вам проверить поддомены, например child.contoso.com, управляемым вместо федеративного.

На портале Azure AD, когда родительский домен является федеративным, и администратор пытается проверить управляемый поддомен на странице " **личные имена доменов** ", вы получите ошибку "не удалось добавить домен" по причине "одно или несколько свойств содержат недопустимые значения". Если вы попытаетесь добавить этот поддомен из центра администрирования Microsoft 365, появится подобная ошибка. Дополнительные сведения об ошибке см. в разделе [дочерний домен не наследует изменения родительского домена в Office 365, Azure или Intune](/office365/troubleshoot/administration/child-domain-fails-inherit-parent-domain-changes).

## <a name="how-to-verify-a-custom-subdomain"></a>Проверка пользовательского поддомена

Так как поддомены наследуют тип проверки подлинности корневого домена по умолчанию, необходимо перевести поддомен в корневой домен в Azure AD с помощью Microsoft Graph, чтобы можно было задать тип проверки подлинности в нужном типе.

### <a name="add-the-subdomain-and-view-its-authentication-type"></a>Добавление поддомена и просмотр его типа проверки подлинности

1. Используйте PowerShell, чтобы добавить новый поддомен, имеющий тип проверки подлинности по умолчанию корневого домена. Центры администрирования Azure AD и Microsoft 365 пока не поддерживают эту операцию.

   ```powershell
   New-MsolDomain -Name "child.mydomain.com" -Authentication Federated
   ```

1. Используйте [Azure AD Graph Explorer](https://graphexplorer.azurewebsites.net) для получения домена. Поскольку домен не является корневым доменом, он наследует тип проверки подлинности корневого домена. Команда и результаты могут выглядеть следующим образом, используя собственный идентификатор клиента:

   ```http
   GET https://graph.windows.net/{tenant_id}/domains?api-version=1.6
   
   Return:
     {
         "authenticationType": "Federated",
         "availabilityStatus": null,
         "isAdminManaged": true,
         "isDefault": false,
         "isDefaultForCloudRedirections": false,
         "isInitial": false,
         "isRoot": false,          <---------------- Not a root domain, so it inherits parent domain's authentication type (federated)
         "isVerified": true,
         "name": "child.mydomain.com",
         "supportedServices": [],
         "forceDeleteState": null,
         "state": null,
         "passwordValidityPeriodInDays": null,
         "passwordNotificationWindowInDays": null
     },
   ```

### <a name="use-azure-ad-graph-explorer-api-to-make-this-a-root-domain"></a>Использование API Azure AD Graph Explorer для создания корневого домена

Используйте следующую команду для продвижения дочернего домена:

```http
POST https://graph.windows.net/{tenant_id}/domains/child.mydomain.com/promote?api-version=1.6
```

### <a name="change-the-subdomain-authentication-type"></a>Изменение типа проверки подлинности поддомена

1. Чтобы изменить тип проверки подлинности поддомена, используйте следующую команду:

   ```powershell
   Set-MsolDomainAuthentication -DomainName child.mydomain.com -Authentication Managed
   ```

1. Проверьте с помощью GET в Azure AD Graph Explorer, что тип проверки подлинности поддомена теперь является управляемым:

   ```http
   GET https://graph.windows.net/{{tenant_id} }/domains?api-version=1.6
   
   Return:
     {
         "authenticationType": "Managed",   <---------- Now this domain is successfully added as Managed and not inheriting Federated status
         "availabilityStatus": null,
         "isAdminManaged": true,
         "isDefault": false,
         "isDefaultForCloudRedirections": false,
         "isInitial": false,
         "isRoot": true,   <------------------------------ Also a root domain, so not inheriting from parent domain any longer
         "isVerified": true,
         "name": "child.mydomain.com",
         "supportedServices": [
             "Email",
             "OfficeCommunicationsOnline",
             "Intune"
         ],
         "forceDeleteState": null,
         "state": null,
         "passwordValidityPeriodInDays": null,
         "passwordNotificationWindowInDays": null }
   ```

## <a name="next-steps"></a>Дальнейшие действия

- [Добавление имен личных доменов](../fundamentals/add-custom-domain.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context)
- [Управление именами доменов](domains-manage.md)
- [Раздел об удалении имени личного домена с помощью операции ForceDelete в API Microsoft Graph](/graph/api/domain-forcedelete?view=graph-rest-beta&preserve-view=true)