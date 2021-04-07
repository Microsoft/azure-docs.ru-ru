---
title: Руководство. Интеграция Azure Active Directory с LoginRadius | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и LoginRadius.
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
ms.openlocfilehash: 832c08123904b9fb889231faa86c1308704a2581
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97606425"
---
# <a name="tutorial-azure-active-directory-integration-with-loginradius"></a>Руководство. Интеграция Azure Active Directory с LoginRadius

В этом руководстве описано, как интегрировать LoginRadius с Azure Active Directory (Azure AD).

Интеграция Azure AD с приложением LoginRadius обеспечивает следующие преимущества:

* С помощью Azure AD вы можете контролировать доступ к LoginRadius.
* Вы можете включить автоматический вход для пользователей в LoginRadius (единый вход) с помощью учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD с LoginRadius, вам потребуется:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/));
* подписка на LoginRadius с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* LoginRadius поддерживает единый вход, инициируемый **поставщиком услуг**.

## <a name="adding-loginradius-from-the-gallery"></a>Добавление LoginRadius из коллекции

Чтобы настроить интеграцию LoginRadius с Azure AD, необходимо добавить LoginRadius из коллекции в список управляемых приложений SaaS.

**Чтобы добавить LoginRadius из коллекции, сделайте следующее:**

1. На **[портале Azure](https://portal.azure.com)** щелкните значок **Azure Active Directory** в области навигации слева.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **LoginRadius**, выберите **LoginRadius** в области результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![LoginRadius в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в LoginRadius с использованием тестового пользователя **Britta Simon**.
Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в LoginRadius.

Чтобы настроить и проверить единый вход Azure AD в LoginRadius, вам потребуется выполнить действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в LoginRadius](#configure-loginradius-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя LoginRadius](#create-loginradius-test-user)** требуется для того, чтобы в LoginRadius существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в LoginRadius, выполните следующие действия.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **LoginRadius** щелкните **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В области **Выбрать метод единого входа** выберите режим **SAML/WS-FED**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните элемент **Изменить**, чтобы открыть панель **Базовая конфигурация SAML**.

    ![Правка базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** сделайте следующее:

   ![Сведения о домене и URL-адресах единого входа для приложения LoginRadius](common/sp-identifier.png)

   1. В текстовом поле **URL-адрес для входа** введите URL-адрес `https://secure.loginradius.com/login`.

   1. В текстовом поле **Идентификатор (сущности)** введите URL-адрес `https://lr.hub.loginradius.com/`.

   1. В текстовом поле **URL-адрес ответа (URL-адрес службы обработчика утверждений)** введите URL-адрес LoginRadius ACS: `https://lr.hub.loginradius.com/saml/serviceprovider/AdfsACS.aspx`. 

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** выберите элемент **Скачать**, чтобы скачать нужный вам **XML-файл метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

6. Требуемые URL-адреса можно скопировать из раздела **настройки LoginRadius**.

   ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

   - URL-адрес входа.

   - Идентификатор Azure AD

   - URL-адрес выхода.

## <a name="configure-loginradius-single-sign-on"></a>Настройка единого входа в LoginRadius

В рамках этого раздела вы включите единый вход в Azure AD в консоли администрирования LoginRadius.

1. Войдите в учетную запись [консоли администрирования](https://adminconsole.loginradius.com/login) LoginRadius.

2. Перейдите к разделу **Team Management** (Управление командой) в [консоли администрирования LoginRadius](https://secure.loginradius.com/account/team).

3. Откройте вкладку **Single Sign-On** (Единый вход) и выберите элемент **Azure AD**.

   ![Снимок экрана с элементом единого входа в меню консоли управления командой LoginRadius](./media/loginradius-tutorial/azure-ad.png)
4. На странице настройки Azure AD выполните следующие действия:

   ![Снимок экрана: настройка Azure Active Directory в консоли управления командой LoginRadius](./media/loginradius-tutorial/single-sign-on.png)

    1. В поле **ID Provider Location** (Расположение поставщика удостоверений) укажите конечную точку входа, которую вы получили из учетной записи Azure AD.

    1. В поле **ID Provider Logout URL** (URL-адрес выхода поставщика удостоверений) укажите конечную точку выхода, которую вы получили из учетной записи Azure AD.
 
    1. В поле **ID Provider Certificate** (Сертификат поставщика удостоверений) укажите сертификат Azure AD, полученный из учетной записи Azure AD. Значение сертификата должно включать верхний и нижний колонтитулы. Например, `-----BEGIN CERTIFICATE-----<certifciate value>-----END CERTIFICATE-----`.

    1. В полях **Service Provider Certificate** (Сертификат поставщика служб) и **Server Provider Certificate Key** (Ключ сертификата поставщика служб) введите соответствующие значения. 

       Вы можете создать самозаверяющий сертификат, выполнив следующие команды в командной строке (Linux или Mac):

       - Команда для получения ключа сертификата поставщика служб: `openssl genrsa -out lr.hub.loginradius.com.key 2048`.

       - Команда для получения сертификата поставщика служб: `openssl req -new -x509 -key lr.hub.loginradius.com.key -out lr.hub.loginradius.com.cert -days 3650 -subj /CN=lr.hub.loginradius.com`.

       > [!NOTE]
       > Обязательно включайте а значения сертификата и ключа верхний и нижний колонтитулы:
       > - Примера формата для сертификата: `-----BEGIN CERTIFICATE-----<certifciate value>-----END CERTIFICATE-----`.
       > - Примера формата для ключа сертификата: `-----BEGIN RSA PRIVATE KEY-----<certifciate key value>-----END RSA PRIVATE KEY-----`.

5. В разделе **Data Mapping** (Сопоставление данных) выберите поля поставщика служб (поля SP) и введите соответствующие поля Azure AD (поля IdP).

    Ниже перечислены примеры полей для Azure AD.

    | Поля    | Ключ профиля                                                          |
    | --------- | -------------------------------------------------------------------- |
    | Email     | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` |
    | FirstName | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`    |
    | LastName  | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`      |

    > [!NOTE]
    > Для поля **Email** сопоставление является обязательным. Для полей **FirstName** и **LastName** сопоставления полей необязательны.

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD 

В этом разделе на портале Azure создается тестовый пользователь с именем Britta Simon.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.

    ![Ссылки "Пользователи и группы" и "Все пользователи"](common/users.png)

2. В верхней части экрана выберите **Новый пользователь**.

    ![Кнопка "Новый пользователь"](common/new-user.png)

3. В разделе свойств **User** (Пользователь) сделайте следующее:

   ![Диалоговое окно "Пользователь"](common/user-properties.png)

   1. В поле **Имя** введите **BrittaSimon**.
  
   1. В поле **Имя пользователя** введите `brittasimon@yourcompanydomain.extension`. Например, BrittaSimon@contoso.com.

   1. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле **Пароль**.

   1. Нажмите кнопку **создания**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к LoginRadius.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **LoginRadius**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **LoginRadius**.

    ![Ссылка на LoginRadius в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем на панели **Добавление назначения** выберите элемент **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** выберите пункт **Britta Simon** в списке **Пользователи**, а затем в нижней части экрана нажмите кнопку **Выбрать**.

6. Если вы планируете получать значение роли в утверждении SAML, на панели **Выбор роли** выберите для пользователя соответствующую роль из списка. Затем в нижней части экрана нажмите кнопку **Выбрать**.

7. В области **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-loginradius-test-user"></a>Создание тестового пользователя LoginRadius

1. Войдите в учетную запись [консоли администрирования](https://adminconsole.loginradius.com/login) LoginRadius.

2. Перейдите к разделу Team Management (Управление командой) в консоли администрирования LoginRadius.

   ![Снимок экрана: консоль администрирования LoginRadius](./media/loginradius-tutorial/team-management.png)
3. Выберите **Add Team Member** (Добавить члена команды) в боковое меню, чтобы открыть форму. 

4. В форме **Add Team Member** (Добавление члена команды) создайте пользователя с именем Britta Simon для сайта LoginRadius, предоставив сведения о пользователе и назначив разрешения, которые должен иметь этот пользователь. Дополнительные сведения о разрешениях, основанных на ролях, см. в разделе [Разрешения на доступ к роли](https://www.loginradius.com/docs/api/v2/admin-console/team-management/manage-team-members#roleaccesspermissions0) в документации LoginRadius [по управлению членами команды](https://www.loginradius.com/docs/api/v2/admin-console/team-management/manage-team-members#roleaccesspermissions0). Перед использованием единого входа необходимо создать и активировать пользователей.

### <a name="test-single-sign-on"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

1. В браузере откройте страницу https://accounts.loginradius.com/auth.aspx и выберите элемент **Fed SSO log in** (Федеративный единый вход).
2. Введите имя приложения LoginRadius и выберите действие **Login** (Вход).
3. Отобразится всплывающее окно с запросом на вход в учетную запись Azure AD.
4. После проверки подлинности это всплывающее окно закроется. В перейдете в консоль администрирования LoginRadius.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)
