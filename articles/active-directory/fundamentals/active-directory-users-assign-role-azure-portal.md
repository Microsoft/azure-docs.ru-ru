---
title: Назначение ролей Azure AD пользователям Azure Active Directory | Документация Майкрософт
description: Инструкции по назначению ролей администратора и других ролей пользователям с помощью Azure Active Directory.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 08/31/2020
ms.author: ajburnle
ms.reviewer: jeffsta
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: fe0c7eec5c023fcba3152ad6329d318210a0c2ab
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92370939"
---
# <a name="assign-administrator-and-non-administrator-roles-to-users-with-azure-active-directory"></a>Назначение ролей администратора и других ролей пользователям с помощью Azure Active Directory

В Azure Active Directory (Azure AD), если одному из пользователей требуется разрешение на управление ресурсами Azure AD, необходимо назначить их роли, предоставляющей необходимые им разрешения. Сведения о том, какие роли управляют ресурсами Azure и какие роли управляют ресурсами Azure AD, см. в статье [роли администратора классической подписки, роли Azure и роли Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md).

Дополнительные сведения о доступных ролях Azure AD см. в разделе [назначение ролей администратора в Azure Active Directory](../roles/permissions-reference.md). Сведения о добавлении пользователей см. в разделе [Добавление новых пользователей в Azure Active Directory](add-users-azure-active-directory.md).

## <a name="assign-roles"></a>Назначение ролей

Распространенным способом назначения ролей Azure AD для пользователя является страница « **назначенные роли** » для пользователя. Кроме того, можно настроить право на повышение прав пользователя в роли с помощью управление привилегированными пользователями (PIM). Дополнительные сведения об использовании PIM см. в разделе [Управление привилегированными пользователями](../privileged-identity-management/index.yml).

> [!Note]
> Если у вас есть план лицензирования Azure AD Premium P2 и уже используется PIM, все задачи управления ролями выполняются в [Управление привилегированными пользователями](../roles/manage-roles-portal.md). В настоящее время эта функция ограничена одновременной назначением одной роли. Сейчас нельзя выбрать несколько ролей и назначить их пользователю одновременно.
>
> ![Роли Azure AD, управляемые в PIM для пользователей, которые уже используют PIM и имеют лицензию Premium P2](./media/active-directory-users-assign-role-azure-portal/pim-manages-roles-for-p2.png)

## <a name="assign-a-role-to-a-user"></a>Назначение роли пользователю

1. Перейдите к [портал Azure](https://portal.azure.com/) и войдите в систему, используя учетную запись глобального администратора для каталога.

2. Найдите и выберите **Azure Active Directory**.

      ![Поиск Azure Active Directory на портале Azure](media/active-directory-users-assign-role-azure-portal/search-azure-active-directory.png)

3. Выберите **Пользователи**.

4. Найдите и выберите пользователя, который получает назначение роли. Например, _Alain Charon_.

      ![Страница "все пользователи" — Выбор пользователя](media/active-directory-users-assign-role-azure-portal/directory-role-select-user.png)

5. На странице **(Alain Чарон-Profile** выберите **назначенные роли**.

    Откроется страница **(Alain Чарон-Administrative Roles** .

6. Выберите **добавить назначения**, выберите роль для назначения (Alain (например, _Администратор приложения_), а затем нажмите **кнопку Выбрать**.

    ![Страница "назначенные роли" — Отображение выбранной роли](media/active-directory-users-assign-role-azure-portal/directory-role-select-role.png)

    Роль администратора приложения назначается (Alain Чарон, и она отображается на странице **(Alain Чарон-административные роли** .

## <a name="remove-a-role-assignment"></a>Удаление назначения ролей

Если необходимо удалить назначение роли от пользователя, это можно также сделать на странице **(Alain Чарон-My Roles (роли администрирования)** .

### <a name="to-remove-a-role-assignment-from-a-user"></a>Удаление назначения роли

1. Выберите **Azure Active Directory**, нажмите **Пользователи**, а затем найдите и выберите пользователя, у которого необходимо удалить роль. Например, _Alain Charon_.

2. Выберите **назначенные роли**, щелкните **Администратор приложения**, а затем выберите **удалить назначение**.

    ![Страница «назначенные роли», показывающая выбранную роль и параметр «Удалить»](media/active-directory-users-assign-role-azure-portal/directory-role-remove-role.png)

    Роль администратора приложения удаляется из (Alain Чарон и больше не отображается на странице **(Alain Чарон-административные роли** .

## <a name="next-steps"></a>Дальнейшие действия

- [Добавление или удаление пользователей](add-users-azure-active-directory.md)

- [Добавление или изменение данных профиля](active-directory-users-profile-azure-portal.md)

- [Добавление гостевых пользователей из другого каталога](../external-identities/what-is-b2b.md)

Другие задачи управления пользователями, которые вы можете извлечь, доступны в [Azure Active Directory документации по управлению пользователями](../enterprise-users/index.yml).