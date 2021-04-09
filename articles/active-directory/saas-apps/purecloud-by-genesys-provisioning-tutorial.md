---
title: Руководство по настройке PureCloud от Genesys для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Сведения о настройке автоматической подготовки и отзыве учетных записей пользователей из Azure AD в PureCloud от Genesys.
services: active-directory
author: Zhchia
writer: Zhchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/05/2020
ms.author: Zhchia
ms.openlocfilehash: bbb9b47e42ce195a98801ee08d177efd409c597e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96181671"
---
# <a name="tutorial-configure-purecloud-by-genesys-for-automatic-user-provisioning"></a>Руководство по настройке PureCloud от Genesys для автоматической подготовки пользователей

В этом учебнике описаны действия, которые необходимо выполнить в PureCloud от Genesys и Azure Active Directory (Azure AD) для настройки автоматической подготовки пользователей. После настройки Azure AD автоматически подготавливает и отзывает пользователей и группы для [PureCloud от Genesys](https://www.genesys.com) с помощью службы подготовки Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Поддерживаемые возможности
> [!div class="checklist"]
> * Создание пользователей в PureCloud от Genesys.
> * Удаление пользователей из PureCloud от Genesys, когда доступ им больше не нужен.
> * Синхронизация атрибутов пользователей между Azure AD и PureCloud от Genesys.
> * Подготовка групп и членства в группах в PureCloud от Genesys.
> * [Единый вход](./purecloud-by-genesys-tutorial.md) в PureCloud от Genesys (рекомендуется).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* [Клиент Azure AD.](../develop/quickstart-create-new-tenant.md) 
* Учетная запись пользователя в Azure AD с [разрешением](../roles/permissions-reference.md) на настройку подготовки (например, администратор приложений, администратор облачных приложений, владелец приложения или глобальный администратор). 
* [Организация](https://help.mypurecloud.com/?p=81984) PureCloud.
* Пользователь с [разрешениями](https://help.mypurecloud.com/?p=24360) для создания клиента OAuth.

## <a name="step-1-plan-your-provisioning-deployment"></a>Шаг 1. Планирование развертывания для подготовки
1. Узнайте, [как работает служба подготовки](../app-provisioning/user-provisioning.md).
2. Определите, кто будет находиться в [области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Определите, какие данные следует [сопоставлять между Azure AD и PureCloud от Genesys](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-purecloud-by-genesys-to-support-provisioning-with-azure-ad"></a>Шаг 2. Настройка PureCloud от Genesys для поддержки подготовки с помощью Azure AD

1. Создайте [клиент OAuth](https://help.mypurecloud.com/?p=188023), настроенный в вашей организации PureCloud.
2. Создайте токен [с помощью клиента OAuth](https://developer.mypurecloud.com/api/rest/authorization/use-client-credentials.html).
3. Если вы хотите автоматически подготавливать членство в группе в PureCloud, необходимо [создать группы](https://help.mypurecloud.com/?p=52397) в PureCloud с идентичным именем группы в Azure AD.

## <a name="step-3-add-purecloud-by-genesys-from-the-azure-ad-application-gallery"></a>Шаг 3. Добавление PureCloud от Genesys из коллекции приложений Azure AD

Добавьте PureCloud от Genesys из коллекции приложений Azure AD, чтобы начать управление подготовкой в PureCloud от Genesys. Если вы ранее настроили единый вход в PureCloud от Genesys, можно использовать то же приложение. Однако при первоначальном тестировании интеграции рекомендуется создать отдельное приложение. Дополнительные сведения о добавлении приложения из коллекции см. [здесь](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Шаг 4. Определение пользователей для включения в область подготовки 

Служба подготовки Azure AD позволяет определить пользователей, которые будут подготовлены, на основе назначения приложению и (или) атрибутов пользователя или группы. Если вы решили определить пользователей на основе назначения, выполните следующие [действия](../manage-apps/assign-user-or-group-access-portal.md), чтобы назначить пользователей и группы приложению. Если вы решили указать, кто именно будет подготовлен, на основе одних только атрибутов пользователя или группы, можете использовать фильтр задания области, как описано [здесь](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* При назначении пользователей и групп для PureCloud от Genesys необходимо выбрать роль, отличную от роли **Доступ по умолчанию**. Пользователи с ролью "Доступ по умолчанию" исключаются из подготовки и будут помечены в журналах подготовки как не назначенные явно. Кроме того, если эта роль является единственной, доступной в приложении, можно [изменить манифест приложения](../develop/howto-add-app-roles-in-azure-ad-apps.md), чтобы добавить дополнительные роли. 

* Начните с малого. Протестируйте небольшой набор пользователей и групп, прежде чем выполнять развертывание для всех. Если в область подготовки включены назначенные пользователи и группы, проверьте этот механизм, назначив приложению одного или двух пользователей либо одну или две группы. Если в область включены все пользователи и группы, можно указать [фильтр области на основе атрибутов](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-purecloud-by-genesys"></a>Шаг 5. Настройка автоматической подготовки пользователей в PureCloud от Genesys 

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в TestApp на основе их назначений в Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-purecloud-by-genesys-in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей в PureCloud от Genesys, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **PureCloud от Genesys**.

    ![Ссылка на PureCloud от Genesys в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите URL-адрес API и токен OAuth PureCloud от Genesys в полях **URL-адрес клиента** и **Секретный токен** соответственно. URL-адрес API будет структурирован в виде `{{API Url}}/api/v2/scim/v2` с использованием URL-адреса API для региона PureCloud из [центра разработчиков PureCloud](https://developer.mypurecloud.com/api/rest/index.html). Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к PureCloud от Genesys. Если установить подключение не удалось, убедитесь, что учетной записи PureCloud от Genesys назначены разрешения администратора, и повторите попытку.

    ![Снимок экрана: диалоговое окно учетных данных администратора, в котором можно ввести URL-адрес клиента и секретный токен.](./media/purecloud-by-genesys-provisioning-tutorial/provisioning.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Щелкните **Сохранить**.

8. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Users to PureCloud by Genesys** (Синхронизировать пользователей Azure Active Directory с PureCloud от Genesys).

9. В разделе **Сопоставление атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы из Azure AD в PureCloud от Genesys. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в PureCloud от Genesys при операциях обновления. Если вы решили изменить [целевой атрибут сопоставления](../app-provisioning/customize-application-attributes.md), сначала убедитесь, что API PureCloud от Genesys поддерживает фильтрацию пользователей по этому атрибуту. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

     |attribute|Тип|
     |---|---|
     |userName|Строка|
     |active|Логическое|
     |displayName|Строка|
     |emails[type eq "work"].value|Строка|
     |title|Строка|
     |phoneNumbers[type eq "mobile"].value|Строка|
     |phoneNumbers[type eq "work"].value|Строка|
     |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|Строка|
     |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|Справочник|
     |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:employeeNumber|Строка|
     

10. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Groups to PureCloud by Genesys** (Синхронизировать группы Azure Active Directory с PureCloud от Genesys).

11. В разделе **Сопоставление атрибутов** просмотрите атрибуты групп, которые синхронизированы из Azure AD в PureCloud от Genesys. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления групп в PureCloud от Genesys при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения. PureCloud от Genesys не поддерживает создание и удаление групп, а поддерживает только их обновление.

      |attribute|Тип|
      |---|---|
      |displayName|Строка|
      |externalId|Строка|
      |members|Справочник|

12. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Чтобы включить службу подготовки Azure AD для PureCloud от Genesys, в разделе **Параметры** измените значение параметра **Состояние подготовки** на **Включено**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

14. Определите пользователей и (или) группы для подготовки в PureCloud от Genesys, выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

15. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется цикл начальной синхронизации всех пользователей и групп, определенных в поле **Область** в разделе **Параметры**. Начальный цикл занимает больше времени, чем последующие циклы. Пока служба подготовки Azure AD запущена, они выполняются примерно каждые 40 минут. 

## <a name="step-6-monitor-your-deployment"></a>Шаг 6. Мониторинг развертывания
После настройки подготовки используйте следующие ресурсы для мониторинга развертывания.

* Используйте [журналы подготовки](../reports-monitoring/concept-provisioning-logs.md), чтобы определить, какие пользователи были подготовлены успешно или неудачно.
* Используйте [индикатор выполнения](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md), чтобы узнать состояние цикла подготовки и приблизительное время до его завершения.
* Если конфигурация подготовки, вероятно, находится в неработоспособном состоянии, приложение перейдет в карантин. Дополнительные сведения о режимах карантина см. [здесь](../app-provisioning/application-provisioning-quarantine-status.md).

## <a name="change-log"></a>Журнал изменений

10.09 — добавлена поддержка корпоративного атрибута "employeeNumber".

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)