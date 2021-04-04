---
title: Руководство по Интеграция единого входа Azure AD со Smart Global Governance
description: Узнайте, как настроить единый вход между Azure Active Directory и Smart Global Governance.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/04/2020
ms.author: jeedes
ms.openlocfilehash: 5a2169db47ca5a6adcddbcc9558161370b896a62
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "95997075"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-smart-global-governance"></a>Руководство по интеграции единого входа Azure Active Directory со Smart Global Governance

В этом руководстве описано, как интегрировать Smart Global Governance с Azure Active Directory (Azure AD). Интеграция Smart Global Governance с Azure AD обеспечивает следующие возможности.

* C помощью Azure AD можно контролировать доступ к Smart Global Governance.
* Вы можете включить автоматический вход пользователей в Smart Global Governance с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Smart Global Governance с поддержкой единого входа.

## <a name="tutorial-description"></a>Описание учебника

В рамках этого учебника вы настроите и проверите единый вход Azure AD в тестовой среде.

Smart Global Governance поддерживает единый вход, инициируемый поставщиком услуг и поставщиком удостоверений.

После настройки Smart Global Governance вы можете применить функцию управления сеансом, которая в режиме реального времени защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа. Элементы управления сеансом являются расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="add-smart-global-governance-from-the-gallery"></a>Добавление Smart Global Governance из коллекции

Чтобы настроить интеграцию Smart Global Governance с Azure AD, необходимо добавить Smart Global Governance из коллекции в список управляемых приложений SaaS.

1. Войдите на [портал Azure](https://portal.azure.com) с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области слева выберите **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Smart Global Governance**.
1. Выберите **Smart Global Governance** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-smart-global-governance"></a>Настройка и проверка единого входа Azure AD для Smart Global Governance

Настройте и проверьте единый вход Azure AD в Smart Global Governance с помощью тестового пользователя B. Simon. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Smart Global Governance.

Чтобы настроить и проверить единый вход Azure AD в Smart Global Governance, вам потребуется выполнить приведенные ниже действия.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD.
    1. **[Предоставление тестовому пользователю доступа](#grant-access-to-the-test-user)** необходимо, чтобы разрешить пользователю использовать единый вход Azure AD.
1. **[Настройка единого входа в Smart Global Governance](#configure-smart-global-governance-sso)** на стороне приложения.
    1. **[Создание тестового пользователя Smart Global Governance](#create-a-smart-global-governance-test-user)** , связанного с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** необходима, чтобы убедиться в корректной работе конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure:

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Smart Global Governance** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить эти параметры:

   ![Значок карандаша для базовой конфигурации SAML](common/edit-urls.png)

1. Если вы хотите настроить приложение в режиме, инициируемом поставщиком удостоверений, в разделе **Базовая конфигурация SAML** выполните следующие действия.

    а. В поле **Идентификатор** введите один из следующих URL-адресов:

    - `https://eu-fr-south.console.smartglobalprivacy.com/platform/authentication-saml2/metadata`
    - `https://eu-fr-south.console.smartglobalprivacy.com/dpo/authentication-saml2/metadata`

    b. В поле **URL-адрес ответа** введите один из следующих URL-адресов:

    - `https://eu-fr-south.console.smartglobalprivacy.com/platform/authentication-saml2/acs`
    - `https://eu-fr-south.console.smartglobalprivacy.com/dpo/authentication-saml2/acs`

1. Если вы хотите настроить приложение для работы в режиме, инициируемом поставщиком услуг, выберите **Задать дополнительные URL-адреса** и выполните следующее действие.

   - В поле **URL-адрес для входа** введите один из следующих URL-адресов:

    - `https://eu-fr-south.console.smartglobalprivacy.com/dpo`
    - `https://eu-fr-south.console.smartglobalprivacy.com/platform`

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите элемент **Сертификат (необработанный)** и щелкните **Скачать**, чтобы скачать сертификат. Сохраните этот сертификат на компьютере:

    ![Ссылка для скачивания сертификата](common/certificateraw.png)

1. Скопируйте требуемые URL-адреса в разделе **Настройка Smart Global Governance**:

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

В этом разделе описано, как включить единый вход Azure для пользователя B. Simon, предоставив этому пользователю доступ к Smart Global Governance.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Smart Global Governance**.
1. В разделе **Управление** на странице сводных данных о приложении выберите **Пользователи и группы**:

   ![Выбор параметра "Пользователи и группы"](common/users-groups-blade.png)

1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** — **Пользователи и группы**:

    ![Выбор элемента "Добавить пользователя"](common/add-assign-user.png)

1. В диалоговом окне **Пользователи и группы** выберите **B. Simon** в списке **Пользователи**, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор роли** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.
1. В диалоговом окне **Добавление назначения** выберите **Назначить**.

## <a name="configure-smart-global-governance-sso"></a>Настройка единого входа в Smart Global Governance

Чтобы настроить единый вход на стороне Smart Global Governance, отправьте скачанный необработанный сертификат и URL-адреса, скопированные на портале Azure, [группе поддержки Smart Global Governance](mailto:support.tech@smartglobal.com). Они настраивают подключение единого входа SAML так, чтобы оно было правильным на обеих сторонах.

### <a name="create-a-smart-global-governance-test-user"></a>Создание тестового пользователя Smart Global Governance

Обратитесь к [группе поддержки Smart Global Governance](mailto:support.tech@smartglobal.com), чтобы добавить пользователя B. Simon в Smart Global Governance. Перед использованием единого входа необходимо создать и активировать пользователей.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе вы с помощью панели доступа проверите конфигурацию единого входа Azure AD.

Щелкнув плитку Smart Global Governance на Панели доступа, вы автоматически войдете в экземпляр Smart Global Governance, для которого настроили единый вход. Дополнительные сведения о панели доступа см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Учебники по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)

- [Пробное использование Smart Global Governance с Azure AD](https://aad.portal.azure.com/)

- [Что такое управление сеансами в Microsoft Cloud App Security?](/cloud-app-security/proxy-intro-aad)

- [Защита приложений с помощью Управления условным доступом к приложениям в Microsoft Cloud App Security](/cloud-app-security/proxy-intro-aad)