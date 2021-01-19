---
title: Руководство по интеграции Azure Active Directory с Palo Alto Networks - Aperture | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Palo Alto Networks - Aperture.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/10/2020
ms.author: jeedes
ms.openlocfilehash: 7b4c8ab55a9b2afc9c1a215236fa1276a630a5cf
ms.sourcegitcommit: f6f928180504444470af713c32e7df667c17ac20
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/07/2021
ms.locfileid: "97963627"
---
# <a name="tutorial-azure-active-directory-integration-with-palo-alto-networks---aperture"></a>Руководство по интеграции Azure Active Directory с Palo Alto Networks - Aperture

В этом руководстве описано, как интегрировать Azure Active Directory (Azure AD) с Palo Alto Networks - Aperture.
Интеграция Azure AD с Palo Alto Networks - Aperture обеспечивает следующие преимущества.

* С помощью Azure AD вы можете контролировать доступ к Palo Alto Networks - Aperture.
* Вы можете включить автоматический вход пользователей (единый вход) в Palo Alto Networks — Aperture с использованием учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с Palo Alto Networks - Aperture, вам потребуется:

* Подписка Azure AD. (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка Palo Alto Networks — Aperture с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Palo Alto Networks — Aperture поддерживает единый вход, инициированный **поставщиком услуг** и **поставщиком удостоверений**.

## <a name="adding-palo-alto-networks---aperture-from-the-gallery"></a>Добавление Palo Alto Networks - Aperture из коллекции

Чтобы настроить интеграцию Palo Alto Networks - Aperture с Azure AD, необходимо добавить Palo Alto Networks - Aperture из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** введите в поле поиска **Palo Alto Networks — Aperture**.
1. Выберите на панели результатов **Palo Alto Networks — Aperture** и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso"></a>Настройка и проверка единого входа Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в Palo Alto Networks — Aperture с использованием тестового пользователя **B. Simon**.
Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Palo Alto Networks — Aperture.

Чтобы настроить и проверить единый вход Azure AD в Palo Alto Networks — Aperture, сделайте следующее.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    * **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
    * **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
2. **[Настройка единого входа в Palo Alto Networks — Aperture](#configure-palo-alto-networks---aperture-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    * **[Создание тестового пользователя Palo Alto Networks — Aperture](#create-palo-alto-networks---aperture-test-user)** требуется для того, чтобы в Palo Alto Networks — Aperture существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
3. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Palo Alto Networks — Aperture** найдите раздел **Управление** и щелкните **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** выполните следующие действия.

    ![Снимок экрана: раздел "Базовая конфигурация SAML" с выбранными полями "Идентификатор", "URL-адрес ответа" и кнопкой "Сохранить".](common/idp-intiated.png)

    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `https://<subdomain>.aperture.paloaltonetworks.com/d/users/saml/metadata`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<subdomain>.aperture.paloaltonetworks.com/d/users/saml/auth`.

5. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    ![Сведения о домене и URL-адресах единого входа для приложения Palo Alto Networks — Aperture для поставщика услуг](common/metadata-upload-additional-signon.png)

    В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://<subdomain>.aperture.paloaltonetworks.com/d/users/saml/sign_in`.

    > [!NOTE]
    > Эти значения приведены для примера. Замените их фактическими значениями идентификатора, URL-адреса ответа и URL-адреса входа. Чтобы получить эти значения, обратитесь к [группе поддержки Palo Alto Networks - Aperture](https://live.paloaltonetworks.com/t5/custom/page/page-id/Support). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

6. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (Base64)** из предложенных вариантов, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

7. Скопируйте требуемый URL-адрес из раздела **Настройка Palo Alto Networks — Aperture**.

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

В этом разделе описано, как предоставить пользователю B. Simon доступ к Palo Alto Networks — Aperture, чтобы он мог использовать единый вход Azure.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. Из списка приложений выберите **Palo Alto Networks - Aperture**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-palo-alto-networks---aperture-sso"></a>Настройка единого входа в Palo Alto Networks — Aperture

1. В другом окне веб-браузера войдите на веб-сайт Palo Alto Networks - Aperture в качестве администратора.

2. В верхней строке меню щелкните **SETTINGS** (Параметры).

    ![Вкладка "SETTINGS" (Параметры)](./media/paloaltonetworks-aperture-tutorial/tutorial_paloaltonetwork_settings.png)

3. Перейдите в раздел **APPLICATION** (Приложение) и щелкните форму **Authentication** (Аутентификация) в левой части меню.

    ![Вкладка "Authentication" (Аутентификация)](./media/paloaltonetworks-aperture-tutorial/tutorial_paloaltonetwork_auth.png)
    
4. На странице **Authentication** (Аутентификация) выполните следующие действия.
    
    ![Вкладка "Authentication" (Аутентификация)](./media/paloaltonetworks-aperture-tutorial/tutorial_paloaltonetwork_singlesignon.png)

    a. Установите флажок **Enable Single Sign-On (Supported SSP Providers are Okta, One login)** (Включить единый вход (поддерживаемые поставщики общих служб: Okta, One login)) в поле **Single Sign-On** (Единый вход).

    b. В текстовое поле **Identity Provider ID** (Идентификатор поставщика удостоверений) вставьте значение **идентификатора Azure AD**, скопированное на портале Azure.

    c. Щелкните **Choose File** (Выбор файла), чтобы передать сертификат, скачанный из Azure AD, и указать его в поле **Identity Provider Certificate** (Сертификат поставщика удостоверений).

    d. В текстовое поле **Identity provider SSO url** (URL-адрес единого входа поставщика удостоверений) вставьте **URL-адрес входа**, скопированный на портале Azure.

    д) Просмотрите сведения о поставщике удостоверений в разделе **Aperture Info** (Сведения об Aperture) и скачайте сертификат из поля **Aperture Key** (Ключ Aperture).

    е) Выберите команду **Сохранить**.


### <a name="create-palo-alto-networks---aperture-test-user"></a>Создание тестового пользователя в Palo Alto Networks — Aperture

В этом разделе описано, как создать пользователя Britta Simon в приложении Palo Alto Networks - Aperture. Обратитесь к [группе поддержки клиентов Palo Alto Networks - Aperture](https://live.paloaltonetworks.com/t5/custom/page/page-id/Support) для добавления пользователей на платформу Palo Alto Networks - Aperture. Перед использованием единого входа необходимо создать и активировать пользователей.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

#### <a name="sp-initiated"></a>Инициация поставщиком услуг:

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены на URL-адрес входа в Palo Alto Networks — Aperture, где можно инициировать процесс входа.  

* Напрямую откройте URL-адрес входа в Palo Alto Networks — Aperture и инициируйте процесс входа.

#### <a name="idp-initiated"></a>Вход, инициированный поставщиком удостоверений

* На портале Azure выберите **Тестировать приложение**, и вы автоматически войдете в приложение Palo Alto Networks — Aperture, для которого настроен единый вход. 

Вы можете также использовать портал "Мои приложения" корпорации Майкрософт для тестирования приложения в любом режиме. Щелкнув плитку Palo Alto Networks — Aperture в разделе "Мои приложения", вы будете перенаправлены на страницу входа приложения, если выполнены настройки для использования в режиме поставщика услуг, или автоматически войдете в приложение Palo Alto Networks — Aperture, для которого настроен единый вход, если выполнены настройки для использования в режиме поставщика удостоверений. Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Дальнейшие действия

После настройки Palo Alto Networks — Aperture вы можете применить функцию управления сеансом, которая в реальном времени защищает конфиденциальные данные вашей организации от кражи и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).