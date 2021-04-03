---
title: Руководство по Интеграция Azure Active Directory с Workspot Control | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Workspot Control.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 3/11/2019
ms.author: jeedes
ms.openlocfilehash: 65e424c6275f9990a5d5a42a54bc4131300a45bf
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92895041"
---
# <a name="tutorial-azure-active-directory-integration-with-workspot-control"></a>Руководство по Интеграция Azure Active Directory с Workspot Control

В этом руководстве описано, как интегрировать Workspot Control с Azure Active Directory (Azure AD). Интеграция Workspot Control с Azure AD обеспечивает следующие возможности.

* С помощью Azure AD вы можете контролировать доступ к Workspot Control.
* Вы можете включить автоматический вход пользователей в Workspot Control (единый вход) с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD со Workspot Control, вам потребуется следующее:

* Подписка Azure AD. (если у вас нет среды Azure AD, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/));

* подписка Workspot Control с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

> [!Note]
> Workspot Control поддерживает единый вход, инициируемый поставщиком услуг и поставщиком удостоверений.


## <a name="adding-workspot-control-from-the-gallery"></a>Добавление Workspot Control из коллекции

Чтобы настроить интеграцию Workspot Control с Azure AD, необходимо добавить приложение Workspot Control из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Workspot Control из коллекции, выполните следующие действия**.

