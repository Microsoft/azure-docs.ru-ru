---
title: Руководство по интеграции Azure Active Directory с SAML SSO for Bitbucket by resolution GmbH | Документация Майкрософт
description: Узнайте, как настроить единый вход для Azure Active Directory и SAML SSO for Bitbucket by resolution GmbH.
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
ms.openlocfilehash: aa5e3e88ceb957728799f27482de7477ba6b7b48
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98621252"
---
# <a name="tutorial-azure-active-directory-integration-with-saml-sso-for-bitbucket-by-resolution-gmbh"></a>Руководство по интеграции Azure Active Directory с SAML SSO for Bitbucket by resolution GmbH

В этом руководстве описано, как интегрировать SAML SSO for Bitbucket by resolution GmbH с Azure Active Directory (Azure AD). Интеграция SAML SSO for Bitbucket by resolution GmbH с Azure AD обеспечивает следующие возможности:

* Контроль доступа к SAML SSO for Bamboo by resolution GmbH с помощью Azure AD.
* Автоматический вход пользователей в SAML SSO for Bitbucket by resolution GmbH с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.


## <a name="prerequisites"></a>Предварительные требования

Для настройки интеграции Azure AD с SAML SSO for Bitbucket by resolution GmbH требуется:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка SAML SSO for Bitbucket by resolution GmbH с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* SAML SSO for Bitbucket by resolution GmbH поддерживает единый вход, инициированный **поставщиком услуг и поставщиком удостоверений**.
* SAML SSO for Bitbucket by resolution GmbH поддерживает **JIT**-подготовку пользователей.


## <a name="add-saml-sso-for-bitbucket-by-resolution-gmbh-from-the-gallery"></a>Добавление SAML SSO for Bitbucket by resolution GmbH из коллекции

Чтобы настроить интеграцию SAML SSO for Bitbucket by resolution GmbH в Azure AD, необходимо добавить SAML SSO for Bitbucket by resolution GmbH из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области слева выберите **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **SAML SSO for Bitbucket by resolution GmbH**.
1. Выберите **SAML SSO for Bitbucket by resolution GmbH** из результатов поиска и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-saml-sso-for-bitbucket-by-resolution-gmbh"></a>Настройка и проверка единого входа Azure AD для SAML SSO for Bitbucket by resolution GmbH

Настройте и проверьте единый вход SAML SSO for Bitbucket by resolution GmbH с помощью тестового пользователя **B. Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в SAML SSO for Bitbucket by resolution GmbH.

Чтобы настроить и проверить единый вход Azure AD в SAML SSO for Bitbucket by resolution GmbH, выполните следующие действия.


