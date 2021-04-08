---
title: Руководство по интеграции Azure Active Directory с Keeper Password Manager & Digital Vault | Документы Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в приложении Keeper Password Manager & Digital Vault.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/13/2020
ms.author: jeedes
ms.openlocfilehash: b70c50e7c2900f884dd4d91c6650205bc626326e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96178047"
---
# <a name="tutorial-azure-active-directory-integration-with-keeper-password-manager--digital-vault"></a>Руководство по интеграции Azure Active Directory с Keeper Password Manager & Digital Vault

В этом учебнике описано, как интегрировать Keeper Password Manager & Digital Vault с Azure Active Directory (Azure AD).
Эта интеграция обеспечивает следующие преимущества:

* С помощью Azure AD вы можете контролировать доступ к Keeper Password Manager & Digital Vault.
* Вы можете включить автоматический вход пользователей в Keeper Password Manager & Digital Vault (единый вход) с помощью учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно — через портал Azure.


## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с Keeper Password Manager & Digital Vault, вам потребуется:

* подписка Azure AD; Если у вас нет среды Azure AD, вы можете получить [пробную версию на один месяц](https://azure.microsoft.com/pricing/free-trial/).
* Подписка на Keeper Password Manager & Digital Vault с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Keeper Password Manager & Digital Vault поддерживает инициированный поставщиком услуг единый вход.

* Keeper Password Manager & Digital Vault поддерживает JIT- подготовку пользователей.

## <a name="add-keeper-password-manager--digital-vault-from-the-gallery"></a>Добавление Keeper Password Manager & Digital Vault из коллекции

Чтобы настроить интеграцию Keeper Password Manager & Digital Vault с Azure AD, добавьте приложение из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавить из коллекции** в поле поиска введите **Keeper Password Manager & Digital Vault**.
1. На панели результатов выберите **Keeper Password Manager & Digital Vault**, а затем добавьте приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-keeper-password-manager--digital-vault"></a>Настройка и проверка единого входа Azure AD для Keeper Password Manager & Digital Vault

Настройте и проверьте единый вход Azure AD в Keeper Password Manager & Digital Vault с помощью тестового пользователя **B.Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем AAD и соответствующим пользователем в Keeper Password Manager & Digital Vault.

Чтобы настроить и проверить единый вход Azure AD для Keeper Password Manager & Digital Vault, следует выполнить нижеперечисленные действия.

1. [Настройка единого входа Azure AD](#configure-azure-ad-sso) необходима, чтобы пользователи могли использовать эту функцию.

    * [Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user) требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
    * [Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user) необходимо, чтобы разрешить Britta Simon использовать единый вход Azure AD.

1. [Настройка единого входа в Keeper Password Manager & Digital Vault](#configure-keeper-password-manager--digital-vault-sso) необходима, чтобы задать параметры единого входа на стороне приложения.
    * [Создание тестового пользователя Keeper Password Manager & Digital Vault](#create-a-keeper-password-manager--digital-vault-test-user) требуется для создания в Keeper Password Manager & Digital Vault пользователя Britta Simon, связанного с соответствующим представлением в Azure AD.
1. [Проверка единого входа](#test-sso) позволяет убедиться в правильности конфигурации.

### <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Keeper Password Manager & Digital Vault** найдите раздел **Управление**. Выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить эти параметры.

   ![Снимок экрана: настройка единого входа с помощью SAML с выделенным значком карандаша.](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** сделайте следующее:

    а. В поле **URL-адрес для входа** введите URL-адрес в таком формате:
    * Для облачного единого входа: `https://keepersecurity.com/api/rest/sso/saml/sso/<CLOUD_INSTANCE_ID>`
    * Для локального единого входа: `https://<KEEPER_FQDN>/sso-connect/saml/login`

    b. В поле **Идентификатор сущности** введите URL-адрес в таком формате:
    * Для облачного единого входа: `https://keepersecurity.com/api/rest/sso/saml/<CLOUD_INSTANCE_ID>`
    * Для локального единого входа: `https://<KEEPER_FQDN>/sso-connect`

    c. В поле **URL-адрес ответа** введите URL-адрес в следующем формате:
    * Для облачного единого входа: `https://keepersecurity.com/api/rest/sso/saml/sso/<CLOUD_INSTANCE_ID>`
    * Для локального единого входа: `https://<KEEPER_FQDN>/sso-connect/saml/sso`

    > [!NOTE]
    > Эти значения приведены в качестве примера. Укажите вместо них фактические значения URL-адреса для входа, идентификатора и URL-адреса ответа. Чтобы получить эти значения, обратитесь в [службу поддержки клиентов Keeper Password Manager & Digital Vault](https://keepersecurity.com/contact.html). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. Приложение Keeper Password Manager & Digital Vault ожидает утверждения SAML в определенном формате, который предполагает, что вы должны добавить сопоставления настраиваемых атрибутов в конфигурацию атрибутов токена SAML. На следующем снимке экрана показан список атрибутов по умолчанию.

    ![Снимок экрана: раздел "Атрибуты пользователей и утверждения"](common/default-attributes.png)

1. Кроме того, приложение Keeper Password Manager & Digital Vault ожидает несколько дополнительных атрибутов в ответе SAML. Это показано в таблице ниже. Эти атрибуты также заранее заполнены, однако их можно изменить в соответствии с требованиями.

    | Имя | Атрибут источника|
    | ------------| --------- |
    | First | user.givenname |
    | Последний | user.surname |
    | Email | user.mail |

5. В области **Настроить единый вход с помощью SAML**, в разделе **Сертификат подписи SAML** выберите значение **Загрузить**. Этот параметр активирует загрузку **XML-файла метаданных федерации** из параметров в соответствии с требованиями и сохраняет его на компьютере.

    ![Снимок экрана: Сертификат подписи SAML с выделенным пунктом "Загрузить"](common/metadataxml.png)

6. В области **Настроить Keeper Password Manager & Digital Vault** скопируйте соответствующие URL-адреса в соответствии с вашими требованиями.

    ![Снимок экрана: настройка Keeper Password Manager & Digital Vault с выделенными URL-адресами.](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD 

В этом разделе описано, как создать на портале Azure тестового пользователя с именем `B.Simon`.

1. Слева на портале Azure выберите **Azure Active Directory** > **Пользователи** > **Все пользователи**.
1. В верхней части экрана щелкните **Новый пользователь**.
1. В разделе **Свойства пользователя** выполните следующие действия.
   1. Для параметра **Имя** введите `B.Simon`.  
   1. В поле **Имя пользователя** введите значение `username@companydomain.extension`. Например, `B.Simon@contoso.com`.
   1. Выберите **Показать пароль**, а затем запишите предоставленное значение.
   1. Нажмите кнопку **создания**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как позволить пользователю B.Simon использовать единый вход Azure, предоставив ему доступ к Keeper Password Manager & Digital Vault.

1. На портале Azure выберите **Корпоративные приложения** > **Все приложения**.
1. В списке приложений выберите **Keeper Password Manager & Digital Vault**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**. В разделе **Добавление назначения** выберите **Пользователи и группы**.
1. В области **Пользователи и группы** выберите **B.Simon** в списке пользователей. Затем нажмите кнопку **Выбрать** в нижней части экрана.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее в списке **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль **Доступ по умолчанию**.
1. В области **Добавление назначения** выберите **Назначить**.


## <a name="configure-keeper-password-manager--digital-vault-sso"></a>Настройка единого входа в Keeper Password Manager & Digital Vault

Чтобы настроить единый вход для приложения, ознакомьтесь с рекомендациями в [Руководстве по Keeper](https://docs.keeper.io/sso-connect-guide/).

### <a name="create-a-keeper-password-manager--digital-vault-test-user"></a>Создание тестового пользователя Keeper Password Manager & Digital Vault

Чтобы пользователи Azure AD могли входить в Keeper Password Manager & Digital Vault, их необходимо подготовить. Приложение поддерживает JIT-подготовку пользователей, поэтому после проверки подлинности пользователи будут созданы в приложении автоматически. Если вы хотите настроить пользователей вручную, обратитесь в [службу поддержки Keeper](https://keepersecurity.com/contact.html).

## <a name="test-sso"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

* На портале Azure выберите **Проверить это приложение**. После этого вы будете перенаправлены на URL-адрес входа для Keeper Password Manager & Digital Vault, где можно будет начать вход. 

* Вы можете перейти непосредственно к URL-адресу входа для этого приложения и начать вход из него.

* Вы можете использовать Панель доступа (Майкрософт). Если выбрать элемента **Keeper Password Manager & Digital Vault** на панели доступа, вы будете перенаправлены на URL-адрес входа для приложения. Дополнительные сведения о Панели доступа см. в статье о [входе на портал "Мои приложения" и запуске приложений на нем](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Дальнейшие действия

После настройки Keeper Password Manager & Digital Vault можно принудительно включить управление сеансом. Это защищает от утечки данных и несанкционированного доступа к конфиденциальным данным вашей организации в режиме реального времени. Управление сеансом является расширением функции условного доступа. Подробные сведения см. в статье об [управлении сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).