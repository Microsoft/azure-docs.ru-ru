---
title: Руководство по интеграции Azure Active Directory с Kantega SSO for Bamboo | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Kantega SSO for Bamboo.
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
ms.openlocfilehash: aa5f908cdf25925db63054adaf1e6dab15f5260b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92459311"
---
# <a name="tutorial-azure-active-directory-integration-with-kantega-sso-for-bamboo"></a>Руководство по интеграции Azure Active Directory с Kantega SSO for Bamboo

В этом руководстве описано, как интегрировать Kantega SSO for Bamboo с Azure Active Directory (Azure AD).
Интеграция Azure AD с приложением Kantega SSO for Bamboo обеспечивает следующие преимущества:

* С помощью Azure AD вы можете контролировать доступ к Kantega SSO for Bamboo.
* Вы можете включить автоматический вход пользователей в Kantega SSO for Bamboo (единый вход) с использованием учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD с Kantega SSO for Bamboo, вам потребуется:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/));
* подписка Kantega SSO for Bamboo с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Kantega SSO for Bamboo поддерживает вход, инициированный **пакетом обновления и IDP**.

## <a name="adding-kantega-sso-for-bamboo-from-the-gallery"></a>Добавление Kantega SSO for Bamboo из коллекции

Чтобы настроить интеграцию Kantega SSO for Bamboo с Azure AD, необходимо добавить Kantega SSO for Bamboo из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Kantega SSO for Bamboo из коллекции, сделайте следующее.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Kantega SSO for Bamboo**, выберите **Kantega SSO for Bamboo** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Kantega SSO for Bamboo в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описаны настройка и проверка единого входа Azure AD в Kantega SSO for Bamboo с использованием тестового пользователя **Britta Simon**.
Чтобы обеспечить работу единого входа, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Kantega SSO for Bamboo.

