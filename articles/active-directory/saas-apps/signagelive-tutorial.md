---
title: Руководство по Интеграция Azure Active Directory с Signagelive | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Signagelive.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 1/11/2019
ms.author: jeedes
ms.openlocfilehash: 78324cfa58a8ac015b085052bdec7e3793befc1b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96348455"
---
# <a name="tutorial-azure-active-directory-integration-with-signagelive"></a>Руководство по Интеграция Azure Active Directory с Signagelive

В этом руководстве описано, как интегрировать Signagelive с Azure Active Directory (Azure AD).
Интеграция Signagelive с Azure AD обеспечивает следующие преимущества:

* В Azure AD вы можете контролировать доступ к Signagelive.
* Вы можете включить автоматический вход пользователей в Signagelive (единый вход) с помощью учетной записи Azure AD.
* Вы можете управлять учетными записями централизованно — через портал Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в руководстве по [получению доступа к приложениям и реализации единого входа с помощью Azure Active Directory](../manage-apps/what-is-single-sign-on.md). Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с Signagelive, вам потребуется:

* Подписка Azure AD. Если у вас нет среды Azure AD, вы можете получить [пробную версию на один месяц](https://azure.microsoft.com/pricing/free-trial/).
* подписка Signagelive с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Signagelive поддерживает единый вход, инициированный поставщиком службы.

## <a name="add-signagelive-from-the-gallery"></a>Добавление Signagelive из коллекции

Чтобы настроить интеграцию Signagelive с Azure AD, необходимо сначала добавить Signagelive из коллекции в список управляемых приложений SaaS.

Чтобы добавить Signagelive из коллекции, сделайте следующее:

1. На [портале Azure](https://portal.azure.com) в области слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Signagelive**. 

     ![Signagelive в списке результатов](common/search-new-app.png)

5. На панели результатов выберите **Signagelive** и нажмите кнопку **Добавить**, чтобы добавить приложение.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD с Signagelive с использованием тестового пользователя **Britta Simon**.
Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Signagelive.

Чтобы настроить и проверить единый вход Azure AD в Signagelive, выполните действия в следующих стандартных блоках.

1. [Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on) необходима, чтобы пользователи могли использовать эту функцию.
2. [Настройка единого входа Signagelive](#configure-signagelive-single-sign-on) необходима, чтобы настроить параметры единого входа на стороне приложения.
3. [Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user) требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. [Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user) необходимо, чтобы разрешить Britta Simon использовать единый вход Azure AD.
5. [Создание тестового пользователя Signagelive](#create-a-signagelive-test-user) требуется для того, чтобы в Signagelive существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. [Проверьте единый вход](#test-single-sign-on), чтобы убедиться, что конфигурация работает правильно.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Signagelive, выполните следующие действия:

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Signagelive** выберите **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбор метода единого входа** выберите режим **SAML**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните следующие действия:

    ![Сведения о домене и URL-адресах единого входа приложения Signagelive](common/sp-signonurl.png)

    В поле **URL-адрес для входа** введите URL-адрес в формате `https://login.signagelive.com/sso/<ORGANIZATIONALUNITNAME>`.

    > [!NOTE]
    > Это значение приведено для примера. Вместо него необходимо указать фактический URL-адрес входа. Чтобы получить это значение, обратитесь в [службу технической поддержки Signagelive](mailto:support@signagelive.com). Можно также просмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать требуемый **сертификат (необработанный)** из предложенных вариантов. Сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificateraw.png)

6. Требуемый URL-адрес вы можете скопировать в разделе **Настройка Signagelive**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

### <a name="configure-signagelive-single-sign-on"></a>Настройка единого входа в Signagelive

Чтобы настроить единый вход на стороне Signagelive, нужно отправить скачанный **сертификат (необработанный)** и соответствующие URL-адреса, скопированные на портале Azure, в [службу поддержки Signagelive](mailto:support@signagelive.com). Там обеспечат правильную настройку единого входа SAML с обеих сторон.

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD 

Цель этого раздела — создать на портале Azure тестового пользователя с именем Britta Simon.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.

    ![Ссылки "Пользователи и группы" и "Все пользователи"](common/users.png)

2. В верхней части экрана выберите **Новый пользователь**.

    ![Кнопка "Новый пользователь"](common/new-user.png)

3. В диалоговом окне **Пользователь** сделайте следующее:

    ![Диалоговое окно "Пользователь"](common/user-properties.png)

    а. В поле **Имя** введите **BrittaSimon**.
  
    b. В поле **Имя пользователя** введите brittasimon@yourcompanydomain.extension. Например, в этом случае можно ввести BrittaSimon@contoso.com.

    c. Установите флажок **Показать пароль** и сохраните значение, которое отображается в поле пароля.

    d. Нажмите кнопку **создания**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив пользователю доступ к Signagelive.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **Signagelive**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Signagelive**.

    ![Ссылка на Signagelive в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**. Затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** в списке **Пользователи** выберите **Britta Simon**. Затем в нижней части экрана нажмите кнопку **Выбрать**.

6. Если вы планируете получать значение роли в утверждении SAML, в диалоговом окне **Выбор ролей** выберите из списка соответствующую роль для пользователя. Затем в нижней части экрана нажмите кнопку **Выбрать**.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-a-signagelive-test-user"></a>Создание тестового пользователя в Signagelive

В этом разделе описано, как создать пользователя Britta Simon в приложении Signagelive. Чтобы добавить пользователей на платформу Signagelive, обратитесь в [службу поддержки Signagelive](mailto:support@signagelive.com). Перед использованием единого входа необходимо создать и активировать пользователей.

### <a name="test-single-sign-on"></a>Проверка единого входа 

В этом разделе объясняется, как проверить конфигурацию единого входа Azure AD с помощью портала MyApps.

Щелкнув плитку **Signagelive** на портале MyApps, вы автоматически войдете в Signagelive. Дополнительные сведения см. в статье о [портале MyApps](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)