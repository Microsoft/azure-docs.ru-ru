---
title: Руководство по Интеграция Azure Active Directory с Mimecast Personal Portal | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Mimecast Personal Portal.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/15/2021
ms.author: jeedes
ms.openlocfilehash: cc12afb9d4a889d53689421a98b522a8e3264da6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101644618"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-mimecast-personal-portal"></a>Руководство по интеграции единого входа Azure Active Directory с Mimecast Personal Portal

В этом учебнике описано, как интегрировать Mimecast Personal Portal с Azure Active Directory (Azure AD). Интеграция Mimecast Personal Portal с Azure AD обеспечивает следующие возможности:

* Контроль доступа к Mimecast Personal Portal с помощью Azure AD.
* Автоматический вход пользователей в Mimecast Personal Portal с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Mimecast Personal Portal с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Mimecast Personal Portal поддерживает единый вход, инициированный **поставщиком услуг и поставщиком удостоверений**.
 
## <a name="add-mimecast-personal-portal-from-the-gallery"></a>Добавление Mimecast Personal Portal из коллекции

Чтобы настроить интеграцию Mimecast Personal Portal с Azure AD, необходимо добавить Mimecast Personal Portal из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Mimecast Personal Portal**.
1. Выберите **Mimecast Personal Portal** на панели результатов, а затем добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-mimecast-personal-portal"></a>Настройка и проверка единого входа Azure AD для Mimecast Personal Portal

Настройте и проверьте единый вход Azure AD в Mimecast Personal Portal с помощью тестового пользователя **B. Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Mimecast Personal Portal.

Чтобы настроить и проверить единый вход Azure AD в Mimecast Personal Portal, выполните следующие действия.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Mimecast Personal Portal](#configure-mimecast-personal-portal-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя Mimecast Personal Portal](#create-mimecast-personal-portal-test-user)** требуется для того, чтобы в Mimecast Personal Portal существовал пользователь B. Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Mimecast Personal Portal** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. Если вы хотите настроить приложение в режиме, инициируемом поставщиком удостоверений, в разделе **Базовая конфигурация SAML** выполните следующие действия:

    а. В текстовом поле **Идентификатор** введите URL-адрес в следующем формате: .

    | Регион  |  Значение | 
    | --------------- | --------------- |
    | Европа          | `https://eu-api.mimecast.com/sso/<accountcode>`|
    | США   | `https://us-api.mimecast.com/sso/<accountcode>`|
    | ЮАР    | `https://za-api.mimecast.com/sso/<accountcode>`|
    | Австралия       | `https://au-api.mimecast.com/sso/<accountcode>`|
    | Внешнее расположение        | `https://jer-api.mimecast.com/sso/<accountcode>`|

    > [!NOTE]
    > Значение `accountcode` можно найти в Mimecast Personal Portal в разделе **Account** > **Settings** > **Account Code** (Учетная запись > Параметры > Код учетной записи). Добавьте `accountcode` к идентификатору.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес .

    | Регион  |  Значение |
    | --------------- | --------------- |
    | Европа          | `https://eu-api.mimecast.com/login/saml`|
    | США   | `https://us-api.mimecast.com/login/saml`|
    | ЮАР    | `https://za-api.mimecast.com/login/saml`|
    | Австралия       | `https://au-api.mimecast.com/login/saml`|
    | Внешнее расположение        | `https://jer-api.mimecast.com/login/saml`|

1. если хотите настроить приложение для работы в режиме, инициируемом **поставщиком услуг**.

    В текстовом поле **URL-адрес для входа** введите URL-адрес: 

    | Регион  |  Значение |
    | --------------- | --------------- |
    | Европа          | `https://eu-api.mimecast.com/login/saml`|
    | США   | `https://us-api.mimecast.com/login/saml`|
    | ЮАР    | `https://za-api.mimecast.com/login/saml`|
    | Австралия       | `https://au-api.mimecast.com/login/saml`|
    | Внешнее расположение        | `https://jer-api.mimecast.com/login/saml`|

1. Выберите команду **Сохранить**.

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

В этом разделе описано, как включить единый вход Azure для пользователя B. Simon, предоставив этому пользователю доступ к Mimecast Personal Portal.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. Из списка приложений выберите **Mimecast Personal Portal**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-mimecast-personal-portal-sso"></a>Настройка единого входа Mimecast Personal Portal

1. В другом окне веб-браузера войдите в консоль администрирования Mimecast.

1. Перейдите в раздел **Administration** > **Services** > **Applications** (Администрирование > Службы > Приложения).

    ![Снимок экрана: окно Mimecast с выбранным пунктом Applications (Приложения)](./media/mimecast-personal-portal-tutorial/services.png)

1. Выберите вкладку **Authentication Profiles** (Профили проверки подлинности).
    
    ![Снимок экрана: вкладка Authentication (Аутентификация) с выбранным параметром Authentication Profiles (Профили аутентификации)](./media/mimecast-personal-portal-tutorial/authentication-profiles.png)

1. Выберите вкладку **New Authentication Profile** (Новый профиль проверки подлинности).

    ![Снимок экрана: вкладка New Authentication Profile (Новый профиль аутентификации)](./media/mimecast-personal-portal-tutorial/new-authenticatio-profile.png)

1. Укажите допустимое описание в текстовом поле **Description** (Описание) и установите флажок **Enforce SAML Authentication for Mimecast Personal Portal** (Принудительная проверка подлинности SAML для Mimecast Personal Portal).

    ![Снимок экрана: вкладка New Authentication Profile (Новый профиль аутентификации)](./media/mimecast-personal-portal-tutorial/selecting-personal-portal.png)

1. На странице **SAML Configuration for Administration Console** (Настройка SAML для Mimecast Personal Portal) сделайте следующее:

    ![Снимок экрана: флажок Enforce SAML Authentication for Administration Console (Принудительная проверка подлинности SAML для консоли администрирования)](./media/mimecast-personal-portal-tutorial/sso-settings.png)

    а. Для **Provider** (Поставщик) выберите **Azure Active Directory**.

    b. В текстовое поле **Metadata URL** (URL-адрес метаданных) вставьте значение **URL-адреса метаданных федерации приложения**, скопированное с портала Azure.

    c. Щелкните **Импорт**. После импорта URL-адреса метаданных поля будут заполнены автоматически, дополнительные действия с ними не требуются.

    d. Не забудьте снять флажки **Use Password protected Context** (Использовать защищенный паролем контекст) и **Use Integrated Authentication Context** (Использовать интегрированный контекст проверки подлинности).

    д) Выберите команду **Сохранить**.

