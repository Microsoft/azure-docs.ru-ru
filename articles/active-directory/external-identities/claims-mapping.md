---
title: Сопоставление утверждений пользователей службы совместной работы B2B — Azure Active Directory
description: Настройка утверждений пользователей, передаваемых в токене SAML для пользователей Azure Active Directory (Azure AD) B2B.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 04/06/2018
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: elisolMS
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3508865d9f89501cf70e09087c6a609beb6662b2
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87910259"
---
# <a name="b2b-collaboration-user-claims-mapping-in-azure-active-directory"></a>Сопоставление утверждений пользователя службы совместной работы B2B в Azure Active Directory

Azure Active Directory (Azure AD) позволяет настраивать утверждения, которые передаются в токене SAML для пользователей службы совместной работы B2B. Когда пользователь проходит аутентификацию в приложении, Azure AD выдает токен SAML для приложения. В токене собрана информация (утверждения), которая однозначно идентифицирует пользователя. По умолчанию здесь указаны имя пользователя, адрес электронной почты, имя и фамилия пользователя.

На [портале Azure](https://portal.azure.com) можно просмотреть или изменить утверждения, которые передаются приложению в токене SAML. Чтобы получить доступ к параметрам, выберите **Azure Active Directory**  >  **корпоративные приложения** > приложение, для которого настроен единый вход > **единый вход**. Параметры токена SAML отобразятся в разделе **Атрибуты пользователя**.

![Список атрибутов токена SAML в пользовательском интерфейсе](media/claims-mapping/view-claims-in-saml-token.png)

У вас могут быть две причины изменять утверждения, выдаваемые в токене SAML:

1. Возможно, приложение ожидает утверждения с другим набором URI и значениями.

2. Приложение ожидает, что утверждение NameIdentifier будет содержать данные, отличные от имени участника-пользователя, которое хранится в Azure Active Directory.

Дополнительные сведения о добавлении и изменении утверждений см. в статье [Настройка утверждений, выпущенных в токене SAML для корпоративных приложений в Azure Active Directory](../develop/active-directory-saml-claims-customization.md).

По соображениям безопасности сопоставление NameID и имени участника-пользователя между клиентами для пользователей службы совместной работы B2B запрещено.

## <a name="next-steps"></a>Дальнейшие действия

- Сведения о свойствах пользователя службы совместной работы B2B см. в статье [Свойства пользователя службы совместной работы Azure Active Directory B2B](user-properties.md).
- Сведения о токенах пользователей службы совместной работы B2B см. в статье [Основные сведения о токенах пользователей в службе совместной работы Azure AD B2B](user-token.md).

