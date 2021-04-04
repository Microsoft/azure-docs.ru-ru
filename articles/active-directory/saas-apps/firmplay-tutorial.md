---
title: Руководство. Интеграция Azure Active Directory с FirmPlay — Employee Advocacy for Recruiting | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в FirmPlay — Employee Advocacy for Recruiting.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/01/2019
ms.author: jeedes
ms.openlocfilehash: 2230958fb41d8e42967beeca57cf10ea048d1ef9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92453473"
---
# <a name="tutorial-azure-active-directory-integration-with-firmplay---employee-advocacy-for-recruiting"></a>Руководство. Интеграция Azure Active Directory с FirmPlay — Employee Advocacy for Recruiting

В этом руководстве описано, как интегрировать Azure Active Directory (Azure AD) с приложением FirmPlay — Employee Advocacy for Recruiting.
Интеграция Azure AD с приложением FirmPlay обеспечивает следующие преимущества.

* Вы можете управлять доступом к FirmPlay — Employee Advocacy for Recruiting с помощью Azure AD.
* Вы можете включить автоматический вход пользователей в FirmPlay — Employee Advocacy for Recruiting (единый вход) с использованием учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD с FirmPlay — Employee Advocacy for Recruiting, вам потребуются следующие компоненты:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/));
* подписка FirmPlay — Employee Advocacy for Recruiting с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* FirmPlay - Employee Advocacy for Recruiting поддерживает единый вход, инициируемый **поставщиком услуг**.

## <a name="adding-firmplay---employee-advocacy-for-recruiting-from-the-gallery"></a>Добавление FirmPlay — Employee Advocacy for Recruiting из коллекции

Чтобы настроить интеграцию FirmPlay — Employee Advocacy for Recruiting с Azure AD, необходимо добавить приложение FirmPlay из коллекции в список управляемых приложений SaaS.

**Чтобы добавить FirmPlay — Employee Advocacy for Recruiting из коллекции, выполните следующие действия.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **FirmPlay — Employee Advocacy for Recruiting**, выберите **FirmPlay — Employee Advocacy for Recruiting** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

     ![FirmPlay - Employee Advocacy for Recruiting в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в FirmPlay — Employee Advocacy for Recruiting с использованием тестового пользователя **Britta Simon**.
Чтобы единый вход функционировал, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в FirmPlay - Employee Advocacy for Recruiting.

Чтобы настроить и проверить единый вход Azure AD в FirmPlay, выполните следующие действия.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в FirmPlay - Employee Advocacy for Recruiting](#configure-firmplay---employee-advocacy-for-recruiting-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя FirmPlay — Employee Advocacy for Recruiting](#create-firmplay---employee-advocacy-for-recruiting-test-user)** требуется для того, чтобы в FirmPlay - Employee Advocacy for Recruiting существовал пользователь Britta Simon, который связан с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в FirmPlay — Employee Advocacy for Recruiting, выполните следующие действия.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **FirmPlay — Employee Advocacy for Recruiting** щелкните **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    ![Сведения о домене и URL-адресах единого входа для FirmPlay - Employee Advocacy for Recruiting](common/sp-signonurl.png)

    В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://<your-subdomain>.firmplay.com/`.

    > [!NOTE]
    > Это значение приведено для примера. Вместо него необходимо указать фактический URL-адрес входа. Чтобы получить это значение, обратитесь к [группе поддержки клиентов FirmPlay — Employee Advocacy for Recruiting](mailto:engineering@firmplay.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (Base64)** из предложенных вариантов, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

6. Требуемые URL-адреса можно скопировать из раздела **Настройка FirmPlay - Employee Advocacy for Recruiting**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

### <a name="configure-firmplay---employee-advocacy-for-recruiting-single-sign-on"></a>Настройка единого входа в FirmPlay - Employee Advocacy for Recruiting

Чтобы настроить единый вход на стороне **FirmPlay — Employee Advocacy for Recruiting**, нужно отправить скачанный **сертификат (Base64)** и соответствующие URL-адреса, скопированные на портале Azure, [группе поддержки FirmPlay - Employee Advocacy for Recruiting](mailto:engineering@firmplay.com). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

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

В этом разделе объясняется, как позволить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к FirmPlay — Employee Advocacy for Recruiting.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **FirmPlay - Employee Advocacy for Recruiting**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **FirmPlay — Employee Advocacy for Recruiting**.

    ![Ссылка на FirmPlay - Employee Advocacy for Recruiting в списке "Приложения"](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-firmplay---employee-advocacy-for-recruiting-test-user"></a>Создание тестового пользователя FirmPlay — Employee Advocacy for Recruiting

В этом разделе вы создадите тестового пользователя FirmPlay — Employee Advocacy for Recruiting с именем Britta Simon. Чтобы добавить пользователей на платформу FirmPlay — Employee Advocacy for Recruiting, обратитесь в [службу поддержки этой платформы](mailto:engineering@firmplay.com). Перед использованием единого входа необходимо создать и активировать пользователей.

### <a name="test-single-sign-on"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув элемент "FirmPlay - Employee Advocacy for Recruiting" на Панели доступа, вы автоматически войдете в приложение FirmPlay — Employee Advocacy for Recruiting, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)