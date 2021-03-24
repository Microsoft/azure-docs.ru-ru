---
title: Руководство по интеграции Azure Active Directory с Five9 Plus Adapter (CTI, Contact Center Agents) | Документы Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Five9 Plus Adapter (CTI, Contact Center Agents).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/04/2019
ms.author: jeedes
ms.openlocfilehash: 68de5b11c131fe33252178ebecdeb9c3855fe239
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92453439"
---
# <a name="tutorial-azure-active-directory-integration-with-five9-plus-adapter-cti-contact-center-agents"></a>Руководство по интеграции Azure Active Directory с Five9 Plus Adapter (CTI, Contact Center Agents)

В этом учебнике описано, как интегрировать Five9 Plus Adapter (CTI, Contact Center Agents) с Azure Active Directory (Azure AD).
Интеграция Five9 Plus Adapter (CTI, Contact Center Agents) с Azure AD предоставляет следующие преимущества:

* Вы можете управлять доступом к Five9 Plus Adapter (CTI, Contact Center Agents) с помощью Azure AD.
* Вы можете включить автоматический вход пользователей в Five9 Plus Adapter (CTI, Contact Center Agents) (единый вход) с использованием учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с Five9 Plus Adapter (CTI, Contact Center Agents), вам потребуется:

* Подписка Azure AD. (если у вас нет среды Azure AD, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/));
* подписка Five9 Plus Adapter (CTI, Contact Center Agents) с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Five9 Plus Adapter (CTI, Contact Center Agents) поддерживает единый вход, инициируемый **поставщиком удостоверений**.

## <a name="adding-five9-plus-adapter-cti-contact-center-agents-from-the-gallery"></a>Добавление Five9 Plus Adapter (CTI, Contact Center Agents) из коллекции

