---
title: 'Руководство: интеграция Azure Active Directory с Certain Admin SSO | Документы Майкрософт'
description: Сведения о настройке единого входа между Azure Active Directory и Certain Admin SSO.
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
ms.openlocfilehash: 03a216f24013413fd03de9ae71eb599fd1334998
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97672969"
---
# <a name="tutorial-azure-active-directory-integration-with-certain-admin-sso"></a>Руководство: интеграция Azure Active Directory с Certain Admin SSO

В этом руководстве описано, как интегрировать Certain Admin SSO с Azure Active Directory (Azure AD).
Интеграция Certain Admin SSO с Azure AD обеспечивает следующие преимущества:

* С помощью Azure AD вы можете контролировать доступ к Certain Admin SSO.
* Вы можете включить автоматический вход пользователей в Certain Admin SSO (единый вход) с помощью учетной записи Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с Certain Admin SSO, вам потребуется:

* Подписка Azure AD. (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка Certain Admin SSO с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* В Certain Admin SSO поддерживается единый вход, инициированный **поставщиком услуг**.

## <a name="adding-certain-admin-sso-from-the-gallery"></a>Добавление Certain Admin SSO из коллекции

Чтобы настроить интеграцию Certain Admin SSO с Azure AD, необходимо добавить Certain Admin SSO из коллекции в список управляемых приложений SaaS.

**Чтобы добавить приложение Certain Admin SSO из коллекции, сделайте следующее:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Certain Admin SSO**, выберите **Certain Admin SSO** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Certain Admin SSO в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описывается настройка и проверка единого входа Azure AD в Certain Admin SSO с использованием тестового пользователя **Britta Simon**.
Чтобы обеспечить работу единого входа, нужно установить связь между пользователем Azure AD и соответствующим пользователем в Certain Admin SSO.

Чтобы настроить и проверить единый вход Azure AD в Certain Admin SSO, вам потребуется выполнить действия в следующих стандартных блоках:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Certain Admin SSO](#configure-certain-admin-sso-single-sign-on)** требуется для того, чтобы определить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя Certain Admin SSO](#create-certain-admin-sso-test-user)** требуется для того, чтобы в Certain Admin SSO существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Certain Admin SSO, сделайте следующее:

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Certain Admin SSO** щелкните **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    ![Сведения о домене и URL-адресах единого входа для приложения Certain Admin SSO](common/sp-identifier.png)

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<YOUR DOMAIN URL>/svcs/sso_admin_login/handleRequest/<ID>`.

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `https://<SUBDOMAIN>.certain.com`.

    > [!NOTE]
    > Эти значения приведены для примера. Необходимо обновить эти значения действующим URL-адресом для входа и идентификатором. Чтобы получить эти значения, обратитесь в [службу поддержки Certain Admin SSO](mailto:integrations@certain.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (необработанный)** из предложенных вариантов и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificateraw.png)

6. Скопируйте требуемый URL-адрес в разделе **Настройка Certain Admin SSO**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD.

    c. URL-адрес выхода.

### <a name="configure-certain-admin-sso-single-sign-on"></a>Настройка единого входа в Certain Admin SSO

Чтобы настроить единый вход на стороне **Certain Admin SSO**, отправьте скачанный **сертификат (необработанный)** и соответствующие URL-адреса, скопированные на портале Azure, в [службу поддержки Certain Admin SSO](mailto:integrations@certain.com). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

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

В этом разделе описано, как предоставить пользователю Britta Simon доступ к Certain Admin SSO, чтобы он мог использовать единый вход Azure.

1. На портале Azure щелкните **Корпоративные приложения**, **Все приложения** и **Certain Admin SSO**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Certain Admin SSO**.

    ![Ссылка на Certain Admin SSO в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-certain-admin-sso-test-user"></a>Создание тестового пользователя Certain Admin SSO

В этом разделе описано, как создать пользователя Britta Simon в приложении Certain Admin SSO. Чтобы добавить пользователей на платформе Certain Admin SSO, обратитесь в [службу поддержки Certain Admin SSO](mailto:integrations@certain.com). Перед использованием единого входа необходимо создать и активировать пользователей.

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку Certain Admin SSO на Панели доступа, вы автоматически войдете в приложение Certain Admin SSO, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)