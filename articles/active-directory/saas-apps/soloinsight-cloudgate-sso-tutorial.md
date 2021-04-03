---
title: Руководство. Интеграция Azure Active Directory с Soloinsight-CloudGate SSO | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Soloinsight-CloudGate SSO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/06/2019
ms.author: jeedes
ms.openlocfilehash: 193d779a2e9246fe36af4828251a15d593e80ac7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92514781"
---
# <a name="tutorial-integrate-soloinsight-cloudgate-sso-with-azure-active-directory"></a>Руководство. Интеграция Soloinsight-CloudGate SSO с Azure Active Directory

В этом руководстве описано, как интегрировать Soloinsight-CloudGate SSO с Azure Active Directory (Azure AD). Интеграция Soloinsight CloudGate SSO с Azure AD обеспечивает следующие возможности.

* С помощью Azure AD вы можете управлять доступом к Soloinsight-CloudGate SSO.
* Вы можете включить автоматический вход пользователей в Soloinsight-CloudGate SSO с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

Чтобы узнать больше об интеграции приложений SaaS с Azure AD, прочитайте статью [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, [здесь](https://azure.microsoft.com/pricing/free-trial/) вы можете получить бесплатную пробную версию сроком на один месяц.
* подписка Soloinsight-CloudGate SSO с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде. Soloinsight-CloudGate SSO поддерживает единый вход, инициируемый **поставщиком услуг**.

## <a name="adding-soloinsight-cloudgate-sso-from-the-gallery"></a>Добавление Soloinsight-CloudGate SSO из коллекции

Чтобы настроить интеграцию Soloinsight-CloudGate SSO с Azure AD, необходимо добавить Soloinsight-CloudGate SSO из коллекции в список управляемых приложений SaaS.

1. Войдите на [портал Azure](https://portal.azure.com) с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Soloinsight-CloudGate SSO**.
1. Выберите **Soloinsight-CloudGate SSO** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

Настройте и проверьте единый вход Azure AD в Soloinsight-CloudGate SSO с использованием тестового пользователя **Britta Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Soloinsight-CloudGate SSO.

Чтобы настроить и проверить единый вход Azure AD в Soloinsight-CloudGate SSO, потребуется выполнить действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Soloinsight-CloudGate SSO](#configure-soloinsight-cloudgate-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создайте тестового пользователя Azure AD](#create-an-azure-ad-test-user)** для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначьте тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** , чтобы позволить Britta Simon использовать единый вход в Azure AD.
5. **[Создание тестового пользователя Soloinsight-CloudGate SSO](#create-soloinsight-cloudgate-sso-test-user)** требуется для того, чтобы в Soloinsight-CloudGate SSO существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-sso)** необходима, чтобы убедиться в корректной работе конфигурации.

### <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Soloinsight-CloudGate SSO** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Базовая конфигурация SAML** введите значения для следующих полей.

    1. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<SUBDOMAIN>.sigateway.com/login`.

    1. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `https://<SUBDOMAIN>.sigateway.com/process/sso`.

   > [!NOTE]
   > Эти значения приведены для примера. Замените их фактическими значениями идентификатора и URL-адреса входа, которые описываются далее в разделе **Настройка единого входа Soloinsight-CloudGate SSO**.

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите пункт **Сертификат (Base64)** и щелкните **Скачать**, чтобы скачать сертификат. Сохраните этот сертификат на компьютере.

   ![Ссылка для скачивания сертификата](common/certificatebase64.png)

1. Требуемые URL-адреса можно скопировать из раздела **Настройка Soloinsight-CloudGate SSO**.

   ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

### <a name="configure-soloinsight-cloudgate-sso"></a>Настройка единого входа в Soloinsight-CloudGate SSO

1. Чтобы автоматизировать настройку в Soloinsight-CloudGate SSO, необходимо установить **расширение браузера My Apps Secure Sign-in Extension**, щелкнув **Установить расширение**.

    ![Расширение "Мои приложения"](common/install-myappssecure-extension.png)

2. Чтобы перейти к приложению Soloinsight-CloudGate SSO после добавления расширения в браузер, щелкните **Настройка Soloinsight-CloudGate SSO**. После этого укажите учетные данные администратора для входа в Soloinsight-CloudGate SSO. Расширение браузера автоматически настроит приложение и автоматизирует шаги 3–8.

    ![Настройка конфигурации](common/setup-sso.png)

3. Если необходимо вручную настроить Soloinsight-CloudGate SSO, откройте новое окно веб-браузера, зайдите на корпоративный сайт Soloinsight-CloudGate SSO с правами администратора и выполните следующие действия.

4. Чтобы получить значения, которые нужно вставить в поля на портале Azure при настройке базовой конфигурации SAML, войдите на веб-портал CloudGate со своими учетными данными и откройте параметры единого входа, выбрав **Home > Administration > System settings > General** ("Главная страница" > "Администрирование" > "Параметры системы" > "Общие").

    ![Параметры CloudGate SSO](./media/soloinsight-cloudgate-sso-tutorial/sso-main-settings.png)

5. **URL-адрес потребителя SAML**

    * Скопируйте ссылки напротив полей **Saml Consumer URL** (URL-адрес потребителя SAML) и **Redirect URL** (URL-адрес перенаправления). Вставьте их в разделе **Базовая конфигурация SAML** портала Azure в поля **Идентификатор (код сущности)** и **URL-адрес ответа** соответственно.

        ![SAMLIdentifier](./media/soloinsight-cloudgate-sso-tutorial/saml-identifier.png)

6. **Сертификат для подписи токена SAML**

    * Перейдите к источнику файла сертификата (Base64), который был скачан из списков сертификатов подписи SAML на портале Azure, и щелкните его правой кнопкой мыши. Выберите **Изменить с помощью Notepad++** в списке. 

        ![SAMLcertificate](./media/soloinsight-cloudgate-sso-tutorial/certificate-file.png)

    * Скопируйте его содержимое в файл сертификата (Base64) с помощью Notepad++.

        ![Копия сертификата](./media/soloinsight-cloudgate-sso-tutorial/certificate-copy.png)

    * Вставьте содержимое в поле **Certificate** (Сертификат) параметров единого входа на веб-портале CloudGate и нажмите кнопку Save (Сохранить).

        ![Портал сертификатов](./media/soloinsight-cloudgate-sso-tutorial/certificate-portal.png)

7. **Группа по умолчанию**

    * Выберите **Business Admin** (Администратор бизнеса) в раскрывающемся списке параметра **Default Group** (Группа по умолчанию) на веб-портале CloudGate.

        ![Группа по умолчанию](./media/soloinsight-cloudgate-sso-tutorial/default-group.png)

8. **Идентификатор AD и URL-адрес для входа**

    * Скопированный **URL-адрес для входа** из раздела параметров **Настройка Soloinsight-CloudGate SSO** на портале Azure нужно ввести в раздел параметров единого входа на веб-портале CloudGate.

    * Вставьте ссылку **URL-адрес для входа** с портала Azure в поле **AD Login URL** (URL-адрес для входа в AD) на веб-портале CloudGate.

    * Вставьте ссылку **Идентификатор Azure AD** с портала Azure в поле **AD Identifier** (Идентификатор AD) на веб-портале CloudGate.

        ![Вход в AD](./media/soloinsight-cloudgate-sso-tutorial/ad-login.png)

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD

В этом разделе описано, как создать на портале Azure тестового пользователя с именем Britta Simon.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.
1. В верхней части экрана выберите **Новый пользователь**.
1. В разделе **Свойства пользователя** выполните следующие действия.
   1. В поле **Имя** введите `Britta Simon`.  
   1. В поле **Имя пользователя** введите username@companydomain.extension. Например, `BrittaSimon@contoso.com`.
   1. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле **Пароль**.
   1. Нажмите кнопку **Создать**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к Soloinsight-CloudGate SSO.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Soloinsight CloudGate SSO**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.

   ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Ссылка "Добавить пользователя"](common/add-assign-user.png)

1. В диалоговом окне **Пользователи и группы** выберите **Britta Simon** из списка пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор роли** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-soloinsight-cloudgate-sso-test-user"></a>Создание тестового пользователя Soloinsight CloudGate SSO

Чтобы создать тестового пользователя, выберите **Employees** (Сотрудники) в главном меню веб-портала CloudGate и заполните форму для добавления нового сотрудника. Тестовому пользователю назначается уровень полномочий **Business Admin** (Администратор бизнеса). Нажмите кнопку **Create** (Создать) после заполнения всех нужных полей.

![Проверка сотрудника](./media/soloinsight-cloudgate-sso-tutorial/employee-test.png)

### <a name="test-sso"></a>Проверка единого входа

Щелкнув элемент "Soloinsight-CloudGate SSO" на Панели доступа, вы автоматически войдете в приложение Soloinsight-CloudGate SSO, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)