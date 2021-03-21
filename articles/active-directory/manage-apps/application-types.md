---
title: Просмотр приложений с помощью клиента Azure Active Directory для управления удостоверениями
description: Узнайте, как просмотреть все приложения, использующие клиент Azure Active Directory для управления удостоверениями.
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: reference
ms.date: 01/07/2021
ms.author: kenwith
ms.openlocfilehash: 443e8c83ec29f9f0478e3881b9e6d6a8eb0bf403
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99259792"
---
# <a name="viewing-apps-using-your-azure-ad-tenant-for-identity-management"></a>Просмотр приложений с помощью клиента Azure AD для управления удостоверениями
В [руководстве по управлению приложениями](view-applications-portal.md) представлены основные принципы. В нем вы узнаете, как просмотреть все приложения с помощью клиента Azure AD для управления удостоверениями. В этой статье более подробно подробно типы приложений.

## <a name="why-does-a-specific-application-appear-in-my-all-applications-list"></a>Почему приложение появляется в моем списке всех приложений?
При фильтрации **всех приложений** в **списке** **все приложения** отображается каждый объект субъекта-службы в клиенте. Объекты участников-служб могут попадать в этот список разными способами.
- При добавлении любого приложения из коллекции приложений, например:
   - **Azure AD — корпоративные приложения** — приложения, добавленные в клиент с помощью параметра **корпоративные приложения** на портале Azure AD. Обычно это приложения, интегрированные с использованием стандарта SAML Standard.
   - **Azure AD — регистрация приложений** — приложения, добавленные в клиент с помощью параметра **Регистрация приложений** на портале Azure AD. Обычно это пользовательские разработанные приложения с использованием стандартов Open ID Connect и OAuth.
   - **Приложения прокси приложения** — приложения, работающее в локальной среде, которым необходимо предоставить безопасный единый вход извне.
- При регистрации или входе в приложение стороннего разработчика, интегрированного с Azure Active Directory. Примерами таких приложений являются [Smartsheet](https://app.smartsheet.com/b/home) или [DocuSign](https://www.docusign.net/member/MemberLogin.aspx).
- Такие приложения Майкрософт, как Microsoft 365.
- При добавлении новой регистрации приложения путем создания собственного приложения с помощью [реестра приложений](../develop/quickstart-register-app.md).
- При добавлении новой регистрации приложения путем создания собственного приложения с помощью [портала регистрации приложений версии 2.0](../develop/quickstart-register-app.md).
- При добавлении приложения вы разрабатываете с помощью [методов проверки Подлинности ASP.NET](https://www.asp.net/visual-studio/overview/2013/creating-web-projects-in-visual-studio#orgauthoptions) в Visual Studio или [подключенные службы](https://devblogs.microsoft.com/visualstudio/connecting-to-cloud-services/)
- При создании субъекта-службы с помощью [модуля Azure AD PowerShell](/powershell/azure/active-directory/install-adv2).
- Когда вы в качестве администратора предоставляете приложению [согласие](../develop/howto-convert-app-to-be-multi-tenant.md) на использование данных в клиенте.
- Когда [пользователь предоставляет приложению согласие](../develop/howto-convert-app-to-be-multi-tenant.md) на использование данных в клиенте.
- При включении определенных служб, которые хранят данные в клиенте. Примером может служить служба сброса паролей, которая оформлена как субъект-служба для безопасного хранения политики сброса паролей.

Дополнительные сведения о том, как и почему приложения добавляются в каталог, см. [в разделе как приложения добавляются в Azure AD](../develop/active-directory-how-applications-are-added.md).

## <a name="next-steps"></a>Дальнейшие действия
[Управление приложениями с помощью Azure Active Directory](what-is-application-management.md)
