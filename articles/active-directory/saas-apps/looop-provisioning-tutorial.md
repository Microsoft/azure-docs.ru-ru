---
title: Руководство по настройке автоматической подготовки пользователей в Looop с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить Azure Active Directory для автоматической подготовки и отмены подготовки учетных записей пользователей в Looop.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/19/2019
ms.author: Zhchia
ms.openlocfilehash: 528003ac482da6f254bf437321c70c389d23844b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94835090"
---
# <a name="tutorial-configure-looop-for-automatic-user-provisioning"></a>Руководство по настройке автоматической подготовки пользователей в Looop

В этом руководстве описаны шаги, которые нужно выполнить в Looop и Azure Active Directory (Azure AD), чтобы настроить Azure AD для автоматической подготовки и отзыва пользователей и (или) групп в Looop.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* клиент Azure AD;
* [клиент Looop](https://www.looop.co/pricing/);
* учетная запись пользователя в Looop с разрешениями администратора.

## <a name="assign-users-to-looop"></a>Назначение пользователей приложению Looop

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется понятие "назначение". В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи и (или) группы в Azure AD должны иметь доступ к Looop. Когда этот вопрос будет решен, этих пользователей и (или) группы можно будет назначить приложению Looop, следуя приведенным ниже указаниям.

* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-looop"></a>Важные рекомендации по назначению пользователей приложению Looop

* Рекомендуем назначить в Looop одного пользователя Azure AD для тестирования конфигурации автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя для Looop в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (если доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="set-up-looop-for-provisioning"></a>Настройка подготовки в Looop

Прежде чем настраивать автоматическую подготовку пользователей в Azure AD для Looop, нужно получить из Looop определенные сведения о подготовке.

1. Войдите в [консоль администрирования Looop](https://app.looop.co/#/login) и выберите элемент **Account** (Учетная запись). В разделе **Account Settings** (Параметры учетной записи) выберите **Authentication** (Проверка подлинности).

    :::image type="content" source="media/looop-provisioning-tutorial/admin.png" alt-text="Снимок экрана: консоль администрирования Looop. На ней выделена и открыта вкладка Account (Учетная запись). В разделе Account settings (Параметры учетной записи) выделен элемент Authentication (Проверка подлинности)." border="false":::

2. Создайте новый маркер, щелкнув **Reset Token** (Сбросить маркер) в разделе **SCIM Integration** (Интеграция SCIM).

    :::image type="content" source="media/looop-provisioning-tutorial/resettoken.png" alt-text="Снимок экрана: раздел SCIM Integration (Интеграция SCIM) на странице консоли администрирования Looop. Выделена кнопка Reset token (Сбросить маркер)." border="false":::

3. Скопируйте значения **SCIM Endpoint** (Конечная точка SCIM) и **Token** (Маркер). Эти значения вы введете в поля **URL-адрес клиента** и **Секретный маркер** на вкладке "Подготовка" для приложения Looop на портале Azure. 

    ![Создание токена в Looop](media/looop-provisioning-tutorial/token.png)

## <a name="add-looop-from-the-gallery"></a>Добавление Looop из коллекции

Чтобы настроить автоматическую подготовку пользователей для Looop в Azure AD, необходимо добавить Looop из коллекции приложений Azure AD в список управляемых приложений SaaS.

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Looop**, затем выберите **Looop** на панели результатов. 

    ![Looop в списке результатов](common/search-new-app.png)

5. Нажмите кнопку **Sign-up for Looop** (Зарегистрироваться в Looop), которая перенаправит вас на страницу входа в Looop. 

    ![Добавление OIDC для Looop](media/looop-provisioning-tutorial/signup.png)

6. Так как Looop — это приложение OpenIDConnect, для входа в Looop необходимо выбрать рабочую учетную запись Майкрософт.

    ![Вход OIDC в Looop](media/looop-provisioning-tutorial/msftlogin.png)

7. После успешной проверки подлинности дайте согласие на странице согласия. Теперь приложение будет автоматически добавлено к вашему клиенту и вы перейдете в учетную запись Looop.

    ![Согласие для OIDC в Looop](media/looop-provisioning-tutorial/accept.png)

## <a name="configure-automatic-user-provisioning-to-looop"></a>Настройка автоматической подготовки пользователей в Looop 

В этом разделе описывается, как настроить в службе подготовки Azure AD создание, обновление и отключение пользователей и (или) групп в Looop на основе их назначений в Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-looop-in-azure-ad"></a>Для настройки автоматической подготовки пользователей в Azure AD для Looop сделайте следующее.

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Looop**.

    ![Ссылка на Looop в списке "Приложения"](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите значение `https://<organisation_domain>.looop.co/scim/v2` в поле **URL-адрес клиента**. Например, `https://demo.looop.co/scim/v2`. Введите в поле **Секретный токен** значение, которое вы ранее получили из Looop и сохранили. Щелкните **Проверить подключение** и убедитесь, что Azure AD может подключиться к Looop. Если установить подключение не удалось, убедитесь, что у учетной записи Looop есть разрешения администратора, и повторите попытку.

    ![URL-адрес клиента + токен](common/provisioning-testconnection-tenanturltoken.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Выберите команду **Сохранить**.

8. В разделе **Сопоставления** выберите **Синхронизировать пользователей Azure Active Directory с Looop**.

    ![Сопоставления пользователей Looop](media/looop-provisioning-tutorial/usermappings.png)

9. В разделе **Сопоставление атрибутов** просмотрите пользовательские атрибуты, которые синхронизируются из Azure AD в Looop. Атрибуты, отмеченные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в Looop при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

   |attribute|Тип|Поддерживается для фильтрации|
   |---|---|---|
   |userName|Строка|&check;|
   |active|Логическое|
   |emails[type eq "work"].value|Строка|
   |name.givenName|Строка|
   |name.familyName|Строка|
   |externalId|Строка|
   |urn:ietf:params:scim:schemas:extension:Looop:2.0:User:area|Строка|
   |urn:ietf:params:scim:schemas:extension:Looop:2.0:User:custom_1|Строка|
   |urn:ietf:params:scim:schemas:extension:Looop:2.0:User:custom_2|Строка|
   |urn:ietf:params:scim:schemas:extension:Looop:2.0:User:custom_3|Строка|
   |urn:ietf:params:scim:schemas:extension:Looop:2.0:User:department|Строка|
   |urn:ietf:params:scim:schemas:extension:Looop:2.0:User:employee_id|Строка|
   |urn:ietf:params:scim:schemas:extension:Looop:2.0:User:location|Строка|
   |urn:ietf:params:scim:schemas:extension:Looop:2.0:User:position|Строка|
   |urn:ietf:params:scim:schemas:extension:Looop:2.0:User:startAt|Строка|

10. В разделе **Сопоставления** выберите параметр **Синхронизировать группы Azure Active Directory с Meta Networks Connector**.

    ![Сопоставления групп в Looop](media/looop-provisioning-tutorial/groupmappings.png)

11. В разделе **Сопоставления атрибутов** просмотрите атрибуты групп, которые синхронизируются из Azure AD в Meta Networks Connector. Атрибуты, выбранные как свойства с меткой **Сопоставление**, используются для сопоставления групп в Meta Networks Connector при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    |attribute|Тип|Поддерживается для фильтрации|
    |---|---|---|
    |displayName|Строка|&check;|
    |members|Справочник|
    |externalId|Строка|


10. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Чтобы включить службу подготовки Azure AD для Looop, в разделе **Параметры** измените значение параметра **Состояние подготовки** на **Включено**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

12. Укажите пользователей и (или) группы для подготовки в Looop, выбрав нужные значения в поле **Область** в разделе **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

13. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Сведения о синхронизации** можно отслеживать ход выполнения, а по дополнительным ссылкам просмотреть отчет о подготовке с полным списком действий, выполняемых службой подготовки Azure AD в приложении Looop.

Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)


