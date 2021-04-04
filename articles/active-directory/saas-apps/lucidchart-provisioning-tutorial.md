---
title: Руководство по настройке Lucidchart для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить автоматическую подготовку и учетных записей пользователей и ее отмену из Azure AD в Lucidchart.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/13/2020
ms.author: Zhchia
ms.openlocfilehash: e7447a8d742caef7a2c2ff0211e7e8002307b9e3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96178016"
---
# <a name="tutorial-configure-lucidchart-for-automatic-user-provisioning"></a>Руководство по настройке Lucidchart для автоматической подготовки пользователей

В этом учебнике описаны действия, которые необходимо выполнить в Lucidchart и Azure Active Directory (Azure AD) для настройки автоматической подготовки пользователей. После настройки Azure AD автоматически осуществляет и отменяет подготовку пользователей и групп для [Lucidchart](https://www.lucidchart.com/user/117598685#/subscriptionLevel) с помощью службы подготовки Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Поддерживаемые возможности
> [!div class="checklist"]
> * Создание пользователей в Lucidchart.
> * Удаление пользователей в Lucidchart, когда доступ им больше не нужен.
> * Синхронизация атрибутов пользователей между Azure AD и Lucidchart.
> * Подготовка групп и членства в группах в Lucidchart.
> * [Единый вход](./lucidchart-tutorial.md) в Lucidchart (рекомендуется).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* [Клиент Azure AD.](../develop/quickstart-create-new-tenant.md) 
* Учетная запись пользователя в Azure AD с [разрешением](../roles/permissions-reference.md) на настройку подготовки (например, администратор приложений, администратор облачных приложений, владелец приложения или глобальный администратор). 
* Клиент Lucidchart с [планом Enterprise](https://www.lucidchart.com/user/117598685#/subscriptionLevel) или выше.
* Учетная запись пользователя в Lucidchart с разрешениями администратора.

## <a name="step-1-plan-your-provisioning-deployment"></a>Шаг 1. Планирование развертывания для подготовки
1. Узнайте, [как работает служба подготовки](../app-provisioning/user-provisioning.md).
2. Определите, кто будет находиться в [области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Определите, какие данные следует [сопоставлять между Azure AD и Lucidchart](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-lucidchart-to-support-provisioning-with-azure-ad"></a>Шаг 2. Настройка Lucidchart для поддержки подготовки с помощью Azure AD

1. Войдите в [консоль администрирования Lucidchart](https://www.lucidchart.com). Перейдите в раздел **Team > App Integration** (Команда > Интеграция приложений).

      :::image type="content" source="./media/lucidchart-provisioning-tutorial/team1.png" alt-text="Снимок экрана консоли администрирования Lucidchart. В ней выделено и открыто меню Team (Команда). В разделе Admin (Администрирование) выделена плитка (Интеграция приложений)." border="false":::

2. Перейти к плитке **SCIM**.

      :::image type="content" source="./media/lucidchart-provisioning-tutorial/scim.png" alt-text="Снимок экрана консоли администрирования Lucidchart. На большой кнопке SCIM выделен текст SCIM и отображается баннер с надписью Enabled (Включено)." border="false":::

3. Прокрутите вниз, чтобы отобразились поля **Bearer token** (Токен носителя) и **Lucidchart Base URL** (Базовый URL-адрес Lucidchart). Скопируйте и сохраните значение поля **Bearer token** (Токен носителя). Его нужно будет ввести на вкладке "Подготовка" в поле **Секретный токен** для приложения Lucidchart на портале Azure. 

      ![Токен Lucidchart](./media/lucidchart-provisioning-tutorial/token.png)

## <a name="step-3-add-lucidchart-from-the-azure-ad-application-gallery"></a>Шаг 3. Добавление Lucidchart из коллекции приложений Azure AD

Добавьте Lucidchart из коллекции приложений Azure AD, чтобы начать управление подготовкой в Lucidchart. Если вы ранее настроили Lucidchart для единого входа, можете использовать то же приложение. Однако при первоначальном тестировании интеграции рекомендуется создать отдельное приложение. Дополнительные сведения о добавлении приложения из коллекции см. [здесь](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Шаг 4. Определение пользователей для включения в область подготовки 

Служба подготовки Azure AD позволяет определить пользователей, которые будут подготовлены, на основе назначения приложению и (или) атрибутов пользователя или группы. Если вы решили определить пользователей на основе назначения, выполните следующие [действия](../manage-apps/assign-user-or-group-access-portal.md), чтобы назначить пользователей и группы приложению. Если вы решили указать, кто именно будет подготовлен, на основе одних только атрибутов пользователя или группы, можете использовать фильтр задания области, как описано [здесь](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* При назначении пользователей и групп для Lucidchart необходимо выбрать роль, отличную от роли **Доступ по умолчанию**. Пользователи с ролью "Доступ по умолчанию" исключаются из подготовки и будут помечены в журналах подготовки как не назначенные явно. Кроме того, если эта роль является единственной, доступной в приложении, можно [изменить манифест приложения](../develop/howto-add-app-roles-in-azure-ad-apps.md), чтобы добавить дополнительные роли. 

* Начните с малого. Протестируйте небольшой набор пользователей и групп, прежде чем выполнять развертывание для всех. Если в область подготовки включены назначенные пользователи и группы, проверьте этот механизм, назначив приложению одного или двух пользователей либо одну или две группы. Если в область включены все пользователи и группы, можно указать [фильтр области на основе атрибутов](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-lucidchart"></a>Шаг 5. Настройка автоматической подготовки пользователей в Lucidchart 

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в TestApp на основе их назначений в Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-lucidchart-in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей Lucidchart в Azure AD, сделайте следующее.

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Lucidchart**.

    ![Ссылка на Lucidchart в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите в поле **Токен носителя** значение, полученное ранее из поля **Secret Token** (Секретный токен). Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к Lucidchart. Если установить подключение не удалось, убедитесь, что у учетной записи Lucidchart есть разрешения администратора, и повторите попытку.

      ![Подготовка](./media/Lucidchart-provisioning-tutorial/lucidchart1.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Щелкните **Сохранить**.

8. В разделе **Сопоставления** выберите **Синхронизировать пользователей Azure Active Directory с Lucidchart**.

9. В разделе **Сопоставления атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы из Azure AD в Lucidchart. Атрибуты, выбранные как свойства **Matching**, используются для сопоставления учетных записей пользователей в Lucidchart для операций обновления. Если вы решили изменить [целевой атрибут сопоставления](../app-provisioning/customize-application-attributes.md), сначала убедитесь, что API Lucidchart поддерживает фильтрацию пользователей по этому атрибуту. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

   |attribute|Тип|
   |---|---|
   |userName|Строка|
   |emails[type eq "work"].value|Строка|
   |active|Логическое|
   |name.givenName|Строка|
   |name.familyName|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:division|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:costCenter|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:organization|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:employeeNumber|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|Справочник|
   |urn:ietf:params:scim:schemas:extension:lucidchart:1.0:User:canEdit|логический|

10. В разделе **Сопоставления** выберите **Синхронизировать группы Azure Active Directory с Lucidchart**.

11. В разделе **Сопоставления атрибутов** просмотрите атрибуты группы, которые синхронизированы из Azure AD в Lucidchart. Атрибуты, выбранные как свойства в разделе **Сопоставление**, используются для сопоставления групп в Lucidchart в операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

      |attribute|Тип|
      |---|---|
      |displayName|Строка|
      |members|Справочник|

12. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Чтобы включить службу подготовки Azure AD для Lucidchart, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

14. Определите пользователей и (или) группы для подготовки в Lucidchart, выбрав нужные значения в поле **Область** в разделе **Параметры**.

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

* 30.04.2020. Добавлена поддержка атрибута корпоративного расширения и настраиваемого атрибута "CanEdit" для пользователей.
* 15.06.2020. Включено программное удаление пользователей (поддержка атрибута [active](https://tools.ietf.org/html/rfc7643)).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)