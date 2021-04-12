---
title: Руководство по интеграции единого входа Azure Active Directory с Datava Enterprise Service Platform | Документация Майкрософт
description: Сведения о том, как настроить единый вход Azure Active Directory в Datava Enterprise Service Platform.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/20/2020
ms.author: jeedes
ms.openlocfilehash: fa91c634aca5428dcca09c4f23df7c639bf07a39
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92454844"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-datava-enterprise-service-platform"></a>Руководство по интеграции единого входа Azure Active Directory с Datava Enterprise Service Platform

В этом учебнике описано, как интегрировать Datava Enterprise Service Platform с Azure Active Directory (Azure AD). Интеграция Datava Enterprise Service Platform с Azure AD обеспечивает следующие возможности:

* Контроль доступа к Datava Enterprise Service Platform с помощью Azure AD.
* Автоматический вход пользователей в Datava Enterprise Service Platform с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

Чтобы узнать больше об интеграции приложений SaaS с Azure AD, прочитайте статью [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка с поддержкой единого входа на Datava Enterprise Service Platform.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Datava Enterprise Service Platform поддерживает единый вход, инициированный **поставщиком услуг**.
* Datava Enterprise Service Platform поддерживает **JIT**-подготовку пользователей.
* После настройки Datava Enterprise Service Platform можно применять элемент управления сеансами, который защищает от хищения и несанкционированного доступа к конфиденциальным данным вашей организации в режиме реального времени. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

> [!NOTE]
> Идентификатор этого приложения — фиксированное строковое значение, поэтому в одном клиенте можно настроить только один экземпляр.

## <a name="adding-datava-enterprise-service-platform-from-the-gallery"></a>Добавление Datava Enterprise Service Platform из коллекции

Чтобы настроить интеграцию Datava Enterprise Service Platform с Azure AD, добавьте Datava Enterprise Service Platform из коллекции в список управляемых приложений SaaS.

1. Войдите на [портал Azure](https://portal.azure.com) с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Datava Enterprise Service Platform**.
1. Выберите **Datava Enterprise Service Platform** на панели результатов, а затем добавьте приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-datava-enterprise-service-platform"></a>Настройка и проверка единого входа в Azure AD для Datava Enterprise Service Platform

Настройте и проверьте единый вход Azure AD в Datava Enterprise Service Platform с помощью тестового пользователя **B.Simon**. Чтобы обеспечить работу единого входа, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Datava Enterprise Service Platform.

Чтобы настроить и проверить единый вход Azure AD в Datava Enterprise Service Platform, выполните действия в следующих стандартных блоках:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Datava Enterprise Service Platform](#configure-datava-enterprise-service-platform-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя Datava Enterprise Service Platform](#create-datava-enterprise-service-platform-test-user)** требуется для того, чтобы в Datava Enterprise Service Platform существовал пользователь B. Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Datava Enterprise Service Platform** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовое поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://go.datava.com/<TENANT_NAME>`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес `https://go.datava.com/saml/module.php/saml/sp/saml2-acs.php/azure-sp`.

    > [!NOTE]
    > Это значение приведено для примера. Вместо него необходимо указать фактический URL-адрес входа. Чтобы получить это значение, обратитесь к [группе поддержки клиентов Datava Enterprise Service Platform](mailto:support@datava.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

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

В этом разделе описано, как предоставить пользователю B. Simon доступ к Datava Enterprise Service Platform, чтобы он мог использовать единый вход Azure.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Datava Enterprise Service Platform**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.

   ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Ссылка "Добавить пользователя"](common/add-assign-user.png)

1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор роли** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-datava-enterprise-service-platform-sso"></a>Настройка единого входа в Datava Enterprise Service Platform

Чтобы настроить единый вход на стороне **Datava Enterprise Service Platform**, необходимо отправить **URL-адрес метаданных федерации приложений** в [группу поддержки Datava Enterprise Service Platform](mailto:support@datava.com). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-datava-enterprise-service-platform-test-user"></a>Создание тестового пользователя Datava Enterprise Service Platform

В этом разделе вы создадите в Datava Enterprise Service Platform пользователя с именем Britta Simon. Приложение Datava Enterprise Service Platform поддерживает JIT-подготовку пользователей. Эта функция включена по умолчанию. В этом разделе никакие действия с вашей стороны не требуются. Если пользователь еще не существует в Datava Enterprise Service Platform, его создадут после проверки подлинности.

## <a name="test-sso"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку Datava Enterprise Service Platform на Панели доступа, вы автоматически войдете в приложение Datava Enterprise Service Platform, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)

- [Попробуйте использовать Datava Enterprise Service Platform с Azure AD](https://aad.portal.azure.com/)

- [Что такое управление сеансами в Microsoft Cloud App Security?](/cloud-app-security/proxy-intro-aad)