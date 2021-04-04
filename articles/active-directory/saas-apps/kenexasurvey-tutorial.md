---
title: Учебник. Интеграция Azure Active Directory с IBM Kenexa Survey Enterprise | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в IBM Kenexa Survey Enterprise.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/08/2019
ms.author: jeedes
ms.openlocfilehash: faea5fdc7fdab11c630f24dbb8527e5bf198765c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92459158"
---
# <a name="tutorial-azure-active-directory-integration-with-ibm-kenexa-survey-enterprise"></a>Руководство по интеграции Azure Active Directory с IBM Kenexa Survey Enterprise

В этом руководстве описано, как интегрировать IBM Kenexa Survey Enterprise с Azure Active Directory (Azure AD).
Интеграция IBM Kenexa Survey Enterprise с Azure AD обеспечивает следующие преимущества:

* С помощью Azure AD можно контролировать, кто будет иметь доступ к IBM Kenexa Survey Enterprise.
* Вы можете включить автоматический вход пользователей в IBM Kenexa Survey Enterprise (единый вход) под учетной записью Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с IBM Kenexa Survey Enterprise, вам потребуется следующее:

* Подписка Azure AD. (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка IBM Kenexa Survey Enterprise с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* IBM Kenexa Survey Enterprise поддерживает единый вход, инициируемый **поставщиком удостоверений**.

## <a name="adding-ibm-kenexa-survey-enterprise-from-the-gallery"></a>Добавление IBM Kenexa Survey Enterprise из коллекции.

Чтобы настроить интеграцию IBM Kenexa Survey Enterprise в Azure AD, необходимо добавить IBM Kenexa Survey Enterprise из коллекции в список управляемых приложений SaaS.

**Чтобы добавить IBM Kenexa Survey Enterprise из коллекции, выполните следующие действия.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **IBM Kenexa Survey Enterprise**, выберите **IBM Kenexa Survey Enterprise** и нажмите кнопку **Добавить**, чтобы добавить это приложение.

     ![IBM Kenexa Survey Enterprise в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в IBM Kenexa Survey Enterprise с использованием тестового пользователя **Britta Simon**.
Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в IBM Kenexa Survey Enterprise.

Чтобы настроить и проверить единый вход Azure AD в IBM Kenexa Survey Enterprise, вам потребуется выполнить действия в следующих стандартных блоках:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в IBM Kenexa Survey Enterprise](#configure-ibm-kenexa-survey-enterprise-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя IBM Kenexa Survey Enterprise](#create-ibm-kenexa-survey-enterprise-test-user)** требуется для создания пользователя Britta Simon в IBM Kenexa Survey Enterprise, связанного с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в IBM Kenexa Survey Enterprise, сделайте следующее.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **IBM Kenexa Survey Enterprise** щелкните **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. На странице **Настройка единого входа с помощью SAML** выполните следующие действия.

    ![Сведения о домене и URL-адресах IBM Kenexa Survey Enterprise](common/idp-intiated.png)

    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `https://surveys.kenexa.com/<companycode>`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://surveys.kenexa.com/<companycode>/tools/sso.asp`.

    > [!NOTE]
    > Эти значения приведены для примера. Измените их на фактические значения идентификатора и URL-адреса ответа. Чтобы получить эти значения, обратитесь в [группу поддержки клиентов IBM Kenexa Survey Enterprise](https://www.ibm.com/support/home/?lnk=fcw). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. Приложение IBM Kenexa Survey Enterprise ожидает утверждения SAML (язык разметки заявлений системы безопасности) в определенном формате, поэтому необходимо добавить настраиваемые сопоставления атрибутов в конфигурацию атрибутов токена SAML. Полученное в ответ значение утверждения идентификатора пользователя должно совпадать с идентификатором единого входа, настроенным в системе Kenexa. Чтобы сопоставить соответствующий идентификатор пользователя в организации для применения единого входа на основе протокола IDP, обратитесь к [группе поддержки IBM Kenexa Survey Enterprise](https://www.ibm.com/support/home/?lnk=fcw).

    По умолчанию Azure AD в качестве идентификатора пользователя задает значение имени участника-пользователя (UPN). Это значение можно изменить на вкладке **Атрибуты пользователя**, как показано на следующем снимке экрана. Интеграция заработает только после правильной настройки сопоставления.

    ![Изображение](common/edit-attribute.png)

6. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (Base64)** из предложенных вариантов, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

7. Требуемые URL-адреса вы можете скопировать из раздела **Настройка IBM Kenexa Survey Enterprise**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

### <a name="configure-ibm-kenexa-survey-enterprise-single-sign-on"></a>Настройка единого входа для IBM Kenexa Survey Enterprise

Чтобы настроить единый вход на стороне **IBM Kenexa Survey Enterprise**, нужно отправить скачанный **сертификат (Base64)** и соответствующие URL-адреса, скопированные на портале Azure, [группе поддержки IBM Kenexa Survey Enterprise](https://www.ibm.com/support/home/?lnk=fcw). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

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

В этом разделе описано, как позволить пользователю Britta Simon использовать единый вход Azure, предоставив ему доступ к IBM Kenexa Survey Enterprise.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **IBM Kenexa Survey Enterprise**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **IBM Kenexa Survey Enterprise**.

    ![Ссылка на IBM Kenexa Survey Enterprise в списке "Приложения"](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-ibm-kenexa-survey-enterprise-test-user"></a>Создание тестового пользователя IBM Kenexa Survey Enterprise

В этом разделе описано, как создать пользователя Britta Simon в приложении IBM Kenexa Survey Enterprise.

Чтобы создать пользователей в системе IBM Kenexa Survey Enterprise и сопоставить с ними идентификатор единого входа, вы можете обратиться к [группе поддержки IBM Kenexa Survey Enterprise](https://www.ibm.com/support/home/?lnk=fcw). Это значение идентификатора единого входа необходимо также сопоставить со значением идентификатора пользователя из Azure AD. Эти заданные по умолчанию параметры можно изменить на вкладке **Атрибут**.

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув элемент IBM Kenexa Survey Enterprise на панели доступа, вы автоматически войдете в приложение IBM Kenexa Survey Enterprise, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)