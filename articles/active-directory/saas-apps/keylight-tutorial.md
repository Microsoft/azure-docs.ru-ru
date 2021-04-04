---
title: Руководство по интеграции Azure Active Directory с LockPath Keylight | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в LockPath Keylight.
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
ms.openlocfilehash: 81949736603d22cac779d08d14bd6db65065d730
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92459091"
---
# <a name="tutorial-azure-active-directory-integration-with-lockpath-keylight"></a>Руководство по интеграции Azure Active Directory с LockPath Keylight

В этом руководстве описано, как интегрировать LockPath Keylight с Azure Active Directory (Azure AD).
Интеграция Azure AD с приложением LockPath Keylight обеспечивает следующие преимущества:

* С помощью Azure AD вы можете контролировать доступ к LockPath Keylight.
* Вы можете включить автоматический вход пользователей в LockPath Keylight (единый вход) с использованием учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD с LockPath Keylight, вам потребуется:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/));
* подписка LockPath Keylight с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* LockPath Keylight поддерживает единый вход, инициируемый **поставщиком услуг**.
* LockPath Keylight поддерживает **JIT**-подготовку пользователей.

## <a name="adding-lockpath-keylight-from-the-gallery"></a>Добавление LockPath Keylight из коллекции

Чтобы настроить интеграцию LockPath Keylight с Azure AD, необходимо добавить LockPath Keylight из коллекции в список управляемых приложений SaaS.

**Чтобы добавить LockPath Keylight из коллекции, выполните следующие действия.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **LockPath Keylight**, выберите **LockPath Keylight** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![LockPath Keylight в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в LockPath Keylight с использованием тестового пользователя **Britta Simon**.
Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем LockPath Keylight.

Чтобы настроить и проверить единый вход Azure AD в LockPath Keylight, вам потребуется выполнить действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в LockPath Keylight](#configure-lockpath-keylight-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя LockPath Keylight](#create-lockpath-keylight-test-user)** требуется для того, чтобы в LockPath Keylight существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в LockPath Keylight, выполните следующие действия.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **LockPath Keylight** щелкните **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    ![Сведения о домене и URL-адресах единого входа для LockPath Keylight](common/sp-identifier-reply.png)

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<company name>.keylightgrc.com/`.

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `https://<company name>.keylightgrc.com`.

    c. В текстовом поле **URL-адрес ответа** введите URL-адрес в следующем формате: `https://<company name>.keylightgrc.com/Login.aspx`.

    > [!NOTE]
    > Эти значения приведены для примера. Вместо них необходимо указать фактические значения URL-адреса входа, идентификатора и URL-адреса ответа. Чтобы получить эти значения, обратитесь к [группе поддержки клиентов LockPath Keylight](https://www.lockpath.com/contact/). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (необработанный)** из предложенных вариантов и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificateraw.png)

6. Требуемые URL-адреса можно скопировать из раздела **Настройка LockPath Keylight**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

### <a name="configure-lockpath-keylight-single-sign-on"></a>Настройка единого входа в LockPath Keylight

1. Чтобы включить единый вход в LockPath Keylight, выполните следующие действия.

    а. Войдите в учетную запись LockPath Keylight в качестве администратора.

    b. В меню вверху щелкните **Person** (Пользователь) и выберите **Keylight Setup** (Настройка Keylight).

    ![Снимок экрана, на котором выделен значок пользователя и выбрано действие настройки Keylight в раскрывающемся меню.](./media/keylight-tutorial/401.png)

    c. В представлении в виде дерева слева щелкните **SAML**.

    ![Снимок экрана для представления в виде дерева, где выбран элемент SAML.](./media/keylight-tutorial/402.png)

    d. В диалоговом окне **SAML Settings** (Параметры SAML) нажмите кнопку **Edit** (Изменить).

    ![Снимок экрана, на котором показано диалоговое окно параметров SAML и выбрана кнопка "Изменить".](./media/keylight-tutorial/404.png)

1. На странице диалогового окна **Изменение параметров SAML** выполните следующие действия:

    ![Настройка единого входа](./media/keylight-tutorial/405.png)

    а. Задайте для параметра **SAML authentication** (Аутентификация SAML) значение **Active** (Активно).

    b. В текстовое поле **Identity Provider Login URL** (URL-адрес входа поставщика удостоверений) вставьте значение **URL-адрес входа**, скопированное на портале Azure.

    c. В текстовое поле **Identity Provider Logout URL** (URL-адрес выхода поставщика удостоверений) вставьте значение **URL-адрес выхода**, скопированное на портале Azure.

    d. Щелкните **Choose File** (Выбрать файл), чтобы выбрать скачанный сертификат LockPath Keylight, а затем нажмите кнопку **Open** (Открыть), чтобы передать этот сертификат.

    д) В поле **SAML User Id location** (Расположение идентификатора пользователя SAML) выберите значение **NameIdentifier element of the subject statement** (Элемент NameIdentifier оператора Subject).

    е) Введите значение **Keylight Service Provider** (Поставщик услуг Keylight), используя следующий формат: `https://<CompanyName>.keylightgrc.com`.

    ж. Задайте для параметра **Auto-provision users** (Автоматическая подготовка пользователей) значение **Active** (Активно).

    h. Задайте для параметра **Auto-provision account type** (Тип учетной записи для автоматической подготовки) значение **Full User** (С полным доступом).

    i. Задайте для параметра **Auto-provision security role** (Роль безопасности для автоматической подготовки) значение **Standard User with SAML** (Права обычного пользователя с SAML).

    j. Задайте для параметра **Auto-provision security config** (Конфигурация безопасности для автоматической подготовки) значение **Standard User Configuration** (Конфигурация обычного пользователя).

    k. В текстовое поле **Email Attribute** (Атрибут электронной почты) введите `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`.

    l. В текстовое поле **First name attribute** (Атрибут имени) введите значение `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`.

    m. В текстовое поле **Last name attribute** (Атрибут фамилии) введите значение `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`.

    n. Выберите команду **Сохранить**.

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

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к LockPath Keylight.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **LockPath Keylight**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **LockPath Keylight**.

    ![Ссылка на LockPath Keylight в списке "Приложения"](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-lockpath-keylight-test-user"></a>Создание тестового пользователя LockPath Keylight

В этом разделе описано, как создать пользователя Britta Simon в приложении LockPath Keylight. LockPath Keylight поддерживает JIT-подготовку пользователей. Эта функция включена по умолчанию. В этом разделе никакие действия с вашей стороны не требуются. Если пользователь еще не существует в LockPath Keylight, он создается после выполнения аутентификации. Если вам нужно вручную создать пользователя, необходимо обратиться к [группе поддержки клиентов LockPath Keylight](https://www.lockpath.com/contact/).

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув элемент "LockPath Keylight" на Панели доступа, вы автоматически войдете в приложение LockPath Keylight, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)