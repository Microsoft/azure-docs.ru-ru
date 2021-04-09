---
title: Руководство по интеграции единого входа Azure Active Directory с Adobe Identity Management | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Adobe Identity Management.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/15/2021
ms.author: jeedes
ms.openlocfilehash: fdca04c645e1bb956c8e9f294c702b639c8e2f74
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98726406"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-adobe-identity-management"></a>Руководство по интеграции единого входа Azure Active Directory с Adobe Identity Management

В этом учебнике описано, как интегрировать Adobe Identity Management с Azure Active Directory (Azure AD). Интеграция Adobe Identity Management с Azure AD предоставляет следующие возможности:

* Управление доступом к Adobe Identity Management с Azure AD.
* Автоматический вход пользователей в Adobe Identity Management с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Adobe Identity Management с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Adobe Identity Management поддерживает единый вход, инициированный **поставщиком услуг**.

## <a name="adding-adobe-identity-management-from-the-gallery"></a>Добавление Adobe Identity Management из коллекции

Чтобы настроить интеграцию Adobe Identity Management с Azure AD, необходимо добавить Adobe Identity Management из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** введите **Adobe Identity Management** в поле поиска.
1. Выберите **Adobe Identity Management** на панели результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-adobe-identity-management"></a>Настройка и проверка единого входа Azure AD для Adobe Identity Management

Настройте и проверьте единый вход Azure AD в Adobe Identity Management с помощью тестового пользователя **B.Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Adobe Identity Management.

Чтобы настроить и проверить единый вход Azure AD в Adobe Identity Management, выполните следующие действия:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Adobe Identity Management](#configure-adobe-identity-management-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя Adobe Identity Management](#create-adobe-identity-management-test-user)** требуется для того, чтобы в Adobe Identity Management существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Adobe Identity Management** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес: `https://adobe.com`

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `https://federatedid-na1.services.adobe.com/federated/saml/metadata/alias/<CUSTOM_ID>`.

    > [!NOTE]
    > Значение идентификатора приведено для примера и не является реальным. Вместо него нужно указать фактический идентификатор. Чтобы получить это значение, обратитесь в [службу поддержки клиентов Adobe Identity Management](mailto:identity@adobe.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите элемент **XML метаданных федерации** и выберите **Скачать**, чтобы скачать сертификат и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

1. Требуемые URL-адреса можно скопировать из раздела **Настройка Adobe Identity Management**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

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

В этом разделе описано, как включить единый вход в Azure для пользователя B.Simon, предоставив этому пользователю доступ к Adobe Identity Management.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Adobe Identity Management**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-adobe-identity-management-sso"></a>Настройка единого входа в Adobe Identity Management

1. Чтобы автоматизировать настройку в Adobe Identity Management, необходимо установить **расширение браузера для защищенного входа на страницу "Мои приложения"** , нажав кнопку **Установить расширение**.

    ![Расширение "Мои приложения"](common/install-myappssecure-extension.png)

2. Чтобы перейти к приложению Adobe Identity Management после добавления расширения в браузер, щелкните **Настройка Adobe Identity Management**. После этого укажите учетные данные администратора для входа в Adobe Identity Management. Расширение браузера автоматически настроит приложение и автоматизирует шаги 3–8.

    ![Настройка конфигурации](common/setup-sso.png)

3. Если вы хотите настроить Adobe Identity Management вручную, в другом окне браузера войдите на свой корпоративный сайт Adobe Identity Management с правами администратора.

4. Перейдите на вкладку **Settings** (Параметры) и нажмите кнопку **Create Directory** (Создать каталог).

    ![Параметры Adobe Identity Management](./media/adobe-identity-management-tutorial/settings.png)

5. Укажите имя каталога в текстовом поле, выберите **Federated ID** (Федеративный идентификатор) и щелкните **Next** (Далее).

    ![Создание каталога Adobe Identity Management](./media/adobe-identity-management-tutorial/create-directory.png)

6. Выберите **Other SAML Providers** (Другие поставщики SAML) и щелкните **Next** (Далее).
 
    ![Поставщики SAML в Adobe Identity Management](./media/adobe-identity-management-tutorial/saml-providers.png)

7. Щелкните ссылку **select** (Выбрать), чтобы передать **XML-файл метаданных**, скачанный на портале Azure.

    ![Настройка SAML в Adobe Identity Management](./media/adobe-identity-management-tutorial/saml-configuration.png)

8. Нажмите кнопку **Готово**.

### <a name="create-adobe-identity-management-test-user"></a>Создание тестового пользователя Adobe Identity Management

1. Перейдите на вкладку **Users** (Пользователи) и нажмите кнопку **Add User** (Добавить пользователя).

    ![Добавление пользователя в Adobe Identity Management](./media/adobe-identity-management-tutorial/add-user.png)

2. В текстовом поле **Enter user’s email address** (Введите адрес электронной почты пользователя) укажите **адрес электронной почты**.

    ![Сохранение данных пользователя в Adobe Identity Management](./media/adobe-identity-management-tutorial/save-user.png)

3. Выберите команду **Сохранить**.

## <a name="test-sso"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов.

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в Adobe Identity Management, где можно инициировать поток входа.

* Перейдите непосредственно по URL-адресу входа в Adobe Identity Management и запустите поток входа.

* Вы можете использовать портал "Мои приложения" корпорации Майкрософт. Щелкнув плитку Adobe Identity Management на портале "Мои приложения", вы перейдете по URL-адресу для входа в Adobe Identity Management. Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Дальнейшие действия

После настройки Adobe Identity Management вы можете применить управление сеансами, которое в реальном времени защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).