---
title: Руководство. Интеграция единого входа Azure Active Directory с International SOS Assistance Products | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в приложении International SOS Assistance Products.
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
ms.openlocfilehash: 77d5ace7ce45ed820053ce8c52d375868c8de305
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98736923"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-international-sos-assistance-products"></a>Руководство. Интеграция единого входа Azure Active Directory с International SOS Assistance Products

В этом руководстве описано, как интегрировать International SOS Assistance Products с Azure Active Directory (Azure AD). Интеграция International SOS Assistance Products с Azure AD предоставляет следующие возможности:

* управление доступом к International SOS Assistance Products с помощью Azure AD;
* автоматический вход пользователей в International SOS Assistance Products с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка International SOS Assistance Products с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* International SOS Assistance Products поддерживает единый вход, инициированный **поставщиком услуг**.

* International SOS Assistance Products поддерживает **JIT**-подготовку пользователей.


## <a name="adding-international-sos-assistance-products-from-the-gallery"></a>Добавление International SOS Assistance Products из коллекции

Чтобы настроить интеграцию International SOS Assistance Products с Azure AD, необходимо добавить International SOS Assistance Products из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **International SOS Assistance Products**.
1. Выберите **International SOS Assistance Products** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.


## <a name="configure-and-test-azure-ad-sso-for-international-sos-assistance-products"></a>Настройка и проверка единого входа Azure AD для International SOS Assistance Products

Настройте и проверьте единый вход Azure AD в International SOS Assistance Products с помощью тестового пользователя **B. Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в International SOS Assistance Products.

Чтобы настроить и проверить единый вход Azure AD в International SOS Assistance Products, выполните следующие действия:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в International SOS Assistance Products](#configure-international-sos-assistance-products-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя International SOS Assistance Products](#create-international-sos-assistance-products-test-user)** требуется для того, чтобы в International SOS Assistance Products существовал пользователь B. Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **International SOS Assistance Products** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<SUBDOMAIN>.outsystemsenterprise.com/myassist`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<SUBDOMAIN>.internationalsos.com/sso/saml2/<CUSTOM_ID>`.

    c. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `https://www.okta.com/saml2/service-provider/<IN>`.

    > [!NOTE]
    > Эти значения приведены для примера. Вместо них необходимо указать фактические значения URL-адреса входа, URL-адреса ответа и идентификатора. Чтобы получить эти значения, обратитесь в [службу поддержки клиентов International SOS Assistance Products](mailto:onlinehelp@internationalsos.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** нажмите кнопку "Копировать", чтобы скопировать **URL-адрес метаданных федерации приложений** и сохранить его на компьютере.

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

В этом разделе описано, как включить единый вход Azure для пользователя B. Simon, предоставив этому пользователю доступ к International SOS Assistance Products.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **International SOS Assistance Products**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-international-sos-assistance-products-sso"></a>Настройка единого входа для International SOS Assistance Products

Чтобы настроить единый вход на стороне приложения **International SOS Assistance Products**, вам потребуется отправить **URL-адрес метаданных федерации приложения** [группе поддержки International SOS Assistance Products](mailto:onlinehelp@internationalsos.com). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-international-sos-assistance-products-test-user"></a>Создание тестового пользователя International SOS Assistance Products

В этом разделе описано, как создать пользователя Britta Simon в приложении International SOS Assistance Products. Приложение International SOS Assistance Products поддерживает JIT-подготовку пользователей, которая включена по умолчанию. В этом разделе никакие действия с вашей стороны не требуются. Если пользователь еще не существует в International SOS Assistance Products, он создается после аутентификации.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в International SOS Assistance Products, где можно инициировать поток входа. 

* Перейдите по URL-адресу для входа в International SOS Assistance Products и инициируйте поток входа.

* Вы можете использовать портал "Мои приложения" корпорации Майкрософт. Щелкнув плитку International SOS Assistance Products на портале "Мои приложения", вы перейдете к URL-адресу для входа в приложение International SOS Assistance Products. Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Дальнейшие действия

После настройки International SOS Assistance Products вы сможете применить функцию управления сеансом, которая в реальном времени защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).