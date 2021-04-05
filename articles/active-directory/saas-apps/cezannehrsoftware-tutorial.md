---
title: Учебник. Интеграция Azure Active Directory с Cezanne HR Software | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Cezanne HR Software.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/12/2019
ms.author: jeedes
ms.openlocfilehash: 0aa0dab7b512c85fbbdf374c962e6ee8e1c7d616
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92456298"
---
# <a name="tutorial-azure-active-directory-integration-with-cezanne-hr-software"></a>Руководство. Интеграция Azure Active Directory с Cezanne HR Software

В этом руководстве описано, как интегрировать Cezanne HR Software с Azure Active Directory (Azure AD).
Интеграция Cezanne HR Software с Azure AD обеспечивает следующие преимущества:

* С помощью Azure AD можно контролировать доступ к Cezanne HR Software.
* Учетные записи Azure AD позволяют включить автоматический вход пользователей в Cezanne HR Software (единый вход).
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD с Cezanne HR Software, вам потребуется следующее:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка на Cezanne HR Software с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* В Cezanne HR Software поддерживается единый вход, инициированный **поставщиком услуг**

## <a name="adding-cezanne-hr-software-from-the-gallery"></a>Добавление Cezanne HR Software из коллекции.

Чтобы настроить интеграцию Cezanne HR Software с Azure AD, вам потребуется добавить Cezanne HR Software из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Cezanne HR Software из коллекции, сделайте следующее:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Cezanne HR Software**, выберите **Cezanne HR Software** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Cezanne HR Software в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В рамках этого раздела вы настроите и проверите единый вход Azure AD в Cezanne HR Software с использованием тестового пользователя **Britta Simon**.
Чтобы обеспечить единый вход, свяжите пользователя Azure AD с соответствующим пользователем в Cezanne HR Software.

Чтобы настроить и проверить единый вход в Azure AD в Cezanne HR Software, вам потребуется выполнить действия в следующих блоках:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Cezanne HR Software](#configure-cezanne-hr-software-single-sign-on)** требуется, чтобы определить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя Cezanne HR Software](#create-cezanne-hr-software-test-user)** требуется, чтобы в Cezanne HR Software существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Cezanne HR Software, сделайте следующее:

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Cezanne HR Software** щелкните **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    ![Сведения о домене и URL-адресах единого входа для приложения Cezanne HR Software](common/sp-identifier-reply.png)

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://w3.cezanneondemand.com/CezanneOnDemand/-/<tenantidentifier>`.

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес: `https://w3.cezanneondemand.com/CezanneOnDemand/`.

    c. В текстовом поле **URL-адрес ответа** введите URL-адрес в следующем формате: `https://w3.cezanneondemand.com:443/cezanneondemand/-/<tenantidentifier>/Saml/samlp`.
    
    > [!NOTE]
    > Эти значения приведены для примера. Измените их на фактические значения URL-адреса входа и URL-адреса ответа. Чтобы получить эти значения, обратитесь в [службу поддержки клиентов Cezanne HR Software](https://cezannehr.com/services/support/).

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (Base64)** из предложенных вариантов, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

6. Требуемые URL-адреса можно скопировать из раздела **настройки Cezanne HR Software**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD.

    c. URL-адрес выхода.

### <a name="configure-cezanne-hr-software-single-sign-on"></a>Настройка единого входа Cezanne HR Software

1. В другом окне веб-браузера войдите на веб-сайт Cezanne HR Software своей компании в качестве администратора.

2. В левой панели навигации щелкните **System Setup**(Настройка системы). Выберите пункт **Параметры безопасности**. Далее перейдите к пункту **Конфигурация единого входа**.

    ![Снимок экрана, на котором показан арендатор Cezanne HR Software с выбранными параметрами "Security Settings" (Параметры безопасности) и "Single Sign-On Configuration" (Конфигурация единого входа)](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_000.png)

3. На панели **Allow users to log in using the following Single Sign-On (SSO) Service** (Разрешить пользователям вход с использованием следующей службы единого входа) установите флажок **SAML 2.0** и выберите параметр **Расширенная конфигурация**.

    ![Снимок экрана, на котором показана область "Allow users" (Разрешить пользователям) с установленным флажком "SAML 2.0" и выбранным параметром "Advanced Configuration" (Расширенная конфигурация)](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_001.png)

4. Нажмите кнопку **Добавить** .

    ![Снимок экрана, на котором показана кнопка "Add New" (Добавить)](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_002.png)

5. В разделе **SAML 2.0 Identity providers** (Поставщики удостоверений SAML) выполните следующие действия.

    ![Снимок экрана, на котором показана область, где можно ввести значения, описанные на этом шаге](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_003.png)

    а. Введите имя поставщика удостоверений в поле **Отображаемое имя**.

    b. В текстовое поле **Идентификатор сущности** вставьте значение **идентификатора Azure AD**, скопированное на портале Azure.

    c. В поле **SAML Binding** (Привязка SAML) укажите значение POST.

    d. В текстовое поле **Security Token Service Endpoint** (Конечная точка службы токенов безопасности) вставьте значение **URL-адреса для входа в учетную запись**, скопированное на портале Azure.

    д) В текстовом поле "User ID Attribute Name" (Имя атрибута идентификатора пользователя) введите `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`.

    е) Чтобы передать сертификат, скачанный с портала Azure, щелкните значок **Upload** (Передать).

    ж. Нажмите кнопку **ОК** .

