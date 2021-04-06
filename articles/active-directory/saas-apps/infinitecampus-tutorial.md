---
title: 'Руководство: интеграция Azure Active Directory с Infinite Campus | Документация Майкрософт'
description: Узнайте, как настроить единый вход между Azure Active Directory и Infinite Campus.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/28/2019
ms.author: jeedes
ms.openlocfilehash: 08cee87d94b20e99bcc46d42cbac7eb4272561bb
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92460246"
---
# <a name="tutorial-azure-active-directory-integration-with-infinite-campus"></a>Руководство: интеграция Azure Active Directory с Infinite Campus

В этом руководстве вы узнаете, как интегрировать Infinite Campus c Azure Active Directory (Azure AD).
Интеграция Infinite Campus с Azure AD обеспечивает следующие преимущества:

* Вы можете контролировать в Azure AD, кто имеет доступ к Infinite Campus.
* Вы можете включить автоматический вход пользователей в Infinite Campus (единый вход) с помощью учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с Infinite Campus, вам потребуется следующее:

* Подписка Azure AD. (если у вас нет среды Azure AD, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/));
* подписка Infinite Campus с поддержкой единого входа.
* По меньшей мере для настройки требуются права администратора Azure Active Directory и роль безопасности продуктов Campus "Student Information System (SIS)" (Информационная система для учащихся).

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Infinite Campus поддерживает единый вход, инициируемый **поставщиком услуг**.

## <a name="adding-infinite-campus-from-the-gallery"></a>Добавление Infinite Campus из коллекции

Чтобы настроить интеграцию Infinite Campus с Azure AD, необходимо добавить Infinite Campus из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Infinite Campus из коллекции, выполните следующие действия.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Infinite Campus**, выберите **Infinite Campus** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Infinite Campus в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в Infinite Campus с использованием тестового пользователя **Britta Simon**.
Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Infinite Campus.

Чтобы настроить и проверить единый вход Azure AD с помощью Infinite Campus, вам потребуется выполнить следующие стандартные блоки:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Infinite Campus](#configure-infinite-campus-single-sign-on)** необходима, чтобы настроить единый вход на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя Infinite Campus](#create-infinite-campus-test-user)** требуется для того, чтобы в Infinite Campus существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Infinite Campus, выполните следующие действия.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции приложения **Infinite Campus** щелкните **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе "Базовая конфигурация SAML" выполните следующие действия (учтите, что домен зависит от модели размещения, но значение **FULLY-QUALIFIED-DOMAIN** должно соответствовать установленному экземпляру Infinite Campus).

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<DOMAIN>.infinitecampus.com/campus/SSO/<DISTRICTNAME>/SIS`

    b. В текстовом поле **Идентификатор** введите URL-адрес в следующем формате: `https://<DOMAIN>.infinitecampus.com/campus/<DISTRICTNAME>`

    c. В текстовом поле **URL-адрес ответа** введите URL-адрес в следующем формате: `https://<DOMAIN>.infinitecampus.com/campus/SSO/<DISTRICTNAME>`.

    ![Сведения о домене и URL-адресах единого входа Infinite Campus](common/sp-identifier-reply.png)

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** нажмите кнопку "Копировать", чтобы копировать **URL-адрес метаданных федерации приложений** и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/copy-metadataurl.png)

### <a name="configure-infinite-campus-single-sign-on"></a>Настройка единого входа в Infinite Campus

1. В другом окне веб-браузера войдите в Infinite Campus как администратор безопасности.

2. В левой части меню щелкните **System Administration** (Системное администрирование).

    ![Администратор](./media/infinitecampus-tutorial/tutorial_infinitecampus_admin.png)

3. Выберите **Безопасность пользователя** > **SAML Management** (Управление SAML) > **SSO Service Provider Configuration** (Настройки поставщика службы единого входа).

    ![SAML](./media/infinitecampus-tutorial/tutorial_infinitecampus_saml.png)

4. На странице **SSO Service Provider Configuration** (Настройки поставщика службы единого входа) выполните следующие действия:

    ![Единый вход](./media/infinitecampus-tutorial/tutorial_infinitecampus_sso.png)

    a. Установите флажок **Enable SAML Single Sign On** (Разрешить единый вход SAML).

    b. Измените **Optional Attribute Name** (Необязательное имя атрибута), обязательно указав значение **name**.

    c. В разделе **Select an option to retrieve Identity Provider (IDP) server data** (Выберите параметр, чтобы получить данные сервера поставщика удостоверений (IDP)) выберите **Metadata URL** (URL-адрес метаданных), вставьте **URL-адрес метаданных федерации приложения** в поле и щелкните **Sync** (Синхронизировать).

    d. После нажатия элемента **Синхронизация** значения автоматически заполнятся на странице **SSO Service Provider Configuration** (Настройка поставщика услуг единого входа). Эти значения можно проверить на соответствие значениям, использованным ранее на шаге 4.

    д) Выберите команду **Сохранить**.

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD

Цель этого раздела — создать на портале Azure тестового пользователя с именем Britta Simon.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.

    ![Ссылки "Пользователи и группы" и "Все пользователи"](common/users.png)

2. В верхней части экрана выберите **Новый пользователь**.

    ![Кнопка "Новый пользователь"](common/new-user.png)

3. В разделе свойств пользователя сделайте следующее:

    ![Диалоговое окно "Пользователь"](common/user-properties.png)

    а. В поле **Имя** введите **BrittaSimon**.
  
    b. В поле **Имя пользователя** введите `brittasimon@yourcompanydomain.extension`. Например, BrittaSimon@contoso.com.

    c. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле "Пароль".

    d. Нажмите кнопку **Создать**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

> [!NOTE]
> Если вы хотите, чтобы все пользователи Azure могли выполнять единый вход в Infinite Campus и использовать внутреннюю систему разрешений Infinite Campus для управления доступом, можно задать для свойства приложения **User Assignment Required** (Требуется назначение пользователей) значение "No" (Нет) и пропустить следующие шаги.

В этом разделе описано, как предоставить пользователю Britta Simon доступ к Infinite Campus, чтобы он мог использовать единый вход Azure.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **Infinite Campus**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Infinite Campus**.

    ![Ссылка на Infinite Campus в списке "Приложения"](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-infinite-campus-test-user"></a>Создание тестового пользователя Infinite Campus

Infinite Campus имеет архитектуру, ориентированную на демографические данные. Обратитесь в [службу технической поддержки Infinite Campus](mailto:sales@infinitecampus.com), чтобы добавить пользователей на платформу Infinite Campus.

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув элемент "Infinite Campus" на Панели доступа, вы автоматически войдете в приложение Infinite Campus, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)