Чтобы настроить и проверить единый вход Azure AD в Kantega SSO for Bamboo, вам потребуется выполнить действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Kantega SSO for Bamboo](#configure-kantega-sso-for-bamboo-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя Kantega SSO for Bamboo](#create-kantega-sso-for-bamboo-test-user)** требуется для того, чтобы в Kantega SSO for Bamboo существовал пользователь Britta Simon, связанный с представлением этого же пользователя в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Kantega SSO for Bamboo, выполните следующие действия.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Kantega SSO for Bamboo** выберите **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** выполните следующие действия.

    ![Снимок экрана, на котором показан раздел "Базовая конфигурация SAML", где можно ввести идентификатор, URL-адрес ответа и нажать кнопку "Сохранить"](common/idp-intiated.png)

    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `https://<server-base-url>/plugins/servlet/no.kantega.saml/sp/<uniqueid>/login`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<server-base-url>/plugins/servlet/no.kantega.saml/sp/<uniqueid>/login`.

5. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    ![Снимок экрана, на котором показан параметр "Задать дополнительные URL-адреса" и поле для ввода URL-адреса входа](common/metadata-upload-additional-signon.png)

    В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://<server-base-url>/plugins/servlet/no.kantega.saml/sp/<uniqueid>/login`.

    > [!NOTE]
    > Эти значения приведены для примера. Замените их фактическими значениями идентификатора, URL-адреса ответа и URL-адреса входа. Эти значения предоставляются во время настройки подключаемого модуля Bamboo, которая описывается далее в этом руководстве.

6. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

7. Скопируйте требуемые URL-адреса в разделе **Настройка Kantega SSO for Bamboo**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

### <a name="configure-kantega-sso-for-bamboo-single-sign-on"></a>Настройка единого входа в Kantega SSO for Bamboo

1. В другом окне веб-браузера войдите на корпоративный сайт Bamboo с правами администратора.

1. Наведите указатель мыши на шестеренку и щелкните **Add-ons** (Надстройки).

    ![Снимок экрана: меню "Параметры" с выбранным пунктом "Надстройки".](./media/kantegassoforbamboo-tutorial/addon1.png)

1. На вкладке "Add-ons" (Надстройки) щелкните **Find new add-ons** (Найти новые надстройки). Найдите подключаемый модуль **Kantega SSO for Bamboo (SAML & Kerberos)** и нажмите кнопку **Install** (Установить), чтобы установить новый подключаемый модуль SAML.

    ![Снимок экрана: администрирование Bamboo с выбранным параметром Kantega SSO for Bamboo.](./media/kantegassoforbamboo-tutorial/addon2.png)

1. Начнется установка подключаемого модуля.

    ![Снимок экрана: ход выполнения установки для Kantega SSO for Bamboo.](./media/kantegassoforbamboo-tutorial/addon21.png)

1. Установка завершится. Щелкните **Закрыть**.

    ![Снимок экрана: кнопка Close (Закрыть).](./media/kantegassoforbamboo-tutorial/addon33.png)

1. Нажмите кнопку **Управление**.

    ![Снимок экрана: кнопка Manage (Управление).](./media/kantegassoforbamboo-tutorial/addon34.png)

1. Щелкните **Configure** (Настройка), чтобы настроить новый подключаемый модуль.

    ![Снимок экрана: установленные пользователем надстройки с выбранной кнопкой Configure (Настройка).](./media/kantegassoforbamboo-tutorial/addon3.png)

1. В разделе **SAML** сделайте следующее. Выберите **Azure Active Directory (Azure AD)** из раскрывающегося списка **Добавление поставщика удостоверений**.

    ![Снимок экрана: раздел Kantega Single Sign-On (Единый вход в Kantega) с вариантом Azure AD, выбранным в качестве поставщика удостоверений.](./media/kantegassoforbamboo-tutorial/addon4.png)

1. Выберите уровень подписки **Базовый**.

    ![Снимок экрана: раздел Preparing Azure AD (Подготовка Azure AD) с выбранным вариантом Basic (Базовый).](./media/kantegassoforbamboo-tutorial/addon5.png)

1. В разделе **Свойства приложения** выполните следующие действия.

    ![Снимок экрана: раздел App properties (Свойства приложения), где можно добавить сведения на этом шаге.](./media/kantegassoforbamboo-tutorial/addon6.png)

    а. Скопируйте значение **App ID URI** (URI кода приложения) и используйте его как **идентификатор, URL-адрес ответа и URL-адрес входа** в разделе **Базовая конфигурация SAML** на портале Azure.

    b. Щелкните **Далее**.

1. В разделе **Metadata import** (Импорт метаданных) выполните следующие действия.

    ![Снимок экрана: раздел Metadata import (Импорт метаданных), где можно перейти к файлу метаданных.](./media/kantegassoforbamboo-tutorial/addon7.png)

    а. Щелкните **Metadata file on my computer** (Файл метаданных на моем компьютере) и передайте файл метаданных, который вы скачали с портала Azure.

    b. Щелкните **Далее**.

1. В разделе **Name and SSO location** (Имя и расположение единого входа) выполните следующее.

    ![Снимок экрана: раздел Name and SSO location (Имя и расположение единого входа), где Azure AD — имя поставщика удостоверений.](./media/kantegassoforbamboo-tutorial/addon8.png)

    а. В текстовом поле **Provider Name** (Имя поставщика) введите имя поставщика (например, Azure AD).

    b. Щелкните **Далее**.

1. Проверьте сертификат для подписи и нажмите кнопку **Next** (Далее).

    ![Снимок экрана: раздел Signature Verification (Проверка подписи).](./media/kantegassoforbamboo-tutorial/addon9.png)

1. В разделе **Bamboo user accounts** (Учетные записи пользователей Bamboo) выполните следующие действия.

    ![Снимок экрана: раздел Bamboo user accounts (Учетные записи пользователей Bamboo) с возможностью создания пользователей.](./media/kantegassoforbamboo-tutorial/addon10.png)

    а. Щелкните переключатель **Create users in Bamboo's internal Directory if needed** (При необходимости создать пользователей во внутреннем каталоге Bamboo) и введите соответствующее имя группы пользователей (это может быть несколько групп, разделенных запятой).

    b. Щелкните **Далее**.

1. Нажмите кнопку **Готово**.

    ![Снимок экрана: страница Summary (Сводка).](./media/kantegassoforbamboo-tutorial/addon11.png)

1. В разделе **Known domains for Azure AD** (Известные домены для Azure AD) выполните следующие действия.

    ![Снимок экрана: раздел Known domains for Azure AD (Известные домены для Azure AD), где можно выполнить эти шаги.](./media/kantegassoforbamboo-tutorial/addon12.png)

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

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к Kantega SSO for Bamboo.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем **Kantega SSO for Bamboo**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Kantega SSO for Bamboo**.

    ![Ссылка на Kantega SSO for Bamboo в списке "Приложения"](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-kantega-sso-for-bamboo-test-user"></a>Создание тестового пользователя Kantega SSO for Bamboo

Чтобы пользователи Azure AD могли выполнять вход в Bamboo, для них нужно выполнить подготовку к работе в Bamboo. Например, для Kantega SSO for Bamboo подготовка выполняется вручную.

**Чтобы подготовить учетную запись пользователя, сделайте следующее:**

1. Войдите на локальный сервер Bamboo с правами администратора.

1. Наведите указатель мыши на шестеренку и щелкните **User management** (Управление пользователями).

    ![Снимок экрана: меню "Параметры" с выбранным пунктом User management (Управление пользователями).](./media/kantegassoforbamboo-tutorial/user1.png)

1. Выберите раздел **Пользователи**. В разделе **Add user** (Добавление пользователя) выполните следующие действия.

    ![Снимок экрана: панель Add user (Добавление пользователя), где выполняются эти действия.](./media/kantegassoforbamboo-tutorial/user2.png)

    а. В текстовом поле **Username** (Имя пользователя) введите электронный адрес пользователя, например Brittasimon@contoso.com.

    b. В текстовом поле **Password** (Пароль) введите пароль пользователя.

    c. В текстовом поле **Confirm Password** (Подтверждение пароля) введите пароль еще раз.

    d. В текстовом поле **Full Name** (Полное имя) введите полное имя пользователя, например Britta Simon.

    д) В текстовом поле **Email** (Электронная почта) введите адрес электронной почты пользователя, например Brittasimon@contoso.com.

    е) Выберите команду **Сохранить**.

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку Kantega SSO for Bamboo на Панели доступа, вы автоматически войдете в приложение Kantega SSO for Bamboo, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)