---
title: Руководство по интеграции Azure Active Directory с Hornbill | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Hornbill.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/15/2019
ms.author: jeedes
ms.openlocfilehash: 4131f1d0ef36eb49641ac184cd3f72fb719ea90e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92443448"
---
# <a name="tutorial-azure-active-directory-integration-with-hornbill"></a>Руководство по интеграции Azure Active Directory с Hornbill

Цель этого руководства — показать, как интегрировать Azure Active Directory (Azure AD) с приложением Hornbill.
Интеграция Azure AD с приложением Hornbill обеспечивает следующие преимущества.

* С помощью Azure AD вы можете контролировать доступ к Hornbill.
* Вы можете включить автоматический вход пользователей в Hornbill (единый вход) с помощью учетной записи Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD с Hornbill, вам потребуются:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка с поддержкой единого входа Hornbill.

> [!NOTE]
> Эту интеграцию также можно использовать в облачной среде Azure AD для государственных организаций США. Это приложение можно найти в коллекции облачных приложений с поддержкой Azure AD для государственных организаций США и настроить таким же образом, как и в общедоступном облаке.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Hornbill поддерживает единый вход, инициированный **поставщиком услуг**.
* Hornbill поддерживает **JIT**-подготовку пользователей.

## <a name="adding-hornbill-from-the-gallery"></a>Добавление Hornbill из коллекции

Чтобы настроить интеграцию Hornbill с Azure AD, необходимо добавить Hornbill из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Hornbill из коллекции, выполните следующие действия.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Hornbill**, выберите **Hornbill** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

     ![Hornbill в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в Hornbill с использованием тестового пользователя **Britta Simon**.
Чтобы обеспечить работу единого входа, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Hornbill.

Чтобы настроить и проверить единый вход Azure AD в Hornbill, вам потребуется выполнить действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Hornbill](#configure-hornbill-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя в Hornbill](#create-hornbill-test-user)** требуется для того, чтобы в Hornbill существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Hornbill, выполните следующие действия:

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Hornbill** щелкните **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    ![Сведения о домене и URL-адресах единого входа приложения Hornbill](common/sp-identifier.png)

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<SUBDOMAIN>.hornbill.com/<INSTANCE_NAME>/`.

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `https://<SUBDOMAIN>.hornbill.com/<INSTANCE_NAME>/lib/saml/auth/simplesaml/module.php/saml/sp/metadata.php/saml`.

    > [!NOTE]
    > Эти значения приведены для примера. Необходимо обновить эти значения действующим URL-адресом для входа и идентификатором. Чтобы получить их, обратитесь к [группе поддержки клиентов Hornbill](https://www.hornbill.com/support/?request/). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** нажмите кнопку "Копировать", чтобы копировать **URL-адрес метаданных федерации приложений** и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/copy-metadataurl.png)

### <a name="configure-hornbill-single-sign-on"></a>Настройка единого входа в Hornbill

1. В другом окне веб-браузера войдите в Hornbill в качестве администратора безопасности.

2. На домашней странице щелкните **System** (Система).

    ![Hornbill: система](./media/hornbill-tutorial/tutorial_hornbill_system.png)

3. Перейдите в раздел **Безопасность**.

    ![Hornbill: безопасность](./media/hornbill-tutorial/tutorial_hornbill_security.png)

4. Щелкните **SSO Profiles** (Профили единого входа).

    ![Hornbill: профили единого входа](./media/hornbill-tutorial/tutorial_hornbill_sso.png)

5. В правой части страницы щелкните **Add logo** (Добавить логотип).

    ![Hornbill: добавление логотипа](./media/hornbill-tutorial/tutorial_hornbill_addlogo.png)

6. На панели **Profile Details** (Сведения о профиле) щелкните **Import SAML Meta logo** (Импорт логотипа метаданных SAML).

    ![Hornbill: логотип](./media/hornbill-tutorial/tutorial_hornbill_logo.png)

7. На открывшейся странице в текстовое поле **URL** (URL-адрес) вставьте значение **URL-адреса метаданных федерации приложения**, скопированное на портале Azure, а затем нажмите кнопку **Process** (Обработать).

    ![Hornbill: обработка](./media/hornbill-tutorial/tutorial_hornbill_process.png)

8. После нажатия кнопки "Process" (Обработать) значения в разделе **Profile Details** (Сведения о профиле) заполнятся автоматически.

    ![Hornbill: страница 1](./media/hornbill-tutorial/tutorial_hornbill_ssopage.png)

    ![Hornbill: страница 2](./media/hornbill-tutorial/tutorial_hornbill_ssopage1.png)

    ![Hornbill: страница 3](./media/hornbill-tutorial/tutorial_hornbill_ssopage2.png)

9. Щелкните **Сохранить изменения**.

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

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к Hornbill.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения** и **Hornbill**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Hornbill**.

    ![Ссылка на Hornbill в списке "Приложения"](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** выберите **Britta Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-hornbill-test-user"></a>Создание тестового пользователя в Hornbill

В этом разделе вы создадите в Hornbill пользователя Britta Simon. Приложение Hornbill поддерживает JIT-подготовку пользователей, которая включена по умолчанию. В этом разделе никакие действия с вашей стороны не требуются. Если пользователь еще не существует в Hornbill, он создается после проверки подлинности.

> [!Note]
> Чтобы создать пользователя вручную, обратитесь к [группе поддержки Hornbill](https://www.hornbill.com/support/?request/).

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку Hornbill на панели доступа, вы автоматически войдете в приложение Hornbill, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)