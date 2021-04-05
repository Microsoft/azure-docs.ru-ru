---
title: Руководство по интеграции Azure Active Directory с Cerner Central | Документы Майкрософт
description: Сведения о настройке единого входа между Azure Active Directory и Cerner Central.
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
ms.openlocfilehash: 8979d71dd1ebb23a42c67de905df77c6aeef8706
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92456377"
---
# <a name="tutorial-azure-active-directory-integration-with-cerner-central"></a>Руководство. Интеграция Azure Active Directory с Cerner Central

В этом руководстве описано, как интегрировать Cerner Central с Azure Active Directory (Azure AD).
Интеграция Azure AD с приложением Cerner Central обеспечивает следующие преимущества:

* С помощью Azure AD вы можете контролировать доступ к Cerner Central.
* Вы можете включить автоматический вход пользователей в Cerner Central (единый вход) с помощью учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с Cerner Central, вам потребуется:

* Подписка Azure AD. (если у вас нет среды Azure AD, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/));
* подписка Cerner Central с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Cerner Central поддерживает единый вход, инициированный **IDP**.
* Cerner Central поддерживает [**автоматическую** подготовку пользователей](cernercentral-provisioning-tutorial.md).

## <a name="adding-cerner-central-from-the-gallery"></a>Добавление Cerner Central из коллекции

Чтобы настроить интеграцию Cerner Central с Azure AD, необходимо добавить Cerner Central из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Cerner Central из коллекции, сделайте следующее.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Cerner Central**, выберите **Cerner Central** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Cerner Central в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описаны настройка и проверка единого входа Azure AD в Cerner Central с использованием тестового пользователя **Britta Simon**.
Чтобы обеспечить работу единого входа, нужно установить связь между пользователем Azure AD и соответствующим пользователем в Cerner Central.

Чтобы настроить и проверить единый вход Azure AD в Cerner Central, вам потребуется выполнить действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Cerner Central](#configure-cerner-central-single-sign-on)** требуется для того, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя Cerner Central](#create-cerner-central-test-user)** требуется для того, чтобы в Cerner Central существовал пользователь Britta Simon, связанный с представлением этого же пользователя в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Cerner Central, сделайте следующее.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Cerner Central** выберите **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. На странице **Настройка единого входа с помощью SAML** выполните следующие действия.

    ![Сведения о домене и URL-адресах единого входа для приложения Cerner Central](common/idp-intiated.png)

    а. В текстовом поле **Идентификатор** введите URL-адрес в таком формате:

    ```http
    https://<instancename>.cernercentral.com/session-api/protocol/saml2/metadata
    https://<instancename>.sandboxcernercentral.com/session-api/protocol/saml2/metadata
    ```

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в таком формате:

    ```http
    https://<instancename>.cernercentral.com/session-api/protocol/saml2/sso
    https://<instancename>.sandboxcernercentral.com/session-api/protocol/saml2/sso
    ```

    > [!NOTE]
    > Эти значения приведены для примера. Измените их на фактические значения идентификатора и URL-адреса ответа. Чтобы получить эти значения, обратитесь в [группу поддержки клиентов Cerner Central](mailto:SISupport@cbre.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** нажмите кнопку "Копировать", чтобы копировать **URL-адрес метаданных федерации приложений** и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/copy-metadataurl.png)

### <a name="configure-cerner-central-single-sign-on"></a>Настройка единого входа в Cerner Central

Для настройки единого входа на стороне **Cerner Central** необходимо отправить **URL-адрес метаданных федерации приложения** в [группу поддержки Cerner Central](mailto:SISupport@cbre.com). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

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

В этом разделе описано, как позволить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к Cerner Central.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем **Cerner Central**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Cerner Central**.

    ![Ссылка на Cerner Central в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-cerner-central-test-user"></a>Создание тестового пользователя Cerner Central

Приложение **Cerner Central** разрешает аутентификацию посредством любого поставщика федеративных удостоверений. Если пользователь может войти на домашнюю страницу приложения, то он является федеративным и не требует подготовки вручную. Дополнительные сведения о настройке автоматической подготовки пользователей можно найти [здесь](cernercentral-provisioning-tutorial.md).

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку Cerner Central на Панели доступа, вы автоматически войдете в приложение Cerner Central, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)

- [Руководство по настройке Google Apps для автоматической подготовки пользователей](cernercentral-provisioning-tutorial.md)