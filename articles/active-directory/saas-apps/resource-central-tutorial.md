---
title: Руководство по интеграции единого входа Azure Active Directory с Resource Central | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Resource Central.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/13/2021
ms.author: jeedes
ms.openlocfilehash: dbc148fcbcd9c3be86a29df1e08755611a347b07
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100586522"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-resource-central"></a>Руководство по интеграции единого входа Azure Active Directory с Resource Central

В этом учебнике описано, как интегрировать Resource Central с Azure Active Directory (Azure AD). Интеграция Resource Central с Azure AD обеспечивает следующие возможности:

* Контроль доступа к Resource Central с помощью Azure AD.
* Автоматический вход пользователей в Resource Central с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Resource Central с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Resource Central поддерживает единый вход, инициированный **поставщиком служб**

* Resource Central поддерживает **JIT-подготовку** пользователей

## <a name="add-resource-central-from-the-gallery"></a>Добавление Resource Central из коллекции

Чтобы настроить интеграцию Resource Central с Azure AD, необходимо добавить Resource Central из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Resource Central**.
1. Выберите **Resource Central** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-resource-central"></a>Настройка и проверка единого входа Azure AD для Resource Central

Настройте и проверьте единый вход Azure AD в Resource Central с помощью тестового пользователя **B. Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Resource Central.

Чтобы настроить и проверить единый вход Azure AD в Resource Central, выполните действия из следующих стандартных блоков:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
    1. **[Создание тестового пользователя Resource Central](#create-resource-central-test-user)** требуется для того, чтобы в Resource Central существовал пользователь B. Simon, связанный с одноименным пользователем в Azure AD.
1. **[Настройка единого входа в Resource Central](#configure-resource-central-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Resource Central** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Базовая конфигурация SAML** введите значения для следующих полей:

   1. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<DOMAIN_NAME>/ResourceCentral`.

   1. В текстовое поле **Идентификатор (идентификатор сущности)** введите URL-адрес в следующем формате: `https://<DOMAIN_NAME>/ResourceCentral`.

   1. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<DOMAIN_NAME>/ResourceCentral/ExAuth/Saml2Authentication/Acs`.

    > [!NOTE]
    > Эти значения приведены в качестве примера. Укажите вместо них фактические значения URL-адреса для входа, идентификатора и URL-адреса ответа. Чтобы получить эти значения, обратитесь к [группе поддержки клиентов Resource Central](mailto:st@aod.vn).  Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите **Сертификат (Base64)** и щелкните **Скачать**, чтобы скачать сертификат. Сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

1. Скопируйте требуемый URL-адрес из раздела **Настройка Resource Central**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD

В этом разделе описано, как создать на портале Azure тестового пользователя с именем B.Simon.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.
1. В верхней части экрана выберите **Новый пользователь**.
1. В разделе **Свойства пользователя** выполните следующие действия.
   1. В поле **Имя** введите `B.Simon`.  
   1. В поле **Имя пользователя** введите `username@companydomain.extension`. Например, `B.Simon@contoso.com`.
   1. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле **Пароль**.
   1. Нажмите кнопку **Создать**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как включить единый вход Azure для пользователя B. Simon, предоставив этому пользователю доступ к Resource Central.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Resource Central**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя** и в области **Добавление назначения** выберите **Пользователи и группы**.
1. В области **Пользователи и группы** выберите **B.Simon** в списке **Пользователи**, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее в списке **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль **Доступ по умолчанию**.
1. В области **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-resource-central-test-user"></a>Создание тестового пользователя в Resource Central

В этом разделе вы создадите в **Resource Central** пользователя **B.Simon**.

1. В Resource Central выберите **Security** > **Persons** > **New** (Безопасность > Пользователи > Создать).
  
    :::image type="content" source="./media/resource-central/new-person.png" alt-text="Снимок экрана: область Persons (Пользователи) в Resource Central с выделенной кнопкой New (Создать).":::

1. В области **Person Details** (Сведения о пользователе) для параметра **Display name** (Отображаемое имя) укажите **B.Simon**. Для параметра **SMTP Address** (Адрес SMTP) укажите имя пользователя Azure AD. Например, `B.Simon@contoso.com`.

    :::image type="content" source="./media/resource-central/person.png" alt-text="Снимок экрана: область Person Details (Сведения о пользователе) в Resource Central.":::

## <a name="configure-resource-central-sso"></a>Настройка единого входа в Resource Central

В этом разделе показано, как настроить единый вход в **Resource Central System Administrator**.

1. В Resource Central System Administrator выберите **External Authentication** (Внешняя аутентификация).
1.  В разделе **Enable Configuration** (Включить конфигурацию) выберите **Yes** (Да).

    ![Снимок экрана: параметр Enable Configuration (Включить конфигурацию) в области External Authentication (Внешняя аутентификация) в Resource Central.](./media/resource-central/enable.png)

1. В поле **Authentication Protocol** (Протокол аутентификации) выберите **SAML2**. 

   :::image type="content" source="./media/resource-central/protocol.png" alt-text="Снимок экрана: вариант SAML2, выбранный в поле Authentication Protocol (Протокол аутентификации) в Resource Central.":::

1. В разделе **SAML2 Configuration** (Конфигурация SAML2) введите значения для следующих полей:

    1. В поля **Identifier (Entity ID)** (Идентификатор (идентификатор сущности)), **Login URL** (URL-адрес входа), **Logout URL** (URL-адрес выхода) и **Azure AD Identifier** (Идентификатор Azure AD) введите соответствующие URL-адреса:

       :::image type="content" source="./media/resource-central/auth.png" alt-text="Снимок экрана: область SAML2 Configuration (Конфигурация SAML2) в Resource Central.":::

        Скопируйте URL-адреса из области **Set up Resource Central** (Настройка Resource Central):

        :::image type="content" source="./media/resource-central/setup.png" alt-text="Снимок экрана: область Set up Resource Central (Настройка Resource Central) в Resource Central.":::

   1. Для параметра **URL-адрес ответа** введите `https://<DOMAIN_NAME>/ResourceCentral/ExAuth/Saml2Authentication/CallbackHandler`.
  
1. Для параметра **Certificate** (Сертификат) загрузите свой сертификат и введите пароль.

   ![Снимок экрана: раздела Certificate (Сертификат) в Resource Central.](./media/resource-central/cert.png)
   
1. Нажмите кнопку **Сохранить**.

1. Вернитесь на **портал Azure**. Для параметра **SAML Signing Certificate** (Сертификат для подписи SAML) загрузите свой сертификат и введите пароль.

   ![Снимок экрана: область Import Certificate (Импорт сертификата) на портале Azure.](./media/resource-central/cert2.png).

1. Выберите **Добавить**.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD. Чтобы проверить единый вход, у вас есть три варианта:

* На портале Azure выберите **Проверить это приложение**. Вы будете перенаправлены по URL-адресу для входа в Resource Central, где можно инициировать поток входа.

* Перейдите по URL-адресу для входа в Resource Central и инициируйте вход.

   :::image type="content" source="./media/resource-central/test.png" alt-text="Снимок экрана: веб-страница проверки единого входа в Resource Central.":::

* Используйте портал "Мои приложения" от корпорации Майкрософт. Щелкнув плитку **Resource Central** на портале "Мои приложения", вы перейдете по URL-адресу для входа в Resource Central. Дополнительные сведения см. в статье о [входе на портал "Мои приложения" и запуске приложений на нем](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Дальнейшие действия

После настройки единого входа Azure AD в Resource Central вы можете применить функцию управления сеансами, которое в реальном времени защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
