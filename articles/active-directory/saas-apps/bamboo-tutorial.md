---
title: Руководство по интеграции Azure Active Directory с SAML SSO for Bamboo by resolution GmbH | Документация Майкрософт
description: Узнайте, как настроить единый вход для Azure Active Directory и SAML SSO for Bamboo by resolution GmbH.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/12/2021
ms.author: jeedes
ms.openlocfilehash: 8ec834d76692f78d1d7bc60ddbd4c73fe4adaede
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101646122"
---
# <a name="tutorial-azure-active-directory-integration-with-saml-sso-for-bamboo-by-resolution-gmbh"></a>Руководство по интеграции Azure Active Directory с SAML SSO for Bamboo by resolution GmbH

В этом руководстве описано, как интегрировать SAML SSO for Bamboo by resolution GmbH с Azure Active Directory (Azure AD). Интеграция SAML SSO for Bamboo by Resolution GmbH с Azure AD обеспечивает следующие возможности:

* Контроль доступа к SAML SSO for Bamboo by resolution GmbH с помощью Azure AD.
* Включение автоматического входа пользователей в SAML SSO for Bamboo by resolution GmbH (единый вход) с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Для настройки интеграции Azure AD с SAML SSO for Bamboo by resolution GmbH требуется:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/));
* подписка SAML SSO for Bamboo by resolution GmbH с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* SAML SSO for Bamboo by resolution GmbH поддерживает единый вход, инициируемый **поставщиком услуг и поставщиком удостоверений**.
* SAML SSO for Bamboo by resolution GmbH поддерживает **JIT**-подготовку пользователей.

## <a name="add-saml-sso-for-bamboo-by-resolution-gmbh-from-the-gallery"></a>Добавление SAML SSO for Bamboo by resolution GmbH из коллекции

Чтобы настроить интеграцию SAML SSO for Bamboo by resolution GmbH в Azure AD, необходимо добавить SAML SSO for Bamboo by resolution GmbH из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **SAML SSO for Bamboo by resolution GmbH**.
1. Выберите **SAML SSO for Bamboo by resolution GmbH** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-with-saml-sso-for-bamboo-by-resolution-gmbh"></a>Настройка и проверка единого входа Azure AD в SAML SSO for Bamboo by resolution GmbH

Настройте и проверьте единый вход SAML SSO for Bamboo by resolution GmbH с помощью тестового пользователя **B. Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в SAML SSO for Bamboo by resolution GmbH.

Чтобы настроить и проверить единый вход Azure AD в SAML SSO for Bamboo by resolution GmbH, выполните следующие действия:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
     1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
     1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
2. **[Настройка единого входа в SAML SSO for Bamboo by resolution GmbH](#configure-saml-sso-for-bamboo-by-resolution-gmbh-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя SAML SSO for Bamboo by resolution GmbH](#create-saml-sso-for-bamboo-by-resolution-gmbh-test-user)** требуется для того, чтобы в SAML SSO for Bamboo by resolution GmbH существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

### <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

В этом разделе описано, как включить единый вход Azure AD на портале Azure.
 
1. На портале Azure на странице интеграции с приложением **SAML SSO for Bamboo by resolution GmbH** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить эти параметры.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)
4. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** выполните следующие действия.

    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `https://<server-base-url>/plugins/servlet/samlsso`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<server-base-url>/plugins/servlet/samlsso`.

5. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

     В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://<server-base-url>/plugins/servlet/samlsso`.

    > [!NOTE]
    > Эти значения приведены для примера. Замените их фактическими значениями идентификатора, URL-адреса ответа и URL-адреса входа. Обратитесь к [группе поддержки SAML SSO for Bamboo by resolution GmbH](https://marketplace.atlassian.com/plugins/com.resolution.atlasplugins.samlsso-bamboo/server/support) для получения этих значений. Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

6. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

7. Требуемые URL-адреса можно скопировать из раздела **Настройка SAML SSO for Bamboo by resolution GmbH**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)


### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD

В этом разделе описано, как на портале Azure создать тестового пользователя с именем B.Simon.

1. Слева на портале Azure выберите **Azure Active Directory** > **Пользователи** > **Все пользователи**.
1. В верхней части экрана выберите **Новый пользователь**.
1. В разделе **Свойства пользователя** выполните следующие действия.
   1. В поле **Имя** введите `B.Simon`.  
   1. В поле **Имя пользователя** введите username@companydomain.extension. Например, `B.Simon@contoso.com`.
   1. Установите флажок **Показать пароль** и запишите значение пароля.
   1. Нажмите кнопку **создания**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как разрешить пользователю B. Simon использовать единый вход Azure, предоставив этому пользователю доступ к SAML SSO for bamboo by resolution GmbH.

