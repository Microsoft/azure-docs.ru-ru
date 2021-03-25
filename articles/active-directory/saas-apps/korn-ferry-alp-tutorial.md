---
title: Руководство по Интеграция Azure Active Directory с приложением Korn Ferry ALP | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Korn Ferry ALP.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/21/2019
ms.author: jeedes
ms.openlocfilehash: 4ed46a7bb7fd8600bf35d102259be257adfacb3f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92458903"
---
# <a name="tutorial-azure-active-directory-integration-with-korn-ferry-alp"></a>Руководство по Интеграция Azure Active Directory с приложением Korn Ferry ALP

В этом руководстве описано, как интегрировать Korn Ferry ALP с Azure Active Directory (Azure AD).
Интеграция Korn Ferry ALP с Azure AD обеспечивает следующие преимущества.

* C помощью Azure AD вы можете контролировать доступ к Korn Ferry ALP.
* Вы можете включить автоматический вход пользователей в Korn Ferry ALP (единый вход) с помощью учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с Korn Ferry ALP, вам потребуется следующее:

* Подписка Azure AD. (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* Подписка Korn Ferry ALP с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Korn Ferry ALP поддерживает единый вход, инициированный **поставщиком услуг**.

## <a name="adding-korn-ferry-alp-from-the-gallery"></a>Добавление Korn Ferry ALP из коллекции

Чтобы настроить интеграцию Korn Ferry ALP с Azure AD, необходимо добавить Korn Ferry ALP из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Korn Ferry ALP из коллекции, выполните следующие действия.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Korn Ferry ALP**, выберите **Korn Ferry ALP** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

     ![Korn Ferry ALP в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в Korn Ferry ALP с использованием тестового пользователя **Britta Simon**.
Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Korn Ferry ALP.

Чтобы настроить и проверить единый вход Azure AD в Korn Ferry ALP, вам потребуется выполнить действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Korn Ferry ALP](#configure-korn-ferry-alp-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя в приложении Korn Ferry ALP](#create-korn-ferry-alp-test-user)** требуется для того, чтобы в Korn Ferry ALP существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Korn Ferry ALP, выполните следующие действия.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Korn Ferry ALP** щелкните **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    ![Сведения о домене и URL-адресах единого входа для приложения Korn Ferry ALP](common/sp-identifier.png)

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: .

    ```http
    https://intappextin01/portalweb/sso/client/audience?guid=<customerguid>
    https://qaassessment.kfnaqa.com/portalweb/sso/client/audience?guid=<customerguid>
    https://assessments.kornferry.com/portalweb/sso/client/audience?guid=<customerguid>
    ```

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате:
    
    ```http
    https://intappextin01/portalweb/sso/client/audience?guid=<customerguid>
    https://qaassessment.kfnaqa.com/portalweb/sso/client/audience?guid=<customerguid>
    https://assessments.kornferry.com/portalweb/sso/client/audience?guid=<customerguid>
    ```

    > [!NOTE]
    > Эти значения приведены для примера. Необходимо обновить эти значения действующим URL-адресом для входа и идентификатором. Чтобы получить эти значения, обратитесь к [группе поддержки клиентов Korn Ferry ALP](mailto:noreply@kornferry.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

4. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** нажмите кнопку "Копировать", чтобы копировать **URL-адрес метаданных федерации приложений** и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/copy-metadataurl.png)

### <a name="configure-korn-ferry-alp-single-sign-on"></a>Настройка единого входа в Korn Ferry ALP

Чтобы настроить единый вход на стороне **Korn Ferry ALP**, необходимо отправить **URL-адрес метаданных федерации приложения** в [группу поддержки Korn Ferry ALP](mailto:noreply@kornferry.com). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

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

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив доступ к Korn Ferry ALP.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **Korn Ferry ALP**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Korn Ferry ALP**.

    ![Ссылка на Korn Ferry ALP в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-korn-ferry-alp-test-user"></a>Создание тестового пользователя в приложении Korn Ferry ALP

В этом разделе описано, как создать пользователя Britta Simon в Korn Ferry ALP. Чтобы добавить пользователей на платформу Korn Ferry ALP, обратитесь в [службу технической поддержки Korn Ferry ALP](mailto:noreply@kornferry.com). Перед использованием единого входа необходимо создать и активировать пользователей.

### <a name="test-single-sign-on"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку Korn Ferry ALP на Панели доступа, вы автоматически войдете в приложение Korn Ferry ALP, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)