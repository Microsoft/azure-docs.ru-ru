---
title: Руководство по интеграции Azure Active Directory со Small Improvements | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Small Improvements.
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
ms.openlocfilehash: 6eced120a05ddaca8d8cf426fd2a977891b3e36b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "95997143"
---
# <a name="tutorial-azure-active-directory-integration-with-small-improvements"></a>Учебник. Интеграция Azure Active Directory со Small Improvements

В этом руководстве описано, как интегрировать Small Improvements с Azure Active Directory (Azure AD).
Интеграция Azure AD с приложением Small Improvements обеспечивает следующие преимущества.

* С помощью Azure AD вы можете контролировать доступ к приложению Small Improvements.
* Вы можете включить автоматический вход пользователей (единый вход) в Small Improvements с использованием учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD со Small Improvements, вам потребуется:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка Small Improvements с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Small Improvements поддерживает единый вход, инициированный **поставщиком услуг**.

## <a name="adding-small-improvements-from-the-gallery"></a>Добавление Small Improvements из коллекции

Чтобы настроить интеграцию Small Improvements с Azure AD, необходимо добавить Small Improvements из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Small Improvements из коллекции, выполните следующие действия.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Small Improvements**, выберите **Small Improvements** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

     ![Small Improvements в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в Small Improvements с использованием тестового пользователя **Britta Simon**.
Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Small Improvements.

Чтобы настроить и проверить единый вход Azure AD в Small Improvements, вам потребуется выполнить действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Small Improvements](#configure-small-improvements-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя Small Improvements](#create-small-improvements-test-user)** требуется для того, чтобы в Small Improvements существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Small Improvements, выполните следующие действия.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Small Improvements** выберите **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    ![Сведения о домене и URL-адресах единого входа для приложения Small Improvements](common/sp-identifier.png)

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<subdomain>.small-improvements.com`.

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `https://<subdomain>.small-improvements.com`.

    > [!NOTE]
    > Эти значения приведены для примера. Необходимо обновить эти значения действующим URL-адресом для входа и идентификатором. Чтобы получить эти значения, обратитесь к [группе поддержки клиентов Small Improvements](mailto:support@small-improvements.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (Base64)** из предложенных вариантов, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

6. Скопируйте требуемый URL-адрес из раздела **Настройка Small Improvements**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

### <a name="configure-small-improvements-single-sign-on"></a>Настройка единого входа Small Improvements

1. В другом окне браузера войдите на корпоративный сайт Small Improvements с правами администратора.

1. На главной странице панели мониторинга нажмите слева кнопку **Администрирование** .

    ![Снимок экрана: выбрана кнопка Administration (Администрирование).](./media/smallimprovements-tutorial/tutorial_smallimprovements_06.png) 

1. Нажмите кнопку **SAML SSO** (Единый вход SAML) в разделе **Integrations** (Интеграции).

    ![Снимок экрана: значок SAML SSO (Единый вход SAML) в разделе Integrations (Интеграции).](./media/smallimprovements-tutorial/tutorial_smallimprovements_07.png) 

1. На странице настройки единого входа выполните следующие действия.

    ![Снимок экрана: страница SSO Setup (Настройка единого входа), где можно ввести описанные значения.](./media/smallimprovements-tutorial/tutorial_smallimprovements_08.png)  

    а. В текстовое поле **HTTP Endpoint** (Конечная точка HTTP) вставьте значение **URL-адреса входа**, скопированное на портале Azure.

    b. Откройте загруженный сертификат в блокноте, скопируйте его содержимое в буфер обмена, а затем вставьте его в текстовое поле **Сертификат X.509** . 

    c. Если вы хотите предоставить пользователям возможность единого входа и аутентификации через форму входа, установите флажок **Enable access via login/password too** (Включить также доступ по имени входа и паролю).  

    d. Введите значение имени единого входа в текстовом поле **SAML Prompt** (Приглашение SAML).  

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
  
    b. В поле **Имя пользователя** введите **brittasimon@yourcompanydomain.extension** .  
    Например BrittaSimon@contoso.com.

    c. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле "Пароль".

    d. Нажмите кнопку **Создать**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к Small Improvements.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **Small Improvements**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Small Improvements**.

    ![Ссылка на Small Improvements в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-small-improvements-test-user"></a>Создание тестового пользователя в Small Improvements

Чтобы пользователи Azure AD могли выполнять вход в Small Improvements, они должны быть подготовлены в Small Improvements. В случае Small Improvements подготовка выполняется вручную.

**Чтобы подготовить учетную запись пользователя, сделайте следующее:**

1. Войдите на корпоративный сайт Small Improvements с правами администратора.

1. На главной странице перейдите к меню в левой части страницы и щелкните **Администрирование**.

1. Нажмите кнопку **Каталог пользователей** в разделе "Управление пользователями".

    ![Снимок экрана: элемент User Directory (Каталог пользователей), выбранный в разделе Administration Overview (Обзор администрирования).](./media/smallimprovements-tutorial/tutorial_smallimprovements_10.png) 

1. Щелкните **Add users** (Добавить пользователей).

    ![Снимок экрана: кнопка Add users (Добавить пользователей).](./media/smallimprovements-tutorial/tutorial_smallimprovements_11.png) 

1. В диалоговом окне **Add Users** (Добавление пользователей) сделайте следующее. 

    ![Снимок экрана: диалоговое окно Add Users (Добавление пользователей), где можно ввести описанные значения.](./media/smallimprovements-tutorial/tutorial_smallimprovements_12.png)

    а. В текстовое поле **First name** (Имя) введите имя пользователя, например **Britta**.

    b. В текстовое поле **Last name** (Фамилия) введите фамилию пользователя, например **Simon**.

    c. В текстовое поле **Email** (Электронная почта) введите электронную почту пользователя, например **brittasimon@contoso.com** .

    d. Кроме того, вы можете ввести персональное сообщение в поле **Отправить уведомление по электронной почте** . Если уведомление отправлять не нужно, снимите этот флажок.

    д) Щелкните **Создать пользователей**.

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку Small Improvements на Панели доступа, вы автоматически войдете в приложение Small Improvements, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)