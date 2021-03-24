---
title: Руководство по интеграции Azure Active Directory с решением SilkRoad Life Suite | Документация Майкрософт
description: Узнайте, как настроить единый вход для Azure Active Directory и SilkRoad Life Suite.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/16/2019
ms.author: jeedes
ms.openlocfilehash: 89d086ce136885e203e300f04bdbf3ade0affeb3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92516058"
---
# <a name="tutorial-azure-active-directory-integration-with-silkroad-life-suite"></a>Руководство по интеграции Azure Active Directory с решением SilkRoad Life Suite

В этом руководстве описано, как интегрировать SilkRoad Life Suite с Azure Active Directory (Azure AD).
Интеграция SilkRoad Life Suite с Azure AD обеспечивает следующие преимущества:

* С помощью Azure AD вы можете контролировать доступ к SilkRoad Life Suite.
* Можно включить автоматический вход пользователей в SilkRoad Life Suite (единый вход) с помощью учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Чтобы настроить интеграцию Azure AD с SilkRoad Life Suite, вам потребуется следующее:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/));
* подписка SilkRoad Life Suite с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* SilkRoad Life Suite поддерживает единый вход, инициируемый **поставщиком услуг**.

## <a name="adding-silkroad-life-suite-from-the-gallery"></a>Добавление SilkRoad Life Suite из коллекции.

Чтобы настроить интеграцию SilkRoad Life Suite в Azure AD, необходимо добавить SilkRoad Life Suite из коллекции в список управляемых приложений SaaS.

**Чтобы добавить SilkRoad Life Suite из коллекции, выполните следующие действия:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **SilkRoad Life Suite**, выберите **SilkRoad Life Suite** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![SilkRoad Life Suite в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в приложение SilkRoad Life Suite с использованием тестового пользователя **Britta Simon**.
Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в SilkRoad Life Suite.

Чтобы настроить и проверить единый вход Azure AD в SilkRoad Life Suite, вам потребуется выполнить действия в следующих стандартных блоках:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в SilkRoad Life Suite](#configure-silkroad-life-suite-single-sign-on)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя SilkRoad Life Suite](#create-silkroad-life-suite-test-user)** требуется для того, чтобы в SilkRoad Life Suite существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход в Azure AD в SilkRoad Life Suite, выполните следующие действия.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **SilkRoad Life Suite** щелкните **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. Если у вас есть **файл метаданных поставщика служб**, выполните следующие действия в разделе **Базовая конфигурация SAML**:

    > [!NOTE]
    > Как получить **файл метаданных поставщика услуг** описано далее в этом руководстве.

    а. Щелкните **Отправить файл метаданных**.

    ![Снимок экрана: раздел "Базовая конфигурация SAML" со ссылкой "Отправить файл метаданных".](common/upload-metadata.png)

    b. Щелкните **значок папки**, выберите файл метаданных и нажмите кнопку **Отправить**.

    ![Снимок экрана: диалоговое окно, где можно выбрать и отправить файл.](common/browse-upload-metadata.png)

    c. После успешной передачи файла метаданных значения **Идентификатор** и **URL-адрес ответа** в разделе "Базовая конфигурация SAML" заполнятся автоматически.

    ![Снимок экрана, на котором показан раздел "Базовая конфигурация SAML", где можно ввести идентификатор, URL-адрес ответа и нажать кнопку "Сохранить"](common/sp-identifier-reply.png)

    > [!Note]
    > Если поля **Идентификатор** и **URL-адрес ответа** автоматически не заполняются значениями, введите эти значения вручную в соответствии со своими требованиями.

    d. В текстовое поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<subdomain>.silkroad-eng.com/Authentication/`.

5. Если у вас нет **файла метаданных поставщика услуг**, выполните следующие действия в разделе **Базовая конфигурация SAML**.

    ![Сведения о домене и URL-адресах единого входа приложения SilkRoad Life Suite](common/sp-identifier-reply.png)

    а. В текстовое поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<subdomain>.silkroad-eng.com/Authentication/`.

    b. В поле **Идентификатор** введите URL-адрес в следующем формате:

    - `https://<subdomain>.silkroad-eng.com/Authentication/SP`
    - `https://<subdomain>.silkroad.com/Authentication/SP`

    c. В текстовом поле **URL-адрес ответа** введите URL-адрес в таком формате:

    - `https://<subdomain>.silkroad-eng.com/Authentication/`
    - `https://<subdomain>.silkroad.com/Authentication/`

    > [!NOTE]
    > Эти значения приведены для примера. Укажите вместо них фактические значения URL-адреса для входа, идентификатора и URL-адреса ответа. Чтобы получить эти значения, обратитесь в [службу поддержки клиентов SilkRoad Life Suite](https://www.silkroad.com/locations/). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

6. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

7. Требуемые URL-адреса можно скопировать из раздела **Настройка SilkRoad Life Suite**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

    а. URL-адрес входа.

    b. Идентификатор Azure AD

    c. URL-адрес выхода.

### <a name="configure-silkroad-life-suite-single-sign-on"></a>Настройка единого входа в SilkRoad Life Suite

1. Войдите на сайт компании SilkRoad в качестве администратора.

    > [!NOTE]
    > Чтобы получить доступ к приложению SilkRoad Life Suite Authentication для настройки федерации с Microsoft Azure AD, обратитесь в службу поддержки компании SilkRoad или к представителю отдела обслуживания клиентов компании SilkRoad.

1. В разделе **Service Provider** (Поставщик услуг) щелкните **Federation Details** (Сведения о федерации).

    ![Снимок экрана: параметр Federation Details (Сведения о федерации), выбранный в разделе Service Provider (Поставщик услуг).](./media/silkroad-life-suite-tutorial/tutorial_silkroad_06.png)

1. Щелкните **Download Federation Metadata**(Загрузить метаданные федерации), а затем сохраните файл метаданных на вашем компьютере. Укажите скачанный файл метаданных федерации как **файл метаданных поставщика услуг** в разделе **Базовая конфигурация SAML** на портале Azure.

    ![Снимок экрана: ссылка для скачивания метаданных федерации.](./media/silkroad-life-suite-tutorial/tutorial_silkroad_07.png)

1. В приложении **SilkRoad** щелкните **Authentication Sources** (Источники проверки подлинности).

    ![Снимок экрана: выбранный элемент Authentication Sources (Источники аутентификации).](./media/silkroad-life-suite-tutorial/tutorial_silkroad_08.png) 

1. Щелкните **Add Authentication Source**(Добавить источник аутентификации).

    ![Снимок экрана: ссылка "Добавить источник аутентификации".](./media/silkroad-life-suite-tutorial/tutorial_silkroad_09.png)

1. В разделе **Add Authentication Source** (Добавление источника аутентификации) выполните следующие действия:

    ![Снимок экрана: раздел добавления источника аутентификации с выбранной кнопкой Create Identity Provider using File Data (Создать поставщик удостоверений с помощью данных файлов).](./media/silkroad-life-suite-tutorial/tutorial_silkroad_10.png)
  
    а. В разделе **Option 2 - Metadata File** (Вариант 2 — файл метаданных) нажмите кнопку **Browse** (Обзор), чтобы отправить скачанный файл метаданных с портала Azure.
  
    b. Нажмите кнопку **Создать поставщик удостоверений с помощью данных из файла**.

1. В разделе **Authentication Sources** (Источники проверки подлинности) нажмите кнопку **Edit** (Изменить).

    ![Снимок экрана: выбранный элемент Edit (Изменить) в меню Authentication Sources (Источники аутентификации).](./media/silkroad-life-suite-tutorial/tutorial_silkroad_11.png)

1. В диалоговом окне **Edit Authentication Source** (Изменение источника аутентификации) выполните следующие действия:

    ![Снимок экрана: диалоговое окно Edit Authentication Source (Изменение источника аутентификации), где можно ввести описанные значения.](./media/silkroad-life-suite-tutorial/tutorial_silkroad_12.png)

    а. В поле **Enabled** (Включено) выберите **Yes** (Да).

    b. В текстовое поле **EntityId** (Идентификатор сущности) вставьте значение **Идентификатор Azure AD**, скопированное на портале Azure.

    c. В текстовом поле **IdP Description** (Описание IdP) введите описание вашей конфигурации (например, *Azure AD SSO*).

    d. В текстовом поле **Файл метаданных** передайте файл **метаданных**, загруженный с портала Azure.
  
    д) В текстовом поле **IdP Name** (Имя IdP) введите уникальное имя вашей конфигурации (например, *Azure SP*).
  
    е) В текстовое поле **Logout Service URL** (URL-адрес службы выхода) вставьте значение **URL-адрес выхода**, скопированное на портале Azure.

    ж. В текстовом поле **Sign-On service URL** (URL-адрес службы входа) вставьте значение **URL-адрес входа**, скопированное на портале Azure.

    h. Выберите команду **Сохранить**.

