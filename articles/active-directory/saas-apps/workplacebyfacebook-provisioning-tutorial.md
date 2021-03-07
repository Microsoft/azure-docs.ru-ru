---
title: Руководство по настройке Workplace by Facebook для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, какие действия необходимо выполнить в Workplace by Facebook и Azure Active Directory (Azure AD) для настройки автоматической подготовки пользователей.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/28/2020
ms.author: jeedes
ms.openlocfilehash: 98a151c9f740c3ab2f1471f98c7fab83cc848a28
ms.sourcegitcommit: 5bbc00673bd5b86b1ab2b7a31a4b4b066087e8ed
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102438410"
---
# <a name="tutorial-configure-workplace-by-facebook-for-automatic-user-provisioning"></a>Руководство по настройке Workplace by Facebook для автоматической подготовки пользователей

В этом учебнике описаны действия, которые необходимо выполнить в Workplace by Facebook и Azure Active Directory (Azure AD) для настройки автоматической подготовки пользователей. После настройки Azure AD автоматически осуществляет и отменяет подготовку пользователей и групп для [Workplace by Facebook](https://work.workplace.com/) с помощью службы подготовки Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="capabilities-supported"></a>Поддерживаемые возможности
> [!div class="checklist"]
> * Создание пользователей в Workplace by Facebook.
> * Удаление пользователей в Workplace by Facebook, когда доступ им больше не нужен.
> * Синхронизация атрибутов пользователей между Azure AD и Workplace by Facebook.
> * [Единый вход](./workplacebyfacebook-tutorial.md) в Workplace by Facebook (рекомендуется).

>[!VIDEO https://www.youtube.com/embed/oF7I0jjCfrY]

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* [Клиент Azure AD.](../develop/quickstart-create-new-tenant.md) 
* Учетная запись пользователя в Azure AD с [разрешением](../roles/permissions-reference.md) на настройку подготовки (например, администратор приложений, администратор облачных приложений, владелец приложения или глобальный администратор).
* подписка Workplace by Facebook с поддержкой единого входа.

> [!NOTE]
> Мы не рекомендуем использовать рабочую среду для проверки действий в этом учебнике.

> [!NOTE]
> Эту интеграцию также можно использовать в облачной среде Azure AD для государственных организаций США. Это приложение можно найти в коллекции облачных приложений с поддержкой Azure AD для государственных организаций США и настроить таким же образом, как и в общедоступном облаке.

При проверке действий в этом учебнике соблюдайте следующие рекомендации:

- Не используйте рабочую среду без необходимости.
- Если у вас нет пробной среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/).

## <a name="step-1-plan-your-provisioning-deployment"></a>Шаг 1. Планирование развертывания для подготовки
1. Узнайте, [как работает служба подготовки](../app-provisioning/user-provisioning.md).
2. Определите, кто будет находиться в [области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Определите, какие данные следует [сопоставлять между Azure AD и Workplace by Facebook](../app-provisioning/customize-application-attributes.md).

## <a name="step-2-configure-workplace-by-facebook-to-support-provisioning-with-azure-ad"></a>Шаг 2. Настройка Workplace by Facebook для поддержки подготовки с помощью Azure AD

Перед настройкой и включением службы подготовки необходимо решить, какие пользователи или группы в Azure AD представляют пользователей, которым требуется доступ к приложению Workplace by Facebook. Приняв это решение, можно будет назначить этих пользователей для приложения Workplace by Facebook, выполнив следующие действия.

*   Рекомендуется назначить одного пользователя Azure AD в Workplace by Facebook для тестирования конфигурации подготовки. Дополнительные пользователи и/или группы можно назначить позднее.

*   При назначении пользователя Workplace by Facebook необходимо выбрать допустимую роль пользователя. Роль "Доступ по умолчанию" не работает для подготовки.

## <a name="step-3-add-workplace-by-facebook-from-the-azure-ad-application-gallery"></a>Шаг 3. Добавление Workplace by Facebook из коллекции приложений Azure AD

Добавьте Workplace by Facebook из коллекции приложений Azure AD, чтобы начать управление подготовкой в Workplace by Facebook. Если вы ранее настроили Workplace by Facebook для единого входа, можете использовать то же приложение. Однако при первоначальном тестировании интеграции рекомендуется создать отдельное приложение. Дополнительные сведения о добавлении приложения из коллекции см. [здесь](../manage-apps/add-application-portal.md).

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Шаг 4. Определение пользователей для включения в область подготовки 

Служба подготовки Azure AD позволяет определить пользователей, которые будут подготовлены, на основе назначения приложению и (или) атрибутов пользователя или группы. Если вы решили определить пользователей на основе назначения, выполните следующие [действия](../manage-apps/assign-user-or-group-access-portal.md), чтобы назначить пользователей и группы приложению. Если вы решили указать, кто именно будет подготовлен, на основе одних только атрибутов пользователя или группы, можете использовать фильтр задания области, как описано [здесь](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* При назначении пользователей и групп для Workplace by Facebook необходимо выбрать роль, отличную от роли **Доступ по умолчанию**. Пользователи с ролью "Доступ по умолчанию" исключаются из подготовки и будут помечены в журналах подготовки как не назначенные явно. Кроме того, если эта роль является единственной, доступной в приложении, можно [изменить манифест приложения](../develop/howto-add-app-roles-in-azure-ad-apps.md), чтобы добавить дополнительные роли. 

* Начните с малого. Протестируйте небольшой набор пользователей и групп, прежде чем выполнять развертывание для всех. Если в область подготовки включены назначенные пользователи и группы, проверьте этот механизм, назначив приложению одного или двух пользователей либо одну или две группы. Если в область включены все пользователи и группы, можно указать [фильтр области на основе атрибутов](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Workplace by Facebook**.

    ![Ссылка на Workplace by Facebook в списке "Приложения"](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** щелкните **Авторизовать**. Вы будете перенаправлены на страницу авторизации Workplace by Facebook. Введите имя пользователя Workplace by Facebook и нажмите кнопку **Continue** (Продолжить). Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к Workplace by Facebook. Если подключение отсутствует, убедитесь, что у учетной записи Workplace by Facebook есть разрешения администратора и повторите попытку.

    ![Снимок экрана, на котором показано диалоговое окно ввода учетных данных администратора с возможностью авторизации.](./media/workplacebyfacebook-provisioning-tutorial/provisioning.png)

    ![Авторизация](./media/workplacebyfacebook-provisioning-tutorial/workplacelogin.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Щелкните **Сохранить**.

8. В разделе **Сопоставления** выберите **Синхронизировать пользователей Azure Active Directory с Workplace by Facebook**.

9. В разделе **Сопоставления атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы из Azure AD в Workplace by Facebook. Атрибуты, выбранные как свойства **Matching**, используются для сопоставления учетных записей пользователей в Workplace by Facebook для операций обновления. Если вы решили изменить [целевой атрибут сопоставления](../app-provisioning/customize-application-attributes.md), сначала убедитесь, что API Workplace by Facebook поддерживает фильтрацию пользователей по этому атрибуту. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

   |attribute|Тип|
   |---|---|
   |userName|Строка|
   |displayName|Строка|
   |active|Логическое|
   |title|логический|
   |emails[type eq "work"].value|Строка|
   |name.givenName|Строка|
   |name.familyName|Строка|
   |name.formatted|Строка|
   |addresses[type eq "work"].formatted|Строка|
   |addresses[type eq "work"].streetAddress|Строка|
   |addresses[type eq "work"].locality|Строка|
   |addresses[type eq "work"].region|Строка|
   |addresses[type eq "work"].country|Строка|
   |addresses[type eq "work"].postalCode|Строка|
   |addresses[type eq "other"].formatted|Строка|
   |phoneNumbers[type eq "work"].value|Строка|
   |phoneNumbers[type eq "mobile"].value|Строка|
   |phoneNumbers[type eq "fax"].value|Строка|
   |externalId|Строка|
   |preferredLanguage|Строка|
   |urn:scim:schemas:extension:enterprise:1.0.manager|Строка|
   |urn:scim:schemas:extension:enterprise:1.0.department|Строка|
   |urn:scim:schemas:extension:enterprise:1.0.division|Строка|
   |urn:scim:schemas:extension:enterprise:1.0.organization|Строка|
   |urn:scim:schemas:extension:enterprise:1.0.costCenter|Строка|
   |urn:scim:schemas:extension:enterprise:1.0.employeeNumber|Строка|
   |urn:scim:schemas:extension:facebook:auth_method:1.0:auth_method|Строка|
   |urn:scim:schemas:extension:facebook:frontline:1.0.is_frontline|логический|
   |urn:scim:schemas:extension:facebook:starttermdates:1.0.startDate|Целое число|


10. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Чтобы включить службу подготовки Azure AD для Workplace by Facebook, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Настройки**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

12. Определите пользователей и (или) группы для подготовки в Workplace by Facebook, выбрав нужные значения в поле **Область** в разделе **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

13. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется цикл начальной синхронизации всех пользователей и групп, определенных в поле **Область** в разделе **Параметры**. Начальный цикл занимает больше времени, чем последующие циклы. Пока служба подготовки Azure AD запущена, они выполняются примерно каждые 40 минут. 

## <a name="step-6-monitor-your-deployment"></a>Шаг 6. Мониторинг развертывания
После настройки подготовки используйте следующие ресурсы для мониторинга развертывания.

1. Используйте [журналы подготовки](../reports-monitoring/concept-provisioning-logs.md), чтобы определить, какие пользователи были подготовлены успешно или неудачно.
2. Используйте [индикатор выполнения](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md), чтобы узнать состояние цикла подготовки и приблизительное время до его завершения.
3. Если конфигурация подготовки, вероятно, находится в неработоспособном состоянии, приложение перейдет в карантин. Дополнительные сведения о режимах карантина см. [здесь](../app-provisioning/application-provisioning-quarantine-status.md).

## <a name="troubleshooting-tips"></a>Советы по устранению неполадок
*  Если вам не удалось создать пользователя, а журнал аудита содержит событие с кодом "1789003", это означает, что пользователь из непроверенного домена.

## <a name="change-log"></a>Журнал изменений

* 10.09.2020. Добавлена поддержка корпоративных атрибутов "division", "organization", "costCenter" и "employeeNumber". Добавлена поддержка настраиваемых атрибутов "startDate", "auth_method" и "frontline".

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)