---
title: Руководство по интеграции Azure Active Directory с Blackboard Learn — Shibboleth | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Blackboard Learn — Shibboleth.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/19/2021
ms.author: jeedes
ms.openlocfilehash: b1b3f265d0e1fcad2953292c5227c2630c6df229
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101649915"
---
# <a name="tutorial-azure-active-directory-integration-with-blackboard-learn---shibboleth"></a>Руководство. Интеграция Azure Active Directory с Blackboard Learn — Shibboleth

В этом руководстве описано, как интегрировать Blackboard Learn — Shibboleth с Azure Active Directory (Azure AD). Интеграция Blackboard Learn — Shibboleth с Azure AD обеспечивает следующие возможности:

* Контроль доступа к Blackboard Learn — Shibboleth с помощью Azure Active Directory.
* Автоматический вход пользователей в Blackboard Learn — Shibboleth с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.
 
* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка на Blackboard Learn — Shibboleth с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Blackboard Learn — Shibboleth поддерживает единый вход, инициированный **поставщиком услуг**.

## <a name="add-blackboard-learn---shibboleth-from-the-gallery"></a>Добавление Blackboard Learn — Shibboleth из коллекции

Чтобы настроить интеграцию Blackboard Learn — Shibboleth с Azure AD, необходимо добавить Blackboard Learn — Shibboleth из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Blackboard Learn — Shibboleth**.
1. Выберите **Blackboard Learn — Shibboleth** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-blackboard-learn---shibboleth"></a>Настройка и проверка единого входа Azure AD для Blackboard Learn — Shibboleth

Настройте и проверьте единый вход Azure AD в Blackboard Learn — Shibboleth с помощью тестового пользователя **B. Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Blackboard Learn — Shibboleth.

Чтобы настроить и проверить единый вход Azure Active Directory в Blackboard Learn — Shibboleth, выполните следующие действия.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Blackboard Learn — Shibboleth](#configure-blackboard-learn---shibboleth-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя Blackboard Learn — Shibboleth](#create-blackboard-learn---shibboleth-test-user)** требуется для того, чтобы в Blackboard Learn — Shibboleth существовал пользователь B. Simon, связанный с одноименным пользователем в Azure Active Directory.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

### <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure Active Directory в Blackboard Learn — Shibboleth, выполните следующие действия.

1. На портале Azure на странице интеграции с приложением **Blackboard Learn — Shibboleth** выберите **Единый вход**.

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

3. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    а. В текстовое поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<yourblackoardlearnserver>.blackboardlearn.com/Shibboleth.sso/Login`.

    b. В поле **Идентификатор** введите URL-адрес в следующем формате: `https://<yourblackoardlearnserver>.blackboardlearn.com/shibboleth-sp`.

    c. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<yourblackoardlearnserver>.blackboardlearn.com/Shibboleth.sso/SAML2/POST`.

    > [!NOTE]
    > Эти значения приведены для примера. Укажите вместо них фактические значения URL-адреса для входа, идентификатора и URL-адреса ответа. Чтобы получить эти значения, обратитесь к [группе поддержки Blackboard Learn — Shibboleth](https://www.blackboard.com/forms/contact-us_form.aspx). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

6. Скопируйте требуемый URL-адрес из раздела **Настройка Blackboard Learn — Shibboleth**.

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

В этом разделе описано, как включить единый вход Azure для пользователя B. Simon, предоставив этому пользователю доступ к Blackboard Learn — Shibboleth.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Blackboard Learn — Shibboleth**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="configure-blackboard-learn---shibboleth-sso"></a>Настройка единого входа в Blackboard Learn — Shibboleth

Чтобы настроить единый вход на стороне **Blackboard Learn — Shibboleth**, нужно отправить скачанный файл **XML метаданных федерации** и соответствующие URL-адреса, скопированные на портале Azure, [группе поддержки Blackboard Learn — Shibboleth](https://www.blackboard.com/forms/contact-us_form.aspx). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-blackboard-learn---shibboleth-test-user"></a>Создание тестового пользователя в Blackboard Learn — Shibboleth

В этом разделе описано, как создать пользователя Britta Simon в приложении Blackboard Learn — Shibboleth. Обратитесь к [группе поддержки Blackboard Learn — Shibboleth](https://www.blackboard.com/forms/contact-us_form.aspx), чтобы добавить пользователей на платформу Blackboard Learn — Shibboleth. Перед использованием единого входа необходимо создать и активировать пользователей.

### <a name="test-sso"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в Blackboard Learn — Shibboleth, где можно инициировать поток входа. 

* Перейдите по URL-адресу для входа в Blackboard Learn — Shibboleth и инициируйте поток входа.

* Вы можете использовать портал "Мои приложения" корпорации Майкрософт. Щелкнув плитку Blackboard Learn — Shibboleth в области "Мои приложения", вы автоматически войдете в приложение Blackboard Learn — Shibboleth, для которого настроили единый вход. Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Дальнейшие действия

После настройки Blackboard Learn — Shibboleth вы можете применить функцию управления сеансами, которая защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа в реальном времени. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).