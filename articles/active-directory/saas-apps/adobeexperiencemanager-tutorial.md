---
title: Руководство по интеграции Azure Active Directory с Adobe Experience Manager | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в приложении Adobe Experience Manager.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/17/2021
ms.author: jeedes
ms.openlocfilehash: 6cb490408cd66d5747156ef48ea9b4b2daa92abf
ms.sourcegitcommit: 24f30b1e8bb797e1609b1c8300871d2391a59ac2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/10/2021
ms.locfileid: "100094724"
---
# <a name="tutorial-azure-active-directory-integration-with-adobe-experience-manager"></a>Руководство. Интеграция Azure Active Directory с Adobe Experience Manager

В этом руководстве описано, как интегрировать Adobe Experience Manager с Azure Active Directory (Azure AD). Интеграция Adobe Experience Manager с Azure AD обеспечивает следующие возможности:

* Контроль доступа к Adobe Experience Manager с помощью Azure AD.
* Автоматический вход пользователей в Adobe Experience Manager с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Adobe Experience Manager с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Adobe Experience Manager поддерживает единый вход инициированного **пакета обновления и выдающей точки распространения**.

* Adobe Experience Manager поддерживает **JIT**-подготовку пользователей.

## <a name="adding-adobe-experience-manager-from-the-gallery"></a>Добавление Adobe Experience Manager из галереи

Чтобы настроить интеграцию Adobe Experience Manager в Azure AD, необходимо добавить Adobe Experience Manager из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Adobe Experience Manager**.
1. Выберите **Adobe Experience Manager** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.


## <a name="configure-and-test-azure-ad-sso-for-adobe-experience-manager"></a>Настройка и проверка единого входа Azure AD для Adobe Experience Manager

Настройте и проверьте единый вход Azure AD в Adobe Experience Manager с помощью тестового пользователя **B.Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Adobe Experience Manager.

