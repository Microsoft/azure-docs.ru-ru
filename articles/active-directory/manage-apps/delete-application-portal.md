---
title: Краткое руководство. Удаление приложения из арендатора Azure Active Directory (Azure AD)
description: В этом кратком руководстве показано, как с помощью портала Azure можно удалить приложение из арендатора Azure Active Directory (Azure AD).
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: quickstart
ms.workload: identity
ms.date: 12/28/2020
ms.author: kenwith
ms.openlocfilehash: e2b8bd47f7d124562a38413d36b849d1f7efacee
ms.sourcegitcommit: 7e97ae405c1c6c8ac63850e1b88cf9c9c82372da
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/29/2020
ms.locfileid: "97802768"
---
# <a name="quickstart-delete-an-application-from-your-azure-active-directory-azure-ad-tenant"></a>Краткое руководство. Удаление приложения из арендатора Azure Active Directory (Azure AD)

В этом кратком руководстве показано, как с помощью портала Azure можно удалить приложение из арендатора Azure Active Directory (Azure AD).

## <a name="prerequisites"></a>Предварительные требования

Чтобы удалить приложение из арендатора Azure AD, вам потребуется следующее:

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Одна из следующих ролей: глобальный администратор, администратор облачных приложений, администратор приложений или владелец субъекта-службы.
- Необязательное действие: Знакомство с инструкциями по [просмотру приложений](view-applications-portal.md).
- Необязательное действие: Знакомство с инструкциями по [добавлению приложения](add-application-portal.md).
- Необязательное действие: Знакомство с инструкциями по [настройке приложения](add-application-portal-configure.md).
- Необязательное действие: [Назначение пользователей приложению](add-application-portal-assign-users.md).
- Необязательное действие: Знакомство со статьей [Настройка единого входа](add-application-portal-setup-sso.md).

>[!IMPORTANT]
>Не используйте рабочую среду для проверки инструкций, приведенных в этом кратком руководстве.

## <a name="delete-an-application-from-your-azure-ad-tenant"></a>Удаление приложения из арендатора Azure AD

Чтобы удалить приложение из арендатора Azure AD, сделайте следующее:

1. На портале Azure AD выберите элемент **Корпоративные приложения**. Затем найдите и выберите приложение, которое нужно удалить. В этом случае мы удалим приложение **GitHub_test**, которое было добавлено с помощью инструкций из предыдущего краткого руководства.
1. В разделе **Управление** в левой области выберите **Свойства**.
1. Выберите **Удалить** и нажмите **Да**, чтобы подтвердить, что вы хотите удалить приложение из своего арендатора Azure AD.

> [!TIP]
> Вы можете автоматизировать управление приложениями с помощью API Graph — см. [соответствующую статью](/graph/application-saml-sso-configure-api).

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы завершили работу с этой серией кратких руководств, удалите приложение, чтобы очистить тестового арендатора. В этом кратком руководстве был рассмотрен процесс удаления приложения.

## <a name="next-steps"></a>Дальнейшие действия

Вы завершили работу с серией кратких руководств! Дополнительные сведения о едином входе см. в [этой статье](what-is-single-sign-on.md). Или изучите рекомендации по управлению приложениями.
> [!div class="nextstepaction"]
> [Рекомендации по управлению приложениями](application-management-fundamentals.md)
