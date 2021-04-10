---
title: Руководство по Интеграция Azure Active Directory с Zscaler Private Access (ZPA) | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Zscaler Private Access (ZPA).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/03/2021
ms.author: jeedes
ms.openlocfilehash: 58e2a19f2d57eafc7d2967141d584dc7a22fe76c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104955675"
---
# <a name="tutorial-integrate-zscaler-private-access-zpa-with-azure-active-directory"></a>Руководство по Интеграция Zscaler Private Access (ZPA) с Azure Active Directory

В этом руководстве описано, как интегрировать Zscaler Private Access (ZPA) с Azure Active Directory (Azure AD). Интеграция Zscaler Private Access (ZPA) с Azure AD обеспечивает следующие возможности.

* С помощью Azure AD вы можете контролировать доступ к Zscaler Private Access (ZPA).
* Вы можете включить автоматический вход пользователей в Zscaler Private Access (ZPA) с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* подписка Zscaler Private Access (ZPA) с поддержкой единого входа.

> [!NOTE]
> Эту интеграцию также можно использовать в облачной среде Azure AD для государственных организаций США. Это приложение можно найти в коллекции облачных приложений с поддержкой Azure AD для государственных организаций США и настроить таким же образом, как и в общедоступном облаке.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде. 

* Zscaler Private Access (ZPA) поддерживает единый вход, инициируемый **поставщиком услуг**.

## <a name="add-zscaler-private-access-zpa-from-the-gallery"></a>Добавление Zscaler Private Access (ZPA) из коллекции

Чтобы настроить интеграцию Zscaler Private Access (ZPA) с Azure AD, необходимо добавить Zscaler Private Access (ZPA) из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Zscaler Private Access (ZPA)** .
1. Выберите **Zscaler Private Access (ZPA)** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso"></a>Настройка и проверка единого входа Azure AD

Настройте и проверьте единый вход Azure AD в Zscaler Private Access (ZPA) с помощью тестового пользователя **B. Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Zscaler Private Access (ZPA).

Чтобы настроить и проверить единый вход Azure AD в Zscaler Private Access (ZPA), выполните действия из следующих стандартных блоков:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Zscaler Private Access (ZPA)](#configure-zscaler-private-access-zpa-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя Zscaler Private Access (ZPA)](#create-zscaler-private-access-zpa-test-user)** требуется для того, чтобы в Zscaler Private Access (ZPA) существовал пользователь B. Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Zscaler Private Access (ZPA)** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Базовая конфигурация SAML** введите значения для следующих полей.

    1. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://samlsp.private.zscaler.com/auth/login?domain=<your-domain-name>`.

    1. В текстовом поле **Идентификатор (сущности)** введите URL-адрес: `https://samlsp.private.zscaler.com/auth/metadata`.

    > [!NOTE]
    > Значение **URL-адреса входа** приведено для примера. Для входа укажите фактический URL-адрес. Обратитесь к [группе поддержки клиентов Zscaler Private Access (ZPA)](https://help.zscaler.com/zpa-submit-ticket), чтобы получить это значение. Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите элемент **XML метаданных федерации** и нажмите кнопку **Скачать**, чтобы скачать сертификат и сохранить его на компьютере.

   ![Ссылка для скачивания сертификата](common/metadataxml.png)

1. Требуемые URL-адреса можно скопировать из раздела **Настройка Zscaler Private Access (ZPA)** .

   ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD

В этом разделе описано, как создать на портале Azure тестового пользователя с именем Britta Simon.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.
1. В верхней части экрана выберите **Новый пользователь**.
1. В разделе **Свойства пользователя** выполните следующие действия.
   1. В поле **Имя** введите `Britta Simon`.  
   1. В поле **Имя пользователя** введите username@companydomain.extension. Например, `BrittaSimon@contoso.com`.
   1. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле **Пароль**.
   1. Нажмите кнопку **Создать**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как разрешить пользователю B. Simon использовать единый вход Azure, предоставив этому пользователю доступ к Zscaler Private Access (ZPA).

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. Из списка приложений выберите **Zscaler Private Access (ZPA)** .
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-zscaler-private-access-zpa-sso"></a>Настройка единого входа в Zscaler Private Access (ZPA)

1. Чтобы автоматизировать настройку в Zscaler Private Access (ZPA), необходимо установить **расширение браузера My Apps Secure Sign-in Extension**, щелкнув **Установить расширение**.

    ![Расширение "Мои приложения"](common/install-myappssecure-extension.png)

2. Чтобы перейти к приложению Zscaler Private Access (ZPA) после добавления расширения в браузер, щелкните **Настройка Zscaler Private Access (ZPA)** . После этого укажите учетные данные администратора для входа в Zscaler Private Access (ZPA). Расширение браузера автоматически настроит приложение и автоматизирует шаги 3–6.

    ![Настройка конфигурации](common/setup-sso.png)

3. Если необходимо вручную настроить Zscaler Private Access (ZPA), откройте новое окно веб-браузера, зайдите на корпоративный сайт Zscaler Private Access (ZPA) с правами администратора и выполните следующие действия.

4. Слева в меню щелкните **Administration** (Администрирование) и перейдите в раздел **AUTHENTICATION** (Аутентификация). Щелкните **IdP Configuration** (Конфигурация IdP).

    ![Администрирование Zscaler Private Access](./media/zscalerprivateaccess-tutorial/administration.png)

5. В правом верхнем углу щелкните **Add IdP Configuration** (Добавить конфигурацию IdP). 

    ![Конфигурация IdP на странице администрирования Zscaler Private Access](./media/zscalerprivateaccess-tutorial/metadata.png)

6. На странице **Add IdP Configuration** (Добавление конфигурации IdP) выполните следующие действия.
 
    ![Выбор параметров на странице администрирования Zscaler Private Access](./media/zscalerprivateaccess-tutorial/select.png)

    а. Щелкните **Select File** (Выбрать файл), чтобы передать скачанный файл метаданных из Azure AD и указать его в поле **IdP Metadata File Upload** (Передача файла метаданных IdP).

    b. Будут прочитаны **метаданные IdP** из Azure AD, а также будут заполнены все поля сведений, как показано ниже.

    ![Конфигурация на странице администрирования Zscaler Private Access](./media/zscalerprivateaccess-tutorial/configure.png)

    c. Выберите домен в поле **Domains** (Домены).
    
    d. Выберите команду **Сохранить**.

### <a name="create-zscaler-private-access-zpa-test-user"></a>Создание тестового пользователя Zscaler Private Access (ZPA)

В этом разделе описано, как создать пользователя Britta Simon в приложении Zscaler Private Access (ZPA). Обратитесь за помощью к [группе поддержки Zscaler Private Access (ZPA)](https://help.zscaler.com/zpa-submit-ticket), чтобы добавить пользователей на платформе Zscaler Private Access (ZPA).

## <a name="test-sso"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в Zscaler Private Access (ZPA), где можно инициировать поток входа. 

* Перейдите по URL-адресу для входа в Zscaler Private Access (ZPA) и инициируйте поток входа.

* Вы можете использовать портал "Мои приложения" корпорации Майкрософт. Щелкнув плитку Zscaler Private Access (ZPA) на портале "Мои приложения", вы перейдете по URL-адресу для входа в Zscaler Private Access (ZPA). Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Дальнейшие действия

После настройки Zscaler Private Access (ZPA) вы можете применить функцию управления сеансом, чтобы в реальном времени защищать конфиденциальные данные своей организации от кражи и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).