1. На портале Azure выберите **Корпоративные приложения** > **Все приложения**.
1. В списке приложений выберите **SAML SSO for Bamboo by resolution GmbH**.
1. На странице сводных сведений о приложении откройте раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**. Затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** из списка пользователей. Затем нажмите кнопку **Выбрать** в нижней части экрана.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** выберите **Назначить**.

### <a name="configure-saml-sso-for-bamboo-by-resolution-gmbh-sso"></a>Настройка единого входа в SAML SSO for Bamboo by resolution GmbH

1. Войдите на свой корпоративный сайт SAML SSO for Bamboo by resolution GmbH с правами администратора.

1. Справа от главной панели инструментов щелкните **Параметры** > **Надстройки**.

    ![Параметры](./media/bamboo-tutorial/tutorial_bamboo_setings.png)

1. В разделе "Безопасность" в строке меню щелкните **SAML SingleSignOn**.

    ![Samlsingle](./media/bamboo-tutorial/tutorial_bamboo_samlsingle.png)

1. На **странице конфигурации подключаемого модуля SAML SIngleSignOn** щелкните **Add idp** (Добавить поставщик удостоверений).

    ![Добавление поставщика удостоверений](./media/bamboo-tutorial/tutorial_bamboo_addidp.png)

1. На странице **Choose your SAML Identity Provider** (Выбор поставщика удостоверений SAML) выполните следующие действия:

    ![Поставщик удостоверений](./media/bamboo-tutorial/tutorial_bamboo_identityprovider.png)

    а. Выберите **Azure AD** в качестве **типа поставщика удостоверений**.

    b. В текстовом поле **Имя** введите имя.

    c. В текстовом поле **Описание** введите описание.

    d. Щелкните **Далее**.

1. На странице **Identity provider configuration page** (Настройка поставщика удостоверений) нажмите кнопку **Далее**.

    ![Настройка удостоверения](./media/bamboo-tutorial/tutorial_bamboo_identityconfig.png)

1. На странице **импорта метаданных SAML поставщика удостоверений** щелкните **Загрузить файл**, чтобы передать файл **метаданных в формате XML**, скачанный на портале Azure.

    ![idpmetadata](./media/bamboo-tutorial/tutorial_bamboo_idpmetadata.png)

1. Щелкните **Далее**.

1. Нажмите кнопку **Сохранить параметры**.

### <a name="create-saml-sso-for-bamboo-by-resolution-gmbh-test-user"></a>Создание тестового пользователя в SAML SSO for Bamboo by resolution GmbH

Цель этого раздела — создать в приложении SAML SSO for Bamboo by resolution GmbH пользователя с именем Britta Simon. SAML SSO for Bamboo by resolution GmbH обеспечивает своевременную подготовку, а также создание пользователей вручную. Для согласования ваших требований обратитесь в [службу поддержки клиентов SAML SSO for Bamboo by resolution GmbH](https://marketplace.atlassian.com/plugins/com.resolution.atlasplugins.samlsso-bamboo/server/support).

### <a name="test-sso"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

#### <a name="sp-initiated"></a>Инициация поставщиком услуг:

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в SAML SSO for Bamboo by resolution GmbH SSO, где можно инициировать поток входа.  

* Перейдите по URL-адресу для входа в SAML SSO for Bamboo by resolution GmbH SSO и инициируйте поток входа.

#### <a name="idp-initiated"></a>Вход, инициированный поставщиком удостоверений

* На портале Azure выберите **Тестировать приложение**, и вы автоматически войдете в приложение SAML SSO for Bamboo by resolution GmbH SSO, для которого настроен единый вход.

Вы можете также использовать портал "Мои приложения" корпорации Майкрософт для тестирования приложения в любом режиме. Щелкнув плитку SAML SSO for Bamboo by resolution GmbH SSO на портале "Мои приложения", вы будете перенаправлены на страницу входа приложения для инициации потока входа (при настройке в режиме поставщика услуг) или автоматически войдете в приложение SAML SSO for Bamboo by resolution GmbH SSO, для которого настроен единый вход (при настройке в режиме поставщика удостоверений). Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Дальнейшие действия

После настройки SAML SSO for Bamboo by resolution GmbH SSO можно применить управление сеансами, которое в реальном времени защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).