---
title: Руководство по настройке OpenText Directory Services для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Сведения о настройке автоматической подготовки и отзыва учетных записей пользователей из Azure AD в OpenText Directory Services.
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: ad55ba5f-c56c-4ed0-bdfd-163d2883ed80
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 10/01/2020
ms.author: Zhchia
ms.openlocfilehash: 2f31eddab1070d073d3fd5a4761dad597e42a2e0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96181901"
---
# <a name="tutorial-configure-opentext-directory-services-for-automatic-user-provisioning"></a>Руководство по настройке OpenText Directory Services для автоматической подготовки пользователей

В этом учебнике описаны действия, которые необходимо выполнить в OpenText Directory Services и Azure Active Directory (Azure AD) для настройки автоматической подготовки пользователей. После настройки Azure AD автоматически подготавливает и отзывает пользователей и группы в OpenText Directory Services с помощью службы подготовки Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Поддерживаемые возможности
> [!div class="checklist"]
> * Создание пользователей в OpenText Directory Services.
> * Удаление пользователей из OpenText Directory Services, когда доступ им больше не нужен.
> * Синхронизация атрибутов пользователей между Azure AD и OpenText Directory Services.
> * Подготовка групп и членства в группах в OpenText Directory Services.
> * [Единый вход](./opentext-directory-services-tutorial.md) в OpenText Directory Services (рекомендуется)

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* [Клиент Azure AD.](../develop/quickstart-create-new-tenant.md) 
* Учетная запись пользователя в Azure AD с [разрешением](../roles/permissions-reference.md) настраивать подготовку (например, администратор приложений, администратор облачных приложений, владелец приложения или глобальный администратор). 
* Установка OTDS, доступная в Azure AD.

