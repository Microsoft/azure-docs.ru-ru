---
title: Руководство по интеграции Azure Active Directory с Zscaler Private Access Administrator | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Zscaler Private Access Administrator.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/03/2019
ms.author: jeedes
ms.openlocfilehash: 803f5c25429b33e6d1af130196d38e125b410eb4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92891114"
---
# <a name="tutorial-azure-active-directory-integration-with-zscaler-private-access-administrator"></a>Руководство по интеграции Azure Active Directory с Zscaler Private Access Administrator

В этом руководстве описано, как интегрировать Zscaler Private Access Administrator с Azure Active Directory (Azure AD).
Интеграция Azure AD с Zscaler Private Access Administrator обеспечивает следующие преимущества.

* С помощью Azure AD вы можете контролировать доступ к Zscaler Private Access Administrator.
* Вы можете включить автоматический вход пользователей в Zscaler Private Access Administrator (единый вход) с помощью учетной записи Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD с Zscaler Private Access Administrator, вам потребуется:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/));
* подписка Zscaler Private Access Administrator с поддержкой единого входа.

> [!NOTE]
> Эту интеграцию также можно использовать в облачной среде Azure AD для государственных организаций США. Это приложение можно найти в коллекции облачных приложений с поддержкой Azure AD для государственных организаций США и настроить таким же образом, как и в общедоступном облаке.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Zscaler Private Access Administrator поддерживает единый вход, инициированный **поставщиком службы** и **поставщиком удостоверений**.

## <a name="adding-zscaler-private-access-administrator-from-the-gallery"></a>Добавление Zscaler Private Access Administrator из коллекции

Чтобы настроить интеграцию Zscaler Private Access Administrator с Azure AD, необходимо добавить Zscaler Private Access Administrator из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Zscaler Private Access Administrator из коллекции, выполните следующее.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Zscaler Private Access Administrator**, выберите **Zscaler Private Access Administrator** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Zscaler Private Access Administrator в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в Zscaler Private Access Administrator для тестового пользователя **Britta Simon**.
Чтобы обеспечить работу единого входа, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Zscaler Private Access Administrator.

Чтобы настроить и проверить единый вход Azure AD в Zscaler Private Access Administrator, вам потребуется выполнить действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Zscaler Private Access Administrator](#configure-zscaler-private-access-administrator-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя Zscaler Private Access Administrator](#create-zscaler-private-access-administrator-test-user)** требуется для того, чтобы в Zscaler Private Access Administrator существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Zscaler Private Access Administrator, выполните следующее:

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Zscaler Private Access Administrator** выберите **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** выполните следующие действия.

    ![Снимок экрана: раздел "Базовая конфигурация SAML", в котором можно ввести описанные значения.](common/idp-relay.png)

    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `https://<subdomain>.private.zscaler.com/auth/metadata`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<subdomain>.private.zscaler.com/auth/sso`.

    c. Щелкните **Задать дополнительные URL-адреса**.

    d. В текстовом поле **Состояние ретранслятора** введите такой URL-адрес: `idpadminsso`

5.  Если вы хотите настроить приложение в **режиме, инициированном поставщиком услуг**, выполните следующие действия.

    ![Снимок экрана, на котором показан параметр "Задать дополнительные URL-адреса" и поле для ввода URL-адреса входа](common/both-signonurl.png)

    В текстовое поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<subdomain>.private.zscaler.com/auth/sso`.   

    > [!NOTE]
    > Эти значения приведены для примера. Замените их фактическими значениями идентификатора, URL-адреса ответа и URL-адреса входа. Обратитесь в [службу поддержки Zscaler Private Access Administrator](https://help.zscaler.com/zpa-submit-ticket), чтобы получить эти значения. Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

6. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

7. Скопируйте требуемый URL-адрес из раздела **Настройка Zscaler Private Access Administrator**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

### <a name="configure-zscaler-private-access-administrator-single-sign-on"></a>Настройка единого входа в Zscaler Private Access Administrator

1. В другом окне веб-браузера войдите в Zscaler Private Access Administrator с правами администратора.

2. В верхней части страницы щелкните **Administration** (Администрирование) и перейдите в раздел **AUTHENTICATION** (Аутентификация). Щелкните **IdP Configuration** (Конфигурация IdP).

    ![Администрирование Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_admin.png)

3. В правом верхнем углу щелкните **Add IdP Configuration** (Добавить конфигурацию IdP). 

    ![Добавление IdP в Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_addpidp.png)

4. На странице **Add IdP Configuration** (Добавление конфигурации IdP) выполните следующие действия.
 
    ![Выбор IdP в Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_idpselect.png)

    а. Щелкните **Select File** (Выбрать файл), чтобы передать скачанный файл метаданных из Azure AD и указать его в поле **IdP Metadata File Upload** (Передача файла метаданных IdP).

    b. Будут прочитаны **метаданные IdP** из Azure AD, а также будут заполнены все поля сведений, как показано ниже.

    ![Конфигурация IdP в Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/idpconfig.png)

    c. Для параметра **Single Sign On** (Единый вход) выберите значение **Administrator** (Администратор).

    d. Выберите домен в поле **Domains** (Домены).
    
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
  
    b. В поле **Имя пользователя** введите `brittasimon@yourcompanydomain.extension`. Например BrittaSimon@contoso.com.

    c. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле "Пароль".

    d. Нажмите кнопку **Создать**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к Zscaler Private Access Administrator.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **Zscaler Private Access Administrator**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Zscaler Private Access Administrator**.

    ![Ссылка на Zscaler Private Access Administrator в списке "Приложения"](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-zscaler-private-access-administrator-test-user"></a>Создание тестового пользователя Zscaler Private Access Administrator

Чтобы пользователи Azure AD могли выполнять вход в Zscaler Private Access Administrator, они должны быть подготовлены в Zscaler Private Access Administrator. В случае Zscaler Private Access Administrator подготовка выполняется вручную.

**Чтобы подготовить учетную запись пользователя, сделайте следующее:**

1. Войдите на корпоративный сайт Zscaler Private Access Administrator с правами администратора.

2. В верхней части страницы щелкните **Administration** (Администрирование) и перейдите в раздел **AUTHENTICATION** (Аутентификация). Щелкните **IdP Configuration** (Конфигурация IdP).

    ![Администрирование Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_admin.png)

3. Щелкните **Administrators** (Администраторы) из левой части меню.

    ![Администратор Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_adminstrator.png)

4. В правом верхнем углу щелкните **Add Administrator** (Добавить администратора).

    ![Добавление администратора Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_addadmin.png)

5. На странице **Add Administrator** (Добавление администратора) сделайте следующее.

    ![Администратор Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_useradmin.png)

    а. В текстовом поле **Username** (Имя пользователя) введите адрес электронной почты пользователя, например BrittaSimon@contoso.com.

    b. В текстовом поле **Password** (Пароль) введите пароль.

    c. В текстовом поле **Confirm Password** (Подтверждение пароля) повторно введите пароль.

    d. Для параметра **Role** (Роль) выберите значение **Zscaler Private Access Administrator**.

    д) В текстовое поле **Email** (Электронная почта) введите адрес электронной почты пользователя, например BrittaSimon@contoso.com.

    е) В текстовом поле **Phone** (Телефон) введите номер телефона.

    ж. В текстовом поле **Timezone** (Часовой пояс) выберите часовой пояс.

    h. Выберите команду **Сохранить**.

### <a name="test-single-sign-on"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку Zscaler Private Access Administrator на панели доступа, вы автоматически войдете в приложение Zscaler Private Access Administrator, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)