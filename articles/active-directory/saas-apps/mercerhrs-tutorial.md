---
title: Руководство по интеграции Azure Active Directory с Mercer BenefitsCentral (MBC) | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Mercer BenefitsCentral (MBC).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/20/2019
ms.author: jeedes
ms.openlocfilehash: fc60b838219e73b008f82271353ca75d0d24d2e3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92517197"
---
# <a name="tutorial-azure-active-directory-integration-with-mercer-benefitscentral-mbc"></a>Руководство по интеграции Azure Active Directory с Mercer BenefitsCentral (MBC)

В этом руководстве описано, как интегрировать Mercer BenefitsCentral (MBC) с Azure Active Directory (Azure AD).
Интеграция Azure AD с Mercer BenefitsCentral (MBC) обеспечивает следующие преимущества:

* С помощью Azure AD вы можете контролировать доступ к Mercer BenefitsCentral (MBC).
* Вы можете включить автоматический вход пользователей в Mercer BenefitsCentral (MBC) (единый вход) с использованием их учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD с Mercer BenefitsCentral (MBC), вам потребуется:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка Mercer BenefitsCentral (MBC) с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Mercer BenefitsCentral (MBC) поддерживает инициированный единый вход **выдающей точки распространения**.

## <a name="adding-mercer-benefitscentral-mbc-from-the-gallery"></a>Добавление Mercer BenefitsCentral (MBC) из коллекции

Чтобы настроить интеграцию Mercer BenefitsCentral (MBC) с Azure AD, необходимо добавить Mercer BenefitsCentral (MBC) из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Mercer BenefitsCentral (MBC) из коллекции, выполните следующие действия.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Mercer BenefitsCentral (MBC)** , выберите **Mercer BenefitsCentral (MBC)** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

     ![Mercer BenefitsCentral (MBC) в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в приложение Mercer BenefitsCentral (MBC) с использованием тестового пользователя **Britta Simon**.
Для работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Mercer BenefitsCentral (MBC).

Чтобы настроить и проверить единый вход Azure AD в Mercer BenefitsCentral (MBC), требуется выполнить действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Mercer BenefitsCentral (MBC)](#configure-mercer-benefitscentral-mbc-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя приложения Mercer BenefitsCentral (MBC)](#create-mercer-benefitscentral-mbc-test-user)** требуется для того, чтобы в Mercer BenefitsCentral (MBC) существовал пользователь Britta Simon, связанный с представлением этого же пользователя в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Mercer BenefitsCentral (MBC), выполните следующие действия:

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Mercer BenefitsCentral (MBC)** выберите **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. На странице **Настройка единого входа с помощью SAML** выполните следующие действия.

    ![Сведения о домене и URL-адресах единого входа приложения Mercer BenefitsCentral (MBC)](common/idp-intiated.png)

    а. В текстовом поле **Идентификатор** введите URL-адрес: `stg.mercerhrs.com/saml2.0`

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://ssous-stg.mercerhrs.com/SP2/Saml2AssertionConsumer.aspx`.

    > [!NOTE]
    > Значение URL-адреса ответа приведено для примера. Вместо него нужно указать фактический URL-адрес ответа. Обратитесь в [группу поддержки клиентов Mercer BenefitsCentral (MBC)](https://www.mercer.com/contact-us.html), чтобы получить это значение. Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

6. Требуемый URL-адрес можно скопировать из раздела **Настройка Mercer BenefitsCentral (MBC)** .

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD.

    c. URL-адрес выхода.

### <a name="configure-mercer-benefitscentral-mbc-single-sign-on"></a>Настройка единого входа Mercer BenefitsCentral (MBC)

Чтобы настроить единый вход на стороне **Mercer BenefitsCentral (MBC)** , нужно отправить скачанный файл **XML метаданных федерации** и соответствующие URL-адреса, скопированные на портале Azure, [группе поддержки Mercer BenefitsCentral (MBC)](https://www.mercer.com/contact-us.html). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD 

Цель этого раздела — создать на портале Azure тестового пользователя с именем Britta Simon.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.

    ![Ссылки "Пользователи и группы" и "Все пользователи"](common/users.png)

2. В верхней части экрана выберите **Новый пользователь**.

    ![Кнопка "Новый пользователь"](common/new-user.png)

3. В разделе свойств пользователя сделайте следующее:

    ![Диалоговое окно "Пользователь"](common/user-properties.png)

    а. В поле **Имя** введите **BrittaSimon**.
  
    b. В поле **Имя пользователя** введите **brittasimon\@домен_вашей_компании.доменная_зона**.  
    Например BrittaSimon@contoso.com.

    c. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле "Пароль".

    d. Нажмите кнопку **Создать**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к Mercer BenefitsCentral (MBC).

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **Mercer BenefitsCentral (MBC)** .

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка "Приложения" выберите **Mercer BenefitsCentral (MBC)** .

    ![Ссылка на Mercer BenefitsCentral (MBC) в списке "Приложения"](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-mercer-benefitscentral-mbc-test-user"></a>Создание тестового пользователя Mercer BenefitsCentral (MBC)

В этом разделе описано, как создать пользователя Britta Simon в Mercer BenefitsCentral (MBC). Чтобы добавить пользователей на платформу Mercer BenefitsCentral (MBC), обратитесь в [службу технической поддержки Mercer BenefitsCentral (MBC)](https://www.mercer.com/contact-us.html). Перед использованием единого входа необходимо создать и активировать пользователей.

### <a name="test-single-sign-on"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку "Mercer BenefitsCentral (MBC)" на панели доступа, вы автоматически войдете в приложение Mercer BenefitsCentral (MBC), для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)