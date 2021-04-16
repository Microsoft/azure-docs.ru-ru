---
title: Руководство по Интеграция Azure Active Directory с MVISION Cloud Azure AD SSO Configuration | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и MVISION Cloud Azure AD SSO Configuration.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/31/2021
ms.author: jeedes
ms.openlocfilehash: eaf6b2526125b13eec67680c292ed1dbae6fcfee
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106284436"
---
# <a name="tutorial-integrate-mvision-cloud-azure-ad-sso-configuration-with-azure-active-directory"></a>Руководство по Интеграция MVISION Cloud Azure AD SSO Configuration с Azure Active Directory

В этом руководстве объясняется, как интегрировать MVISION Cloud Azure AD SSO Configuration с Azure Active Directory (Azure AD). Интеграция MVISION Cloud Azure AD SSO Configuration с Azure AD предоставит следующие возможности:

* Вы сможете через Azure AD управлять доступом к MVISION Cloud Azure AD SSO Configuration.
* Вы позволите пользователям автоматически входить в MVISION Cloud Azure AD SSO Configuration с учетными записями Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка MVISION Cloud Azure AD SSO Configuration с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* MVISION Cloud Azure AD SSO Configuration поддерживает единый вход, инициированный **поставщиком услуг и поставщиком удостоверений**.

## <a name="add-mvision-cloud-azure-ad-sso-configuration-from-the-gallery"></a>Добавление MVISION Cloud Azure AD SSO Configuration из коллекции

Чтобы настроить интеграцию MVISION Cloud Azure AD SSO Configuration с Azure AD, необходимо добавить MVISION Cloud Azure AD SSO Configuration из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** введите **MVISION Cloud Azure AD SSO Configuration** в поле поиска.
1. Выберите **MVISION Cloud Azure AD SSO Configuration** на панели результатов, а затем добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-mvision-cloud-azure-ad-sso-configuration"></a>Настройка и тестирование единого входа Azure AD в MVISION Cloud Azure AD SSO Configuration

Настройте и проверьте единый вход Azure AD в MVISION Cloud Azure AD SSO Configuration с помощью тестового пользователя с именем **Britta Simon**. Чтобы обеспечить единый вход, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в MVISION Cloud Azure AD SSO Configuration.

Чтобы настроить и проверить единый вход в MVISION Cloud Azure AD SSO Configuration, выполните следующие действия:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
    4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в MVISION Cloud Azure AD SSO Configuration](#configure-mvision-cloud-azure-ad-sso-configuration-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя MVISION Cloud Azure AD SSO Configuration](#create-mvision-cloud-azure-ad-sso-configuration-test-user)** требуется для того, чтобы в MVISION Cloud Azure AD SSO Configuration существовал пользователь Britta Simon, связанный с представлением этого пользователя в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

### <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Datadog** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** выполните следующие действия.

    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `https://<ENV>.myshn.net/shndash/saml/Azure_SSO`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<ENV>.myshn.net/shndash/response/saml-postlogin`.

5. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://<ENV>.myshn.net/shndash/saml/Azure_SSO`.

    > [!NOTE]
    > Эти значения приведены для примера. Замените их фактическими значениями идентификатора, URL-адреса ответа и URL-адреса входа. Чтобы получить эти значения, обратитесь в [службу технической поддержки клиентов MVISION Cloud Azure AD SSO Configuration](mailto:support@skyhighnetworks.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

6. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (Base64)** из предложенных вариантов, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

7. Скопируйте требуемые URL-адреса из раздела **настройки MVISION Cloud Azure AD SSO Configuration**.

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

В этом разделе объясняется, как разрешить пользователю B.Simon использовать единый вход Azure, предоставив доступ к MVISION Cloud Azure AD SSO Configuration.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **MVISION Cloud Azure AD SSO Configuration**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-mvision-cloud-azure-ad-sso-configuration-sso"></a>Настройка единого входа в MVISION Cloud Azure AD SSO Configuration

Чтобы настроить единый вход на стороне **MVISION Cloud Azure AD SSO Configuration**, нужно отправить скачанный **сертификат (Base64)** и соответствующие URL-адреса, скопированные с портала Azure, в [службу технической поддержки MVISION Cloud Azure AD SSO Configuration](mailto:support@skyhighnetworks.com). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-mvision-cloud-azure-ad-sso-configuration-test-user"></a>Создание тестового пользователя в MVISION Cloud Azure AD SSO Configuration

В рамках этого раздела вы создадите пользователя с именем B.Simon в MVISION Cloud Azure AD SSO Configuration. Чтобы добавить пользователей на платформу MVISION Cloud Azure AD SSO Configuration, обратитесь в [службу поддержки MVISION Cloud Azure AD SSO Configuration](mailto:support@skyhighnetworks.com). Перед использованием единого входа необходимо создать и активировать пользователей.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

#### <a name="sp-initiated"></a>Инициация поставщиком услуг:

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в MVISION Cloud Azure AD SSO Configuration, где можно инициировать поток входа.  

* Перейдите по URL-адресу для входа в MVISION Cloud Azure AD SSO Configuration и инициируйте поток входа.

#### <a name="idp-initiated"></a>Вход, инициированный поставщиком удостоверений

* На портале Azure выберите элемент **Тестировать приложение**, и вы автоматически войдете в приложение MVISION Cloud Azure AD SSO Configuration, для которого настроен единый вход. 

Вы можете также использовать портал "Мои приложения" корпорации Майкрософт для тестирования приложения в любом режиме. Щелкните плитку MVISION Cloud Azure AD SSO Configuration в разделе "Мои приложения". Вы перейдете на страницу входа приложения для инициации потока входа (при настройке в режиме поставщика службы) или автоматически войдете в приложение MVISION Cloud Azure AD SSO Configuration, для которого настроен единый вход (при настройке в режиме поставщика удостоверений). Дополнительные сведения о портале "Мои приложения" см. в [этой статье](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="next-steps"></a>Дальнейшие действия

После настройки MVISION Cloud Azure AD SSO Configuration вы можете применить функцию управления сеансами, которая в реальном времени защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).
