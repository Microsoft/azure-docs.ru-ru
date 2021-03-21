---
title: GMSA командлеты PowerShell для агента подготовки облака Azure AD Connect
description: Узнайте, как использовать командлет подготовки gMSA в облаке Azure AD Connect PowerShell.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 11/16/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 411a8e46151a762bcd270fb676f78a91f760ac4f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101653802"
---
# <a name="azure-ad-connect-cloud-provisioning-agent-gmsa-powershell-cmdlets"></a>GMSA командлеты PowerShell для агента подготовки облака Azure AD Connect

В этом документе описывается, как описать командлеты gMSA для агента подготовки облака Azure AD Connect в PowerShell. Эти командлеты позволяют повысить степень детализации разрешений, применяемых к учетной записи службы (gmsa). По умолчанию Azure AD Connect Cloud Sync применяет все разрешения, аналогичные Azure AD Connect, в gmsa по умолчанию или пользовательском gmsa. 

В этом документе будут рассмотрены следующие командлеты:  

`Set-AADCloudSyncRestrictedPermissions`

`Ste-AADCloudSyncPermissions` 

## <a name="how-to-use-the-cmdlets"></a>Использование командлетов:  

Для использования этих командлетов необходимы следующие необходимые компоненты.

1. Установите агент подготовки. 
2. Импортируйте модуль агента подготовки PS в сеанс PowerShell. 

 ```PowerShell
 Import-Module "C:\Program Files\Microsoft Azure AD Connect Provisioning Agent\Microsoft.CloudSync.Powershell.dll"  
 ```
3. Удалите существующие разрешения.  Чтобы удалить все существующие разрешения для учетной записи службы, кроме САМОСТОЯТЕЛЬНОго использования, сделайте следующее: `Set-AADCloudSyncRestrictedPermission` .  

    Для этого командлета требуется параметр с именем `Credential` , который может быть передан, или запрос, если он вызывается без него.

    Чтобы создать переменную, используйте  

   `$credential = Get-Credential` 

   При этом пользователю будет предложено ввести имя пользователя и пароль. Учетные данные должны иметь права администратора домена (домена, на котором установлен агент), а также администратора предприятия. 

4.  Затем можно вызвать командлет, чтобы удалить дополнительные разрешения: 
   ```PowerShell
   Set-AADCloudSyncRestrictedPermissions -Credential $credential 
   ```
5. Или можно просто вызвать метод 

   `Set-AADCloudSyncRestrictedPermissions` в этом случае будут запрашиваться учетные данные. 

 6.  Добавить конкретный тип разрешения.  Добавленные разрешения совпадают с Azure AD Connect.  Примеры настройки разрешений см. в разделе [использование Set-аадклаудсинкпермиссионс](#using-set-aadcloudsyncpermissions) ниже.

## <a name="using-set-aadcloudsyncpermissions"></a>Использование Set-AADCloudSyncPermissions 
`Set-AADCloudSyncPermissions` поддерживает следующие типы разрешений, которые идентичны разрешениям, используемым Azure AD Connect. Поддерживаются следующие типы разрешений: 

|Тип разрешения|Описание|
|-----|-----|
|басикреад| См. раздел разрешения [басикреад](../../active-directory/hybrid/how-to-connect-configure-ad-ds-connector-account.md#configure-basic-read-only-permissions) для Azure AD Connect|
|пассвордхашсинк|См. раздел разрешения [пассвордхашсинк](../../active-directory/hybrid/how-to-connect-configure-ad-ds-connector-account.md#permissions-for-password-hash-synchronization) для Azure AD Connect|
|пассвордвритебакк|См. раздел разрешения [пассвордвритебакк](../../active-directory/hybrid/how-to-connect-configure-ad-ds-connector-account.md#permissions-for-password-writeback) для Azure AD Connect|
|хибридексчанжепермиссионс|См. раздел разрешения [хибридексчанжепермиссионс](../../active-directory/hybrid/how-to-connect-configure-ad-ds-connector-account.md#permissions-for-exchange-hybrid-deployment) для Azure AD Connect| 
|ексчанжемаилпубликфолдерпермиссионс| См. раздел разрешения [ексчанжемаилпубликфолдерпермиссионс](../../active-directory/hybrid/how-to-connect-configure-ad-ds-connector-account.md#permissions-for-exchange-mail-public-folders-preview) для Azure AD Connect| 
|клаудхр| Применяет "полный доступ" к "дочерние объекты пользователя" и "Создание и удаление пользовательских объектов" для "Этот объект и все дочерние объекты"| 
|Все|Добавляет все указанные выше разрешения.| 



Аадклаудсинкпермиссионс можно использовать одним из двух способов:
- [Предоставление определенного разрешения всем настроенным доменам](#grant-a-certain-permission-to-all-configured-domains) 
- [Предоставление определенного разрешения для определенного домена](#grant-a-certain-permission-to-a-specific-domain) 
## <a name="grant-a-certain-permission-to-all-configured-domains"></a>Предоставление определенного разрешения всем настроенным доменам 
Для предоставления определенных разрешений всем настроенным доменам потребуется использовать учетную запись администратора предприятия.


 ```PowerShell
Set-AADCloudSyncPermissions -PermissionType “Any mentioned above” -EACredential $credential (prepopulated same as above [$credential = Get-Credential]) 
```

## <a name="grant-a-certain-permission-to-a-specific-domain"></a>Предоставление определенного разрешения для определенного домена 
Для предоставления определенных разрешений конкретному домену потребуется, по крайней мере, учетная запись администратора домена, которую вы пытаетесь добавить.


 ```PowerShell
Set-AADCloidSyncPermissions -PermissionType “Any mentioned above” -TargetDomain “FQDN of domain” (has to be already configured through wizard) -TargetDomaincredential $credential(same as above) 
```
 

Примечание. для 1. Учетные данные должны принадлежать минимальному администратору предприятия. 

В разделе 2. Учетные данные могут быть администратором домена или администратора предприятия. 

  

