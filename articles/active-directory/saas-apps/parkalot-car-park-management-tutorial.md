---
title: Руководство. Интеграция единого входа Azure Active Directory с Parkalot — Car park management | Документация Майкрософт
description: Сведения о том, как настроить единый вход Azure Active Directory в Parkalot — Car park management.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/11/2021
ms.author: jeedes
ms.openlocfilehash: 0098d28d2b211ad9e4bbe60a44c5de4058f2b96b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101651434"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-parkalot---car-park-management"></a>Руководство. Интеграция единого входа Azure Active Directory с Parkalot — Car park management

В этом руководстве описано, как интегрировать приложение Parkalot — Car park management с Azure Active Directory (Azure AD). Интеграция Parkalot — Car park management с Azure AD предоставляет следующие возможности:

* контроль доступа к Parkalot — Car park management с помощью Azure AD;
* автоматический вход пользователей в Parkalot — Car park management с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Parkalot — Car park management с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Parkalot — Car park management поддерживает единый вход, инициированный **поставщиком услуг**.

* Parkalot — Car park management поддерживает **JIT**-подготовку пользователей.

## <a name="adding-parkalot---car-park-management-from-the-gallery"></a>Добавление Parkalot — Car park management из коллекции

Чтобы настроить интеграцию Parkalot — Car park management с Azure AD, необходимо добавить Parkalot — Car park management из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Parkalot — Car park management**.
1. Выберите **Parkalot — Car park management** на панели результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.


## <a name="configure-and-test-azure-ad-sso-for-parkalot---car-park-management"></a>Настройка и проверка единого входа Azure AD для Parkalot — Car park management

Настройте и проверьте единый вход Azure AD в Parkalot — Car park management с помощью тестового пользователя **B.Simon**. Для работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Parkalot — Car park management.

Чтобы настроить и проверить единый вход Azure AD в Parkalot — Car park management, выполните следующие действия:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Parkalot — Car park management](#configure-parkalot-car-park-management-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя Parkalot — Car park management](#create-parkalot-car-park-management-test-user)** требуется для того, чтобы в Parkalot — Car park management существовал пользователь B.Simon, связанный с одноименным представлением в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Parkalot — Car park management** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовое поле **Идентификатор (сущности)** введите URL-адрес в следующем формате:

    | Идентификатор (идентификатор сущности) |
    | -------------- |
    | `https://parkalot.io` |
    | `https://<CUSTOMERNAME>.parkalot.io` |
    |

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в одном из таких форматов:

    | URL-адрес ответа |
    | -------------- |
    | `https://<CUSTOMERNAME>.parkalot.io` |
    | `https://parkalot-saml.firebaseapp.com/__/auth/handler` |
    | `https://parkalot-saml.web.app/__/auth/handler` |
    | `https://<CustomerName>.parkalot.io/__/auth/handler` |
    |

    c. В текстовом поле **URL-адрес входа** введите URL-адрес в одном из таких форматов:

    | URL-адрес входа |
    | -------------- |
    | `https://<CUSTOMERNAME>.parkalot.io/#/login` |
    | `https://parkalot-saml.firebaseapp.com/#/login` |
    | `https://parkalot-saml.web.app/#/login` |
    |

    > [!NOTE]
    > Эти значения приведены для примера. Замените их фактическими значениями идентификатора, URL-адреса ответа и URL-адреса входа. Чтобы получить эти значения, обратитесь в [техническую поддержку клиентов Parkalot — Car park management](mailto:contact-us@parkalot.io). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите пункт **Сертификат (Base64)** и щелкните **Скачать**, чтобы скачать сертификат. Сохраните этот сертификат на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

1. Требуемые URL-адреса можно скопировать из раздела **Настройка Parkalot — Car park management**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD

В этом разделе описано, как на портале Azure создать тестового пользователя с именем B.Simon.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.
1. В верхней части экрана выберите **Новый пользователь**.
1. В разделе **Свойства пользователя** выполните следующие действия.
   1. В поле **Имя** введите `B.Simon`.  
   1. В поле **Имя пользователя** введите username@companydomain.extension. Например, `B.Simon@contoso.com`.
   1. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле **Пароль**.
   1. Нажмите кнопку **Создать**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как разрешить пользователю B.Simon использовать единый вход Azure, предоставив такому пользователю доступ к Parkalot — Car park management.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. Выберите **Parkalot — Car park management** в списке приложений.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-parkalot-car-park-management-sso"></a>Настройка единого входа для Parkalot — Car park management

1. В другом окне веб-браузера войдите на корпоративный сайт Parkalot — Car park management с правами администратора.

1. Выберите **Setup SAML** (Настроить SAML) и щелкните значок **Edit** (Редактировать) на карточке **Add New** (Добавить).

    ![Значок Edit (Редактировать) на карточке Add New (Добавить).](./media/parkalot-car-park-management-tutorial/setup-saml.png)

1. Выполните описанные действия на следующей странице.

    ![Настройка единого входа для Parkalot — Car park management](./media/parkalot-car-park-management-tutorial/configuration.png)

    а. В текстовое поле **Display Name** (Отображаемое имя) введите допустимое имя.

    b. В текстовое поле **IdP Entity ID** (Идентификатор сущности поставщика удостоверений) вставьте значение **идентификатора Azure AD**, скопированное на портале Azure.

    c. В текстовое поле **SSO URL** (URL-адрес единого входа) вставьте значение **URL-адреса входа**, скопированное на портале Azure.

    d. Откройте в блокноте скачанный с портала Azure файл **сертификата (Base64)** и вставьте его содержимое в текстовое поле **Certificate** (Сертификат).

    д) Щелкните **СОХРАНИТЬ**.

### <a name="create-parkalot-car-park-management-test-user"></a>Создание тестового пользователя Parkalot — Car park management

С помощью инструкций из этого раздела вы создадите в Parkalot — Car park management пользователя с именем Britta Simon. В Parkalot — Car park management поддерживается JIT-подготовка пользователей, которая включена по умолчанию. В этом разделе никакие действия с вашей стороны не требуются. Если пользователь еще не существует в Parkalot — Car park management, пользователь будет создан после аутентификации.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в Parkalot — Car park management, где можно инициировать поток входа. 

* Перейдите по URL-адресу для входа в Parkalot — Car park management и инициируйте поток входа.

* Вы можете использовать портал "Мои приложения" корпорации Майкрософт. Щелкнув плитку Parkalot — Car park management на портале "Мои приложения", вы перейдете по URL-адресу для входа в Parkalot — Car park management. Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Дальнейшие действия

После настройки Parkalot — Car park management вы можете применить функцию управления сеансами, которая в реальном времени защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).