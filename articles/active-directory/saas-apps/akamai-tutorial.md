---
title: Руководство по интеграции единого входа Azure Active Directory с Akamai | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Akamai.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/04/2021
ms.author: jeedes
ms.openlocfilehash: a22a214104357b9ad99238a8db157839a1c9fd46
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104591951"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-akamai"></a>Руководство по интеграции единого входа Azure Active Directory с Akamai

В этом руководстве описано, как интегрировать Akamai с Azure Active Directory (Azure AD). Интеграция Akamai с Azure AD обеспечивает следующие возможности:

* Контроль доступа к Akamai с помощью Azure AD.
* Автоматический вход пользователей в Akamai с учетными записями Azure AD.
* Централизованное управление учетными записями через портал Azure.

Интеграция Azure Active Directory c Akamai Enterprise Application Access обеспечивает простой доступ к устаревшим приложениям, размещенным в облаке или локально. Для доступа к устаревшим приложениям без их изменения или установки агентов в интегрированном решении используются все преимущества современных возможностей Azure Active Directory, в том числе [условный доступ в Azure AD](../conditional-access/overview.md), [Защита идентификации Azure AD](../identity-protection/overview-identity-protection.md) и [Azure AD Identity Governance](../governance/identity-governance-overview.md).

На рисунке ниже показано место, которое Akamai ЕАА занимает в более обширном гибридном сценарии безопасного доступа.

![Место Akamai ЕАА в более обширном гибридном сценарии безопасного доступа](./media/header-akamai-tutorial/introduction-1.png)

### <a name="key-authentication-scenarios"></a>Сценарии проверки подлинности с использованием ключа

Помимо встроенной поддержки интеграции с Azure Active Directory для современных протоколов проверки подлинности, таких как Open ID Connect, SAML и WS-Fed, Akamai EAA обеспечивает для приложений с устаревшей проверкой подлинности безопасный внутренний и внешний доступ с помощью Azure AD, что позволяет использовать для них современные сценарии (например, доступ без пароля). В том числе:

* приложения с проверкой подлинности на основе заголовков;
* Удаленный рабочий стол
* SSH (Secure Shell);
* приложения с проверкой подлинности Kerberos;
* VNC (Virtual Network Computing);
* приложения с анонимной проверкой подлинности или без встроенной проверки подлинности;
* приложения с проверкой подлинности NTLM (защита с двойным подтверждением для пользователя);
* приложение на основе форм (защита с двойным подтверждением для пользователя).

### <a name="integration-scenarios"></a>Сценарии интеграции

Партнерство между корпорацией Майкрософт и компанией Akamai в отношении решения ЕАА обеспечивает гибкие возможности для соблюдения бизнес-требований благодаря поддержке нескольких сценариев интеграции в зависимости от потребностей организации. С их помощью можно предоставить недостающие компоненты для всех приложений, а также постепенно классифицировать их и настроить соответствующие классификации политик.

#### <a name="integration-scenario-1"></a>Сценарий интеграции 1

Akamai ЕАА настраивается в Azure AD как единое приложение. Администратор может настроить политику условного доступа для приложения, а при выполнении соответствующих условий пользователи могут получить доступ к порталу Akamai ЕАА.

**Преимущества**.

* Достаточно один раз настроить поставщик удостоверений.

**Недостатки**.

* У пользователей в итоге оказывается два портала приложений.

* Для всех приложений применяется одна общая политика условного доступа.

![Сценарий интеграции 1](./media/header-akamai-tutorial/scenario-1.png)

#### <a name="integration-scenario-2"></a>Сценарий интеграции 2

Приложение Akamai ЕАА настраивается отдельно на портале Azure AD. Администратор может настроить отдельные политики условного доступа для приложений, а при выполнении соответствующих условий пользователи могут напрямую перенаправляться к конкретному приложению.

**Преимущества**.

* Можно определить отдельные Политики ЦС.

* Все приложения представлены в меню запуска приложений O365 и на панели myApps.microsoft.com.


**Недостатки**.

* Необходимо настроить несколько поставщиков удостоверений.

![Сценарий интеграции 2](./media/header-akamai-tutorial/scenario-2.png)

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Akamai с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

- Akamai поддерживает единый вход, инициированный поставщиком удостоверений.

#### <a name="important"></a>Важно!

Все перечисленные ниже настройки одинаковы для сценариев интеграции **1** и **2**. Для **сценария интеграции 2** необходимо настроить отдельный поставщик удостоверений в Akamai ЕАА, а значение свойства "URL-адрес" необходимо изменить на URL-адрес приложения.

