---
title: Руководство по настройке автоматической подготовки пользователей в Clarizen One с помощью Azure Active Directory | Документация Майкрософт
description: Сведения о том, как настроить в Azure AD автоматическую подготовку и отзыв учетных записей пользователей для Clarizen One.
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: d8021105-eb5b-4a20-8739-f02e0e22c147
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 10/01/2020
ms.author: Zhchia
ms.openlocfilehash: f3a19d3c3bf3e4340bb36fd683453541fa15eb6c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101650846"
---
# <a name="tutorial-configure-clarizen-one-for-automatic-user-provisioning"></a>Руководство по настройке автоматической подготовки пользователей в Clarizen One

В этом руководстве описаны действия, которые нужно выполнить в Clarizen One и Azure Active Directory (Azure AD) для настройки автоматической подготовки пользователей. После настройки Azure AD автоматически осуществляет и отменяет подготовку пользователей и групп для [Clarizen One](https://www.clarizen.com/) с помощью службы подготовки Azure AD. Сведения о возможностях и принципе работы этой службы, а также часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и их отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="capabilities-supported"></a>Поддерживаемые возможности

> [!div class="checklist"]
> * Создание пользователей в Clarizen One.
> * Удаление из Clarizen One пользователей, которым уже не нужен доступ.
> * Синхронизация атрибутов пользователей между Azure AD и Clarizen One.
> * Подготовка групп и членства в группах в Clarizen One.
> * Мы также рекомендуем [настроить единый вход](./clarizen-tutorial.md) для Clarizen One.

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* [Клиент Azure AD.](../develop/quickstart-create-new-tenant.md)
* Учетная запись пользователя в Azure AD с [разрешением](../roles/permissions-reference.md) на настройку подготовки. Подходят следующие роли: администратор приложений, администратор облачных приложений, владелец приложений или глобальный администратор.
* Учетная запись пользователя Clarizen One с [разрешениями](https://success.clarizen.com/hc/articles/360011833079-API-Keys-Support) **Integration User** (Пользователь интеграции) и **Lite Admin** (Администратор Lite).

## <a name="step-1-plan-your-provisioning-deployment"></a>Шаг 1. Планирование развертывания для подготовки

1. Узнайте, [как работает служба подготовки](../app-provisioning/user-provisioning.md).
1. Определите, кто будет находиться в [области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
1. Определите, какие данные следует [сопоставлять между Azure AD и Clarizen One](../app-provisioning/customize-application-attributes.md).

## <a name="step-2-configure-clarizen-one-to-support-provisioning-with-azure-ad"></a>Шаг 2. Настройка Clarizen One для поддержки подготовки с помощью Azure AD

1. Выберите один из следующих URL-адресов клиента в зависимости от используемых среды и центра обработки данных Clarizen One.
      * Рабочий центр обработки данных в США: https://servicesapp2.clarizen.com/scim/v2
      * Рабочий центр обработки данных в Европе: https://serviceseu1.clarizen.com/scim/v2
      * Тестовый центр обработки данных в США: https://servicesapp.clarizentb.com/scim/v2
      * Тестовый центр обработки данных в Европе: https://serviceseu.clarizentb.com/scim/v2

1. Сгенерируйте [ключ API](https://success.clarizen.com/hc/articles/360011833079-API-Keys-Support). Его нужно будет ввести в поле **Секретный токен** на вкладке **Подготовка** для приложения Clarizen One на портале Azure.

## <a name="step-3-add-clarizen-one-from-the-azure-ad-application-gallery"></a>Шаг 3. Добавление Clarizen One из коллекции приложений Azure AD

Добавьте Clarizen One из коллекции приложений Azure AD, чтобы начать управление подготовкой в Clarizen One. Если вы уже настроили единый вход в Clarizen One, можете использовать то же приложение. При начальном тестировании интеграции создайте отдельное приложение. Дополнительные сведения о добавлении приложения в клиент Azure AD из коллекции см. [здесь](../manage-apps/add-application-portal.md).

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Шаг 4. Определение пользователей для включения в область подготовки

Служба подготовки Azure AD позволяет определить пользователей, которые будут подготовлены, назначая их приложению и (или) присваивая атрибуты пользователям или группам. Если вы решили указать, кто именно будет подготовлен к работе в приложении, на основе назначения, выполните процедуру из статьи [Управление назначением пользователей для приложения в Azure Active Directory](../manage-apps/assign-user-or-group-access-portal.md), чтобы назначить пользователей и группы приложению. Если вы решили указать, кто именно будет подготовлен, на основе одних только атрибутов пользователя или группы, можете использовать фильтр области, как описано в статье [Подготовка приложений на основе атрибутов с использованием фильтров области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

* При назначении пользователей и групп для Clarizen One необходимо выбрать роль, отличную от роли **Доступ по умолчанию**. Пользователи с такой ролью исключаются из подготовки и будут помечены в журналах подготовки как не имеющие разрешений. Если в приложении доступна только эта роль, можно [изменить манифест приложения](../develop/howto-add-app-roles-in-azure-ad-apps.md), чтобы добавить другие роли.
* Начните с малого. Протестируйте небольшой набор пользователей и групп, прежде чем выполнять развертывание для всех. Если в область подготовки включены назначенные пользователи и группы, для сохранения контроля назначьте приложению одного или двух пользователей либо одну или две группы. Если в область включены все пользователи и группы, можно указать [фильтр области на основе атрибутов](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

## <a name="step-5-configure-automatic-user-provisioning-to-clarizen-one"></a>Шаг 5. Настройка автоматической подготовки пользователей в Clarizen One

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей или групп в Clarizen One на основе их назначений в Azure AD.

### <a name="configure-automatic-user-provisioning-for-clarizen-one-in-azure-ad"></a>Настройка автоматической подготовки пользователей в Azure AD для Clarizen One

1. Войдите на [портал Azure](https://portal.azure.com). Щелкните **Корпоративные приложения** > **Все приложения**.

      ![Снимок экрана: панель "Корпоративные приложения".](common/enterprise-applications.png)

1. В списке приложений выберите **Clarizen One**.

      ![Снимок экрана, на котором показана ссылка на Clarizen One в списке приложений.](common/all-applications.png)

1. Выберите вкладку **Подготовка**.

      ![Снимок экрана: вкладка "Подготовка".](common/provisioning.png)

1. Для параметра **Режим подготовки к работе** выберите значение **Автоматически**.

      ![Снимок экрана: вкладка "Подготовка", где выбран вариант "Автоматически".](common/provisioning-automatic.png)

1. В разделе **Учетные данные администратора** укажите **Tenant URL** (URL-адрес клиента) и **Secret Token** (Токен секрета) из Clarizen One. Щелкните **Проверить подключение** и убедитесь, что Azure AD может подключиться к Clarizen One. Если установить подключение не удалось, убедитесь, что у учетной записи Clarizen One есть разрешения администратора, и повторите попытку.

    ![Снимок экрана, на котором показано поле "Секретный токен".](common/provisioning-testconnection-tenanturltoken.png)

1. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки. Установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Снимок экрана, на котором показано поле "Почтовое уведомление".](common/provisioning-notification-email.png)

1. Нажмите кнопку **Сохранить**.

1. В разделе **Сопоставления** выберите **Синхронизировать пользователей Azure Active Directory с Clarizen One**.

1. В разделе **Сопоставления атрибутов** просмотрите атрибуты пользователей, которые синхронизируются из Azure AD в Clarizen One. Атрибуты, выбранные как свойства **Сопоставление**, используются для сопоставления учетных записей пользователей в Clarizen One при операциях обновления. Если вы измените [соответствующий целевой атрибут](../app-provisioning/customize-application-attributes.md), обязательно убедитесь, что API Clarizen One поддерживает фильтрацию пользователей по этому атрибуту. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

   |attribute|Тип|
   |---|---|
   |userName|Строка|
   |displayName|Строка|
   |active|Логическое|
   |title|Строка|
   |emails[type eq "work"].value|Строка|
   |emails[type eq "home"].value|Строка|
   |emails[type eq "other"].value|Строка|
   |preferredLanguage|Строка|
   |name.givenName|Строка|
   |name.familyName|Строка|
   |name.formatted|Строка|
   |name.honorificPrefix|Строка|
   |name.honorificSuffix|Строка|
   |addresses[type eq "other"].formatted|Строка|
   |addresses[type eq "work"].formatted|Строка|
   |addresses[type eq "work"].country|Строка|
   |addresses[type eq "work"].region|Строка|
   |addresses[type eq "work"].locality|Строка|
   |addresses[type eq "work"].postalCode|Строка|
   |addresses[type eq "work"].streetAddress|Строка|
   |phoneNumbers[type eq "work"].value|Строка|
   |phoneNumbers[type eq "mobile"].value|Строка|
   |phoneNumbers[type eq "fax"].value|Строка|
   |phoneNumbers[type eq "home"].value|Строка|
   |phoneNumbers[type eq "other"].value|Строка|
   |phoneNumbers[type eq "pager"].value|Строка|
   |externalId|Строка|
   |nickName|Строка|
   |локаль|Строка|
   |roles[primary eq "True".type]|Строка|
   |roles[primary eq "True".value]|Строка|
   |timezone|Строка|
   |userType|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:employeeNumber|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|Справочник|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:costCenter|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:division|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:division|Строка|

1. В разделе **Сопоставления** выберите **Синхронизировать пользователей Azure Active Directory с Clarizen One**.

1. В разделе **Сопоставления атрибутов** просмотрите атрибуты групп, которые синхронизируются из Azure AD в Clarizen One. Атрибуты, выбранные как свойства **Сопоставление**, используются для сопоставления групп в Clarizen One при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

      |attribute|Тип|
      |---|---|
      |displayName|Строка|
      |externalId|Строка|
      |members|Справочник|

1. Чтобы настроить фильтры области, ознакомьтесь с инструкциями в статье [Подготовка приложений на основе атрибутов с использованием фильтров области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

1. Чтобы включить службу подготовки Azure AD для Clarizen One, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

      ![Снимок экрана, на котором показано состояние подготовки "Включено".](common/provisioning-toggle-on.png)

1. Определите пользователей или группы для подготовки в Clarizen One, выбрав нужные значения в поле **Область** в разделе **Параметры**.

      ![Снимок экрана, на котором показана область подготовки.](common/provisioning-scope.png)

1. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

      ![Снимок экрана, на котором показано сохранение конфигурации подготовки.](common/provisioning-configuration-save.png)

После этого начнется цикл начальной синхронизации всех пользователей и групп, определенных в поле **Область** в разделе **Параметры**. Начальный цикл занимает больше времени, чем последующие циклы. Пока служба подготовки Azure AD запущена, они выполняются примерно каждые 40 минут.

## <a name="step-6-monitor-your-deployment"></a>Шаг 6. Мониторинг развертывания

Завершив настройку подготовки, используйте следующие ресурсы для мониторинга развертывания.

1. Используйте [журналы подготовки](../reports-monitoring/concept-provisioning-logs.md), чтобы определить, какие пользователи были подготовлены успешно или неудачно.
1. Используйте [индикатор выполнения](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md), чтобы узнать состояние цикла подготовки и близость его завершения.
1. Если конфигурация подготовки, вероятно, находится в неработоспособном состоянии, приложение перейдет в карантин. Дополнительные сведения о состояниях карантина см. в статье [Состояние подготовки приложений в карантине](../app-provisioning/application-provisioning-quarantine-status.md).

## <a name="troubleshooting-tips"></a>Советы по устранению неполадок

Назначая пользователя для приложения Clarizen One из коллекции, выберите только роль **Пользователь**. Следующие роли являются недопустимыми:

* администратор;
* пользователь отчетов по электронной почте;
* внешний пользователь;
* финансовый пользователь;
* пользователь социальных сетей;
* суперпользователь;
* пользователь по времени и расходам.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)