---
title: Руководство по интеграции единого входа Azure Active Directory с приложением Greenlight Enterprise Business Controls Platform | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Greenlight Enterprise Business Controls Platform.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/04/2020
ms.author: jeedes
ms.openlocfilehash: e1e15c3fdf9900d016581f848fdfdba5d91b0997
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92447126"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-greenlight-enterprise-business-controls-platform"></a>Руководство по интеграции единого входа Azure Active Directory с приложением Greenlight Enterprise Business Controls Platform

В этом руководстве описано, как интегрировать Greenlight Enterprise Business Controls Platform с Azure Active Directory (Azure AD). При интеграции Greenlight Enterprise Business Controls Platform с Azure AD вы получите следующие возможности:

* Через Azure AD вы сможете управлять доступом к Greenlight Enterprise Business Controls Platform.
* Ваши пользователи смогут автоматически входить в Greenlight Enterprise Business Controls Platform с учетными записями Azure AD.
* Централизованное управление учетными записями через портал Azure.

Чтобы узнать больше об интеграции приложений SaaS с Azure AD, прочитайте статью [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Greenlight Enterprise Business Controls Platform с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Платформа Greenlight Enterprise Business Controls Platform поддерживает единый вход, инициируемый **поставщиком служб и поставщиком удостоверений**.

* После настройки приложения Greenlight Enterprise Business Controls Platform вы сможете применять управление сеансами, которое в реальном времени защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-greenlight-enterprise-business-controls-platform-from-the-gallery"></a>Добавление Greenlight Enterprise Business Controls Platform из коллекции

Чтобы настроить интеграцию приложения Greenlight Enterprise Business Controls Platform с Azure AD, добавьте это приложение из коллекции в список управляемых приложений SaaS.

1. Войдите на [портал Azure](https://portal.azure.com) с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Greenlight Enterprise Business Controls Platform**.
1. Выберите **Greenlight Enterprise Business Controls Platform** на панели результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.


## <a name="configure-and-test-azure-ad-sso-for-greenlight-enterprise-business-controls-platform"></a>Настройка и проверка единого входа Azure AD для Greenlight Enterprise Business Controls Platform

Настройте и проверьте единый вход Azure AD в Greenlight Enterprise Business Controls Platform с помощью тестового пользователя **B. Simon**. Для работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Greenlight Enterprise Business Controls Platform.

Чтобы настроить и проверить единый вход Azure AD в Greenlight Enterprise Business Controls Platform, выполните действия в следующих стандартных блоках:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Greenlight Enterprise Business Controls Platform](#configure-greenlight-enterprise-business-controls-platform-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя Greenlight Enterprise Business Controls Platform](#create-greenlight-enterprise-business-controls-platform-test-user)** требуется для того, чтобы в приложении Greenlight Enterprise Business Controls Platform существовал пользователь B. Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Greenlight Enterprise Business Controls Platform** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `https://<CUSTOMER_NAME>.gltcloud.com/ebcpplatform/saml`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<CUSTOMER_NAME>.gltcloud.com/ebcpplatform/saml`.

1. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://<CUSTOMER_NAME>.gltcloud.com/ebcpplatform/saml`.

    > [!NOTE]
    > Эти значения приведены для примера. Замените их фактическими значениями идентификатора, URL-адреса ответа и URL-адреса входа. Чтобы получить эти значения, обратитесь к [группе поддержки Greenlight Enterprise Business Controls Platform](mailto:support@greenlightcorp.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите элемент **XML метаданных федерации** и выберите **Скачать**, чтобы скачать сертификат и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

1. Скопируйте требуемые URL-адреса на **странице настройки Greenlight Enterprise Business Controls Platform**.

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

В этом разделе описано, как включить единый вход Azure для пользователя B. Simon, предоставив этому пользователю доступ к Greenlight Enterprise Business Controls Platform.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Greenlight Enterprise Business Controls Platform**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.

   ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Ссылка "Добавить пользователя"](common/add-assign-user.png)

1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор роли** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-greenlight-enterprise-business-controls-platform-sso"></a>Настройка единого входа в Greenlight Enterprise Business Controls Platform

Чтобы настроить единый вход на стороне **Greenlight Enterprise Business Controls Platform**, нужно отправить скачанный файл **XML метаданных федерации** и соответствующие URL-адреса, скопированные на портале Azure, [группе поддержки Greenlight Enterprise Business Controls Platform](mailto:support@greenlightcorp.com). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-greenlight-enterprise-business-controls-platform-test-user"></a>Тестовый пользователь в Greenlight Enterprise Business Controls Platform

В этом разделе описано, как создать пользователя B. Simon в Greenlight Enterprise Business Controls Platform. Чтобы добавить пользователей на платформу Greenlight Enterprise Business Controls, обратитесь в [службу поддержки Greenlight Enterprise Business Controls](mailto:support@greenlightcorp.com). Перед использованием единого входа необходимо создать и активировать пользователей.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку Greenlight Enterprise Business Controls Platform на панели доступа, вы автоматически войдете в приложение Greenlight Enterprise Business Controls Platform, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)

- [Попробуйте работу в Greenlight Enterprise Business Controls Platform с Azure AD](https://aad.portal.azure.com/)

- [Что такое управление сеансами в Microsoft Cloud App Security?](/cloud-app-security/proxy-intro-aad)

- [Защита приложений с помощью функции управления настройками условного доступа для приложений в Microsoft Cloud App Security](/cloud-app-security/proxy-intro-aad)