---
title: Руководство по Интеграция Azure Active Directory с приложением Splunk Enterprise and Splunk Cloud | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Splunk Enterprise и Splunk Cloud.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/02/2021
ms.author: jeedes
ms.openlocfilehash: 81107b3655ae86d51e36445ce46661d553ab3b5a
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101649867"
---
# <a name="tutorial-azure-active-directory-integration-with-splunk-enterprise-and-splunk-cloud"></a>Руководство по Интеграция Azure Active Directory с приложением Splunk Enterprise and Splunk Cloud

В этом руководстве описано, как интегрировать Splunk Enterprise и Splunk Cloud с Azure Active Directory (Azure AD). Интеграция Splunk Enterprise и Splunk Cloud с Azure AD обеспечивает следующие возможности:

* Контроль доступа к Splunk Enterprise и Splunk Cloud с помощью Azure AD.
* Автоматический вход пользователей в Splunk Enterprise и Splunk Cloud с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Splunk Enterprise и Splunk Cloud с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Splunk Enterprise and Splunk Cloud поддерживает единый вход, инициированный **поставщиком услуг**.

## <a name="add-splunk-enterprise-and-splunk-cloud-from-the-gallery"></a>Добавление Splunk Enterprise и Splunk Cloud из коллекции

Чтобы настроить интеграцию Splunk Enterprise и Splunk Cloud с Azure AD, необходимо добавить Splunk Enterprise и Splunk Cloud из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Splunk Enterprise и Splunk Cloud**.
1. Выберите **Splunk Enterprise и Splunk Cloud** в результатах поиска и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-splunk-enterprise-and-splunk-cloud"></a>Настройка и проверка единого входа Azure AD для Splunk Enterprise и Splunk Cloud

Настройте и проверьте единый вход Azure AD в приложения Splunk Enterprise и Splunk Cloud с помощью тестового пользователя **B. Simon**. Чтобы обеспечить работу единого входа, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Splunk Enterprise и Splunk Cloud.

Чтобы настроить и проверить единый вход Azure AD в Splunk Enterprise и Splunk Cloud, выполните следующие действия.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Splunk Enterprise и Splunk Cloud](#configure-splunk-enterprise-and-splunk-cloud-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя Splunk Enterprise и Splunk Cloud](#create-splunk-enterprise-and-splunk-cloud-test-user)** требуется для того, чтобы в Splunk Enterprise и Splunk Cloud существовал пользователь B. Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложениями **Splunk Enterprise и Splunk Cloud** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)
4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    а. В текстовое поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<splunkserverUrl>/app/launcher/home`.

    b. В поле **Идентификатор** введите URL-адрес в следующем формате: `<splunkserverUrl>`.

    c. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<splunkserver>/saml/acs`.

    > [!NOTE]
    > Эти значения приведены для примера. Укажите вместо них фактические значения URL-адреса для входа, идентификатора и URL-адреса ответа. Чтобы получить эти значения, обратитесь в [службу поддержки клиентов Splunk Enterprise и Splunk Cloud](https://www.splunk.com/en_us/about-splunk/contact-us.html). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

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

В этом разделе описано, как включить единый вход Azure для пользователя B. Simon, предоставив этому пользователю доступ к Splunk Enterprise и Splunk Cloud.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Splunk Enterprise и Splunk Cloud**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-splunk-enterprise-and-splunk-cloud-sso"></a>Настройка единого входа в Splunk Enterprise и Splunk Cloud

  Для настройки единого входа на стороне **Splunk Enterprise and Splunk Cloud** необходимо отправить загруженный **XML метаданных федерации** и соответствующие URL-адреса, скопированные на портале Azure, [группе поддержки Splunk Enterprise and Splunk Cloud](https://www.splunk.com/en_us/about-splunk/contact-us.html). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-splunk-enterprise-and-splunk-cloud-test-user"></a>Создание тестового пользователя Splunk Enterprise and Splunk Cloud

В этом разделе описано, как создать пользователя Britta Simon в Splunk Enterprise и Splunk Cloud. Чтобы добавить пользователей на платформу Splunk Enterprise and Splunk Cloud, обратитесь в [службу поддержки Splunk Enterprise and Splunk Cloud](https://www.splunk.com/en_us/about-splunk/contact-us.html). Перед использованием единого входа необходимо создать и активировать пользователей.

## <a name="test-sso"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в Splunk Enterprise и Splunk Cloud, где можно инициировать поток входа. 

* Перейдите по URL-адресу для входа в Splunk Enterprise и Splunk Cloud и инициируйте поток входа.

* Вы можете использовать портал "Мои приложения" корпорации Майкрософт. Щелкнув плитку Splunk Enterprise и Splunk Cloud в области "Мои приложения", вы будете перенаправлены на URL-адрес входа в Splunk Enterprise и Splunk Cloud. Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Дальнейшие действия

После настройки Splunk Enterprise и Splunk Cloud вы можете применить функцию управления сеансами, которая защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа в реальном времени. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).