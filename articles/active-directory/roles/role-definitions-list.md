---
title: Вывод списка определений ролей Azure AD в Azure AD
description: Узнайте, как получить список встроенных и пользовательских ролей Azure.
services: active-directory
author: rolyon
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: how-to
ms.date: 03/07/2021
ms.author: rolyon
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: d37b2988d32c854e4184adee998341ebadcee053
ms.sourcegitcommit: 3ea12ce4f6c142c5a1a2f04d6e329e3456d2bda5
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/15/2021
ms.locfileid: "103467924"
---
# <a name="list-azure-ad-role-definitions"></a>Вывод списка определений ролей Azure AD

Определение роли — это набор разрешений, которые могут быть выполнены, например чтение, запись и удаление. Обычно это называется ролью. Azure Active Directory имеет более 60 встроенных ролей или можно создавать собственные пользовательские роли. Если вы когда-нибудь заинтересовались «что делают эти роли?», можно просмотреть подробный список разрешений для каждой из ролей.

В этой статье описывается, как получить список встроенных и пользовательских ролей Azure AD вместе с их разрешениями.

## <a name="list-all-roles"></a>Вывод списка всех ролей

1. Войдите в [центр администрирования Azure AD](https://aad.portal.azure.com) и выберите **Azure Active Directory**.

1. Выберите **роли и администраторы** , чтобы просмотреть список всех доступных ролей.

    ![Список ролей на портале Azure AD](./media/role-definitions-list/view-roles-in-azure-active-directory.png)

1. Справа выберите многоточие, а затем **Описание** , чтобы просмотреть полный список разрешений для роли.

    Страница содержит ссылки на соответствующую документацию, которая поможет вам управлять ролями.

    ![Снимок экрана, на котором отображается страница "Глобальный администратор — описание".](./media/role-definitions-list/role-description.png)

## <a name="next-steps"></a>Дальнейшие действия

* Вы можете оставить комментарий на [форуме об административных ролях Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=166032).
* Дополнительные сведения о ролях и назначении роли администратора см. в разделе [Назначение ролей администратора](permissions-reference.md).
* Сведения о разрешениях пользователей по умолчанию см. в разделе [сравнение разрешений гостевых пользователей и пользователей-участников](../fundamentals/users-default-permissions.md).
