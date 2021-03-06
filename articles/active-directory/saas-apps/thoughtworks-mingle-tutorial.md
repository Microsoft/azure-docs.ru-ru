---
title: Руководство по интеграции Azure Active Directory с Thoughtworks Mingle | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Thoughtworks Mingle.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: jeedes
ms.openlocfilehash: c3eeb1577e628965e3e5a35fa20c072224383149
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92514631"
---
# <a name="tutorial-azure-active-directory-integration-with-thoughtworks-mingle"></a>Учебник. Интеграция Azure Active Directory с Thoughtworks Mingle

В этом руководстве описано, как интегрировать Thoughtworks Mingle с Azure Active Directory (Azure AD).
Интеграция Azure AD с Thoughtworks Mingle обеспечивает следующие преимущества.

* С помощью Azure AD вы можете контролировать доступ к Thoughtworks Mingle.
* Вы можете включить автоматический вход пользователей в Thoughtworks Mingle (единый вход) с помощью учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с Thoughtworks Mingle, вам потребуется:

* Подписка Azure AD. (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка Thoughtworks Mingle с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Thoughtworks Mingle поддерживает единый вход, инициированный **поставщиком услуг**.

## <a name="adding-thoughtworks-mingle-from-the-gallery"></a>Добавление Thoughtworks Mingle из коллекции

Чтобы настроить интеграцию Thoughtworks Mingle с Azure AD, необходимо добавить Thoughtworks Mingle из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Thoughtworks Mingle из коллекции, сделайте следующее:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Thoughtworks Mingle**, выберите **Thoughtworks Mingle** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

     ![ThoughtWorks Mingle в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в Thoughtworks Mingle с использованием тестового пользователя **Britta Simon**.
Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Thoughtworks Mingle.

Чтобы настроить и проверить единый вход Azure AD в Thoughtworks Mingle, вам потребуется выполнить действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Thoughtworks Mingle](#configure-thoughtworks-mingle-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя Thoughtworks Mingle](#create-thoughtworks-mingle-test-user)** требуется для того, чтобы в Thoughtworks Mingle существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Thoughtworks Mingle, сделайте следующее.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Thoughtworks Mingle** выберите **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    ![Сведения о домене и URL-адресах единого входа для приложения Thoughtworks Mingle](common/sp-signonurl.png)

    В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://<companyname>.mingle.thoughtworks.com`.

    > [!NOTE]
    > Это значение приведено для примера. Вместо него необходимо указать фактический URL-адрес входа. Чтобы получить это значение, обратитесь в [службу поддержки клиентов Thoughtworks Mingle](https://support.thoughtworks.com/hc/categories/201743486-Mingle-Community-Support). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

6. Требуемый URL-адрес можно скопировать из раздела **Настройка Thoughtworks Mingle**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

### <a name="configure-thoughtworks-mingle-single-sign-on"></a>Настройка единого входа Thoughtworks Mingle

1. Войдите на корпоративный веб-сайт **Thoughtworks Mingle** в качестве администратора.

2. Щелкните вкладку **Администратор**, а затем щелкните **Настройка единого входа**.
   
    ![Вкладка администратора](./media/thoughtworks-mingle-tutorial/ic785157.png "Конфигурация единого входа")

3. В разделе **Настройка единого входа** выполните следующее:
   
    ![Конфигурация единого входа](./media/thoughtworks-mingle-tutorial/ic785158.png "Конфигурация единого входа")
    
    a. Чтобы передать скачанный файл метаданных, щелкните **Выбрать файл**. 

    b. Щелкните **Сохранить изменения**.

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD 

Цель этого раздела — создать на портале Azure тестового пользователя с именем Britta Simon.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.

    ![Ссылки "Пользователи и группы" и "Все пользователи"](common/users.png)

2. В верхней части экрана выберите **Новый пользователь**.

    ![Кнопка "Новый пользователь"](common/new-user.png)

3. В разделе свойств пользователя сделайте следующее:

    ![Диалоговое окно "Пользователь"](common/user-properties.png)

    а. В поле **Имя** введите **BrittaSimon**.
  
    b. В поле **Имя пользователя** введите brittasimon@yourcompanydomain.extension. Например BrittaSimon@contoso.com.

    c. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле "Пароль".

    d. Нажмите кнопку **Создать**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как предоставить пользователю Britta Simon доступ к Thoughtworks Mingle, чтобы он мог использовать единый вход Azure.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **Thoughtworks Mingle**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Thoughtworks Mingle**.

    ![Ссылка на Thoughtworks Mingle в списке "Приложения"](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-thoughtworks-mingle-test-user"></a>Создание тестового пользователя Thoughtworks Mingle

Чтобы пользователи AAD могли войти систему, их необходимо подготовить для приложения Thoughtworks Mingle, используя их имена пользователей Azure Active Directory. В случае с Thoughtworks Mingle подготовка выполняется вручную.

**Чтобы настроить подготовку учетных записей пользователей, выполните следующие действия.**

1. Войдите на корпоративный веб-сайт Thoughtworks Mingle в качестве администратора.

2. Щелкните **Профиль**.
   
    ![Ваш первый проект](./media/thoughtworks-mingle-tutorial/ic785160.png "Ваш первый проект")

3. Щелкните вкладку **Администратор**, затем щелкните **Пользователи**.
   
    ![Пользователи](./media/thoughtworks-mingle-tutorial/ic785161.png "Пользователи")

4. Нажмите **Новый пользователь**.
   
    ![Новый пользователь](./media/thoughtworks-mingle-tutorial/ic785162.png "Новый пользователь")

5. На странице диалогового окна **Новый пользователь** выполните следующие действия.
   
    ![Диалоговое окно нового пользователя](./media/thoughtworks-mingle-tutorial/ic785163.png "Новый пользователь")  
 
    a. Введите в текстовые поля **Sign-in name** (Учетное имя), **Display name** (Отображаемое имя), **Choose password** (Выберите пароль) и **Confirm password** (Подтвердите пароль) соответствующие данные действующей учетной записи Azure AD, которую вы хотите подготовить. 

    b. Для параметра **Тип пользователя** выберите значение **Полноправный пользователь**.

    c. Щелкните **Создать этот профиль**.

>[!NOTE]
>Вы можете использовать любые другие инструменты создания учетных записей пользователя Thoughtworks Mingle или API-интерфейсы, предоставляемые Thoughtworks Mingle для подготовки учетных записей пользователей AAD.
> 

### <a name="test-single-sign-on"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку Thoughtworks Mingle на Панели доступа, вы автоматически войдете в приложение Thoughtworks Mingle, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)