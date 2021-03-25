---
title: Руководство по интеграции единого входа Azure Active Directory с Hosted Heritage Online SSO | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Hosted Heritage Online SSO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/17/2019
ms.author: jeedes
ms.openlocfilehash: fe09da61f4d87fbef4ce6c8739cd79c7bcda6796
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92442764"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-hosted-heritage-online-sso"></a>Руководство по интеграции единого входа Azure Active Directory с Hosted Heritage Online SSO

В этом руководстве описано, как интегрировать Hosted Heritage Online SSO с Azure Active Directory (Azure AD). Интеграция Hosted Heritage Online SSO с Azure AD обеспечивает следующие возможности:

* С помощью Azure AD вы можете контролировать доступ к Hosted Heritage Online SSO.
* Вы можете включить автоматический вход пользователей в Hosted Heritage Online SSO с использованием учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

Чтобы узнать больше об интеграции приложений SaaS с Azure AD, прочитайте статью [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Hosted Heritage Online SSO с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Hosted Heritage Online SSO поддерживает единый вход, инициируемый **поставщиком служб**.

## <a name="adding-hosted-heritage-online-sso-from-the-gallery"></a>Добавление Hosted Heritage Online SSO из коллекции

Чтобы настроить интеграцию Hosted Heritage Online SSO с Azure AD, необходимо добавить Hosted Heritage Online SSO из коллекции в список управляемых приложений SaaS.

1. Войдите на [портал Azure](https://portal.azure.com) с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Hosted Heritage Online SSO**.
1. Выберите **Hosted Heritage Online SSO** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-single-sign-on-for-hosted-heritage-online-sso"></a>Настройка и проверка единого входа Azure AD для Hosted Heritage Online SSO

Настройте и проверьте единый вход Azure AD в Hosted Heritage Online SSO с помощью тестового пользователя **B.Simon**. Для единого входа нужно установить связь между пользователем Azure AD и соответствующим пользователем в Hosted Heritage Online SSO.

Чтобы настроить и проверить единый вход Azure AD в Hosted Heritage Online SSO, потребуется выполнить действия в следующих стандартных блоках:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Hosted Heritage Online SSO](#configure-hosted-heritage-online-sso-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя Hosted Heritage Online SSO](#create-hosted-heritage-online-sso-test-user)** требуется для того, чтобы в Hosted Heritage Online SSO существовал пользователь B.Simon, связанный с представлением этого же пользователя в AAD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Hosted Heritage Online SSO** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<SUBDOMAIN>.cirqahosting.com/Shibboleth.sso/Login`.

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `https://<SUBDOMAIN>.cirqahosting.com/shibboleth`.

    > [!NOTE]
    > Эти значения приведены для примера. Необходимо обновить эти значения действующим URL-адресом для входа и идентификатором. Чтобы получить эти значения, обратитесь в [службу поддержки клиентов Hosted Heritage Online SSO](mailto:support@isoxford.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

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

В этом разделе объясняется, как включить единый вход в Azure для пользователя B.Simon, предоставив этому пользователю доступ к Hosted Heritage Online SSO.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Hosted Heritage Online SSO**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.

   ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Ссылка "Добавить пользователя"](common/add-assign-user.png)

1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор роли** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-hosted-heritage-online-sso-sso"></a>Настройка единого входа на стороне Hosted Heritage Online SSO

Чтобы настроить единый вход на стороне **Hosted Heritage Online SSO**, отправьте в [службу поддержки Hosted Heritage Online SSO](mailto:support@isoxford.com)**URL-адрес метаданных федерации приложения**. Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-hosted-heritage-online-sso-test-user"></a>Создание тестового пользователя в Hosted Heritage Online SSO

В этом разделе объясняется, как создать пользователя B.Simon в приложении Hosted Heritage Online SSO. Обратитесь к [группе поддержки Hosted Heritage Online SSO](mailto:support@isoxford.com), чтобы добавить пользователей на платформу Hosted Heritage Online SSO. Перед использованием единого входа необходимо создать и активировать пользователей.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкните плитку Hosted Heritage Online SSO на панели доступа, и вы автоматически войдете в приложение Hosted Heritage Online SSO, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)

- [Пробная работа с Hosted Heritage Online SSO и AAD](https://aad.portal.azure.com/)