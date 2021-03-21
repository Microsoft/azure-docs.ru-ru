---
title: Использование настраиваемых ролей Azure в PIM — Azure AD | Документация Майкрософт
description: Узнайте, как использовать пользовательские роли Azure в Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: pim
ms.date: 11/08/2019
ms.author: curtand
ms.collection: M365-identity-device-management
ms.openlocfilehash: 24b7845ec66a85e6ced4f1df9caec409a94016bf
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88782606"
---
# <a name="use-azure-custom-roles-in-privileged-identity-management"></a>Использование пользовательских ролей Azure в управление привилегированными пользователями

Вам может потребоваться применить параметры управление привилегированными пользователями (PIM) к некоторым пользователям привилегированной роли в Организации Azure Active Directory (Azure AD), обеспечивая более широкие автономность для других пользователей. Рассмотрим пример сценария, в котором ваша организация нанимает несколько контрактов, чтобы помочь в разработке приложения, которое будет выполняться в подписке Azure.

Вам как администратору ресурсов было бы удобно, чтобы сотрудники получали доступ без обязательных подтверждений. Тем не менее, все они должны получать подтверждение при запросе на доступ к ресурсам организации.

Выполните действия, описанные в следующем разделе, чтобы настроить целевые параметры управление привилегированными пользователями для ролей ресурсов Azure.

## <a name="create-the-custom-role"></a>Создание настраиваемой роли

Чтобы создать настраиваемую роль для ресурса, выполните действия, описанные в разделе [пользовательские роли Azure](../../role-based-access-control/custom-roles.md).

После создания настраиваемой роли присвойте ей описательное имя, чтобы легко определять, какие из встроенных ролей нужно дублировать.

> [!NOTE]
> Убедитесь, что настраиваемая роль является дубликатом встроенной роли, которую вы хотите дублировать, и что ее область действия соответствует встроенной роли.

## <a name="apply-pim-settings"></a>Применение параметров PIM

После создания роли в вашей организации Azure AD перейдите на страницу **Управление привилегированными пользователями (ресурсы Azure)** в портал Azure. Выберите ресурс, к которому применяется роль.

![Панель "Управление привилегированными пользователями — ресурсы Azure"](media/pim-resource-roles-custom-role-policy/aadpim-manage-azure-resource-some-there.png)

[Настройте параметры роли Управление привилегированными пользователями](pim-resource-roles-configure-role-settings.md) , которые должны применяться к этим членам роли.

Наконец, [назначьте роли](pim-resource-roles-assign-roles.md) определенной группе участников, для которых вы хотите применить настроенные параметры.

## <a name="next-steps"></a>Дальнейшие действия

- [Настройка параметров роли ресурсов Azure в управление привилегированными пользователями](pim-resource-roles-configure-role-settings.md)
- [Пользовательские роли в Azure](../../role-based-access-control/custom-roles.md)