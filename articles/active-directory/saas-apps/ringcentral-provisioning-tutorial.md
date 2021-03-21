---
title: Руководство по настройке RingCentral для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить автоматическую отмену и подготовку учетных записей пользователей из Azure AD в RingCentral.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/30/2019
ms.author: Zhchia
ms.openlocfilehash: f57114fc4cb76c500cc422966635273c3a923046
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96181634"
---
# <a name="tutorial-configure-ringcentral-for-automatic-user-provisioning"></a>Руководство по настройке RingCentral для автоматической подготовки пользователей

В этом учебнике описаны действия, которые необходимо выполнить в RingCentral и Azure Active Directory (Azure AD) для настройки автоматической подготовки пользователей. После настройки Azure AD автоматически осуществляет и отменяет подготовку пользователей и групп для [RingCentral](https://www.ringcentral.com/office/plansandpricing.html) с помощью службы подготовки Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Поддерживаемые возможности
> [!div class="checklist"]
> * Создание пользователей в RingCentral.
> * Удаление пользователей в RingCentral, когда доступ им больше не нужен.
> * Синхронизация атрибутов пользователей между Azure AD и RingCentral.
> * [Единый вход](./ringcentral-tutorial.md) в RingCentral (рекомендуется).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* [Клиент Azure AD.](../develop/quickstart-create-new-tenant.md) 
* Учетная запись пользователя в Azure AD с [разрешением](../roles/permissions-reference.md) на настройку подготовки (например, администратор приложений, администратор облачных приложений, владелец приложения или глобальный администратор). 
* [Клиент RingCentral](https://www.ringcentral.com/office/plansandpricing.html).
* Учетная запись пользователя в RingCentral с разрешениями администратора.

## <a name="step-1-plan-your-provisioning-deployment"></a>Шаг 1. Планирование развертывания для подготовки
1. Узнайте, [как работает служба подготовки](../app-provisioning/user-provisioning.md).
2. Определите, кто будет находиться в [области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Определите, какие данные следует [сопоставлять между Azure AD и RingCentral](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-ringcentral-to-support-provisioning-with-azure-ad"></a>Шаг 2. Настройка RingCentral для поддержки подготовки с помощью Azure AD

1. Войдите в [консоль администрирования RingCentral](https://login.ringcentral.com/sw.html). Выберите **Tools > Directory Integration** ( Средства > Интеграция каталогов).

    ![Консоль администрирования RingCentral](media/ringcentral-provisioning-tutorial/admin.png)

2.  Выберите **SCIM** в разделе **Select Directory Provider** (Выбор поставщика каталога). (В будущем будет доступен параметр с именем Azure Active Directory). Щелкните **Enable SCIM service** (Включить службу SCIM).

    ![Добавление SCIM в RingCentral](media/ringcentral-provisioning-tutorial/scim.png)

3.  Обратитесь в службу поддержки RingCentral по адресу matthew.hunt@ringcentral.com, чтобы получить **токен проверки подлинности SCIM**. Его нужно будет ввести на вкладке "Подготовка" в поле "Секретный токен" для приложения RingCentral на портале Azure.

> [!NOTE]
> Чтобы назначить лицензии пользователям, ознакомьтесь с видео по [этой ссылке](https://support.ringcentral.com/s/article/5-10-Adding-Extensions-via-Web?language).

## <a name="step-3-add-ringcentral-from-the-azure-ad-application-gallery"></a>Шаг 3. Добавление RingCentral из коллекции приложений Azure AD

Добавьте RingCentral из коллекции приложений Azure AD, чтобы начать управление подготовкой в RingCentral. Если вы ранее настроили RingCentral для единого входа, можете использовать то же приложение. Однако при первоначальном тестировании интеграции рекомендуется создать отдельное приложение. Дополнительные сведения о добавлении приложения из коллекции см. [здесь](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Шаг 4. Определение пользователей для включения в область подготовки 

Служба подготовки Azure AD позволяет определить пользователей, которые будут подготовлены, на основе назначения приложению и (или) атрибутов пользователя или группы. Если вы решили определить пользователей на основе назначения, выполните следующие [действия](../manage-apps/assign-user-or-group-access-portal.md), чтобы назначить пользователей и группы приложению. Если вы решили указать, кто именно будет подготовлен, на основе одних только атрибутов пользователя или группы, можете использовать фильтр задания области, как описано [здесь](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* При назначении пользователей и групп для RingCentral необходимо выбрать роль, отличную от роли **Доступ по умолчанию**. Пользователи с ролью "Доступ по умолчанию" исключаются из подготовки и будут помечены в журналах подготовки как не назначенные явно. Кроме того, если эта роль является единственной, доступной в приложении, можно [изменить манифест приложения](../develop/howto-add-app-roles-in-azure-ad-apps.md), чтобы добавить дополнительные роли. 

* Начните с малого. Протестируйте небольшой набор пользователей и групп, прежде чем выполнять развертывание для всех. Если в область подготовки включены назначенные пользователи и группы, проверьте этот механизм, назначив приложению одного или двух пользователей либо одну или две группы. Если в область включены все пользователи и группы, можно указать [фильтр области на основе атрибутов](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-ringcentral"></a>Шаг 5. Настройка автоматической подготовки пользователей в RingCentral 

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в TestApp на основе их назначений в Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-ringcentral-in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей в RingCentral в Azure AD, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **RingCentral**.

    ![Ссылка на RingCentral в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите значение `https://platform.ringcentral.com/scim/v2` в поле **URL-адрес клиента**. Введите полученное ранее значение **токена проверки подлинности SCIM** в поле **Секретный токен**. Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к RingCentral. Если установить подключение не удалось, убедитесь, что у учетной записи RingCentral есть разрешения администратора, и повторите попытку.

    ![Снимок экрана: текстовые поля "URL-адрес клиента" и "Секретный токен" с вызванным параметром "Проверка подключения".](./media/ringcentral-provisioning-tutorial/provisioning.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Щелкните **Сохранить**.

8. В разделе **сопоставления** выберите **Synchronize Azure Active Directory Users to RingCentral** (Синхронизировать пользователей Azure Active Directory с RingCentral).

9. В разделе **Сопоставление атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы из Azure AD в RingCentral. Атрибуты, выбранные как свойства в разделе **Сопоставления**, используются для сопоставления учетных записей пользователей в RingCentral для операций обновления. Если вы решили изменить [соответствующий целевой атрибут](../app-provisioning/customize-application-attributes.md), потребуется убедиться, что API RingCentral поддерживает фильтрацию пользователей по этому атрибуту. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

   |attribute|Тип|
   |---|---|
   |userName|Строка|
   |externalId|Строка|
   |active|Логическое|
   |title|Строка|
   |emails[type eq "work"].value|Строка|
   |addresses[type eq "work"].country|Строка|
   |addresses[type eq "work"].region|Строка|
   |addresses[type eq "work"].locality|Строка|
   |addresses[type eq "work"].postalCode|Строка|
   |addresses[type eq "work"].streetAddress|Строка|
   |name.givenName|Строка|
   |name.familyName|Строка|
   |phoneNumbers[type eq "mobile"].value|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|Строка|

10. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Чтобы включить службу подготовки Azure AD для RingCentral, в разделе **Параметры** установите переключатель **Состояние подготовки** в положение **Включено**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

12. Определите пользователей и (или) группы для подготовки в RingCentral, выбрав нужные значения в поле **Область** в разделе **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

13. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется цикл начальной синхронизации всех пользователей и групп, определенных в поле **Область** в разделе **Параметры**. Начальный цикл занимает больше времени, чем последующие циклы. Пока служба подготовки Azure AD запущена, они выполняются примерно каждые 40 минут. 

## <a name="step-6-monitor-your-deployment"></a>Шаг 6. Мониторинг развертывания
После настройки подготовки используйте следующие ресурсы для мониторинга развертывания.

1. Используйте [журналы подготовки](../reports-monitoring/concept-provisioning-logs.md), чтобы определить, какие пользователи были подготовлены успешно или неудачно.
2. Используйте [индикатор выполнения](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md), чтобы узнать состояние цикла подготовки и приблизительное время до его завершения.
3. Если конфигурация подготовки, вероятно, находится в неработоспособном состоянии, приложение перейдет в карантин. Дополнительные сведения о режимах карантина см. [здесь](../app-provisioning/application-provisioning-quarantine-status.md).

## <a name="change-log"></a>Журнал изменений

* 10.09.2020 — удалена поддержка атрибутов displayName и "Руководитель".

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)