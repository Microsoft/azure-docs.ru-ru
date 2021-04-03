---
title: Руководство по интеграции Azure Active Directory с LiquidFiles | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в LiquidFiles.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/14/2019
ms.author: jeedes
ms.openlocfilehash: b9ca860043d5c4a492908aa45bf0a89ab368d50f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92458478"
---
# <a name="tutorial-azure-active-directory-integration-with-liquidfiles"></a>Руководство по интеграции Azure Active Directory с LiquidFiles

В этом руководстве описано, как интегрировать приложение LiquidFiles с Azure Active Directory (Azure AD).
Интеграция Azure AD с приложением LiquidFiles обеспечивает следующие преимущества:

* С помощью Azure AD вы можете контролировать доступ к LiquidFiles.
* Вы можете включить автоматический вход для пользователей в LiquidFiles (единый вход) с помощью учетной записи Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с LiquidFiles, вам потребуется:

* Подписка Azure AD. (если у вас нет среды Azure AD, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/));
* подписка LiquidFiles с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* LiquidFiles поддерживает единый вход, инициированный **пакетом обновления**.

## <a name="adding-liquidfiles-from-the-gallery"></a>Добавление LiquidFiles из коллекции

Чтобы настроить интеграцию LiquidFiles с Azure AD, необходимо добавить LiquidFiles из коллекции в список управляемых приложений SaaS.

**Чтобы добавить LiquidFiles из коллекции, сделайте следующее.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **LiquidFiles**, выберите **LiquidFiles** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![LiquidFiles в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в LiquidFiles с использованием тестового пользователя **Britta Simon**.
Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в LiquidFiles.

Чтобы настроить и проверить единый вход Azure AD в LiquidFiles, вам потребуется выполнить действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в LiquidFiles](#configure-liquidfiles-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя LiquidFiles](#create-liquidfiles-test-user)** требуется для того, чтобы в LiquidFiles существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в LiquidFiles, выполните следующие действия.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **LiquidFiles** выберите **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    ![Сведения о домене и URL-адресах единого входа для приложения LiquidFiles](common/sp-identifier-reply.png)

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<YOUR_SERVER_URL>/saml/init`.

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `https://<YOUR_SERVER_URL>`.

    c. В текстовом поле **URL-адрес ответа** введите URL-адрес в следующем формате: `https://<YOUR_SERVER_URL>/saml/consume`.

    > [!NOTE]
    > Эти значения приведены для примера. Необходимо обновить эти значения действующим URL-адресом для входа и идентификатором. Чтобы получить эти значения, обратитесь к [группе поддержки клиентов LiquidFiles](https://www.liquidfiles.com/support.html). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. В разделе **Сертификат подписи SAML** щелкните кнопку **Правка**, чтобы открыть диалоговое окно **Сертификат подписи SAML**.

    ![Изменение сертификата подписи SAML](common/edit-certificate.png)

6. В разделе **Сертификат подписи SAML** скопируйте значение **Отпечаток** и сохраните его на компьютере.

    ![Копирование значения "Отпечаток"](common/copy-thumbprint.png)

7. Скопируйте требуемые URL-адреса из раздела **Настройка LiquidFiles**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

### <a name="configure-liquidfiles-single-sign-on"></a>Настройка единого входа LiquidFiles

1. Войдите на корпоративный сайт LiquidFiles в качестве администратора.

1. В меню **Admin > Configuration** (Администрирование > Конфигурация) щелкните **Single Sign-On** (Единый вход).

1. На странице **Single Sign-On Configuration** (Конфигурация единого входа) выполните следующие действия.

    ![Настройка единого входа](./media/liquidfiles-tutorial/tutorial_single_01.png)

    а. Для параметра **Single Sign On Method** (Метод единого входа) выберите значение **SAML 2**.

    b. В текстовое поле **IdP Login URL** (URL-адрес для входа с помощью IdP) вставьте значение **URL-адреса входа**, скопированное на портале Azure.

    c. В текстовое поле **IdP Logout URL** (URL-адрес для выхода с помощью IdP) вставьте значение **URL-адреса выхода**, скопированное на портале Azure.

    d. В текстовое поле **IDP Cert Fingerprint** (Отпечаток сертификата IdP) вставьте значение **Отпечаток**, которое вы скопировали на портале Azure.

    д) В текстовое поле "Формат идентификатора имени" введите значение `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress`.

    е) В текстовое поле Authn Context (Контекст Authn) введите значение `urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport`.

    ж. Выберите команду **Сохранить**.

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD

Цель этого раздела — создать на портале Azure тестового пользователя с именем Britta Simon.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.

    ![Ссылки "Пользователи и группы" и "Все пользователи"](common/users.png)

2. В верхней части экрана выберите **Новый пользователь**.

    ![Кнопка "Новый пользователь"](common/new-user.png)

3. В разделе свойств пользователя сделайте следующее:

    ![Диалоговое окно "Пользователь"](common/user-properties.png)

    а. В поле **Имя** введите **BrittaSimon**.
  
    b. В поле **Имя пользователя** введите `brittasimon@yourcompanydomain.extension`. Например BrittaSimon@contoso.com.

    c. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле "Пароль".

    d. Нажмите кнопку **Создать**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к LiquidFiles.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем **LiquidFiles**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **LiquidFiles**.

    ![Ссылка на LiquidFiles в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-liquidfiles-test-user"></a>Создание тестового пользователя LiquidFiles

Цель этого раздела — создать пользователя с именем Britta Simon в LiquidFiles. Обратитесь к администратору сервера LiquidFiles, чтобы он добавил для вас пользователя, после чего выполните вход в приложение LiquidFiles.

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку LiquidFiles на Панели доступа, вы автоматически войдете в приложение LiquidFiles, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)