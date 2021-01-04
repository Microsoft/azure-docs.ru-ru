---
title: Руководство. Интеграция Azure Active Directory с Brightspace (разработка Desire2Learn) | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Brightspace by Desire2Learn.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/08/2019
ms.author: jeedes
ms.openlocfilehash: f999818ab791cabac6b0877b7735fa730dab89e2
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/18/2020
ms.locfileid: "97673407"
---
# <a name="tutorial-azure-active-directory-integration-with-brightspace-by-desire2learn"></a>Руководство. Интеграция Azure Active Directory с Brightspace (разработка Desire2Learn)

В этом руководстве описано, как интегрировать Azure Active Directory (Azure AD) с приложением Brightspace by Desire2Learn.
Интеграция Brightspace by Desire2Learn с Azure AD дает следующие преимущества:

* С помощью Azure AD вы можете контролировать доступ к Brightspace by Desire2Learn.
* Вы можете включить автоматический вход пользователей в Brightspace by Desire2Learn (единый вход) с учетной записью Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD с Brightspace by Desire2Learn, вам потребуется:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* Подписка с поддержкой единого входа Brightspace (разработка Desire2Learn)

> [!NOTE]
> Эту интеграцию также можно использовать в облачной среде Azure AD для государственных организаций США. Это приложение можно найти в коллекции облачных приложений с поддержкой Azure AD для государственных организаций США и настроить таким же образом, как и в общедоступном облаке.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Brightspace (разработка Desire2Learn) поддерживает единый вход, инициированный **поставщиком удостоверений**

## <a name="adding-brightspace-by-desire2learn-from-the-gallery"></a>Добавление Brightspace by Desire2Learn из коллекции

Чтобы настроить интеграцию Brightspace by Desire2Learn с Azure AD, необходимо добавить Brightspace by Desire2Learn из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Brightspace by Desire2Learn из коллекции, сделайте следующее.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Brightspace (разработка Desire2Learn)** , выберите **Brightspace (разработка Desire2Learn)** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Brightspace (разработка Desire2Learn) в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в Brightspace (разработка Desire2Learn) с использованием тестового пользователя **Britta Simon**.
Чтобы единый вход работал, нужно установить связь между пользователем Azure AD и соответствующим пользователем в Brightspace by Desire2Learn.

Чтобы настроить и проверить единый вход Azure AD в Brightspace by Desire2Learn, вам потребуется выполнить действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Brightspace (разработка Desire2Learn)](#configure-brightspace-by-desire2learn-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя Brightspace by Desire2Learn](#create-brightspace-by-desire2learn-test-user)** нужно для того, чтобы в Brightspace by Desire2Learn также существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Brightspace (разработка Desire2Learn), выполните следующие действия.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Brightspace (разработка Desire2Learn)** щелкните **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. На странице **Настройка единого входа с помощью SAML** выполните следующие действия.

    ![Сведения о домене и URL-адресах единого входа приложения Brightspace (разработка Desire2Learn)](common/idp-intiated.png)

    а. В текстовом поле **Идентификатор** введите URL-адрес в таком формате:

    ```http
    https://<companyname>.tenants.brightspace.com/samlLogin
    https://<companyname>.desire2learn.com/shibboleth-sp
    ```

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<companyname>.desire2learn.com/d2l/lp/auth/login/samlLogin.d2l`.

    > [!NOTE]
    > Эти значения приведены для примера. Измените их на фактические значения идентификатора и URL-адреса ответа. Чтобы получить эти значения, обратитесь к [группе поддержки Brightspace (разработка Desire2Learn)](https://www.d2l.com/contact/). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

6. Скопируйте требуемый URL-адрес из раздела **Настройка Brightspace (разработка Desire2Learn)** .

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD.

    c. URL-адрес выхода.

### <a name="configure-brightspace-by-desire2learn-single-sign-on"></a>Настройка единого входа Brightspace (разработка Desire2Learn)

Чтобы настроить единый вход на стороне **Brightspace (разработка Desire2Learn)** , нужно отправить скачанный файл **XML метаданных федерации** и соответствующие URL-адреса, скопированные на портале Azure, [группе поддержки Brightspace (разработка Desire2Learn)](https://www.d2l.com/contact/). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

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

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к Brightspace by Desire2Learn.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **Brightspace (разработка Desire2Learn)** .

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Brightspace by Desire2Learn**.

    ![Из списка приложений выберите Brightspace (разработка Desire2Learn).](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-brightspace-by-desire2learn-test-user"></a>Создание тестового пользователя Brightspace (разработка Desire2Learn)

В этом разделе описано, как создать пользователя Britta Simon в приложении Brightspace (разработка Desire2Learn). Совместно со [службой технической поддержки Brightspace (разработка Desire2Learn)](https://www.d2l.com/contact/) добавьте пользователей на платформу Brightspace (разработка Desire2Learn). Перед использованием единого входа необходимо создать и активировать пользователей.

> [!NOTE]
> Вы можете использовать любые другие средства создания учетной записи пользователя Brightspace (разработка Desire2Learn) или API, предоставляемые Brightspace (разработка Desire2Learn) для подготовки учетных записей пользователя Azure Active Directory.

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув элемент Brightspace (разработка Desire2Learn) на панели доступа, вы автоматически войдете в приложение Brightspace (разработка Desire2Learn), для которого вы настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)