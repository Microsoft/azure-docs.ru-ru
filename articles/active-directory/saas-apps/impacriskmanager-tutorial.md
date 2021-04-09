---
title: Руководство по интеграции Azure Active Directory с IMPAC Risk Manager | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в IMPAC Risk Manager.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/14/2019
ms.author: jeedes
ms.openlocfilehash: 1462e815b5f045ab9d52bdea59b78b12ff77c19b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92460301"
---
# <a name="tutorial-azure-active-directory-integration-with-impac-risk-manager"></a>Руководство по интеграции Azure Active Directory с IMPAC Risk Manager

В этом руководстве описано, как интегрировать IMPAC Risk Manager с Azure Active Directory (Azure AD).
Интеграция Azure AD с IMPAC Risk Manager обеспечивает следующие преимущества:

* С помощью Azure AD можно контролировать доступ к IMPAC Risk Manager.
* Вы можете включить автоматический вход пользователей в IMPAC Risk Manager (единый вход) через учетную запись Azure Active Directory.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD с приложением IMPAC Risk Manager, вам потребуется следующее:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка IMPAC Risk Manager с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* IMPAC Risk Manager поддерживает единый вход, инициированный **поставщиком услуг и поставщиком удостоверений**.

## <a name="adding-impac-risk-manager-from-the-gallery"></a>Добавление IMPAC Risk Manager из коллекции.

Чтобы настроить интеграцию IMPAC Risk Manager в Azure AD, необходимо добавить IMPAC Risk Manager из коллекции в список управляемых приложений SaaS.

**Чтобы добавить IMPAC Risk Manager из коллекции, выполните следующие действия:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **IMPAC Risk Manager** и выберите **IMPAC Risk Manager** на панели результатов, а затем нажмите кнопку **Добавить**, чтобы добавить приложение.

    ![IMPAC Risk Manager в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в IMPAC Risk Manager с использованием тестового пользователя **Britta Simon**.
Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в IMPAC Risk Manager.

Чтобы настроить и проверить единый вход Azure AD в IMPAC Risk Manager, потребуется выполнить действия в следующих стандартных блоках:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в IMPAC Risk Manager](#configure-impac-risk-manager-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя IMPAC Risk Manager](#create-impac-risk-manager-test-user)** требуется для того, чтобы в IMPAC Risk Manager существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в IMPAC Risk Manager, выполните приведенные ниже действия.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **IMPAC Risk Manager** выберите **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** выполните следующие действия.

    ![Снимок экрана, на котором показан раздел "Базовая конфигурация SAML", где можно ввести идентификатор, URL-адрес ответа и нажать кнопку "Сохранить"](common/idp-intiated.png)

    а. В текстовом поле **Идентификатор** введите значение, предоставленное IMPAC.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в таком формате:

    | Среда | Шаблон URL-адреса |
    | ---------------|--------------- |
    | Рабочая среда |`https://www.riskmanager.co.nz/DotNet/SSOv2/AssertionConsumerService.aspx?client=<ClientSuffix>`|
    | Промежуточное хранение и обучение  |`https://staging.riskmanager.co.nz/DotNet/SSOv2/AssertionConsumerService.aspx?client=<ClientSuffix>`|
    | Разработка  |`https://dev.riskmanager.co.nz/DotNet/SSOv2/AssertionConsumerService.aspx?client=<ClientSuffix>`|
    | Контроль качества |`https://QA.riskmanager.co.nz/DotNet/SSOv2/AssertionConsumerService.aspx?client=<ClientSuffix>`|
    | Тестирование |`https://test.riskmanager.co.nz/DotNet/SSOv2/AssertionConsumerService.aspx?client=<ClientSuffix>`|
    | | |

5. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    ![Снимок экрана, на котором показан параметр "Задать дополнительные URL-адреса" и поле для ввода URL-адреса входа](common/metadata-upload-additional-signon.png)

    В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате:

    | Среда | Шаблон URL-адреса |
    | ---------------|--------------- |
    | Рабочая среда |`https://www.riskmanager.co.nz/SSOv2/<ClientSuffix>`|
    | Промежуточное хранение и обучение  |`https://staging.riskmanager.co.nz/SSOv2/<ClientSuffix>`|
    | Разработка  |`https://dev.riskmanager.co.nz/SSOv2/<ClientSuffix>`|
    | Контроль качества |`https://QA.riskmanager.co.nz/SSOv2/<ClientSuffix>`|
    | Тестирование |`https://test.riskmanager.co.nz/SSOv2/<ClientSuffix>`|
    | | |

    > [!NOTE]
    > Эти значения приведены для примера. Замените их фактическими значениями идентификатора, URL-адреса ответа и URL-адреса входа. Чтобы получить эти значения, обратитесь в [службу поддержки клиентов IMPAC Risk Manager](mailto:rmsupport@Impac.co.nz). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

6. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (Base64)** из предложенных вариантов, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

7. Скопируйте требуемый URL-адрес из раздела **Настройка IMPAC Risk Manager**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

### <a name="configure-impac-risk-manager-single-sign-on"></a>Настройка единого входа IMPAC Risk Manager

Чтобы настроить единый вход на стороне **IMPAC Risk Manager**, нужно отправить скачанный **сертификат (Base64)** и соответствующие URL-адреса, скопированные на портале Azure, [группе поддержки IMPAC Risk Manager](mailto:rmsupport@Impac.co.nz). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

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

В этом разделе описано, как предоставить пользователю Britta Simon доступ к IMPAC Risk Manager, чтобы он мог использовать единый вход Azure.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **IMPAC Risk Manager**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **IMPAC Risk Manager**.

    ![Ссылка на IMPAC Risk Manager в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-impac-risk-manager-test-user"></a>Создание тестового пользователя приложения IMPAC Risk Manager

В этом разделе описано, как создать пользователя Britta Simon в приложении IMPAC Risk Manager. Чтобы добавить пользователей на платформу IMPAC Risk Manager, обратитесь в [службу поддержки IMPAC Risk Manager](mailto:rmsupport@Impac.co.nz). Перед использованием единого входа необходимо создать и активировать пользователей.

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку IMPAC Risk Manager на Панели доступа, вы автоматически войдете в приложение IMPAC Risk Manager, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)