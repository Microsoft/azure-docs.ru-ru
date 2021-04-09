---
title: Руководство по настройке автоматической подготовки пользователей в Cofense Recipient Sync с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить автоматическую отмену и подготовку учетных записей пользователей из Azure AD в Cofense Recipient Sync.
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: 84fe20ef-0de0-4f7c-9b42-6385f3d834db
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 09/11/2020
ms.author: Zhchia
ms.openlocfilehash: 70766fd93f0a70ecf6b5ee54dbdba3571f783878
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101646050"
---
# <a name="tutorial-configure-cofense-recipient-sync-for-automatic-user-provisioning"></a>Руководство по Настройка Cofense Recipient Sync для автоматической подготовки пользователей

В этом руководстве описаны действия, которые необходимо выполнить в Cofense Recipient Sync и Azure Active Directory (Azure AD) для настройки автоматической подготовки пользователей. После настройки Azure AD автоматически подготавливает и отзывает пользователей в [Cofense Recipient Sync](https://cofense.com/) с помощью службы подготовки Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Поддерживаемые возможности
> [!div class="checklist"]
> * Создание пользователей в Cofense Recipient Sync.
> * Удаление из Cofense Recipient Sync пользователей, которым больше не нужен доступ.
> * Синхронизация атрибутов пользователей между Azure AD и Cofense Recipient Sync.

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* [Клиент Azure AD.](../develop/quickstart-create-new-tenant.md) 
* Учетная запись пользователя в Azure AD с [разрешением](../roles/permissions-reference.md) настраивать подготовку (например, администратор приложений, администратор облачных приложений, владелец приложения или глобальный администратор). 
* Стандартная учетная запись оператора в Cofense PhishMe.

## <a name="step-1-plan-your-provisioning-deployment"></a>Шаг 1. Планирование развертывания для подготовки
1. Узнайте, [как работает служба подготовки](../app-provisioning/user-provisioning.md).
2. Определите, кто будет находиться в [области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Определите, какие данные следует [сопоставлять между Azure AD и Cofense Recipient Sync](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-cofense-recipient-sync-to-support-provisioning-with-azure-ad"></a>Шаг 2. Настройка поддержки подготовки в Cofense Recipient Sync с помощью Azure AD

1. Войдите в Cofense PhishMe. Перейдите в раздел **Recipients (Получатели) > Recipient Sync (Синхронизация получателей)** . 
2. Примите условия и щелкните **Get Started** (Приступить к работе).

    ![Recepient Sync tnc](media/cofense-provisioning-tutorial/recipient-sync-toc.png)

3. Скопируйте значения из полей **URL** (URL-адрес) и **Token** (Токен).

    ![Recepient Sync](media/cofense-provisioning-tutorial/recipient-sync-getting-started.png)


## <a name="step-3-add-cofense-recipient-sync-from-the-azure-ad-application-gallery"></a>Шаг 3. Добавление Cofense Recipient Sync из коллекции приложений Azure AD

Добавьте Cofense Recipient Sync из коллекции приложений Azure AD, чтобы начать управление подготовкой в Cofense Recipient Sync. Если вы уже настроили единый вход для Cofense Recipient Sync, можете использовать то же приложение. Однако при первоначальном тестировании интеграции рекомендуется создать отдельное приложение. Дополнительные сведения о добавлении приложения из коллекции см. [здесь](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Шаг 4. Определение пользователей для включения в область подготовки 

Служба подготовки Azure AD позволяет определить пользователей, которые будут подготовлены, на основе назначения приложению и (или) атрибутов пользователя или группы. Если вы решили определить пользователей на основе назначения, выполните следующие [действия](../manage-apps/assign-user-or-group-access-portal.md), чтобы назначить пользователей и группы приложению. Если вы решили указать, кто именно будет подготовлен, на основе одних только атрибутов пользователя или группы, можете использовать фильтр задания области, как описано [здесь](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* При назначении пользователей и групп для Cofense Recipient Sync необходимо выбрать роль, отличную от роли **Доступ по умолчанию**. Пользователи с ролью "Доступ по умолчанию" исключаются из подготовки и будут помечены в журналах подготовки как не назначенные явно. Кроме того, если эта роль является единственной, доступной в приложении, можно [изменить манифест приложения](../develop/howto-add-app-roles-in-azure-ad-apps.md), чтобы добавить дополнительные роли. 

* Начните с малого. Протестируйте небольшой набор пользователей и групп, прежде чем выполнять развертывание для всех. Если в область подготовки включены назначенные пользователи и группы, проверьте этот механизм, назначив приложению одного или двух пользователей либо одну или две группы. Если в область включены все пользователи и группы, можно указать [фильтр области на основе атрибутов](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-cofense-recipient-sync"></a>Шаг 5. Настройка автоматической подготовки пользователей в Cofense Recipient Sync 

В этом разделе объясняется, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и групп в Cofense Recipient Sync по данным о пользователях в Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-cofense-recipient-sync-in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей в Cofense Recipient Sync, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Cofense Recipient Sync**.

    ![Ссылка на Cofense в списке "Приложения"](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Вкладка "Подготовка"](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Вкладка "Подготовка" с выделенным значением "Авто"](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите полученные на этапе 2 значения **базового URL-адреса SCIM 2.0 и токена проверки подлинности SCIM**. Щелкните **Проверить подключение** и убедитесь, что Azure AD может подключиться к Cofense Recipient Sync. Если установить подключение не удалось, убедитесь, что у учетной записи Cofense Recipient Sync есть разрешения администратора и попробуйте еще раз.

    ![Токен и URL-адрес клиента](common/provisioning-testconnection-tenanturltoken.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Щелкните **Сохранить**.

8. В разделе **сопоставления** выберите **Synchronize Azure Active Directory Users to Cofense Recipient Sync** (Синхронизировать пользователей Azure Active Directory с Cofense Recipient Sync).

9. В разделе **Сопоставления атрибутов** просмотрите атрибуты пользователя, синхронизированные из Azure AD в Cofense Recipient Sync. Атрибуты, отмеченные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в Cofense Recipient Sync при операциях обновления.  Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

   |attribute|Тип|Поддерживается для фильтрации|
   |---|---|---|
   |externalId|Строка|&check;|
   |userName|Строка|
   |active|Логическое|
   |displayName|Строка|
   |name.formatted|Строка|
   |name.givenName|Строка|
   |name.familyName|Строка|
   |name.honorificSuffix|Строка|
   |phoneNumbers[type eq "work"].value|Строка|
   |phoneNumbers[type eq "home"].value|Строка|
   |phoneNumbers[type eq "other"].value|Строка|
   |phoneNumbers[type eq "pager"].value|Строка|
   |phoneNumbers[type eq "mobile"].value|Строка|
   |phoneNumbers[type eq "fax"].value|Строка|
   |addresses[type eq "other"].formatted|Строка|
   |addresses[type eq "work"].formatted|Строка|
   |addresses[type eq "work"].streetAddress|Строка|
   |addresses[type eq "work"].locality|Строка|
   |addresses[type eq "work"].region|Строка|
   |addresses[type eq "work"].postalCode|Строка|
   |addresses[type eq "work"].country|Строка|
   |title|Строка|
   |emails[type eq "work"].value|Строка|
   |emails[type eq "home"].value|Строка|
   |emails[type eq "other"].value|Строка|
   |preferredLanguage|Строка|
   |nickName|Строка|
   |userType|Строка|
   |локаль|Строка|
   |timezone|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:employeeNumber|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|Справочник|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:costCenter|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:division|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:organization|Строка|

10. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Чтобы включить службу подготовки Azure AD для Cofense Recipient Sync, в разделе **Параметры** установите переключатель **Состояние подготовки** в положение **Включено**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

12. Определите пользователей и (или) группы, подготовку которых нужно выполнить в Cofense Recipient Sync, выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

13. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется цикл начальной синхронизации всех пользователей и групп, определенных в поле **Область** в разделе **Параметры**. Начальный цикл занимает больше времени, чем последующие циклы. Пока служба подготовки Azure AD запущена, они выполняются примерно каждые 40 минут. 

## <a name="step-6-monitor-your-deployment"></a>Шаг 6. Мониторинг развертывания
После настройки подготовки используйте следующие ресурсы для мониторинга развертывания.

* Используйте [журналы подготовки](../reports-monitoring/concept-provisioning-logs.md), чтобы определить, какие пользователи были подготовлены успешно или неудачно.
* Используйте [индикатор выполнения](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md), чтобы узнать состояние цикла подготовки и приблизительное время до его завершения.
* Если конфигурация подготовки, вероятно, находится в неработоспособном состоянии, приложение перейдет в карантин. Дополнительные сведения о режимах карантина см. [здесь](../app-provisioning/application-provisioning-quarantine-status.md). 

## <a name="change-log"></a>Журнал изменений

* 15.01.2020. Для сопоставления objectId и externalId реализовано изменение со значения "Только в ходе создания объектов" на "Всегда".

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)