6. Нажмите кнопку **Сохранить** .

    ![Снимок экрана, на котором показана кнопка "Save" (Сохранить) для раздела "Single Sign-On Configuration" (Конфигурация единого входа)](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_004.png)

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

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure путем предоставления доступа к Cezanne HR Software.

1. На портале Azure последовательно выберите **Корпоративные приложения**, **Все приложения** и **Cezanne HR Software**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Cezanne HR Software**.

    ![Ссылка на Cezanne HR Software в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-cezanne-hr-software-test-user"></a>Создание тестового пользователя Cezanne HR Software

Чтобы разрешить пользователям Azure AD вход в Cezanne HR Software, их необходимо подготовить для Cezanne HR Software. В случае с Cezanne HR Software подготовка выполняется вручную.

**Чтобы подготовить учетную запись пользователя, сделайте следующее:**

1. Войдите на корпоративный сайт Cezanne HR Software в качестве администратора.

2. В левой панели навигации щелкните **System Setup**(Настройка системы). Перейдите в раздел **Управление пользователями**. Выберите **Добавить нового пользователя**.

    ![Снимок экрана, на котором показан арендатор Cezanne HR Software с выбранными параметрами "Manage Users" (Управление пользователями) и "Add New User" (Добавить пользователя)](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_005.png "Новый пользователь")

3. В разделе **PERSON DETAILS** (Личные данные) выполните следующие действия:

    ![Снимок экрана, на котором показан раздел "PERSON DETAILS" (Личные данные), где можно ввести значения, описанные на этом шаге](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_006.png "Новый пользователь")

    а. Установите для параметра **Внутренний пользователь** значение "Выкл.".

    b. В текстовом поле **First name** (Имя) введите имя пользователя, например **Britta**.  

    c. В текстовом поле **Last name** (Фамилия) введите фамилию, например **Simon**.

    d. В текстовом поле **Email** (Электронная почта) введите адрес электронной почты пользователя, например Brittasimon@contoso.com.

4. В разделе **Account Information** (Сведения об учетной записи) выполните следующие действия.

    ![Снимок экрана, на котором показан раздел "ACCOUNT INFORMATION" (Сведения об учетной записи), где можно ввести сведения, описанные на этом шаге](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_007.png "Новый пользователь")

    а. В текстовом поле **Username** (Имя пользователя) введите электронный адрес пользователя, например Brittasimon@contoso.com.

    b. В текстовом поле **Password** (Пароль) введите пароль пользователя.

    c. В раскрывающемся списке **Роль безопасности** выберите **Специалист отдела кадров**.

    d. Нажмите кнопку **ОК**.

5. Перейдите на вкладку **Единый вход** и нажмите кнопку **Добавить** в области **Идентификаторы SAML 2.0)** .

    ![Снимок экрана, на котором показана складка "Single Sign On" (Единый вход), где можно нажать кнопку "Add New" (Добавить)](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_008.png "Пользователь")

6. Выберите значение в списке **Поставщик удостоверений** и в текстовом поле **Идентификатор пользователя** введите адрес электронной почты учетной записи Britta Simon.

    ![Снимок экрана, на котором показана область "SAML 2.0 Identifiers" (Идентификаторы SAML 2.0), где можно выбрать поставщик удостоверений и идентификатор пользователя](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_009.png "Пользователь")

7. Нажмите кнопку **Сохранить** .

    ![Снимок экрана, на котором показана кнопка "Save" (Сохранить) для параметров пользователя](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_010.png "Пользователь")

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку Cezanne HR Software на Панели доступа вы автоматически войдете в приложение Cezanne HR Software, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)