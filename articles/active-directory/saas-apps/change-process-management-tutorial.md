---
title: Руководство по Интеграция единого входа Azure AD с Change Process Management
description: Из этой статьи вы узнаете, как настроить единый вход Azure Active Directory в Change Process Management.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/07/2020
ms.author: jeedes
ms.openlocfilehash: 03c78f05566876356e4f486368dc2a5b3a29de43
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92456264"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-change-process-management"></a>Руководство по интеграции единого входа Azure Active Directory с приложением Change Process Management

В этом руководстве описано, как интегрировать приложение Change Process Management с Azure Active Directory (Azure AD). Интеграция Change Process Management с Azure AD обеспечивает следующие возможности:

* Используйте Azure AD для управления доступом к Change Process Management.
* Поддержка автоматического входа пользователей в Change Process Management с учетными записями Azure AD.
* Централизованное управление учетными записями через портал Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Change Process Management с поддержкой единого входа.

## <a name="tutorial-description"></a>Описание учебника

В рамках этого учебника вы настроите и проверите единый вход Azure AD в тестовой среде.

Change Process Management поддерживает единый вход, инициированный поставщиком удостоверений.

После настройки Change Process Management вы можете применить функцию управления сеансом, которая в реальном времени защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа. Элементы управления сеансом являются расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="add-change-process-management-from-the-gallery"></a>Добавление Change Process Management из коллекции

Чтобы настроить интеграцию Change Process Management с Azure AD, необходимо добавить Change Process Management из коллекции в список управляемых приложений SaaS.

1. Войдите на [портал Azure](https://portal.azure.com) с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области слева выберите **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Change Process Management**.
1. Выберите **Change Process Management** на панели результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-change-process-management"></a>Настройка и проверка единого входа Azure AD для Change Process Management

Настройте и проверьте единый вход Azure AD в Change Process Management с помощью тестового пользователя B.Simon. Для работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Change Process Management.

Чтобы настроить и проверить единый вход Azure AD с Change Process Management, вам потребуется выполнить приведенные ниже действия.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD.
    1. **[Предоставление тестовому пользователю доступа](#grant-access-to-the-test-user)** необходимо, чтобы разрешить пользователю использовать единый вход Azure AD.
1. **[Настройка единого входа в Change Process Management](#configure-change-process-management-sso)** на стороне приложения.
    1. **[Создание тестового пользователя Change Process Management](#create-a-change-process-management-test-user)** , связанного с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** необходима, чтобы убедиться в корректной работе конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Change Process Management** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить эти параметры:

   ![Значок карандаша для базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Настройка единого входа с помощью SAML** выполните следующие действия:

    а. В поле **Идентификатор** введите URL-адрес в следующем формате: `https://<hostname>:8443/`

    b. В поле **URL-адрес ответа** введите URL-адрес в следующем формате: `https://<hostname>:8443/changepilot/saml/sso`

    > [!NOTE]
    > Предыдущий **идентификатор** и **URL-адрес ответа** не являются фактическими значениями, которые следует использовать. Чтобы получить эти значения, обратитесь в [службу технической поддержки Change Process Management](mailto:support@realtech-us.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите элемент **Сертификат (Base64)** и щелкните **Скачать**, чтобы скачать сертификат. Сохраните этот сертификат на компьютере:

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

1. Требуемые URL-адреса можно скопировать из раздела **Настройка Change Process Management**:

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD

В этом разделе описано, как создать на портале Azure тестового пользователя с именем B.Simon.

1. На портале Azure в области слева щелкните **Azure Active Directory**. Щелкните **Пользователи**, а затем **Все пользователи**.
1. В верхней части экрана выберите **Новый пользователь**.
1. В свойствах **пользователя** выполните следующие действия:
   1. В поле **Имя** введите **B.Simon**.  
   1. В поле **Имя пользователя** введите \<username>@\<companydomain>.\<extension> Например, `B.Simon@contoso.com`.
   1. Выберите **Показать пароль** и запишите значение, которое отображается в поле **Пароль**.
   1. Нажмите кнопку **создания**.

### <a name="grant-access-to-the-test-user"></a>Предоставление доступа тестовому пользователю

В этом разделе описано, как разрешить пользователю B. Simon использовать единый вход Azure, предоставив ему доступ к Change Process Management.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. Выберите **Change Process Management** в списке приложений.
1. В разделе **Управление** на странице сводных данных о приложении выберите **Пользователи и группы**:

   ![Выбор параметра "Пользователи и группы"](common/users-groups-blade.png)

1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** — **Пользователи и группы**.

    ![Выбор элемента "Добавить пользователя"](common/add-assign-user.png)

1. В диалоговом окне **Пользователи и группы** выберите **B. Simon** в списке **Пользователи**, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор роли** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.
1. В диалоговом окне **Добавление назначения** выберите **Назначить**.

## <a name="configure-change-process-management-sso"></a>Настройка единого входа в Change Process Management

Чтобы настроить единый вход на стороне Change Process Management, нужно отправить скачанный сертификат Base64 и соответствующие URL-адреса, скопированные на портале Azure, в [службу поддержки Change Process Management](mailto:support@realtech-us.com). Они настраивают подключение единого входа SAML так, чтобы оно было правильным на обеих сторонах.

### <a name="create-a-change-process-management-test-user"></a>Создание тестового пользователя Change Process Management
 Обратитесь в [службу поддержки Change Process Management](mailto:support@realtech-us.com), чтобы добавить пользователя с именем B. Simon в Change Process Management. Перед использованием единого входа необходимо создать и активировать пользователей.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе вы с помощью панели доступа проверите конфигурацию единого входа Azure AD.

Щелкнув плитку Change Process Management на Панели доступа, вы автоматически войдете в экземпляр Change Process Management, для которого настроили единый вход. Дополнительные сведения о панели доступа см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Учебники по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md).

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)

- [Пробное использование Change Process Management с Azure AD](https://aad.portal.azure.com/)

- [Что такое управление сеансами в Microsoft Cloud App Security?](/cloud-app-security/proxy-intro-aad)

- [Защита Change Process Management с помощью функции управления настройками условного доступа](/cloud-app-security/proxy-intro-aad)