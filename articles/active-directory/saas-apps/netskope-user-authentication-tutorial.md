---
title: Руководство по интеграции единого входа Azure Active Directory с Netskope User Authentication | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Netskope User Authentication.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/01/2019
ms.author: jeedes
ms.openlocfilehash: 1ae9e39f5a23725b17451d7144c8304194d2a1d0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92514373"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-netskope-user-authentication"></a>Руководство по интеграции единого входа Azure Active Directory с Netskope User Authentication

В этом учебнике описано, как интегрировать Netskope User Authentication с Azure Active Directory (Azure AD). Интеграция Netskope User Authentication с Azure AD обеспечивает следующие возможности:

* Вы можете управлять доступом к Netskope User Authentication с помощью Azure AD.
* Вы можете включить автоматический вход пользователей (единый вход) в Netskope User Authentication с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

Чтобы узнать больше об интеграции приложений SaaS с Azure AD, прочитайте статью [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Netskope User Authentication с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Netskope User Authentication поддерживает единый вход, инициируемый **поставщиком услуг и поставщиком удостоверений**.

## <a name="adding-netskope-user-authentication-from-the-gallery"></a>Добавление Netskope User Authentication из коллекции

Чтобы настроить интеграцию Netskope User Authentication с Azure AD, необходимо добавить это приложение из коллекции в список управляемых приложений SaaS.

1. Войдите на [портал Azure](https://portal.azure.com) с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Netskope User Authentication**.
1. Выберите **Netskope User Authentication** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.


## <a name="configure-and-test-azure-ad-single-sign-on-for-netskope-user-authentication"></a>Настройка и проверка единого входа Azure AD для Netskope User Authentication

Настройте и проверьте единый вход Azure AD в Netskope User Authentication с помощью тестового пользователя **B. Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Netskope User Authentication.

Чтобы настроить и проверить единый вход Azure AD в Netskope User Authentication, выполните действия в следующих стандартных блоках:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    * **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    * **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Netskope User Authentication](#configure-netskope-user-authentication-sso)** . Настройка единого входа на стороне приложения.
    * **[Создание тестового пользователя Netskope User Authentication](#create-netskope-user-authentication-test-user)** требуется для того, чтобы в Netskope User Authentication существовал пользователь B. Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Netskope User Authentication** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `https://<tenantname>.goskope.com/<customer entered string>`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<tenantname>.goskope.com/nsauth/saml2/http-post/<customer entered string>`.

    > [!NOTE]
    > Эти значения приведены для примера. Измените их на фактические значения идентификатора и URL-адреса ответа. Они объяснены далее в этом учебнике.

1. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://<tenantname>.goskope.com`.

    > [!NOTE]
    > Значение URL-адреса для входа приведено для примера. Замените его фактическим URL-адресом для входа. Чтобы получить значение URL-адреса входа, обратитесь в [службу поддержки клиентов Netskope User Authentication](mailto:support@netskope.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

1. Требуемые URL-адреса можно скопировать из раздела **Настройка Netskope User Authentication**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD

В этом разделе описано, как на портале Azure создать тестового пользователя с именем B.Simon.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.
1. В верхней части экрана выберите **Новый пользователь**.
1. В разделе **Свойства пользователя** выполните следующие действия.
   1. В поле **Имя** введите `B.Simon`.  
   1. В поле **Имя пользователя** введите username@companydomain.extension. Например, `B.Simon@contoso.com`.
   1. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле **Пароль**.
   1. Нажмите кнопку **Создать**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как включить единый вход Azure для пользователя B. Simon, предоставив этому пользователю доступ к Netskope User Authentication.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. Из списка приложений выберите **Netskope User Authentication**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.

   ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Ссылка "Добавить пользователя"](common/add-assign-user.png)

1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор роли** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-netskope-user-authentication-sso"></a>Настройка единого входа для Netskope User Authentication

1. Откройте новую вкладку в браузере и войдите на корпоративный сайт Netskope User Authentication с правами администратора.

1. Перейдите на вкладку **Active Platform** (Активная платформа).

    ![Снимок экрана: меню Settings (Параметры) с выбранным пунктом Active Platform (Активная платформа).](./media/netskope-user-authentication-tutorial/user1.png)

1. Прокрутите вниз до раздела **FORWARD PROXY** (ПРОКСИ-СЕРВЕР ПЕРЕАДРЕСАЦИИ) и выберите **SAML**.

    ![Снимок экрана, где в качестве активной платформы выбран SAML.](./media/netskope-user-authentication-tutorial/config-saml.png)

1. На странице **Параметры SAML** выполните следующие действия.

    ![Снимок экрана: страница параметров SAML для ввода описанных значений.](./media/netskope-user-authentication-tutorial/configure-copyurls.png)

    а. Скопируйте значение **SAML Entity ID** (Идентификатор сущности SAML) и вставьте его в текстовое поле **Идентификатор** в разделе **Базовая конфигурация SAML** на портале Azure.

    b. Скопируйте значение **SAML ACS URL** (URL-адрес ACS SAML) и вставьте его в текстовое поле **URL-адрес ответа** в разделе **Базовая конфигурация SAML** на портале Azure.

1. Нажмите кнопку **ADD ACCOUNT** (ДОБАВИТЬ УЧЕТНУЮ ЗАПИСЬ).

    ![Снимок экрана для панели SAML, где выбрано действие добавления учетной записи.](./media/netskope-user-authentication-tutorial/config-addaccount.png)

1. На странице **добавления учетной записи SAML** выполните следующие действия:

    ![Снимок экрана: страница добавления учетной записи SAML для ввода описанных значений.](./media/netskope-user-authentication-tutorial/config-settings1.png)

    а. В текстовом поле **NAME** (ИМЯ) введите имя, например Azure AD.

    b. В текстовое поле **IDP URL** (URL-АДРЕС ПОСТАВЩИКА УДОСТОВЕРЕНИЙ) вставьте значение **URL-адреса входа**, скопированное на портале Azure.

    c. В текстовое поле **IDP ENTITY ID** (ИДЕНТИФИКАТОР СУЩНОСТИ ПОСТАВЩИКА УДОСТОВЕРЕНИЙ) вставьте значение **Идентификатора Azure AD**, скопированное на портале Azure.

    d. Откройте скачанный файл метаданных в Блокноте, скопируйте его содержимое в буфер обмена, а затем вставьте его в текстовое поле **IDP CERTIFICATE** (СЕРТИФИКАТ ПОСТАВЩИКА УДОСТОВЕРЕНИЙ).

    д) Щелкните **СОХРАНИТЬ**.

### <a name="create-netskope-user-authentication-test-user"></a>Создание тестового пользователя в Netskope User Authentication

1. Откройте новую вкладку в браузере и войдите на корпоративный сайт Netskope User Authentication с правами администратора.

1. Перейдите на вкладку **Settings** (Параметры) в области навигации слева.

    ![Снимок экрана, где выбрана вкладка параметров.](./media/netskope-user-authentication-tutorial/config-settings.png)

1. Перейдите на вкладку **Active Platform** (Активная платформа).

    ![Снимок экрана: меню Settings (Параметры) с выбранным пунктом Active Platform (Активная платформа).](./media/netskope-user-authentication-tutorial/user1.png)

1. Откройте вкладку **Пользователи**.

    ![Снимок экрана: в разделе Active Platform (Активная платформа) выбран пункт Users (Пользователи).](./media/netskope-user-authentication-tutorial/add-user.png)

1. Нажмите кнопку **ADD USERS** (ДОБАВИТЬ ПОЛЬЗОВАТЕЛЕЙ).

    ![Снимок экрана: диалоговое окно Users (Пользователи), где можно нажать кнопку ADD USERS (ДОБАВИТЬ ПОЛЬЗОВАТЕЛЕЙ).](./media/netskope-user-authentication-tutorial/user-add.png)

1. Введите адрес электронной почты пользователя, которого требуется добавить, и нажмите кнопку **ADD** (ДОБАВИТЬ).

    ![Снимок экрана: раздел Add Users (Добавление пользователей), где можно ввести список пользователей.](./media/netskope-user-authentication-tutorial/add-user-popup.png)

## <a name="test-sso"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку Netskope User Authentication на Панели доступа, вы автоматически войдете в приложение Netskope User Authentication, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)

- [Попробовать использовать Netskope User Authentication с Azure AD](https://aad.portal.azure.com/)