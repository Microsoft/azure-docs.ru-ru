---
title: Роли, которыми нельзя управлять в управление привилегированными пользователями-Azure Active Directory | Документация Майкрософт
description: Сведения о ролях, которыми вы не можете управлять в Azure Active Directory Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.subservice: pim
ms.date: 05/11/2020
ms.author: curtand
ms.custom: pim ; H1Hack27Feb2017;oldportal;it-pro;
ms.collection: M365-identity-device-management
ms.openlocfilehash: c72587c5486ed61215fd20c215a1dd194f4b7bc4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92372418"
---
# <a name="roles-you-cant-manage-in-privileged-identity-management"></a>Роли, которыми нельзя управлять в управление привилегированными пользователями

Azure Active Directory (Azure AD) управление привилегированными пользователями (PIM) позволяет управлять всеми ролями Azure [AD](../roles/permissions-reference.md) и всеми [ролями Azure](../../role-based-access-control/built-in-roles.md). Роли Azure также могут включать пользовательские роли, присоединенные к группам управления, подпискам, группам ресурсов и ресурсам. Однако, есть также несколько ролей, которыми вы не можете управлять. В этой статье описываются роли, которыми нельзя управлять в управление привилегированными пользователями.

## <a name="classic-subscription-administrator-roles"></a>Роли классического администратора подписки

Вы не можете управлять следующими ролями администратора классической подписки в управление привилегированными пользователями:

- Администратор учетной записи
- Администратор служб
- Соадминистратор

Дополнительные сведения о ролях администратора классической подписки см. в разделе [роли администратора классической подписки, роли Azure и роли администратора Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md).

## <a name="what-about-microsoft-365-admin-roles"></a>Что насчет ролей администратора Microsoft 365?

Поддерживаются все роли Microsoft 365 на портале ролей и администраторов Azure AD, такие как администратор Exchange и администратор SharePoint, но мы не поддерживаем определенные роли в Exchange RBAC и RBAC. Дополнительные сведения об этих Microsoft 365 службах см. в разделе [роли администратора Microsoft 365](/office365/admin/add-users/about-admin-roles).

> [!NOTE]
> У соответствующих пользователей для роли администратора SharePoint, роли администратора устройства и всех ролей, пытающихся получить доступ к центру безопасности и соответствия требованиям Майкрософт, может возникать задержка до нескольких часов после активации своей роли. Мы работаем с этими командами для решения проблем.

## <a name="next-steps"></a>Дальнейшие действия

- [Назначение ролей Azure AD в управление привилегированными пользователями](pim-how-to-add-role-to-user.md)
- [Назначение ролей ресурсов Azure в управление привилегированными пользователями](pim-resource-roles-assign-roles.md)