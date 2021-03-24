---
title: Руководство по интеграции Azure Active Directory с ON24 Virtual Environment SAML Connection | Документация Майкрософт
description: Сведения о настройке единого входа между Azure Active Directory и ON24 Virtual Environment SAML Connection.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/13/2019
ms.author: jeedes
ms.openlocfilehash: 30fd3843b50ac6b075d33e961986b94ee2496fef
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92518530"
---
# <a name="tutorial-azure-active-directory-integration-with-on24-virtual-environment-saml-connection"></a>Руководство по интеграции Azure Active Directory с ON24 Virtual Environment SAML Connection

В этом руководстве описано, как интегрировать ON24 Virtual Environment SAML Connection с Azure Active Directory (Azure AD).
Интеграция ON24 Virtual Environment SAML Connection с Azure AD обеспечивает следующие преимущества:

* С помощью Azure AD вы можете контролировать доступ к приложению ON24 Virtual Environment SAML Connection.
* Вы можете включить автоматический вход пользователей в ON24 Virtual Environment SAML Connection (единый вход) под учетной записью Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD с ON24 Virtual Environment SAML Connection, вам потребуется:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка ON24 Virtual Environment SAML Connection с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* ON24 Virtual Environment SAML Connection поддерживает единый вход, инициированный **поставщиком услуг** и **поставщиком удостоверений**

## <a name="adding-on24-virtual-environment-saml-connection-from-the-gallery"></a>Добавление ON24 Virtual Environment SAML Connection из коллекции

Чтобы настроить интеграцию приложения ON24 Virtual Environment SAML Connection с Azure AD, необходимо добавить это приложении из коллекции в список управляемых приложений SaaS.

**Чтобы добавить ON24 Virtual Environment SAML Connection из коллекции, сделайте следующее:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **ON24 Virtual Environment SAML Connection**, выберите **ON24 Virtual Environment SAML Connection** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить приложение.

     ![ON24 Virtual Environment SAML Connection в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в ON24 Virtual Environment SAML Connection с использованием тестового пользователя **Britta Simon**.
Для обеспечения единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в ON24 Virtual Environment SAML Connection.

Чтобы настроить и проверить единый вход Azure AD в ON24 Virtual Environment SAML Connection, вам потребуется выполнить действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в ON24 Virtual Environment SAML Connection](#configure-on24-virtual-environment-saml-connection-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя ON24 Virtual Environment SAML Connection](#create-on24-virtual-environment-saml-connection-test-user)** требуется для того, чтобы в ON24 Virtual Environment SAML Connection существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в ON24 Virtual Environment SAML Connection, сделайте следующее:

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **ON24 Virtual Environment SAML Connection** щелкните **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** выполните следующие действия.

    ![Сведения о домене и URL-адресах единого входа ON24 Virtual Environment SAML Connection](common/idp-relay.png)

    а. В текстовом поле **Идентификатор** введите URL-адрес:

     **URL-адрес рабочей среды**
    
    `SAML-VSHOW.on24.com`

    `SAML-Gateway.on24.com` 

    `SAP PROD SAML-EliteAudience.on24.com` 
                
     **URL-адрес среды контроля качества**
    
    `SAMLQA-VSHOW.on24.com` 

    `SAMLQA-Gateway.on24.com` 

    `SAMLQA-EliteAudience.on24.com`

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес:

     **URL-адрес рабочей среды**
    
    `https://federation.on24.com/sp/ACS.saml2`

    `https://federation.on24.com/sp/eyJ2c2lkIjoiU0FNTC1WU2hvdy5vbjI0LmNvbSJ9/ACS.saml2`

    `https://federation.on24.com/sp/eyJ2c2lkIjoiU0FNTC1HYXRld2F5Lm9uMjQuY29tIn0/ACS.saml2`

    `https://federation.on24.com/sp/eyJ2c2lkIjoiU0FNTC1FbGl0ZUF1ZGllbmNlLm9uMjQuY29tIn0/ACS.saml2`

     **URL-адрес среды контроля качества**
    
    `https://qafederation.on24.com/sp/ACS.saml2`

    `https://qafederation.on24.com/sp/eyJ2c2lkIjoiU0FNTFFBLVZzaG93Lm9uMjQuY29tIn0/ACS.saml2`

    `https://qafederation.on24.com/sp/eyJ2c2lkIjoiU0FNTFFBLUdhdGV3YXkub24yNC5jb20ifQ/ACS.saml2`
     
    `https://qafederation.on24.com/sp/eyJ2c2lkIjoiU0FNTFFBLUVsaXRlQXVkaWVuY2Uub24yNC5jb20ifQ/ACS.saml2` 

    c. Щелкните **Задать дополнительные URL-адреса**. 

    d. В текстовом поле **Состояние ретранслятора** введите такой URL-адрес: `https://vshow.on24.com/vshow/ms_azure_saml_test?r=<ID>`

5.  Если вы хотите настроить приложение в **режиме, инициированном поставщиком услуг**, выполните следующие действия.

    ![Снимок экрана: раздел "Задать дополнительные URL-адреса" с выделенным текстовым полем "URL-адрес для входа".](common/both-signonurl.png)

    В текстовое поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://vshow.on24.com/vshow/<INSTANCENAME>`.

    > [!NOTE]
    > Эти значения приведены для примера. Замените их фактическими значениями состояния ретранслятора и URL-адреса для входа. Чтобы получить эти значения, обратитесь в [службу поддержки клиентов ON24 Virtual Environment SAML Connection](https://www.on24.com/contact-us/). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

4. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

6. Скопируйте требуемые URL-адреса в разделе **Настройка ON24 Virtual Environment SAML Connection** .

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

### <a name="configure-on24-virtual-environment-saml-connection-single-sign-on"></a>Настройка единого входа в ON24 Virtual Environment SAML Connection

Чтобы настроить единый вход на стороне **ON24 Virtual Environment SAML Connection**, отправьте скачанный **XML-файл метаданных федерации** и соответствующие URL-адреса, скопированные на портале Azure, [группе поддержки ON24 Virtual Environment SAML Connection](https://www.on24.com/about-us/support/). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

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

В этом разделе описано, как позволить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к ON24 Virtual Environment SAML Connection.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **ON24 Virtual Environment SAML Connection**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **ON24 Virtual Environment SAML Connection**.

    ![Ссылка на ON24 Virtual Environment SAML Connection в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-on24-virtual-environment-saml-connection-test-user"></a>Создание тестового пользователя в ON24 Virtual Environment SAML Connection

В этом разделе вы создадите пользователя с именем Britta Simon в ON24 Virtual Environment SAML Connection. Обратитесь в [группу поддержки ON24 Virtual Environment SAML Connection](https://www.on24.com/about-us/support/), чтобы добавить пользователей на платформу ON24 Virtual Environment SAML Connection. Перед использованием единого входа необходимо создать и активировать пользователей.

### <a name="test-single-sign-on"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку ON24 Virtual Environment SAML Connection на Панели доступа, вы автоматически войдете в приложение ON24 Virtual Environment SAML Connection, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)