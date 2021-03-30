---
title: Руководство по настройке G Suite для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить автоматическую отмену и подготовку учетных записей пользователей из Azure AD в G Suite.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/18/2021
ms.author: Zhchia
ms.openlocfilehash: f6ef2f91d178db01e618c19df55705bc9c1209e4
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104583757"
---
# <a name="tutorial-configure-g-suite-for-automatic-user-provisioning"></a>Руководство по настройке G Suite для автоматической подготовки пользователей

В этом руководстве описаны действия, которые нужно выполнить в G Suite и Azure Active Directory (Azure AD) для настройки автоматической подготовки пользователей. После настройки Azure AD автоматически осуществляет и отменяет подготовку пользователей и групп для [G Suite](https://gsuite.google.com/) с помощью службы подготовки Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md). 

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="capabilities-supported"></a>Поддерживаемые возможности
> [!div class="checklist"]
> * Создание пользователей в G Suite.
> * Удаление пользователей в G Suite, которым больше не нужен доступ.
> * Синхронизация атрибутов пользователей между Azure AD и G Suite.
> * Подготовка групп и членства в группах в G Suite.
> * [Единый вход](./google-apps-tutorial.md) в G Suite (рекомендуется).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* [Клиент Azure AD.](../develop/quickstart-create-new-tenant.md) 
* Учетная запись пользователя в Azure AD с [разрешением](../roles/permissions-reference.md) на настройку подготовки (например, администратор приложений, администратор облачных приложений, владелец приложения или глобальный администратор). 
* [Клиент G Suite.](https://gsuite.google.com/pricing.html)
* Учетная запись пользователя G Suite с разрешениями администратора.

## <a name="step-1-plan-your-provisioning-deployment"></a>Шаг 1. Планирование развертывания для подготовки
1. Узнайте, [как работает служба подготовки](../app-provisioning/user-provisioning.md).
2. Определите, кто будет находиться в [области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Определите, какие данные следует [сопоставлять между Azure AD и G Suite](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-g-suite-to-support-provisioning-with-azure-ad"></a>Шаг 2. Настройка G Suite для поддержки подготовки с помощью Azure AD

Прежде чем настраивать автоматическую подготовку пользователей в Azure AD для G Suite, нужно включить подготовку SCIM для G Suite.

1. Войдите в [консоль администрирования G Suite](https://admin.google.com/) с учетной записью администратора, а затем выберите **Security** (Безопасность). Если эта ссылка не отображается, она может быть скрыта в меню **More Controls** (Другие элементы управления) в нижней части экрана.

    ![G Suite: Security (Безопасность)](./media/g-suite-provisioning-tutorial/gapps-security.png)

2. На странице **Security** (Безопасность) выберите **API Reference** (Справочник по API).

    ![G Suite: API](./media/g-suite-provisioning-tutorial/gapps-api.png)

3. Выберите **Включить доступ через API**.

    ![G Suite: включен доступ через API](./media/g-suite-provisioning-tutorial/gapps-api-enabled.png)

    > [!IMPORTANT]
   > Имя каждого пользователя в Azure AD, которого вы хотите подготовить для G Suite, **должно** быть связано с личным доменом. Например, такие имена пользователей, как bob@contoso.onmicrosoft.com, не будут приняты G Suite. а bob@contoso.com — будут. Чтобы изменить домен для существующего пользователя, воспользуйтесь [этой инструкцией](../fundamentals/add-custom-domain.md).

4. Завершив добавление и проверку личных доменов в Azure AD, их необходимо еще раз проверить, на этот раз в G Suite. Чтобы проверить домены в G Suite, выполните следующие действия:

    a. В [консоли администрирования G Suite](https://admin.google.com/) выберите **Domains** (Домены).

    ![G Suite: домены](./media/g-suite-provisioning-tutorial/gapps-domains.png)

    b. Выберите **Add a domain or a domain alias** (Добавить домен или псевдоним домена).

    ![G Suite: добавление домена](./media/g-suite-provisioning-tutorial/gapps-add-domain.png)

    c. Выберите **Add another domain** (Добавить другой домен), а затем введите имя домена, который нужно добавить.

    ![G Suite: добавление другого домена](./media/g-suite-provisioning-tutorial/gapps-add-another.png)

    d. Выберите **Continue and verify domain ownership** (Перейти к проверке владельца домена). Затем выполните действия, подтверждающие, что вы владеете доменным именем. Подробные инструкции по проверке домена в Google см. на странице [Как подтвердить право собственности на сайт](https://support.google.com/webmasters/answer/35179).

    д) Повторите описанные выше шаги для других доменов, которые нужно добавить в G Suite.

5. Затем определите, какую учетную запись администратора вы хотите использовать для управления подготовкой пользователей в G Suite. Перейдите к разделу **Admin Roles** (Роли администратора).

    ![G Suite: администратор](./media/g-suite-provisioning-tutorial/gapps-admin.png)

6. Для **роли администратора** этой учетной записи измените свойство **Privileges** (Привилегии). Убедитесь, что для учетной записи выбраны все параметры в разделе **Admin API Privileges** (Административные права доступа к API). Это позволит использовать ее для подготовки.

    ![G Suite: права администратора](./media/g-suite-provisioning-tutorial/gapps-admin-privileges.png)

## <a name="step-3-add-g-suite-from-the-azure-ad-application-gallery"></a>Шаг 3. Добавление G Suite из коллекции приложений Azure AD

Добавьте G Suite из коллекции приложений Azure AD, чтобы начать управление подготовкой в G Suite. Если вы уже настроили единый вход в G Suite, можете использовать то же приложение. Однако при первоначальном тестировании интеграции рекомендуется создать отдельное приложение. Дополнительные сведения о добавлении приложения из коллекции см. [здесь](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Шаг 4. Определение пользователей для включения в область подготовки 

Служба подготовки Azure AD позволяет определить пользователей, которые будут подготовлены, на основе назначения приложению и (или) атрибутов пользователя или группы. Если вы решили определить пользователей на основе назначения, выполните следующие [действия](../manage-apps/assign-user-or-group-access-portal.md), чтобы назначить пользователей и группы приложению. Если вы решили указать, кто именно будет подготовлен, на основе одних только атрибутов пользователя или группы, можете использовать фильтр задания области, как описано [здесь](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* При назначении пользователей и групп для G Suite нужно выбрать роль, отличную от роли **Доступ по умолчанию**. Пользователи с ролью "Доступ по умолчанию" исключаются из подготовки и будут помечены в журналах подготовки как не назначенные явно. Кроме того, если эта роль является единственной, доступной в приложении, можно [изменить манифест приложения](../develop/howto-add-app-roles-in-azure-ad-apps.md), чтобы добавить дополнительные роли. 

* Начните с малого. Протестируйте небольшой набор пользователей и групп, прежде чем выполнять развертывание для всех. Если в область подготовки включены назначенные пользователи и группы, проверьте этот механизм, назначив приложению одного или двух пользователей либо одну или две группы. Если в область включены все пользователи и группы, можно указать [фильтр области на основе атрибутов](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-g-suite"></a>Шаг 5. Настройка автоматической подготовки пользователей в G Suite 

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в TestApp на основе их назначений в Azure AD.

> [!NOTE]
> Дополнительные сведения о конечной точке API каталога G Suite можно найти [здесь](https://developers.google.com/admin-sdk/directory).

### <a name="to-configure-automatic-user-provisioning-for-g-suite-in-azure-ad"></a>Для настройки автоматической подготовки пользователей в Azure AD для G Suite сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**. Пользователям придется выполнять вход на сайте portal.azure.com, а сайт aad.portal.azure.com они не увидят.

    ![Колонка "Корпоративные приложения"](./media/g-suite-provisioning-tutorial/enterprise-applications.png)

    ![Колонка "Все приложения"](./media/g-suite-provisioning-tutorial/all-applications.png)

2. В списке приложений выберите **G Suite**.

    ![Ссылка на G Suite в списке "Приложения"](common/all-applications.png)

3. Выберите вкладку **Подготовка**. Нажмите кнопку **Начало работы**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

      ![Колонка "Начало работы"](./media/g-suite-provisioning-tutorial/get-started.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** щелкните **Авторизовать**. Вы перейдете к новому окну браузера, где откроется диалоговое окно авторизации Google.

      ![G Suite: авторизация](./media/g-suite-provisioning-tutorial/authorize-1.png)

6. Подтвердите, что вы готовы предоставить Azure AD разрешение на внесение изменений в клиент G Suite. Выберите **Принять**.

     ![G Suite: аутентификация клиента](./media/g-suite-provisioning-tutorial/gapps-auth.png)

7. На портале Azure щелкните **Проверить подключение** и убедитесь, что Azure AD может подключиться к G Suite. Если установить подключение не удалось, убедитесь, что у учетной записи G Suite есть разрешения администратора, и повторите попытку. Затем повторите шаг **авторизации**.

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Щелкните **Сохранить**.

8. В разделе **Сопоставления** выберите **Подготовка пользователей Azure Active Directory**.

9. В разделе **Сопоставления атрибутов** просмотрите атрибуты пользователей, которые синхронизируются из Azure AD в G Suite. Атрибуты, выбранные как свойства с меткой **Сопоставление**, используются для сопоставления учетных записей пользователей в G Suite при операциях обновления. Если вы решили изменить [целевой атрибут сопоставления](../app-provisioning/customize-application-attributes.md), потребуется убедиться, что API G Suite поддерживает фильтрацию пользователей по этому атрибуту. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

   |attribute|Тип|
   |---|---|
   |primaryEmail|Строка|
   |relations.[type eq "manager"].value|Строка|
   |name.familyName|Строка|
   |name.givenName|Строка|
   |suspended (приостановлено)|Строка|
   |externalIds.[type eq "custom"].value|Строка|
   |externalIds.[type eq "organization"].value|Строка|
   |addresses.[type eq "work"].country|Строка|
   |addresses.[type eq "work"].streetAddress|Строка|
   |addresses.[type eq "work"].region|Строка|
   |addresses.[type eq "work"].locality|Строка|
   |addresses.[type eq "work"].postalCode|Строка|
   |emails.[type eq "work"].address|Строка|
   |organizations.[type eq "work"].department|Строка|
   |organizations.[type eq "work"].title|Строка|
   |phoneNumbers.[type eq "work"].value|Строка|
   |phoneNumbers.[type eq "mobile"].value|Строка|
   |phoneNumbers.[type eq "work_fax"].value|Строка|
   |emails.[type eq "work"].address|Строка|
   |organizations.[type eq "work"].department|Строка|
   |organizations.[type eq "work"].title|Строка|
   |phoneNumbers.[type eq "work"].value|Строка|
   |phoneNumbers.[type eq "mobile"].value|Строка|
   |phoneNumbers.[type eq "work_fax"].value|Строка|
   |addresses.[type eq "home"].country|Строка|
   |addresses.[type eq "home"].formatted|Строка|
   |addresses.[type eq "home"].locality|Строка|
   |addresses[type eq "home"].postalCode|Строка|
   |addresses[type eq "home"].region|Строка|
   |addresses.[type eq "home"].streetAddress|Строка|
   |addresses.[type eq "other"].country|Строка|
   |addresses.[type eq "other"].formatted|Строка|
   |addresses.[type eq "other"].locality|Строка|
   |addresses.[type eq "other"].postalCode|Строка|
   |addresses.[type eq "other"].region|Строка|
   |addresses.[type eq "other"].streetAddress|Строка|
   |addresses.[type eq "work"].formatted|Строка|
   |changePasswordAtNextLogin|Строка|
   |emails.[type eq "home"].address|Строка|
   |emails.[type eq "other"].address|Строка|
   |externalIds.[type eq "account"].value|Строка|
   |externalIds.[type eq "custom"].customType|Строка|
   |externalIds.[type eq "customer"].value|Строка|
   |externalIds.[type eq "login_id"].value|Строка|
   |externalIds.[type eq "network"].value|Строка|
   |gender.type|Строка|
   |GeneratedImmutableId|Строка|
   |Идентификатор|Строка|
   |ims.[type eq "home"].protocol|Строка|
   |ims.[type eq "other"].protocol|Строка|
   |ims.[type eq "work"].protocol|Строка|
   |includeInGlobalAddressList|Строка|
   |ipWhitelisted|Строка|
   |organizations.[type eq "school"].costCenter|Строка|
   |organizations.[type eq "school"].department|Строка|
   |organizations.[type eq "school"].domain|Строка|
   |organizations.[type eq "school"].fullTimeEquivalent|Строка|
   |organizations.[type eq "school"].location|Строка|
   |organizations.[type eq "school"].name|Строка|
   |organizations.[type eq "school"].symbol|Строка|
   |organizations.[type eq "school"].title|Строка|
   |organizations.[type eq "work"].costCenter|Строка|
   |organizations.[type eq "work"].domain|Строка|
   |organizations.[type eq "work"].fullTimeEquivalent|Строка|
   |organizations.[type eq "work"].location|Строка|
   |organizations.[type eq "work"].name|Строка|
   |organizations.[type eq "work"].symbol|Строка|
   |OrgUnitPath|Строка|
   |phoneNumbers.[type eq "home"].value|Строка|
   |phoneNumbers.[type eq "other"].value|Строка|
   |websites.[type eq "home"].value|Строка|
   |websites.[type eq "other"].value|Строка|
   |websites.[type eq "work"].value|Строка|
   

10. В разделе **Сопоставления** выберите **Provision Azure Active Directory Groups** (Подготовка групп Azure Active Directory).

11. В разделе **Сопоставления атрибутов** просмотрите атрибуты групп, которые синхронизируются из Azure AD в G Suite. Атрибуты, выбранные как свойства с меткой **Сопоставление**, используются для сопоставления групп в G Suite при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

      |attribute|Тип|
      |---|---|
      |email|Строка|
      |Элементы|Строка|
      |name|Строка|
      |description;|Строка|

12. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Чтобы включить службу подготовки Azure AD для G Suite, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

14. Укажите пользователей и (или) группы для подготовки в G Suite, выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

15. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется цикл начальной синхронизации всех пользователей и групп, определенных в поле **Область** в разделе **Параметры**. Начальный цикл занимает больше времени, чем последующие циклы. Пока служба подготовки Azure AD запущена, они выполняются примерно каждые 40 минут.

> [!NOTE]
> Если у пользователей уже есть личные или потребительские учетные записи для адреса электронной почты, который указан в Azure AD, могут возникнуть некоторые проблемы, для устранения которых следует применить средство переноса Google перед выполнением синхронизации каталогов.

## <a name="step-6-monitor-your-deployment"></a>Шаг 6. Мониторинг развертывания
После настройки подготовки используйте следующие ресурсы для мониторинга развертывания.

1. Используйте [журналы подготовки](../reports-monitoring/concept-provisioning-logs.md), чтобы определить, какие пользователи были подготовлены успешно или неудачно.
2. Используйте [индикатор выполнения](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md), чтобы узнать состояние цикла подготовки и приблизительное время до его завершения.
3. Если конфигурация подготовки, вероятно, находится в неработоспособном состоянии, приложение перейдет в карантин. Дополнительные сведения о режимах карантина см. [здесь](../app-provisioning/application-provisioning-quarantine-status.md).  

## <a name="change-log"></a>Журнал изменений

* 17.10.2020: добавлена поддержка дополнительных атрибутов пользователей и групп G Suite.
* 17.10.2020: обновлены целевые имена атрибутов G Suite для соответствия приведенному [здесь](https://developers.google.com/admin-sdk/directory) определению.
* 17.10.2020: обновлены стандартные сопоставления атрибутов.
* 18.03.2021 — электронная почта руководителя теперь синхронизируется вместо идентификатора для всех новых пользователей. Для всех существующих пользователей, которые были подготовлены с руководителем в качестве идентификатора, можно выполнить перезагрузку через [Microsoft Graph](https://docs.microsoft.com/graph/api/synchronization-synchronizationjob-restart?view=graph-rest-beta&tabs=http) со всей областью, чтобы убедиться, что электронная почта подготовлена. Это изменение влияет только на задание подготовки GSuite, но не на более старое задание подготовки, начиная с Goov2OutDelta. Обратите внимание, что сообщение электронной почты руководителя подготавливается при первом создании пользователя или изменении руководителя. Электронная почта руководителя не будет подготавливаться, если руководитель изменит свой адрес электронной почты. 

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)