### <a name="create-mimecast-personal-portal-test-user"></a>Создание тестового пользователя Mimecast Personal Portal

1. В другом окне веб-браузера войдите в консоль администрирования Mimecast.

1. Перейдите к разделу **Administration** > **Directories** > **Internal Directories** (Администрирование > Каталоги > Внутренние каталоги).

    ![Снимок экрана: страница SAML Configuration for Mimecast Personal Portal (Настройка SAML для персонального портала Mimecast), где можно ввести описанные здесь значения](./media/mimecast-personal-portal-tutorial/internal-directories.png)

1. Выберите домен, если он указан ниже. В противном случае создайте новый домен с помощью ссылки **New Domain** (Новый домен).

    ![Снимок экрана: окно Mimecast с выбранным пунктом Internal Directories (Внутренние каталоги)](./media/mimecast-personal-portal-tutorial/domain-name.png)

1. Щелкните вкладку **New Address** (Новый адрес).

    ![Снимок экрана: выбранный домен](./media/mimecast-personal-portal-tutorial/new-address.png)

1. Введите обязательные сведения о пользователе на следующей странице:

    ![Снимок экрана: страница для ввода описанных значений](./media/mimecast-personal-portal-tutorial/user-information.png)

    а. В текстовом поле **Email address** (Адрес электронной почты) введите адрес электронной почты пользователя, например `B.Simon@yourdomainname.com`.

    b. В текстовом поле **GLobal Name** (Глобальное имя) введите **полное имя** пользователя.

    c. В текстовые поля **Password** (Пароль) и **Confirm Password** (Подтверждение пароля) введите пароль пользователя.

    d. Установите флажок **Force Change at Login** (Потребовать смену при входе).

    д) Выберите команду **Сохранить**.

    е) Чтобы назначить пользователю роли, щелкните **Role Edit** (Изменить роли) и выберите роль, которая лучше всего соответствует требованиям вашей организации.

    ![Снимок экрана: раздел Address Settings (Параметры адреса) с пунктом Role Edit (Изменить роли)](./media/mimecast-personal-portal-tutorial/assign-role.png)


## <a name="test-sso"></a>Проверка единого входа 
В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

#### <a name="sp-initiated"></a>Инициация поставщиком услуг:

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в Mimecast Personal Portal, где можно инициировать поток входа.  

* Перейдите по URL-адресу для входа в Mimecast Personal Portal и инициируйте поток входа.

#### <a name="idp-initiated"></a>Вход, инициированный поставщиком удостоверений

* На портале Azure выберите элемент **Тестировать приложение**, и вы автоматически войдете в приложение Mimecast Personal Portal, для которого настроили единый вход 

Вы можете также использовать портал "Мои приложения" корпорации Майкрософт для тестирования приложения в любом режиме. Щелкнув плитку Mimecast Personal Portal на портале "Мои приложения", вы автоматически перейдете на страницу входа приложения для инициации потока входа (при настройке в режиме поставщика службы) или в приложение Mimecast Personal Portal, для которого настроен единый вход (при настройке в режиме поставщика удостоверений). Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Дальнейшие действия

После настройки Mimecast Personal Portal вы можете применить функцию управления сеансом, которая защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа в реальном времени. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).