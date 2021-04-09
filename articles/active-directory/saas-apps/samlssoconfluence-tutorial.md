---
title: Руководство по Интеграция Azure Active Directory с SAML SSO for Confluence by resolution GmbH | Документация Майкрософт
description: Узнайте, как настроить единый вход для Azure Active Directory и SAML SSO for Confluence by resolution GmbH.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/04/2021
ms.author: jeedes
ms.openlocfilehash: 9290272920bbb20144f4e0d957ba2d9ce60d06a9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101651286"
---
# <a name="tutorial-azure-active-directory-integration-with-saml-sso-for-confluence-by-resolution-gmbh"></a>Руководство по Интеграция Azure Active Directory с SAML SSO for Confluence by resolution GmbH

В этом руководстве описано, как интегрировать SAML SSO for Confluence by resolution GmbH с Azure Active Directory (Azure AD). Интеграция SAML SSO for Confluence by resolution GmbH с Azure AD обеспечивает следующие возможности:

* контроль доступа к SAML SSO for Confluence by resolution GmbH с помощью Azure AD;
* включение автоматического входа пользователей в SAML SSO for Confluence by resolution GmbH (единый вход) с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка SAML SSO for Confluence by resolution GmbH с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* SAML SSO for Confluence by resolution GmbH поддерживает единый вход инициированного **пакета обновления и выдающей точки распространения**

## <a name="add-saml-sso-for-confluence-by-resolution-gmbh-from-the-gallery"></a>Добавление SAML SSO for Confluence by resolution GmbH из коллекции

Чтобы настроить интеграцию SAML SSO for Confluence by resolution GmbH в Azure AD, необходимо добавить SAML SSO for Confluence by resolution GmbH из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **SAML SSO for Confluence by resolution GmbH**.
1. Выберите **SAML SSO for Confluence by resolution GmbH** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-saml-sso-for-confluence-by-resolution-gmbh"></a>Настройка и проверка единого входа Azure AD для SAML SSO for Confluence by resolution GmbH

Настройте и проверьте единый вход SAML SSO for Confluence by resolution GmbH с помощью тестового пользователя **B.Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в SAML SSO for Confluence by resolution GmbH.

