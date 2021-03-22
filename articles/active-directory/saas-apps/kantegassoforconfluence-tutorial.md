---
title: Руководство по Интеграция Azure Active Directory с Kantega SSO for Confluence | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Kantega SSO for Confluence.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/25/2019
ms.author: jeedes
ms.openlocfilehash: be86e04359c29696d208994d85d36b7740b60cc3
ms.sourcegitcommit: b4647f06c0953435af3cb24baaf6d15a5a761a9c
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/02/2021
ms.locfileid: "101646220"
---
# <a name="tutorial-azure-active-directory-integration-with-kantega-sso-for-confluence"></a>Руководство по Интеграция Azure Active Directory с Kantega SSO for Confluence

В этом руководстве описано, как интегрировать Kantega SSO for Confluence с Azure Active Directory (Azure AD).
Интеграция Azure AD с приложением Kantega SSO for Confluence обеспечивает следующие преимущества:

* С помощью Azure AD вы можете контролировать доступ к Kantega SSO for Confluence.
* Вы можете включить автоматический вход пользователей в Kantega SSO for Confluence (единый вход) с использованием учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с Kantega SSO for Confluence, вам потребуется:

* Подписка Azure AD. (если у вас нет среды Azure AD, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/));
* подписка Kantega SSO for Confluence с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Kantega SSO for Confluence поддерживает вход, инициированный **пакетом обновления и IDP**.

## <a name="adding-kantega-sso-for-confluence-from-the-gallery"></a>Добавление Kantega SSO for Confluence из коллекции

Чтобы настроить интеграцию Kantega SSO for Confluence с Azure AD, необходимо добавить Kantega SSO for Confluence из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Kantega SSO for Confluence из коллекции, сделайте следующее.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Kantega SSO for Confluence**, выберите **Kantega SSO for Confluence** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Kantega SSO for Confluence в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описаны настройка и проверка единого входа Azure AD в Kantega SSO for Confluence с использованием тестового пользователя **Britta Simon**.
Чтобы обеспечить работу единого входа, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Kantega SSO for Confluence.