## <a name="step-1-plan-your-provisioning-deployment"></a>Шаг 1. Планирование развертывания для подготовки
1. Узнайте, [как работает служба подготовки](../app-provisioning/user-provisioning.md).
2. Определите, кто будет находиться в [области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Определите, какие данные следует [сопоставлять между Azure AD и OpenText Directory Services](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-opentext-directory-services-to-support-provisioning-with-azure-ad"></a>Шаг 2. Настройка OpenText Directory Services для поддержки подготовки с помощью Azure AD

> [!NOTE]
> Приведенные ниже шаги применяются к установке OpenText Directory Services. Они неприменимы для клиентов OpenText CoreShare или OpenText OT2.

1. Создание выделенного конфиденциального **клиента OAuth**.
2. Задайте длительное **время существования маркера доступа**.

      ![Время жизни маркера доступа](media/open-text-directory-services-provisioning-tutorial/token-life.png)

3. Не указывайте URL-адреса перенаправления. Они не требуются. 
4. OTDS создаст и отобразит **секрет клиента**. Сохраните значения **идентификатор клиента** и **секрет клиента** в секретном расположении.

      ![Секрет клиента](media/open-text-directory-services-provisioning-tutorial/client-secret.png)

5. Создайте раздел для пользователей и групп, которые будут синхронизированы из Azure AD.

      ![Страница раздела](media/open-text-directory-services-provisioning-tutorial/partition.png)

6. Предоставьте административные права клиенту OAuth, созданному в разделе, который будете использоваться для синхронизируемых пользователей и групп Azure AD.    
      * Раздел -> Действия ->Изменение администраторов

      ![Страница администратора](media/open-text-directory-services-provisioning-tutorial/administrator.png)

5. Секретный токен необходимо получить и настроить в Azure AD. Для этого можно воспользоваться любым клиентским приложением HTTP. Ниже приведены шаги по получению с помощью приложения Swagger API, включенного в OTDS.
      * В веб-браузере перейдите по адресу {OTDS URL}/otdsws/oauth2
      * Перейдите в раздел /token и щелкните значок блокировки в правом верхнем углу. Введите идентификатор клиента OAuth и полученный ранее секрет в качестве имени пользователя и пароля соответственно. Нажмите кнопку "Авторизовать".

      ![Кнопка авторизации](media/open-text-directory-services-provisioning-tutorial/authorization.png)

6. Выберите **client_credentials** для grant_type и щелкните **Выполнить**.

      ![Кнопка выполнения](media/open-text-directory-services-provisioning-tutorial/execute.png)

7. Маркер доступа в ответе следует использовать в поле **Секретный токен** в Azure AD.

      ![Маркер доступа](media/open-text-directory-services-provisioning-tutorial/access-token.png)

## <a name="step-3-add-opentext-directory-services-from-the-azure-ad-application-gallery"></a>Шаг 3. Добавление OpenText Directory Services из коллекции приложений Azure AD

Добавьте OpenText Directory Services из коллекции приложений Azure AD, чтобы начать управление подготовкой для OpenText Directory Services. Если вы ранее настроили единый вход в OpenText Directory Services, можно использовать это же приложение. Однако при первоначальном тестировании интеграции рекомендуется создать отдельное приложение. Дополнительные сведения о добавлении приложения из коллекции см. [здесь](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Шаг 4. Определение пользователей для включения в область подготовки 

Служба подготовки Azure AD позволяет определить пользователей, которые будут подготовлены, на основе назначения приложению и (или) атрибутов пользователя или группы. Если вы решили определить пользователей на основе назначения, выполните следующие [действия](../manage-apps/assign-user-or-group-access-portal.md), чтобы назначить пользователей и группы приложению. Если вы решили указать, кто именно будет подготовлен, на основе одних только атрибутов пользователя или группы, можете использовать фильтр задания области, как описано [здесь](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* При назначении пользователей и групп для OpenText Directory Services необходимо выбрать роль, отличающуюся от роли **Доступ по умолчанию**. Пользователи с ролью "Доступ по умолчанию" исключаются из подготовки и будут помечены в журналах подготовки как не назначенные явно. Кроме того, если эта роль является единственной, доступной в приложении, можно [изменить манифест приложения](../develop/howto-add-app-roles-in-azure-ad-apps.md), чтобы добавить дополнительные роли. 

* Начните с малого. Протестируйте небольшой набор пользователей и групп, прежде чем выполнять развертывание для всех. Если в область подготовки включены назначенные пользователи и группы, проверьте этот механизм, назначив приложению одного или двух пользователей либо одну или две группы. Если в область включены все пользователи и группы, можно указать [фильтр области на основе атрибутов](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-opentext-directory-services"></a>Шаг 5. Настройка автоматической подготовки пользователей в OpenText Directory Services 

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в TestApp на основе их назначений в Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-opentext-directory-services-in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей в OpenText Directory Services, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **OpenText Directory Services**.

    ![Ссылка на OpenText Directory Services в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Вкладка "Подготовка"](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Вкладка "Подготовка" с выделенным значением "Авто"](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите URL-адрес клиента OpenText Directory Services.
   * URL-адрес без указания клиента: {OTDS URL}/scim/{partitionName}
   * URL-адрес с указанием клиента: {OTDS URL}/otdstenant/{tenantID}/scim/{partitionName}

6. Введите секретный токен, полученный на шаге 2. Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к OpenText Directory Services. Если установить подключение не удалось, убедитесь, что у учетной записи OpenText Directory Services есть разрешения администратора, и повторите попытку.

      ![Токен](common/provisioning-testconnection-tenanturltoken.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Щелкните **Сохранить**.

8. В разделе **Сопоставления** выберите **Синхронизировать пользователей Azure Active Directory с OpenText Directory Services**.

9. В разделе **Сопоставления атрибутов** просмотрите пользовательские атрибуты, которые синхронизируются из Azure AD в OpenText Directory Services. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в OpenText Directory Services для операций обновления. Если вы решили изменить [целевой атрибут сопоставления](../app-provisioning/customize-application-attributes.md), сначала убедитесь, что API OpenText Directory Services поддерживает фильтрацию пользователей по этому атрибуту. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

   |attribute|Тип|
   |---|---|
   |userName|Строка|
   |active|Логическое|
   |displayName|Строка|
   |title|Строка|
   |emails[type eq "work"].value|Строка|
   |preferredLanguage|Строка|
   |name.givenName|Строка|
   |name.familyName|Строка|
   |name.formatted|Строка|
   |addresses[type eq "work"].formatted|Строка|
   |addresses[type eq "work"].streetAddress|Строка|
   |addresses[type eq "work"].locality|Строка|
   |addresses[type eq "work"].region|Строка|
   |addresses[type eq "work"].postalCode|Строка|
   |addresses[type eq "work"].country|Строка|
   |phoneNumbers[type eq "work"].value|Строка|
   |phoneNumbers[type eq "mobile"].value|Строка|
   |phoneNumbers[type eq "fax"].value|Строка|
   |externalId|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:employeeNumber|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|Справочник| 

10. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Groups to OpenText Directory Services** (Синхронизировать группы Azure Active Directory с OpenText Directory Services).

11. В разделе **Сопоставления атрибутов** просмотрите атрибуты группы, которые синхронизируются из Azure AD в OpenText Directory Services. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления групп в OpenText Directory Services в операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

      |attribute|Тип|
      |---|---|
      |displayName|Строка|
      |externalId|Строка|
      |members|Справочник|

12. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Чтобы включить службу подготовки Azure AD для OpenText Directory Services, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

14. Определите пользователей и (или) группы для подготовки в OpenText Directory Services, выбрав нужные значения в поле **Область** раздела **Параметры**.

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

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)