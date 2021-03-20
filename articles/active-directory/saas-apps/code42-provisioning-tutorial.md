---
title: Руководство по настройке Code42 для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить автоматическую отмену и подготовку учетных записей пользователей из Azure AD в Code42.
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: ddcb950b-3f9a-4ebb-bf78-4ec42d16d52d
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 09/23/2020
ms.author: Zhchia
ms.openlocfilehash: ffae286437a476d5ebe08c3ee442d0d52af84eb7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96179350"
---
# <a name="tutorial-configure-code42-for-automatic-user-provisioning"></a>Руководство по настройке Code42 для автоматической подготовки пользователей

В этом учебнике описаны действия, которые необходимо выполнить в Code42 и Azure Active Directory (Azure AD) для настройки автоматической подготовки пользователей. После настройки Azure AD автоматически осуществляет и отменяет подготовку пользователей и групп для [Code42](https://www.crashplan.com/) с помощью службы подготовки Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).


## <a name="capabilities-supported"></a>Поддерживаемые возможности
> [!div class="checklist"]
> * Создание пользователей в Code42.
> * Удаление пользователей в Code42, когда им больше не нужен доступ.
> * Синхронизация атрибутов пользователей между Azure AD и Code42.
> * Подготовка групп и членства в группах в Code42.
> * [Единый вход](./code42-tutorial.md) в Code42 (рекомендуется)

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* [Клиент Azure AD.](../develop/quickstart-create-new-tenant.md)
* Учетная запись пользователя в Azure AD с [разрешением](../roles/permissions-reference.md) настраивать подготовку (например, администратор приложений, администратор облачных приложений, владелец приложения или глобальный администратор).
* Клиент Code42 с включенным управлением удостоверениями.
* Учетная запись пользователя Code42 с правами [облачного администратора клиента](https://support.code42.com/Administrator/Cloud/Monitoring_and_managing/Roles_reference#Customer_Cloud_Admin).

## <a name="step-1-plan-your-provisioning-deployment"></a>Шаг 1. Планирование развертывания для подготовки
1. Узнайте, [как работает служба подготовки](../app-provisioning/user-provisioning.md).
2. Определите, кто будет находиться в [области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Определите, какие данные следует [сопоставлять между Azure AD и Code42](../app-provisioning/customize-application-attributes.md).

## <a name="step-2-configure-code42-to-support-provisioning-with-azure-ad"></a>Шаг 2. Настройка Code42 для поддержки подготовки с помощью Azure AD

В этом разделе описано, как настроить Azure AD в качестве поставщика подготовки в разделе "Управление удостоверениями" консоли Code42. Это позволит Code42 безопасно получать запросы на подготовку из Azure AD. Перед подготовкой к работе с Azure AD рекомендуется ознакомиться с [документацией по Code42](https://support.code42.com/Administrator/Cloud/Configuring/Introduction_to_SCIM_provisioning/How_to_provision_users_to_Code42_from_Azure_AD).

### <a name="to-create-a-provisioning-provider-in-code42s-console"></a>Чтобы создать поставщик подготовки в консоли Code42, выполните следующие действия.

1. Войдите в консоль Code42. Выберите **Администрирование**, чтобы развернуть меню навигации. Выберите **Параметры** затем **Управление удостоверениями**.
2. Выберите вкладку **Подготовка**. Затем разверните меню **Add provisioning provider** (Добавить поставщик подготовки) и выберите **Add SCIM provider** (Добавить поставщик SCIM).
3. В поле **Отображаемое имя** введите уникальное имя для поставщика подготовки. Задайте для параметра **Authentication credential type** (Тип учетных данных проверки подлинности) значение **Токен OAuth**. Выберите **Далее**, чтобы создать учетные данные.

> [!NOTE]
>* Не закрывайте это окно, пока не отобразится запрос на **Базовый URL-адрес** и **Маркер**, необходимые на следующих шагах.
>* Кроме того, скопируйте эти сведения во временное расположение, чтобы использовать их позднее.

## <a name="step-3-add-code42-from-the-azure-ad-application-gallery"></a>Шаг 3. Добавление Code42 из коллекции приложений Azure AD

Добавьте Code42 из коллекции приложений Azure AD, чтобы начать управление подготовкой в Code42. Если вы ранее настроили Code42 для единого входа, можете использовать то же приложение. Однако при первоначальном тестировании интеграции рекомендуется создать отдельное приложение. Дополнительные сведения о добавлении приложения из коллекции см. [здесь](../manage-apps/add-application-portal.md).

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Шаг 4. Определение пользователей для включения в область подготовки

Служба подготовки Azure AD позволяет определить пользователей, которые будут подготовлены, на основе назначения приложению и (или) атрибутов пользователя или группы. Если вы решили определить пользователей на основе назначения, выполните следующие [действия](../manage-apps/assign-user-or-group-access-portal.md), чтобы назначить пользователей и группы приложению. Если вы решили указать, кто именно будет подготовлен, на основе одних только атрибутов пользователя или группы, можете использовать фильтр задания области, как описано [здесь](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

* При назначении пользователей и групп для Code42 необходимо выбрать роль, отличную от роли **Доступ по умолчанию**. Пользователи с ролью "Доступ по умолчанию" исключаются из подготовки и будут помечены в журналах подготовки как не назначенные явно. Кроме того, если эта роль является единственной, доступной в приложении, можно [изменить манифест приложения](../develop/howto-add-app-roles-in-azure-ad-apps.md), чтобы добавить дополнительные роли.

* Начните с малого. Протестируйте небольшой набор пользователей и групп, прежде чем выполнять развертывание для всех. Если в область подготовки включены назначенные пользователи и группы, проверьте этот механизм, назначив приложению одного или двух пользователей либо одну или две группы. Если в область включены все пользователи и группы, можно указать [фильтр области на основе атрибутов](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).


## <a name="step-5-configure-automatic-user-provisioning-to-code42"></a>Шаг 5. Настройка автоматической подготовки пользователей в Code42

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в TestApp на основе их назначений в Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-code42-in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей в Code42, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Code42**.

    ![Ссылка на Code42 в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Вкладка "Подготовка"](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Вкладка "Подготовка" с выделенным значением "Авто"](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите полученные ранее значения **SCIM 2.0 base URL** (Базовый URL-адрес SCIM 2.0) и Access Token (Маркер доступа) bp Code42 в поля **URL-адрес клиента** и **Секретный токен** соответственно. Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к Code42. Если установить подключение не удалось, убедитесь, что в учетной записи Code42 есть разрешения администратора, и повторите попытку.

    ![Токен](common/provisioning-testconnection-tenanturltoken.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Щелкните **Сохранить**.

8. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Users to Code42** (Синхронизировать пользователей Azure Active Directory с Code42).

9. В разделе **Сопоставление атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы из Azure AD в Code42. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в Code42 для операций обновления. Если вы решили изменить [соответствующий целевой атрибут](../app-provisioning/customize-application-attributes.md), потребуется убедиться, что API Code42 поддерживает фильтрацию пользователей по этому атрибуту. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

   |attribute|Тип|
   |---|---|
   |userName|Строка|
   |active|Логическое|
   |title|Строка|
   |emails[type eq "work"].value|Строка|
   |name.givenName|Строка|
   |name.familyName|Строка|
   |addresses[type eq "work"].locality|Строка|
   |addresses[type eq "work"].region|Строка|
   |addresses[type eq "work"].country|Строка|
   |externalId|Строка|
   |userType|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:division|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|Справочник|

10. В разделе **Сопоставления** выберите **Синхронизировать группы Azure Active Directory с Code42**.

11. В разделе **Сопоставление атрибутов** просмотрите атрибуты группы, которые синхронизированы из Azure AD в Code42. Атрибуты, выбранные как свойства в разделе **Сопоставления**, используются для сопоставления групп в Code42 для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

      |attribute|Тип|
      |---|---|
      |displayName|Строка|
      |externalId|Строка|
      |members|Справочник|

12. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Чтобы включить службу подготовки Azure AD для Code42, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

14. Укажите пользователей и (или) группы для подготовки в Code42, выбрав нужные значения в поле **Область** в разделе **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

15. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется цикл начальной синхронизации всех пользователей и групп, определенных в поле **Область** в разделе **Параметры**. Начальный цикл занимает больше времени, чем последующие циклы. Пока служба подготовки Azure AD запущена, они выполняются примерно каждые 40 минут.

## <a name="step-6-monitor-your-deployment"></a>Шаг 6. Мониторинг развертывания
После настройки подготовки используйте следующие ресурсы для мониторинга развертывания.

1. Используйте [журналы подготовки](../reports-monitoring/concept-provisioning-logs.md), чтобы определить, какие пользователи были подготовлены успешно или неудачно.
2. Используйте [индикатор выполнения](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md), чтобы узнать состояние цикла подготовки и приблизительное время до его завершения.
3. Если конфигурация подготовки, вероятно, находится в неработоспособном состоянии, приложение перейдет в карантин. Дополнительные сведения о режимах карантина см. [здесь](../app-provisioning/application-provisioning-quarantine-status.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)
* [Настройка назначений организации на основе групп SCIM в Code42](https://support.code42.com/Administrator/Cloud/Configuring/Introduction_to_SCIM_provisioning/How_to_provision_users_to_Code42_from_Azure_AD#Step_6:_Choose_an_organization_mapping_method)
* [Настройка назначений ролей на основе групп SCIM в Code42](https://support.code42.com/Administrator/Cloud/Configuring/Introduction_to_SCIM_provisioning/How_to_provision_users_to_Code42_from_Azure_AD#Step_7:_Configure_role_mapping)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)