Чтобы настроить интеграцию Five9 Plus Adapter (CTI, Contact Center Agents) с Azure AD, необходимо добавить Five9 Plus Adapter (CTI, Contact Center Agents) из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Five9 Plus Adapter (CTI, Contact Center Agents) из коллекции, выполните следующие действия:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Five9 Plus Adapter (CTI, Contact Center Agents)**, выберите **Five9 Plus Adapter (CTI, Contact Center Agents)** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

     ![Five9 Plus Adapter (CTI, Contact Center Agents) в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure Active Directory в Five9 Plus Adapter (CTI, Contact Center Agents) с использованием тестового пользователя **Britta Simon**.
Чтобы единый вход функционировал, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Five9 Plus Adapter (CTI, Contact Center Agents).

Чтобы настроить и проверить единый вход Azure AD в Five9 Plus Adapter (CTI, Contact Center Agents), вам потребуется выполнить действия в следующих стандартных блоках:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в Five9 Plus Adapter (CTI, Contact Center Agents)](#configure-five9-plus-adapter-cti-contact-center-agents-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя Five9 Plus Adapter (CTI, Contact Center Agents)](#create-five9-plus-adapter-cti-contact-center-agents-test-user)** требуется для того, чтобы в Five9 Plus Adapter (CTI, Contact Center Agents) существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD для Five9 Plus Adapter (CTI, Contact Center Agents), выполните следующие действия.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Five9 Plus Adapter (CTI, Contact Center Agents)** щелкните **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. На странице **Настройка единого входа с помощью SAML** выполните следующие действия.

    ![Сведения о домене и URL-адресах единого входа для Five9 Plus Adapter (CTI, Contact Center Agents)](common/idp-intiated.png)

    а. В текстовом поле **Идентификатор** введите URL-адрес в таком формате:
    
    |    Среда      |       URL-адрес      |
    | :-- | :-- |
    | Для "Five9 Plus Adapter for Microsoft Dynamics CRM" | `https://app.five9.com/appsvcs/saml/metadata/alias/msdc` |
    | Для "Five9 Plus Adapter for Zendesk" | `https://app.five9.com/appsvcs/saml/metadata/alias/zd` |
    | Для "Five9 Plus Adapter for Agent Desktop Toolkit" | `https://app.five9.com/appsvcs/saml/metadata/alias/adt` |

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в таком формате:

    |      Среда     |      URL-адрес      |
    | :--                  | :--           |
    | Для "Five9 Plus Adapter for Microsoft Dynamics CRM" | `https://app.five9.com/appsvcs/saml/SSO/alias/msdc` |
    | Для "Five9 Plus Adapter for Zendesk" | `https://app.five9.com/appsvcs/saml/SSO/alias/zd` |
    | Для "Five9 Plus Adapter for Agent Desktop Toolkit" | `https://app.five9.com/appsvcs/saml/SSO/alias/adt` |

6. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (Base64)** из предложенных вариантов, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

7. Требуемые URL-адреса можно скопировать в разделе **Настройка Five9 Plus Adapter (CTI, Contact Center Agents)**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

### <a name="configure-five9-plus-adapter-cti-contact-center-agents-single-sign-on"></a>Настройка единого входа в Five9 Plus Adapter (CTI, Contact Center Agents)

1. Чтобы настроить единый вход на стороне **Five9 Plus Adapter (CTI, Contact Center Agents)**, нужно отправить скачанный **сертификат (Base64)** и соответствующие URL-адреса, которые в скопировали, [группе поддержки Five9 Plus Adapter (CTI, Contact Center Agents)](https://www.five9.com/about/contact). Для дальнейшей настройки единого входа выполните следующие действия в соответствии с адаптером:

    a. Руководство по администрированию "Five9 Plus Adapter for Agent Desktop Toolkit": [https://webapps.five9.com/assets/files/for_customers/documentation/integrations/agent-desktop-toolkit/plus-agent-desktop-toolkit-administrators-guide.pdf](https://webapps.five9.com/assets/files/for_customers/documentation/integrations/agent-desktop-toolkit/plus-agent-desktop-toolkit-administrators-guide.pdf)
    
    b. Руководство по администрированию "Five9 Plus Adapter for Microsoft Dynamics CRM": [https://webapps.five9.com/assets/files/for_customers/documentation/integrations/microsoft/microsoft-administrators-guide.pdf](https://webapps.five9.com/assets/files/for_customers/documentation/integrations/microsoft/microsoft-administrators-guide.pdf)
    
    c. Руководство по администрированию "Five9 Plus Adapter for Zendesk": [https://webapps.five9.com/assets/files/for_customers/documentation/integrations/zendesk/zendesk-plus-administrators-guide.pdf](https://webapps.five9.com/assets/files/for_customers/documentation/integrations/zendesk/zendesk-plus-administrators-guide.pdf)

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD 

Цель этого раздела — создать на портале Azure тестового пользователя с именем Britta Simon.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.

    ![Ссылки "Пользователи и группы" и "Все пользователи"](common/users.png)

2. В верхней части экрана выберите **Новый пользователь**.

    ![Кнопка "Новый пользователь"](common/new-user.png)

3. В разделе свойств пользователя сделайте следующее:

    ![Диалоговое окно "Пользователь"](common/user-properties.png)

    а. В поле **Имя** введите **BrittaSimon**.
  
    b. В поле **Имя пользователя** введите `brittasimon@yourcompanydomain.extension`. Например BrittaSimon@contoso.com.

    c. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле "Пароль".

    d. Нажмите кнопку **Создать**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к Five9 Plus Adapter (CTI, Contact Center Agents).

1. На портале Azure выберите **Корпоративные приложения**, выберите **Все приложения**, а затем — **Five9 Plus Adapter (CTI, Contact Center Agents)**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Five9 Plus Adapter (CTI, Contact Center Agents)**.

    ![Ссылка на Five9 Plus Adapter (CTI, Contact Center Agents) в списке "Приложения"](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-five9-plus-adapter-cti-contact-center-agents-test-user"></a>Создание тестового пользователя Five9 Plus Adapter (CTI, Contact Center Agents)

В этом разделе вы создадите тестового пользователя по имени Britta Simon в Five9 Plus Adapter (CTI, Contact Center Agents). Чтобы добавить пользователей в Five9 Plus Adapter (CTI, Contact Center Agents), обратитесь в [службу поддержки Five9 Plus Adapter (CTI, Contact Center Agents)](https://www.five9.com/about/contact). Перед использованием единого входа необходимо создать и активировать пользователей. 

### <a name="test-single-sign-on"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув элемент "Five9 Plus Adapter (CTI, Contact Center Agents)" на Панели доступа, вы автоматически войдете в приложение Five9 Plus Adapter (CTI, Contact Center Agents), для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)