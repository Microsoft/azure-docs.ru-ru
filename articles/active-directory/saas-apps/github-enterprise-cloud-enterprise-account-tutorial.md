---
title: Руководство по интеграции единого входа Azure Active Directory с GitHub Enterprise Cloud — учетная запись Enterprise | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в GitHub Enterprise Cloud — учетная запись Enterprise.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/12/2021
ms.author: jeedes
ms.openlocfilehash: 7360ad5abc7342043152c2da11038b624b0bfadc
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101649998"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-github-enterprise-cloud---enterprise-account"></a>Руководство по интеграции единого входа Azure Active Directory с GitHub Enterprise Cloud — учетная запись Enterprise

В этом руководстве описано, как интегрировать GitHub Enterprise Cloud — учетная запись Enterprise с Azure Active Directory (Azure AD). При интеграции GitHub Enterprise Cloud — учетная запись Enterprise с Azure AD вы получаете следующие возможности:

* Управление доступом к учетной записи GitHub Enterprise в Azure AD, а также любыми организациями в учетной записи Enterprise.


## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Учетная запись [GitHub Enterprise](https://docs.github.com/en/free-pro-team@latest/github/setting-up-and-managing-your-enterprise/about-enterprise-accounts).
* Учетная запись пользователя GitHub с правами владельца учетной записи Enterprise. 

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* GitHub Enterprise Cloud - Enterprise Account поддерживает единый вход, инициированный **поставщиком служб** или **поставщиком удостоверений**.
* GitHub Enterprise Cloud - Enterprise Account поддерживает **JIT**-подготовку пользователей.

## <a name="adding-github-enterprise-cloud---enterprise-account-from-the-gallery"></a>Добавление учетной записи GitHub Enterprise Cloud — учетная запись Enterprise из коллекции

Чтобы настроить интеграцию GitHub Enterprise Cloud — учетная запись Enterprise с Azure AD, необходимо добавить GitHub Enterprise Cloud — учетная запись Enterprise из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **GitHub Enterprise Cloud (учетная запись Enterprise)** .
1. Выберите **GitHub Enterprise Cloud — учетная запись Enterprise** на панели результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.


## <a name="configure-and-test-azure-ad-sso-for-github-enterprise-cloud---enterprise-account"></a>Настройка и проверка единого входа Azure AD для GitHub Enterprise Cloud — учетная запись Enterprise

Настройте и проверьте единый вход Azure AD в GitHub Enterprise Cloud — учетная запись Enterprise, используя тестового пользователя с именем **B. Simon**. Чтобы обеспечить работу единого входа, необходимо создать связь между пользователем в Azure AD и соответствующим пользователем в GitHub Enterprise Cloud — учетная запись Enterprise.

