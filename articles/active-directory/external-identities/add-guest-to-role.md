---
title: Добавление пользователя службы совместной работы B2B в роль Azure Active Directory
description: Добавление роли гостевого пользователя в Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 05/08/2018
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: c5e9983c265fcc032908424f747066f8f348771f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87910355"
---
# <a name="grant-permissions-to-users-from-partner-organizations-in-your-azure-active-directory-tenant"></a>Предоставление пользователям разрешений от партнерских организаций в клиенте Azure Active Directory

Пользователи службы совместной работы Azure Active Directory (Azure AD) B2B добавляются в каталог как гостевые пользователи, а разрешения гостей в каталоге по умолчанию ограничены. Ваша организация может быть заинтересована в назначении некоторым гостевым пользователям более привилегированных ролей. Чтобы это сделать, вы можете добавлять гостевых пользователей в любые роли, исходя из потребностей своей организации.

## <a name="default-role"></a>Роль по умолчанию

![Снимок экрана, показывающий роль каталога по умолчанию](./media/add-guest-to-role/default-role.png)

## <a name="global-administrator-role"></a>Роль "Глобальный администратор"

![Снимок экрана, показывающий роль глобального администратора](./media/add-guest-to-role/global-admin-role.png)

## <a name="limited-administrator-role"></a>Роль "Администратор с ограниченными правами"

![Снимок экрана, показывающий роль ограниченного администратора](./media/add-guest-to-role/limited-admin-role.png)

## <a name="next-steps"></a>Дальнейшие действия

- [Что такое служба совместной работы Azure AD B2B?](what-is-b2b.md)
- [Свойства пользователя службы совместной работы Azure Active Directory B2B](user-properties.md)
