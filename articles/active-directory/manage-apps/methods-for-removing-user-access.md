---
title: Удаление доступа пользователя к приложению в Azure Active Directory
description: Узнайте, как удалить доступ пользователя к приложению в Azure Active Directory
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 11/02/2020
ms.author: kenwith
ms.openlocfilehash: e6a6c00811a7b87156802897db62a4a10130f130
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99257361"
---
# <a name="how-to-remove-a-users-access-to-an-application"></a>Удаление доступа пользователя к приложению

В этой статье представлены сведения об удалении доступа пользователя к приложению.

## <a name="i-want-to-remove-a-specific-users-or-groups-assignment-to-an-application"></a>Я хочу отменить доступ к приложению для определенного пользователя или группы

Чтобы удалить доступ к приложению, назначенный пользователю или группе, выполните действия, описанные в статье [Удаление назначения доступа к корпоративному приложению для пользователя или группы в предварительной версии Azure Active Directory](./assign-user-or-group-access-portal.md).

## <a name="i-want-to-disable-all-access-to-an-application-for-every-user"></a>Я хочу отключить доступ к приложению для всех пользователей

Чтобы отключить вход в приложение для всех пользователей, выполните действия, описанные в статье [Отключение входа пользователя в корпоративное приложение в предварительной версии Azure Active Directory](./disable-user-sign-in-portal.md).

## <a name="i-want-to-delete-an-application-entirely"></a>Я хочу полностью удалить приложение

В [серии примеров по управлению приложениями](delete-application-portal.md) содержатся рекомендации по удалению приложения из клиента Azure Active Directory.

## <a name="i-want-to-disable-all-future-user-consent-operations-to-any-application"></a>Я хочу отключить все будущие операции пользователя по предоставлению согласия для всех приложений

Если вы отключите для всего каталога возможность предоставлять согласие, пользователь не сможет согласиться с условиями использования приложения. Администраторы могут по-прежнему предоставлять согласие от имени пользователя. Дополнительные сведения о согласии в приложениях и возможных мотивах для его отключения или предоставления см. в разделе [Получение согласия пользователя и администратора](../develop/howto-convert-app-to-be-multi-tenant.md#understand-user-and-admin-consent). См. также [Типы разрешений и согласие](../develop/v2-permissions-and-consent.md).

Чтобы **отключить все будущие операции по предоставлению согласия пользователя во всем каталоге**, выполните следующие инструкции:

1.  Откройте [**портал Azure**](https://portal.azure.com/) и войдите в систему как **глобальный администратор**.

2.  Откройте **расширение Azure Active Directory**. 

3.  Щелкните **Корпоративные приложения** в меню навигации.

5.  Щелкните **Параметры пользователя**.

6.  Установите переключатель **Пользователи могут разрешать приложениям доступ к своим данным** на **нет** и нажмите кнопку "Сохранить".


## <a name="next-steps"></a>Дальнейшие действия

[Управление доступом к приложениям](what-is-access-management.md)