1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
2. **[Настройка единого входа в SAML SSO for Bitbucket by resolution GmbH](#configure-saml-sso-for-bitbucket-by-resolution-gmbh-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя SAML SSO for Bitbucket by resolution GmbH](#create-saml-sso-for-bitbucket-by-resolution-gmbh-test-user)** требуется, чтобы создать в SAML SSO for Bitbucket by resolution GmbH пользователя Britta Simon, связанного с соответствующим пользователем в Azure AD.
6. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

В этом разделе описано, как включить единый вход Azure AD на портале Azure.
 
1. На портале Azure на странице интеграции с приложением **SAML SSO for Bitbucket by resolution GmbH** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить эти параметры.

    ![Правка базовой конфигурации SAML](common/edit-urls.png)

4. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** выполните следующие действия.


    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `https://<server-base-url>/plugins/servlet/samlsso`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<server-base-url>/plugins/servlet/samlsso`.

    c. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.
    
    В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://<server-base-url>/plugins/servlet/samlsso`.

    > [!NOTE]
    > Эти значения приведены для примера. Замените их фактическими значениями идентификатора, URL-адреса ответа и URL-адреса входа. Для получения этих значений обратитесь к [группе поддержки SAML SSO for Bitbucket by resolution GmbH](https://marketplace.atlassian.com/apps/1217045/saml-single-sign-on-sso-bitbucket?hosting=server&tab=support). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

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

В этом разделе описано, как включить единый вход Azure для пользователя B. Simon, предоставив этому пользователю доступ к SAML SSO for Bitbucket by resolution GmbH.

1. На портале Azure выберите **Корпоративные приложения** > **Все приложения**.
1. Из списка приложений выберите **SAML SSO for Bitbucket by resolution GmbH**.
1. На странице сводных сведений о приложении откройте раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**. Затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** из списка пользователей. Затем нажмите кнопку **Выбрать** в нижней части экрана.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** выберите **Назначить**.

## <a name="configure-saml-sso-for-bitbucket-by-resolution-gmbh-sso"></a>Настройка единого входа в SAML SSO for Bitbucket by resolution GmbH

1. Войдите на свой корпоративный сайт SAML SSO for Bitbucket by resolution GmbH с правами администратора.

2. Справа от главной панели инструментов щелкните **Параметры**.

3. В разделе "Учетные записи" в строке меню щелкните **SAML SingleSignOn**.

    ![Samlsingle](./media/bitbucket-tutorial/tutorial_bitbucket_samlsingle.png)

4. На **странице конфигурации подключаемого модуля SAML SIngleSignOn** щелкните **Add idp** (Добавить поставщик удостоверений). 

    ![Добавление поставщика удостоверений](./media/bitbucket-tutorial/tutorial_bitbucket_addidp.png)

5. На странице **Choose your SAML Identity Provider** (Выбор поставщика удостоверений SAML) выполните следующие действия:

    ![Поставщик удостоверений](./media/bitbucket-tutorial/tutorial_bitbucket_identityprovider.png)

    а. Выберите **Azure AD** в качестве **типа поставщика удостоверений**.

    b. В текстовом поле **Имя** введите имя.

    c. В текстовом поле **Описание** введите описание.

    d. Щелкните **Далее**.

6. На странице **Identity provider configuration page** (Настройка поставщика удостоверений) нажмите кнопку **Далее**.

    ![Настройка удостоверения](./media/bitbucket-tutorial/tutorial_bitbucket_identityconfig.png)

7.  На странице **импорта метаданных SAML поставщика удостоверений** щелкните **Загрузить файл**, чтобы передать файл **метаданных в формате XML**, скачанный на портале Azure.

    ![idpmetadata](./media/bitbucket-tutorial/tutorial_bitbucket_idpmetadata.png)

8. Щелкните **Далее**.

9. Нажмите кнопку **Сохранить параметры**.

    ![Сохранение](./media/bitbucket-tutorial/tutorial_bitbucket_save.png)


## <a name="create-saml-sso-for-bitbucket-by-resolution-gmbh-test-user"></a>Создание тестового пользователя в SAML SSO for Bitbucket by resolution GmbH

Цель этого раздела — создать в приложении SAML SSO for Bitbucket by resolution GmbH пользователя с именем Britta Simon. SAML SSO for Bitbucket by resolution GmbH обеспечивает своевременную подготовку, а также создание пользователей вручную. Для согласования ваших требований обратитесь в [службу поддержки клиентов SAML SSO for Bitbucket by resolution GmbH](https://marketplace.atlassian.com/plugins/com.resolution.atlasplugins.samlsso-bitbucket/server/support).

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

#### <a name="sp-initiated"></a>Инициация поставщиком услуг:

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в SAML SSO for Bitbucket by resolution GmbH, где можно инициировать поток входа.  

* Перейдите по URL-адресу для входа в SAML SSO for Bitbucket by resolution GmbH и инициируйте поток входа.

#### <a name="idp-initiated"></a>Вход, инициированный поставщиком удостоверений

* На портале Azure выберите **Тестировать приложение**, и вы автоматически войдете в приложение SAML SSO for Bitbucket by resolution GmbH, для которого настроили единый вход.

Вы можете также использовать портал "Мои приложения" корпорации Майкрософт для тестирования приложения в любом режиме. Щелкнув плитку SAML SSO for Bitbucket by resolution GmbH на портале "Мои приложения", вы будете перенаправлены на страницу входа приложения для инициации потока входа (при настройке в режиме поставщика услуг) или автоматически войдете в приложение SAML SSO for Bitbucket by resolution GmbH, для которого настроен единый вход (при настройке в режиме поставщика удостоверений). Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Дальнейшие действия

После настройки SAML SSO for Bitbucket by resolution GmbH вы можете применить функцию управления сеансами, которая защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа в реальном времени. Элементы управления сеансом являются расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).