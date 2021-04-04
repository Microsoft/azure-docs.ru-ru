---
title: Руководство по интеграции Azure Active Directory с Wizergos Productivity Software | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Wizergos Productivity Software.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/28/2019
ms.author: jeedes
ms.openlocfilehash: b4cddae25bbf7ff113d2ea67700e28eb81c0e7c4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92638029"
---
# <a name="tutorial-azure-active-directory-integration-with-wizergos-productivity-software"></a>Руководство. Интеграция Azure Active Directory с Wizergos Productivity Software

В этом руководстве описано, как интегрировать Wizergos Productivity Software с Azure Active Directory (Azure AD).
Интеграция Wizergos Productivity Software с Azure AD обеспечивает следующие преимущества:

* С помощью Azure AD можно контролировать доступ к Wizergos Productivity Software.
* Вы можете включить автоматический вход пользователей в Wizergos Productivity Software (единый вход) с использованием учетных записей Azure Active Directory.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD с Wizergos Productivity Software, вам потребуется следующее:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/));
* подписка на Wizergos Productivity Software с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Приложение Wizergos Productivity Software поддерживает единый вход, инициированный **поставщиком удостоверений**.

## <a name="adding-wizergos-productivity-software-from-the-gallery"></a>Добавление Wizergos Productivity Software из коллекции.

Чтобы настроить интеграцию Wizergos Productivity Software с Azure AD, вам потребуется добавить Wizergos Productivity Software из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Wizergos Productivity Software из коллекции, сделайте следующее:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Wizergos Productivity Software**, выберите **Wizergos Productivity Software** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

     ![Wizergos Productivity Software в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure Active Directory в Wizergos Productivity Software с использованием тестового пользователя **Britta Simon**.
Для обеспечения работы единого входа необходимо установить связь между пользователем Azure Active Directory и соответствующим пользователем в Wizergos Productivity Software.

Чтобы настроить и проверить единый вход в Azure AD в Wizergos Productivity Software, вам потребуется выполнить действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Wizergos Productivity Software](#configure-wizergos-productivity-software-single-sign-on)** требуется, чтобы определить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя Wizergos Productivity Software](#create-wizergos-productivity-software-test-user)** требуется для того, чтобы в приложении Wizergos Productivity Software существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure Active Directory.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход в Azure Active Directory с помощью Wizergos Productivity Software, сделайте следующее:

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Wizergos Productivity Software** выберите **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    ![Сведения о домене и URL-адресах единого входа приложения Wizergos Productivity Software](common/idp-identifier.png)

    В текстовом поле **Идентификатор** введите URL-адрес: `https://www.wizergos.net`

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (Base64)** из предложенных вариантов, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

6. Требуемые URL-адреса можно скопировать из раздела **Настройка Wizergos Productivity Software**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

### <a name="configure-wizergos-productivity-software-single-sign-on"></a>Настройка единого входа в Wizergos Productivity Software

1. В другом окне веб-браузера войдите на веб-сайт Wizergos Productivity Software своей компании в качестве администратора.

2. В меню с тремя полосками выберите **Администратор**.

    ![Снимок экрана: в меню выбран значок администрирования.](./media/wizergosproductivitysoftware-tutorial/tutorial_wizergosproductivitysoftware_000.png)

3. На странице администрирования в меню слева выберите **Проверка подлинности** и нажмите кнопку **Azure AD**.

    ![Снимок экрана: на странице настройки проверки подлинности выбран вариант Azure AD.](./media/wizergosproductivitysoftware-tutorial/tutorial_wizergosproductivitysoftware_002.png)

4. В разделе **Проверка подлинности** выполните следующие действия.

    ![Снимок экрана: страница настройки проверки подлинности, где можно ввести указанные значения.](./media/wizergosproductivitysoftware-tutorial/tutorial_wizergosproductivitysoftware_003.png)
    
    а. Чтобы отправить сертификат, скачанный с Azure AD, нажмите кнопку **UPLOAD** (Отправить).
    
    b. В текстовое поле **URL-адрес издателя** вставьте значение **идентификатора Azure AD**, скопированное на портале Azure.
    
    c. В текстовое поле **URL-адрес для единого входа** вставьте значение **URL-адреса входа**, скопированное на портале Azure.
    
    d. В текстовое поле **Single Sign-Out URL** (URL-адрес единого выхода) вставьте значение **URL-адреса выхода**, скопированное на портале Azure.
    
    д) Нажмите кнопку **Сохранить** .

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

В этом разделе описано, как включить единый вход Azure для пользователя Britta Simon, предоставив этому пользователю доступ к Wizergos Productivity Software.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **Wizergos Productivity Software**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Wizergos Productivity Software**.

    ![Ссылка на Wizergos Productivity Software в списке "Приложения"](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-wizergos-productivity-software-test-user"></a>Создание тестового пользователя Wizergos Productivity Software

В этом разделе описано, как создать пользователя Britta Simon в приложении Wizergos Productivity Software. Обратитесь к [группе поддержки Wizergos Productivity Software](mailTo:support@wizergos.com), чтобы добавить пользователей на платформу Wizergos Productivity Software.

### <a name="test-single-sign-on"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку Wizergos Productivity Software на Панели доступа, вы автоматически войдете в приложение Wizergos Productivity Software, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)