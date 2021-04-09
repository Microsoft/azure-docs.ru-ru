---
title: Руководство по интеграции Azure Active Directory с Trisotech Digital Enterprise Server | Документация Майкрософт
description: Сведения о том, как настроить единый вход Azure Active Directory в Trisotech Digital Enterprise Server.
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
ms.openlocfilehash: 05978b7300c401a0658d6c3316f37735fcb70c20
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92521957"
---
# <a name="tutorial-azure-active-directory-integration-with-trisotech-digital-enterprise-server"></a>Руководство по интеграции Azure Active Directory с Trisotech Digital Enterprise Server

В этом руководстве описано, как интегрировать Trisotech Digital Enterprise Server с Azure Active Directory (Azure AD).
Интеграция Trisotech Digital Enterprise Server с Azure AD обеспечивает следующие преимущества:

* С помощью Azure AD можно контролировать доступ к Trisotech Digital Enterprise Server.
* Вы можете включить автоматический вход пользователей в Trisotech Digital Enterprise Server (единый вход) с применением учетной записи Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD с приложением Trisotech Digital Enterprise Server, вам потребуется следующее:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка Trisotech Digital Enterprise Server с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Trisotech Digital Enterprise Server поддерживает единый вход, инициированный **поставщиком услуг**.

* Trisotech Digital Enterprise Server поддерживает **JIT**-подготовку пользователей.

## <a name="adding-trisotech-digital-enterprise-server-from-the-gallery"></a>Добавление Trisotech Digital Enterprise Server из коллекции

Чтобы настроить интеграцию Trisotech Digital Enterprise Server с Azure AD, необходимо добавить Trisotech Digital Enterprise Server из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Trisotech Digital Enterprise Server из коллекции, сделайте следующее:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Trisotech Digital Enterprise Server**, выберите **Trisotech Digital Enterprise Server** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

     ![Trisotech Digital Enterprise Server в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в Trisotech Digital Enterprise Server для тестового пользователя **Britta Simon**.
Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Trisotech Digital Enterprise Server.

Чтобы настроить и проверить единый вход Azure AD в Trisotech Digital Enterprise Server, вам потребуется выполнить действия в следующих стандартных блоках:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Trisotech Digital Enterprise Server](#configure-trisotech-digital-enterprise-server-single-sign-on)** необходима, чтобы задать параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя Trisotech Digital Enterprise Server](#create-trisotech-digital-enterprise-server-test-user)** требуется для того, чтобы в Trisotech Digital Enterprise Server существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Trisotech Digital Enterprise Server, сделайте следующее:

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Trisotech Digital Enterprise Server** выберите **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    ![Сведения о домене и URL-адресах приложения Trisotech Digital Enterprise Server](common/sp-identifier.png)

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<companyname>.trisotech.com`.

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `https://<companyname>.trisotech.com`.

    > [!NOTE]
    > Эти значения приведены для примера. Необходимо обновить эти значения действующим URL-адресом для входа и идентификатором. Обратитесь в [клиентскую службу поддержки Trisotech Digital Enterprise Server](mailto:support@trisotech.com), чтобы получить эти значения. Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

4. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** нажмите кнопку "Копировать", чтобы копировать **URL-адрес метаданных федерации приложений** и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/copy-metadataurl.png)

### <a name="configure-trisotech-digital-enterprise-server-single-sign-on"></a>Настройка единого входа в Trisotech Digital Enterprise Server

1. В другом окне веб-браузера войдите на сайт компании Trisotech Digital Enterprise Server с правами администратора.

2. Щелкните **значок меню**, а затем выберите **Administration** (Администрирование).

    ![Снимок экрана, на котором показан значок администрирования в Microsoft Digital Enterprise Server.](./media/trisotechdigitalenterpriseserver-tutorial/user1.png)

3. Выберите пункт **User Provider** (Поставщик пользователей).

    ![Снимок экрана, на котором показан выбранный пункт меню User Provider (Поставщик пользователей).](./media/trisotechdigitalenterpriseserver-tutorial/user2.png)

4. В разделе **User Provider Configurations** (Конфигурация поставщика пользователей) выполните следующие действия:

    ![Снимок экрана для страницы настройки поставщика пользователей, где можно ввести описанные значения.](./media/trisotechdigitalenterpriseserver-tutorial/user3.png)

    а. В раскрывающемся списке в поле **Authentication Method** (Способ проверки подлинности) выберите **язык разметки заявлений системы безопасности 2 (SAML 2)** .

    b. В текстовое поле **URL-адрес метаданных** вставьте значение **URL-адреса метаданных федерации приложений**, скопированное на портале Azure.

    c. В текстовое поле **Application ID** (Идентификатор приложения) введите URL-адрес в следующем формате: `https://<companyname>.trisotech.com`.

    d. Нажмите кнопку **Сохранить**

    д) Введите доменное имя в текстовое поле **Allowed Domains (empty means everyone)** (Разрешенные домены (если оставить это поле пустым, будут использоваться все домены)). Пользователям, которые соответствуют разрешенным доменам, будут автоматически назначены лицензии.

    е) Нажмите кнопку **Сохранить**

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

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив доступ к Trisotech Digital Enterprise Server.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **Trisotech Digital Enterprise Server**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Trisotech Digital Enterprise Server**.

    ![Ссылка на приложение Trisotech Digital Enterprise Server в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-trisotech-digital-enterprise-server-test-user"></a>Создание тестового пользователя Trisotech Digital Enterprise Server

В этом разделе вы создадите в Trisotech Digital Enterprise Server пользователя с именем Britta Simon. Приложение Trisotech Digital Enterprise Server поддерживает JIT-подготовку пользователей. Эта функция включена по умолчанию. В этом разделе никакие действия с вашей стороны не требуются. Если пользователь еще не существует в Trisotech Digital Enterprise Server, его создадут после проверки подлинности.

>[!Note]
>Если необходимо создать пользователя вручную, обратитесь в [службу поддержки Trisotech Digital Enterprise Server](mailto:support@trisotech.com).

### <a name="test-single-sign-on"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку Trisotech Digital Enterprise Server на Панели доступа, вы автоматически войдете в приложение Trisotech Digital Enterprise Server, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)