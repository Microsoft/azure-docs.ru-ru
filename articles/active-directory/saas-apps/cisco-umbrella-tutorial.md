---
title: Руководство по Интеграция Azure Active Directory с Cisco Umbrella | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Cisco Umbrella.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/17/2019
ms.author: jeedes
ms.openlocfilehash: dde618b28e004e87edc2783bc44c5e7dd9f0ebba
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/18/2020
ms.locfileid: "97670678"
---
# <a name="tutorial-azure-active-directory-integration-with-cisco-umbrella"></a>Руководство по Интеграция Azure Active Directory с Cisco Umbrella

В этом руководстве описано, как интегрировать Cisco Umbrella с Azure Active Directory (Azure AD).
Интеграция Azure AD с Cisco Umbrella обеспечивает следующие преимущества.

* С помощью Azure AD вы можете контролировать доступ к Cisco Umbrella.
* Вы можете включить автоматический вход пользователей в Cisco Umbrella (единый вход) с помощью учетной записи Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с Cisco Umbrella, вам потребуется:

* Подписка Azure AD. (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* Подписка с поддержкой единого входа в Cisco Umbrella.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Cisco Umbrella поддерживает запущенный единый вход **SP и IDP**.

## <a name="adding-cisco-umbrella-from-the-gallery"></a>Добавление Cisco Umbrella из коллекции

Чтобы настроить интеграцию Cisco Umbrella с Azure AD, необходимо добавить Cisco Umbrella из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Cisco Umbrella из коллекции, сделайте следующее:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Cisco Umbrella**, выберите **Cisco Umbrella** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Cisco Umbrella в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в [название приложения] с использованием тестового пользователя **Britta Simon**.
Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в [название приложения].

Чтобы настроить и проверить единый вход Azure AD в [название приложения], вам потребуется выполнить действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Cisco Umbrella](#configure-cisco-umbrella-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя Cisco Umbrella](#create-cisco-umbrella-test-user)** требуется для того, чтобы в Cisco Umbrella существовал пользователь Britta Simon, связанный с представлением этого же пользователя в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в [название приложения], выполните следующие действия.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Cisco Umbrella** выберите **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** не нужно выполнять никаких действий, так как приложение уже предварительно интегрировано с Azure.

    ![Сведения о домене и URL-адресах единого входа приложения Cisco Umbrella](common/both-preintegrated-signon.png)

    а. Если вы хотите настроить приложение в **режиме, инициируемом поставщиком услуг**, сделайте следующее:

    b. Щелкните **Задать дополнительные URL-адреса**.

    c. В текстовом поле **URL-адрес для входа** введите URL-адрес в формате `https://login.umbrella.com/sso`.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

6. Скопируйте требуемый URL-адрес из раздела **Настройка Cisco Umbrella**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD.

    c. URL-адрес выхода.

### <a name="configure-cisco-umbrella-single-sign-on"></a>Настройка единого входа в Cisco Umbrella

1. В другом окне браузера войдите на сайт Cisco Umbrella компании в качестве администратора.

2. В левой части меню щелкните **Admin** (Администратор) и перейдите к пункту **Authentication** (Проверка подлинности), а затем щелкните **SAML**.

    ![Администратор](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_admin.png)

3. Выберите **Other** (Другое) и щелкните **Next** (Далее).

    ![Другое](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_other.png)

4. На странице **Cisco Umbrella Metadata** (Метаданные Cisco Umbrella) нажмите кнопку **Next** (Далее).

    ![Метаданные](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_metadata.png)

5. На вкладке **Upload Metadata** (Отправка метаданных), если вы предварительно настроили SAML, выберите параметр **Click here to change them** (Щелкните здесь, чтобы изменить их) и сделайте следующее.

    ![Далее](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_next.png)

6. В разделе **Option A: Upload XML file** (Вариант А. Отправка XML-файла) отправьте файл **XML метаданных федерации**, скачанный с портала Azure, после чего приведенные ниже значения заполнятся автоматически, а затем щелкните **Next** (Далее).

    ![Выбор файла](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_choosefile.png)

7. В разделе **Validate SAML Configuration** (Проверка конфигурации SAML) щелкните **Test your SAML configuration** (Тестирование конфигурации SAML).

    ![Тестирование](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_test.png)

8. Щелкните **СОХРАНИТЬ**.

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

В этом разделе описано, как предоставить пользователю Britta Simon доступ к Cisco Umbrella, чтобы он мог использовать единый вход Azure.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем **Cisco Umbrella**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений введите и выберите **Cisco Umbrella**.

    ![Ссылка на Cisco Umbrella в списке "Приложения"](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-cisco-umbrella-test-user"></a>Создание тестового пользователя Cisco Umbrella

Чтобы пользователи Azure AD могли выполнить вход в Cisco Umbrella, они должны быть подготовлены для Cisco Umbrella.  
В случае с Cisco Umbrella подготовка выполняется вручную.

**Чтобы подготовить учетную запись пользователя, сделайте следующее:**

1. В другом окне браузера войдите на сайт Cisco Umbrella компании в качестве администратора.

2. В левой части меню щелкните **Admin** (Администратор) и перейдите к пункту **Accounts** (Учетные записи).

    ![Учетная запись](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_account.png)

3. На странице **Accounts** (Учетные записи) нажмите кнопку **Add** (Добавить) в правом верхнем углу страницы и сделайте следующее.

    ![Пользователь](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_createuser.png)

    а. В поле **First Name** (Имя) введите имя, например **Britta**.

    b. В поле **Last Name** (Фамилия) введите фамилию, например **Simon**.

    c. В разделе **Choose Delegated Admin Role** (Выбор делегированной роли администратора) выберите свою роль.

    d. В поле **Email address** (Адрес электронной почты) введите адрес электронной почты пользователя, например **brittasimon\@contoso.com**.

    д) Введите пароль в поле **Password** (Пароль).

    е) Введите пароль повторно в поле **Confirm Password** (Повторите пароль).

    ж. Нажмите кнопку **Create** (Создать).

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку Cisco Umbrella на панели доступа, вы автоматически войдете в приложение Cisco Umbrella, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)