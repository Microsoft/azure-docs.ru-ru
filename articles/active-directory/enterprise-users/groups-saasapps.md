---
title: Использование группы для управления доступом к приложениям SaaS в Azure AD | Документация Майкрософт
description: Использование групп в Azure Active Directory для предоставления доступа к приложениям SaaS, интегрированным с Azure Active Directory.
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.subservice: enterprise-users
ms.workload: identity
ms.topic: how-to
ms.date: 12/02/2020
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: df36cd334dbb455cf1717bf18fc6c8337d0ee6d2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96547888"
---
# <a name="using-a-group-to-manage-access-to-saas-applications"></a>Использование группы для управления доступом к приложениям SaaS

Используя Azure Active Directory (Azure AD) с Azure AD Premiumным планом лицензирования, вы можете использовать группы для назначения доступа к приложению SaaS, интегрированному с Azure AD. Например, если вы хотите предоставить отделу маркетинга доступ к пяти разным приложениям SaaS, можно создать группу, содержащую пользователей из отдела маркетинга, и затем предоставить этой группе доступ к пяти приложениям SaaS, необходимым этому отделу. Таким образом можно сэкономить время, управляя членством пользователей отдела маркетинга в одном месте. Пользователи приложения назначаются при добавлении в группу маркетинга, и их назначения будут удалены из приложения, когда они будут удалены из группы. Эту возможность можно использовать для сотен приложений, которые можно добавлять из коллекции приложений Azure AD.

> [!IMPORTANT]
> Эту функцию можно использовать только после запуска Azure AD Premium пробной или приобретенной Azure AD Premium плане лицензии.
> Назначение на основе групп поддерживается только для групп безопасности.
> Сейчас членство во вложенных группах не поддерживается в рамках назначения приложений на основе групп.

## <a name="to-assign-access-for-a-user-or-group-to-a-saas-application"></a>Назначение пользователю или группе доступа к приложению SaaS

1. В [центре администрирования Azure AD](https://aad.portal.azure.com) выберите **Корпоративные приложения**.
2. Выберите приложение, которое добавили из коллекции приложений, чтобы открыть его.
3. Выберите **Пользователи и группы**, затем — **Добавить пользователя**.
4. На странице **Добавление назначения** выберите **Пользователи и группы**, чтобы открыть список выбора **Пользователи и группы**.
6. Выберите любое количество групп или пользователей, затем щелкните или коснитесь **Выбрать**, чтобы добавить их в список **Добавление назначения**. На этом этапе можно также назначить роль пользователю.
7. Щелкните **Назначить**, чтобы назначить пользователей или группы для выбранного корпоративного приложения.

## <a name="next-steps"></a>Дальнейшие действия
В следующих статьях содержатся дополнительные сведения об Azure Active Directory.

* [Управление доступом к ресурсам с помощью групп Azure Active Directory](../fundamentals/active-directory-manage-groups.md)
* [Управление приложениями в Azure Active Directory](../manage-apps/what-is-application-management.md)
* [Настройка параметров групп с помощью командлетов Azure Active Directory](../enterprise-users/groups-settings-cmdlets.md)
* [Что такое Azure Active Directory?](../fundamentals/active-directory-whatis.md)
* [Интеграция локальных удостоверений с Azure Active Directory](../hybrid/whatis-hybrid-identity.md)
