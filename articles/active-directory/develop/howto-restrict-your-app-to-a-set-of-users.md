---
title: Ограничить приложение Azure AD набором пользователей | Службы
titleSuffix: Microsoft identity platform
description: Сведения о предоставлении доступа к приложениям, зарегистрированным в AAD, только для выбранного набора пользователей.
services: active-directory
author: kalyankrishna1
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: how-to
ms.date: 09/24/2018
ms.author: kkrishna
ms.reviewer: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 71cfe08da42b8eec9ddbd0e4246ba1b72f895414
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103199598"
---
# <a name="how-to-restrict-your-azure-ad-app-to-a-set-of-users-in-an-azure-ad-tenant"></a>Как ограничить приложение Azure AD набором пользователей в клиенте Azure AD

Приложения, зарегистрированные в клиенте Azure Active Directory (Azure AD), по умолчанию являются доступными для всех пользователей клиента, которые успешно прошли проверку подлинности.

Аналогичным образом, в частности в случае [мультитенантного](howto-convert-app-to-be-multi-tenant.md) приложения, все пользователи в клиенте Azure AD, в котором подготовлено это приложение, будут иметь возможность доступа к этому приложению сразу после успешной проверки подлинности в своих соответствующих клиентах.

У администраторов и разработчиков клиента часто есть требования относительно ограничения доступа к приложению определенным набором пользователей. Разработчики могут делать то же самое, используя популярные шаблоны авторизации, такие как управление доступом на основе ролей в Azure (Azure RBAC), но этот подход требует значительного объема работы на стороне разработчика.

Администраторы и разработчики клиента могут ограничить приложение определенным набором пользователей или групп безопасности в клиенте с помощью этой встроенной функции Azure AD.

## <a name="supported-app-configurations"></a>Поддерживаемые конфигурации приложения

Функция ограничения приложения для определенного набора пользователей или групп безопасности в клиенте работает со следующими типами приложений:

- Приложения, настроенные для федеративного единого входа с проверкой подлинности на основе SAML.
- Приложения прокси приложения, использующие предварительную проверку подлинности Azure AD.
- Приложения, созданные непосредственно на платформе приложений Azure AD, использующих проверку подлинности OAuth 2.0 или OpenID Connect после того, как пользователь или администратор согласились с условиями использования этого приложения.

     > [!NOTE]
     > Эта функция доступна только для веб-приложений, веб-API и корпоративных приложений. Приложения, зарегистрированные как [собственные](./quickstart-register-app.md), нельзя ограничивать набором пользователей или групп безопасности в клиенте.

## <a name="update-the-app-to-enable-user-assignment"></a>Обновление приложения для включения назначения пользователя

Существует два способа создания приложения с включенным назначением пользователей. Одна из них требует роли **глобального администратора** , а вторая — нет.

### <a name="enterprise-applications-requires-the-global-administrator-role"></a>Корпоративные приложения (требуется роль глобального администратора)

1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a> как **глобальный администратор**.
1. Если у вас есть доступ к нескольким клиентам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать клиент, в котором следует зарегистрировать приложение.
1. Найдите и выберите **Azure Active Directory**.
1. В разделе **Управление** выберите **корпоративные приложения**  >  **все приложения**.
1. Выберите из списка приложение, которому требуется назначить пользователя или группу безопасности. 
    Используйте фильтры в верхней части окна, чтобы найти конкретное приложение.
1. На странице **Обзор** приложения в разделе **Управление** выберите **Свойства**.
1. Найдите параметр **Требуется ли назн. польз.?** и присвойте ему значение **Да**. Если этот параметр имеет значение **Да**, пользователи в клиенте сначала должны быть назначены этому приложению или не смогут входить в это приложение.
1. Щелкните **Сохранить**.

### <a name="app-registration"></a>Регистрация приложений

1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>.
1. Если у вас есть доступ к нескольким клиентам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать клиент, в котором следует зарегистрировать приложение.
1. Найдите и выберите **Azure Active Directory**.
1. В разделе **Управление** выберите **Регистрация приложений**.
1. Создайте или выберите приложение, которым требуется управлять. Необходимо быть **владельцем** этого приложения.
1. На странице **Обзор** приложения выберите ссылку **управляемое приложение в локальном каталоге** в разделе **основные компоненты** .
1. В разделе **Управление** выберите **Свойства**.
1. Найдите параметр **Требуется ли назн. польз.?** и присвойте ему значение **Да**. Если этот параметр имеет значение **Да**, пользователи в клиенте сначала должны быть назначены этому приложению или не смогут входить в это приложение.
1. Щелкните **Сохранить**.

## <a name="assign-users-and-groups-to-the-app"></a>Назначение пользователей и групп для приложения

После того как вы настроили приложение для включения назначения пользователя, вы можете продолжить и назначить приложению пользователей и группы.

1. В разделе **Управление** выберите **Пользователи и группы**  >  **Добавить пользователя или группу** .
1. Выберите селектор **пользователей** . 

     Отобразится список пользователей и групп безопасности, а также текстовое поле для поиска определенного пользователя или группы. На этом экране можно выбрать сразу несколько пользователей и групп.

1. Завершив выбор пользователей и групп, нажмите **кнопку Выбрать**.
1. Используемых Если в приложении определены роли приложения, можно использовать параметр **Выбор роли** , чтобы назначить выбранных пользователей и группы одной из ролей приложения. 
1. Выберите **назначить** , чтобы завершить назначение пользователей и групп приложению. 
1. Убедитесь, что добавленные пользователи и группы отображаются в обновленном списке **Пользователи и группы**.

## <a name="more-information"></a>Дополнительные сведения

- [Как добавить роли приложения в приложение](./howto-add-app-roles-in-azure-ad-apps.md)
- [Добавление авторизации с помощью ролей приложения и утверждения ролей в веб-приложение ASP.NET Core](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/5-WebApp-AuthZ/5-1-Roles)
- [August 9: Using application roles and security groups in your apps (Video)](https://www.youtube.com/watch?v=LRoc-na27l0) (9 августа. Использование ролей приложения и групп безопасности в приложениях (видео))
- [Azure Active Directory, now with Group Claims and Application Roles](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-Active-Directory-now-with-Group-Claims-and-Application/ba-p/243862) (Azure Active Directory с утверждениями групп и ролями приложения)
- [Манифест приложения Azure Active Directory](./reference-app-manifest.md)
