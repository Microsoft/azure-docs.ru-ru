---
title: Учебник по настройке Netskope User Authentication для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить Azure Active Directory для автоматических подготовки и отзыва учетных записей пользователей в Netskope User Authentication.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/07/2019
ms.author: Zhchia
ms.openlocfilehash: 46766a7439185714648572f3f1b9d51ef96abba6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94357480"
---
# <a name="tutorial-configure-netskope-user-authentication-for-automatic-user-provisioning"></a>Учебник по настройке Netskope User Authentication для автоматической подготовки пользователей

В этом учебнике описаны шаги, которые нужно выполнить в Netskope User Authentication и Azure Active Directory (Azure AD), чтобы настроить Azure AD для автоматических подготовки и отзыва пользователей и (или) групп в Netskope User Authentication.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* клиент Azure AD;
* [Клиент Netskope User Authentication](https://www.netskope.com/);
* учетная запись пользователя в Netskope User Authentication с разрешениями администратора.

## <a name="assigning-users-to-netskope-user-authentication"></a>Назначение пользователей для Netskope User Authentication

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая *назначением*. В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи и (или) группы в Azure AD должны иметь доступ к Netskope User Authentication. Когда этот вопрос будет решен, этих пользователей и (или) группы можно будет назначить приложению Netskope User Authentication, выполнив следующие инструкции:
* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-netskope-user-authentication"></a>Важные советы по назначению пользователей для Netskope User Authentication

* Рекомендуется назначить одного пользователя Azure AD в Netskope User Authentication для тестирования конфигурации автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя в Netskope User Authentication в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (если доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="set-up-netskope-user-authentication-for-provisioning"></a>Настройка подготовки в Netskope User Authentication

1. Войдите в [консоль администрирования Netskope User Authentication](https://netskope.goskope.com/). Перейдите к разделу **Домашняя страница > Параметры**.

    ![Экран "Консоль администрирования Netskope User Authentication"](media/netskope-administrator-console-provisioning-tutorial/admin.png)

2.  Перейдите к пункту **Инструменты**. В меню **Инструменты** выберите пункт **Directory Tools (Инструменты каталога) > SCIM INTEGRATION (ИНТЕГРАЦИЯ SCIM)** .

    ![Экран "Инструменты Netskope User Authentication"](media/netskope-administrator-console-provisioning-tutorial/tools.png)

    ![Экран "Пункт "Добавление SCIM" в Netskope User Authentication"](media/netskope-administrator-console-provisioning-tutorial/directory.png)

3. Прокрутите вниз и щелкните кнопку **Добавить токен**. В диалоговом окне **Add OAuth Client Name** (Добавление имени клиента OAuth) введите **имя клиента** и нажмите кнопку **Сохранить**.

    ![Экран "Пункт "Добавление токена" в Netskope User Authentication"](media/netskope-administrator-console-provisioning-tutorial/add.png)

    ![Экран "Пункт "Имя клиента" в Netskope User Authentication"](media/netskope-administrator-console-provisioning-tutorial/clientname.png)

3.  Скопируйте **URL-адрес сервера SCIM** и **токен**. Эти значения будут указаны в полях "URL-адрес клиента" и "Секретный токен" соответственно на вкладке "Подготовка" приложения Netskope User Authentication на портале Azure.

    ![Экран "Пункт "Создание токена" в Netskope User Authentication"](media/netskope-administrator-console-provisioning-tutorial/token.png)

## <a name="add-netskope-user-authentication-from-the-gallery"></a>Добавление Netskope User Authentication из коллекции

Перед настройкой Netskope User Authentication для автоматической подготовки пользователей в Azure AD необходимо добавить Netskope User Authentication из коллекции приложений Azure AD в список управляемых приложений SaaS.

**Чтобы добавить Netskope User Authentication из коллекции приложений Azure AD, выполните следующие действия.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Netskope User Authentication**, выберите **Netskope User Authentication** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить приложение.

    ![Экран "Netskope User Authentication в списке результатов"](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-netskope-user-authentication"></a>Настройка автоматической подготовки пользователей для Netskope User Authentication 

В этом разделе объясняется, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в Netskope User Authentication на основе их назначений в Azure AD.

> [!TIP]
> Кроме того, вы можете включить для Netskope User Authentication единый вход на основе SAML. Для этого выполните инструкции, приведенные в [учебнике по настройке единого входа для Netskope User Authentication](./netskope-cloud-security-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности хорошо дополняют друг друга.

> [!NOTE]
> Дополнительные сведения о конечной точке SCIM Netskope User Authentication см. [здесь](https://docs.google.com/document/d/1n9P_TL98_kd1sx5PAvZL2HS6MQAqkQqd-OSkWAAU6ck/edit#heading=h.prxq74iwdpon).

### <a name="to-configure-automatic-user-provisioning-for-netskope-user-authentication-in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей Netskope User Authentication в Azure AD, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Netskope User Authentication**.

    ![Экран "Ссылка на Netskope User Authentication в списке "Приложения"](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите в поле **URL-адрес клиента** значение **URL-адрес сервера SCIM**, полученное ранее. Введите полученное ранее значение **токена** в поле **Секретный токен**. Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к Netskope User Authentication. Если установить подключение не удалось, убедитесь, что в учетной записи Netskope User Authentication есть разрешения администратора, и повторите попытку.

    ![URL-адрес клиента + токен](common/provisioning-testconnection-tenanturltoken.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Выберите команду **Сохранить**.

8. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Users to Netskope User Authentication** (Синхронизировать пользователей Azure Active Directory с Netskope User Authentication).

    ![Экран "Раздел сопоставлений пользователей в Netskope User Authentication"](media/netskope-administrator-console-provisioning-tutorial/usermappings.png)

9. В разделе **Сопоставления атрибутов** просмотрите атрибуты пользователя, синхронизированные из Azure AD в Netskope User Authentication. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в Netskope User Authentication для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Экран "Раздел атрибутов пользователей в Netskope User Authentication"](media/netskope-administrator-console-provisioning-tutorial/userattributes.png)

10. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Groups to Netskope User Authentication** (Синхронизировать группы Azure Active Directory с Netskope User Authentication).

    ![Экран "Раздел сопоставлений групп в Netskope User Authentication"](media/netskope-administrator-console-provisioning-tutorial/groupmappings.png)

11. В разделе **Сопоставления атрибутов** просмотрите атрибуты группы, синхронизированные из Azure AD в Netskope User Authentication. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления групп в Netskope User Authentication для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Экран "Раздел атрибутов групп в Netskope User Authentication"](media/netskope-administrator-console-provisioning-tutorial/groupattributes.png)

12. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Чтобы включить службу подготовки Azure AD для Netskope User Authentication, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

14. Определите пользователей и (или) группы для подготовки в Netskope User Authentication, выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

15. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Сведения о синхронизации** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с Netskope User Authentication.

Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)