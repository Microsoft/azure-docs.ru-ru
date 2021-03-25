---
title: Руководство по интеграции Azure Active Directory с My Award Points Top Sub/Top Team | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и My Award Points Top Sub/Top Team.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/01/2019
ms.author: jeedes
ms.openlocfilehash: f2328bd51712089f706c8491007f9f51eba52337
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92508134"
---
# <a name="tutorial-azure-active-directory-integration-with-my-award-points-top-subtop-team"></a>Руководство по интеграции Azure Active Directory с My Award Points Top Sub/Top Team

Из этого руководства вы узнаете, как интегрировать My Award Points Top Sub/Top Team с Azure Active Directory (AAD).
Интеграция Azure AD с приложением My Award Points Top Sub/Top Team обеспечивает следующие преимущества:

* Вы можете через Azure AD настраивать доступ в My Award Points Top Sub/Top Team.
* Вы можете включить автоматический вход пользователей в My Award Points Top Sub/Top Team (единый вход) с помощью учетной записи Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с My Award Points Top Sub/Top Team, вам потребуется следующее:

* Подписка Azure AD. (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка My Award Points Top Sub/Top Team с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* My Award Points Top Sub/Top Team поддерживает единый вход, инициируемый **поставщиком услуг**.

## <a name="adding-my-award-points-top-subtop-team-from-the-gallery"></a>Добавление My Award Points Top Sub/Top Team из коллекции

Чтобы настроить интеграцию My Award Points Top Sub/Top Team с Azure AD, нужно добавить My Award Points Top Sub/Top Team из коллекции в список управляемых приложений SaaS.

**Чтобы добавить My Award Points Top Sub/Top Team из коллекции, выполните следующие действия:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **My Award Points Top Sub/Top Team**, выберите **My Award Points Top Sub/Top Team** на панели результатов и щелкните **Добавить**, чтобы добавить это приложение.

     ![Приложение My Award Points Top Sub/Top Team в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

Из этого раздела вы узнаете, как настроить и проверить единый вход Azure AD в My Award Points Top Sub/Top Team с использованием тестового пользователя **Britta Simon**.
Чтобы единый вход функционировал, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в My Award Points Top Sub/Top Team.

Чтобы настроить и проверить единый вход Azure AD в My Award Points Top Sub/Top Team, вам потребуется выполнить действия в следующих стандартных блоках:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **Настройка единого входа в My Award Points Top Sub/Top Team** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **Создание тестового пользователя My Award Points Top Sub/Top Team** требуется для того, чтобы в My Award Points Top Sub/Top Team существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход в Azure AD с My Award Points Top Sub/Top Team, сделайте следующее.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **My Award Points Top Sub/Top Team** щелкните **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    ![Сведения о домене и URL-адресах единого входа для My Award Points Top Sub/Top Team](common/sp-signonurl.png)

    В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://microsoftrr.performnet.com/biwv1auth/Shibboleth.sso/Login?providerId=<Azure AD Identifier>`.

    > [!NOTE]
    > Это значение приведено для примера. Вы получите значение `<Azure AD Identifier>` на последующих этапах этого руководства.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

6. Требуемые URL-адреса вы можете скопировать из раздела **Настройка My Award Points Top Sub/Top Team**. 

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

    >[!NOTE]
    >Добавьте скопированное значение идентификатора Azure AD с URL-адресом входа вместо `<Azure AD Identifier>` в раздел **Базовая конфигурация SAML** на портале Azure.

### <a name="configure-my-award-points-top-subtop-team-single-sign-on"></a>Настройка единого входа в My Award Points Top Sub/Top Team

Чтобы настроить единый вход на стороне **My Award Points Top Sub/Top Team**, нужно отправить скачанный **XML-файл метаданных федерации** и соответствующие URL-адреса, скопированные на портале Azure, [группе поддержки My Award Points Top Sub/Top Team](mailto:myawardpoints@biworldwide.com). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

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

В этом разделе описано, как предоставить пользователю Britta Simon доступ к My Award Points Top Sub/Top Team и возможность использовать единый вход Azure.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **My Award Points Top Sub/Top Team**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **My Award Points Top Sub/Top Team**.

    ![Ссылка на My Award Points Top Sub/Top Team в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-my-award-points-top-subtop-team-test-user"></a>Создание тестового пользователя My Award Points Top Sub/Top Team

В этом разделе описано, как создать пользователя Britta Simon в приложении My Award Points Top Sub/Top Team. Обратитесь к [службе поддержки клиентов My Award Points Top Sub/Top Team](mailto:myawardpoints@biworldwide.com), чтобы добавить пользователей на платформу My Award Points Top Sub/Top Team. Перед использованием единого входа необходимо создать и активировать пользователей.

### <a name="test-single-sign-on"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув элемент "My Award Points Top Sub/Top Team" на Панели доступа, вы автоматически войдете в приложение My Award Points Top Sub/Top Team, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)