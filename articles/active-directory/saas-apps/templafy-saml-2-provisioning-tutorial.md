---
title: Руководство. Настройка Templafy SAML2 для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить Azure Active Directory для автоматической подготовки и отзыва учетных записей пользователей в Templafy SAML2.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.assetid: 8a966ef5-e364-435b-9e29-3caf27ffb498
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/19/2021
ms.author: zhchia
ms.openlocfilehash: 4cc89c1ad94ae738b25e4ea18d5fe687880969c1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101648243"
---
# <a name="tutorial-configure-templafy-saml2-for-automatic-user-provisioning"></a>Руководство. Настройка Templafy SAML2 для автоматической подготовки пользователей

В этом руководстве описаны действия, которые нужно выполнить в Templafy SAML2 и Azure Active Directory (Azure AD), чтобы настроить Azure AD для автоматической подготовки и отзыва пользователей и групп в Templafy SAML2.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* Клиент Azure AD.
* [Клиент Templafy](https://www.templafy.com/pricing/).
* учетная запись пользователя в Templafy с разрешениями администратора.

## <a name="step-1-plan-your-provisioning-deployment"></a>Шаг 1. Планирование развертывания для подготовки
1. Узнайте, [как работает служба подготовки](../app-provisioning/user-provisioning.md).
2. Определите, кто будет находиться в [области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Определите, какие данные следует [сопоставлять между Azure AD и Templafy SAML2](../app-provisioning/customize-application-attributes.md). 

## <a name="assigning-users-to-templafy-saml2"></a>Назначение пользователей Templafy SAML2

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая *назначением*. В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи или группы в Azure AD должны иметь доступ к Templafy SAML2. Когда этот вопрос будет решен, таких пользователей или группы можно будет назначить приложению Templafy SAML2, следуя инструкциям:
* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-templafy-saml2"></a>Важные рекомендации по назначению пользователей в Templafy SAML2

* Рекомендуется назначить одного пользователя Azure AD в Templafy SAML2 для тестирования конфигурации автоматической подготовки пользователей. Дополнительных пользователей и (или) группы можно назначить позднее.

* При назначении пользователя в Templafy SAML2 в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (если это доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="step-2-configure-templafy-saml2-to-support-provisioning-with-azure-ad"></a>Шаг 2. Настройка Templafy SAML2 для поддержки подготовки с помощью Azure AD

Прежде чем настраивать автоматическую подготовку пользователей в Azure AD для Templafy SAML2, нужно включить подготовку SCIM для Templafy SAML2.

1. Войдите в консоль администрирования Templafy. Щелкните **Administration** (Администрирование).

    ![Консоль администрирования Templafy](media/templafy-saml-2-provisioning-tutorial/templafy-admin.png)

2. Щелкните **Authentication Method** (Способ проверки подлинности).

    ![Снимок экрана раздела администрирования Templafy с выбранным параметром Authentication method (Способ проверки подлинности).](media/templafy-saml-2-provisioning-tutorial/templafy-auth.png)

3. Скопируйте значение **ключа API SCIM**. Его нужно будет ввести в поле **Секретный токен** на вкладке "Подготовка" для приложения Templafy SAML2 на портале Azure.

    ![Снимок экрана, на котором показан ключ API SCIM.](media/templafy-saml-2-provisioning-tutorial/templafy-token.png)

## <a name="step-3-add-templafy-saml2-from-the-gallery"></a>Шаг 3. Добавление Templafy SAML2 из коллекции

Чтобы настроить Templafy SAML2 для автоматической подготовки пользователей в Azure AD, необходимо добавить Templafy SAML2 из коллекции приложений Azure AD в список управляемых приложений SaaS.

**Чтобы добавить Templafy SAML2 из коллекции приложений Azure AD, сделайте следующее:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Templafy SAML2**, выберите **Templafy SAML2** в области результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Templafy SAML2 в списке результатов](common/search-new-app.png)

## <a name="step-4-configure-automatic-user-provisioning-to-templafy-saml2"></a>Шаг 4. Настройка автоматической подготовки пользователей в Templafy SAML2 

В этом разделе объясняется, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в Templafy SAML2 на основе их назначений в Azure AD.

> [!TIP]
> Для Templafy также можно включить единый вход на основе SAML. Для этого выполните инструкции, приведенные в [руководстве по единому входу в Templafy](templafy-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности дополняют друг друга.

### <a name="to-configure-automatic-user-provisioning-for-templafy-saml2-in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей Templafy SAML2 в Azure AD, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Templafy SAML2**.

    ![Ссылка на Templafy SAML2 в списке "Приложения"](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите значение `https://scim.templafy.com/scim` в поле **URL-адрес клиента**. Введите полученное ранее значение **ключа API SCIM** в поле **Секретный токен**. Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к Templafy. Если установить подключение не удалось, убедитесь, что у учетной записи Templafy есть разрешения администратора, и повторите попытку.

    ![URL-адрес клиента + токен](common/provisioning-testconnection-tenanturltoken.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Выберите команду **Сохранить**.

8. В разделе **Сопоставления** выберите **Синхронизировать пользователей Azure Active Directory с Templafy SAML2**.

    ![Сопоставления пользователей Templafy SAML2](media/templafy-saml-2-provisioning-tutorial/user-mapping.png)

9. В разделе **Сопоставления атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы из Azure AD в Templafy SAML2. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в Templafy SAML2 для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

   |attribute|Тип|Поддерживается для фильтрации|
   |---|---|---|
   |userName|Строка|&check;|
   |active|Логическое|
   |displayName|Строка|
   |title|Строка|
   |preferredLanguage|Строка|
   |name.givenName|Строка|
   |name.familyName|Строка|
   |phoneNumbers[type eq "work"].value|Строка|
   |phoneNumbers[type eq "mobile"].value|Строка|
   |phoneNumbers[type eq "fax"].value|Строка|
   |externalId|Строка|
   |addresses[type eq "work"].locality|Строка|
   |addresses[type eq "work"].postalCode|Строка|
   |addresses[type eq "work"].region|Строка|
   |addresses[type eq "work"].streetAddress|Строка|
   |addresses[type eq "work"].country|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:organization|Строка|

10. В разделе **Сопоставления** выберите **Синхронизировать группы Azure Active Directory с Templafy**.

    ![Сопоставления групп в Templafy SAML2](media/templafy-saml-2-provisioning-tutorial/group-mapping.png)

11. В разделе **Сопоставления атрибутов** просмотрите атрибуты групп, которые синхронизированы из Azure AD в Templafy SAML2. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления групп в Templafy SAML2 для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

      |attribute|Тип|Поддерживается для фильтрации|
      |---|---|---|
      |displayName|Строка|&check;|
      |members|Справочник|
      |externalId|Строка|      


12. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Чтобы включить службу подготовки Azure AD для Templafy SAML2, в разделе **Параметры** измените значение параметра **Состояние подготовки** на **Включено**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

14. Определите пользователей или группы для подготовки в Templafy SAML2, выбрав нужные значения в поле **Область** в разделе **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

15. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

    После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Сведения о синхронизации** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с Templafy SAML2.

## <a name="step-5-monitor-your-deployment"></a>Шаг 5. Мониторинг развертывания
После настройки подготовки используйте следующие ресурсы для мониторинга развертывания.

* Используйте [журналы подготовки](../reports-monitoring/concept-provisioning-logs.md), чтобы определить, какие пользователи были подготовлены успешно или неудачно.
* Используйте [индикатор выполнения](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md), чтобы узнать состояние цикла подготовки и приблизительное время до его завершения.
* Если конфигурация подготовки, вероятно, находится в неработоспособном состоянии, приложение перейдет в карантин. Дополнительные сведения о режимах карантина см. [здесь](../app-provisioning/application-provisioning-quarantine-status.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)