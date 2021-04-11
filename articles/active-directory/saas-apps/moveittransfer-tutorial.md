---
title: Руководство по интеграции Azure Active Directory с приложением MOVEit Transfer - Azure AD integration | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в MOVEit Transfer - Azure AD integration.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/27/2021
ms.author: jeedes
ms.openlocfilehash: 73f378bb0f71df4df3731a3ef2a1fdb7f8abb4aa
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101653078"
---
# <a name="tutorial-azure-active-directory-integration-with-moveit-transfer---azure-ad-integration"></a>Руководство по интеграции Azure Active Directory с MOVEit Transfer - Azure AD integration

В этом руководстве описано, как интегрировать MOVEit Transfer - Azure AD integration с Azure Active Directory (AAD). Интеграция MOVEit Transfer - Azure AD integration с Azure AD обеспечивает следующие возможности.

* С помощью Azure AD вы можете контролировать доступ к MOVEit Transfer - Azure AD integration.
* Вы можете включить автоматический вход пользователей в MOVEit Transfer - Azure AD integration с учетными записями AAD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка MOVEit Transfer - Azure AD integration с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* MOVEit Transfer - Azure AD integration поддерживает единый вход, инициируемый **поставщиком услуг**.

## <a name="add-moveit-transfer---azure-ad-integration-from-the-gallery"></a>Добавление MOVEit Transfer - Azure AD integration из коллекции

Чтобы настроить интеграцию MOVEit Transfer - Azure AD integration с Azure AD, необходимо добавить MOVEit Transfer - Azure AD integration из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **MOVEit Transfer - Azure AD integration**.
1. Выберите **MOVEit Transfer - Azure AD integration** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-moveit-transfer---azure-ad-integration"></a>Настройка и проверка единого входа Azure AD для MOVEit Transfer - Azure AD integration

Настройте и проверьте единый вход Azure AD в MOVEit Transfer - Azure AD integration с помощью тестового пользователя **B. Simon**. Чтобы обеспечить работу единого входа, нужно установить связь между пользователем Azure AD и соответствующим пользователем в MOVEit Transfer - Azure AD integration.

