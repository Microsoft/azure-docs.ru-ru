---
title: Руководство по интеграции Azure Active Directory с Symantec Web Security Service (WSS) | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Symantec Web Security Service (WSS).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/25/2018
ms.author: jeedes
ms.openlocfilehash: 230f7fd9c62f657ce8ab893db2256808dce9a7ba
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92518370"
---
# <a name="tutorial-azure-active-directory-integration-with-symantec-web-security-service-wss"></a>Руководство по интеграции Azure Active Directory с Symantec Web Security Service (WSS)

Изучив это руководство, вы сможете интегрировать учетную запись Symantec Web Security Service (WSS) с учетной записью Azure Active Directory (Azure AD). После этого WSS сможет выполнять аутентификацию SAML для конечных пользователей, подготовленных в Azure AD, и применять к ним правила политик на уровне пользователя или группы.

Интеграция Azure AD с Symantec Web Security Service (WSS) обеспечивает следующие преимущества:

- Управление через портал Azure AD всеми конечными пользователями и группами, используемыми в учетной записи WSS.

- Аутентификация конечных пользователей в WSS с учетными данными из Azure AD.

- Применение правил политики на уровне пользователей и групп, определенных в учетной записи WSS.

Дополнительные сведения об интеграции приложений SaaS с Azure AD см. в статье [Единый вход в приложениях в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с Symantec Web Security Service (WSS), вам потребуется:

* Подписка Azure AD. (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* Подписка Symantec Web Security Service (WSS) с поддержкой единого входа

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Symantec Web Security Service (WSS) поддерживает единый вход инициированной **выдающей точки распространения**.

## <a name="adding-symantec-web-security-service-wss-from-the-gallery"></a>Добавление Symantec Web Security Service (WSS) из коллекции

Чтобы настроить интеграцию Symantec Web Security Service (WSS) с Azure AD, вам потребуется добавить Symantec Web Security Service (WSS) из коллекции в список управляемых приложений SaaS.

**Чтобы добавить Symantec Web Security Service (WSS) из коллекции, выполните следующие действия.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Symantec Web Security Service (WSS)**, затем выберите **Symantec Web Security Service (WSS)** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить приложение.

     ![Служба Symantec Web Security Service (WSS) в списке результатов](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD с помощью Symantec Web Security Service (WSS) с использованием тестового пользователя **Britta Simon**.
Иными словами, для создания единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Symantec Web Security Service (WSS).

Чтобы настроить и проверить единый вход Azure AD в Symantec Web Security Service (WSS), вам потребуется выполнить действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
2. **Настройка единого входа для Symantec Web Security Service (WSS)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы разрешить пользователю Britta Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя Symantec Web Security Service (WSS)](#create-symantec-web-security-service-wss-test-user)** требуется для того, чтобы в Symantec Web Security Service (WSS) существовал пользователь Britta Simon, связанный с представлением пользователя в Azure AD.
6. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы проверить работу конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано включение единого входа Azure AD на портале Azure.

Чтобы настроить единый вход Azure AD в Symantec Web Security Service (WSS), выполните следующие действия.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Symantec Web Security Service (WSS)** выберите **Единый вход**.

    ![Ссылка "Настройка единого входа"](common/select-sso.png)

2. В диалоговом окне **Выбрать метод единого входа** выберите режим **SAML/WS-Fed**, чтобы включить единый вход.

    ![Режим выбора единого входа](common/select-saml-option.png)

3. На странице **Настройка единого входа с помощью SAML** щелкните **Изменить**, чтобы открыть диалоговое окно **Базовая конфигурация SAML**.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В диалоговом окне **Базовая конфигурация SAML** выполните следующие действия.

    ![Информация о домене и URL-адресах для единого входа в службе Symantec Web Security Service (WSS)](common/idp-intiated.png)

    а. В текстовом поле **Идентификатор** введите URL-адрес: `https://saml.threatpulse.net:8443/saml/saml_realm`

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес `https://saml.threatpulse.net:8443/saml/saml_realm/bcsamlpost`

    > [!NOTE]
    > Если по какой-либо причине предложенные значения **Идентификатора** и **URL-адреса ответа** не работают, обратитесь в [группу поддержки Symantec Web Security Service (WSS)](https://www.symantec.com/contact-us). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

### <a name="configure-symantec-web-security-service-wss-single-sign-on"></a>Настройка единого входа для Symantec Web Security Service (WSS)

Чтобы настроить единый вход на стороне службы безопасности Symantec Web Symantec Web Security Service (WSS), воспользуйтесь электронной документацией по WSS. Скачанный **XML метаданных федерации** следует импортировать на портал WSS. Если вам потребуется помощь в использовании портала WSS, обратитесь в [службу поддержки Symantec Web Security Service (WSS)](https://www.symantec.com/contact-us).

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

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к Symantec Web Security Service (WSS).

1. На портале Azure выберите **Корпоративные приложения**, **Все приложения**, а затем **Symantec Web Security Service (WSS)**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений введите и выберите **Symantec Web Security Service (WSS)**.

    ![Ссылка на Symantec Web Security Service (WSS) в списке приложений](common/all-applications.png)

3. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

4. Нажмите кнопку **Добавить пользователя**, а затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"](common/add-assign-user.png)

5. В диалоговом окне **Пользователи и группы** из списка пользователей выберите **Britta Simon**, а затем в верхней части экрана нажмите кнопку **Выбрать**.

6. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор ролей** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.

7. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-symantec-web-security-service-wss-test-user"></a>Создание тестового пользователя Symantec Web Security Service (WSS)

В этом разделе описано, как создать пользователя Britta Simon в Symantec Web Security Service (WSS). Соответствующее имя пользователя можно создать вручную на портале WSS. Также можно дождаться автоматической синхронизации с порталом WSS всех пользователей и групп, подготовленных в Azure AD (на это потребуется несколько минут, обычно около 15). Перед использованием единого входа необходимо создать и активировать пользователей. Общедоступный IP-адрес компьютера конечного пользователя, который будет использоваться для просмотра веб-сайтов, также должен быть подготовлен на портале Symantec Web Security Service (WSS).

> [!NOTE]
> Щелкните [здесь](https://www.bing.com/search?q=my+ip+address&qs=AS&pq=my+ip+a&sc=8-7&cvid=29A720C95C78488CA3F9A6BA0B3F98C5&FORM=QBLH&sp=1), чтобы получить общедоступный IP-адрес своего компьютера.

### <a name="test-single-sign-on"></a>Проверка единого входа 

В этом разделе вы проверите работу единого входа, который вы настроили для учетной записи WSS, чтобы для нее выполнялась аутентификация SAML через Azure AD.

После того как вы настроите в веб-браузере передачу трафика через WSS, при попытке перехода на любой сайт будет открываться страница входа в Azure. Введите учетные данные тестового конечного пользователя, которые вы подготовили в Azure AD (в нашем примере это BrittaSimon). После аутентификации вы получите доступ к просмотру выбранного веб-сайта. Если вы создадите правило политики, которое запрещает пользователю BrittaSimon просмотр определенного сайта, при попытке входа на этот сайт с учетными данными BrittaSimon вы должны увидеть страницу блокировки WSS.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)