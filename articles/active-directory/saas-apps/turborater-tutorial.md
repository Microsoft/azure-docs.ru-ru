---
title: Руководство по интеграции Azure Active Directory с TurboRater | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в приложении TurboRater.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 3/8/2019
ms.author: jeedes
ms.openlocfilehash: 7483e7a9300088d07e85d238ad897522d59c5464
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92519390"
---
# <a name="tutorial-azure-active-directory-integration-with-turborater"></a>Руководство по интеграции Azure Active Directory с TurboRater

В этом руководстве описано, как интегрировать TurboRater с Azure Active Directory (Azure AD).

Интеграция Azure AD с приложением TurboRater обеспечивает следующие преимущества.

* С помощью Azure AD вы можете контролировать доступ к TurboRater.
* Вы можете включить автоматический вход пользователей в TurboRater (единый вход) с помощью учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно — через портал Azure.

Дополнительные сведения об интеграции приложения SaaS с Azure AD см. в статье [о доступе к приложениям и едином входе с помощью Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD с TurboRater, вам потребуется следующее:

* подписка Azure AD; Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.
* подписка TurboRater с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

TurboRater поддерживает единый вход, инициируемый поставщиком удостоверений.

## <a name="add-turborater-from-the-azure-marketplace"></a>Добавление TurboRater из Azure Marketplace

Чтобы настроить интеграцию TurboRater с Azure AD, необходимо добавить TurboRater из Azure Marketplace в список управляемых приложений SaaS.

1. Войдите на [портал Azure](https://portal.azure.com?azure-portal=true).
1. В области слева выберите **Azure Active Directory**.

    ![Пункт Azure Active Directory](common/select-azuread.png)

1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Параметр "Корпоративные приложения"](common/enterprise-applications.png)

1. Чтобы добавить новое приложение, выберите **+ Новое приложение** в верхней части панели.

    ![Элемент "Новое приложение"](common/add-new-app.png)

1. В поле поиска введите **TurboRater**. В результатах поиска выберите **TurboRater** и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![TurboRater в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в TurboRater с использованием тестового пользователя **B. Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в TurboRater.

Чтобы настроить и проверить единый вход Azure AD в TurboRater, вам потребуется выполнить действия в следующих стандартных блоках.

1. **[Настройте единый вход в Azure AD](#configure-azure-ad-single-sign-on)** , чтобы пользователи могли использовать эту функцию.
1. **[Настройка единого входа в TurboRater](#configure-turborater-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя B. Simon.
1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить B. Simon использовать единый вход Azure AD.
1. **[Создание тестового пользователя TurboRater](#create-a-turborater-test-user)** требуется для того, чтобы в TurboRater существовал пользователь B. Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверьте единый вход](#test-single-sign-on)** , чтобы убедиться в корректной работе конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в TurboRater, сделайте следующее.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **TurboRater** выберите **Единый вход**.

    ![Параметр для настройки единого входа](common/select-sso.png)

1. В области **Выбрать метод единого входа** выберите режим **SAML/WS-FED**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

1. В области **Настройка единого входа с помощью SAML** щелкните **Изменить** (значок карандаша), чтобы открыть панель **Базовая конфигурация SAML**.

    ![Правка базовой конфигурации SAML](common/edit-urls.png)

1. В области **Базовая конфигурация SAML** выполните приведенные ниже действия.

    ![Сведения о домене и URL-адресах единого входа для TurboRater](common/idp-intiated.png)

    1. В поле **Идентификатор (сущности)** введите URL-адрес:

       `https://www.itcdataservices.com`

    1. В поле **Reply URL (Assertion Consumer Service URL)** (URL-адреса ответа (URL-адрес службы обработчика утверждений)) введите URL-адрес в следующем формате:

       | Среда | URL-адрес |
       | ---------------| --------------- |
       | Тест  | `https://ratingqa.itcdataservices.com/webservices/imp/saml/login` |
       | Реальные данные  | `https://www.itcratingservices.com/webservices/imp/saml/login` |

    > [!NOTE]
    > Эти значения приведены в качестве примера. Замените их фактическими значениями идентификатора и URL-адреса ответа. Чтобы получить эти значения, обратитесь к [группе поддержки TurboRater](https://www.getitc.com/support). Можно также посмотреть шаблоны на панели **Базовая конфигурация SAML** на портале Azure.

1. На панели **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать подходящий **XML-файл метаданных федерации** из предложенных вариантов и сохранить его на своем компьютере.

    ![Элемент для скачивания XML-файла метаданных федерации](common/metadataxml.png)

1. Требуемые URL-адреса можно скопировать в разделе **Настройка TurboRater**.

   * **Login URL** (URL-адрес для входа)
   * **идентификатор Azure AD**;
   * **URL-адрес выхода**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

### <a name="configure-turborater-single-sign-on"></a>Настройка единого входа в TurboRater

Чтобы настроить единый вход на стороне TurboRater, отправьте скачанный XML-файл метаданных федерации и URL-адреса, скопированные на портале Azure, [группе поддержки TurboRater](https://www.getitc.com/support). Группа поддержки TurboRater обеспечит правильную настройку подключения для единого входа SAML с обеих сторон.

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD

В этом разделе описано, как создать на портале Azure тестового пользователя с именем Britta Simon.

1. На портале Azure на панели слева выберите **Azure Active Directory**   > **Пользователи** > **Все пользователи**.

    ![Пункты меню "Пользователи" и "Все пользователи"](common/users.png)

1. В верхней части экрана щелкните **+ Новый пользователь**.

    ![Элемент "Новый пользователь"](common/new-user.png)

1. На панели **Пользователь** выполните следующие действия:

    ![Панель "Пользователь"](common/user-properties.png)

    1. В поле **Имя** введите **BSimon**.
  
    1. В поле **Имя пользователя** введите **BSimon\@\<yourcompanydomain>.\<extension>** . Например, **BSimon\@contoso.com**.

    1. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле **Пароль**.

    1. Нажмите кнопку **создания**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как разрешить пользователю B. Simon использовать единый вход Azure, предоставив этому пользователю доступ к TurboRater.

1. На портале Azure выберите **Корпоративные приложения** > **Все приложения** > **TurboRater**.

    ![Область "Корпоративные приложения"](common/enterprise-applications.png)

1. Из списка приложений выберите **TurboRater**.

    ![TurboRater в списке приложений](common/all-applications.png)

1. В области слева последовательно выберите **Управление** и **Пользователи и группы**.

    ![Пункт меню "Пользователи и группы"](common/users-groups-blade.png)

1. Выберите **+ Добавить пользователя**, а в области **Добавление назначения** щелкните **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

1. В области **Пользователи и группы** выберите **B. Simon** из списка **Пользователи**, а в нижней области щелкните **Выбрать**.

1. Если вы планируете получать значение роли в утверждении SAML, на панели **Выбор роли** выберите в списке соответствующую роль для пользователя. В нижней части панели нажмите кнопку **Выбрать**.

1. В области **Добавление назначения** выберите **Назначить**.

### <a name="create-a-turborater-test-user"></a>Создание тестового пользователя TurboRater

В этом разделе описано, как создать пользователя B. Simon в приложении TurboRater. Обратитесь к [группе поддержки TurboRater](https://www.getitc.com/support), чтобы добавить пользователя B. Simon в TurboRater. Перед использованием единого входа необходимо создать и активировать пользователей.

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе объясняется, как проверить конфигурацию единого входа Azure AD с помощью портала "Мои приложения".

Щелкнув **TurboRater** на портале "Мои приложения", вы автоматически выполните вход с подпиской TurboRater, для которой настроили единый вход. Дополнительные сведения о портале "Мои приложения" см. в статье [Access and use apps on the My Apps portal](../user-help/my-apps-portal-end-user-access.md) (Доступ к приложениям и их использование на портале "Мои приложения").

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Список руководств по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

* [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)