Чтобы настроить и проверить единый вход Azure AD в MOVEit Transfer - Azure AD integration, выполните следующие действия:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в MOVEit Transfer - Azure AD integration](#configure-moveit-transfer---azure-ad-integration-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя MOVEit Transfer - Azure AD integration](#create-moveit-transfer---azure-ad-integration-test-user)** требуется для того, чтобы в MOVEit Transfer - Azure AD integration существовал пользователь B. Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

### <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **MOVEit Transfer - Azure AD integration** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. Если у вас есть **файл метаданных поставщика служб**, выполните следующие действия в разделе **Базовая конфигурация SAML**:

    а. Щелкните **Отправить файл метаданных**.

    ![Передача файла метаданных](common/upload-metadata.png)

    b. Щелкните **значок папки**, выберите файл метаданных и нажмите кнопку **Отправить**.

    ![Выбор файла метаданных](common/browse-upload-metadata.png)

    c. После успешной передачи файла метаданных значения **Идентификатор** и **URL-адрес ответа** в разделе **Базовая конфигурация SAML** заполнятся автоматически.

    ![Сведения о домене и URL-адресах единого входа для MOVEit Transfer - Azure AD integration](common/sp-identifier-reply.png)

    В текстовом поле **URL-адрес входа** введите URL-адрес: `https://contoso.com`.

    > [!NOTE]
    > Значение **URL-адрес входа** приведено для примера. Вместо него необходимо указать фактический URL-адрес входа. Обратитесь к [группе поддержки клиентов MOVEit Transfer - Azure AD integration](https://community.ipswitch.com/s/support), чтобы получить это значение. Вы можете скачать **файл метаданных поставщика услуг**, воспользовавшись **URL-адресом метаданных поставщика услуг**. Это описано далее в разделе **Настройка единого входа в MOVEit Transfer - Azure AD integration** этого руководства. Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

4. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

6. Скопируйте требуемые URL-адреса из раздела **Настройка MOVEit Transfer - Azure AD integration**.

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

В этом разделе описано, как предоставить пользователю B. Simon доступ к приложению MOVEit Transfer - Azure AD integration, чтобы он мог использовать единый вход Azure.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **MOVEit Transfer - Azure AD integration**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="configure-moveit-transfer---azure-ad-integration-sso"></a>Настройка единого входа в MOVEit Transfer - Azure AD integration

1. Войдите в клиент MOVEit Transfer с правами администратора.

2. В левой панели навигации щелкните **Settings**(Параметры).

    ![Раздел Settings (Параметры) на стороне приложения](./media/moveittransfer-tutorial/settings.png)

3. Щелкните ссылку **Single Signon** (Единый вход), которая расположена в разделе **Security Policies -> User Auth** (Политики безопасности -> Проверка подлинности пользователя).

    ![Политики безопасности на стороне приложения](./media/moveittransfer-tutorial/sso.png)

4. Щелкните ссылку с URL-адресом метаданных, чтобы скачать документ метаданных.

    ![URL-адрес метаданных поставщика службы](./media/moveittransfer-tutorial/metadata.png)
    
   * Проверьте, соответствует ли **entityID** значению **Идентификатор** в разделе **Базовая конфигурация SAML**.
   * Убедитесь, что URL-адрес расположения **AssertionConsumerService** совпадает со значением **URL-адрес ответа** в разделе **Базовая конфигурация SAML**.
    
     ![Настройка единого входа на стороне приложения](./media/moveittransfer-tutorial/xml.png)

5. Щелкните кнопку **Add Identity Provider** (Добавить поставщик удостоверений), чтобы добавить новый федеративный поставщик удостоверений.

    ![Добавление поставщика удостоверений](./media/moveittransfer-tutorial/idp.png)

6. Щелкните **Browse...** (Обзор...), чтобы выбрать файл метаданных, который вы скачали на портале Azure, затем нажмите кнопку **Add Identity Provider** (Добавить поставщик удостоверений) и отправьте скачанный файл.

    ![Поставщик удостоверений SAML](./media/moveittransfer-tutorial/saml.png)

7. Выберите значение **Yes** (Да) для параметра **Enabled** (Включено) в разделе **Edit Federated Identity Provider Settings...** (Изменение параметров федеративного поставщика удостоверений) и нажмите кнопку **Save** (Сохранить).

    ![Параметры федеративного поставщика удостоверений](./media/moveittransfer-tutorial/save.png)

8. На странице **Edit Federated Identity Provider User Settings** (Изменение параметров федеративного поставщика удостоверений) выполните следующие действия:
    
    ![Изменение параметров федеративного поставщика удостоверений](./media/moveittransfer-tutorial/attributes.png)
    
    а. Выберите значение **SAML NameID** для параметра **Login name** (Имя для входа).
    
    b. Выберите **Other** (Другое) в качестве **полного имени** и в тестовом поле **Attribute name** (Имя атрибута) введите значение: `http://schemas.microsoft.com/identity/claims/displayname`.
    
    c. Выберите **Other** (Другое) в качестве **электронной почты** и в тестовом поле **Attribute name** (Имя атрибута) введите значение: `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`.
    
    d. Выберите значение **Yes** (Да) для параметра **Auto-create account on signon** (Автоматическое создание учетной записи при первом входе).
    
    д) Нажмите кнопку **Сохранить** .

### <a name="create-moveit-transfer---azure-ad-integration-test-user"></a>Создание тестового пользователя MOVEit Transfer - Azure AD integration

Цель этого раздела — создать пользователя с именем Britta Simon в MOVEit Transfer - Azure AD integration. Приложение MOVEit Transfer - Azure AD integration поддерживает JIT-подготовку, и вы уже включили ее. В этом разделе никакие действия с вашей стороны не требуются. Пользователь будет создан при попытке получить доступ к MOVEit Transfer - Azure AD integration (если он еще не создан).

>[!NOTE]
>Если вам нужно вручную создать пользователя, необходимо обратиться в [службу поддержки клиентов MOVEit Transfer - Azure AD integration](https://community.ipswitch.com/s/support).

### <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в MOVEit Transfer - Azure AD integration, где можно инициировать поток входа. 

* Перейдите по URL-адресу для входа в MOVEit Transfer - Azure AD integration и инициируйте поток входа.

* Вы можете использовать портал "Мои приложения" корпорации Майкрософт. Щелкнув плитку MOVEit Transfer - Azure AD integration на портале "Мои приложения", вы автоматически войдете в приложение MOVEit Transfer - Azure AD integration, для которого настроен единый вход. Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Дальнейшие действия

После настройки MOVEit Transfer - Azure AD integration вы можете применить функцию управления сеансами, которая в реальном времени защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).