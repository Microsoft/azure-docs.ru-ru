---
title: Руководство по интеграции Azure Active Directory с Attendance Management Services | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Attendance Management Services.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/15/2019
ms.author: jeedes
ms.openlocfilehash: 1f404d3613f9de8daadc4bb2ceb39282cf3b619e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101689000"
---
# <a name="tutorial-azure-active-directory-integration-with-attendance-management-services"></a>Руководство по интеграции Azure Active Directory с Attendance Management Services

В этом руководстве описано, как интегрировать Attendance Management Services с Azure Active Directory (Azure AD).
Интеграция Azure AD с Attendance Management Services обеспечивает следующие преимущества.

* С помощью Azure AD вы можете контролировать доступ к Attendance Management Services.
* Вы можете включить автоматический вход пользователей в Attendance Management Services (единый вход) с помощью учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD с Attendance Management Services, вам потребуется следующее:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/));
* подписка Attendance Management Services с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Attendance Management Services поддерживает единый вход, инициируемый **поставщиком услуг**.

## <a name="adding-attendance-management-services-from-the-gallery"></a>Добавление Attendance Management Services из коллекции

Чтобы настроить интеграцию Attendance Management Services с Azure AD, вам потребуется добавить Attendance Management Services из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Attendance Management Services из коллекции, выполните следующее.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Attendance Management Services**, выберите **Attendance Management Services** на панели результатов, а затем нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Attendance Management Services в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в Attendance Management Services с использованием тестового пользователя **Britta Simon**.
Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Attendance Management Services.

Чтобы настроить и проверить единый вход Azure AD в Attendance Management Services, вам потребуется выполнить следующие действия.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Attendance Management Services](#configure-attendance-management-services-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя Attendance Management Services](#create-attendance-management-services-test-user)** требуется для создания соответствующего пользователя Britta Simon в Attendance Management Services, связанного с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Attendance Management Services, выполните следующее.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Attendance Management Services** щелкните **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    ![Сведения о домене и URL-адресах единого входа для Attendance Management Services](common/sp-identifier.png)

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://id.obc.jp/<tenant information >/`.

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `https://id.obc.jp/<tenant information >/`.

    > [!NOTE]
    > Эти значения приведены для примера. Необходимо обновить эти значения действующим URL-адресом для входа и идентификатором. Обратитесь к [группу поддержки Attendance Management Services](https://www.obcnet.jp/) для получения этих значений. Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (Base64)** из предложенных вариантов, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

6. Требуемые URL-адреса можно скопировать из раздела **Настройка Attendance Management Services**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

### <a name="configure-attendance-management-services-single-sign-on"></a>Настройка единого входа в Attendance Management Services

1. В другом окне браузера войдите на свой корпоративный сайт Attendance Management Services в качестве администратора.

1. Щелкните **SAML authentication** (Аутентификация SAML) в разделе **Security management** (Управление безопасностью).

    ![Снимок экрана: элемент "Аутентификация SAML", выбранный на странице, где используются символы, отличные от латинских.](./media/attendancemanagementservices-tutorial/user1.png)

1. Выполните следующие действия:

    ![Снимок экрана: окно, в котором можно выполнять задачи, описанные в этом шаге.](./media/attendancemanagementservices-tutorial/user2.png)

    а. Установите флажок **Use SAML authentication** (Использовать аутентификацию SAML).

    b. В текстовое поле **Identifier** (Идентификатор) вставьте значение **Идентификатор Azure AD**, скопированное на портале Azure.

    c. В текстовое поле **Authentication endpoint URL** (URL-адрес конечной точки аутентификации) вставьте значение **URL-адрес входа**, скопированное на портале Azure.

    d. Щелкните **Select a file** (Выбрать файл), чтобы передать сертификат, скачанный из Azure AD.

    д) Установите флажок **Disable password authentication** (Отключить проверку пароля).

    е) Щелкните **Registration** (Регистрация).

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

В этом разделе описано, как предоставить пользователю Britta Simon доступ к Attendance Management Services, чтобы этот пользователь мог использовать единый вход Azure.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **Attendance Management Services**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Attendance Management Services**.

    ![Ссылка на Attendance Management Services в списке "Приложения"](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-attendance-management-services-test-user"></a>Создание тестового пользователя Attendance Management Services

Чтобы пользователи Azure AD могли входить в Attendance Management Services, их необходимо подготовить к работе в Attendance Management Services. В случае Attendance Management Services подготовка выполняется вручную.

**Чтобы подготовить учетную запись пользователя, сделайте следующее:**

1. Войдите на свой корпоративный сайт Attendance Management Services с правами администратора.

1. Щелкните **User management** (Управление пользователями) в разделе **Security management** (Управление безопасностью).

    ![Снимок экрана: элемент "Управление пользователями", выбранный на странице, где используются символы, отличные от латинских.](./media/attendancemanagementservices-tutorial/user5.png)

1. Щелкните **New rules login** (Создать имя для входа на основе правил).

    ![Снимок экрана: выбор кнопки со знаком "плюс".](./media/attendancemanagementservices-tutorial/user3.png)

1. В разделе **OBCiD information** (Сведения OBCiD) сделайте следующее.

    ![Снимок экрана: окно, в котором можно выполнять описанные задачи.](./media/attendancemanagementservices-tutorial/user4.png)

    а. В текстовое поле **OBCiD** введите электронную почту пользователя, например `BrittaSimon@contoso.com`.

    b. В текстовом поле **Password** (Пароль) введите пароль пользователя.

    c. Щелкните **Registration** (Регистрация).

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув элемент "Attendance Management Services" на панели доступа, вы автоматически войдете в приложение Attendance Management Services, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)