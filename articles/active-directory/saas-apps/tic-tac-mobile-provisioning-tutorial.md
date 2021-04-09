---
title: Руководство по настройке Tic-Tac Mobile для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить автоматическую подготовку и учетных записей пользователей и ее отмену из Azure AD в Tic-Tac Mobile.
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: d0f24e81-fecf-4e71-bd8a-ab911366fdf5
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 10/08/2020
ms.author: Zhchia
ms.openlocfilehash: 91ae51b9a2785dbc40c55fa58b26763916e8d16c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101644584"
---
# <a name="tutorial-configure-tic-tac-mobile-for-automatic-user-provisioning"></a>Руководство по настройке Tic-Tac Mobile для автоматической подготовки пользователей

В этом учебнике описаны действия, которые нужно выполнить в Tic-Tac Mobile и Azure Active Directory (Azure AD) для настройки автоматической подготовки пользователей. После настройки Azure AD автоматически осуществляет и отменяет подготовку пользователей и групп для [Tic-Tac Mobile](https://www.tictacmobile.com/) с помощью службы подготовки Azure AD. Сведения о возможностях и принципе работы этой службы, а также часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и их отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).


## <a name="capabilities-supported"></a>Поддерживаемые возможности

> [!div class="checklist"]
> * Создание пользователей в Tic-Tac Mobile.
> * Удаление из Tic-Tac Mobile пользователей, которым уже не нужен доступ.
> * Синхронизация атрибутов пользователей между Azure AD и Tic-Tac Mobile.

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* [Клиент Azure AD.](../develop/quickstart-create-new-tenant.md)
* Учетная запись пользователя в Azure AD с [разрешением](../roles/permissions-reference.md) на настройку подготовки. Подходят следующие роли: администратор приложений, администратор облачных приложений, владелец приложений или глобальный администратор.
* Учетная запись [Tic-Tac Mobile](https://www.tictacmobile.com/) с ролью суперадминистратора.


## <a name="step-1-plan-your-provisioning-deployment"></a>Шаг 1. Планирование развертывания для подготовки

1. Узнайте, [как работает служба подготовки](../app-provisioning/user-provisioning.md).
1. Определите, кто будет находиться в [области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
1. Определите, какие данные следует [сопоставлять между Azure AD и Tic-Tac Mobile](../app-provisioning/customize-application-attributes.md).

## <a name="step-2-configure-tic-tac-mobile-to-support-provisioning-with-azure-ad"></a>Шаг 2. Настройка Tic-Tac Mobile для поддержки подготовки с помощью Azure AD

Отправьте запрос по адресу support@tictacmobile.com, чтобы получить **URL-адрес клиента** и **секретный токен**. Для получения токена необходимо иметь роль суперадминистратора в Tic-Tac Mobile. Его нужно будет ввести в поле **Секретный токен** на вкладке **Подготовка** для приложения Tic-Tac Mobile на портале Azure.

## <a name="step-3-add-tic-tac-mobile-from-the-azure-ad-application-gallery"></a>Шаг 3. Добавление Tic-Tac Mobile из коллекции приложений Azure AD

Добавьте Tic-Tac Mobile из коллекции приложений Azure AD, чтобы начать управление подготовкой в Tic-Tac Mobile. Если вы уже настроили единый вход в Tic-Tac Mobile, можете использовать то же приложение. При начальном тестировании интеграции создайте отдельное приложение. Дополнительные сведения о добавлении приложения из коллекции см. в статье о [подготовке приложений на основе атрибутов с фильтрами области](../manage-apps/add-application-portal.md).

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Шаг 4. Определение пользователей для включения в область подготовки

Служба подготовки Azure AD позволяет определить пользователей, которые будут подготовлены, назначая их приложению и (или) присваивая атрибуты пользователям или группам. Если вы решили указать, кто именно будет подготовлен к работе в приложении, на основе назначения, выполните процедуру из статьи [Управление назначением пользователей для приложения в Azure Active Directory](../manage-apps/assign-user-or-group-access-portal.md), чтобы назначить пользователей и группы приложению. Если вы решили указать, кто именно будет подготовлен, на основе одних только атрибутов пользователя или группы, можете использовать фильтр области, как описано в статье [Подготовка приложений на основе атрибутов с использованием фильтров области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

* При назначении пользователей и групп для Tic-Tac Mobile необходимо выбрать роль, отличную от роли **Доступ по умолчанию**. Пользователи с такой ролью исключаются из подготовки и будут помечены в журналах подготовки как не имеющие разрешений. Если в приложении доступна только эта роль, можно [изменить манифест приложения](../develop/howto-add-app-roles-in-azure-ad-apps.md), чтобы добавить другие роли.
* Начните с малого. Протестируйте небольшой набор пользователей и групп, прежде чем выполнять развертывание для всех. Если в область подготовки включены назначенные пользователи и группы, для сохранения контроля назначьте приложению одного или двух пользователей либо одну или две группы. Если в область включены все пользователи и группы, можно указать [фильтр области на основе атрибутов](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

## <a name="step-5-configure-automatic-user-provisioning-to-tic-tac-mobile"></a>Шаг 5. Настройка автоматической подготовки пользователей для Tic-Tac Mobile

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей или групп в Clarizen One на основе их назначений в Azure AD.

### <a name="configure-automatic-user-provisioning-for-tic-tac-mobile-in-azure-ad"></a>Настройка автоматической подготовки пользователей для Tic-Tac Mobile в Azure AD

1. Войдите на [портал Azure](https://portal.azure.com). Щелкните **Корпоративные приложения** > **Все приложения**.

    ![Снимок экрана: панель "Корпоративные приложения".](common/enterprise-applications.png)

1. Из списка приложений выберите **Tic-Tac Mobile**.

    ![Снимок экрана, на котором показана ссылка на Tic-Tac Mobile в списке приложений.](common/all-applications.png)

1. Выберите вкладку **Подготовка**.

    ![Снимок экрана: вкладка "Подготовка".](common/provisioning.png)

1. Для параметра **Режим подготовки к работе** выберите значение **Автоматически**.

    ![Снимок экрана: вкладка "Подготовка", где выбран вариант "Автоматически".](common/provisioning-automatic.png)

1. В разделе **Учетные данные администратора** укажите **URL-адрес клиента** и **секретный токен** Tic-Tac Mobile. Щелкните **Проверить подключение** и убедитесь, что Azure AD может подключиться к Tic-Tac Mobile. Если установить подключение не удалось, убедитесь, что у учетной записи Tic-Tac Mobile есть разрешения администратора, и повторите попытку.

    ![Снимок экрана, на котором показано поле "Секретный токен".](common/provisioning-testconnection-tenanturltoken.png)

1. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки. Установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Снимок экрана, на котором показано поле "Почтовое уведомление".](common/provisioning-notification-email.png)

1. Нажмите кнопку **Сохранить**.

1. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Users to Tic-Tac Mobile** (Синхронизировать пользователей Azure Active Directory с Tic-Tac Mobile).

1. В разделе **Сопоставление атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы из Azure AD в Tic-Tac Mobile. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в Tic-Tac Mobile для операций обновления. Если вы измените [соответствующий целевой атрибут](../app-provisioning/customize-application-attributes.md), обязательно убедитесь, что API Tic-Tac Mobile поддерживает фильтрацию пользователей по этому атрибуту. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

   |attribute|Тип|
   |---|---|
   |userName|Строка|
   |name.givenName|Строка|
   |name.familyName|Строка|
   |externalId|Строка|
   |title|Строка|
   |emails[type eq "work"].value|Строка|
   |preferredLanguage|Строка|
   |externalId|Строка|
   |userType|Строка|
   |локаль|Строка|
   |timezone|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:employeeNumber|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:costCenter|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:organization|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:division|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|Строка|

1. Чтобы настроить фильтры области, ознакомьтесь с инструкциями в [этом учебнике](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

1. Чтобы включить службу подготовки Azure AD для Tic-Tac Mobile, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Снимок экрана, на котором показано состояние подготовки "Включено".](common/provisioning-toggle-on.png)

1. Определите пользователей или группы для подготовки в Tic-Tac Mobile, выбрав нужные значения в поле **Область** в разделе **Параметры**.

    ![Снимок экрана, на котором показана область подготовки.](common/provisioning-scope.png)

1. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Снимок экрана, на котором показано сохранение конфигурации подготовки.](common/provisioning-configuration-save.png)

После этого начнется цикл начальной синхронизации всех пользователей и групп, определенных в поле **Область** в разделе **Параметры**. Начальный цикл занимает больше времени, чем последующие циклы. Пока служба подготовки Azure AD запущена, они выполняются примерно каждые 40 минут.

## <a name="step-6-monitor-your-deployment"></a>Шаг 6. Мониторинг развертывания

Завершив настройку подготовки, используйте следующие ресурсы для мониторинга развертывания.

1. Используйте [журналы подготовки](../reports-monitoring/concept-provisioning-logs.md), чтобы определить, какие пользователи были подготовлены успешно или неудачно.
1. Используйте [индикатор выполнения](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md), чтобы узнать состояние цикла подготовки и близость его завершения.
1. Если конфигурация подготовки, вероятно, находится в неработоспособном состоянии, приложение перейдет в карантин. Дополнительные сведения о состояниях карантина см. в статье [Состояние подготовки приложений в карантине](../app-provisioning/application-provisioning-quarantine-status.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)