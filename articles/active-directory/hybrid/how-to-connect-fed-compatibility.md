---
title: Список совместимости с федерацией Azure AD
description: На этой странице указаны сторонние поставщики, которые можно использовать для реализации единого входа.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: curtand
ms.assetid: 22c8693e-8915-446d-b383-27e9587988ec
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/23/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 895e7347feea3190ba9bdc273200d2985dfaa0d1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89661831"
---
# <a name="azure-ad-federation-compatibility-list"></a>Список совместимости с федерацией Azure AD
Azure Active Directory предоставляет единый вход и улучшенную безопасность доступа к приложениям для Microsoft 365 и других служб Microsoft Online Services для гибридных и облачных реализаций без необходимости решения сторонних разработчиков. Microsoft 365, как и большинство веб-служб Майкрософт, интегрирована с Azure Active Directory для служб каталогов, проверки подлинности и авторизации. Azure Active Directory также предоставляет возможность единого входа в тысячи приложений SaaS и локальные веб-приложения. Список поддерживаемых приложений SaaS см. в [коллекции приложений](https://azuremarketplace.microsoft.com/marketplace/apps/category/azure-active-directory-apps) Azure Active Directory. 

## <a name="idp-validation"></a>Проверка IdP
Если в вашей организации используется стороннее решение Федерации, вы можете настроить единый вход для локальных Active Directory пользователей с помощью служб Microsoft Online Services, таких как Microsoft 365, при условии, что стороннее решение Федерации совместимо с Azure Active Directory.  По вопросам совместимости обратитесь к поставщику удостоверений.  Если вы хотите просмотреть список поставщиков удостоверений, которые ранее проверялись на совместимость с Azure AD, Майкрософт, см. [документацию по совместимости поставщика удостоверений Azure AD](https://www.microsoft.com/download/details.aspx?id=56843). 

>[!NOTE]
>Корпорация Майкрософт больше не предоставляет проверочное тестирование независимых поставщиков удостоверений, чтобы обеспечить совместимость с Azure Active Directory. Если вы хотите проверить взаимодействие своего продукта, ознакомьтесь с этими [рекомендациями](https://www.microsoft.com/download/details.aspx?id=56843). 

## <a name="next-steps"></a>Дальнейшие действия

- [Интеграция локальных каталогов с Azure Active Directory](whatis-hybrid-identity.md)
- [Azure AD Connect и федерация](how-to-connect-fed-whatis.md)
