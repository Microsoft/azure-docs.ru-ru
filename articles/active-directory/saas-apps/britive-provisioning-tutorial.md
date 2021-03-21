---
title: Руководство. Настройка Бритиве для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как автоматически подготавливать и отзывать учетные записи пользователей из Azure AD в Бритиве.
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: 622688b3-9d20-482e-aab9-ce2a1f01e747
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/05/2021
ms.author: Zhchia
ms.openlocfilehash: 8bebcb49bc7bf31614a161c08d33d5910679b614
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103225767"
---
# <a name="tutorial-configure-britive-for-automatic-user-provisioning"></a>Учебник. Настройка Бритиве для автоматической подготовки пользователей

В этом руководстве описываются действия, которые необходимо выполнить в Бритиве и Azure Active Directory (Azure AD) для настройки автоматической подготовки пользователей. При настройке Azure AD автоматически подготавливает и отменяет подготовку пользователей и групп для [бритиве](https://www.britive.com/) с помощью службы подготовки Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Поддерживаемые возможности
> [!div class="checklist"]
> * Создание пользователей в Бритиве
> * Удалить пользователей в Бритиве, когда им больше не нужен доступ
> * Синхронизация атрибутов пользователей между Azure AD и Бритиве
> * Подготавливайте группы и членство в группах в Бритиве
> * [Единый вход](britive-tutorial.md) в бритиве (рекомендуется)

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* [Клиент Azure AD.](../develop/quickstart-create-new-tenant.md) 
* Учетная запись пользователя в Azure AD с [разрешением](../roles/permissions-reference.md) настраивать подготовку (например, администратор приложений, администратор облачных приложений, владелец приложения или глобальный администратор). 
* Клиент [бритиве](https://www.britive.com/) .
* Учетная запись пользователя в Бритиве с разрешениями администратора.


## <a name="step-1-plan-your-provisioning-deployment"></a>Шаг 1. Планирование развертывания для подготовки
1. Узнайте, [как работает служба подготовки](../app-provisioning/user-provisioning.md).
1. Определите, кто будет находиться в [области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
1. Определите, какие данные должны [сопоставляться между Azure AD и бритиве](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-britive-to-support-provisioning-with-azure-ad"></a>Шаг 2. Настройка Бритиве для поддержки подготовки с помощью Azure AD

Приложение должно быть настроено вручную с помощью приведенных ниже действий.
1. Вход в приложение Бритиве с правами администратора
1. Щелкните **Администратор->администрирование пользователей — >поставщики удостоверений** .
1. Щелкните **Добавить поставщик удостоверений**. Введите имя и описание. Нажмите кнопку Добавить поставщик удостоверений.

    ![Поставщик удостоверений](media/britive-provisioning-tutorial/identity.png)

1. Отобразится страница конфигурации, похожая на ту, которая показана ниже.

    ![Страница «Конфигурация»](media/britive-provisioning-tutorial/configuration.png)

1. Перейдите на вкладку **scim** . Измените поставщик SCIM с универсального на Azure и сохраните изменения. Скопируйте URL-адрес SCIM и запишите его. Эти значения будут указаны в полях **URL-адрес клиента** на вкладке Подготовка приложения бритиве в портал Azure.

    ![Страница SCIM](media/britive-provisioning-tutorial/scim.png)

1. Щелкните **создать токен**. Выберите допустимость маркера, а затем нажмите кнопку создать токен.

    ![Создать токен](media/britive-provisioning-tutorial/create-token.png)

1. Скопируйте созданный маркер и запишите его. Щелкните ОК. Обратите внимание, что пользователь больше не сможет увидеть маркер. Нажмите кнопку Re-Create, чтобы при необходимости создать новый маркер. Эти значения будут указаны в полях **секретный токен** и URL-адрес клиента на вкладке Подготовка в приложении в портал Azure.

    ![Копировать токен](media/britive-provisioning-tutorial/copy-token.png) 


## <a name="step-3-add-britive-from-the-azure-ad-application-gallery"></a>Шаг 3. Добавление Бритиве из коллекции приложений Azure AD

Добавьте Бритиве из коллекции приложений Azure AD, чтобы начать управление подготовкой в Бритиве. Если вы ранее настроили Бритиве для единого входа, вы можете использовать то же приложение. Однако при первоначальном тестировании интеграции рекомендуется создать отдельное приложение. Дополнительные сведения о добавлении приложения из коллекции см. [здесь](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Шаг 4. Определение пользователей для включения в область подготовки 

Служба подготовки Azure AD позволяет определить пользователей, которые будут подготовлены, на основе назначения приложению и (или) атрибутов пользователя или группы. Если вы решили определить пользователей на основе назначения, выполните следующие [действия](../manage-apps/assign-user-or-group-access-portal.md), чтобы назначить пользователей и группы приложению. Если вы решили указать, кто именно будет подготовлен, на основе одних только атрибутов пользователя или группы, можете использовать фильтр задания области, как описано [здесь](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* При назначении пользователей и групп для Бритиве необходимо выбрать роль, отличную от **доступа по умолчанию**. Пользователи с ролью "Доступ по умолчанию" исключаются из подготовки и будут помечены в журналах подготовки как не назначенные явно. Кроме того, если эта роль является единственной, доступной в приложении, можно [изменить манифест приложения](../develop/howto-add-app-roles-in-azure-ad-apps.md), чтобы добавить дополнительные роли. 

* Начните с малого. Протестируйте небольшой набор пользователей и групп, прежде чем выполнять развертывание для всех. Если в область подготовки включены назначенные пользователи и группы, проверьте этот механизм, назначив приложению одного или двух пользователей либо одну или две группы. Если в область включены все пользователи и группы, можно указать [фильтр области на основе атрибутов](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-britive"></a>Шаг 5. Настройка автоматической подготовки пользователей в Бритиве 

В этом разделе описано, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в Бритиве на основе назначений пользователей и групп в Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-britive-in-azure-ad"></a>Чтобы настроить автоматическую подготовку учетных записей пользователей для Бритиве в Azure AD, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

1. В списке приложений выберите **Britive**.

    ![Ссылка на Бритиве в списке "приложения"](common/all-applications.png)

1. Выберите вкладку **Подготовка**.

    ![Вкладка "Подготовка"](common/provisioning.png)

1. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Вкладка "Подготовка" с выделенным значением "Авто"](common/provisioning-automatic.png)

1. В разделе **учетные данные администратора** введите URL-адрес клиента Бритиве и маркер секрета. Щелкните **проверить подключение** , чтобы убедиться, что Azure AD может подключиться к бритиве. Если подключение не выполняется, убедитесь, что у учетной записи Бритиве есть разрешения администратора, и повторите попытку.

    ![Токен](common/provisioning-testconnection-tenanturltoken.png)

1. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Почтовое уведомление](common/provisioning-notification-email.png)

1. Щелкните **Сохранить**.

1. В разделе **сопоставления** выберите **синхронизировать Azure Active Directory пользователей с бритиве**.

1. Проверьте атрибуты пользователя, которые синхронизированы из Azure AD в Бритиве в разделе " **сопоставление атрибутов** ". Атрибуты, выбранные как свойства **Matching** , используются для сопоставления учетных записей пользователей в бритиве для операций обновления. Если вы решили изменить [соответствующий целевой атрибут](../app-provisioning/customize-application-attributes.md), необходимо убедиться, что API бритиве поддерживает фильтрацию пользователей на основе этого атрибута. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

   |attribute|Тип|Поддерживается для фильтрации|
   |---|---|---|
   |userName|Строка|&check;
   |active|Логическое|
   |displayName|Строка|
   |title|Строка|
   |externalId|Строка|
   |preferredLanguage|Строка|
   |name.givenName|Строка|
   |name.familyName|Строка|
   |nickName|Строка|
   |userType|Строка|
   |локаль|Строка|
   |timezone|Строка|
   |emails[type eq "home"].value|Строка|
   |emails[type eq "other"].value|Строка|
   |emails[type eq "work"].value|Строка|
   |phoneNumbers[type eq "home"].value|Строка|
   |phoneNumbers[type eq "other"].value|Строка|
   |phoneNumbers[type eq "pager"].value|Строка|
   |phoneNumbers[type eq "work"].value|Строка|
   |phoneNumbers[type eq "mobile"].value|Строка|
   |phoneNumbers[type eq "fax"].value|Строка|
   |addresses[type eq "work"].formatted|Строка|
   |addresses[type eq "work"].streetAddress|Строка|
   |addresses[type eq "work"].locality|Строка|
   |addresses[type eq "work"].region|Строка|
   |addresses[type eq "work"].postalCode|Строка|
   |addresses[type eq "work"].country|Строка|
   |addresses[type eq "home"].formatted|Строка|
   |addresses[type eq "home"].streetAddress|Строка|
   |addresses[type eq "home"].locality|Строка|
   |addresses[type eq "home"].region|Строка|
   |addresses[type eq "home"].postalCode|Строка|
   |addresses[type eq "home"].country|Строка|
   |addresses[type eq "other"].formatted|Строка|
   |addresses[type eq "other"].streetAddress|Строка|
   |addresses[type eq "other"].locality|Строка|
   |addresses[type eq "other"].region|Строка|
   |addresses[type eq "other"].postalCode|Строка|
   |addresses[type eq "other"].country|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:employeeNumber|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:costCenter|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:organization|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:division|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|Справочник|


1. В разделе **сопоставления** выберите **синхронизировать Azure Active Directory группы с бритиве**.

1. Проверьте атрибуты группы, которые синхронизированы из Azure AD в Бритиве в разделе " **сопоставление атрибутов** ". Атрибуты, выбранные как свойства **Matching** , используются для сопоставления групп в бритиве для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

      |attribute|Тип|Поддерживается для фильтрации|
      |---|---|---|
      |displayName|Строка|&check;
      |externalId|Строка|
      |members|Справочник|

1. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

1. Чтобы включить службу подготовки Azure AD для Бритиве, измените значение параметра **состояние подготовки** на **включено** в разделе **Параметры** .

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

1. Определите пользователей и (или) группы, которые вы хотите подготавливать к Бритиве, выбрав нужные значения в **области** в разделе **Параметры** .

    ![Область действия подготовки](common/provisioning-scope.png)

1. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

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