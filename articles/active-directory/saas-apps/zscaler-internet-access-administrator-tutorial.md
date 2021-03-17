---
title: Руководство. Интеграция Azure Active Directory с Zscaler Internet Access Administrator | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Zscaler Internet Access Administrator.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/25/2021
ms.author: jeedes
ms.openlocfilehash: 70afa0a02f4e303105aec1884b966796854c6f49
ms.sourcegitcommit: f6193c2c6ce3b4db379c3f474fdbb40c6585553b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2021
ms.locfileid: "102449329"
---
# <a name="tutorial-azure-active-directory-integration-with-zscaler-internet-access-administrator"></a>Руководство. Интеграция Azure Active Directory с Zscaler Internet Access Administrator

В этом руководстве описано, как интегрировать Zscaler Internet Access Administrator с Azure Active Directory (Azure AD). Интеграция Zscaler Internet Access Administrator с Azure AD обеспечивает следующие возможности:

* Контроль доступа к Zscaler Internet Access Administrator с помощью Azure AD.
* Включение автоматического входа пользователей в Zscaler Internet Access Administrator с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Zscaler Internet Access Administrator с поддержкой единого входа.

> [!NOTE]
> Эту интеграцию также можно использовать в облачной среде Azure AD для государственных организаций США. Это приложение можно найти в коллекции облачных приложений с поддержкой Azure AD для государственных организаций США и настроить таким же образом, как и в общедоступном облаке.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Zscaler Internet Access Administrator поддерживает единый вход, инициированный **поставщиком удостоверений**.

## <a name="add-zscaler-internet-access-administrator-from-the-gallery"></a>Добавление Zscaler Internet Access Administrator из коллекции

Чтобы настроить интеграцию Zscaler Internet Access Administrator с Azure Active Directory, необходимо добавить Zscaler Internet Access Administrator из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Zscaler Internet Access Administrator**.
1. Выберите **Zscaler Internet Access Administrator** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-zscaler-internet-access-administrator"></a>Настройка и проверка единого входа Azure AD для Zscaler Internet Access Administrator

Настройте и проверьте единый вход Azure AD в Zscaler Internet Access Administrator с помощью тестового пользователя **B. Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Zscaler Internet Access Administrator.

