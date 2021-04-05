---
title: Руководство по интеграции Azure Active Directory с Menlo Security | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Menlo Security.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/20/2019
ms.author: jeedes
ms.openlocfilehash: e10045eb14f6a72648fb9534ade077faaa4ef918
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92517333"
---
# <a name="tutorial-azure-active-directory-integration-with-menlo-security"></a>Руководство по интеграции Azure Active Directory с Menlo Security

В этом руководстве описано, как интегрировать Menlo Security с Azure Active Directory (Azure AD).
Интеграция Menlo Security с Azure AD имеет следующие преимущества.

* С помощью Azure AD вы можете контролировать, у кого есть доступ к приложению Menlo Security.
* Вы можете включить автоматический вход для пользователей в Menlo Security (единый вход) с помощью их учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с приложением Menlo Security, вам потребуется:

* Подписка Azure AD. (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка Menlo Security с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Menlo Security поддерживает инициированный единый вход **пакета обновлений**.

## <a name="adding-menlo-security-from-the-gallery"></a>Добавление Menlo Security из коллекции

Чтобы настроить интеграцию приложения Menlo Security с Azure AD, вам нужно добавить это приложение из коллекции в свой список управляемых приложений SaaS.

**Добавление приложения Menlo Security из коллекции**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Menlo Security**, выберите **Menlo Security** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

     ![Menlo Security в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в приложение Menlo Security с использованием тестового пользователя **Britta Simon**.
Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Menlo Security.

Чтобы настроить и проверить единый вход Azure AD в Menlo Security, выполните следующие действия:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Menlo Security](#configure-menlo-security-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя Menlo Security](#create-menlo-security-test-user)** требуется для того, чтобы в Menlo Security существовал пользователь Britta Simon, связанный с представлением этого же пользователя в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Menlo Security, выполните следующие действия:

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Menlo Security** выберите **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    ![Сведения о домене и URL-адресах единого входа для приложения Menlo Security](common/sp-identifier.png)

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<subdomain>.menlosecurity.com/account/login`.

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `https://<subdomain>.menlosecurity.com/safeview-auth-server/saml/metadata`.

    > [!NOTE]
    > Эти значения приведены для примера. Необходимо обновить эти значения действующим URL-адресом для входа и идентификатором. Чтобы получить эти значения, обратитесь в [службу поддержки клиентов Menlo Security](https://www.menlosecurity.com/menlo-contact). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (Base64)** из предложенных вариантов, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

6. Скопируйте требуемый URL-адрес из раздела **Настройка Menlo Security**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

### <a name="configure-menlo-security-single-sign-on"></a>Настройка единого входа Menlo Security

1. Чтобы настроить единый вход в приложении **Menlo Security**, войдите на соответствующий сайт **Menlo Security** в качестве администратора.

2. В разделе **Параметры** выберите **Проверка подлинности** и выполните следующие действия:
    
    ![Настройка единого входа](./media/menlosecurity-tutorial/menlo_user_setup.png)

    а. Установите флажок **Enable user authentication using SAML** (Включить проверку подлинности пользователя с помощью SAML).

    b. Задайте для параметра **Allow External Access** (Разрешить внешний доступ) значение **Да**.

    c. В разделе **SAML Provider** (Поставщик SAML) выберите **Azure Active Directory**.

    d. **SAML 2.0 Endpoint** (Конечная точка SAML 2.0). Вставьте **URL-адрес входа**, скопированный на портале Azure.

    д) **Service Identifier (Issuer)** (Идентификатор службы (издатель)). Вставьте **идентификатор Azure AD**, скопированный на портале Azure.

    е) **Сертификат X.509**. Откройте **сертификат (Base64)**, скачанный на портале Azure, в Блокноте и вставьте его в это поле.

    ж. Нажмите кнопку **Сохранить** , чтобы сохранить параметры.

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

В этом разделе мы разрешим пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к Menlo Security.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **Menlo Security**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Menlo Security**.

    ![Ссылка на Menlo Security в списке "Приложения"](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-menlo-security-test-user"></a>Создание тестового пользователя Menlo Security

В этом разделе описано, как создать пользователя Britta Simon в приложении Menlo Security. Обратитесь в [службу поддержки клиентов Menlo Security](https://www.menlosecurity.com/menlo-contact), чтобы добавить пользователей на платформу Menlo Security. Перед использованием единого входа необходимо создать и активировать пользователей.

### <a name="test-single-sign-on"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку "Menlo Security" на панели доступа, вы автоматически войдете в приложение Menlo Security, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)