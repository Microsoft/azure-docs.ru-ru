---
title: Руководство по Интеграция единого входа Azure Active Directory с OfficeSpace Software | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в OfficeSpace Software.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/23/2019
ms.author: jeedes
ms.openlocfilehash: bb994e154f410fb15b66588b41688326509a8f4a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92518544"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-officespace-software"></a>Руководство по Интеграция единого входа Azure Active Directory с OfficeSpace Software

В этом учебнике описано, как интегрировать OfficeSpace Software с Azure Active Directory (Azure AD). Интеграция OfficeSpace Software с Azure AD обеспечивает следующие возможности.

* Контроль доступа к OfficeSpace Software с помощью Azure AD.
* Автоматический вход пользователей в OfficeSpace Software с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

Чтобы узнать больше об интеграции приложений SaaS с Azure AD, прочитайте статью [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка OfficeSpace Software с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* В OfficeSpace Software поддерживается единый вход, инициированный **поставщиком услуг**.


* OfficeSpace Software поддерживает **JIT**-подготовку пользователей.


## <a name="adding-officespace-software-from-the-gallery"></a>Добавление OfficeSpace Software из коллекции

Чтобы настроить интеграцию OfficeSpace Software с Azure AD, необходимо добавить OfficeSpace Software из коллекции в список управляемых приложений SaaS.

1. Войдите на [портал Azure](https://portal.azure.com) с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **OfficeSpace Software**.
1. Выберите **OfficeSpace Software** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.


## <a name="configure-and-test-azure-ad-single-sign-on-for-officespace-software"></a>Настройка и проверка единого входа Azure AD для OfficeSpace Software

Настройте и проверьте единый вход Azure AD в OfficeSpace Software с помощью тестового пользователя **B. Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в OfficeSpace Software.

Чтобы настроить и проверить единый вход Azure AD в OfficeSpace Software, выполните действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в OfficeSpace Software](#configure-officespace-software-sso)** требуется, чтобы определить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя OfficeSpace Software](#create-officespace-software-test-user)** требуется, чтобы в OfficeSpace Software существовал пользователь B. Simon, связанный с соответствующим представлением в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **OfficeSpace Software** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<company name>.officespacesoftware.com/users/sign_in/saml`.

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `<company name>.officespacesoftware.com`.

    > [!NOTE]
    > Эти значения приведены для примера. Необходимо обновить эти значения действующим URL-адресом для входа и идентификатором. Обратитесь в [службу поддержки клиентов OfficeSpace Software](mailto:support@officespacesoftware.com), чтобы получить эти значения. Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. Приложение OfficeSpace Software ожидает проверочные утверждения SAML в определенном формате, для которого необходимо добавить настраиваемые сопоставления атрибутов в конфигурацию атрибутов токена SAML. На следующем снимке экрана показан список атрибутов по умолчанию, в котором **nameidentifier** сопоставляется с **user.userprincipalname**. Приложение OfficeSpace Software ожидает, что **nameidentifier** будет сопоставляться с **user.mail**, поэтому необходимо изменить сопоставление атрибутов, щелкнув значок **Изменить**.

    ![Изображение](common/edit-attribute.png)

1. Кроме того, приложение OfficeSpace Software ожидает в ответе SAML несколько дополнительных атрибутов, которые показаны ниже. Эти атрибуты также заранее заполнены, но вы можете изменить их в соответствии со своими требованиями.

    | Имя | Исходный атрибут|
    | ---------------| --------------- |
    | email | user.mail |
    | name | user.displayname |
    | first_name | user.givenname |
    | last_name | user.surname |

1. В разделе **Сертификат подписи SAML** щелкните кнопку **Правка**, чтобы открыть диалоговое окно **Сертификат подписи SAML**.

    ![Изменение сертификата подписи SAML](common/edit-certificate.png)

1. В разделе **Сертификат подписи SAML** скопируйте **значение отпечатка** и сохраните его на компьютере.

    ![Копирование значения "Отпечаток"](common/copy-thumbprint.png)

1. Требуемые URL-адреса можно скопировать из раздела **Настройка OfficeSpace Software**.

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

В этом разделе описано, как разрешить пользователю B. Simon использовать единый вход Azure путем предоставления доступа к OfficeSpace Software.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. Из списка приложений выберите **OfficeSpace Software**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.

   ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Ссылка "Добавить пользователя"](common/add-assign-user.png)

1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор роли** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="configure-officespace-software-sso"></a>Настройка единого входа в OfficeSpace Software

1. В другом окне веб-браузера войдите в свой клиент OfficeSpace Software в качестве администратора.

2. Выберите **Settings** (Параметры) и щелкните **Connectors** (Соединители).

    ![Снимок экрана: меню "Settings" (Параметры) с выбранным пунктом "Connectors" (Соединители).](./media/officespace-tutorial/tutorial_officespace_002.png)

3. Щелкните **SAML Authentication** (Аутентификация SAML).

    ![Снимок экрана: раздел "Authentication" (Аутентификация) с выбранным действием "SAML Authentication" (Аутентификация SAML).](./media/officespace-tutorial/tutorial_officespace_003.png)

4. В разделе **Проверка подлинности SAML** сделайте следующее:

    ![Настройка единого входа на стороне приложения](./media/officespace-tutorial/tutorial_officespace_004.png)

    а. В текстовое поле **Logout provider url** (URL-адрес поставщика для выхода) вставьте значение **URL-адреса выхода**, скопированное с портала Azure.

    b. В текстовое поле **Client idp target url** (Целевой URL-адрес поставщика удостоверений клиента) вставьте значение **URL-адреса входа**, скопированное с портала Azure.

    c. В текстовое поле **Client IDP certificate fingerprint** (Отпечаток сертификата IDP клиента) вставьте значение **отпечатка**, скопированное с портала Azure. 

    d. Нажмите кнопку **Сохранить параметры**.

### <a name="create-officespace-software-test-user"></a>Создание тестового пользователя в OfficeSpace Software

В этом разделе вы создадите в OfficeSpace Software пользователя с именем B. Simon. Приложение OfficeSpace Software поддерживает JIT-подготовку пользователей, которая включена по умолчанию. В этом разделе никакие действия с вашей стороны не требуются. Если пользователь еще не существует в OfficeSpace Software, он создается после проверки подлинности.

> [!NOTE]
> Чтобы создать пользователя вручную, необходимо обратиться к [группе поддержки OfficeSpace Software](mailto:support@officespacesoftware.com).

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку OfficeSpace Software на Панели доступа, вы автоматически войдете в приложение OfficeSpace Software, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)

- [Попробуйте использовать Officespace Software с Azure AD](https://aad.portal.azure.com/)