Чтобы настроить и протестировать единый вход Azure AD в Zscaler Internet Access Administrator, выполните следующие действия:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
2. **[Настройка единого входа в Zscaler Internet Access Administrator](#configure-zscaler-internet-access-administrator-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя Zscaler Internet Access Administrator](#create-zscaler-internet-access-administrator-test-user)** требуется, чтобы в Zscaler Internet Access Administrator существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure Active Directory.
3. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Zscaler Internet Access Administrator** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовое поле **Идентификатор** введите один из следующих URL-адресов согласно требованиям:

    | Идентификатор |
    |------------|
    | `https://admin.zscaler.net` |
    | `https://admin.zscalerone.net` |
    | `https://admin.zscalertwo.net` |
    | `https://admin.zscalerthree.net` |
    | `https://admin.zscloud.net` |
    | `https://admin.zscalerbeta.net` |

    b. В текстовое поле **URL-адрес ответа** введите один из следующих URL-адресов согласно требованиям:

    | URL-адрес ответа |
    |-----------|
    | `https://admin.zscaler.net/adminsso.do` |
    | `https://admin.zscalerone.net/adminsso.do` |
    | `https://admin.zscalertwo.net/adminsso.do` |
    | `https://admin.zscalerthree.net/adminsso.do` |
    | `https://admin.zscloud.net/adminsso.do` |
    | `https://admin.zscalerbeta.net/adminsso.do` |

5. Приложение Zscaler Internet Access Administrator ожидает утверждения SAML в определенном формате. Настройте следующие утверждения для этого приложения. Управлять значениями этих атрибутов можно в разделе **User Attributes & Claims** (Атрибуты пользователя и утверждения) на странице интеграции приложения. На странице **Настройка единого входа с помощью SAML** нажмите кнопку **Изменить**, чтобы открыть диалоговое окно **User Attributes & Claims** (Атрибуты пользователя и утверждения).

    ![Ссылка "Атрибуты"](./media/zscaler-internet-access-administrator-tutorial/attributes.png)

6. В разделе **Утверждения пользователя** диалогового окна **Атрибуты пользователя** настройте атрибут токена SAML, как показано на рисунке выше, и выполните следующие действия.

    | Имя  | Исходный атрибут  |
    | ---------| ------------ |
    | Роль | user.assignedroles |

    а. Щелкните **Добавить новое утверждение**, чтобы открыть диалоговое окно **Управление утверждениями пользователя**.

    b. Из списка **Атрибут источника** выберите значение атрибута.

    c. Нажмите кнопку **ОК**.

    d. Выберите команду **Сохранить**.

    > [!NOTE]
    > Перейдите по [этой ссылке](../develop/howto-add-app-roles-in-azure-ad-apps.md#app-roles-ui--preview), чтобы прочитать о настройке роли в Azure AD.

7. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (Base64)** из предложенных вариантов, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

8. Скопируйте требуемый URL-адрес из раздела **Настройка Zscaler Internet Access Administrator**.

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

В этом разделе описано, как разрешить пользователю B. Simon использовать единый вход Azure, предоставив этому пользователю доступ к Zscaler Internet Access Administrator.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Zscaler Internet Access Administrator**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если вы настроили роли, как описано выше, вы можете выбрать роль из раскрывающегося списка **Выберите роль**.
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-zscaler-internet-access-administrator-sso"></a>Настройка единого входа в Zscaler Internet Access Administrator

1. В другом окне веб-браузера войдите в пользовательский интерфейс администратора Zscaler Internet Access.

2. Выберите **Administration > Administrator Management** (Администрирование) > Управление администраторами), выполните следующие действия и щелкните Save (Сохранить):

    ![Снимок экрана, на котором показана вкладка управления администраторами с параметрами для включения проверки подлинности SAML, отправки сертификата SSL и указания издателя.](./media/zscaler-internet-access-administrator-tutorial/management.png "Администрирование")

    а. Установите флажок **Enable SAML Authentication** (Включить аутентификацию SAML).

    b. Нажмите **Upload** (Отправить), чтобы загрузить сертификат для подписи SAML, который вы скачали на портале Azure в разделе **Public SSL Certificate** (Публичный SSL-сертификат).

    c. При необходимости для обеспечения дополнительной безопасности добавьте сведения об **издателе** для проверки издателя ответа SAML.

3. В пользовательском интерфейсе администратора выполните следующие действия:

    ![Снимок экрана, на котором показан пользовательский интерфейс администратора, где выполняются эти действия.](./media/zscaler-internet-access-administrator-tutorial/activation.png)

    а. Наведите указатель мыши на меню **Activation** (Активация) в нижнем левом углу.

    b. Нажмите кнопку **Активировать**.

### <a name="create-zscaler-internet-access-administrator-test-user"></a>Создание тестового пользователя Zscaler Internet Access Administrator

Цель этого раздела — создать пользователя с именем Britta Simon в Zscaler Internet Access Administrator. Zscaler Internet Access не поддерживает JIT-подготовку для единого входа администратора. Необходимо вручную создать учетную запись администратора.
Инструкции по созданию учетной записи администратора см. в документации по Zscaler:

https://help.zscaler.com/zia/adding-admins

## <a name="test-sso"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов.

* На портале Azure выберите "Тестировать приложение" и вы автоматически войдете в приложение Zscaler Internet Access Administrator, для которого настроен единый вход.

* Вы можете использовать портал "Мои приложения" корпорации Майкрософт. Щелкнув плитку Zscaler Internet Access Administrator на портале "Мои приложения", вы автоматически войдете в приложение Zscaler Internet Access Administrator, для которого настроили единый вход. Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Дальнейшие действия

После настройки Zscaler Internet Access Administrator вы можете применить управление сеансами, которое в реальном времени защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
