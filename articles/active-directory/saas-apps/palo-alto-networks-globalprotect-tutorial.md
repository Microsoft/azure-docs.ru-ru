---
title: Руководство по интеграции единого входа Azure Active Directory c Palo Alto Networks (GlobalProtect) | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Palo Alto Networks (GlobalProtect).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/10/2020
ms.author: jeedes
ms.openlocfilehash: 72072feebfcf8dba249d2045a399e09714177698
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97963695"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-palo-alto-networks---globalprotect"></a>Руководство по интеграции единого входа Azure Active Directory c Palo Alto Networks (GlobalProtect)

В этом учебнике описано, как интегрировать Palo Alto Networks (GlobalProtect) с Azure Active Directory (Azure AD). Интеграция Palo Alto Networks (GlobalProtect) с Azure AD обеспечивает следующие возможности:

* Контроль доступа к Palo Alto Networks (GlobalProtect) с помощью Azure AD.
* Автоматический вход пользователей в Palo Alto Networks (GlobalProtect) с использованием учетной записи Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Palo Alto Networks (GlobalProtect) с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Palo Alto Networks (GlobalProtect) поддерживает единый вход, инициированный **поставщиком услуг**.
* Palo Alto Networks (GlobalProtect) поддерживает **JIT**-подготовку пользователей.

## <a name="adding-palo-alto-networks---globalprotect-from-the-gallery"></a>Добавление Palo Alto Networks (GlobalProtect) из коллекции

Чтобы настроить интеграцию Palo Alto Networks (GlobalProtect) с Azure AD, необходимо добавить Palo Alto Networks (GlobalProtect) из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** введите **Palo Alto Networks (GlobalProtect)** в поле поиска.
1. Выберите **Palo Alto Networks (GlobalProtect)** на панели результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-palo-alto-networks---globalprotect"></a>Настройка и проверка единого входа Azure AD для Palo Alto Networks (GlobalProtect)

Настройте и проверьте единый вход Azure AD в Palo Alto Networks (GlobalProtect) с помощью тестового пользователя **B. Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Palo Alto Networks (GlobalProtect).

Чтобы настроить и протестировать единый вход Azure AD для Palo Alto Networks (GlobalProtect), сделайте следующее:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Palo Alto Networks (GlobalProtect)](#configure-palo-alto-networks---globalprotect-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя Palo Alto Networks (GlobalProtect)](#create-palo-alto-networks---globalprotect-test-user)** требуется, чтобы в Palo Alto Networks (GlobalProtect) существовал пользователь B. Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Palo Alto Networks (GlobalProtect)** найдите раздел **Управление** и щелкните элемент **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<Customer Firewall URL>`.

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `https://<Customer Firewall URL>/SAML20/SP`.

    > [!NOTE]
    > Эти значения приведены для примера. Необходимо обновить эти значения действующим URL-адресом для входа и идентификатором. Для получения этих значений обратитесь в [службу поддержки клиентов Palo Alto Networks (GlobalProtect)](https://support.paloaltonetworks.com/support). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите элемент **XML метаданных федерации** и выберите **Скачать**, чтобы скачать сертификат и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

1. Скопируйте требуемый URL-адрес из раздела **Настройка Palo Alto Networks (GlobalProtect)** .

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

В этом разделе описано, как предоставить пользователю B. Simon доступ к Palo Alto Networks (GlobalProtect), чтобы он мог использовать единый вход Azure.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. Выберите **Palo Alto Networks (GlobalProtect)** из списка приложений.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-palo-alto-networks---globalprotect-sso"></a>Настройка единого входа в Palo Alto Networks (GlobalProtect)

1. В другом окне браузера откройте Palo Alto Networks (GlobalProtect) с правами администратора.

2. Щелкните **Device** (Устройство).

    ![Настройка единого входа в Palo Alto 1](./media/paloaltoglobalprotect-tutorial/tutorial_paloaltoadmin_admin1.png)

3. Чтобы импортировать файл метаданных, в левой области навигации выберите **SAML Identity Provider** (Поставщик удостоверений SAML) и нажмите кнопку Import (Импорт).

    ![Настройка единого входа в Palo Alto 2](./media/paloaltoglobalprotect-tutorial/tutorial_paloaltoadmin_admin2.png)

4. Сделайте следующее в окне Import (Импорт).

    ![Настройка единого входа в Palo Alto 3](./media/paloaltoglobalprotect-tutorial/tutorial_paloaltoadmin_admin3.png)

    а. Укажите имя в текстовом поле **Profile Name** (Имя профиля), например Azure AD GlobalProtect.

    b. В разделе **Identity Provider Metadata** (Метаданные поставщика удостоверений) щелкните **Browse** (Обзор) и выберите файл metadata.xml, загруженный ранее с портала Azure.

    c. Нажмите кнопку **ОК**.

### <a name="create-palo-alto-networks---globalprotect-test-user"></a>Создание тестового пользователя в Palo Alto Networks (GlobalProtect)

В этом разделе создается пользователь B. Simon в приложении Palo Alto Networks (GlobalProtect). Приложение Palo Alto Networks (GlobalProtect) поддерживает JIT-подготовку пользователей, которая включена по умолчанию. В этом разделе никакие действия с вашей стороны не требуются. Если пользователь еще не существует в Palo Alto Networks (GlobalProtect), он создается после проверки подлинности.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены на URL-адрес входа в Palo Alto Networks (GlobalProtect), где можно инициировать поток входа. 

* Напрямую перейдите по URL-адресу входа в Palo Alto Networks (GlobalProtect) и инициируйте поток входа.

* Вы можете использовать портал "Мои приложения" корпорации Майкрософт. Щелкнув плитку Palo Alto Networks (GlobalProtect) на портале "Мои приложения", вы автоматически войдете в приложение Palo Alto Networks (GlobalProtect), для которого настроили единый вход. Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Дальнейшие действия

После настройки Palo Alto Networks (GlobalProtect) вы можете применить функцию управления сеансом, которая в реальном времени защищает конфиденциальные данные вашей организации от кражи и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).