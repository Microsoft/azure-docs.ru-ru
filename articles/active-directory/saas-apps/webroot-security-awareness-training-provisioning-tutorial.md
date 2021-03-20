---
title: Руководство по настройке Webroot Security Awareness Training для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Сведения о настройке автоматической подготовки и отзыве учетных записей пользователей из Azure AD в Webroot Security Awareness Training.
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: 455f4396-930e-4db5-a167-d3ea6a860a17
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 07/06/2020
ms.author: Zhchia
ms.openlocfilehash: f015718c59178b8094f535280edadd641d22a806
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98741933"
---
# <a name="tutorial-configure-webroot-security-awareness-training-for-automatic-user-provisioning"></a>Руководство по настройке Webroot Security Awareness Training для автоматической подготовки пользователей

В этом учебнике описаны действия, которые необходимо выполнить в Webroot Security Awareness Training и Azure Active Directory (Azure AD) для настройки автоматической подготовки пользователей. После настройки Azure AD автоматически осуществляет подготовку и отзыв пользователей и групп для [Webroot Security Awareness Training](https://www.webroot.com/) с помощью службы подготовки Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Поддерживаемые возможности
> [!div class="checklist"]
> * Создание пользователей в Webroot Security Awareness Training.
> * Удаление пользователей из Webroot Security Awareness Training, когда доступ им больше не нужен.
> * Синхронизация атрибутов пользователей между Azure AD и Webroot Security Awareness Training.
> * Подготовка групп и членства в группах в Webroot Security Awareness Training.

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* [Клиент Azure AD.](../develop/quickstart-create-new-tenant.md) 
* Учетная запись пользователя в Azure AD с [разрешением](../roles/permissions-reference.md) настраивать подготовку (например, администратор приложений, администратор облачных приложений, владелец приложения или глобальный администратор).
* Консоль поставщика управляемых служб с включенным приложением Webroot Security Awareness Training по крайней мере на одном из ваших сайтов.

## <a name="step-1-plan-your-provisioning-deployment"></a>Шаг 1. Планирование развертывания для подготовки
1. Узнайте, [как работает служба подготовки](../app-provisioning/user-provisioning.md).
2. Определите, кто будет находиться в [области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Определите, какие данные следует [сопоставить между Azure AD и Webroot Security Awareness Training](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-webroot-security-awareness-training-to-support-provisioning-with-azure-ad"></a>Шаг 2. Настройка в Webroot Security Awareness Training поддержки автоматической подготовки пользователей с помощью Azure AD

### <a name="obtain-a-secret-token"></a>Получение секретного токена

Чтобы подключить сайт к Azure AD, необходимо получить **секретный токен** для этого сайта в консоли управления Webroot.

1. Войдите в [консоль управления Webroot](https://identity.webrootanywhere.com/v1/Account/login#tab_customers)

2. На вкладке **Sites** (Сайты) в столбце Security Awareness Training (Обучение для повышения осведомленности по вопросам безопасности) щелкните значок шестеренки для сайта, который необходимо подключить к Azure AD.

    ![Значок шестеренки](./media/webroot-security-awareness-training-provisioning-tutorial/gear-icon.png)

3. Нажмите кнопку **Configure Azure AD Integration** (Настроить интеграцию Azure AD).

    ![Настройка интеграции Azure AD](./media/webroot-security-awareness-training-provisioning-tutorial/configure-azure-ad-integration.png)

4. Скопируйте и сохраните **секретный токен**. Его нужно будет ввести в поле "Секретный токен" на вкладке "Подготовка" для приложения Webroot Security Awareness Training на портале Azure.

5. Нажмите кнопку **Done**(Готово).

    ![Копирование секретного токена](./media/webroot-security-awareness-training-provisioning-tutorial/copy-secret-token.png)

## <a name="step-3-add-webroot-security-awareness-training-from-the-azure-ad-application-gallery"></a>Шаг 3. Добавление Webroot Security Awareness Training из коллекции приложений Azure AD

Добавьте Webroot Security Awareness Training в коллекцию приложений Azure AD, чтобы начать управление подготовкой в Webroot Security Awareness Training. Дополнительные сведения о добавлении приложения из коллекции см. [здесь](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Шаг 4. Определение пользователей для включения в область подготовки 

Служба подготовки Azure AD позволяет определить пользователей, которые будут подготовлены, на основе назначения приложению и (или) атрибутов пользователя или группы. Если вы решили определить пользователей на основе назначения, выполните следующие [действия](../manage-apps/assign-user-or-group-access-portal.md), чтобы назначить пользователей и группы приложению. Если вы решили указать, кто именно будет подготовлен, на основе одних только атрибутов пользователя или группы, можете использовать фильтр задания области, как описано [здесь](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* При назначении пользователей и групп для Webroot Security Awareness Training необходимо выбрать роль, отличную от роли **Доступ по умолчанию**. Пользователи с ролью "Доступ по умолчанию" исключаются из подготовки и будут помечены в журналах подготовки как не назначенные явно. Кроме того, если эта роль является единственной, доступной в приложении, можно [изменить манифест приложения](../develop/howto-add-app-roles-in-azure-ad-apps.md), чтобы добавить дополнительные роли. 

* Начните с малого. Протестируйте небольшой набор пользователей и групп, прежде чем выполнять развертывание для всех. Если в область подготовки включены назначенные пользователи и группы, проверьте этот механизм, назначив приложению одного или двух пользователей либо одну или две группы. Если в область включены все пользователи и группы, можно указать [фильтр области на основе атрибутов](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-webroot-security-awareness-training"></a>Шаг 5. Настройка автоматической подготовки пользователей в Webroot Security Awareness Training 

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в TestApp на основе их назначений в Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-webroot-security-awareness-training-in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей в Webroot Security Awareness Training, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Webroot Security Awareness Training**.

    ![Ссылка на Webroot Security Awareness Training в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите значение `https://awarenessapi.webrootanywhere.com/api/v2/scim` в поле **URL-адрес клиента**. Введите значение секретного токена, полученное ранее на шаге **Секретный токен**. Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к Webroot Security Awareness Training. Если установить подключение не удалось, убедитесь, что у учетной записи Security Awareness Training есть разрешения администратора, и повторите попытку.

    ![Снимок экрана: диалоговое окно учетных данных администратора, в котором можно ввести URL-адрес клиента и секретный токен.](./media/webroot-security-awareness-training-provisioning-tutorial/provisioning.png)

6. В поле **Электронная почта для уведомлений** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Щелкните **Сохранить**.

8. В разделе **Сопоставления** выберите **Подготовка пользователей Azure Active Directory**.

9. В разделе **Сопоставления атрибутов** просмотрите атрибуты пользователей, которые синхронизируются из Azure AD в Webroot Security Awareness Training. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в Webroot Security Awareness Training для операций обновления. Если вы решили изменить [целевой атрибут сопоставления](../app-provisioning/customize-application-attributes.md), сначала убедитесь, что API Webroot Security Awareness Training поддерживает фильтрацию пользователей по этому атрибуту. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

   |attribute|Тип|Поддерживается для фильтрации|
   |---|---|---|
   |userName|Строка|&check;|
   |externalId|Строка|
   |name.givenName|Строка|
   |name.familyName|Строка|
   |emails[type eq "work"].value|Строка|

10. В разделе **Сопоставления** выберите **Provision Azure Active Directory Groups** (Подготовка групп Azure Active Directory).

11. В разделе **Сопоставления атрибутов** просмотрите атрибуты группы, которые синхронизируются из Azure AD в Webroot Security Awareness Training. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления групп в Webroot Security Awareness Training для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

      |attribute|Тип|Поддерживается для фильтрации|
      |---|---|---|
      |displayName|Строка|&check;|
      |members|Справочник|
      |externalId|Строка|

12. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Чтобы включить службу подготовки Azure AD для Webroot Security Awareness Training, в разделе **Параметры** установите переключатель **Состояние подготовки** в положение **Включено**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

14. Определите пользователей и (или) группы для подготовки в Webroot Security Awareness Training, выбрав нужные значения в поле **Область** в разделе **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

15. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется цикл начальной синхронизации всех пользователей и групп, определенных в поле **Область** в разделе **Параметры**. Начальный цикл занимает больше времени, чем последующие циклы. Пока служба подготовки Azure AD запущена, они выполняются примерно каждые 40 минут. 

## <a name="step-6-monitor-your-deployment"></a>Шаг 6. Мониторинг развертывания
После настройки подготовки используйте следующие ресурсы для мониторинга развертывания.

1. Используйте [журналы подготовки](../reports-monitoring/concept-provisioning-logs.md), чтобы определить, какие пользователи были подготовлены успешно или неудачно.
2. Используйте [индикатор выполнения](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md), чтобы узнать состояние цикла подготовки и приблизительное время до его завершения.
3. Если конфигурация подготовки, вероятно, находится в неработоспособном состоянии, приложение перейдет в карантин. Дополнительные сведения о режимах карантина см. [здесь](../app-provisioning/application-provisioning-quarantine-status.md).

## <a name="change-log"></a>Журнал изменений
* 21.01.2021 — добавлена поддержка для основного атрибута "userName" для пользователей.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)