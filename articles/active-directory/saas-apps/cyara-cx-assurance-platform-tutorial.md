---
title: Руководство по интеграции единого входа Azure Active Directory с Cyara CX Assurance Platform | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Cyara CX Assurance Platform.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/06/2020
ms.author: jeedes
ms.openlocfilehash: 18a0037f2af221101b4202883e5c145471af1183
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92455062"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-cyara-cx-assurance-platform"></a>Руководство по интеграции единого входа Azure Active Directory с Cyara CX Assurance Platform

В этом руководстве описано, как интегрировать Cyara CX Assurance Platform с Azure Active Directory (Azure AD). Интеграция Cyara CX Assurance Platform с Azure AD обеспечивает возможности, перечисленные ниже.

* С помощью Azure AD вы можете контролировать доступ к Cyara CX Assurance Platform.
* Вы можете включить автоматический вход пользователей в Cyara CX Assurance Platform с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

Чтобы узнать больше об интеграции приложений SaaS с Azure AD, прочитайте статью [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Cyara CX Assurance Platform с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Cyara CX Assurance Platform поддерживает единый вход, инициированный **поставщиком удостоверений**.
* После настройки Cyara CX Assurance Platform вы сможете применять элемент управления сеансами, который защищает от хищения конфиденциальных данных вашей организации и несанкционированного доступа к ним в реальном времени. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-cyara-cx-assurance-platform-from-the-gallery"></a>Добавление Cyara CX Assurance Platform из коллекции

Чтобы настроить интеграцию Cyara CX Assurance Platform с Azure AD, добавьте Cyara CX Assurance Platform из коллекции в список управляемых приложений SaaS.

1. Войдите на [портал Azure](https://portal.azure.com) с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Cyara CX Assurance Platform**.
1. Выберите **Cyara CX Assurance Platform** в области результатов, а затем добавьте приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-single-sign-on-for-cyara-cx-assurance-platform"></a>Настройка и проверка единого входа Azure AD для Cyara CX Assurance Platform

Настройте и проверьте единый вход Azure AD в Cyara CX Assurance Platform с помощью тестового пользователя **B.Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Cyara CX Assurance Platform.

Чтобы настроить и проверить единый вход Azure AD в Cyara CX Assurance Platform, выполните действия в следующих стандартных блоках:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Cyara CX Assurance Platform](#configure-cyara-cx-assurance-platform-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя Cyara CX Assurance Platform](#create-cyara-cx-assurance-platform-test-user)** требуется для того, чтобы в Cyara CX Assurance Platform существовал пользователь B.Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Cyara CX Assurance Platform** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Настройка единого входа с помощью SAML** введите значения для следующих полей:

    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `https://www.cyaraportal.us/cyarawebidentity/identity/<provider>`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://www.cyaraportal.us/cyarawebidentity/identity/<provider>/Acs`.

    > [!NOTE]
    > Эти значения приведены для примера. Измените их на фактические значения идентификатора и URL-адреса ответа. Чтобы получить эти значения, обратитесь в [службу технической поддержки Cyara CX Assurance Platform](mailto:support@cyara.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. В разделе **Сертификат подписи SAML** щелкните кнопку **Правка**, чтобы открыть диалоговое окно **Сертификат подписи SAML**.

    ![Изменение сертификата подписи SAML](common/edit-certificate.png)

1. В разделе **Сертификат подписи SAML** скопируйте **значение отпечатка** и сохраните его на компьютере.

    ![Копирование значения "Отпечаток"](common/copy-thumbprint.png)

1. Требуемые URL-адреса можно скопировать в разделе **Настройка Cyara CX Assurance Platform**.

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

В этом разделе описано, как включить единый вход Azure для пользователя B.Simon, предоставив этому пользователю доступ к Cyara CX Assurance Platform.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Cyara CX Assurance Platform**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.

   ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Ссылка "Добавить пользователя"](common/add-assign-user.png)

1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор роли** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-cyara-cx-assurance-platform-sso"></a>Настройка единого входа в Cyara CX Assurance Platform

Чтобы настроить единый вход на стороне **Cyara CX Assurance Platform**, нужно отправить **значение отпечатка** и соответствующие URL-адреса, скопированные на портале Azure, [группе поддержки Cyara CX Assurance Platform](mailto:support@cyara.com). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-cyara-cx-assurance-platform-test-user"></a>Создание тестового пользователя в Cyara CX Assurance Platform

В этом разделе описано, как создать пользователя B.Simon в приложении Cyara CX Assurance Platform. Обратитесь к [группе поддержки Cyara CX Assurance Platform](mailto:support@cyara.com), чтобы добавить пользователей на платформу Cyara CX Assurance Platform. Перед использованием единого входа необходимо создать и активировать пользователей.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку Cyara CX Assurance Platform на панели доступа, вы автоматически войдете в приложение Cyara CX Assurance Platform, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)

- [Пробное использование Cyara CX Assurance Platform с Azure AD](https://aad.portal.azure.com/)

- [Что такое управление сеансами в Microsoft Cloud App Security?](/cloud-app-security/proxy-intro-aad)

- [Защита приложений с помощью Управления условным доступом к приложениям в Microsoft Cloud App Security](/cloud-app-security/proxy-intro-aad)