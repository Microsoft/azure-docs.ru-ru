---
title: Руководство по интеграции Azure Active Directory с SAML SSO for Bitbucket by resolution GmbH | Документация Майкрософт
description: Узнайте, как настроить единый вход для Azure Active Directory и SAML SSO for Bitbucket by resolution GmbH.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/27/2018
ms.author: jeedes
ms.openlocfilehash: 5e0b1d8d2cb80886131a88c01a6c24102ad7c77c
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/18/2020
ms.locfileid: "97673566"
---
# <a name="tutorial-azure-active-directory-integration-with-saml-sso-for-bitbucket-by-resolution-gmbh"></a>Руководство по интеграции Azure Active Directory с SAML SSO for Bitbucket by resolution GmbH

В этом руководстве описано, как интегрировать SAML SSO for Bitbucket by resolution GmbH с Azure Active Directory (Azure AD).
Интеграция SAML SSO for Bitbucket by resolution GmbH с Azure AD предоставляет следующие преимущества:

* Можно управлять доступом пользователей Azure AD к SAML SSO for Bitbucket by resolution GmbH.
* Вы можете включить автоматический вход пользователей в SAML SSO for Bitbucket by resolution GmbH (единый вход) с помощью учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Для настройки интеграции Azure AD с SAML SSO for Bitbucket by resolution GmbH требуется:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка SAML SSO for Bitbucket by resolution GmbH с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* SAML SSO for Bitbucket by resolution GmbH поддерживает единый вход, инициированный **поставщиком услуг и поставщиком удостоверений**.
* SAML SSO for Bitbucket by resolution GmbH поддерживает **JIT**-подготовку пользователей.


## <a name="adding-saml-sso-for-bitbucket-by-resolution-gmbh-from-the-gallery"></a>Добавление SAML SSO for Bitbucket by resolution GmbH из коллекции

Чтобы настроить интеграцию SAML SSO for Bitbucket by resolution GmbH в Azure AD, необходимо добавить SAML SSO for Bitbucket by resolution GmbH из коллекции в список управляемых приложений SaaS.

**Чтобы добавить SAML SSO for Bitbucket by resolution GmbH из коллекции, сделайте следующее:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. Чтобы добавить приложение, в поле поиска введите **SAML SSO for Bitbucket by resolution GmbH**, затем на панели результатов выберите **SAML SSO for Bitbucket by resolution GmbH** и нажмите кнопку **Добавить**.

    ![Список результатов SAML SSO for Bitbucket by resolution GmbH](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в SAML SSO for Bitbucket by resolution GmbH с использованием тестового пользователя **Britta Simon**.
Чтобы единый вход работал, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в SAML SSO for Bitbucket by resolution GmbH.

Чтобы настроить и проверить единый вход Azure AD в SAML SSO for Bitbucket by resolution GmbH, требуется выполнить действия в следующих стандартных блоках:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в SAML SSO for Bitbucket by resolution GmbH](#configure-saml-sso-for-bitbucket-by-resolution-gmbh-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя SAML SSO for Bitbucket by resolution GmbH](#create-saml-sso-for-bitbucket-by-resolution-gmbh-test-user)** требуется, чтобы создать в SAML SSO for Bitbucket by resolution GmbH пользователя Britta Simon, связанного с соответствующим пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в SAML SSO for Bitbucket by resolution GmbH, сделайте следующее:

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **SAML SSO for Bitbucket by resolution GmbH** выберите **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Правка базовой конфигурации SAML](common/edit-urls.png)

4. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** выполните следующие действия.

    ![Снимок экрана, на котором показан раздел "Базовая конфигурация SAML", где можно ввести идентификатор, URL-адрес ответа и нажать кнопку "Сохранить"](common/idp-intiated.png)

    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `https://<server-base-url>/plugins/servlet/samlsso`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<server-base-url>/plugins/servlet/samlsso`.

    c. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    ![Снимок экрана, на котором показан параметр "Задать дополнительные URL-адреса" и поле для ввода URL-адреса входа](common/metadata-upload-additional-signon.png)

    В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://<server-base-url>/plugins/servlet/samlsso`.

    > [!NOTE]
    > Эти значения приведены для примера. Замените их фактическими значениями идентификатора, URL-адреса ответа и URL-адреса входа. Для получения этих значений обратитесь к [группе поддержки SAML SSO for Bitbucket by resolution GmbH](https://marketplace.atlassian.com/apps/1217045/saml-single-sign-on-sso-bitbucket?hosting=server&tab=support). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

### <a name="configure-saml-sso-for-bitbucket-by-resolution-gmbh-single-sign-on"></a>Настройка единого входа в SAML SSO for Bitbucket by resolution GmbH

1. Войдите на свой корпоративный сайт SAML SSO for Bitbucket by resolution GmbH с правами администратора.

2. Справа от главной панели инструментов щелкните **Параметры**.

3. В разделе "Учетные записи" в строке меню щелкните **SAML SingleSignOn**.

    ![Samlsingle](./media/bitbucket-tutorial/tutorial_bitbucket_samlsingle.png)

4. На **странице конфигурации подключаемого модуля SAML SIngleSignOn** щелкните **Add idp** (Добавить поставщик удостоверений). 

    ![Добавление поставщика удостоверений](./media/bitbucket-tutorial/tutorial_bitbucket_addidp.png)

5. На странице **Choose your SAML Identity Provider** (Выбор поставщика удостоверений SAML) выполните следующие действия:

    ![Поставщик удостоверений](./media/bitbucket-tutorial/tutorial_bitbucket_identityprovider.png)

    а. Выберите **Azure AD** в качестве **типа поставщика удостоверений**.

    b. В текстовом поле **Имя** введите имя.

    c. В текстовом поле **Описание** введите описание.

    d. Щелкните **Далее**.

6. На странице **Identity provider configuration page** (Настройка поставщика удостоверений) нажмите кнопку **Далее**.

    ![Настройка удостоверения](./media/bitbucket-tutorial/tutorial_bitbucket_identityconfig.png)

7.  На странице **импорта метаданных SAML поставщика удостоверений** щелкните **Загрузить файл**, чтобы передать файл **метаданных в формате XML**, скачанный на портале Azure.

    ![idpmetadata](./media/bitbucket-tutorial/tutorial_bitbucket_idpmetadata.png)

8. Щелкните **Далее**.

9. Нажмите кнопку **Сохранить параметры**.

    ![Сохранение](./media/bitbucket-tutorial/tutorial_bitbucket_save.png)

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

В этом разделе описано, как позволить пользователю Britta Simon использовать единый вход Azure путем предоставления ей доступа к SAML SSO for Bitbucket by resolution GmbH.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **SAML SSO for Bitbucket by resolution GmbH**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений введите и выберите **SAML SSO for Bitbucket by resolution GmbH**.

    ![Ссылка на SAML SSO for Bitbucket by resolution GmbH link в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-saml-sso-for-bitbucket-by-resolution-gmbh-test-user"></a>Создание тестового пользователя в SAML SSO for Bitbucket by resolution GmbH

Цель этого раздела — создать в приложении SAML SSO for Bitbucket by resolution GmbH пользователя с именем Britta Simon. SAML SSO for Bitbucket by resolution GmbH обеспечивает своевременную подготовку, а также создание пользователей вручную. Для согласования ваших требований обратитесь в [службу поддержки клиентов SAML SSO for Bitbucket by resolution GmbH](https://marketplace.atlassian.com/plugins/com.resolution.atlasplugins.samlsso-bitbucket/server/support).

### <a name="test-single-sign-on"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув элемент SAML SSO for Bitbucket by resolution GmbH на панели доступа, вы должны автоматически войти в приложение SAML SSO for Jira by Bitbucket GmbH, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)