Чтобы настроить и проверить единый вход Azure AD в SAML SSO for Confluence by resolution GmbH, выполните следующие действия:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
2. **[Настройка единого входа в SAML SSO for Confluence by resolution GmbH](#configure-saml-sso-for-confluence-by-resolution-gmbh-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя SAML SSO for Confluence by resolution GmbH](#create-saml-sso-for-confluence-by-resolution-gmbh-test-user)** требуется, чтобы создать в SAML SSO for Confluence by resolution GmbH пользователя Britta Simon, связанного с соответствующим пользователем в Azure AD.
6. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **SAML SSO for Confluence by resolution GmbH** найдите раздел **Управление** и выберите элемент **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `https://<server-base-url>/plugins/servlet/samlsso`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<server-base-url>/plugins/servlet/samlsso`.

1. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://<server-base-url>/plugins/servlet/samlsso`.

    > [!NOTE]
    > Эти значения приведены для примера. Замените их фактическими значениями идентификатора, URL-адреса ответа и URL-адреса входа. Обратитесь к [группе поддержки SAML SSO for Confluence by resolution GmbH](https://www.resolution.de/go/support) для получения этих значений. Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

4. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)


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

В этом разделе объясняется, как позволить пользователю B.Simon использовать единый вход Azure путем предоставления доступа к SAML SSO for Confluence by resolution GmbH.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. Из списка приложений выберите **SAML SSO for Confluence by resolution GmbH**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.


## <a name="configure-saml-sso-for-confluence-by-resolution-gmbh-sso"></a>Настройка единого входа для SAML SSO for Confluence by resolution GmbH

1. В другом окне браузера войдите на **портал администрирования SAML SSO for Confluence by resolution GmbH** с правами администратора.

2. Наведите указатель мыши на шестеренку и щелкните **Add-ons** (Надстройки).
    
    ![Снимок экрана: выбранные в раскрывающемся меню значок шестеренки и элемент Add-ons (Надстройки).](./media/saml-sso-confluence-tutorial/add-on-1.png)

3. Вы перейдете на страницу доступа с правами администратора. Введите пароль и нажмите кнопку **Confirm** (Подтвердить).

    ![Снимок экрана: страница доступа администратора с нажатой кнопкой Confirm (Подтвердить).](./media/saml-sso-confluence-tutorial/add-on-2.png)

4. В разделе **ATLASSIAN MARKETPLACE** щелкните **Find new add-ons** (Найти новые надстройки). 

    ![Снимок экрана: вкладка Attlassian Marketplace с выбранным пунктом Find new add-ons (Найти новые надстройки).](./media/saml-sso-confluence-tutorial/add-on.png)

5. Найдите подключаемый модуль **SAML Single Sign On (SSO) for Confluence** и нажмите кнопку **Install** (Установить), чтобы установить новый подключаемый модуль SAML.

    ![Снимок экрана: страница Find new add-ons (Найти новые надстройки) с модулем SAML Single Sign On (SSO) for Confluence в поле поиска и нажатой кнопкой Install (Установить).](./media/saml-sso-confluence-tutorial/add-on-7.png)

6. Начнется установка подключаемого модуля. Щелкните **Закрыть**.

    ![Снимок экрана: диалоговое окно установки.](./media/saml-sso-confluence-tutorial/add-on-8.png)

    ![Снимок экрана: диалоговое окно установки и готовности к использованию с выбранным действием Close (Закрыть).](./media/saml-sso-confluence-tutorial/add-on-9.png)

7.  Нажмите кнопку **Управление**.

    ![Снимок экрана: страница приложения SAML Single Sign On (SSO) for Confluence с выбранной кнопкой управления.](./media/saml-sso-confluence-tutorial/add-on-10.png)
    
8. Щелкните **Configure** (Настройка), чтобы настроить новый подключаемый модуль.

    ![Снимок экрана: страница управления с выбранной кнопкой Configure (Настройка).](./media/saml-sso-confluence-tutorial/add-on-11.png)

9. Этот новый подключаемый модуль можно также найти на вкладке **USERS & SECURITY** (Пользователи и безопасность).

    ![Снимок экрана: вкладка Users & Security (Пользователи и безопасность) с выбранным элементом SAML SingleSignOn.](./media/saml-sso-confluence-tutorial/add-on-3.png)
    
10. На странице **SAML SingleSignOn Plugin Configuration** (Конфигурация подключаемого модуля единого входа SAML) нажмите кнопку **Add new IdP** (Добавить новый поставщик удостоверений), чтобы настроить параметры поставщика удостоверений.

    ![Снимок экрана: страница SAML SingleSignOn Plugin Configuration (Конфигурация подключаемого модуля единого входа SAML) с выбранной кнопкой Add new IdP (Добавить новый поставщик удостоверений).](./media/saml-sso-confluence-tutorial/add-on-4.png)

11. На странице **Choose your SAML Identity Provider** (Выбор поставщика удостоверений SAML) выполните следующие действия:

    ![Снимок экрана: страница Choose your SAML Identity Provider (Выбор поставщика удостоверений SAML) с выделенными текстовыми полями IdP Type (Тип поставщика удостоверений), Name (Имя) и Description (Описание).](./media/saml-sso-confluence-tutorial/add-on-5-a.png)
 
    а. Для типа поставщика удостоверений выберите значение **Azure AD**.
    
    b. Добавьте **имя** поставщика удостоверений (например, Azure AD).
    
    c. Добавьте **описание** поставщика удостоверений (например, Azure AD).
    
    d. Щелкните **Далее**.
    
12. На странице **Identity provider configuration** (Настройка поставщика удостоверений) нажмите кнопку **Next** (Далее).

    ![Снимок экрана: страница Identity provider configuration (Конфигурация поставщика удостоверений) с нажатой кнопкой Next (Далее).](./media/saml-sso-confluence-tutorial/add-on-5-b.png)

13. На странице **Import SAML IdP Metadata** (Импорт метаданных поставщика удостоверений SAML) выполните следующие действия:

    ![Снимок экрана: страница Import SAML IdP Metadata (Импорт метаданных поставщика удостоверений SAML) с нажатыми кнопками Import (Импортировать), Load File (Загрузить файл) и Next (Далее).](./media/saml-sso-confluence-tutorial/add-on-5-c.png)

    a. Нажмите кнопку **Load File** (Загрузить файл) и выберите XML-файл метаданных, который вы скачали на шаге 5.

    b. Нажмите кнопку **Import** (Импортировать).
    
    c. Дождитесь завершения импорта.
    
    d. Нажмите кнопку **Next** (Далее).
    
14. На странице **User ID attribute and transformation** (Атрибут и преобразование идентификатора пользователя) нажмите кнопку **Next** (Далее).

    ![Снимок экрана: страница User ID attribute and transformation (Атрибут и преобразование идентификатора пользователя) с нажатой кнопкой Next (Далее).](./media/saml-sso-confluence-tutorial/add-on-5-d.png)
    
15. На странице **User creation and update** (Создание и изменение пользователя) нажмите кнопку **Save & Next** (Сохранить и продолжить), чтобы сохранить параметры.   
    
    ![Снимок экрана: страница User creation and update (Создание и изменение пользователя) с нажатой кнопкой Save & Next (Сохранить и продолжить).](./media/saml-sso-confluence-tutorial/add-on-6-a.png)
    
16. На странице **Test your settings** (Проверка параметров) нажмите кнопку **Skip test & configure manually** (Пропустить проверку и настроить вручную), чтобы не выполнять проверку на этом этапе. Проверка будет выполнена на одном из следующих этапов, и для этого потребуется выполнить некоторые настройки на портале Azure. 
    
    ![Снимок экрана: страница Test your settings (Проверка параметров) с нажатой кнопкой Skip test & configure manually (Пропустить проверку и настроить вручную).](./media/saml-sso-confluence-tutorial/add-on-6-b.png)
    
17. В открывшемся окне с сообщением **Skipping the test means...** (Если вы пропустите проверку...) нажмите кнопку **OK**.
    
    ![Настройка единого входа](./media/saml-sso-confluence-tutorial/add-on-6-c.png)


### <a name="create-saml-sso-for-confluence-by-resolution-gmbh-test-user"></a>Создание тестового пользователя SAML SSO for Confluence by resolution GmbH

Чтобы пользователи Azure AD могли входить в SAML SSO for Confluence by resolution GmbH, их необходимо подготовить в SAML SSO for Confluence by resolution GmbH.  
Подготовка в SAML SSO for Confluence by resolution GmbH выполняется вручную.

**Чтобы подготовить учетную запись пользователя, сделайте следующее:**

1. Войдите на свой корпоративный сайт SAML SSO for Confluence by resolution GmbH с правами администратора.

2. Наведите указатель мыши на шестеренку и щелкните **User management** (Управление пользователями).

    ![Снимок экрана: выбранные значок шестеренки и пункт User management (Управление пользователями) в меню.](./media/saml-sso-confluence-tutorial/user-1.png) 

3. В разделе "Users" (Пользователи) выберите вкладку **Add users** (Добавление пользователей). На диалоговой странице **Add a User** (Добавление пользователя) выполните следующее.

    ![Добавление сотрудника](./media/saml-sso-confluence-tutorial/user-2.png) 

    а. В текстовом поле **Username** (Имя пользователя) введите электронный адрес пользователя, например Britta Simon.

    b. В текстовом поле **Full Name** (Полное имя) введите полное имя пользователя, например Britta Simon.

    c. В текстовом поле **Email** (Электронная почта) введите адрес электронной почты пользователя, например Brittasimon@contoso.com.

    d. В текстовом поле **Password** (Пароль) введите пароль пользователя Britta Simon.

    д) Щелкните **Confirm Password** (Подтвердить пароль) и повторно введите пароль.
    
    е) Нажмите кнопку **Добавить**.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

#### <a name="sp-initiated"></a>Инициация поставщиком услуг:

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в SAML SSO for Confluence by resolution GmbH, где можно инициировать поток входа.  

* Перейдите по URL-адресу для входа в SAML SSO for Confluence by resolution GmbH и инициируйте поток входа.

#### <a name="idp-initiated"></a>Вход, инициированный поставщиком удостоверений

* На портале Azure выберите элемент **Тестировать приложение**, и вы автоматически войдете в приложение SAML SSO for Confluence by resolution GmbH, для которого настроили единый вход. 

Вы можете также использовать портал "Мои приложения" корпорации Майкрософт для тестирования приложения в любом режиме. Щелкнув плитку SAML SSO for Confluence by resolution GmbH на портале "Мои приложения", вы будете перенаправлены на страницу входа приложения для инициации потока входа (при настройке в режиме поставщика услуг) или автоматически войдете в приложение SAML SSO for Confluence by resolution GmbH, для которого настроен единый вход (при настройке в режиме поставщика удостоверений). Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Дальнейшие действия

После настройки SAML SSO for Confluence by resolution GmbH можно применить функцию управления сеансами, которая в реальном времени защищает конфиденциальные данные вашей организации от кражи и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).