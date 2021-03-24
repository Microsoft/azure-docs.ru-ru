---
title: Руководство по интеграции Azure Active Directory со SkyDesk Email | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в SkyDesk Email.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/07/2019
ms.author: jeedes
ms.openlocfilehash: 30e94c9737241ff49c29898adcc5e50c6b73a9b1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92516044"
---
# <a name="tutorial-azure-active-directory-integration-with-skydesk-email"></a>Учебник. Интеграция Azure Active Directory со SkyDesk Email

В этом руководстве описано, как интегрировать SkyDesk Email с Azure Active Directory (Azure AD).
Интеграция SkyDesk Email с Azure AD дает приведенные ниже преимущества.

* С помощью Azure AD вы можете контролировать доступ к SkyDesk Email.
* Вы можете включить автоматический вход пользователей (единый вход) в SkyDesk Email с помощью учетной записи Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD со SkyDesk Email, вам потребуется:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка SkyDesk Email с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* SkyDesk Email поддерживает единый вход, инициированный **поставщиком услуг**.

## <a name="adding-skydesk-email-from-the-gallery"></a>Добавление SkyDesk Email из коллекции

Чтобы настроить интеграцию SkyDesk Email с Azure AD, необходимо добавить приложение SkyDesk Email из коллекции в список управляемых приложений SaaS.

**Чтобы добавить SkyDesk Email из коллекции, выполните указанные ниже действия.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **SkyDesk Email**, выберите **SkyDesk Email** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

     ![SkyDesk Email в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в SkyDesk Email с использованием тестового пользователя **Britta Simon**.
Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в SkyDesk Email.

Чтобы настроить и проверить единый вход в Azure AD в SkyDesk Email, вам потребуется выполнить действия в приведенных ниже стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в SkyDesk Email](#configure-skydesk-email-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя SkyDesk Email](#create-skydesk-email-test-user)** требуется для того, чтобы в SkyDesk Email существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход в Azure AD в SkyDesk Email, выполните указанные ниже действия.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **SkyDesk Email** выберите **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    ![Сведения о домене и URL-адресах единого входа для приложения SkyDesk Email](common/sp-signonurl.png)

    В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://mail.skydesk.jp/portal/<companyname>`.

    > [!NOTE]
    > Это значение приведено для примера. Вместо него необходимо указать фактический URL-адрес входа. Чтобы получить это значение, обратитесь к [группе поддержки клиентов SkyDesk Email](https://www.skydesk.jp/apps/support/). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (Base64)** из предложенных вариантов, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

6. Скопируйте требуемый URL-адрес из раздела **Настройка SkyDesk Email**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

### <a name="configure-skydesk-email-single-sign-on"></a>Настройка единого входа в SkyDesk Email

1. В другом веб-браузере войдите в учетную запись SkyDesk Email от имени администратора.

1. В меню в верхней части страницы щелкните **Setup** (Настройка) и выберите **Org** (Организация).

    ![Снимок экрана: элемент Org (Организация), выбранный в меню Setup (Настройка).](./media/skydeskemail-tutorial/tutorial_skydeskemail_51.png)
  
1. На панели слева выберите **Domains** (Домены).

    ![Снимок экрана: элемент Org (Организация), выбранный на панели управления.](./media/skydeskemail-tutorial/tutorial_skydeskemail_53.png)

1. Щелкните **Add Domain** (Добавить домен).

    ![Снимок экрана: выбранный элемент Add Domain (Добавить домен).](./media/skydeskemail-tutorial/tutorial_skydeskemail_54.png)

1. Введите доменное имя, а затем проверьте домен.

    ![Снимок экрана: вкладка добавления домена, на которой можно указать домен.](./media/skydeskemail-tutorial/tutorial_skydeskemail_55.png)

1. На панели слева выберите пункт **SAML Authentication** (Аутентификация SAML).

    ![Снимок экрана: элемент SAML Authentication (Аутентификация SAML), выбранный на панели управления.](./media/skydeskemail-tutorial/tutorial_skydeskemail_52.png)

1. На странице **SAML Authentication** выполните указанные ниже действия.

    ![Снимок экрана: диалоговое окно со сведениями об аутентификации SAML, где можно ввести описанные значения.](./media/skydeskemail-tutorial/tutorial_skydeskemail_56.png)

    > [!NOTE]
    > Для использования проверки подлинности на основе SAML у вас должен быть настроен **проверенный домен** или **URL-адрес портала**. Для URL-адреса портала можно настроить уникальное имя.

    ![Снимок экрана: поле URL-адрес портала, в котором вводится имя.](./media/skydeskemail-tutorial/tutorial_skydeskemail_57.png)

    а. В текстовое поле **Login URL** (URL-адрес входа) вставьте **URL-адрес входа**, скопированный на портале Azure.

    b. В текстовое поле **Logout URL** (URL-адрес выхода) вставьте **URL-адрес выхода**, скопированный с портала Azure.

    c. **Изменить URL-адрес пароля** является необязательным, поэтому оставьте его пустым.

    d. Щелкните **Get Key From File** (Получить ключ из файла), чтобы выбрать сертификат, скачанный с портала Azure, затем нажмите кнопку **Open** (Открыть), чтобы передать этот сертификат.

    д) В поле **Algorithm** (Алгоритм) задайте значение **RSA**.

    е) Нажмите кнопку **ОК** , чтобы сохранить изменения.

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

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к SkyDesk Email.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **SkyDesk Email**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **SkyDesk Email**.

    ![Ссылка на SkyDesk Email в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-skydesk-email-test-user"></a>Создание тестового пользователя в SkyDesk Email

В этом разделе описано, как создать пользователя Britta Simon в приложении SkyDesk Email.

На левой панели в SkyDesk Email выберите пункт **User Access** (Доступ пользователя), а затем введите свое имя пользователя.

![Снимок экрана: элемент User Access (Доступ пользователя), выбранный на панели управления.](./media/skydeskemail-tutorial/tutorial_skydeskemail_58.png)

> [!NOTE]
> Если вам нужно массово создать пользователей, обратитесь к [группе поддержки клиентов SkyDesk Email](https://www.skydesk.jp/apps/support/).

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку SkyDesk Email на Панели доступа, вы автоматически войдете в приложение SkyDesk Email, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)