Чтобы настроить и проверить единый вход Azure AD в Kantega SSO for Confluence, вам потребуется выполнить действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Kantega SSO for Confluence](#configure-kantega-sso-for-confluence-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя Kantega SSO for Confluence](#create-kantega-sso-for-confluence-test-user)** требуется для того, чтобы в Kantega SSO for Confluence существовал пользователь Britta Simon, связанный с представлением этого же пользователя в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Kantega SSO for Confluence, выполните следующие действия.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Kantega SSO for Confluence** выберите **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** выполните следующие действия.

    ![Снимок экрана: раздел "Базовая конфигурация SAML" с выбранными полями "Идентификатор", "URL-адрес ответа" и кнопкой "Сохранить".](common/idp-intiated.png)

    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `https://<server-base-url>/plugins/servlet/no.kantega.saml/sp/<uniqueid>/login`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<server-base-url>/plugins/servlet/no.kantega.saml/sp/<uniqueid>/login`.

5. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    ![Сведения о домене и URL-адресах единого входа для приложения Kantega SSO for Confluence](common/metadata-upload-additional-signon.png)

    В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://<server-base-url>/plugins/servlet/no.kantega.saml/sp/<uniqueid>/login`.

    > [!NOTE]
    > Эти значения приведены для примера. Замените их фактическими значениями идентификатора, URL-адреса ответа и URL-адреса входа. Эти значения предоставляются во время настройки подключаемого модуля Confluence, которая описывается далее в этом руководстве.

6. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

7. Скопируйте требуемые URL-адреса в разделе **Настройка Kantega SSO for Confluence**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

### <a name="configure-kantega-sso-for-confluence-single-sign-on"></a>Настройка единого входа в Kantega SSO for Confluence

1. В другом окне веб-браузера войдите на **портал администрирования Confluence** с правами администратора.

1. Наведите указатель мыши на шестеренку и щелкните **Add-ons** (Надстройки).

    ![Снимок экрана: выбранные значок меню шестеренки и пункт "Надстройки".](./media/kantegassoforconfluence-tutorial/addon1.png)

1. В разделе **ATLASSIAN MARKETPLACE** щелкните **Find new add-ons** (Найти новые надстройки).

    ![Снимок экрана: вкладка ATTLASSIAN MARKETPLACE с выбранным пунктом Find new add-ons (Найти новые надстройки).](./media/kantegassoforconfluence-tutorial/addon.png)

1. Найдите подключаемый модуль **Kantega SSO for Confluence (SAML & Kerberos)** и нажмите кнопку **Install** (Установить), чтобы установить новый подключаемый модуль SAML.

    ![Снимок экрана: страница Find new add-ons (Найти новые надстройки) с параметром Kantega SSO for Confluence SAML Kerberos в поле поиска и выбранной кнопкой Install (Установить).](./media/kantegassoforconfluence-tutorial/addon2.png)

1. Начнется установка подключаемого модуля.

    ![Снимок экрана: экран установки подключаемого модуля.](./media/kantegassoforconfluence-tutorial/addon3.png)

1. Установка завершится. Щелкните **Закрыть**.

    ![Снимок экрана: экран установки и готовности к использованию с выбранной кнопкой Close (Закрыть).](./media/kantegassoforconfluence-tutorial/addon33.png)

1. Нажмите кнопку **Управление**.

    ![Снимок экрана: подключаемый модуль Kantega Single Sign-on for Kerberos and SAML с выбранной кнопкой управления.](./media/kantegassoforconfluence-tutorial/addon34.png)

1. Щелкните **Configure** (Настройка), чтобы настроить новый подключаемый модуль.

    ![Снимок экрана: страница Kantega Single Sign-on with Kerberos and SAML с выбранной кнопкой Configure (Настройка).](./media/kantegassoforconfluence-tutorial/addon35.png)

1. Этот новый подключаемый модуль можно также найти на вкладке **USERS & SECURITY** (Пользователи и безопасность).

    ![Снимок экрана: вкладка USERS & SECURITY (Пользователи и безопасность) с выбранным действием Kantega Single Sign-on.](./media/kantegassoforconfluence-tutorial/addon36.png)

1. В разделе **SAML** сделайте следующее. Выберите **Azure Active Directory (Azure AD)** из раскрывающегося списка **Добавление поставщика удостоверений**.

    ![Снимок экрана: раздел SAML с выбранным пунктом Azure Active Directory (Azure AD) из раскрывающегося списка добавления поставщика удостоверений.](./media/kantegassoforconfluence-tutorial/addon4.png)

1. Выберите уровень подписки **Базовый**.

    ![Снимок экрана: страница Preparing Azure AD (Подготовка Azure AD) с выбранным параметром Basic (Базовый).](./media/kantegassoforconfluence-tutorial/addon5.png)

1. В разделе **Свойства приложения** выполните следующие действия.

    ![Снимок экрана: раздел App properties (Свойства приложения) с выбранными полем URL-адреса идентификатора приложения, кнопкой копирования и кнопкой Next (Далее).](./media/kantegassoforconfluence-tutorial/addon6.png)

    а. Скопируйте значение **App ID URI** (URI кода приложения) и используйте его как **идентификатор, URL-адрес ответа и URL-адрес входа** в разделе **Базовая конфигурация SAML** на портале Azure.

    b. Щелкните **Далее**.

1. В разделе **Metadata import** (Импорт метаданных) выполните следующие действия. 

    ![Снимок экрана: раздел импорта метаданных с выбранным параметром Metadata file on my computer (Файл метаданных на моем компьютере).](./media/kantegassoforconfluence-tutorial/addon7.png)

    а. Щелкните **Metadata file on my computer** (Файл метаданных на моем компьютере) и передайте файл метаданных, который вы скачали с портала Azure.

    b. Щелкните **Далее**.

1. В разделе **Name and SSO location** (Имя и расположение единого входа) выполните следующее.

    ![Снимок экрана: раздел имени и расположения единого входа с выбранными текстовым полем Identity provider name (Имя поставщика удостоверений) и кнопкой Next (Далее).](./media/kantegassoforconfluence-tutorial/addon8.png)

    а. В текстовом поле **Provider Name** (Имя поставщика) введите имя поставщика (например, Azure AD).

    b. Щелкните **Далее**.

1. Проверьте сертификат для подписи и нажмите кнопку **Next** (Далее).

    ![Снимок экрана: раздел проверки подписи с выбранной кнопкой Next (Далее).](./media/kantegassoforconfluence-tutorial/addon9.png)

1. В разделе **Confluence user accounts** (Учетные записи пользователей Confluence) выполните следующие действия.

    ![Снимок экрана: раздел учетных записей пользователей Confluence с выделенными параметром Create users in Confluence's Internal Directory if needed (При необходимости создать пользователей во внутреннем каталоге Confluence) и кнопкой Next (Далее).](./media/kantegassoforconfluence-tutorial/addon10.png)

    а. Щелкните переключатель **Create users in Confluence's internal Directory if needed** (При необходимости создать пользователей во внутреннем каталоге Confluence) и введите соответствующее имя группы пользователей (это может быть несколько групп, разделенных запятой).

    b. Щелкните **Далее**.

1. Нажмите кнопку **Готово**.

    ![Снимок экрана: страница сводки с выбранной кнопкой Finish (Готово).](./media/kantegassoforconfluence-tutorial/addon11.png)

1. В разделе **Known domains for Azure AD** (Известные домены для Azure AD) выполните следующие действия. 

    ![Снимок экрана: страница известных доменов для Azure AD с выбранными текстовым полем известных доменов и кнопкой Save (Сохранить).](./media/kantegassoforconfluence-tutorial/addon12.png)

    а. Щелкните **Known domains** (Известные домены) на левой панели страницы.

    b. Введите имя домена в текстовое поле **Known domains** (Известные домены).

    c. Выберите команду **Сохранить**.

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD

Цель этого раздела — создать на портале Azure тестового пользователя с именем Britta Simon.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.

    ![Ссылки "Пользователи и группы" и "Все пользователи"](common/users.png)

2. В верхней части экрана выберите **Новый пользователь**.

    ![Кнопка "Новый пользователь"](common/new-user.png)

3. В разделе свойств пользователя сделайте следующее:

    ![Диалоговое окно "Пользователь"](common/user-properties.png)

    а. В поле **Имя** введите **BrittaSimon**.
  
    b. В поле **Имя пользователя** введите `brittasimon@yourcompanydomain.extension`.  
    Например BrittaSimon@contoso.com.

    c. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле "Пароль".

    d. Нажмите кнопку **Создать**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к Kantega SSO for Confluence.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем **Kantega SSO for Confluence**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Kantega SSO for Confluence**.

    ![Ссылка на Kantega SSO for Confluence в списке "Приложения"](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-kantega-sso-for-confluence-test-user"></a>Создание тестового пользователя Kantega SSO for Confluence

Чтобы пользователи Azure AD могли выполнить вход в Confluence, для них нужно выполнить подготовку к работе в Confluence. В случае с Kantega SSO for Confluence подготовка выполняется вручную.

**Чтобы подготовить учетную запись пользователя, сделайте следующее:**

1. Войдите на корпоративный сайт Kantega SSO for Confluence с правами администратора.

1. Наведите указатель мыши на шестеренку и щелкните **User management** (Управление пользователями).

    ![Снимок экрана: выбранные значок шестеренки и пункт User management (Управление пользователями).](./media/kantegassoforconfluence-tutorial/user1.png)

1. В разделе "Users" (Пользователи) выберите вкладку **Add Users** (Добавление пользователей). На диалоговой странице **Add a User** (Добавление пользователя) выполните следующее.

    ![Добавление сотрудника](./media/kantegassoforconfluence-tutorial/user2.png)

    а. В текстовом поле **Username** (Имя пользователя) введите электронный адрес пользователя, например Brittasimon@contoso.com.

    b. В текстовом поле **Full Name** (Полное имя) введите полное имя пользователя, например Britta Simon.

    c. В текстовом поле **Email** (Электронная почта) введите адрес электронной почты пользователя, например Brittasimon@contoso.com.

    d. В текстовом поле **Password** (Пароль) введите пароль пользователя.

    д) Щелкните **Confirm Password** (Подтвердить пароль) и повторно введите пароль.

    е) Нажмите кнопку **Добавить**.

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув элемент Kantega SSO for Confluence на Панели доступа, вы автоматически войдете в приложение Kantega SSO for Confluence, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)