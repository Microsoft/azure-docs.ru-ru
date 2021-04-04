---
title: Руководство по интеграции Azure Active Directory с Birst Agile Business Analytics | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Birst Agile Business Analytics.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/07/2019
ms.author: jeedes
ms.openlocfilehash: 124485819bf7fab02e2d62bec46ad50468589773
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97674354"
---
# <a name="tutorial-azure-active-directory-integration-with-birst-agile-business-analytics"></a>Руководство. Интеграция Azure Active Directory с Birst Agile Business Analytics

В этом руководстве описано, как интегрировать Birst Agile Business Analytics с Azure Active Directory (Azure AD).
Интеграция Azure AD с приложением Birst Agile Business Analytics обеспечивает следующие преимущества:

* С помощью Azure AD вы можете контролировать доступ к Birst Agile Business Analytics.
* Вы можете включить автоматический вход пользователей в Birst Agile Business Analytics (единый вход) с использованием учетной записи Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD с Birst Agile Business Analytics, вам потребуется:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка на Birst Agile Business Analytics с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Birst Agile Business Analytics поддерживает единый вход, инициированный **поставщиком услуг**.

## <a name="adding-birst-agile-business-analytics-from-the-gallery"></a>Добавление Birst Agile Business Analytics из коллекции

Чтобы настроить интеграцию Birst Agile Business Analytics с Azure AD, необходимо добавить Birst Agile Business Analytics из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Birst Agile Business Analytics из коллекции, сделайте следующее:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Birst Agile Business Analytics**, выберите **Birst Agile Business Analytics** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Birst Agile Business Analytics в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в Birst Agile Business Analytics с использованием тестового пользователя **Britta Simon**.
Для работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Birst Agile Business Analytics.

Чтобы настроить и проверить единый вход Azure AD в Birst Agile Business Analytics, вам потребуется выполнить действия в следующих стандартных блоках:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Birst Agile Business Analytics](#configure-birst-agile-business-analytics-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя Birst Agile Business Analytics](#create-birst-agile-business-analytics-test-user)** нужно для того, чтобы в Birst Agile Business Analytics также существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Birst Agile Business Analytics, сделайте следующее:

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Birst Agile Business Analytics** выберите **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    ![Сведения о домене и URL-адресах единого входа для приложения Birst Agile Business Analytics](common/sp-intiated.png)

    В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://login.bws.birst.com/SAMLSSO/Services.aspx?birst.idpid=TENANTIDPID`

    URL-адрес зависит от центра обработки данных, в котором расположена учетная запись Birst.

   * Для центра обработки данных в США используйте шаблон `https://login.bws.birst.com/SAMLSSO/Services.aspx?birst.idpid=TENANTIDPID`.

   * Для центра обработки данных в Европе используйте шаблон `https://login.eu1.birst.com/SAMLSSO/Services.aspx?birst.idpid=TENANTIDPID`.

     > [!NOTE]
     > Это значение приведено для справки. Вместо него необходимо указать фактический URL-адрес входа. Обратитесь к [группе поддержки Birst Agile Business Analytics](mailto:info@birst.com) для получения этого значения.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (Base64)** из предложенных вариантов, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

6. Скопируйте соответствующий вашим требованиям URL-адрес из раздела **Настройка Birst Agile Business Analytics**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD.

    c. URL-адрес выхода.

### <a name="configure-birst-agile-business-analytics-single-sign-on"></a>Настройка единого входа в Birst Agile Business Analytics

Чтобы настроить единый вход на стороне **Birst Agile Business Analytics**, нужно отправить скачанный **сертификат (Base64)** и соответствующие скопированные URL-адреса с портала Azure в [группу поддержки Birst Agile Business Analytics](mailto:info@birst.com). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

> [!NOTE]
> Укажите в письме к группе поддержки Birst, что для интеграции необходимо использовать алгоритм SHA256 (SHA1 не будет поддерживаться), чтобы специалисты по поддержке настроили единый вход на соответствующем сервере, например **app2101**.

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

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к Birst Agile Business Analytics.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **Birst Agile Business Analytics**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Birst Agile Business Analytics**.

    ![Ссылка на Birst Agile Business Analytics в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-birst-agile-business-analytics-test-user"></a>Создание тестового пользователя в Birst Agile Business Analytics

В этом разделе в Birst Agile Business Analytics создается пользователь с именем Britta Simon. Обратитесь к [группе поддержки Birst Agile Business Analytics](mailto:info@birst.com), чтобы добавить пользователей на платформу Birst Agile Business Analytics. Перед использованием единого входа необходимо создать и активировать пользователей.

### <a name="test-single-sign-on"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку Birst Agile Business Analytics на Панели доступа, вы автоматически войдете в приложение Birst Agile Business Analytics, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)