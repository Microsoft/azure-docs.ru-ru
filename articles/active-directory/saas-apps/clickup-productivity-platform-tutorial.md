---
title: Руководство по Интеграция Azure Active Directory с ClickUp Productivity Platform | Документация Майкрософт
description: В этой статье объясняется, как настроить единый вход между Azure Active Directory и ClickUp Productivity Platform.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/21/2019
ms.author: jeedes
ms.openlocfilehash: 753256adddf8b28518bac45011d582bc7a079629
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97670483"
---
# <a name="tutorial-azure-active-directory-integration-with-clickup-productivity-platform"></a>Руководство по Интеграция Azure Active Directory с ClickUp Productivity Platform

В этом руководстве описано, как интегрировать ClickUp Productivity Platform с Azure Active Directory (Azure AD).
Интеграция Azure AD с ClickUp Productivity Platform обеспечивает следующие преимущества:

* С помощью Azure AD можно контролировать доступ к ClickUp Productivity Platform.
* Вы можете включить автоматический вход пользователей в ClickUp Productivity Platform (единый вход) с использованием учетной записи Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с ClickUp Productivity Platform, вам потребуется:

* Подписка Azure AD. (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка ClickUp Productivity Platform с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* ClickUp Productivity Platform поддерживает единый вход, инициированный **поставщиком услуг**.

## <a name="adding-clickup-productivity-platform-from-the-gallery"></a>Добавление ClickUp Productivity Platform из коллекции

Чтобы настроить интеграцию ClickUp Productivity Platform с Azure AD, необходимо добавить ClickUp Productivity Platform из коллекции в список управляемых приложений SaaS.

**Чтобы добавить ClickUp Productivity Platform из коллекции, выполните следующие действия:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **ClickUp Productivity Platform**, выберите **ClickUp Productivity Platform** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![ClickUp Productivity Platform в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в ClickUp Productivity Platform с использованием тестового пользователя **Britta Simon**.
Чтобы обеспечить работу единого входа, нужно установить связь между пользователем Azure AD и соответствующим пользователем в ClickUp Productivity Platform.

Чтобы настроить и проверить единый вход Azure AD в ClickUp Productivity Platform, вам потребуется выполнить действия в следующих стандартных блоках:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в ClickUp Productivity Platform](#configure-clickup-productivity-platform-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя ClickUp Productivity Platform](#create-clickup-productivity-platform-test-user)** необходимо, чтобы в ClickUp Productivity Platform существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в ClickUp Productivity Platform, выполните следующие действия:

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **ClickUp Productivity Platform** выберите **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    ![Сведения о домене и URL-адресах единого входа для приложения ClickUp Productivity Platform](common/sp-identifier.png)

    а. В текстовом поле **URL-адрес входа** введите URL-адрес: `https://app.clickup.com/login/sso`.

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `https://api.clickup.com/v1/team/<team_id>/microsoft`.

    > [!NOTE]
    > Значение идентификатора приведено для примера и не является реальным. Замените его на фактический идентификатор, как описано далее в этом руководстве.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** нажмите кнопку "Копировать", чтобы копировать **URL-адрес метаданных федерации приложений** и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/copy-metadataurl.png)

### <a name="configure-clickup-productivity-platform-single-sign-on"></a>Настройка единого входа в ClickUp Productivity Platform

1. В другом окне веб-браузера войдите в свой клиент ClickUp Productivity Platform с правами администратора.

2. Щелкните значок **профиля пользователя** и выберите пункт **Settings** (Параметры).

    ![Снимок экрана, на котором показан арендатор ClickUp Productivity с выбранным значком "Settings" (Параметры)](./media/clickup-productivity-platform-tutorial/configure0.png)

    ![Снимок экрана, на котором показан раздел "Settings" (Параметры)](./media/clickup-productivity-platform-tutorial/configure1.png)

3. Выберите **Microsoft** (Майкрософт) в разделе "Single Sign-On (SSO) Provider" (Поставщик единого входа).

    ![Снимок экрана, на котором показана область "Authentication" (Аутентификация) с выбранным элементом "Microsoft" (Майкрософт)](./media/clickup-productivity-platform-tutorial/configure2.png)

4. На странице **Configure Microsoft Single Sign On** (Настройка единого входа Майкрософт) выполните следующие действия:

    ![Снимок экрана, на котором показана страница "Configure Microsoft Single Sign On" (Настройка единого входа Майкрософт), где можно скопировать идентификатор сущности и сохранить URL-адрес метаданных федерации Azure](./media/clickup-productivity-platform-tutorial/configure3.png)

    а. Щелкните **Copy** (Копировать), чтобы скопировать идентификатор сущности, и вставьте его в текстовое поле **Идентификатор (сущности)** в разделе **Базовая конфигурация SAML** на портале Azure.

    b. В текстовое поле **Azure Federation Metadata URL** (URL-адрес метаданных федерации приложения Azure) вставьте URL-адрес метаданных федерации приложения, скопированный на портале Azure, и щелкните **Save** (Сохранить).

5. Чтобы завершить настройку, щелкните **Authenticate With Microsoft to complete setup** (Выполнить аутентификацию с помощью средств Майкрософт, чтобы завершить настройку) и выполните аутентификацию с помощью учетной записи Майкрософт.

    ![Снимок экрана, на котором показана кнопка "Authenticate With Microsoft to complete setup" (Выполнить аутентификацию с помощью средств Майкрософт, чтобы завершить настройку)](./media/clickup-productivity-platform-tutorial/configure4.png)

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD

Цель этого раздела — создать на портале Azure тестового пользователя с именем Britta Simon.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.

    ![Ссылки "Пользователи и группы" и "Все пользователи"](common/users.png)

2. В верхней части экрана выберите **Новый пользователь**.

    ![Кнопка "Новый пользователь"](common/new-user.png)

3. В разделе свойств пользователя сделайте следующее:

    ![Диалоговое окно "Пользователь"](common/user-properties.png)

    а. В поле **Имя** введите **BrittaSimon**.

    b. В поле **Имя пользователя** введите **brittasimon\@домен_вашей_компании.доменная_зона**.  
    Например BrittaSimon@contoso.com.

    c. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле "Пароль".

    d. Нажмите кнопку **Создать**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure путем предоставления доступа к ClickUp Productivity Platform.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения** и **ClickUp Productivity Platform**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **ClickUp Productivity Platform**.

    ![Ссылка на ClickUp Productivity Platform в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-clickup-productivity-platform-test-user"></a>Создание тестового пользователя ClickUp Productivity Platform

1. В другом окне веб-браузера войдите в свой клиент ClickUp Productivity Platform с правами администратора.

2. Щелкните значок **профиля пользователя** и выберите пункт **People** (Пользователи).

    ![Снимок экрана, на котором показан арендатор ClickUp Productivity](./media/clickup-productivity-platform-tutorial/configure0.png)

    ![Снимок экрана, на котором показана выбранная ссылка "People" (Люди)](./media/clickup-productivity-platform-tutorial/user1.png)

3. Введите в текстовом поле адрес электронной почты пользователя и щелкните **Invite** (Пригласить).

    ![Снимок экрана, на котором показан раздел "Team Users Settings" (Параметры участников команды), позволяющий пригласить людей по электронной почте](./media/clickup-productivity-platform-tutorial/user2.png)

    > [!NOTE]
    > Пользователь получит уведомление, и ему потребуется принять приглашение, чтобы активировать учетную запись.

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку ClickUp Productivity Platform на Панели доступа, вы автоматически войдете в приложение ClickUp Productivity Platform, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)