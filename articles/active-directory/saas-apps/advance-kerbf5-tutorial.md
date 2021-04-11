---
title: Руководство по интеграции единого входа Azure Active Directory с F5 | Документация Майкрософт
description: В этой статье описаны действия, которые нужно выполнить для интеграции F5 с Azure Active Directory (Azure AD).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/11/2019
ms.author: jeedes
ms.openlocfilehash: db8977e484e8d1f2cf4b30427d47ba45969f2147
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101654465"
---
# <a name="tutorial-azure-active-directory-ad-single-sign-on-sso-integration-with-f5"></a>Руководство по интеграции единого входа Azure Active Directory с F5

В этом учебнике описано, как интегрировать F5 с Azure Active Directory (Azure AD). Интеграция F5 с Azure AD обеспечивает следующие возможности:

* Контроль доступа к F5 с помощью Azure AD.
* Автоматический вход пользователей в F5 с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

Чтобы узнать больше об интеграции приложений SaaS с Azure AD, прочитайте статью [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка F5 с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

F5 поддерживает единый вход, инициированный **поставщиком службы и поставщиком удостоверений**.

Единый вход в F5 можно настроить тремя способами:

- [Настройка единого входа в F5 для расширенного приложения Kerberos](#configure-f5-single-sign-on-for-advanced-kerberos-application)

- [настройка единого входа в F5 для приложения на основе заголовка](headerf5-tutorial.md);

- [Настройка единого входа в F5 для приложения Kerberos](kerbf5-tutorial.md)

## <a name="adding-f5-from-the-gallery"></a>Добавление F5 из коллекции

Чтобы настроить интеграцию F5 с Azure AD, необходимо добавить F5 из коллекции в список управляемых приложений SaaS.

1. Войдите на [портал Azure](https://portal.azure.com) с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **F5**.
1. Выберите **F5** на панели результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-single-sign-on-for-f5"></a>Настройка и проверка единого входа Azure AD для F5

Настройте и проверьте единый вход с Azure AD в F5, используя тестового пользователя **B.Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в F5.

Чтобы настроить и проверить единый вход Azure AD в F5, выполните действия в следующих стандартных блоках:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в F5-SSO](#configure-f5-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя приложения F5](#create-f5-test-user)** требуется для того, чтобы в F5 существовал пользователь B.Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **F5** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `https://<YourCustomFQDN>.f5.com/`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<YourCustomFQDN>.f5.com/`.

1. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://<YourCustomFQDN>.f5.com/`.

    > [!NOTE]
    > Эти значения приведены для примера. Замените их фактическими значениями идентификатора, URL-адреса ответа и URL-адреса входа. Чтобы получить эти значения, обратитесь в [службу поддержки клиентов F5](https://support.f5.com/csp/knowledge-center/software/BIG-IP?module=BIG-IP%20APM45). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите элемент **XML метаданных федерации** и выберите **Скачать**, чтобы скачать сертификат и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

1. Требуемые URL-адреса можно скопировать из раздела **Настройка F5**.

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

В этом разделе описано, как включить единый вход в Azure для пользователя B.Simon, предоставив этому пользователю доступ к F5.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **F5**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.

   ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Ссылка "Добавить пользователя"](common/add-assign-user.png)

1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор роли** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-f5-sso"></a>Настройка единого входа в F5

- [настройка единого входа в F5 для приложения на основе заголовка](headerf5-tutorial.md);

- [Настройка единого входа в F5 для приложения Kerberos](kerbf5-tutorial.md)

### <a name="configure-f5-single-sign-on-for-advanced-kerberos-application"></a>Настройка единого входа в F5 для расширенного приложения Kerberos

1. Откройте новое окно веб-браузера, зайдите на сайт F5 (расширенный Kerberos) с правами администратора и сделайте следующее:

1. Необходимо импортировать в F5 (расширенный Kerberos) сертификат метаданных, который будет использоваться позже в процессе настройки. Выберите **System > Certificate Management > Traffic Certificate Management >> SSL Certificate List** (Система > Управление сертификатами > Управление сертификатами трафика > Список SSL-сертификатов). Щелкните **"Import"** ("Импортировать") в правом верхнем углу

    ![Снимок экрана, на котором выбрана кнопка "Import" (Импортировать) для импорта сертификата метаданных](./media/advance-kerbf5-tutorial/configure01.png)
 
1. Чтобы настроить SAML IDP, перейдите на страницу **Доступ > Федерация > Поставщик службы SAML > Создать > Из метаданных.**

    ![Снимок экрана, на котором показано, как создать поставщик удостоверений SAML из метаданных](./media/advance-kerbf5-tutorial/configure02.png)

    ![Снимок экрана, на котором показан экран "Create New SAML IdP Connector" (Создание соединителя поставщика удостоверений SAML)](./media/advance-kerbf5-tutorial/configure03.png)
 
    ![Настройка F5 (расширенный Kerberos)](./media/advance-kerbf5-tutorial/configure04.png)

    ![Снимок экрана, на котором показан экран "Single Sign On Service Settings" (Параметры службы единого входа) ](./media/advance-kerbf5-tutorial/configure05.png)
 
1. Укажите сертификат, отправленный на этапе 3

    ![Снимок экрана, на котором показан экран "Edit SAML IdP Connector" (Изменение соединителя поставщика удостоверений SAML)](./media/advance-kerbf5-tutorial/configure06.png)

    ![Снимок экрана, на котором показан экран "Single Logout Service Settings" (Параметры службы единого выхода)](./media/advance-kerbf5-tutorial/configure07.png)

 1. Чтобы настроить SAML IDP, перейдите на страницу **Доступ > Федерация > Поставщик службы SAML > Локальные службы SP > Создать**.

    ![Снимок экрана, на котором показана страница для создания локальной службы SP](./media/advance-kerbf5-tutorial/configure08.png)
 
1. Нажмите кнопку **ОК**.

1. Выберите конфигурацию SP и щелкните **Bind/UnBind IdP Connectors** (Привязать/отменить привязку соединителей IDP).

     ![Снимок экрана, на котором показан поставщик службы SAML](./media/advance-kerbf5-tutorial/configure09.png)
 
 
1. Щелкните **Добавить новую строку** и выберите **Внешний соединитель IDP**, созданный на предыдущем шаге.

    ![Снимок экрана, на котором выбрана кнопка "Add New Row" (Добавить новую строку)](./media/advance-kerbf5-tutorial/configure10.png)
 
1. Для настройки единого входа Kerberos откройте **Доступ > Единый вход > Kerberos**.

    >[!Note]
    >Вам потребуется создать и указать учетную запись делегирования Kerberos. См. раздел KCD (см. приложение для ссылок на переменные).

    •   Источник имени пользователя `session.saml.last.attr.name.http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`

    •   Источник области пользователя `session.logon.last.domain`

    ![Снимок экрана, на котором показана страница "Access" (Доступ) > "Single Sign On" (Единый вход)](./media/advance-kerbf5-tutorial/configure11.png)

1. Чтобы настроить профиль доступа, откройте **Доступ > Профиль или политики > Профиль доступа (для политик сеанса)** .

    ![Снимок экрана, на котором показана вкладка "Properties" (Свойства) в меню "Profiles/Policies" (Профили и политики)](./media/advance-kerbf5-tutorial/configure12.png)

    ![Снимок экрана, на котором показана вкладка "SSO/Auth Domains" (Домены единого входа и аутентификации)](./media/advance-kerbf5-tutorial/configure13.png)

    ![Снимок экрана, на котором показана вкладка "Access Policy" (Политика доступа)](./media/advance-kerbf5-tutorial/configure14.png)

    ![Снимок экрана, на котором показана вкладка "Properties" (Свойства) в разделе "Access Policy" (Политика доступа)](./media/advance-kerbf5-tutorial/configure15.png)

    ![Снимок экрана, на котором показаны свойства для назначения переменной](./media/advance-kerbf5-tutorial/configure16.png)
 
    * session.logon.last.usernameUPN   expr {[mcget {session.saml.last.identity}]}

    * session.ad.lastactualdomain  TEXT superdemo.live

    ![Снимок экрана, на котором показаны свойства запроса Active Directory](./media/advance-kerbf5-tutorial/configure17.png)

    * (userPrincipalName=%{session.logon.last.usernameUPN})

    ![Снимок экрана, на котором показана вкладка "Branch Rules" (Правила ветви) и правило "Check Account" (Проверка учетной записи)](./media/advance-kerbf5-tutorial/configure18.png)

    ![Снимок экрана, на котором показаны текстовые поля настраиваемой переменной и настраиваемого выражения](./media/advance-kerbf5-tutorial/configure19.png)

    * session.logon.last.username  expr { "[mcget {session.ad.last.attr.sAMAccountName}]" }

    ![Снимок экрана, на котором показаны значения в полях "SSO Token Name" (Имя токена единого входа) и "SSO Token Password" (Пароль токена единого входа)](./media/advance-kerbf5-tutorial/configure20.png)

    * mcget {session.logon.last.username}
    * mcget {session.logon.last.password}

1. Чтобы добавить новый узел, перейдите в раздел **Локальный трафик > Узлы > Список узлов > +** .

    ![Снимок экрана, на котором показан раздел "Local Traffic" (Локальный трафик) > "Nodes" (Узлы)](./media/advance-kerbf5-tutorial/configure21.png)
 
1. Чтобы создать новый пул, перейдите в раздел **Локальный трафик > Пулы > Список пулов > Создать**.

     ![Снимок экрана, на котором показан раздел "Local Traffic" (Локальный трафик) > "Pools" (Пулы)](./media/advance-kerbf5-tutorial/configure22.png)

 1. Чтобы создать новый виртуальный сервер, перейдите в раздел **Локальный трафик > виртуальные серверы > Список виртуальных серверов > +** .

    ![Снимок экрана, на котором показан раздел "Local Traffic" (Локальный трафик) > "Virtual Servers" (Виртуальные серверы)](./media/advance-kerbf5-tutorial/configure23.png)

1. Укажите профиль доступа, созданный на предыдущем шаге.

    ![Снимок экрана, на котором показано, где вы указали созданный профиль доступа](./media/advance-kerbf5-tutorial/configure24.png) 

### <a name="setting-up-kerberos-delegation"></a>Настройка делегирования Kerberos 

>[!Note]
>Дополнительные сведения см. [здесь](https://www.f5.com/pdf/deployment-guides/kerberos-constrained-delegation-dg.pdf).

* **Шаг 1. Создание учетной записи делегирования**

    * Пример
    ```
    Domain Name : superdemo.live
    Sam Account Name : big-ipuser

    New-ADUser -Name "APM Delegation Account" -UserPrincipalName host/big-ipuser.superdemo.live@superdemo.live -SamAccountName "big-ipuser" -PasswordNeverExpires $true -Enabled $true -AccountPassword (Read-Host -AsSecureString "Password!1234")
    ```

* **Шаг 2. Задание имени участника-службы (для учетной записи делегирования APM)**

    *  Пример
    ```
    setspn –A host/big-ipuser.superdemo.live big-ipuser
    ```

* **Шаг 3. Делегирование имени участника-службы (для учетной записи Службы приложений)**

    * Настройте подходящее делегирование для учетной записи делегирования F5.
    * В приведенном ниже примере для учетной записи делегирования APM настраивается ограниченное делегирование Kerberos для приложения FRP-App1.superdemo.live.

        ![Снимок экрана, на котором показана вкладка "APM Delegation Account Properties" (Свойства учетной записи делегирования APM) > "Delegation" (Делегирование)](./media/advance-kerbf5-tutorial/configure25.png)

1. Укажите сведения, указанные в приведенном выше справочном [документе](https://techdocs.f5.com/kb/en-us/products/big-ip_apm/manuals/product/apm-authentication-single-sign-on-11-5-0/2.html)

1. Приложение. Ниже представлено сопоставление переменных для SAML - F5 BIG-IP:

    ![Снимок экрана, на котором показана вкладка "Overview" (Обзор) > "Active Sessions" (Активные сеансы)](./media/advance-kerbf5-tutorial/configure26.png)

    ![Снимок экрана, на котором показаны переменные и сеансовые ключи](./media/advance-kerbf5-tutorial/configure27.png) 

1. Ниже приведен полный список атрибутов SAML по умолчанию. GivenName представляется с помощью следующей строки.
`session.saml.last.attr.name.http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`

| Сеанс | attribute |
| -- | -- |
| eb46b6b6.session.saml.last.assertionID | `<TENANT ID>` |
| eb46b6b6.session.saml.last.assertionIssueInstant  | `<ID>` |
| eb46b6b6.session.saml.last.assertionIssuer | `https://sts.windows.net/<TENANT ID>`/ |
| eb46b6b6.session.saml.last.attr.name.http:\//schemas.microsoft.com/claims/authnmethodsreferences | `http://schemas.microsoft.com/ws/2008/06/identity/authenticationmethod/password` |
| eb46b6b6.session.saml.last.attr.name.http:\//schemas.microsoft.com/identity/claims/displayname | user0 |
| eb46b6b6.session.saml.last.attr.name.http:\//schemas.microsoft.com/identity/claims/identityprovider | `https://sts.windows.net/<TENANT ID>/` |
| eb46b6b6.session.saml.last.attr.name.http:\//schemas.microsoft.com/identity/claims/objectidentifier | `<TENANT ID>` |
| eb46b6b6.session.saml.last.attr.name.http:\//schemas.microsoft.com/identity/claims/tenantid | `<TENANT ID>` |
| eb46b6b6.session.saml.last.attr.name.http:\//schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress | `user0@superdemo.live` |
| eb46b6b6.session.saml.last.attr.name.http:\//schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname | user0 |
| eb46b6b6.session.saml.last.attr.name.http:\//schemas.xmlsoap.org/ws/2005/05/identity/claims/name | `user0@superdemo.live` |
| eb46b6b6.session.saml.last.attr.name.http:\//schemas.xmlsoap.org/ws/2005/05/identity/claims/surname | 0 |
| eb46b6b6.session.saml.last.audience | `https://kerbapp.superdemo.live` |
| eb46b6b6.session.saml.last.authNContextClassRef | urn:oasis:names:tc:SAML:2.0:ac:classes:Password |
| eb46b6b6.session.saml.last.authNInstant | `<ID>` |
| eb46b6b6.session.saml.last.identity | `user0@superdemo.live` |
| eb46b6b6.session.saml.last.inResponseTo | `<TENANT ID>` |
| eb46b6b6.session.saml.last.nameIDValue | `user0@superdemo.live` |
| eb46b6b6.session.saml.last.nameIdFormat | urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress |
| eb46b6b6.session.saml.last.responseDestination | `https://kerbapp.superdemo.live/saml/sp/profile/post/acs` |
| eb46b6b6.session.saml.last.responseId | `<TENANT ID>` |
| eb46b6b6.session.saml.last.responseIssueInstant | `<ID>` |
| eb46b6b6.session.saml.last.responseIssuer | `https://sts.windows.net/<TENANT ID>/` |
| eb46b6b6.session.saml.last.result | 1 |
| eb46b6b6.session.saml.last.samlVersion | 2.0 |
| eb46b6b6.session.saml.last.sessionIndex | `<TENANT ID>` |
| eb46b6b6.session.saml.last.statusValue | urn:oasis:names:tc:SAML:2.0:status:Success |
| eb46b6b6.session.saml.last.subjectConfirmDataNotOnOrAfter | `<ID>` |
| eb46b6b6.session.saml.last.subjectConfirmDataRecipient | `https://kerbapp.superdemo.live/saml/sp/profile/post/acs` |
| eb46b6b6.session.saml.last.subjectConfirmMethod | urn:oasis:names:tc:SAML:2.0:cm:bearer |
| eb46b6b6.session.saml.last.validityNotBefore | `<ID>` |
| eb46b6b6.session.saml.last.validityNotOnOrAfter | `<ID>` |

### <a name="create-f5-test-user"></a>Создание тестового пользователя в F5

В этом разделе вы узнаете, как создать пользователя B.Simon в приложении F5. Чтобы добавить пользователей на платформу F5, обратитесь в [службу поддержки клиентов F5](https://support.f5.com/csp/knowledge-center/software/BIG-IP?module=BIG-IP%20APM45). Перед использованием единого входа необходимо создать и активировать пользователей. 

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку F5 на Панели доступа, вы автоматически войдете в F5, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)

- [Попробуйте использовать F5 с Azure AD](https://aad.portal.azure.com/)

- [настройка единого входа в F5 для приложения на основе заголовка](headerf5-tutorial.md);

- [Настройка единого входа в F5 для приложения Kerberos](kerbf5-tutorial.md)

- [Интеграция F5 BIG-IP APM и Azure AD для обеспечения безопасного гибридного доступа](../manage-apps/f5-aad-integration.md)

- [Руководство по развертыванию виртуальной машины F5 BIG-IP Virtual Edition в Azure IaaS для обеспечения безопасного гибридного доступа](../manage-apps/f5-bigip-deployment-guide.md)

- [Руководство по интеграции единого входа Azure Active Directory с F5 BIG-IP для обеспечения VPN-подключения без использования пароля](../manage-apps/f5-aad-password-less-vpn.md)