Чтобы настроить и проверить единый вход Azure AD в GitHub Enterprise Cloud - Enterprise Account, выполните действия в следующих стандартных блоках:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение пользователя Azure AD и учетной записи тестового пользователя приложению GitHub](#assign-your-azure-ad-user-and-the-test-user-account-to-the-github-app)** позволяет разрешить единый вход Azure AD для вашей учетной записи пользователя и тестового пользователя `B.Simon`.
1. **[Включение и тестирование SAML для учетной записи Enterprise и связанных с ней организаций](#enable-and-test-saml-for-the-enterprise-account-and-its-organizations)** позволяет настроить параметры единого входа на стороне приложения.
    1. **[Тестирование единого входа от имени другого владельца учетной записи Enterprise или с помощью учетной записи члена организации](#test-sso-with-another-enterprise-account-owner-or-organization-member-account)** позволяет убедиться в правильности настройки.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **GitHub Enterprise Cloud - Enterprise Account** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `https://github.com/enterprises/<ENTERPRISE-SLUG>`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://github.com/enterprises/<ENTERPRISE-SLUG>/saml/consume`.

1. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

     В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://github.com/enterprises/<ENTERPRISE-SLUG>/sso`

    > [!NOTE]
    > Замените `<ENTERPRISE-SLUG>` фактическим именем вашей учетной записи GitHub Enterprise.

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите пункт **Сертификат (Base64)** и щелкните **Скачать**, чтобы скачать сертификат. Сохраните этот сертификат на компьютере.

    ![Ссылка для скачивания сертификата](common/certificateBase64.png)

1. Скопируйте требуемый URL-адрес в разделе **Настройка GitHub Enterprise Cloud — учетная запись Enterprise**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD

С помощью инструкций из этого раздела вы создадите на портале Azure тестового пользователя `B.Simon`.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.
1. В верхней части экрана выберите **Новый пользователь**.
1. В разделе **Свойства пользователя** выполните следующие действия.
   1. В поле **Имя** введите `B.Simon`.  
   1. В поле **Имя пользователя** введите username@companydomain.extension. Например, `B.Simon@contoso.com`.
   1. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле **Пароль**.
   1. Нажмите кнопку **Создать**.

<a name="assign-the-azure-ad-test-user"></a>

### <a name="assign-your-azure-ad-user-and-the-test-user-account-to-the-github-app"></a>Назначение пользователя Azure AD и учетной записи тестового пользователя приложению GitHub

В этом разделе описано, как предоставить пользователю `B.Simon` и вашей учетной записи пользователя доступ к учетной записи Enterprise в GitHub Enterprise Cloud для использования единого входа Azure.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **GitHub Enterprise Cloud — учетная запись Enterprise**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** и свою учетную запись пользователя в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="enable-and-test-saml-for-the-enterprise-account-and-its-organizations"></a>Включение и проверка SAML для учетной записи Enterprise и связанных с ней организаций

Чтобы настроить единый вход для **учетной записи Enterprise в GitHub Enterprise Cloud**, выполните инструкции, приведенные в [документации по GitHub](https://docs.github.com/en/free-pro-team@latest/github/setting-up-and-managing-your-enterprise/enforcing-security-settings-in-your-enterprise-account#enabling-saml-single-sign-on-for-organizations-in-your-enterprise-account). 
1. Войдите на сайт GitHub.com с учетной записью пользователя с правами [владельца учетной записи Enterprise](https://docs.github.com/en/free-pro-team@latest/github/setting-up-and-managing-your-enterprise/roles-in-an-enterprise#enterprise-owner). 
1. Скопируйте значение из поля `Login URL` в приложении на портале Azure и вставьте его в поле `Sign on URL` в параметрах SAML учетной записи GitHub Enterprise. 
1. Скопируйте значение из поля `Azure AD Identifier` в приложении на портале Azure и вставьте его в поле `Issuer` в параметрах SAML учетной записи GitHub Enterprise. 
1. Скопируйте содержимое файла **сертификата (Base64)** , который вы скачали ранее на портале Azure, и вставьте его в соответствующее поле в параметрах SAML учетной записи GitHub Enterprise. 
1. Щелкните `Test SAML configuration` и убедитесь, что вы можете успешно выполнить аутентификацию в Azure AD из учетной записи GitHub Enterprise.
1. Если тестирование было успешным, сохраните параметры. 
1. После первой аутентификации с помощью SAML из учетной записи GitHub Enterprise в ней будет создан _связанный внешний идентификатор_, который связывает учетную запись пользователя GitHub, выполнившего вход, с учетной записью пользователя в Azure AD.  
 
После того как вы включите единый вход через SAML для своей учетной записи GitHub Enterprise, он будет включен по умолчанию для всех организаций, принадлежащих вашей учетной записи Enterprise. Все члены должны будут выполнять аутентификацию с использованием единого входа через SAML, чтобы получить доступ к своим организациям, а владельцы учетных записей Enterprise должны будут выполнять такую аутентификацию при входе в свою учетную запись.

<a name="test-sso"></a>

## <a name="test-sso-with-another-enterprise-account-owner-or-organization-member-account"></a>Проверка единого входа от имени другого владельца учетной записи Enterprise или с помощью учетной записью члена организации

После настройки интеграции SAML с учетной записью GitHub Enterprise (которая также действует для организаций GitHub в учетной записи Enterprise), другие владельцы учетных записей Enterprise, которые назначены приложению в Azure AD, смогут переходить по URL-адресу учетной записи GitHub Enterprise (`https://github.com/enterprises/<enterprise account>`), выполнять аутентификацию через SAML и получать доступ к политикам и параметрам в учетной записи GitHub Enterprise. 

Владелец организации в учетной записи Enterprise сможет [отправлять пользователям приглашения на присоединение к организации GitHub](https://docs.github.com/en/free-pro-team@latest/github/setting-up-and-managing-organizations-and-teams/inviting-users-to-join-your-organization). Войдите на сайт GitHub.com с учетной записью владельца организации и выполните приведенные в этой статье шаги, чтобы пригласить пользователя `B.Simon` в организацию. Для пользователя `B.Simon` необходимо будет создать учетную запись GitHub (если она еще не существует). 

Чтобы протестировать доступ к организации GitHub в учетной записи Enterprise с использованием учетной записи тестового пользователя `B.Simon`, сделайте следующее:
1. Пригласите `B.Simon` в организацию в учетной записи Enterprise от имени владельца организации. 
1. Войдите на сайт GitHub.com с учетной записью пользователя, которую вы хотите связать с учетной записью пользователя `B.Simon` в Azure AD.
1. Войдите в Azure AD с учетной записью пользователя `B.Simon`.
1. Перейдите к организации GitHub. Пользователю должен отобразиться запрос на аутентификацию через SAML. После успешной аутентификации через SAML пользователь `B.Simon` сможет получить доступ к ресурсам организации. 

## <a name="next-steps"></a>Дальнейшие действия

После настройки GitHub Enterprise Cloud — учетная запись Enterprise вы можете применить функцию управления сеансом, которая в реальном времени защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