1. На [портале Azure](https://portal.azure.com) в области слева щелкните **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите к разделу **Корпоративные приложения** и выберите **Все приложения**.

    ![Область "Корпоративные приложения"](common/enterprise-applications.png)

3. Выберите **Новое приложение** в верхней части окна.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Workspot Control**, выберите **Workspot Control** в области результатов, а затем щелкните **Добавить**.

     ![Окно "Добавление из коллекции"](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в Workspot Control с использованием тестового пользователя Britta Simon.
Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Workspot Control.

Чтобы настроить и проверить единый вход Azure AD в Workspot Control, потребуется выполнить следующие задачи.

1. [Настроить единый вход Azure AD](#configure-azure-ad-single-sign-on), чтобы пользователи могли использовать эту функцию.
2. [Настройка единого входа в Workspot Control](#configure-workspot-control-single-sign-on) необходима, чтобы настроить параметры единого входа на стороне приложения.
3. [Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user) необходимо, чтобы проверить работу единого входа Azure AD от имени пользователя Britta Simon.
4. [Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user) необходимо, чтобы разрешить Britta Simon использовать единый вход Azure AD.
5. [Создание тестового пользователя Workspot Control](#create-a-workspot-control-test-user) требуется для того, чтобы в Workspot Control существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. [Проверьте единый вход](#test-single-sign-on), чтобы убедиться, что конфигурация работает правильно.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure Active Directory в Workspot Control, сделайте следующее.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Workspot Control** выберите **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В окне **Выбрать метод единого входа** выберите режим **SAML**, чтобы включить единый вход.

    ![Окно "Выбрать метод единого входа"](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить** (значок карандаша), чтобы открыть область **Базовая конфигурация SAML**.

    ![Выделенный значок "Изменить" в области "Базовая конфигурация SAML"](common/edit-urls.png)

4. Если вы хотите настроить приложение в режиме, инициируемом поставщиком удостоверений, в разделе **Базовая конфигурация SAML** выполните следующие действия.

    ![Снимок экрана, на котором показан раздел "Базовая конфигурация SAML", где можно ввести идентификатор, URL-адрес ответа и нажать кнопку "Сохранить"](common/idp-intiated.png)

    1. В текстовое поле **Идентификатор** введите URL-адрес в следующем формате:<br/>
    ***https://<<i></i>ИМЯ_ЭКЗЕМПЛЯРА>-saml.workspot.com/saml/metadata***

    1. В текстовое поле **URL-адрес ответа** введите URL-адрес в следующем формате:<br/>
    ***https://<<i></i>ИМЯ_ЭКЗЕМПЛЯРА>-saml.workspot.com/saml/assertion***

5. Если вы хотите настроить приложение для работы в режиме, инициируемом поставщиком услуг, выберите **Задать дополнительные URL-адреса**.

    ![Снимок экрана, на котором показан параметр "Задать дополнительные URL-адреса" и поле для ввода URL-адреса входа](common/metadata-upload-additional-signon.png)

    В текстовое поле **URL-адрес входа** введите URL-адрес в следующем формате:<br/>
    ***https://<<i></i>ИМЯ_ЭКЗЕМПЛЯРА>-saml.workspot.com/***

    > [!NOTE]
    > Эти значения приведены для примера. Замените их фактическими значениями идентификатора, URL-адреса ответа и URL-адреса входа. Чтобы получить эти значения, обратитесь к [группе поддержки клиентов Workspot Control](mailto:support@workspot.com). Можно также просмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

6. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать требуемый **сертификат (Base64)** из предложенных вариантов. Сохраните его на компьютере.

    ![Ссылки для скачивания сертификата в кодировке Base64](common/certificatebase64.png)

7. Требуемые URL-адреса можно скопировать из раздела **Настройка Workspot Control**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    - **Login URL** (URL-адрес для входа)

    - **идентификатор Azure AD**;

    - **URL-адрес выхода**.

### <a name="configure-workspot-control-single-sign-on"></a>Настройка единого входа в Workspot Control

1. В другом окне веб-браузера войдите в Workspot Control с правами администратора безопасности.

2. На панели инструментов в верхней части страницы щелкните **Setup** (Настройка), затем — **SAML**.

    ![Параметры установки](./media/workspotcontrol-tutorial/tutorial_workspotcontrol_setup.png)

3. В окне **Security Assertion Markup Language Configuration** (Конфигурация SAML) выполните следующие действия.
 
    ![Окно "Security Assertion Markup Language Configuration" (Конфигурация SAML)](./media/workspotcontrol-tutorial/tutorial_workspotcontrol_saml.png)

    1. В поле **Entity ID** (Идентификатор сущности) вставьте значение **Идентификатор Azure AD**, скопированное на портале Azure.

    1. В поле **Signon Service URL** (URL-адрес службы входа) вставьте значение **URL-адрес входа**, скопированное на портале Azure.

    1. В поле **Logout Service URL** (URL-адрес службы выхода) вставьте значение **URL-адрес выхода**, скопированное на портале Azure.

    1. Щелкните **Update File** (Обновить файл), чтобы передать сертификат в кодировке Base64, который был скачан с портала Azure, в сертификат X.509.

    1. Щелкните **Сохранить**.

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD

В этом разделе описано, как создать на портале Azure тестового пользователя.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.

    ![Ссылки "Пользователи и группы" и "Все пользователи"](common/users.png)

2. В верхней части окна выберите **Новый пользователь**.

    ![Кнопка "Новый пользователь"](common/new-user.png)

3. В разделе свойств пользователя выполните следующие действия.

    ![Окно "Свойства пользователя"](common/user-properties.png)

    1. В поле **Имя** введите **BrittaSimon**.
  
    1. В поле **Имя пользователя** введите **brittasimon @* ваш_домен.доменная_зона***. Например, введите **BrittaSimon@contoso.<i></i>com**.

    1. Установите флажок **Показать пароль**. Запишите значение, которое отображается в поле **Пароль**.

    1. Нажмите кнопку **создания**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к Workspot Control.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **Workspot Control**.

    ![Область "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Workspot Control**.

    ![Ссылка на Workspot Control в списке "Приложения"](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**. Затем в окне **Добавление назначения** щелкните **Пользователи и группы**.

    ![Окно "Добавление назначения"](common/add-assign-user.png)

5. В окне **Пользователи и группы** из списка **Пользователи** выберите **Britta Simon**. Затем щелкните **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка. Щелкните **Выбрать** внизу.

7. В окне **Добавление назначения** выберите **Назначить**.

### <a name="create-a-workspot-control-test-user"></a>Создание тестового пользователя Workspot Control

Чтобы пользователи Azure AD могли выполнять вход в Workspot Control, их следует подготовить к работе в Workspot Control. В подготовка к работе выполняется вручную.

**Чтобы подготовить к работе учетную запись пользователя, выполните следующие действия.**

1. Войдите в Workspot Control с правами администратора безопасности.

2. На панели инструментов в верхней части страницы щелкните **Setup** (Настройка), затем — **Add User** (Добавить пользователя).

    ![Параметры в окне "Users" (Пользователи)](./media/workspotcontrol-tutorial/tutorial_workspotcontrol_adduser.png)

3. В окне **Add a New User** (Добавление пользователя) выполните следующие действия.

    ![Окно "Add a New User" (Добавление пользователя)](./media/workspotcontrol-tutorial/tutorial_workspotcontrol_addnewuser.png)

    1. В поле **First Name** (Имя) введите имя пользователя, например **Britta**.

    1. В поле **Last Name** (Фамилия) введите фамилию пользователя, например **Simon**.

    1. В поле **Email** (Электронная почта) введите адрес электронной почты пользователя, например **Brittasimon@contoso.<i></i>com**.

    1. Выберите соответствующую роль пользователя из раскрывающегося списка **Role** (Роль).

    1. Выберите соответствующую группу пользователей из раскрывающегося списка **Group** (Группа).

    1. Щелкните **Add User** (Добавить пользователя).

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью *Панели доступа*.

Щелкнув элемент **Workspot Control** на панели доступа, вы автоматически войдете в приложение Workspot Control, для которого настроили единый вход. Дополнительные сведения см. в статье [Общие сведения о панели доступа](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Руководства по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)