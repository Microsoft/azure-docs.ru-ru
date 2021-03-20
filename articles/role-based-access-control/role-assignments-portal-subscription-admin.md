---
title: Назначение пользователя администратором подписки Azure с помощью Azure RBAC
description: Узнайте, как сделать пользователя администратором подписки Azure с помощью портал Azure и управления доступом на основе ролей Azure (Azure RBAC).
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.topic: how-to
ms.workload: identity
ms.date: 01/11/2021
ms.author: rolyon
ms.openlocfilehash: dec5888127ed1fc291bec244a44cfb71e343e3bb
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100556836"
---
# <a name="assign-a-user-as-an-administrator-of-an-azure-subscription"></a>Назначение пользователя администратором подписки Azure

Чтобы предоставить пользователю права администратора подписки Azure, назначьте ему роль [Владелец](built-in-roles.md#owner) в этой подписке. Роль Владелец предоставляет пользователю полный доступ ко всем ресурсам в подписке, включая разрешение на предоставление доступа другим пользователям. Этот процесс не отличается от любой процедуры назначения ролей.

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [Azure role assignment prerequisites](../../includes/role-based-access-control/prerequisites-role-assignments.md)]

## <a name="step-1-open-the-subscription"></a>Шаг 1. Открытие подписки

1. Войдите на [портал Azure](https://portal.azure.com).

1. В поле поиска вверху найдите элемент подписки.

    ![портал Azure Поиск группы ресурсов](./media/shared/sub-portal-search.png)

1. Щелкните подписку, которую хотите использовать.

    Ниже приведен пример подписки.

    ![Общие сведения о группе ресурсов](./media/shared/sub-overview.png)

## <a name="step-2-open-the-add-role-assignment-pane"></a>Шаг 2. Открытие панели "Добавление назначения ролей"

**Управление доступом (IAM)** — это страница, которая обычно используется для назначения ролей для предоставления доступа к ресурсам Azure. Он также называется управление удостоверениями и доступом (IAM) и появляется в нескольких расположениях в портал Azure.

1. Щелкните **Управление доступом (IAM)** .

    Ниже приведен пример страницы управления доступом (IAM) для подписки.

    ![Страница управления доступом (IAM) для группы ресурсов](./media/shared/sub-access-control.png)

1. Перейдите на вкладку **назначения ролей** , чтобы просмотреть назначения ролей в этой области.

1. Нажмите кнопку **Добавить**  >  **добавить назначение ролей**.
   Если у вас нет прав назначать роли, функция "Добавить назначение роли" будет неактивна.

   ![Меню "Добавить назначение ролей"](./media/shared/add-role-assignment-menu.png)

    Откроется панель "Добавить назначение ролей".

   ![Область "Добавить назначение ролей"](./media/shared/add-role-assignment.png)

## <a name="step-3-select-the-owner-role"></a>Шаг 3. Выбор роли владельца

Роль [владельца](built-in-roles.md#owner) предоставляет полный доступ для управления всеми ресурсами, включая возможность назначать роли в Azure RBAC. У вас должно быть не более трех владельцев подписок, чтобы снизить вероятность нарушения защиты от скомпрометированного владельца.

- В списке **роль** выберите роль **владелец** .

   ![Выбор роли владельца в области "Добавление назначения ролей"](./media/role-assignments-portal-subscription-admin/add-role-assignment-role-owner.png)

## <a name="step-4-select-who-needs-access"></a>Шаг 4. Выбор пользователей, которым требуется доступ

1. В списке **назначить доступ к** выберите **пользователь, группа или субъект-служба**.

1. В разделе **Выбор** найдите пользователя, введя строку или прокрутите список.

   ![Выбор пользователя в добавлении назначения ролей](./media/role-assignments-portal-subscription-admin/add-role-assignment-user-admin.png)

1. Найдя пользователя, щелкните его, чтобы выбрать.

## <a name="step-5-assign-role"></a>Шаг 5. Назначение роли

1. Чтобы назначить роль, нажмите кнопку **сохранить**.

   Через несколько секунд пользователю назначается роль в выбранной области.

1. На вкладке **назначения ролей** убедитесь, что в списке отображается назначение роли.

    ![Добавление назначения роли сохранено](./media/role-assignments-portal-subscription-admin/sub-role-assignments-owner.png)

## <a name="next-steps"></a>Дальнейшие действия

- [Назначение ролей Azure с помощью портала Azure](role-assignments-portal.md)
- [Вывод списка назначений ролей Azure с помощью портала Azure](role-assignments-list-portal.md).
- [Упорядочение ресурсов с помощью групп управления Azure](../governance/management-groups/overview.md)
