---
title: Руководство по настройке New Relic by Organization для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Сведения об автоматической подготовке и отзыве учетных записей пользователей из Azure AD в New Relic by Organization.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/14/2020
ms.author: Zhchia
ms.openlocfilehash: 6d196f7037fe9c0209e66e9d6e9ab25bdf807b59
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96181914"
---
# <a name="tutorial-configure-new-relic-by-organization-for-automatic-user-provisioning"></a>Руководство по настройке New Relic by Organization для автоматической подготовки пользователей

В этом учебнике описаны действия, которые необходимо выполнить в New Relic by Organization и Azure Active Directory (Azure AD) для настройки автоматической подготовки пользователей. После настройки Azure AD автоматически подготавливает и отзывает пользователей и группы в [New Relic by Organization](https://newrelic.com/) с помощью службы подготовки Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Поддерживаемые возможности
> [!div class="checklist"]
> * Создание пользователей в New Relic by Organization.
> * Удаление пользователей из New Relic by Organization, когда доступ им больше не нужен.
> * Синхронизация атрибутов пользователей между Azure AD и New Relic by Organization.
> * Подготовка групп и членства в группах в New Relic by Organization.
> * [Единый вход](./new-relic-limited-release-tutorial.md) в New Relic by Organization (рекомендуется).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* [Клиент Azure AD.](../develop/quickstart-create-new-tenant.md) 
* Учетная запись пользователя в Azure AD с [разрешением](../roles/permissions-reference.md) на настройку подготовки (например, администратор приложений, администратор облачных приложений, владелец приложения или глобальный администратор). 
* Одна или несколько учетных записей в New Relic by Organization, к которым вы хотите предоставить пользователям доступ. 

## <a name="step-1-plan-your-provisioning-deployment"></a>Шаг 1. Планирование развертывания для подготовки
1. Узнайте, [как работает служба подготовки](../app-provisioning/user-provisioning.md).
2. Определите, кто будет находиться в [области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Определите, какие данные следует [сопоставлять между Azure AD и New Relic by Organization](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-new-relic-by-organization-to-support-provisioning-with-azure-ad"></a>Шаг 2. Настройка New Relic by Organization для поддержки подготовки с помощью Azure AD

Обратитесь к представителю вашей учетной записи или получите поддержку на сайте support.newrelic.com, чтобы настроить SCIM и единый вход для своей организации. Предоставьте представителю учетной записи следующие сведения:

- Название вашей организации.
- Список идентификаторов учетных записей New Relic для связи с организацией.

Используя эти сведения, представитель учетной записи создаст для вас запись организации в нашей новой системе и свяжет ваши учетные записи с этой организацией.

Представитель учетной записи предоставит вам следующие сведения, которые понадобятся для настройки приложения SCIM/SSO New Relic для вашего поставщика удостоверений:

- конечная точка SCIM (URL-адрес клиента);
- токен носителя SCIM (секретный токен).

Токен носителя SCIM позволяет инициализировать пользователей в New Relic, поэтому храните это значение защищенным. Представитель поставщика учетной записи передаст вам токен носителя SCIM безопасным способом.

## <a name="step-3-add-new-relic-by-organization-from-the-azure-ad-application-gallery"></a>Шаг 3. Добавление New Relic by Organization из коллекции приложений Azure AD

Добавьте New Relic by Organization из коллекции приложений Azure AD, чтобы начать управление подготовкой New Relic by Organization. Если вы ранее настроили единый вход в New Relic by Organization, можно использовать то же приложение. Однако при первоначальном тестировании интеграции рекомендуется создать отдельное приложение. Дополнительные сведения о добавлении приложения из коллекции см. [здесь](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Шаг 4. Определение пользователей для включения в область подготовки 

Служба подготовки Azure AD позволяет определить пользователей, которые будут подготовлены, на основе назначения приложению и (или) атрибутов пользователя или группы. Если вы решили определить пользователей на основе назначения, выполните следующие [действия](../manage-apps/assign-user-or-group-access-portal.md), чтобы назначить пользователей и группы приложению. Если вы решили указать, кто именно будет подготовлен, на основе одних только атрибутов пользователя или группы, можете использовать фильтр задания области, как описано [здесь](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* При назначении пользователей и групп для New Relic by Organization необходимо выбрать роль, отличную от роли **Доступ по умолчанию**. Пользователи с ролью "Доступ по умолчанию" исключаются из подготовки и будут помечены в журналах подготовки как не назначенные явно. Кроме того, если эта роль является единственной, доступной в приложении, можно [изменить манифест приложения](../develop/howto-add-app-roles-in-azure-ad-apps.md), чтобы добавить дополнительные роли. 

* Начните с малого. Протестируйте небольшой набор пользователей и групп, прежде чем выполнять развертывание для всех. Если в область подготовки включены назначенные пользователи и группы, проверьте этот механизм, назначив приложению одного или двух пользователей либо одну или две группы. Если в область включены все пользователи и группы, можно указать [фильтр области на основе атрибутов](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-new-relic-by-organization"></a>Шаг 5. Настройка автоматической подготовки пользователей в New Relic by Organization 

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в TestApp на основе их назначений в Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-new-relic-by-organization-in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей в New Relic by Organization, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **New Relic by Organization**.

    ![Ссылка на New Relic в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите значение `https://scim-provisioning.service.newrelic.com/scim/v2` в поле "URL-адрес клиента". Введите полученное ранее значение токена проверки подлинности SCIM в поле **Секретный токен**. Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к New Relic. Если установить подключение не удалось, убедитесь, что у учетной записи New Relic есть разрешения администратора, и повторите попытку.

    ![Снимок экрана: диалоговое окно учетных данных администратора, в котором можно ввести URL-адрес клиента и секретный токен.](./media/new-relic-by-organization-provisioning-tutorial/provisioning.png)

6. В поле **Электронная почта для уведомлений** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Щелкните **Сохранить**.

8. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Users to New Relic by Organization** (Синхронизировать пользователей Azure Active Directory с New Relic by Organization).

9. В разделе **Сопоставление атрибутов** просмотрите атрибуты пользователей, которые синхронизированы из Azure AD в New Relic by Organization. Атрибуты, которые выбраны в качестве свойств **сопоставления**, будут использоваться для сопоставления учетных записей пользователей в New Relic by Organization для операций обновления. Если вы решили изменить [целевой атрибут сопоставления](../app-provisioning/customize-application-attributes.md), сначала убедитесь, что API New Relic by Organization поддерживает фильтрацию пользователей по этому атрибуту. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

   |attribute|Тип|
   |---|---|
   |userName|Строка|
   |externalId|Строка|
   |active|Логическое|
   |emails[type eq "work"].value|Строка|
   |name.givenName|Строка|
   |name.formatted|Строка|
   |timezone|Строка|

10. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Groups to New Relic by Organization** (Синхронизировать группы Azure Active Directory с New Relic by Organization).

11. В разделе **Сопоставление атрибутов** просмотрите атрибуты групп, которые синхронизированы из Azure AD в New Relic by Organization. Атрибуты, которые выбраны в качестве свойств **сопоставления**, будут использоваться для сопоставления групп в New Relic by Organization для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

      |attribute|Тип|
      |---|---|
      |displayName|Строка|
      |externalId|Строка|
      |members|Справочник|

12. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Чтобы включить службу подготовки Azure AD для New Relic by Organization, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

14. Определите пользователей и (или) группы для подготовки в New Relic by Organization, выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

15. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется цикл начальной синхронизации всех пользователей и групп, определенных в поле **Область** в разделе **Параметры**. Начальный цикл занимает больше времени, чем последующие циклы. Пока служба подготовки Azure AD запущена, они выполняются примерно каждые 40 минут. 

## <a name="step-6-monitor-your-deployment"></a>Шаг 6. Мониторинг развертывания
После настройки подготовки используйте следующие ресурсы для мониторинга развертывания.

* Используйте [журналы подготовки](../reports-monitoring/concept-provisioning-logs.md), чтобы определить, какие пользователи были подготовлены успешно или неудачно.
* Используйте [индикатор выполнения](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md), чтобы узнать состояние цикла подготовки и приблизительное время до его завершения.
* Если конфигурация подготовки, вероятно, находится в неработоспособном состоянии, приложение перейдет в карантин. Дополнительные сведения о режимах карантина см. [здесь](../app-provisioning/application-provisioning-quarantine-status.md).  


## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)