---
title: Руководство по интеграции Azure Active Directory с SmartRecruiters | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в приложении SmartRecruiters.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/07/2019
ms.author: jeedes
ms.openlocfilehash: b7de874cc074faa63203c58638fb04cc5cb2282b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92514934"
---
# <a name="tutorial-azure-active-directory-integration-with-smartrecruiters"></a>Руководство по интеграции Azure Active Directory с SmartRecruiters

Из этого руководства вы узнаете, как интегрировать SmartRecruiters с Azure Active Directory (Azure AD).
Интеграция Azure AD с приложением SmartRecruiters обеспечивает следующие преимущества:

* C помощью Azure AD вы можете контролировать доступ к SmartRecruiters.
* Вы можете включить автоматический вход пользователей в SmartRecruiters (единый вход) с использованием учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD с SmartRecruiters, вам потребуется следующее:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка SmartRecruiters с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* SmartRecruiters поддерживает единый вход, инициированный **поставщиком услуг и поставщиком удостоверений**.

## <a name="adding-smartrecruiters-from-the-gallery"></a>Добавление SmartRecruiters из коллекции

Чтобы настроить интеграцию SmartRecruiters с Azure AD, необходимо добавить SmartRecruiters из коллекции в список управляемых приложений SaaS.

**Чтобы добавить SmartRecruiters из коллекции, выполните инструкции ниже.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **SmartRecruiters**, выберите **SmartRecruiters** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

     ![SmartRecruiters в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в SmartRecruiters при помощи тестового пользователя **Britta Simon**.
Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в SmartRecruiters.

Чтобы настроить и проверить единый вход Azure AD в SmartRecruiters, вам потребуется выполнить действия в указанных ниже стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в SmartRecruiters](#configure-smartrecruiters-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создав тестового пользователя SmartRecruiters](#create-smartrecruiters-test-user)** Britta Simon, мы сможем связать его с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в SmartRecruiters, выполните инструкции ниже.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **SmartRecruiters** выберите **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** выполните следующие действия.

    ![Снимок экрана, на котором показан раздел "Базовая конфигурация SAML", где можно ввести идентификатор, URL-адрес ответа и нажать кнопку "Сохранить"](common/idp-intiated.png)

    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `https://www.smartrecruiters.com/web-sso/saml/<companyname>`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://www.smartrecruiters.com/web-sso/saml/<companyname>/callback`.

5. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    ![Снимок экрана, на котором показан параметр "Задать дополнительные URL-адреса" и поле для ввода URL-адреса входа](common/metadata-upload-additional-signon.png)

    В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://www.smartrecruiters.com/web-sso/saml/<companyname>/login`.

    > [!NOTE]
    > Эти значения приведены для примера. Замените их фактическими значениями идентификатора, URL-адреса ответа и URL-адреса входа. Чтобы получить эти значения, обратитесь в [службу поддержки клиентов SmartRecruiters](https://www.smartrecruiters.com/about-us/contact-us/). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

6. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (Base64)** из предложенных вариантов, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

7. Скопируйте требуемый URL-адрес из раздела **Настройка SmartRecruiters**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

### <a name="configure-smartrecruiters-single-sign-on"></a>Настройка единого входа в SmartRecruiters

1. В другом окне веб-браузера войдите на корпоративный сайт SmartRecruiters в качестве администратора.

1. Последовательно выберите элементы **Параметры / Администратор**.

    ![Снимок экрана: в меню выбран пункт Settings / Admin (Параметры / Администратор).](./media/smartrecruiters-tutorial/configure.png)

1. В разделе **Конфигурация** щелкните **Web SSO** (Единый вход через Интернет).

    ![Снимок экрана: раздел Configuration (Конфигурация) с выбранным пунктом Web SSO (Единый вход через Интернет).](./media/smartrecruiters-tutorial/configure1.png)

1. Переключите **Enable Web SSO** (Включить единый вход через Интернет).

    ![Снимок экрана: элемент управления Enable Web SSO (Включить единый вход через Интернет).](./media/smartrecruiters-tutorial/configure2.png)

1. В разделе **Identity Provider Configuration** (Конфигурация поставщика удостоверений) выполните следующие действия:

    ![Снимок экрана: страница конфигурации поставщика удостоверений, где можно ввести описанные здесь значения.](./media/smartrecruiters-tutorial/configure4.png)

    а. В текстовое поле **Identity Provider URL** (URL-адрес поставщика удостоверений) вставьте **URL-адрес для входа**, скопированный на портале Azure.

    b. Откройте **сертификат (Base64)** , который вы скачали с портала Azure, в Notepad, скопируйте его содержимое и вставьте в текстовое поле **Identity Provider certificate** (Сертификат поставщика удостоверений).

1. Щелкните **Save Web SSO configuration** (Сохранить конфигурацию единого входа через Интернет).

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD 

Цель этого раздела — создать на портале Azure тестового пользователя с именем Britta Simon.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.

    ![Ссылки "Пользователи и группы" и "Все пользователи"](common/users.png)

2. В верхней части экрана выберите **Новый пользователь**.

    ![Кнопка "Новый пользователь"](common/new-user.png)

3. В разделе свойств пользователя сделайте следующее:

    ![Диалоговое окно "Пользователь"](common/user-properties.png)

    а. В поле **Имя** введите **BrittaSimon**.
  
    b. В поле **Имя пользователя** введите **brittasimon@yourcompanydomain.extension** .  
    Например BrittaSimon@contoso.com.

    c. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле "Пароль".

    d. Нажмите кнопку **Создать**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе пользователю Britta Simon предоставляется разрешение на использование единого входа Azure для доступа к SmartRecruiters.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **SmartRecruiters**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **SmartRecruiters**.

    ![Ссылка на SmartRecruiters в списке "Приложения"](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-smartrecruiters-test-user"></a>Создание тестового пользователя SmartRecruiters

Из этого раздела вы узнаете, как создать пользователя Britta Simon в SmartRecruiters. Для добавления пользователей на платформе SmartRecruiters обратитесь в [службу поддержки SmartRecruiters](https://www.smartrecruiters.com/about-us/contact-us/). Перед использованием единого входа необходимо создать и активировать пользователей.

### <a name="test-single-sign-on"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку SmartRecruiters на Панели доступа, вы автоматически войдете в приложение SmartRecruiters, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)