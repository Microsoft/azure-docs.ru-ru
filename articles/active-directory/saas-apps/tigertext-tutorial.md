---
title: Руководство по интеграции Azure Active Directory с TigerConnect Secure Messenger | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и TigerConnect Secure Messenger.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/21/2020
ms.author: jeedes
ms.openlocfilehash: 6831e96ebd8fd2db51d640ff3d93955b1a9f1477
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96182161"
---
# <a name="tutorial-azure-active-directory-integration-with-tigerconnect-secure-messenger"></a>Руководство по интеграции Azure Active Directory с TigerConnect Secure Messenger

В этом учебнике описано, как интегрировать приложение TigerConnect Secure Messenger с Azure Active Directory (Azure AD).

Интеграция Azure AD с приложением TigerConnect Secure Messenger обеспечивает следующие возможности:

* Контроль доступа к TigerConnect Secure Messenger с помощью Azure AD.
* Автоматический вход пользователей в TigerConnect Secure Messenger (единый вход) с помощью учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно — через портал Azure.

Дополнительные сведения об интеграции приложения SaaS с Azure AD см. в статье [о доступе к приложениям и едином входе с помощью Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD с TigerConnect Secure Messenger, вам потребуется:

* подписка Azure AD; Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.
* подписка TigerConnect Secure Messenger с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого учебника вы настроите и проверите единый вход Azure AD в тестовой среде и интеграцию TigerConnect Secure Messenger с Azure AD.

* TigerConnect Secure Messenger поддерживает единый вход, инициированный **поставщиком услуг**.
* После настройки TigerConnect Secure Messenger вы сможете применить функцию управления сеансом, чтобы в реальном времени защищать конфиденциальные данные своей организации от кражи и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-tigerconnect-secure-messenger-from-the-gallery"></a>Добавление TigerConnect Secure Messenger из коллекции

Чтобы настроить интеграцию TigerConnect Secure Messenger с Azure AD, необходимо добавить TigerConnect Secure Messenger из коллекции в список управляемых приложений SaaS.

1. Войдите на [портал Azure](https://portal.azure.com) с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **TigerConnect Secure Messenger**.
1. В области результатов выберите **TigerConnect Secure Messenger** и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso"></a>Настройка и проверка единого входа Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в TigerConnect Secure Messenger с использованием тестового пользователя **Britta Simon**. Для работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в TigerConnect Secure Messenger.

Чтобы настроить и проверить единый вход Azure AD в TigerConnect Secure Messenger, выполните действия в следующих стандартных блоках:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    * **[Создайте тестового пользователя Azure AD](#create-an-azure-ad-test-user)** для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
    * **[Назначьте тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** , чтобы позволить Britta Simon использовать единый вход в Azure AD.
1. **[Настройка единого входа в TigerConnect Secure Messenger](#configure-tigerconnect-secure-messenger-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    * **[Создание тестового пользователя TigerConnect Secure Messenger](#create-a-tigerconnect-secure-messenger-test-user)** требуется для того, чтобы в TigerConnect Secure Messenger существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** необходима, чтобы убедиться в корректной работе конфигурации.

### <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в TigerConnect Secure Messenger, сделайте следующее.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **TigerConnect Secure Messenger** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Базовая конфигурация SAML** введите значения следующих полей.

    1. В поле **URL-адрес для входа** введите URL-адрес:

       `https://home.tigertext.com`

    1. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате:

       `https://saml-lb.tigertext.me/v1/organization/<instance ID>`

    > [!NOTE]
    > Указано ненастоящее значение **Идентификатор (сущности)**. Замените его реальным идентификатором. Чтобы получить это значение, обратитесь в [группу поддержки TigerConnect Secure Messenger](mailto:prosupport@tigertext.com). Можно также посмотреть шаблоны на панели **Базовая конфигурация SAML** на портале Azure.

1. На панели **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать подходящий **XML-файл метаданных федерации** из предложенных вариантов и сохранить его на своем компьютере.

    ![Элемент для скачивания XML-файла метаданных федерации](common/metadataxml.png)

1. Скопируйте требуемый URL-адрес из раздела **Настройка TigerConnect Secure Messenger**.

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

В этом разделе описано, как включить единый вход Azure для пользователя B. Simon, предоставив этому пользователю доступ к TigerConnect Secure Messenger.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **TigerConnect Secure Messenger**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.

   ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Ссылка "Добавить пользователя"](common/add-assign-user.png)

1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор роли** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-tigerconnect-secure-messenger-sso"></a>Настройка единого входа TigerConnect Secure Messenger

Чтобы настроить единый вход на стороне TigerConnect Secure Messenger, отправьте скачанный XML-файл метаданных федерации и URL-адреса, скопированные на портале Azure, в [группу поддержки TigerConnect Secure Messenger](mailto:prosupport@tigertext.com). Группа поддержки TigerConnect Secure Messenger обеспечит правильную настройку подключения для единого входа SAML с обеих сторон.

## <a name="create-a-tigerconnect-secure-messenger-test-user"></a>Создание тестового пользователя TigerConnect Secure Messenger

В этом разделе описано, как создать пользователя Britta Simon в приложении TigerConnect Secure Messenger. Чтобы добавить пользователя Britta Simon в TigerConnect Secure Messenger, обратитесь в [группу поддержки TigerConnect Secure Messenger](mailto:prosupport@tigertext.com). Перед использованием единого входа необходимо создать и активировать пользователей.

## <a name="test-sso"></a>Проверка единого входа

В этом разделе объясняется, как проверить конфигурацию единого входа Azure AD с помощью портала "Мои приложения".

Щелкнув **TigerConnect Secure Messenger** на портале "Мои приложения", вы автоматически выполните вход с подпиской TigerConnect Secure Messenger, для которой настроили единый вход. Дополнительные сведения о портале "Мои приложения" см. в статье [Access and use apps on the My Apps portal](../user-help/my-apps-portal-end-user-access.md) (Доступ к приложениям и их использование на портале "Мои приложения").

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список руководств по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)

- [Попробуйте использовать TigerConnect Secure Messenger с Azure AD](https://aad.portal.azure.com/)

- [Что такое управление сеансами в Microsoft Cloud App Security?](/cloud-app-security/proxy-intro-aad)