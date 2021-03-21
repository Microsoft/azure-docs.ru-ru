---
title: Руководство. Интеграция Azure Active Directory с Carlson Wagonlit Travel | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Carlson Wagonlit Travel.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/12/2019
ms.author: jeedes
ms.openlocfilehash: 86c2d9585abd546acb3efb22256d3f3b70bfb972
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97674401"
---
# <a name="tutorial-azure-active-directory-integration-with-carlson-wagonlit-travel"></a>Руководство. Интеграция Azure Active Directory с Carlson Wagonlit Travel

В этом руководстве описано, как интегрировать Carlson Wagonlit Travel с Azure Active Directory (Azure AD).
Интеграция Azure AD с Carlson Wagonlit Travel обеспечивает следующие преимущества.

* С помощью Azure AD вы можете контролировать доступ к Carlson Wagonlit Travel.
* Учетные записи Azure AD позволяют включить автоматический вход пользователей в Carlson Wagonlit Travel (единый вход).
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с Carlson Wagonlit Travel, вам потребуется:

* Подписка Azure AD. (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка Carlson Wagonlit Travel с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* В Carlson Wagonlit Travel поддерживается единый вход, инициированный **поставщиком удостоверений**.

## <a name="adding-carlson-wagonlit-travel-from-the-gallery"></a>Добавление Carlson Wagonlit Travel из коллекции

Чтобы настроить интеграцию Carlson Wagonlit Travel с Azure AD, необходимо добавить Carlson Wagonlit Travel из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Carlson Wagonlit Travel из коллекции, выполните следующие действия.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Carlson Wagonlit Travel**, а затем выберите **Carlson Wagonlit Travel** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить приложение.

    ![Carlson Wagonlit Travel в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В рамках этого раздела вы настроите и проверите единый вход Azure AD в Carlson Wagonlit Travel с использованием тестового пользователя **Britta Simon**.
Чтобы обеспечить единый вход, свяжите пользователя Azure AD с соответствующим пользователем в Carlson Wagonlit Travel.

Чтобы настроить и проверить единый вход Azure AD в Carlson Wagonlit Travel, вам потребуется выполнить действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Carlson Wagonlit Travel](#configure-carlson-wagonlit-travel-single-sign-on)** требуется, чтобы определить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя приложения Carlson Wagonlit Travel](#create-carlson-wagonlit-travel-test-user)** требуется, чтобы в Carlson Wagonlit Travel существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Carlson Wagonlit Travel, сделайте следующее:

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Carlson Wagonlit Travel** щелкните **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    ![Сведения о домене и URL-адресах для единого входа в приложение Carlson Wagonlit Travel](common/idp-identifier.png)

    В текстовом поле **Идентификатор** введите значение `cwt-stage`.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

6. Требуемые URL-адреса можно скопировать из раздела **настройки Carlson Wagonlit Travel**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD.

    c. URL-адрес выхода.

### <a name="configure-carlson-wagonlit-travel-single-sign-on"></a>Настройка единого входа в Carlson Wagonlit Travel

Чтобы настроить единый вход на стороне **Carlson Wagonlit Travel**, отправьте скачанный **XML-файл метаданных федерации** и URL-адреса, скопированные на портале Azure, в [службу технической поддержки Carlson Wagonlit Travel](https://www.mycwt.com/traveler-help/). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

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

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к Carlson Wagonlit Travel.

1. На портале Azure последовательно выберите **Корпоративные приложения**, **Все приложения** и **Carlson Wagonlit Travel**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Carlson Wagonlit Travel**.

    ![Ссылка на Carlson Wagonlit Travel в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** выберите **Britta Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-carlson-wagonlit-travel-test-user"></a>Создание тестового пользователя Carlson Wagonlit Travel

В этом разделе описано, как создать пользователя Britta Simon в приложении Carlson Wagonlit Travel. Обратитесь в [службу поддержки Carlson Wagonlit Travel](https://www.mycwt.com/traveler-help/), чтобы добавить пользователей на платформу Carlson Wagonlit Travel. Перед использованием единого входа необходимо создать и активировать пользователей.

### <a name="test-single-sign-on"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку Carlson Wagonlit Travel на Панели доступа, вы автоматически войдете в приложение Carlson Wagonlit Travel, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)