1. Отключите все остальные источники аутентификации.

    ![Снимок экрана: раздел Authentication Sources (Источники аутентификации), где можно отключить другие источники. ](./media/silkroad-life-suite-tutorial/tutorial_silkroad_13.png)

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD

Цель этого раздела — создать на портале Azure тестового пользователя с именем Britta Simon.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.

    ![Ссылки "Пользователи и группы" и "Все пользователи"](common/users.png)

2. В верхней части экрана выберите **Новый пользователь**.

    ![Кнопка "Новый пользователь"](common/new-user.png)

3. В разделе свойств пользователя сделайте следующее:

    ![Диалоговое окно "Пользователь"](common/user-properties.png)

    а. В поле **Имя** введите **BrittaSimon**.
  
    b. В поле **Имя пользователя** введите `brittasimon@yourcompanydomain.extension`.  
    Например BrittaSimon@contoso.com.

    c. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле "Пароль".

    d. Нажмите кнопку **Создать**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе мы предоставим пользователю Britta Simon доступ к SilkRoad Life Suite, чтобы он мог использовать единый вход Azure.

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем — **SilkRoad Life Suite**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **SilkRoad Life Suite**.

    ![Ссылка SilkRoad Life Suite в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-silkroad-life-suite-test-user"></a>Создание тестового пользователя SilkRoad Life Suite

В этом разделе описано, как создать пользователя Britta Simon в приложении SilkRoad Life Suite. Сотрудничайте со [службой поддержки клиентов SilkRoad Life Suite](https://www.silkroad.com/locations/) для добавления пользователей на платформу SilkRoad Life Suite. Перед использованием единого входа необходимо создать и активировать пользователей.

### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув элемент "SilkRoad Life Suite" на Панели доступа, вы автоматически войдете в приложение SilkRoad Life Suite, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)