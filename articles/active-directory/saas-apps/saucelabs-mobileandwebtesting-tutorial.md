---
title: Руководство по интеграции Azure Active Directory с Sauce Labs - Mobile and Web Testing | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Sauce Labs - Mobile and Web Testing.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/22/2019
ms.author: jeedes
ms.openlocfilehash: 087e2d7f2db8f27378b54675095e97256d6aae9b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92895132"
---
# <a name="tutorial-azure-active-directory-integration-with-sauce-labs---mobile-and-web-testing"></a>Руководство по интеграции Azure Active Directory с Sauce Labs - Mobile and Web Testing

В этом руководстве описано, как интегрировать Sauce Labs - Mobile and Web Testing с Azure Active Directory (Azure AD).
Интеграция Sauce Labs - Mobile and Web Testing с Azure AD обеспечивает следующие преимущества.

* С помощью Azure AD вы можете контролировать доступ к Sauce Labs - Mobile and Web Testing.
* Вы можете настроить автоматический вход пользователей в Sauce Labs - Mobile and Web Testing (единый вход) с помощью учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD с Sauce Labs - Mobile and Web Testing, вам потребуются:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/));
* подписка Sauce Labs - Mobile and Web Testing с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Sauce Labs - Mobile and Web Testing поддерживает единый вход, инициируемый **поставщиком удостоверений**.
* Sauce Labs - Mobile and Web Testing поддерживает **JIT**-подготовку пользователей.

## <a name="adding-sauce-labs---mobile-and-web-testing-from-the-gallery"></a>Добавление Sauce Labs - Mobile and Web Testing из коллекции

Чтобы настроить интеграцию Sauce Labs - Mobile and Web Testing с Azure AD, необходимо добавить Sauce Labs - Mobile and Web Testing из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Sauce Labs - Mobile and Web Testing из коллекции, выполните следующие действия.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Sauce Labs - Mobile and Web Testing**, выберите **Sauce Labs - Mobile and Web Testing** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Sauce Labs - Mobile and Web Testing в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описывается настройка и проверка единого входа Azure AD в Sauce Labs - Mobile and Web Testing с использованием тестового пользователя **Britta Simon**.
Чтобы единый вход работал, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Sauce Labs - Mobile and Web Testing.

Чтобы настроить и проверить единый вход Azure AD в Sauce Labs - Mobile and Web Testing, потребуется выполнить действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Sauce Labs - Mobile and Web Testing](#configure-sauce-labs---mobile-and-web-testing-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя Sauce Labs - Mobile and Web Testing](#create-sauce-labs---mobile-and-web-testing-test-user)** требуется для того, чтобы в Sauce Labs - Mobile and Web Testing существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Sauce Labs - Mobile and Web Testing, сделайте следующее.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Sauce Labs - Mobile and Web Testing** щелкните **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** не нужно выполнять никаких действий, так как приложение уже предварительно интегрировано с Azure.

    ![Сведения о домене и URL-адресах единого входа для Sauce Labs - Mobile and Web Testing](common/preintegrated.png)

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

6. Требуемые URL-адреса вы можете скопировать из раздела **Настройка Sauce Labs - Mobile and Web Testing**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

### <a name="configure-sauce-labs---mobile-and-web-testing-single-sign-on"></a>Настройка единого входа в Sauce Labs - Mobile and Web Testing

1. В другом окне веб-браузера войдите на сайт Sauce Labs - Mobile and Web Testing своей организации в качестве администратора.

2. Щелкните значок **User** (Пользователь) и выберите вкладку **Team Management** (Управление командой).

    ![Снимок экрана, на котором показан значок пользователя и выбран раскрывающийся список управления командой.](./media/saucelabs-mobileandwebtesting-tutorial/configure1.png)

3. Введите доменное имя в текстовом поле **Domain name** (Доменное имя).

    ![Снимок экрана, на котором в текстовом поле показан пример доменного имени.](./media/saucelabs-mobileandwebtesting-tutorial/configure2.png)

4. Выберите вкладку **Configure** (Настройка).

    ![Снимок экрана: в разделе включенного единого входа выделена вкладка для Configure (Настройка).](./media/saucelabs-mobileandwebtesting-tutorial/configure3.png)

5. В разделе **Configure Single Sign On** (Настройка единого входа) сделайте следующее.

    ![Настройка единого входа](./media/saucelabs-mobileandwebtesting-tutorial/configure4.png)

    а. Нажмите кнопку **Browse** (Обзор) и передайте файл метаданных, скачанный из Azure AD.

    b. Установите флажок **ALLOW JUST-IN-TIME PROVISIONING** (Разрешить JIT-подготовку).

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

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к Sauce Labs - Mobile and Web Testing.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **Sauce Labs - Mobile and Web Testing**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Sauce Labs - Mobile and Web Testing**.

    ![Ссылка на Sauce Labs - Mobile and Web Testing в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-sauce-labs---mobile-and-web-testing-test-user"></a>Создание тестового пользователя Sauce Labs - Mobile and Web Testing

В этом разделе вы создадите в Sauce Labs - Mobile and Web Testing пользователя с именем Britta Simon. Sauce Labs - Mobile and Web Testing поддерживает JIT-подготовку пользователей. Эта функция включена по умолчанию. В этом разделе никакие действия с вашей стороны не требуются. Если пользователь еще не существует в Sauce Labs - Mobile and Web Testing, он создается после выполнения аутентификации.

> [!Note]
> Чтобы создать пользователя вручную, обратитесь к [группе поддержки Sauce Labs - Mobile and Web Testing](mailto:support@saucelabs.com).

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув элемент "Sauce Labs - Mobile and Web Testing" на Панели доступа, вы автоматически войдете в приложение Sauce Labs - Mobile and Web Testing, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)