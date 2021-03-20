---
title: Федерацию нескольких Azure AD с помощью одного AD FS Azure
description: Из этого документа вы узнаете, как создать федерацию нескольких экземпляров Azure AD с одним экземпляром AD FS.
keywords: федерация, ADFS, AD FS, несколько клиентов, один клиент AD FS, один экземпляр ADFS, федерация нескольких клиентов, федерация ADFS с несколькими лесами, AAD Сonnect, федерация, федерация между клиентами
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/17/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 956428b6f197912e2ab7c3a94133ed9d59f37749
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89279930"
---
# <a name="federate-multiple-instances-of-azure-ad-with-single-instance-of-ad-fs"></a>Федерация нескольких экземпляров Azure AD с одним экземпляром AD FS

Одна высокодоступная ферма AD FS может объединять несколько лесов при наличии взаимного доверия между ними. Эти несколько лесов могут соответствовать или не соответствовать одной службе Azure Active Directory. Эта статья содержит инструкции по настройке федерации между одним экземпляром AD FS и несколькими лесами, которые синхронизируются с другим экземпляром Azure AD.

![Федерация нескольких клиентов с одним экземпляром AD FS](./media/how-to-connect-fed-single-adfs-multitenant-federation/concept.png)
 
> [!NOTE]
> В этом сценарии не поддерживаются обратная запись устройств и автоматическое присоединение устройств.

> [!NOTE]
> Azure AD Connect нельзя использовать для настройки федерации в этом сценарии, так как Azure AD Connect может настраивать федерацию для доменов в одном экземпляре Azure AD.

## <a name="steps-for-federating-ad-fs-with-multiple-azure-ad"></a>Шаги для федерации AD FS и нескольких экземпляров Azure AD

Представьте, что домен contoso.com в Azure Active Directory (contoso.onmicrosoft.com) включен в федерацию локального экземпляра AD FS, установленного в локальной среде Active Directory contoso.com. Fabrikam.com является доменом в fabrikam.onmicrosoft.com Azure Active Directory.

## <a name="step-1-establish-a-two-way-trust"></a>Шаг 1. Установка взаимного доверия
 
Чтобы служба AD FS в contoso.com могла проверять подлинность пользователей в fabrikam.com, требуется взаимное доверие между contoso.com и fabrikam.com. Следуйте рекомендациям, приведенным в этой [статье](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc816590(v=ws.10)), чтобы создать отношения взаимного доверия.
 
## <a name="step-2-modify-contosocom-federation-settings"></a>Шаг 2. Изменение параметров федерации contoso.com 
 
Издателем отдельного домена, включенного в федерацию AD FS, по умолчанию является http\://ADFSServiceFQDN/adfs/services/trust, например `http://fs.contoso.com/adfs/services/trust`. Служба Azure Active Directory требует отдельного издателя для каждого федеративного домена. Так как один и тот же экземпляр AD FS будет включать в федерацию два домена, значение издателя должно быть изменено, чтобы оно было уникальным для каждого домена AD FS, которое входит в федерацию с Azure Active Directory. 
 
На сервере AD FS откройте Azure AD PowerShell (убедитесь, что модуль MSOnline установлен) и выполните следующие действия:
 
Подключитесь к службе Azure Active Directory, которая содержит домен contoso.com (Connect-MsolService). Обновите параметры федерации для contoso.com (Update-MsolFederatedDomain - DomainName contoso.com — SupportMultipleDomain).
 
Издатель в параметре федерации домена будет изменен на http\://contoso.com/adfs/services/trust, и для проверяющей стороны Azure AD будет добавлено правило утверждения выдачи, чтобы она выдавала правильный идентификатор издателя на основе суффикса имени участника-пользователя.
 
## <a name="step-3-federate-fabrikamcom-with-ad-fs"></a>Шаг 3. Федерация fabrikam.com с AD FS
 
В сеансе Azure AD PowerShell подключитесь к службе Azure Active Directory, которая содержит домен fabrikam.com.

```powershell
Connect-MsolService
```
Преобразуйте управляемый домен fabrikam.com в федеративный:

```powershell
Convert-MsolDomainToFederated -DomainName fabrikam.com -Verbose -SupportMultipleDomain
```
 
Вышеуказанная операция добавит домен fabrikam.com в федерацию с тем же экземпляром AD FS. Параметры обоих доменов можно проверить с помощью командлета Get-MsolDomainFederationSettings.

## <a name="next-steps"></a>Дальнейшие действия
[Подключение Active Directory к Azure Active Directory](whatis-hybrid-identity.md)