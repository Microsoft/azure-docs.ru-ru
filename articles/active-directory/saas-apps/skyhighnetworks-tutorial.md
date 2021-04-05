---
title: Руководство по Интеграция Azure Active Directory с MVISION Cloud Azure AD SSO Configuration | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и MVISION Cloud Azure AD SSO Configuration.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/23/2020
ms.author: jeedes
ms.openlocfilehash: 456348f5492af44274a37a6400fb8d95e9bd55a4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96015151"
---
# <a name="tutorial-integrate-mvision-cloud-azure-ad-sso-configuration-with-azure-active-directory"></a>Руководство по Интеграция MVISION Cloud Azure AD SSO Configuration с Azure Active Directory

В этом руководстве объясняется, как интегрировать MVISION Cloud Azure AD SSO Configuration с Azure Active Directory (Azure AD). Интеграция MVISION Cloud Azure AD SSO Configuration с Azure AD предоставит следующие возможности:

* Вы сможете через Azure AD управлять доступом к MVISION Cloud Azure AD SSO Configuration.
* Вы позволите пользователям автоматически входить в MVISION Cloud Azure AD SSO Configuration с учетными записями Azure AD.
* Централизованное управление учетными записями через портал Azure.

Чтобы узнать больше об интеграции приложений SaaS с Azure AD, прочитайте статью [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, [здесь](https://azure.microsoft.com/pricing/free-trial/) вы можете получить бесплатную пробную версию сроком на один месяц.
* Подписка MVISION Cloud Azure AD SSO Configuration с поддержкой единого входа.


## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* MVISION Cloud Azure AD SSO Configuration поддерживает единый вход, инициированный **поставщиком услуг и поставщиком удостоверений**.
* После настройки Dropbox можете применить функцию управления сеансом, которая защищает от хищения и несанкционированного доступа к конфиденциальным данным вашей организации в реальном времени. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).

## <a name="adding-mvision-cloud-azure-ad-sso-configuration-from-the-gallery"></a>Добавление MVISION Cloud Azure AD SSO Configuration из коллекции

Чтобы настроить интеграцию MVISION Cloud Azure AD SSO Configuration с Azure AD, необходимо добавить MVISION Cloud Azure AD SSO Configuration из коллекции в список управляемых приложений SaaS.

1. Войдите на [портал Azure](https://portal.azure.com) с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** введите **MVISION Cloud Azure AD SSO Configuration** в поле поиска.
1. Выберите **MVISION Cloud Azure AD SSO Configuration** на панели результатов, а затем добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

Настройте и проверьте единый вход Azure AD в MVISION Cloud Azure AD SSO Configuration с помощью тестового пользователя с именем **Britta Simon**. Чтобы обеспечить единый вход, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в MVISION Cloud Azure AD SSO Configuration.

Чтобы настроить и проверить единый вход в MVISION Cloud Azure AD SSO Configuration, выполните действия из следующих стандартных блоков:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
    4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в MVISION Cloud Azure AD SSO Configuration](#configure-mvision-cloud-azure-ad-sso-configuration-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя MVISION Cloud Azure AD SSO Configuration](#create-mvision-cloud-azure-ad-sso-configuration-test-user)** требуется для того, чтобы в MVISION Cloud Azure AD SSO Configuration существовал пользователь Britta Simon, связанный с представлением этого пользователя в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

### <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Datadog** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)


4. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** выполните следующие действия.

    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `https://<ENV>.myshn.net/shndash/saml/Azure_SSO`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<ENV>.myshn.net/shndash/response/saml-postlogin`.

5. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    ![Сведения о домене и URL-адресах единого входа в приложение MVISION Cloud Azure AD SSO Configuration](common/metadata-upload-additional-signon.png)

    В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://<ENV>.myshn.net/shndash/saml/Azure_SSO`.

    > [!NOTE]
    > Эти значения приведены для примера. Замените их фактическими значениями идентификатора, URL-адреса ответа и URL-адреса входа. Чтобы получить эти значения, обратитесь в [службу технической поддержки клиентов MVISION Cloud Azure AD SSO Configuration](mailto:support@skyhighnetworks.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

6. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (Base64)** из предложенных вариантов, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

7. Скопируйте требуемые URL-адреса из раздела **настройки MVISION Cloud Azure AD SSO Configuration**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)


### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD 

Цель этого раздела — создать на портале Azure тестового пользователя с именем Britta Simon.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.

    ![Ссылки "Пользователи и группы" и "Все пользователи"](common/users.png)

2. В верхней части экрана выберите **Новый пользователь**.

    ![Кнопка "Новый пользователь"](common/new-user.png)

3. В разделе свойств пользователя сделайте следующее:

    ![Диалоговое окно "Пользователь"](common/user-properties.png)

    а. В поле **Имя** введите **BrittaSimon**.
  
    b. В поле **Имя пользователя** введите **brittasimon@yourcompanydomain.extension** .  
    Например BrittaSimon@contoso.com.

    c. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле "Пароль".

    d. Нажмите кнопку **Создать**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе объясняется, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив доступ к MVISION Cloud Azure AD SSO Configuration.

1. На портале Azure последовательно выберите элементы **Корпоративные приложения**, **Все приложения**, а затем — **MVISION Cloud Azure AD SSO Configuration**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **MVISION Cloud Azure AD SSO Configuration**.

    ![Ссылка на MVISION Cloud Azure AD SSO Configuration в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.


## <a name="configure-mvision-cloud-azure-ad-sso-configuration-sso"></a>Настройка единого входа в MVISION Cloud Azure AD SSO Configuration

Чтобы настроить единый вход на стороне **MVISION Cloud Azure AD SSO Configuration**, нужно отправить скачанный **сертификат (Base64)** и соответствующие URL-адреса, скопированные с портала Azure, в [службу технической поддержки MVISION Cloud Azure AD SSO Configuration](mailto:support@skyhighnetworks.com). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.


### <a name="create-mvision-cloud-azure-ad-sso-configuration-test-user"></a>Создание тестового пользователя в MVISION Cloud Azure AD SSO Configuration

В рамках этого раздела вы создадите пользователя с именем B.Simon в MVISION Cloud Azure AD SSO Configuration. Чтобы добавить пользователей на платформу MVISION Cloud Azure AD SSO Configuration, обратитесь в [службу поддержки MVISION Cloud Azure AD SSO Configuration](mailto:support@skyhighnetworks.com). Перед использованием единого входа необходимо создать и активировать пользователей.

### <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув элемент "MVISION Cloud Azure AD SSO Configuration" на Панели доступа, вы автоматически войдете в приложение MVISION Cloud Azure AD SSO Configuration, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)

- [Ознакомление с MVISION Cloud Azure AD SSO Configuration в сочетании с Azure Active Directory](https://aad.portal.azure.com/)

- [Что такое управление сеансами в Microsoft Cloud App Security?](/cloud-app-security/proxy-intro-aad)