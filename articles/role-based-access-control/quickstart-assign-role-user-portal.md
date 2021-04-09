---
title: Руководство по Предоставление доступа пользователям к ресурсам Azure с помощью портала Azure — Azure RBAC
description: Узнайте, как предоставить пользователям доступ к ресурсам Azure с помощью портала Azure и управления доступом на основе ролей Azure (Azure RBAC).
services: role-based-access-control
documentationCenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: role-based-access-control
ms.devlang: ''
ms.topic: tutorial
ms.tgt_pltfrm: ''
ms.workload: identity
ms.date: 02/22/2019
ms.author: rolyon
ms.openlocfilehash: fcba9cad208c2ac170f91cc06a6db22e271f2a70
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100559312"
---
# <a name="tutorial-grant-a-user-access-to-azure-resources-using-the-azure-portal"></a>Руководство по Предоставление доступа пользователям к ресурсам Azure с помощью портала Azure

[Управление доступом на основе ролей Azure (Azure RBAC)](overview.md) — это способ управления доступом к ресурсам в Azure. В этом руководстве объясняется, как предоставить пользователю доступ к созданию и администрированию виртуальных машин в группе ресурсов.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Предоставление доступа пользователю в области действия группы ресурсов
> * Запрет доступа

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите на портал Azure по адресу https://portal.azure.com.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

1. В списке переходов щелкните **Группы ресурсов**.

1. Чтобы открыть колонку **Группа ресурсов**, щелкните **Добавить**.

   ![Добавление новой группы ресурсов](./media/quickstart-assign-role-user-portal/resource-group.png)

1. Для **имени группы ресурсов** введите **rbac-resource-group**.

1. Выберите подписку и расположение группы ресурсов.

1. Щелкните **Создать** , чтобы создать группу ресурсов.

1. Чтобы обновить список групп ресурсов, щелкните **Обновить**.

   Новая группа ресурсов появится в списке групп ресурсов.

   ![Список групп ресурсов](./media/quickstart-assign-role-user-portal/resource-group-list.png)

## <a name="grant-access"></a>Предоставление доступа

Чтобы предоставить доступ в Azure RBAC, нужно назначить роль Azure.

1. В списке **Группы ресурсов** щелкните новую группу ресурсов **rbac-resource-group**.

1. Щелкните **Управление доступом (IAM)** .

1. Щелкните вкладку **Назначения ролей**, чтобы просмотреть выбранный список назначения ролей.

   ![Колонка "Управление доступом (IAM)" для группы ресурсов](./media/quickstart-assign-role-user-portal/access-control.png)

1. Щелкните **Добавить** > **Добавить назначение ролей**, чтобы открыть панель Добавить назначение ролей.

   Если у вас нет прав назначать роли, функция "Добавить назначение роли" будет неактивна.

   ![Меню "Добавить назначение ролей"](./media/shared/add-role-assignment-menu.png)

    Откроется панель "Добавить назначение ролей".

   ![Область "Добавить назначение ролей"](./media/quickstart-assign-role-user-portal/add-role-assignment.png)

1. В раскрывающемся списке **Роль** выберите **Участник виртуальных машин**.

1. В списке **Выбор** выберите себя или другого пользователя.

1. Чтобы присвоить роль, щелкните **Сохранить**.

   Через несколько секунд пользователю будет назначена роль "Участник виртуальной машины" в пределах группы ресурсов rbac-resource-group.

   ![Назначение роли "Участник виртуальных машин"](./media/quickstart-assign-role-user-portal/vm-contributor-assignment.png)

## <a name="remove-access"></a>Запрет доступа

При использовании Azure RBAC для удаления доступа нужно удалить назначение роли.

1. В списке назначения ролей установите флажок рядом с пользователем с ролью "Участник виртуальной машины".

1. Щелкните **Удалить**.

   ![Сообщение об удалении назначения роли](./media/quickstart-assign-role-user-portal/remove-role-assignment.png)

1. В появившемся сообщении об отзыве назначения роли щелкните **Да**.

## <a name="clean-up"></a>Очистка

1. В списке переходов щелкните **Группы ресурсов**.

1. Чтобы открыть группу ресурсов, щелкните **rbac-resource-group**.

1. Чтобы удалить группу ресурсов, щелкните **Удалить группу ресурсов**.

   ![Удалить группу ресурсов](./media/quickstart-assign-role-user-portal/delete-resource-group.png)

1. В колонке **Вы действительно хотите удалить** введите имя группы ресурсов: **rbac-resource-group**.

1. Чтобы удалить группу ресурсов, щелкните **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Руководство. Предоставление доступа пользователям к ресурсам Azure с помощью Azure PowerShell](tutorial-role-assignments-user-powershell.md)
