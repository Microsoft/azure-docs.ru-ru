---
title: Руководство по Интеграция единого входа Azure Active Directory с Pulse Secure Virtual Traffic Manager | Документация Майкрософт
description: Сведения о том, как настроить единый вход Azure Active Directory в Pulse Secure Virtual Traffic Manager.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/11/2020
ms.author: jeedes
ms.openlocfilehash: 1e295075a5c1ae8daa6673757770bbef01208c1d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92505751"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-pulse-secure-virtual-traffic-manager"></a>Руководство по Интеграция единого входа Azure Active Directory с Pulse Secure Virtual Traffic Manager

В этом учебнике описывается, как интегрировать Pulse Secure Virtual Traffic Manager с Azure Active Directory (Azure AD). Интеграция Pulse Secure Virtual Traffic Manager с Azure AD обеспечивает следующие возможности:

* Управление доступом к Pulse Secure Virtual Traffic Manager с помощью Azure AD.
* Автоматический вход пользователей в Pulse Secure Virtual Traffic Manager с учетными записями Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Pulse Secure Virtual Traffic Manager с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Pulse Secure Virtual Traffic Manager поддерживает единый вход, инициированный **поставщиком услуг**.

## <a name="adding-pulse-secure-virtual-traffic-manager-from-the-gallery"></a>Добавление Pulse Secure Virtual Traffic Manager из коллекции

Чтобы настроить интеграцию Pulse Secure Virtual Traffic Manager с Azure AD, необходимо добавить Pulse Secure Virtual Traffic Manager из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Pulse Secure Virtual Traffic Manager**.
1. Выберите **Pulse Secure Virtual Traffic Manager** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.


## <a name="configure-and-test-azure-ad-sso-for-pulse-secure-virtual-traffic-manager"></a>Настройка и проверка единого входа Azure AD для Pulse Secure Virtual Traffic Manager

Настройте и проверьте единый вход Azure AD в Pulse Secure Virtual Traffic Manager с помощью тестового пользователя **B.Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Pulse Secure Virtual Traffic Manager.

