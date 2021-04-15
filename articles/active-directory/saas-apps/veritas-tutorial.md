---
title: Руководство. Интеграция Azure Active Directory с Veritas Enterprise Vault.cloud SSO | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Veritas Enterprise Vault.cloud SSO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/22/2021
ms.author: jeedes
ms.openlocfilehash: 2796928316197bd48723253dbc97dfcd34ac95e8
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/02/2021
ms.locfileid: "106222288"
---
# <a name="tutorial-azure-active-directory-integration-with-veritas-enterprise-vaultcloud-sso"></a>Руководство. Интеграция Azure Active Directory с Veritas Enterprise Vault.cloud SSO

В этом руководстве описано, как интегрировать Veritas Enterprise Vault.cloud SSO с Azure Active Directory (Azure AD). Интеграция Veritas Enterprise Vault.cloud SSO с Azure AD обеспечивает следующие возможности:

* Контроль доступа к Veritas Enterprise Vault.cloud SSO с помощью Azure AD.
* Автоматический вход пользователей в Veritas Enterprise Vault.cloud SSO с учетными записями Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с Veritas Enterprise Vault.cloud SSO, вам потребуется следующее:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/));
* подписка Veritas Enterprise Vault.cloud SSO с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Veritas Enterprise Vault.cloud SSO поддерживает единый вход, инициированный **поставщиком услуг**.

> [!NOTE]
> Идентификатор этого приложения — фиксированное строковое значение, поэтому в одном клиенте можно настроить только один экземпляр.

## <a name="add-veritas-enterprise-vaultcloud-sso-from-the-gallery"></a>Добавление Veritas Enterprise Vault.cloud SSO из коллекции

Чтобы настроить интеграцию Veritas Enterprise Vault.cloud SSO c Azure AD, необходимо добавить Veritas Enterprise Vault.cloud SSO из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Veritas Enterprise Vault.cloud SSO**.
1. Выберите **Veritas Enterprise Vault.cloud SSO** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-veritas-enterprise-vaultcloud-sso"></a>Настройка и проверка единого входа Azure AD для Veritas Enterprise Vault.cloud SSO

Настройте и проверьте единый вход Azure AD в Veritas Enterprise Vault.cloud SSO с помощью тестового пользователя **B.Simon**. Чтобы обеспечить работу единого входа, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Veritas Enterprise Vault.cloud SSO.

Чтобы настроить и проверить единый вход Azure AD в Veritas Enterprise Vault.cloud SSO, выполните следующие действия:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Veritas Enterprise Vault.cloud SSO](#configure-veritas-enterprise-vaultcloud-sso-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя Veritas Enterprise Vault.cloud SSO](#create-veritas-enterprise-vaultcloud-sso-test-user)** требуется для создания пользователя B.Simon в Veritas Enterprise Vault.cloud SSO, связанного с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Veritas Enterprise Vault.cloud SSO** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    а. В текстовое поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://personal.ap.archive.veritas.com/CID=<CUSTOMERID>`.

    b. В поле **Идентификатор** введите один из URL-адресов для соответствующего центра обработки данных:

    | Центр обработки данных| URL-адрес |
    |----------|----|
    | Северная Америка| `https://auth.lax.archivecloud.net` |
    | Европа | `https://auth.ams.archivecloud.net` |
    | Азиатско-Тихоокеанский регион| `https://auth.syd.archivecloud.net`|

    c. В текстовом поле **URL-адрес ответа** введите один из URL-адресов для соответствующего центра обработки данных:

    | Центр обработки данных| URL-адрес |
    |----------|----|
    | Северная Америка| `https://auth.lax.archivecloud.net` |
    | Европа | `https://auth.ams.archivecloud.net` |
    | Азиатско-Тихоокеанский регион| `https://auth.syd.archivecloud.net`|

    > [!NOTE]
    > Это значение приведено для справки. Вместо него необходимо указать фактический URL-адрес входа. Для получения этого значения обратитесь в [службу поддержки клиентов Veritas Enterprise Vault.cloud SSO](https://www.veritas.com/support/.html). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (Base64)** из предложенных вариантов, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

6. Требуемые URL-адреса вы можете скопировать из раздела **Настройка Veritas Enterprise Vault.cloud SSO**.

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

В этом разделе описано, как разрешить пользователю B.Simon использовать единый вход Azure, предоставив доступ к Veritas Enterprise Vault.cloud SSO.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Veritas Enterprise Vault.cloud SSO**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-veritas-enterprise-vaultcloud-sso-sso"></a>Настройка единого входа для Veritas Enterprise Vault.cloud SSO

Чтобы настроить единый вход на стороне **Veritas Enterprise Vault.cloud SSO**, нужно передать скачанный **сертификат в кодировке Base64** и соответствующие URL-адреса, скопированные на портале Azure, в [службу поддержки Veritas Enterprise Vault.cloud SSO](https://www.veritas.com/support/.html). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-veritas-enterprise-vaultcloud-sso-test-user"></a>Создание тестового пользователя Veritas Enterprise Vault.cloud SSO

В этом разделе описано, как создать пользователя Britta Simon в Veritas Enterprise Vault.cloud SSO. Обратитесь к [группе поддержки Veritas Enterprise Vault.cloud SSO](https://www.veritas.com/support/.html), чтобы добавить пользователей на платформу Veritas Enterprise Vault.cloud SSO. Перед использованием единого входа необходимо создать и активировать пользователей.

## <a name="test-sso"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в Veritas Enterprise Vault.cloud SSO, где можно инициировать поток входа. 

* Перейдите по URL-адресу для входа в Veritas Enterprise Vault.cloud SSO и инициируйте поток входа.

* Вы можете использовать портал "Мои приложения" корпорации Майкрософт. Щелкнув элемент Veritas Enterprise Vault.cloud SSO в области "Мои приложения", вы перейдете по URL-адресу входа Veritas Enterprise Vault.cloud SSO. Дополнительные сведения о портале "Мои приложения" см. в [этой статье](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="next-steps"></a>Дальнейшие действия

После настройки приложения Veritas Enterprise Vault.cloud SSO вы сможете применить функцию управления сеансами, чтобы в реальном времени защищать конфиденциальные данные своей организации от мошенничества и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).
