---
title: Руководство по Интеграция Azure Active Directory со Sectigo Certificate Manager | Документация Майкрософт
description: Из этой статьи вы узнаете, как настроить единый вход Azure Active Directory в Sectigo Certificate Manager.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/15/2019
ms.author: jeedes
ms.openlocfilehash: c589db84b6221aa23868b3b49aea84f33623619f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92673857"
---
# <a name="tutorial-azure-active-directory-integration-with-sectigo-certificate-manager"></a>Руководство по Интеграция Azure Active Directory со Sectigo Certificate Manager

В этом руководстве описано, как интегрировать Sectigo Certificate Manager (сокращенно SCM) с Azure Active Directory (Azure AD).

Интеграция Azure AD с Sectigo Certificate Manager обеспечивает следующие преимущества:

* С помощью Azure AD вы можете контролировать доступ к Sectigo Certificate Manager.
* Пользователи могут автоматически входить в Sectigo Certificate Manager с учетными записями Azure AD (единый вход).
* Вы можете управлять учетными записями централизованно, через портал Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье о [едином входе в приложения в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с приложением Sectigo Certificate Manager, вам потребуется следующее:

* Подписка Azure AD. Если у вас еще нет подписки Azure AD, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.
* Учетная запись Sectigo Certificate Manager.

> [!NOTE]
> Sectigo запускает несколько экземпляров Sectigo Certificate Manager. Основным экземпляром для Sectigo Certificate Manager является **https:\//cert-manager.com** и этот URL-адрес используется в руководстве.  Если ваша учетная запись размещена в другом экземпляре, настройте URL-адрес соответствующим образом.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде и интеграцию Sectigo Certificate Manager с Azure AD.

Sectigo Certificate Manager поддерживает следующие функции:

* **единый вход, инициированный поставщиком службы**;
* **единый вход, инициированный поставщиком удостоверений**;

## <a name="add-sectigo-certificate-manager-in-the-azure-portal"></a>Добавление Sectigo Certificate Manager на портале Azure

Чтобы интегрировать Sectigo Certificate Manager с Azure AD, нужно добавить Sectigo Certificate Manager в список управляемых приложений SaaS.

1. Войдите на [портал Azure](https://portal.azure.com).

1. В меню слева выберите **Azure Active Directory**.

    ![Пункт Azure Active Directory](common/select-azuread.png)

1. Щелкните **Корпоративные приложения** > **Все приложения**.

    ![Область "Корпоративные приложения"](common/enterprise-applications.png)

1. Чтобы добавить приложение, выберите **Новое приложение**.

    ![Элемент "Новое приложение"](common/add-new-app.png)

1. В поле поиска введите **Sectigo Certificate Manager**. В результатах поиска выберите **Sectigo Certificate Manager** и щелкните **Добавить**.

    ![Sectigo Certificate Manager в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в Sectigo Certificate Manager для пользователя **Britta Simon**. Чтобы обеспечить работу единого входа, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Sectigo Certificate Manager.

Чтобы настроить и проверить единый вход Azure AD в Sectigo Certificate Manager, потребуется выполнить действия в следующих стандартных блоках:

| Задача | Описание |
| --- | --- |
| **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** | Предоставление пользователям возможности использовать эту функцию. |
| **[Настройка единого входа для Sectigo Certificate Manager](#configure-sectigo-certificate-manager-single-sign-on)** | Настройка параметров единого входа в приложении. |
| **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** | Проверка работы единого входа Azure AD на примере пользователя Britta Simon. |
| **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** | Предоставление пользователю Britta Simon возможности использовать единый вход Azure AD. |
| **[Создание тестового пользователя Sectigo Certificate Manager](#create-a-sectigo-certificate-manager-test-user)** | Создание пользователя Britta Simon в Sectigo Certificate Manager, связанного с соответствующим пользователем в Azure AD. |
| **[Проверка единого входа](#test-single-sign-on)** | необходима, чтобы убедиться, что конфигурация работает правильно. |

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описывается настройка единого входа Azure AD в Sectigo Certificate Manager на портале Azure.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Sectigo Certificate Manager** выберите **Единый вход**.

    ![Параметр для настройки единого входа](common/select-sso.png)

1. На панели **Выбрать метод единого входа** выберите режим **SAML** или **SAML/WS-FED**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

1. На панели **Настройка единого входа с помощью SAML** щелкните **Изменить** (значок карандаша), чтобы открыть панель **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. В области **Базовая конфигурация SAML** выполните описанные ниже действия.

    1. В поле **Идентификатор (ИД сущности)** введите **https:\//cert-manager.com/shibboleth** для основного экземпляра Sectigo Certificate Manager.

    1. В поле **URL-адрес ответа** введите **https:\//cert-manager.com/Shibboleth.sso/SAML2/POST** для основного экземпляра Sectigo Certificate Manager.
        
    > [!NOTE]
    > Хотя поле **URL-адрес входа** является обязательным для *режима, инициированного поставщиком услуг*, оно не требуется для входа из Sectigo Certificate Manager.        

1. Дополнительно, для настройки *режима, инициированного поставщиком удостоверений*, и разрешения выполнения **проверки** в разделе **Базовая конфигурация SAML** выполните следующие шаги:

    1. Щелкните **Задать дополнительные URL-адреса**.

    1. Введите в поле **Состояние ретранслятора** клиентский URL-адрес вашего Sectigo Certificate Manager. Введите для основой подписки Sectigo Certificate Manager **https:\//cert-manager.com/customer/\<customerURI\>/idp**.

    ![Сведения о домене и URL-адресах единого входа для приложения Sectigo Certificate Manager](common/idp-relay.png)

1. В разделе **User Attributes & Claims** (Пользовательские атрибуты и утверждения) выполните следующие шаги.

    1. Удалите все **дополнительные утверждения**.
    
    1. Выберите **Добавить новое утверждение** и добавьте следующие четыре утверждения:
    
        | Имя | Пространство имен | Источник | Атрибут источника | Описание |
        | --- | --- | --- | --- | --- |
        | eduPersonPrincipalName | пустых | attribute | user.userprincipalname | Для администраторов поле **IdP Person ID** (Идентификатор поставщика удостоверений) должно совпадать в Sectigo Certificate Manager. |
        | mail | пустых | attribute | user.mail | Обязательно |
        | givenName | пустых | attribute | user.givenname | Необязательно |
        | sn | пустых | attribute | user.surname | Необязательно |

       ![Sectigo Certificate Manager — добавление четырех новых утверждений](media/sectigo-certificate-manager-tutorial/additional-claims.png)

1. В разделе **Сертификат подписи SAML** выберите **Загрузить** рядом с **XML метаданных федерации**. Сохраните XML-файл на вашем компьютере.

    ![Элемент для скачивания XML-файла метаданных федерации](common/metadataxml.png)

### <a name="configure-sectigo-certificate-manager-single-sign-on"></a>Настройка единого входа для Sectigo Certificate Manager

Чтобы настроить единый вход на стороне Sectigo Certificate Manager, отправьте сохраненный файл XML метаданных федерации в службу поддержки [Sectigo Certificate Manager](https://sectigo.com/support). Специалисты службы поддержки Sectigo Certificate Manager на основе полученной информации выполнят все необходимое для правильной настройки единого входа SAML с обеих сторон.

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD 

В этом разделе описано, как создать на портале Azure тестового пользователя с именем Britta Simon.

1. На портале Azure выберите **Azure Active Directory** > **Пользователи** > **Все пользователи**.

    ![Пункты меню "Пользователи" и "Все пользователи"](common/users.png)

1. Выберите **Новый пользователь**.

    ![Элемент "Новый пользователь"](common/new-user.png)

1. В области **Пользователь** выполните приведенные ниже действия.

    1. В поле **Имя** введите **BrittaSimon**.
  
    1. В поле **Имя пользователя** введите **brittasimon\@\<your-company-domain>.\<extension\>** . Например, **brittasimon\@contoso.com**.

    1. Установите флажок **Показать пароль**. Запишите значение, отображаемое в поле **Пароль**.

    1. Нажмите кнопку **Создать**.

    ![Панель "Пользователь"](common/user-properties.png)

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе вы предоставите для Britta Simon доступ к Sectigo Certificate Manager, чтобы пользователь мог использовать единый вход Azure.

1. На портале Azure выберите **Корпоративные приложения** > **Все приложения** > **Sectigo Certificate Manager**.

    ![Область "Корпоративные приложения"](common/enterprise-applications.png)

1. В списке приложений выберите **Sectigo Certificate Manager**.

    ![Sectigo Certificate Manager в списке приложений](common/all-applications.png)

1. В меню выберите **Пользователи и группы**.

    ![Пункт меню "Пользователи и группы"](common/users-groups-blade.png)

1. Выберите **Добавить пользователя**. Затем в области **Добавление назначения** щелкните **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

1. На панели **Пользователи и группы** в списке пользователей выберите **Britta Simon**. Щелкните **Выбрать**.

1. Если вы планируете получать значение роли в утверждении SAML, выберите на панели **Выбор роли** подходящую роль для пользователя из предложенного списка. Щелкните **Выбрать**.

1. В области **Добавление назначения** выберите **Назначить**.

### <a name="create-a-sectigo-certificate-manager-test-user"></a>Создание тестового пользователя Sectigo Certificate Manager

В этом разделе описано, как создать пользователя Britta Simon на платформе Sectigo Certificate Manager. Чтобы добавить пользователя на платформу Sectigo Certificate Manager, обратитесь в [службу поддержки Sectigo Certificate Manager](https://sectigo.com/support). Перед использованием единого входа необходимо создать и активировать пользователей.

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD.

#### <a name="test-from-sectigo-certificate-manager-sp-initiated-single-sign-on"></a>Проверка из Sectigo Certificate Manager (единый вход, инициированный поставщиком услуг)

Перейдите к клиентскому URL-адресу (для основного экземпляра Sectigo Certificate Manager) https:\//cert-manager.com/customer/\<customerURI\> и нажмите кнопку под пунктом **Или войти с помощью**.  При правильной настройке вы автоматически войдете в Sectigo Certificate Manager.

#### <a name="test-from-azure-single-sign-on-configuration-idp-initiated-single-sign-on"></a>Проверка из конфигурации единого входа Azure (единый вход, инициированный поставщиком удостоверений)

На панели интеграции приложения **Sectigo Certificate Manager** выберите **Единый вход** и нажмите кнопку **Проверка**.  При правильной настройке вы автоматически войдете в Sectigo Certificate Manager.

#### <a name="test-by-using-the-my-apps-portal-idp-initiated-single-sign-on"></a>Проверка с помощью портала "Мои приложения" (единый вход, инициированный поставщиком удостоверений)

Выберите **Sectigo Certificate Manager** на портале "Мои приложения".  При правильной настройке вы автоматически войдете в Sectigo Certificate Manager. Дополнительные сведения о портале "Мои приложения" см. в статье [Access and use apps on the My Apps portal](../user-help/my-apps-portal-end-user-access.md) (Доступ к приложениям и их использование на портале "Мои приложения").

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в статьях ниже:

- [Список руководств по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)
- [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md)
- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)