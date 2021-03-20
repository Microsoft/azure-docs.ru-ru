---
title: Распространенные сценарии в управлении назначениями — Azure AD
description: Ознакомьтесь с общими действиями, которые следует выполнить для распространенных сценариев в Azure Active Directory управлении обслуживанием.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: mwahl
ms.collection: M365-identity-device-management
ms.openlocfilehash: c0f45a1481661aa350815560d795ab7411f99545
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92317842"
---
# <a name="common-scenarios-in-azure-ad-entitlement-management"></a>Распространенные сценарии в управлении назначениями Azure AD

Существует несколько способов настройки управления правами для организации. Тем не менее, если вы только начинаете работу, полезно понимать распространенные сценарии для администраторов, владельцев каталогов, менеджеров пакетов доступа, утверждающих и запрашивающих лиц.

## <a name="delegate"></a>Делегат

### <a name="administrator-delegate-management-of-resources"></a>Администратор: Делегирование управления ресурсами

1. [Просмотр видео: делегирование из ИТ в менеджер отдела](https://www.microsoft.com/videoplayer/embed/RE3Lq00)
1. [Делегирование пользователей в роль создателя каталога](entitlement-management-delegate-catalog.md)

### <a name="catalog-creator-delegate-management-of-resources"></a>Создатель каталога: Делегирование управления ресурсами

- [Создать новый каталог](entitlement-management-catalog-create.md#create-a-catalog)

### <a name="catalog-owner-delegate-management-of-resources"></a>Владелец каталога: Делегирование управления ресурсами

1. [Добавление совладельцев в каталог](entitlement-management-catalog-create.md#add-additional-catalog-owners)
1. [Добавление ресурсов в каталог](entitlement-management-catalog-create.md#add-resources-to-a-catalog)

### <a name="catalog-owner-delegate-management-of-access-packages"></a>Владелец каталога: Делегирование управления пакетами доступа

1. [Просмотр видео: делегирование владельца каталога для доступа к диспетчеру пакетов](https://www.microsoft.com/videoplayer/embed/RE3Lq08)
1. [Делегирование пользователям доступа к роли диспетчера пакетов](entitlement-management-delegate-managers.md)

## <a name="govern-access-for-users-in-your-organization"></a>Управление доступом для пользователей в организации

### <a name="access-package-manager-allow-employees-in-your-organization-to-request-access-to-resources"></a>Доступ к диспетчеру пакетов: разрешить сотрудникам в вашей организации запрашивать доступ к ресурсам

1. [Создание пакета для доступа](entitlement-management-access-package-create.md#start-new-access-package)
1. [Добавление групп, команд, приложений или сайтов SharePoint для доступа к пакету](entitlement-management-access-package-create.md#resource-roles)
1. [Добавьте политику запросов, чтобы разрешить пользователям в вашем каталоге запрашивать доступ.](entitlement-management-access-package-create.md#for-users-in-your-directory)
1. [Укажите параметры срока действия](entitlement-management-access-package-create.md#lifecycle)

### <a name="requestor-request-access-to-resources"></a>Запрашивающая стороны: запрос доступа к ресурсам

1. [Войдите на портал доступа](entitlement-management-request-access.md#sign-in-to-the-my-access-portal)
1. Найти пакет Access
1. [Запрос доступа](entitlement-management-request-access.md#request-an-access-package)

### <a name="approver-approve-requests-to-resources"></a>Утверждающий: утвердить запросы к ресурсам

1. [Открыть запрос на портале доступа](entitlement-management-request-approve.md#open-request)
1. [Утверждение или отклонение запроса на доступ](entitlement-management-request-approve.md#approve-or-deny-request)

### <a name="requestor-view-the-resources-you-already-have-access-to"></a>Запрашивающая стороны: Просмотр ресурсов, к которым у вас уже есть доступ

1. [Войдите на портал доступа](entitlement-management-request-access.md#sign-in-to-the-my-access-portal)
1. Просмотр пакетов активного доступа

## <a name="govern-access-for-users-outside-your-organization"></a>Управление доступом для пользователей в организации

### <a name="administrator-collaborate-with-an-external-partner-organization"></a>Администратор: совместная работа с внешней партнерской организацией

1. [Узнайте, как работает доступ для внешних пользователей](entitlement-management-external-users.md#how-access-works-for-external-users)
1. [Проверка параметров внешних пользователей](entitlement-management-external-users.md#settings-for-external-users)
1. [Добавление подключения к внешней организации](entitlement-management-organization.md)

### <a name="access-package-manager-collaborate-with-an-external-partner-organization"></a>Доступ к диспетчеру пакетов. Совместная работа с внешней организацией партнера

1. [Создание пакета для доступа](entitlement-management-access-package-create.md#start-new-access-package)
1. [Добавление групп, команд, приложений или сайтов SharePoint для доступа к пакету](entitlement-management-access-package-resources.md#add-resource-roles)
1. [Добавьте политику запросов, чтобы разрешить пользователям, которые не находятся в вашем каталоге, запрашивать доступ.](entitlement-management-access-package-request-policy.md#for-users-not-in-your-directory)
1. [Укажите параметры срока действия](entitlement-management-access-package-create.md#lifecycle)
1. [Копирование ссылки для запроса пакета Access](entitlement-management-access-package-settings.md)
1. Отправьте ссылку на партнера по связи с внешним партнером, чтобы поделиться ими с пользователями

### <a name="requestor-request-access-to-resources-as-an-external-user"></a>Запрашивающий объект. запрос доступа к ресурсам как внешним пользователям

1. Найдите ссылку на пакет Access, полученный от вашего контакта
1. [Войдите на портал доступа](entitlement-management-request-access.md#sign-in-to-the-my-access-portal)
1. [Запрос доступа](entitlement-management-request-access.md#request-an-access-package)

### <a name="approver-approve-requests-to-resources"></a>Утверждающий: утвердить запросы к ресурсам

1. [Открыть запрос на портале доступа](entitlement-management-request-approve.md#open-request)
1. [Утверждение или отклонение запроса на доступ](entitlement-management-request-approve.md#approve-or-deny-request)

### <a name="requestor-view-the-resources-your-already-have-access-to"></a>Запрашивающая стороны: Просмотр ресурсов, к которым у вас уже есть доступ

1. [Войдите на портал доступа](entitlement-management-request-access.md#sign-in-to-the-my-access-portal)
1. Просмотр пакетов активного доступа

## <a name="day-to-day-management"></a>Повседневное управление

### <a name="access-package-manager-update-the-resources-for-a-project"></a>Доступ к диспетчеру пакетов: обновление ресурсов для проекта

1. [Просмотр видео: Управление повседневными задачами. изменения](https://www.microsoft.com/videoplayer/embed/RE3LD4Z)
1. Открытие пакета Access
1. [Добавление или удаление групп, команд, приложений или сайтов SharePoint](entitlement-management-access-package-resources.md#add-resource-roles)

### <a name="access-package-manager-update-the-duration-for-a-project"></a>Доступ к диспетчеру пакетов: обновление длительности проекта

1. [Просмотр видео: Управление повседневными задачами. изменения](https://www.microsoft.com/videoplayer/embed/RE3LD4Z)
1. Открытие пакета Access
1. [Открытие параметров жизненного цикла](entitlement-management-access-package-lifecycle-policy.md#open-lifecycle-settings)
1. [Обновление параметров срока действия](entitlement-management-access-package-lifecycle-policy.md#lifecycle) 

### <a name="access-package-manager-update-how-access-is-approved-for-a-project"></a>Доступ к диспетчеру пакетов: обновление порядка утверждения доступа для проекта

1. [Просмотр видео: Управление повседневными задачами. изменения](https://www.microsoft.com/videoplayer/embed/RE3LD4Z)
1. [Открытие существующей политики параметров запроса](entitlement-management-access-package-request-policy.md#open-an-existing-access-package-and-add-a-new-policy-of-request-settings)
1. [Обновление параметров утверждения](entitlement-management-access-package-approval-policy.md#change-approval-settings-of-an-existing-access-package)

### <a name="access-package-manager-update-the-people-for-a-project"></a>Доступ к диспетчеру пакетов: Обновление пользователей для проекта

1. [Просмотр видео: Управление повседневными задачами. изменения](https://www.microsoft.com/videoplayer/embed/RE3LD4Z)
1. [Удаление пользователей, которым больше не нужен доступ](entitlement-management-access-package-assignments.md)
1. [Открытие существующей политики параметров запроса](entitlement-management-access-package-request-policy.md#open-an-existing-access-package-and-add-a-new-policy-of-request-settings)
1. [Добавление пользователей, которым требуется доступ](entitlement-management-access-package-request-policy.md#for-users-in-your-directory)

### <a name="access-package-manager-directly-assign-specific-users-to-an-access-package"></a>Доступ к диспетчеру пакетов: непосредственно назначить конкретных пользователей для пакета Access

1. [Если пользователям требуются другие параметры жизненного цикла, добавьте новую политику в пакет Access.](entitlement-management-access-package-request-policy.md#open-an-existing-access-package-and-add-a-new-policy-of-request-settings)
1. [Прямое назначение конкретных пользователей пакету доступа](entitlement-management-access-package-assignments.md#directly-assign-a-user)

## <a name="assignments-and-reports"></a>Назначения и отчеты

### <a name="administrator-view-who-has-assignments-to-an-access-package"></a>Администратор: Просмотр пользователей, имеющих назначения для пакета Access

1. Открытие пакета Access
1. [Просмотр назначений](entitlement-management-access-package-assignments.md#view-who-has-an-assignment)
1. [Архивация отчетов и журналов](entitlement-management-logs-and-reporting.md)

### <a name="administrator-view-resources-assigned-to-users"></a>Администратор: Просмотр ресурсов, назначенных пользователям

1. [Просмотр пакетов доступа для пользователя](entitlement-management-reports.md#view-access-packages-for-a-user)
1. [Просмотр назначений ресурсов для пользователя](entitlement-management-reports.md#view-resource-assignments-for-a-user)

## <a name="programmatic-administration"></a>Программное администрирование

Вы также можете управлять пакетами доступа, каталогами, политиками, запросами и назначениями с помощью Microsoft Graph.  Пользователь в соответствующей роли с приложением с делегированным `EntitlementManagement.ReadWrite.All` допуском может вызывать [API управления назначением](/graph/tutorial-access-package-api?view=graph-rest-beta).

## <a name="next-steps"></a>Дальнейшие действия

- [Делегирование и роли](entitlement-management-delegate.md)
- [Обработка запросов и уведомления по электронной почте](entitlement-management-process.md)