Чтобы настроить и проверить единый вход Azure AD в Adobe Experience Manager, выполните следующие действия:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
2. **[Настройка единого входа в Adobe Experience Manager](#configure-adobe-experience-manager-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя Adobe Experience Manager](#create-adobe-experience-manager-test-user)** требуется для того, чтобы в Adobe Experience Manager существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Adobe Experience Manager** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **Идентификатор** введите уникальное значение, которое определяется на сервере AEM.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<AEM Server Url>/saml_login`.

    > [!NOTE]
    > Значение URL-адреса ответа приведено для примера. Обновите значение URL-адреса ответа актуальным значением. Чтобы его получить, свяжитесь с [группой поддержки клиента Adobe Experience Manager Client](https://helpx.adobe.com/support/experience-manager.html). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    В текстовом поле **URL-адрес для входа** введите URL-адрес сервера Adobe Experience Manager.

6. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (Base64)** из предложенных вариантов, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

7. Скопируйте требуемый URL-адрес из раздела **Set up Adobe Experience Manager** (Настройка Adobe Experience Manager).

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

В этом разделе описано, как включить единый вход в Azure для пользователя B.Simon, предоставив этому пользователю доступ к Adobe Experience Manager.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. Из списка приложений выберите **Adobe Experience Manager**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-adobe-experience-manager-sso"></a>Настройка единого входа в Adobe Experience Manager

1. Откройте портал администрирования **Adobe Experience Manager** в другом окне браузера.

2. Выберите **Параметры** > **Безопасность** > **Пользователи**.

    ![Снимок экрана, на котором показана плитка "Users" (Пользователи) в Adobe Experience Manager](./media/adobe-experience-manager-tutorial/user-1.png)

3. Выберите **Администратора** или любого другого соответствующего пользователя.

    ![Снимок экрана, на котором выделен пользователь "Administrator" (Администратор)](./media/adobe-experience-manager-tutorial/tutorial-admin-6.png)

4. Выберите **Account settings** (Параметры учетной записи)  > **Manage TrustStore** (Управление TrustStore).

    ![Снимок экрана, на котором показан элемент "Manage TrustStore" (Управление TrustStore) в разделе "Account settings" (Параметры учетной записи)](./media/adobe-experience-manager-tutorial/manage-trust.png)

5. Щелкните **Select Certificate File** (Выбрать файл сертификата) в разделе **Add Certificate from CER file** (Добавление сертификата из CER-файла). Найдите и укажите путь к файлу сертификата, скачанному с портала Azure.

    ![Снимок экрана, на котором выбрана кнопка "Select Certificate File" (Выбрать файл сертификата)](./media/adobe-experience-manager-tutorial/user-2.png)

6. Сертификат добавляется в TrustStore. Обратите внимание на псевдоним сертификата.

    ![Снимок экрана, на котором показано, что сертификат добавлен в TrustStore](./media/adobe-experience-manager-tutorial/tutorial-admin-7.png)

7. На странице **Users** (Пользователи) выберите **authentication-service**.

    ![Снимок экрана, на котором показан элемент authentication-service](./media/adobe-experience-manager-tutorial/tutorial-admin-8.png)

8. Выберите **Параметры учетной записи** > **Create/Manage KeyStore** (Создание хранилища ключей/Управление хранилищем ключей). Создайте хранилище ключей, указав пароль.

    ![Снимок экрана, на котором выбран элемент "Manage KeyStore" (Управление хранилищем ключей)](./media/adobe-experience-manager-tutorial/tutorial-admin-9.png)

9. Вернитесь на страницу администрирования. Выберите **Settings** (Параметры)  > **Operations** (Операции)  > **Web Console** (Веб-консоль).

    ![Снимок экрана, на котором выбран элемент "Web Console" (Веб-консоль) в области "Operations" (Операции) в разделе "Settings" (Параметры)](./media/adobe-experience-manager-tutorial/tutorial-admin-1.png)

    Откроется страница конфигурации.

    ![Кнопка "Сохранить" в окне настройки единого входа](./media/adobe-experience-manager-tutorial/tutorial-admin-2.png)

10. Найдите **Adobe Granite SAML 2.0 Authentication Handler**. Щелкните значок **Add** (Добавить).

    ![Снимок экрана, на котором выбран элемент "Adobe Granite SAML 2.0 Authentication Handler" (Обработчик аутентификации Adobe Granite SAML 2.0)](./media/adobe-experience-manager-tutorial/tutorial-admin-3.png)

11. Выполните следующие действия на этой странице.

    ![Кнопка "Сохранить" в окне настройки единого входа](./media/adobe-experience-manager-tutorial/tutorial-admin-4.png)

    а. В текстовом поле **Path** (Путь) введите **/** .

    b. В поле **IDP URL** (URL-адрес IDP) вставьте значение **Login URL** (URL-адрес входа), скопированное на портале Azure.

    c. В поле **IDP Certificate Alias** (Псевдоним сертификата IdP) введите значение **Certificate Alias** (Псевдоним сертификата), которое вы добавили в TrustStore.

    d. В поле **Security Provided Entity ID** (Идентификатор сущности системы безопасности) введите уникальное значение **Azure Ad Identifier** (Идентификатор Azure AD), настроенное на портале Azure.

    д) В поле **Assertion Consumer Service URL** (URL-адрес службы обработчика утверждений) введите значение **URL-адрес ответа**, настроенное на портале Azure.

    е) В поле **Password of Key Store** (Пароль хранилища ключей) введите **пароль**, установленный в хранилище ключей.

    ж. В поле **User Attribute ID** (Идентификатор пользовательского атрибута) введите **идентификатор имени** или другой пользовательский идентификатор, который используется в вашем случае.

    h. Выберите **Autocreate CRX Users** (Автоматическое создание пользователей CRX).

    i. В поле **Logout URL** (URL-адрес выхода) вставьте значение **URL-адрес выхода**, скопированное на портале Azure.

    j. Щелкните **Сохранить**.

### <a name="create-adobe-experience-manager-test-user"></a>Создание тестового пользователя Adobe Experience Manager

В этом разделе описано, как создать пользователя Britta Simon в приложении Adobe Experience Manager. Если выбран параметр **Autocreate CRX Users** (Автоматическое создание пользователей CRX), то пользователи будут создаваться автоматически после успешной аутентификации.

Если вы хотите создать пользователей вручную, обратитесь к [группе поддержки Adobe Experience Manager](https://helpx.adobe.com/support/experience-manager.html) для добавления пользователей на платформу Adobe Experience Manager.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

#### <a name="sp-initiated"></a>Инициация поставщиком услуг:

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в Adobe Experience Manager, где можно инициировать поток входа.  

* Перейдите непосредственно по URL-адресу входа в Adobe Experience Manager и запустите оттуда поток входа.

#### <a name="idp-initiated"></a>Вход, инициированный поставщиком удостоверений

* На портале Azure выберите **Тестировать приложение**, и вы автоматически войдете в приложение Adobe Experience Manager, для которого настроили единый вход 

Вы можете также использовать портал "Мои приложения" корпорации Майкрософт для тестирования приложения в любом режиме. Щелкнув плитку Adobe Experience Manager на портале "Мои приложения", вы будете перенаправлены на страницу входа приложения для инициации потока входа (при настройке в режиме поставщика услуг) или автоматически войдете в приложение Adobe Experience Manager, для которого настроен единый вход (при настройке в режиме поставщика удостоверений). Дополнительные сведения о портале "Мои приложения" см. в [этой статье](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).


## <a name="next-steps"></a>Дальнейшие действия

После настройки Adobe Experience Manager вы сможете применить функцию управления сеансом, которая защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа в режиме реального времени. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).
