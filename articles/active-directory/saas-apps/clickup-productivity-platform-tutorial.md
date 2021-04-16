---
title: Руководство по Интеграция Azure Active Directory с ClickUp Productivity Platform | Документация Майкрософт
description: В этой статье объясняется, как настроить единый вход между Azure Active Directory и ClickUp Productivity Platform.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/30/2021
ms.author: jeedes
ms.openlocfilehash: 29bc02466825aa2ea2c1a9ece2826b1262293392
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106285218"
---
# <a name="tutorial-azure-active-directory-integration-with-clickup-productivity-platform"></a>Руководство по Интеграция Azure Active Directory с ClickUp Productivity Platform

В этом руководстве описано, как интегрировать ClickUp Productivity Platform с Azure Active Directory (Azure AD). Интеграция ClickUp Productivity Platform с Azure AD обеспечивает следующие возможности:

* Контроль доступа к ClickUp Productivity Platform с помощью Azure AD.
* Автоматический вход пользователей в ClickUp Productivity Platform с учетными записями Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* подписка ClickUp Productivity Platform с поддержкой единого входа (SSO).

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* ClickUp Productivity Platform поддерживает единый вход, инициированный **поставщиком услуг**.

## <a name="add-clickup-productivity-platform-from-the-gallery"></a>Добавление ClickUp Productivity Platform из коллекции

Чтобы настроить интеграцию ClickUp Productivity Platform с Azure AD, необходимо добавить ClickUp Productivity Platform из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **ClickUp Productivity Platform**.
1. Выберите **ClickUp Productivity Platform** в области результатов, а затем добавьте приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-clickup-productivity-platform"></a>Настройка и проверка единого входа Azure AD для ClickUp Productivity Platform

Настройте и проверьте единый вход Azure AD в ClickUp Productivity Platform с помощью тестового пользователя **B.Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в ClickUp Productivity Platform.

Чтобы настроить и проверить единый вход Azure AD в ClickUp Productivity Platform, выполните следующие действия:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в ClickUp Productivity Platform](#configure-clickup-productivity-platform-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя ClickUp Productivity Platform](#create-clickup-productivity-platform-test-user)** необходимо, чтобы в ClickUp Productivity Platform существовал пользователь B.Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **ClickUp Productivity Platform** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес: `https://app.clickup.com/login/sso`

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `https://api.clickup.com/v1/team/<team_id>/microsoft`.

    > [!NOTE]
    > Значение идентификатора приведено для примера и не является реальным. Замените его на фактический идентификатор, как описано далее в этом руководстве.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** нажмите кнопку "Копировать", чтобы копировать **URL-адрес метаданных федерации приложений** и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/copy-metadataurl.png)

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD

В этом разделе описано, как на портале Azure создать тестового пользователя с именем B.Simon.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.
1. В верхней части экрана выберите **Новый пользователь**.
1. В разделе **Свойства пользователя** выполните следующие действия.
   1. В поле **Имя** введите `B.Simon`.  
   1. В поле **Имя пользователя** введите username@companydomain.extension. Например, `B.Simon@contoso.com`.
   1. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле **Пароль**.
   1. Нажмите кнопку **Создать**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как включить единый вход в Azure для пользователя B.Simon, предоставив этому пользователю доступ к ClickUp Productivity Platform.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **ClickUp Productivity Platform**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-clickup-productivity-platform-sso"></a>Настройка единого входа в ClickUp Productivity Platform

1. В другом окне веб-браузера войдите в свой клиент ClickUp Productivity Platform с правами администратора.

2. Щелкните значок **профиля пользователя** и выберите пункт **Settings** (Параметры).

    ![Снимок экрана, на котором показан арендатор ClickUp Productivity с выбранным значком "Settings" (Параметры)](./media/clickup-productivity-platform-tutorial/configure-0.png)

    ![Снимок экрана, на котором показан раздел "Settings" (Параметры)](./media/clickup-productivity-platform-tutorial/configure-1.png)

3. Выберите **Microsoft** (Майкрософт) в разделе "Single Sign-On (SSO) Provider" (Поставщик единого входа).

    ![Снимок экрана, на котором показана область "Authentication" (Аутентификация) с выбранным элементом "Microsoft" (Майкрософт)](./media/clickup-productivity-platform-tutorial/configure-2.png)

4. На странице **Configure Microsoft Single Sign On** (Настройка единого входа Майкрософт) выполните следующие действия:

    ![Снимок экрана, на котором показана страница "Configure Microsoft Single Sign On" (Настройка единого входа Майкрософт), где можно скопировать идентификатор сущности и сохранить URL-адрес метаданных федерации Azure](./media/clickup-productivity-platform-tutorial/configure-3.png)

    а. Щелкните **Copy** (Копировать), чтобы скопировать идентификатор сущности, и вставьте его в текстовое поле **Идентификатор (сущности)** в разделе **Базовая конфигурация SAML** на портале Azure.

    b. В текстовое поле **Azure Federation Metadata URL** (URL-адрес метаданных федерации приложения Azure) вставьте URL-адрес метаданных федерации приложения, скопированный на портале Azure, и щелкните **Save** (Сохранить).

5. Чтобы завершить настройку, щелкните **Authenticate With Microsoft to complete setup** (Выполнить аутентификацию с помощью средств Майкрософт, чтобы завершить настройку) и выполните аутентификацию с помощью учетной записи Майкрософт.

    ![Снимок экрана, на котором показана кнопка "Authenticate With Microsoft to complete setup" (Выполнить аутентификацию с помощью средств Майкрософт, чтобы завершить настройку)](./media/clickup-productivity-platform-tutorial/configure-4.png)


### <a name="create-clickup-productivity-platform-test-user"></a>Создание тестового пользователя ClickUp Productivity Platform

1. В другом окне веб-браузера войдите в свой клиент ClickUp Productivity Platform с правами администратора.

2. Щелкните значок **профиля пользователя** и выберите пункт **People** (Пользователи).

    ![Снимок экрана, на котором показан арендатор ClickUp Productivity](./media/clickup-productivity-platform-tutorial/configure-0.png)

    ![Снимок экрана, на котором показана выбранная ссылка "People" (Люди)](./media/clickup-productivity-platform-tutorial/user-1.png)

3. Введите в текстовом поле адрес электронной почты пользователя и щелкните **Invite** (Пригласить).

    ![Снимок экрана, на котором показан раздел "Team Users Settings" (Параметры участников команды), позволяющий пригласить людей по электронной почте](./media/clickup-productivity-platform-tutorial/user-2.png)

    > [!NOTE]
    > Пользователь получит уведомление, и ему потребуется принять приглашение, чтобы активировать учетную запись.

## <a name="test-sso"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в ClickUp Productivity Platform, где можно инициировать поток входа. 

* Вручную введите URL-адрес для входа в ClickUp Productivity Platform и инициируйте поток входа.

* Вы можете использовать портал "Мои приложения" корпорации Майкрософт. Щелкнув элемент ClickUp Productivity Platform в разделе "Мои приложения", вы перейдете по URL-адресу единого входа для ClickUp Productivity Platform. Дополнительные сведения о портале "Мои приложения" см. в [этой статье](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="next-steps"></a>Дальнейшие действия

После настройки ClickUp Productivity Platform вы сможете применить функцию управления сеансами, чтобы в реальном времени защищать конфиденциальные данные своей организации от мошенничества и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).