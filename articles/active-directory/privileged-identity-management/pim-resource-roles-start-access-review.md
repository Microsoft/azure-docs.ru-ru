---
title: Создание проверки доступа для ролей ресурсов Azure в PIM — Azure AD | Документация Майкрософт
description: Узнайте, как создать проверку доступа к ролям ресурсов Azure в Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: pim
ms.date: 03/09/2021
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4276b48584ecbad91794de58abafd7e3367f6877
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102564039"
---
# <a name="create-an-access-review-of-azure-resource-roles-in-privileged-identity-management"></a>Создание проверки доступа для ролей ресурсов Azure в управление привилегированными пользователями

Потребность в доступе к привилегированным ролям ресурсов Azure сотрудниками изменяется с течением времени. Чтобы снизить риск, связанный с устаревшими назначениями ролей, следует регулярно проверять доступ. Вы можете использовать Azure Active Directory (Azure AD) управление привилегированными пользователями (PIM) для создания проверок доступа для привилегированного доступа к ролям ресурсов Azure. Также можно настроить автоматические проверки доступа, которые выполняются автоматически. В этой статье описывается, как создать одну или несколько проверок доступа.

## <a name="prerequisite-role"></a>Предварительная роль

 Для создания проверок доступа необходимо назначить роль Azure " [владелец](../../role-based-access-control/built-in-roles.md#owner) " или " [администратор доступа пользователей](../../role-based-access-control/built-in-roles.md#user-access-administrator) " для ресурса.

## <a name="open-access-reviews"></a>Открытие проверок доступа

1. Войдите в [портал Azure](https://portal.azure.com/) с помощью пользователя, назначенного одной из обязательных ролей.

1. Откройте **Azure AD privileged Identity Management**.

1. В меню слева выберите **ресурсы Azure**.

1. Выберите ресурс, которым вы хотите управлять, например подписку.

1. В разделе Управление выберите **Проверка доступа**.

    ![Ресурсы Azure — список проверок доступа, отображающий состояние всех проверок](./media/pim-resource-roles-start-access-review/access-reviews.png)

[!INCLUDE [Privileged Identity Management access reviews](../../../includes/active-directory-privileged-identity-management-access-reviews.md)]

## <a name="start-the-access-review"></a>Запуск проверки доступа

Завершив настройку параметров для проверки доступа, щелкните **Запустить**. Проверка доступа будет отображаться в списке с индикатором состояния.

![Список проверок доступа, отображающий состояние начатой проверки](./media/pim-resource-roles-start-access-review/access-reviews-list.png)

По умолчанию Azure AD отправляет электронные сообщения рецензентам вскоре после запуска проверки. Если вы выбрали не отправлять сообщения через службу Azure AD, обязательно сообщите рецензентам, что им необходимо выполнить проверку доступа. Вы можете просмотреть инструкции по [просмотру доступа к ролям ресурсов Azure](pim-resource-roles-perform-access-review.md).

## <a name="manage-the-access-review"></a>Управление проверкой доступа

Ход выполнения проверок можно отслеживать на странице **Обзор** проверки доступа. Права доступа не изменяются в каталоге до [завершения проверки](pim-resource-roles-complete-access-review.md).

![Страница "Обзор проверок доступа" с подробными сведениями о проверке](./media/pim-resource-roles-start-access-review/access-review-overview.png)

Если это одноразовая проверка, то после выполнения периода проверки доступа или после остановки проверки доступа администратор должен выполнить действия, описанные в статье [Завершение проверки доступа ролей ресурсов Azure](pim-resource-roles-complete-access-review.md) для просмотра и применения результатов.  

Чтобы управлять серией проверок доступа, перейдите к проверке доступа, и вы увидите предстоящие вхождения в плановых проверках и измените дату окончания или добавьте или удалите проверяющих.

В зависимости от сделанного выбора в **параметрах завершения**, автоматическое применение будет выполняться после даты окончания проверки или при ручной отмене проверки. Состояние проверки будет изменено с **завершено** на промежуточные состояния, такие как **применение** и, наконец, к **примененному** состоянию. Вы должны видеть запрещенных пользователей (если они есть), которые удаляются из ролей через несколько минут.

## <a name="next-steps"></a>Дальнейшие действия

- [Проверка доступа к ролям ресурсов Azure](pim-resource-roles-perform-access-review.md)
- [Выполнение проверки доступа к ролям ресурсов Azure](pim-resource-roles-complete-access-review.md)
- [Создание проверки доступа для ролей Azure AD](pim-how-to-start-security-review.md)
