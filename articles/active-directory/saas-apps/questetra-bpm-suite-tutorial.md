---
title: Руководство по интеграции Azure Active Directory с Questetra BPM Suite | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Questetra BPM Suite.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: jeedes
ms.openlocfilehash: 13f05005529d3983e042398f5274fd7f981b8c8c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92516602"
---
# <a name="tutorial-azure-active-directory-integration-with-questetra-bpm-suite"></a>Учебник. Интеграция Azure Active Directory с Questetra BPM Suite

В этом руководстве описано, как интегрировать Questetra BPM Suite с Azure Active Directory (Azure AD).
Интеграция Questetra BPM Suite с Azure AD обеспечивает следующие преимущества:

* С помощью Azure AD вы можете контролировать доступ к Questetra BPM Suite.
* Вы можете включить автоматический вход пользователей в Questetra BPM Suite (единый вход) с помощью учетной записи Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD с Questetra BPM Suite, вам потребуется следующее:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка Questetra BPM Suite с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Questetra BPM Suite поддерживает единый вход, инициированный **поставщиком служб**.

## <a name="adding-questetra-bpm-suite-from-the-gallery"></a>Добавление Questetra BPM Suite из коллекции

Чтобы настроить интеграцию Questetra BPM Suite в Azure AD, необходимо добавить Questetra BPM Suite из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Questetra BPM Suite из коллекции, выполните следующие действия.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Questetra BPM Suite**, выберите **Questetra BPM Suite** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

     ![Questetra BPM Suite в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в приложение Questetra BPM Suite с использованием тестового пользователя **Britta Simon**.
Чтобы единый вход функционировал, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Questetra BPM Suite.

Чтобы настроить и проверить единый вход Azure AD в Questetra BPM Suite, вам потребуется выполнить действия в следующих стандартных блоках:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Questetra BPM Suite](#configure-questetra-bpm-suite-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя Questetra BPM Suite](#create-questetra-bpm-suite-test-user)** требуется для того, чтобы в Questetra BPM Suite существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Questetra BPM Suite, выполните следующие действия:

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Questetra BPM** Suite щелкните **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    ![Сведения о домене и URL-адресах единого входа приложения Questetra BPM Suite](common/sp-identifier.png)

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<subdomain>.questetra.net/saml/SSO/alias/bpm`.

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `https://<subdomain>.questetra.net/`.

    > [!NOTE]
    > Эти значения приведены для примера. Замените эти значения фактическим URL-адресом для входа и идентификатором. Вы можете получить эти значения в разделе **SP Information** (Сведения о SP) на своем корпоративном сайте **Questetra BPM Suite**. Как это сделать, описывается далее в этом руководстве. Вы можете также обратиться к [группе поддержки клиентов Questetra BPM Suite](https://www.questetra.com/contact/). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (Base64)** из предложенных вариантов, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

6. Требуемый URL-адрес можно скопировать из раздела **Настройка Questetra BPM Suite**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

### <a name="configure-questetra-bpm-suite-single-sign-on"></a>Настройка единого входа Questetra BPM Suite

1. В новом окне веб-браузера войдите на корпоративный веб-сайт **Questetra BPM Suite** в качестве администратора.

2. В верхнем меню щелкните пункт **Системные параметры**. 
   
    ![Снимок экрана: выбор пункта System Settings (Системные параметры) на корпоративном сайте Questetra BPM Suite.][10]

3. Чтобы открыть страницу **SingleSignOnSAML**, щелкните **SSO (SAML)** (Единый вход (SAML)). 
   
    ![Снимок экрана, на котором показан выбранный элемент SSO (SAML) (Единый вход, SAML).][11]

4. На своем корпоративном сайте **Questetra BPM Suite** в разделе **SP Information** (Сведения о SP) выполните следующие действия.

    а. Скопируйте **URL-адрес ACS** и вставьте его в текстовое поле **URL-адрес для входа** в разделе **Базовая конфигурация SAML** на портале Azure.
    
    b. Скопируйте **идентификатор сущности** и вставьте его в текстовое поле **Идентификатор** в разделе **Базовая конфигурация SAML** на портале Azure.

5. На своем корпоративном сайте **Questetra BPM Suite** выполните следующие действия. 
   
    ![Настройка единого входа][15]
   
    а. Выберите пункт **Включить единый вход**.
   
    b. В текстовое поле **Entity ID** (Идентификатор сущности) вставьте значение **Идентификатор Azure AD**, скопированное на портале Azure.
    
    c. В текстовое поле **Sign-in page URL** (URL-адрес страницы входа) вставьте **URL-адрес входа**, скопированный на портале Azure.
    
    d. В текстовое поле **Sign-out page URL** (URL-адрес страницы выхода) вставьте значение **URL-адреса выхода**, скопированное на портале Azure.
    
    д) В текстовом поле **NameID format** (Формат идентификатора имени) введите `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress`.

    е) Откройте в Блокноте сертификат в кодировке **Base64**, скачанный с портала Azure, скопируйте его содержимое в буфер обмена и вставьте в текстовое поле **Validation certificate** (Сертификат проверки). 

    ж. Выберите команду **Сохранить**.

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD 

Цель этого раздела — создать на портале Azure тестового пользователя с именем Britta Simon.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.

    ![Ссылки "Пользователи и группы" и "Все пользователи"](common/users.png)

2. В верхней части экрана выберите **Новый пользователь**.

    ![Кнопка "Новый пользователь"](common/new-user.png)

3. В разделе свойств пользователя сделайте следующее:

    ![Диалоговое окно "Пользователь"](common/user-properties.png)

    а. В поле **Имя** введите **BrittaSimon**.
  
    b. В поле **Имя пользователя** введите brittasimon@yourcompanydomain.extension. Например BrittaSimon@contoso.com.

    c. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле "Пароль".

    d. Нажмите кнопку **Создать**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе вы разрешите пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к Questetra BPM Suite.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **Questetra BPM Suite**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Questetra BPM Suite**.

    ![Ссылка на Questetra BPM Suite в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-questetra-bpm-suite-test-user"></a>Создание тестового пользователя Questetra BPM Suite

Цель этого раздела — создать пользователя с именем Britta Simon в Questetra BPM Suite.

**Чтобы создать пользователя с именем Britta Simon в Questetra BPM Suite, выполните следующие действия.**

1. Войдите на корпоративный сайт Questetra BPM Suite в качестве администратора.

2. Выберите **System Settings > User List > New User** (Системные параметры > Список пользователей > Новый пользователь).
 
3. В диалоговом окне создания нового пользователя выполните следующие действия: 
   
    ![Создание тестового пользователя][300] 
   
    а. В текстовое поле **Name** (Имя) введите **имя** пользователя britta.simon@contoso.com.
   
    b. В текстовое поле **Email** (Адрес электронной почты) введите **адрес электронной почты** пользователя britta.simon@contoso.com.
   
    c. В текстовое поле **Password** (Пароль) введите **пароль** для пользователя.
    
    d. Нажмите кнопку **Добавить нового пользователя**.

### <a name="test-single-sign-on"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку Questetra BPM Suite на Панели доступа, вы автоматически войдете в приложение Questetra BPM Suite, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)

<!--Image references-->

[10]: ./media/questetra-bpm-suite-tutorial/questera_bpm_suite_03.png
[11]: ./media/questetra-bpm-suite-tutorial/questera_bpm_suite_04.png
[15]: ./media/questetra-bpm-suite-tutorial/questera_bpm_suite_08.png
[300]: ./media/questetra-bpm-suite-tutorial/questera_bpm_suite_11.png