![Снимок экрана вкладки General (Общие) для AZURESSO-SP в Akamai Enterprise Application Access. На снимке выделено поле URL-адреса в разделе конфигурации проверки подлинности.](./media/header-akamai-tutorial/important.png)

## <a name="add-akamai-from-the-gallery"></a>Добавление Akamai из коллекции

Чтобы настроить интеграцию Akamai с Azure AD, необходимо добавить Akamai из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Akamai**.
1. Выберите **Akamai** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-akamai"></a>Настройка и проверка единого входа Azure AD для Akamai

Настройте и проверьте единый вход Azure AD в Akamai с помощью тестового пользователя **B.Simon**. Чтобы обеспечить работу единого входа, нужно установить связь между пользователем Azure AD и соответствующим пользователем в Akamai.

Чтобы настроить и проверить единый вход Azure AD в Akamai, выполните следующие действия:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    * **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    * **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Akamai](#configure-akamai-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    * **[Настройка поставщика удостоверений](#setting-up-idp)**
    * **[Проверка подлинности на основе заголовков](#header-based-authentication)**
    * **[Удаленный рабочий стол](#remote-desktop)**
    * **[SSH](#ssh)**
    * **[Проверка подлинности Kerberos](#kerberos-authentication)**
    * **[Создание тестового пользователя в Akamai](#create-akamai-test-user)** нужно для того, чтобы в Akamai существовал пользователь B.Simon, связанный с представлением этого же пользователя в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Akamai** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `https://<Yourapp>.login.go.akamai-access.com/saml/sp/response`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https:// <Yourapp>.login.go.akamai-access.com/saml/sp/response`.

    > [!NOTE]
    > Эти значения приведены для примера. Измените их на фактические значения идентификатора и URL-адреса ответа. Чтобы получить эти значения, обратитесь в [службу поддержки клиентов Akamai](https://www.akamai.com/us/en/contact-us/). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите элемент **XML метаданных федерации** и выберите **Скачать**, чтобы скачать сертификат и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

1. Требуемые URL-адреса можно скопировать в разделе **Настройка Akamai**.

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

В этом разделе описано, как включить единый вход в Azure для пользователя B.Simon, предоставив этому пользователю доступ к Akamai.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Akamai**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-akamai-sso"></a>Настройка единого входа Akamai

### <a name="setting-up-idp"></a>Настройка IDP

**Конфигурация поставщика удостоверений Akamai ЕАА**

1. Войдите в консоль **Akamai Enterprise Application Access**.
1. В **консоли Akamai ЕАА** выберите **Identity** > **Identity Providers** (Удостоверение > Поставщики удостоверений) и щелкните **Add Identity Provider** (Добавить поставщик удостоверений).

    ![Снимок экрана, на котором показано окно поставщиков удостоверений для консоли Akamai ЕАА. В меню Identity (Удостоверение) выберите пункт Identity Providers (Поставщики удостоверений) и нажмите Add Identity Provider (Добавить поставщик удостоверений).](./media/header-akamai-tutorial/configure-1.png)

1. В разделе **Create New Identity Provider** (Создание поставщика удостоверений) выполните следующие действия.

    ![Снимок экрана с диалоговым окном создания поставщиков удостоверений в консоли Akamai ЕАА.](./media/header-akamai-tutorial/configure-2.png)

    а. Укажите уникальное имя в **Name** (Имя).

    b. Выберите **Third Party SAML** (SAML стороннего производителя) и щелкните **Create Identity Provider and Configure** (Создать и настроить поставщик удостоверений).

### <a name="general-settings"></a>Общие параметры

1. **Identity Intercept** (Перехват идентификаторов) — укажите имя (базовый URL-адрес поставщика служб, который будет использоваться в конфигурации Azure AD).

    > [!NOTE]
    > Вы можете выбрать собственный пользовательский домен (для этого потребуется запись DNS и сертификат). В этом примере мы будем использовать домен Akamai.

1. **Akamai Cloud Zone** (Облачная зона Akamai) — выберите соответствующую облачную зону.
1. **Certificate Validation** (Проверка сертификата) — описано в документации по Akamai (необязательно).

    ![Снимок экрана консоли Akamai ЕАА с вкладкой General (Общие), на которой отображаются параметры для перехвата удостоверений, облачной зоны Akamai и проверки сертификата.](./media/header-akamai-tutorial/configure-3.png)

### <a name="authentication-configuration"></a>Настройка проверки подлинности

1. URL-адрес — укажите URL-адрес, который вы задали для перехвата идентификаторов (именно сюда направляются пользователи после аутентификации).
2. Logout URL (URL-адрес выхода): укажите URL-адрес выхода.
3. Sign SAML Request (Подписать запрос SAML): по умолчанию этот флажок снят.
4. Для файла метаданных IDP добавьте приложение в консоли Azure AD.

    ![Снимок экрана страницы конфигурации проверки подлинности консоли Akamai ЕАА, на которой отображаются параметры для URL-адреса, URL-адреса выхода, подписанного запроса SAML и файла метаданных IDP.](./media/header-akamai-tutorial/configure-4.png)

### <a name="session-settings"></a>Параметры сеанса

Оставьте параметры по умолчанию без изменений.

![Снимок экрана с диалоговым окном параметров сеанса в консоли Akamai ЕАА.](./media/header-akamai-tutorial/session-settings.png)

### <a name="directories"></a>Каталоги

Пропустите настройку каталога.

![Снимок экрана консоли Akamai ЕАА с вкладкой Directories (Каталоги).](./media/header-akamai-tutorial/directories.png)

### <a name="customization-ui"></a>Пользовательский интерфейс настройки

Поставщик удостоверений можно настроить.

![Снимок экрана консоли Akamai ЕАА с вкладкой настройки, на которой отображаются параметры для пользовательского интерфейса, языковых параметров и тем.](./media/header-akamai-tutorial/customization.png)

### <a name="advanced-settings"></a>Дополнительные параметры

Пропустите настройку дополнительных параметров. Дополнительные сведения см. в документации Akamai.

![Снимок экрана консоли Akamai ЕАА с вкладкой Advanced Settings (Дополнительные параметры), на которой показаны параметры для клиента ЕАА, расширенные параметры и параметры для моста между от OIDC к SAML.](./media/header-akamai-tutorial/advance-settings.png)

### <a name="deployment"></a>Развертывание

1. Щелкните Deploy Identity Provider (Развернуть поставщик удостоверений).

    ![Снимок экрана консоли Akamai ЕАА с вкладкой Deployment (Развертывание), на которой отображается кнопка Deploy Identity Provider (Развернуть поставщик удостоверений).](./media/header-akamai-tutorial/deployment.png)

2. Проверьте успешность развертывания.

### <a name="header-based-authentication"></a>Проверка подлинности на основе заголовков

Проверка подлинности Akamai на основе заголовков

1. Выберите вариант **Custom HTTP** (Настраиваемый HTTP) в мастере добавления приложений.

    ![Снимок экрана консоли Akamai ЕАА с мастером добавления приложений, в котором в разделе Access Apps (Доступ к приложениям) отображается CustomHTTP.](./media/header-akamai-tutorial/configure-5.png)

2. Заполните поля **Application Name** (Имя приложения) и **Description** (Описание).

    ![Снимок экрана диалогового окна Custom HTTP App (Пользовательское HTTP-приложение), в котором отображаются параметры для имени и описания приложения.](./media/header-akamai-tutorial/configure-6.png)

    ![Снимок экрана консоли Akamai ЕАА с вкладкой General (Общие), на которой отображаются общие параметры для MYHEADERAPP.](./media/header-akamai-tutorial/configure-7.png)

    ![Снимок экрана консоли Akamai ЕАА, в которой отображаются параметры для сертификата и расположения.](./media/header-akamai-tutorial/configure-8.png)

#### <a name="authentication"></a>Аутентификация

1. Перейдите на вкладку **Authentication** (Проверка подлинности).

    ![Снимок экрана консоли Akamai ЕАА с выбранной вкладкой Authentication (Проверка подлинности).](./media/header-akamai-tutorial/configure-9.png)

2. Назначьте **поставщик удостоверений**.

    ![Снимок экрана консоли Akamai ЕАА с вкладкой Authentication (Проверка подлинности) для MYHEADERAPP, на которой указано, что в качестве поставщика удостоверений используется служба единого входа Azure AD.](./media/header-akamai-tutorial/configure-10.png)

#### <a name="services"></a>Службы

Щелкните "Save and Go to Authentication" (Сохранить и перейти к проверке подлинности).

![Снимок экрана консоли Akamai ЕАА с вкладкой Services (Службы) для MYHEADERAPP, на которой в правом нижнем углу показана кнопка Save and go to Authentication (Сохранить и перейти к проверке подлинности).](./media/header-akamai-tutorial/configure-11.png)

#### <a name="advanced-settings"></a>Дополнительные параметры

1. В разделе **Customer HTTP Headers** (Настраиваемые заголовки HTTP) укажите значения **CustomerHeader** и **SAML Attribute**.

    ![Снимок экрана консоли Akamai ЕАА с вкладкой Advanced Settings (Дополнительные параметры), на которой выделено поле SSO Logged URL (Зарегистрированный URL-адрес для единого входа) в разделе Authentication (Проверка подлинности).](./media/header-akamai-tutorial/configure-12.png)

1. Нажмите кнопку **Save and go to Deployment** (Завершить и перейти к развертыванию).

    ![Снимок экрана консоли Akamai ЕАА с вкладкой Advanced Settings (Дополнительные параметры), на которой в правом нижнем углу показана кнопка Save and go to Deployment (Завершить и перейти к развертыванию).](./media/header-akamai-tutorial/configure-13.png)

#### <a name="deploy-the-application"></a>Развертывание приложения

1. Нажмите кнопку **Deploy Application** (Развернуть приложение).

    ![Снимок экрана консоли Akamai ЕАА с вкладкой Deployment (Развертывание), на которой отображается кнопка Deploy Application (Развернуть приложение).](./media/header-akamai-tutorial/configure-14.png)

1. Убедитесь, что приложение развернуто успешно.

    ![Снимок экрана консоли Akamai ЕАА с вкладкой Deployment (Развертывание), на которой отображается сообщение о состоянии приложения: Application Successfully Deployed (Приложение успешно развернуто).](./media/header-akamai-tutorial/configure-15.png)

1. Проверьте взаимодействие с пользователем.

    ![Снимок экрана, на котором показано открытие сайта myapps.microsoft.com с фоновым изображением и диалоговым окном входа.](./media/header-akamai-tutorial/end-user-1.png)

    ![Снимок экрана, на котором показана часть окна "Приложения" со значками для надстройки, HRWEB, Akamai — CorpApps, Expense, Groups и Access reviews. ](./media/header-akamai-tutorial/end-user-2.png)

1. Проверьте условный доступ.

    ![Снимок экрана с сообщением "Утверждение запроса на вход. На ваше мобильное устройство отправлено уведомление. Ответьте на него, чтобы продолжить работу".](./media/header-akamai-tutorial/conditional-access-1.png)

    ![Снимок экрана приложения со значком для MyHeaderApp.](./media/header-akamai-tutorial/conditional-access-2.png)

#### <a name="remote-desktop"></a>Удаленный рабочий стол

1. Выберите вариант **RDP** в мастере добавления приложений.

    ![Снимок экрана консоли Akamai ЕАА с мастером добавления приложений, в котором в разделе Access Apps (Доступ к приложениям) в списке приложений отображается RDP.](./media/header-akamai-tutorial/configure-16.png)

1. Заполните поля **Application Name** (Имя приложения) и **Description** (Описание).

    ![Снимок экрана диалогового окна RDP App (Приложение RDP), в котором отображаются параметры для имени и описания приложения.](./media/header-akamai-tutorial/configure-17.png)

    ![Снимок экрана консоли Akamai ЕАА с вкладкой General (Общие), на которой отображаются параметры удостоверения приложения для SECRETRDPAPP.](./media/header-akamai-tutorial/configure-18.png)

1. Укажите соединитель, который будет обслуживать это подключение.

    ![Снимок экрана консоли Akamai ЕАА, в которой отображаются параметры для сертификата и расположения. Для связанных соединителей задано значение USWST-CON1.](./media/header-akamai-tutorial/configure-19.png)

#### <a name="authentication"></a>Аутентификация

Щелкните **Save and Go to Services** (Сохранить и перейти к службам).

![Снимок экрана консоли Akamai ЕАА с вкладкой Authentication (Проверка подлинности) для SECRETRDPAPP, на которой в правом нижнем углу показана кнопка Save and go to Services (Сохранить и перейти к службам).](./media/header-akamai-tutorial/configure-20.png)

#### <a name="services"></a>Службы

Щелкните **Save and go to Advanced Settings** (Сохранить и перейти к дополнительным параметрам).

![Снимок экрана консоли Akamai ЕАА с вкладкой Services (Службы) для SECRETRDPAPP, на которой в правом нижнем углу показана кнопка Save and go to Authentication (Сохранить и перейти к проверке подлинности).](./media/header-akamai-tutorial/configure-21.png)

#### <a name="advanced-settings"></a>Дополнительные параметры

1. Щелкните **Save and go to Deployment** (Завершить и перейти к развертыванию).

    ![Снимок экрана консоли Akamai ЕАА с вкладкой Advanced Settings (Дополнительные параметры) для SECRETRDPAPP, на которой показаны параметры конфигурации удаленного рабочего стола.](./media/header-akamai-tutorial/configure-22.png)

    ![Снимок экрана консоли Akamai ЕАА с вкладкой Advanced Settings (Дополнительные параметры) для SECRETRDPAPP, на которой показаны параметры конфигурации проверки подлинности и работоспособности.](./media/header-akamai-tutorial/configure-23.png)

    ![Снимок экрана консоли Akamai ЕАА с вкладкой Custom HTTP headers (Пользовательские HTTP-заголовки) для SECRETRDPAPP, на которой в правом нижнем углу показана кнопка Save and go to Deployment (Завершить и перейти к развертыванию).](./media/header-akamai-tutorial/configure-24.png)

1. Проверьте взаимодействие с пользователем.

    ![Снимок экрана, на котором показано окно myapps.microsoft.com с фоновым изображением и диалоговым окном входа.](./media/header-akamai-tutorial/end-user-3.png)

    ![Снимок экрана, на котором показано окно "Приложения" на сайте myapps.microsoft.com со значками для надстройки, HRWEB, Akamai — CorpApps, Expense, Groups и Access reviews.](./media/header-akamai-tutorial/end-user-2.png)

1. Условный доступ

    ![Снимок экрана с сообщением об условном доступе: "Утверждение запроса на вход. На ваше мобильное устройство отправлено уведомление. Ответьте на него, чтобы продолжить работу".](./media/header-akamai-tutorial/conditional-access-4.png)

    ![Снимок экрана приложения со значком для MyHeaderApp и SecretRDPApp.](./media/header-akamai-tutorial/conditional-access-5.png)

    ![Снимок экрана Windows Server 2012 RS, на котором показаны общие значки пользователей. Значки для администратора, пользователя 0 и пользователя 1 указывают на то, что они вошли в систему.](./media/header-akamai-tutorial/conditional-access-6.png)

1. Кроме того, можно непосредственно ввести URL-адрес приложения для работы по протоколу RDP.

#### <a name="ssh"></a>SSH

1. Перейдите к разделу Add Applications (Добавление приложений) и выберите **SSH**.

    ![Снимок экрана консоли Akamai ЕАА с мастером добавления приложений, в котором в разделе Access Apps (Доступ к приложениям) в списке приложений отображается SSH.](./media/header-akamai-tutorial/configure-25.png)

1. Заполните поля **Application Name** (Имя приложения) и **Description** (Описание).

    ![Снимок экрана диалогового окна SSH App (Приложение RDP), в котором отображаются параметры для имени и описания приложения.](./media/header-akamai-tutorial/configure-26.png)

1. Настройте удостоверение приложения.

    ![Снимок экрана консоли Akamai ЕАА с вкладкой General (Общие), на которой отображаются параметры удостоверения приложения для SSH-SECURE.](./media/header-akamai-tutorial/configure-27.png)

    а. Укажите имя и описание.

    b. Укажите IP-адрес или полное доменное имя сервера приложений и порт для подключения SSH.

    c. Укажите имя пользователя и парольную фразу "Проверка Akamai ЕАА" для SSH.

    d. Укажите значение в поле "External host Name" (Имя внешнего узла).

    д) Укажите расположение соединителя и выберите нужный соединитель.

#### <a name="authentication"></a>Аутентификация

Щелкните **Save and Go to Services** (Сохранить и перейти к службам).

![Снимок экрана консоли Akamai ЕАА с вкладкой Authentication (Проверка подлинности) для SSH-SECURE, на которой в правом нижнем углу показана кнопка Save and go to Services (Сохранить и перейти к службам).](./media/header-akamai-tutorial/configure-28.png)

#### <a name="services"></a>Службы

Щелкните **Save and go to Advanced Settings** (Сохранить и перейти к дополнительным параметрам).

![Снимок экрана консоли Akamai ЕАА с вкладкой Services (Службы) для SSH-SECURE, на которой в правом нижнем углу показана кнопка Save and go to Authentication (Сохранить и перейти к проверке подлинности).](./media/header-akamai-tutorial/configure-29.png)

#### <a name="advanced-settings"></a>Дополнительные параметры

Щелкните "Save and go to Deployment" (Сохранить и перейти к развертыванию).

![Снимок экрана консоли Akamai ЕАА с вкладкой Advanced Settings (Дополнительные параметры) для SSH-SECURE, на которой показаны параметры конфигурации проверки подлинности и работоспособности.](./media/header-akamai-tutorial/configure-30.png)

![Снимок экрана консоли Akamai ЕАА с вкладкой Custom HTTP headers (Пользовательские HTTP-заголовки) для SSH-SECURE, на которой в правом нижнем углу показана кнопка Save and go to Deployment (Завершить и перейти к развертыванию).](./media/header-akamai-tutorial/configure-31.png)

#### <a name="deployment"></a>Развертывание

1. Щелкните **Deploy Application** (Развернуть приложение).

    ![Снимок экрана консоли Akamai ЕАА с вкладкой Deployment (Развертывание) для SSH-SECURE, на которой отображается кнопка Deploy Application (Развернуть приложение).](./media/header-akamai-tutorial/configure-32.png)

1. Проверьте взаимодействие с пользователем.

    ![Снимок экрана диалогового окна входа в окне myapps.microsoft.com.](./media/header-akamai-tutorial/end-user-3.png)

    ![Снимок экрана, на котором показано окно "Приложения" сайта myapps.microsoft.com со значками для надстройки, HRWEB, Akamai — CorpApps, Expense, Groups и Access reviews.](./media/header-akamai-tutorial/end-user-4.png)

1. Условный доступ

    ![Снимок экрана с сообщением "Утверждение запроса на вход. На ваше мобильное устройство отправлено уведомление. Ответьте на него, чтобы продолжить работу".](./media/header-akamai-tutorial/conditional-access-4.png)

    ![Снимок экрана приложения со значком для MyHeaderApp, SSH-Secure и SecretRDPApp.](./media/header-akamai-tutorial/conditional-access-7.png)

    ![Снимок экрана командного окна для ssh-secure-go.akamai-access.com, в котором показан запрос пароля.](./media/header-akamai-tutorial/conditional-access-8.png)

    ![Снимок экрана: командное окно для ssh-secure-go.akamai-access.com с информацией о приложении и командной строкой.](./media/header-akamai-tutorial/conditional-access-9.png)

### <a name="kerberos-authentication"></a>Проверка подлинности Kerberos

В приведенном ниже примере показано, как опубликовать внутренний веб-сервер <code>http://frp-app1.superdemo.live</code> и разрешить единый вход с применением ограниченного делегирования Kerberos.

#### <a name="general-tab"></a>Вкладка «Общие»

![Снимок экрана консоли Akamai ЕАА с вкладкой General (Общие) для MYKERBOROSAPP.](./media/header-akamai-tutorial/general-tab.png)

#### <a name="authentication-tab"></a>Вкладка "Authentication" (Аутентификация)

Назначьте поставщик удостоверений.

![Снимок экрана консоли Akamai ЕАА с вкладкой Authentication (Проверка подлинности) для MYKERBOROSAPP, на которой указано, что в качестве поставщика удостоверений используется служба единого входа Azure AD.](./media/header-akamai-tutorial/authentication-tab.png)

#### <a name="services-tab"></a>Вкладка Services (Службы)

![Снимок экрана консоли Akamai ЕАА с вкладкой Services (Службы) для MYKERBOROSAPP.](./media/header-akamai-tutorial/services-tab.png)

#### <a name="advanced-settings"></a>Дополнительные параметры

![Снимок экрана консоли Akamai ЕАА с вкладкой Advanced Settings (Дополнительные параметры) для MYKERBOROSAPP, на которой показаны параметры для связанных приложений и проверки подлинности.](./media/header-akamai-tutorial/advance-settings-2.png)

> [!NOTE]
> Имя субъекта-службы для веб-сервера должно быть задано в формате SPN@Domain. Например, в данном случае это `HTTP/frp-app1.superdemo.live@SUPERDEMO.LIVE`. Оставьте для остальных параметров значение по умолчанию.

#### <a name="deployment-tab"></a>Вкладка Deployment (Развертывание)

![Снимок экрана консоли Akamai ЕАА с вкладкой Deployment (Развертывание) для MYKERBOROSAPP, на которой отображается кнопка Deploy Application (Развернуть приложение).](./media/header-akamai-tutorial/deployment-tab.png)

#### <a name="adding-directory"></a>Добавление каталога

1. Выберите в раскрывающемся списке **AD**.

    ![Снимок экрана окна Directories (Каталоги) консоли Akamai ЕАА, в котором отображается диалоговое окно Create New Directory (Создание каталога) со службой AD, выбранной в раскрывающемся списке Directory Type (Тип каталога).](./media/header-akamai-tutorial/configure-33.png)

1. Укажите необходимые данные.

    ![Снимок экрана окна SUPERDEMOLIVE консоли Akamai ЕАА с параметрами для имени каталога, службы каталогов, соединителя и сопоставления атрибутов.](./media/header-akamai-tutorial/configure-34.png)

1. Проверьте создание каталога.

    ![Снимок экрана окна Directories (Каталоги) консоли Akamai ЕАА, в котором показано, что был добавлен каталог superdemo.live.](./media/header-akamai-tutorial/directory-domain.png)

1. Добавьте группы или подразделения, которым требуется доступ.

    ![Снимок экрана параметров для каталога superdemo.live. Выделен значок, выбранный для добавления групп или подразделений.](./media/header-akamai-tutorial/add-group.png)

1. На изображении ниже группа называется EAAGroup и состоит из одного члена.

    ![Снимок экрана окна GROUPS ON SUPERDEMOLIVE DIRECTORY (ГРУППЫ В КАТАЛОГЕ SUPERDEMOLIVE) консоли Akamai ЕАА. В списке Groups (Группы) указана EAAGroup с 1 пользователем.](./media/header-akamai-tutorial/eaagroup.png)

1. Добавьте каталог для поставщика удостоверений. Для этого выберите **Identity** > **Identity Providers** (Удостоверение > Поставщики удостоверений), а затем перейдите на вкладку **Directories** (Каталоги) и щелкните **Assign directory** (Назначить каталог).

    ![Снимок экрана консоли Akamai ЕАА с вкладкой Directories (Каталоги) для единого входа с помощью Azure AD, на которой в списке назначенных в данный момент каталогов отображается superdemo.live.](./media/header-akamai-tutorial/assign-directory.png)

### <a name="configure-kcd-delegation-for-eaa-walkthrough"></a>Пошаговое руководство по настройке ограниченного делегирования Kerberos для ЕАА

#### <a name="step-1-create-an-account"></a>Шаг 1. Создание учетной записи 

1. В этом примере мы будем использовать учетную запись с именем **EAADelegation**. Это можно сделать с помощью оснастки **Пользователи и компьютеры Active Directory**.

    ![Снимок экрана консоли Akamai ЕАА с вкладкой Directories (Каталоги) для единого входа с помощью Azure AD. В списке Currently assigned directories (Назначенные в данный момент каталоги) отображается каталог superdemo.live.](./media/header-akamai-tutorial/assign-directory.png)

    > [!NOTE]
    > Имя пользователя должно быть указано в определенном формате и составлено на основе имени, которое задано в поле **Identity Intercept** (Перехват идентификаторов). На рис. 1 показано **corpapps.login.go.akamai-access.com**.

1. Именем для входа пользователя будет `HTTP/corpapps.login.go.akamai-access.com`.

    ![Снимок экрана, на котором показаны свойства EAADelegation с именем EAADelegation и именем для входа пользователя HTTP/corpapps.login.go.akamai-access.com.](./media/header-akamai-tutorial/eaadelegation.png)

#### <a name="step-2-configure-the-spn-for-this-account"></a>Шаг 2. Настройка имени субъекта-службы для этой учетной записи

1. Для данного примера имя субъекта-службы будет следующим:

2. setspn -s **Http/corpapps.login.go.akamai-access.com eaadelegation**

    ![Снимок экрана командной строки администратора с результатами выполнения команды setspn -s Http/corpapps.login.go.akamai-access.com eaadelegation.](./media/header-akamai-tutorial/spn.png)

#### <a name="step-3-configure-delegation"></a>Шаг 3. Настройка делегирования

1. Для учетной записи EAADelegation перейдите на вкладку Delegation (Делегирование).

    ![Снимок экрана командной строки администратора с командой для настройки имени субъекта-службы.](./media/header-akamai-tutorial/spn.png)

    * Укажите, что можно использовать любой протокол аутентификации.
    * Нажмите Add (Добавить) и добавьте учетную запись пула приложений для веб-сайта Kerberos. Если настройки заданы правильно, она должна автоматически разрешаться в правильное имя субъекта-службы.

#### <a name="step-4-create-a-keytab-file-for-akamai-eaa"></a>Шаг 4. Создание файла keytab для Akamai ЕАА

1. Ниже приведен общий синтаксис.

1. ktpass /out ActiveDirectorydomain.keytab /princ `HTTP/yourloginportalurl@ADDomain.com` /mapuser serviceaccount@ADdomain.com /pass +rdnPass /crypto All /ptype KRB5_NT_PRINCIPAL

1. Пояснения к примеру

    | Фрагмент кода | Объяснение |
    | - | - |
    | Ktpass /out EAADemo.keytab | // Имя выходного файла keytab |
    | /princ HTTP/corpapps.login.go.akamai-access.com@superdemo.live | // HTTP/yourIDPName@YourdomainName |
    | /mapuser eaadelegation@superdemo.live | // Учетная запись делегирования EAA |
    | /pass RANDOMPASS | // Пароль учетной записи делегирования EAA |
    | /crypto All ptype KRB5_NT_PRINCIPAL | // См. в документации по Akamai ЕАА |
    | | |

1. Ktpass /out EAADemo.keytab /princ HTTP/corpapps.login.go.akamai-access.com@superdemo.live /mapuser eaadelegation@superdemo.live /pass RANDOMPASS /crypto All ptype KRB5_NT_PRINCIPAL

    ![Снимок экрана командной строки администратора с результатами выполнения команды для создания keytab для AKAMAI EAA.](./media/header-akamai-tutorial/administrator.png)

#### <a name="step-5-import-keytab-in-the-akamai-eaa-console"></a>Шаг 5. Импорт keytab в консоли Akamai ЕАА

1. Щелкните **System** > **Keytabs** (Система > Файлы keytab).

    ![Снимок экрана консоли Akamai ЕАА с меню System, в котором выбран пункт Keytabs (Файлы keytab).](./media/header-akamai-tutorial/keytabs.png)

1. В поле Keytab Type (Тип keytab) выберите **Kerberos Delegation** (Делегирование Kerberos).

    ![Снимок экрана EAAKEYTAB в консоли Akamai ЕАА, на котором показаны параметры файла keytab. Для Keytab Type (Тип keytab) задано значение Kerberos Delegation (Делегирование Kerberos).](./media/header-akamai-tutorial/keytab-delegation.png)

1. Убедитесь, что файл keytab отображается как развернутый и проверенный.

    ![Снимок экрана KEYTABS в консоли Akamai ЕАА, на котором для ЕАА keytab задано значение Keytab deployed and verified (Файл keytab развернут и проверен).](./media/header-akamai-tutorial/keytabs-2.png)

1. Действия пользователя

    ![Снимок экрана диалогового окна входа на сайте myapps.microsoft.com. ](./media/header-akamai-tutorial/end-user-3.png)

    ![Снимок экрана окна "Приложения" для myapps.microsoft.com, в котором отображаются значки приложений.](./media/header-akamai-tutorial/end-user-4.png)

1. Условный доступ

    ![Снимок экрана с сообщением "Утверждение запроса на вход". сообщение.](./media/header-akamai-tutorial/conditional-access-4.png)

    ![Снимок экрана приложения со значком для MyHeaderApp, SSH-Secure, SecretRDPApp и myKerberosApp.](./media/header-akamai-tutorial/conditional-access-10.png)

    ![Снимок экрана-заставки для myKerberosApp. Поверх фонового изображения отображается сообщение "Добро пожаловать, superdemo\user1".](./media/header-akamai-tutorial/conditional-access-11.png)

### <a name="create-akamai-test-user"></a>Создание тестового пользователя Akamai

Из этого разделе вы узнаете, как создать пользователя B.Simon в приложении Akamai. Обратитесь в [службу поддержки клиентов Akamai](https://www.akamai.com/us/en/contact-us/), чтобы добавить пользователей на платформу Akamai. Перед использованием единого входа необходимо создать и активировать пользователей. 

## <a name="test-sso"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов.

* Щелкните "Тестировать приложение" на портале Azure, и вы автоматически войдете в приложение Akamai, для которого настроен единый вход.

* Вы можете использовать портал "Мои приложения" корпорации Майкрософт. Щелкнув плитку Akamai на портале "Мои приложения", вы автоматически войдете в приложение Akamai, для которого настроили единый вход. Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Дальнейшие действия

После настройки Akamai вы можете применить функцию управления сеансами, которая в реальном времени защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).