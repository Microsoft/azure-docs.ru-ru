---
title: Руководство по интеграции Azure Active Directory с SecureW2 JoinNow Connector | Документация Майкрософт
description: Сведения о том, как настроить единый вход Azure Active Directory в SecureW2 JoinNow Connector.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/07/2019
ms.author: jeedes
ms.openlocfilehash: 024de5acfbc4e6c4aaa510de84c0bceb6bde2b3d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92672965"
---
# <a name="tutorial-integrate-securew2-joinnow-connector-with-azure-active-directory"></a>Руководство по Интеграция SecureW2 JoinNow Connector с Azure Active Directory

В этом руководстве описано, как интегрировать SecureW2 JoinNow Connector с Azure Active Directory (Azure AD). Интеграция SecureW2 JoinNow Connector с Azure Active Directory обеспечивает следующие возможности:

* контроль доступа к SecureW2 JoinNow Connector с помощью Azure AD;
* включение автоматического входа пользователей в SecureW2 JoinNow Connector с использованием учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

Чтобы узнать больше об интеграции приложений SaaS с Azure AD, прочитайте статью [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка SecureW2 JoinNow Connector с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* SecureW2 JoinNow Connector поддерживает единый вход, инициированный **поставщиком служб**.





## <a name="adding-securew2-joinnow-connector-from-the-gallery"></a>Добавление SecureW2 JoinNow Connector из коллекции

Чтобы настроить интеграцию SecureW2 JoinNow Connector в Azure AD, нужно добавить SecureW2 JoinNow Connector из коллекции в список управляемых приложений SaaS.

1. Войдите на [портал Azure](https://portal.azure.com) с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **SecureW2 JoinNow Connector**.
1. Выберите **SecureW2 JoinNow Connector** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.


## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

Настройте и проверьте единый вход Azure AD в SecureW2 JoinNow Connector с помощью тестового пользователя **B.Simon**. Чтобы обеспечить работу единого входа, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в SecureW2 JoinNow Connector.

Чтобы настроить и проверить единый вход Azure AD в SecureW2 JoinNow Connector, выполните действия из следующих стандартных блоков:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в SecureW2 JoinNow Connector](#configure-securew2-joinnow-connector-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя SecureW2 JoinNow Connector](#create-securew2-joinnow-connector-test-user)** требуется для того, чтобы в SecureW2 JoinNow Connector существовал пользователь B.Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

### <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **SecureW2 JoinNow Connector** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<organization-identifier>-auth.securew2.com/auth/saml/SSO`.

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `https://<organization-identifier>-auth.securew2.com/auth/saml`.

    > [!NOTE]
    > Эти значения приведены для примера. Необходимо обновить эти значения действующим URL-адресом для входа и идентификатором. Чтобы получить эти значения, обратитесь в [службу поддержки клиентов SecureW2 JoinNow Connector](mailto:support@securew2.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите элемент **XML метаданных** и нажмите кнопку **Скачать**, чтобы скачать сертификат и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

1. Требуемые URL-адреса вы можете скопировать из раздела **Настройка SecureW2 JoinNow Connector**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

### <a name="configure-securew2-joinnow-connector-sso"></a>Настройка единого входа в SecureW2 JoinNow Connector

Чтобы настроить единый вход на стороне **SecureW2 JoinNow Connector**, отправьте в [службу поддержки SecureW2 JoinNow Connector](mailto:support@securew2.com) скачанный **XML-файл метаданных** и соответствующие URL-адреса, скопированные на портале Azure. Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.
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

В этом разделе описано, как разрешить пользователю B.Simon использовать единый вход Azure, предоставив этому пользователю доступ к SecureW2 JoinNow Connector.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **SecureW2 JoinNow Connector**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.

   ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Ссылка "Добавить пользователя"](common/add-assign-user.png)

1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор роли** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-securew2-joinnow-connector-test-user"></a>Создание тестового пользователя SecureW2 JoinNow Connector

В этом разделе описано, как создать пользователя Britta Simon в приложении SecureW2 JoinNow Connector. Чтобы добавить пользователей на платформу SecureW2 JoinNow Connector, обратитесь в [службу технической поддержки SecureW2 JoinNow Connector](mailto:support@securew2.com). Перед использованием единого входа необходимо создать и активировать пользователей.

### <a name="test-sso"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку SecureW2 JoinNow Connector на панели доступа, вы автоматически войдете в приложение SecureW2 JoinNow Connector, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)