Чтобы настроить и проверить единый вход Azure AD в Pulse Secure Virtual Traffic Manager, выполните действия из следующих стандартных блоков:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Pulse Secure Virtual Traffic Manager](#configure-pulse-secure-virtual-traffic-manager-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя в Pulse Secure Virtual Traffic Manager](#create-pulse-secure-virtual-traffic-manager-test-user)** нужно для того, чтобы в Pulse Secure Virtual Traffic Manager существовал пользователь B.Simon, связанный с представлением этого пользователя в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Pulse Secure Virtual Traffic Manager** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<published virtual server FQDN>/saml/consume`.

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `https://<published virtual server FQDN>/saml/metadata`.

    c. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<published virtual server FQDN>/saml/consume`.

    > [!NOTE]
    > Эти значения приведены для примера. Необходимо обновить эти значения действующим URL-адресом для входа и идентификатором. Чтобы получить эти данные, обратитесь к [группе поддержки клиентов Pulse Secure Virtual Traffic Manager](mailto:support@pulsesecure.net). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите пункт **Сертификат (Base64)** и щелкните **Скачать**, чтобы скачать сертификат. Сохраните этот сертификат на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

1. Скопируйте требуемые URL-адреса из раздела **Настройка Pulse Secure Virtual Traffic Manager**.

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

В этом разделе описано, как включить единый вход в Azure для пользователя B.Simon, предоставив этому пользователю доступ к Pulse Secure Virtual Traffic Manager.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Pulse Secure Virtual Traffic Manager**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-pulse-secure-virtual-traffic-manager-sso"></a>Настройка единого входа в Pulse Secure Virtual Traffic Manager

В этом разделе описывается конфигурация, необходимая для включения проверки подлинности SAML в Azure AD для Pulse Virtual Traffic Manager. Все изменения конфигурации выполняются в Pulse Virtual Traffic Manager с помощью пользовательского веб-интерфейса администрирования. 

#### <a name="create-a-saml-trusted-identity-provider"></a>Создание доверенного поставщика удостоверений SAML

a. Перейдите на страницу **Pulse Virtual Traffic Manager Appliance Admin UI > Catalog > SAML > Trusted Identity Providers Catalog** (Пользовательский интерфейс администратора устройства Pulse Virtual Traffic Manager > Каталог > SAML > Каталог доверенных поставщиков удостоверений) и щелкните **Edit** (Изменить).

![страница каталогов SAML](./media/pulse-secure-virtual-traffic-manager-tutorial/saml-catalogs.png)

b. Добавьте сведения о новом доверенном поставщике удостоверений SAML. Для этого скопируйте данные со страницы параметров единого входа приложения Azure AD Enterprise, а затем щелкните **Create New Trusted Identity Provider** (Создать новый доверенный поставщик удостоверений).

![Создание доверенного поставщика удостоверений](./media/pulse-secure-virtual-traffic-manager-tutorial/create-trusted-identity-provider.png)

* В текстовом поле **Имя** введите имя доверенного поставщика удостоверений. 

* В текстовом поле **Entity_id** (Идентификатор сущности) введите значение **Идентификатор Azure AD**, скопированное на портале Azure.  

* В текстовое поле **URL** (URL-адрес) введите значение **URL-адреса входа**, скопированное на портале Azure. 

* Откройте в блокноте скачанный с портала Azure файл **сертификата** и вставьте его содержимое в текстовое поле **Certificate** (Сертификат).

c. Убедитесь, что новый поставщик удостоверений SAML успешно создан. 

![Проверка доверенного поставщика удостоверений](./media/pulse-secure-virtual-traffic-manager-tutorial/verify-saml-identity-provider.png)

#### <a name="configure-the-virtual-server-to-use-azure-ad-authentication"></a>Настройка виртуального сервера для аутентификации Azure AD

a. Перейдите на страницу **Pulse Virtual Traffic Manager Appliance Admin UI > Services > Virtual Servers** (Пользовательский интерфейс администратора устройства Pulse Virtual Traffic Manager > Службы > Виртуальные серверы) и щелкните **Edit** (Изменить) рядом с созданным ранее виртуальным сервером.

![Изменение виртуальных серверов](./media/pulse-secure-virtual-traffic-manager-tutorial/virtual-servers.png)

b. В разделе **Authentication** (Проверка подлинности) нажмите кнопку **Edit** (Изменить). 

![Раздел Authentication (Проверка подлинности)](./media/pulse-secure-virtual-traffic-manager-tutorial/authentication.png)

c. Настройте следующие параметры проверки подлинности для виртуального сервера: 

1. Проверка подлинности

    ![параметры проверки подлинности для виртуального сервера](./media/pulse-secure-virtual-traffic-manager-tutorial/authentication-1.png)

    a. В поле **Auth!type** выберите **SAML Service Provider** (Поставщик услуг SAML) 

    b. В поле **auth!verbose** выберите Yes (Да), если необходимо устранить все проблемы проверки подлинности. В противном случае оставьте значение по умолчанию No (Нет). 

2. Управление сеансами проверки подлинности

    ![Управление сеансами проверки подлинности](./media/pulse-secure-virtual-traffic-manager-tutorial/authentication-session.png)

    a. Для **Auth!session!cookie_name** оставьте значение по умолчанию "VS_SamlSP_Auth". 

    b. Для **auth!session!timeout** оставьте значение по умолчанию "7200". 

    c. В поле **auth!session!log_external_state** выберите Yes (Да), если необходимо устранить все проблемы проверки подлинности. В противном случае оставьте значение по умолчанию No (Нет). 

    d. Измените значение поля **auth!session!cookie_attributes** на "HTTPOnly". 

3. Поставщик услуг SAML

    ![Поставщик услуг SAML](./media/pulse-secure-virtual-traffic-manager-tutorial/saml-service-provider.png)

    a. В текстовом поле **auth!saml!sp_entity_id** задайте тот же URL-адрес, который использовался в качестве идентификатора конфигурации единого входа Azure AD (идентификатор сущности). Например, `https://pulseweb.labb.info/saml/metadata`. 

    b. В поле **auth!saml!sp_acs_url** задайте тот же URL-адрес, который использовался в качестве URL-адреса для воспроизведения конфигурации единого входа Azure AD (URL-адрес службы обработчика утверждений). Например, `https://pulseweb.labb.info/saml/consume`. 

    c. В разделе **auth!saml!idp** выберите **доверенный поставщик удостоверений**, созданный на предыдущем шаге. 

    d. В поле In the auth!saml!time_tolerance оставьте значение по умолчанию (5 секунд). 

    д) В поле auth!saml!nameid_format выберите **unspecified** (не задано).

    е) Примените изменения, нажав **Update** (Обновить) в нижней части страницы.
    
### <a name="create-pulse-secure-virtual-traffic-manager-test-user"></a>Создание тестового пользователя в Pulse Secure Virtual Traffic Manager

В этом разделе описано, как создать пользователя Britta Simon в приложении Virtual Traffic Manager. Обратитесь в [службу поддержки Pulse Secure Virtual Traffic Manager](mailto:support@pulsesecure.net), чтобы добавить пользователей на платформу Pulse Secure Virtual Traffic Manager. Перед использованием единого входа необходимо создать и активировать пользователей.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

1. Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены на URL-адрес входа в Pulse Secure Virtual Traffic Manager, где можно инициировать процесс входа. 

2. Напрямую откройте URL-адрес входа в Pulse Secure Virtual Traffic Manager и инициируйте процесс входа.

3. Вы можете использовать Панель доступа (Майкрософт). Щелкнув плитку Pulse Secure Virtual Traffic Manager на Панели доступа, вы перейдете по URL-адресу для входа в Pulse Secure Virtual Traffic Manager. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="next-steps"></a>Дальнейшие действия

После настройки Pulse Secure Virtual Traffic Manager вы можете применить функцию управления сеансами, которая в реальном времени защищает конфиденциальные данные вашей организации от кражи и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).