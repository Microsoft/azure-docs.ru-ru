---
title: Руководство по интеграции Azure Active Directory с Carbonite Endpoint Backup | Документация Майкрософт
description: Сведения о настройке единого входа Azure Active Directory в Carbonite Endpoint Backup.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/06/2019
ms.author: jeedes
ms.openlocfilehash: ff19275270e5b6572fb7d637b88c4736a3aa6ea0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92456489"
---
# <a name="tutorial-integrate-carbonite-endpoint-backup-with-azure-active-directory"></a>Руководство по интеграции Carbonite Endpoint Backup с Azure Active Directory

В этом руководстве описано, как интегрировать Carbonite Endpoint Backup с Azure Active Directory (Azure AD). Интеграция Carbonite Endpoint Backup с Azure AD позволяет:

* Контролировать в Azure AD доступ к Carbonite Endpoint Backup.
* Включить автоматический вход пользователей в Carbonite Endpoint Backup с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

Чтобы узнать больше об интеграции приложений SaaS с Azure AD, прочитайте статью [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Carbonite Endpoint Backup с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Carbonite Endpoint Backup поддерживает единый вход, инициированный **поставщиком услуг и поставщиком удостоверений**.

## <a name="adding-carbonite-endpoint-backup-from-the-gallery"></a>Добавление Carbonite Endpoint Backup из коллекции

Чтобы настроить интеграцию Carbonite Endpoint Backup с Azure AD, необходимо добавить Carbonite Endpoint Backup из коллекции в список управляемых приложений SaaS.

1. Войдите на [портал Azure](https://portal.azure.com) с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Carbonite Endpoint Backup**.
1. Выберите **Carbonite Endpoint Backup** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

Настройте и проверьте единый вход Azure AD в Carbonite Endpoint Backup с помощью тестового пользователя **B. Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Carbonite Endpoint Backup.

Чтобы настроить и проверить единый вход Azure AD в Carbonite Endpoint Backup, выполните действия в следующих стандартных блоках:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Carbonite Endpoint Backup](#configure-carbonite-endpoint-backup-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя в Carbonite Endpoint Backup](#create-carbonite-endpoint-backup-test-user)** требуется для того, чтобы в Carbonite Endpoint Backup существовал пользователь B. Simon, связанный с одноименным пользователем в Azure Active Directory.
6. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

### <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Carbonite Endpoint Backup** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **Идентификатор** введите любой из следующих URL-адресов:

    ```http
    https://red-us.mysecuredatavault.com
    https://red-apac.mysecuredatavault.com
    https://red-fr.mysecuredatavault.com
    https://red-emea.mysecuredatavault.com
    https://kamino.mysecuredatavault.com
    ```

    b. В текстовом поле **URL-адрес ответа** введите любой из следующих URL-адресов:

    ```http
    https://red-us.mysecuredatavault.com/AssertionConsumerService.aspx
    https://red-apac.mysecuredatavault.com/AssertionConsumerService.aspx
    https://red-fr.mysecuredatavault.com/AssertionConsumerService.aspx
    https://red-emea.mysecuredatavault.com/AssertionConsumerService.aspx
    ```

1. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    В текстовом поле **URL-адрес входа** введите любой из следующих URL-адресов:

    ```http
    https://red-us.mysecuredatavault.com/
    https://red-apac.mysecuredatavault.com/
    https://red-fr.mysecuredatavault.com/
    https://red-emea.mysecuredatavault.com/
    ```

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите пункт **Сертификат (Base64)** и щелкните **Скачать**, чтобы скачать сертификат. Сохраните этот сертификат на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

1. Скопируйте требуемый URL-адрес в разделе **Настройка Carbonite Endpoint Backup**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

### <a name="configure-carbonite-endpoint-backup-sso"></a>Настройка единого входа в Carbonite Endpoint Backup

1. Чтобы автоматизировать настройку Carbonite Endpoint Backup, необходимо установить **расширение браузера My Apps Secure Sign-in**, щелкнув **Установить расширение**.

    ![Расширение "Мои приложения"](common/install-myappssecure-extension.png)

2. После добавления расширения в браузер, щелкнув **Настройка Carbonite Endpoint Backup**, вас будет перенаправлено в приложение конечной точки резервной копии Carbonite. В нем укажите учетные данные администратора для входа в Carbonite Endpoint Backup. Расширение браузера автоматически настроит приложение и автоматизирует шаги 3–7.

    ![Настройка конфигурации](common/setup-sso.png)

3. Если вы хотите настроить Carbonite Endpoint Backup вручную, откройте новое окно веб-браузера и войдите на свой корпоративный сайт Carbonite Endpoint Backup с правами администратора и выполните следующие действия.

4. Щелкните **Company** (Организация) в левой области.

    ![Снимок экрана, на котором показан сайт Carbonite Endpoint с выбранным элементом "Company" (Организация)](media/carbonite-endpoint-backup-tutorial/configure1.png)

5. Щелкните **Single sign-on** (Единый вход).

    ![Снимок экрана, на котором показана страница "Company" (Организация) с выбранным параметром "Single sign-on" (Единый вход)](media/carbonite-endpoint-backup-tutorial/configure2.png)

6. Щелкните **Enable** (Включить), а затем выберите **Edit settings** (Изменить параметры), чтобы выполнить настройку.

    ![Снимок экрана, на котором показана вкладка "Single sign-on" (Единый вход) с выбранными параметрами" Enable" (Включить) и "Edit settings" (Изменить параметры)](media/carbonite-endpoint-backup-tutorial/configure3.png)

7. На странице параметров **единого входа** сделайте следующее:

    ![Снимок экрана, на котором показана вкладка "Single sign-on" (Единый вход) с информацией, описанной на этом шаге](media/carbonite-endpoint-backup-tutorial/configure4.png)

    1. В текстовое поле **Identity provider name** (Имя поставщика удостоверений) вставьте значение **идентификатора Azure AD**, скопированное на портале Azure.

    1. В текстовое поле **Identity provider URL** (URL-адрес входа поставщика удостоверений) вставьте значение **URL-адрес входа**, скопированное на портале Azure.

    1. Нажмите кнопку **Choose file** (Выбрать файл), чтобы передать **сертификат в формате Base64**, скачанный с портала Azure.

    1. Выберите команду **Сохранить**.

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

В этом разделе описано, как включить единый вход Azure для пользователя B. Simon, предоставив этому пользователю доступ к Carbonite Endpoint Backup.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. Из списка приложений выберите **Carbonite Endpoint Backup**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.

   ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Ссылка "Добавить пользователя"](common/add-assign-user.png)

1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор роли** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-carbonite-endpoint-backup-test-user"></a>Создание тестового пользователя в Carbonite Endpoint Backup

1. В другом окне веб-браузера войдите на свой корпоративный веб-сайт Carbonite Endpoint Backup в качестве администратора.

1. Щелкните **Users** (Пользователи) в области слева, а затем выберите **Add user** (Добавить пользователя).

    ![Снимок экрана, на котором показана страница Carbonite Endpoint с последовательно выбранными элементами "Users" (Пользователи) и "Add user" (Добавить пользователя)](media/carbonite-endpoint-backup-tutorial/adduser1.png)

1. На странице **добавления пользователя** сделайте следующее:

    ![Снимок экрана, на котором показана страница "Add user" (Добавление пользователя), где можно выполнить описанные здесь действия](media/carbonite-endpoint-backup-tutorial/adduser2.png)

    1. Укажите **адрес электронной почты**, **имя** и **фамилию** пользователя и предоставьте ему необходимые разрешения согласно требованиям организации.

    1. Нажмите кнопку **Добавить пользователя**.

### <a name="test-sso"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку Carbonite Endpoint Backup на Панели доступа, вы автоматически войдете в приложение Carbonite Endpoint Backup, для которого настроен единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)