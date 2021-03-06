---
title: Руководство по Интеграция Azure Active Directory с Aha! | Документы Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Aha!.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/20/2021
ms.author: jeedes
ms.openlocfilehash: a39371e7a22334b11be1d1a0a9d28557efe177c6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101654448"
---
# <a name="tutorial-integrate-aha-with-azure-active-directory"></a>Руководство по Интеграция Aha! с Azure Active Directory

В этом руководстве вы узнаете, как интегрировать Aha! с Azure Active Directory (Azure AD). Интеграция Aha! с Azure AD обеспечивает приведенные ниже возможности:

* С помощью Azure AD вы можете контролировать доступ к Aha!.
* Разрешить автоматический вход пользователей в Aha! с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Aha! Подписка Aha! с поддержкой единого входа.

> [!NOTE]
> Эту интеграцию также можно использовать в облачной среде Azure AD для государственных организаций США. Это приложение можно найти в коллекции облачных приложений с поддержкой Azure AD для государственных организаций США и настроить таким же образом, как и в общедоступном облаке.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Aha! поддерживает единый вход, инициированный **поставщиком услуг**.
* Aha! поддерживает **JIT**-подготовку пользователей.

## <a name="add-aha-from-the-gallery"></a>Добавление Aha! из коллекции

Чтобы настроить интеграцию Aha! с Azure AD, необходимо добавить Aha! из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** введите **Aha!** в поле поиска.
1. Выберите **Aha!** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-aha"></a>Настройка и проверка единого входа Azure AD для Aha!

Настройка и проверка единого входа Azure AD в Aha! с помощью тестового пользователя с именем **B.Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Aha!.

Чтобы настроить и проверить единый вход Azure AD в Aha!, выполните следующие действия:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
2. **[Настройка единого входа в Aha!](#configure-aha-sso)** необходима для настройки параметров единого входа на стороне приложения.
    1. **[Создание тестового пользователя Aha!](#create-aha-test-user)** требуется для создания в Aha! пользователя B.Simon, связанного с соответствующим представлением в Azure AD.
3. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Aha!** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<companyname>.aha.io/session/new`.

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `https://<companyname>.aha.io`.

    > [!NOTE]
    > Эти значения приведены для примера. Необходимо обновить эти значения действующим URL-адресом для входа и идентификатором. Обратитесь к [группе поддержки Aha!](https://www.aha.io/company/contact), чтобы получить эти значения. Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

4. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите элемент **XML метаданных федерации** и нажмите кнопку **Скачать**, чтобы скачать сертификат и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

6. Требуемый URL-адрес можно скопировать из раздела **Настройка Aha!** .

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

В этом разделе описано, как включить единый вход в Azure для пользователя B.Simon, предоставив этому пользователю доступ к Aha!.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. Из списка приложений выберите **Aha!** .
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-aha-sso"></a>Настройка Aha! Единый вход

1. Для автоматизации настройки в Aha! необходимо установить **Расширение браузера "Безопасный вход в мои приложения"** , щелкнув **Установить расширение**.

    ![Расширение "Мои приложения"](common/install-myappssecure-extension.png)

2. После добавления расширения в браузер, щелкнув **Установить AHA!** , вы будете направленны к приложении Aha!. После этого укажите учетные данные администратора для входа в Aha!. Расширение браузера автоматически настроит приложение и автоматизирует шаги 3–8.

    ![Настройка конфигурации](common/setup-sso.png)

3. Если вы хотите настроить AHA! вручную, откройте новое окно веб-браузера и войдите на AHA! сайт компании в качестве администратора и выполните следующие действия:

4. В верхнем меню нажмите пункт **Параметры**.

    ![Параметры](./media/aha-tutorial/setting.png "Параметры")

5. Выберите раздел **Учетная запись**.

    ![Профиль](./media/aha-tutorial/account.png "Профиль")

6. Нажмите **Безопасность и единый вход**.

    ![Снимок экрана, на котором показан пункт меню "Security and single sign-on" (Безопасность и единый вход)](./media/aha-tutorial/security.png "Безопасность и единый вход")

7. В разделе **Единый вход** в качестве **поставщика удостоверений** выберите **SAML 2.0**.

    ![Безопасность и единый вход](./media/aha-tutorial/saml.png "Безопасность и единый вход")

8. На странице настроек **Единый вход** выполните следующие действия.

    ![Единый вход](./media/aha-tutorial/sso.png "Единый вход")

    а. В текстовом поле **Имя** введите имя конфигурации.

    b. Для параметра **Configure using** (Использовать при настройке) выберите значение **Файл метаданных**.

    c. Чтобы отправить загруженный файл метаданных, нажмите кнопку **Обзор**.

    d. Нажмите кнопку **Обновить**.

### <a name="create-aha-test-user"></a>Создание тестового пользователя Aha!

В этом разделе описано, как в приложении Aha! создать пользователя с именем B.Simon. Aha! поддерживает JIT-подготовку пользователей, которая включена по умолчанию. В этом разделе никакие действия с вашей стороны не требуются. Если пользователь еще не существует в Aha!, он создается после проверки подлинности.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в Aha!, где можно инициировать поток входа. 

* Перейдите по URL-адресу для входа в Aha! и инициируйте поток входа.

* Вы можете использовать портал "Мои приложения" корпорации Майкрософт. Щелкнув плитку Aha! на портале "Мои приложения", вы перейдете по ULR-адресу для входа в Aha! URL-адрес входа. Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Дальнейшие действия

После настройки Aha! вы можете применить управление сеансами, которое защищает от хищения конфиденциальных данных вашей организации и несанкционированного доступа к ним в реальном времени. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).