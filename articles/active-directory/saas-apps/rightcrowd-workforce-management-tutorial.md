---
title: Руководство по Интеграция единого входа Azure Active Directory с приложением RightCrowd Workforce Management | Документация Майкрософт
description: Сведения о том, как настроить единый вход между Azure Active Directory и RightCrowd Workforce Management.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/24/2020
ms.author: jeedes
ms.openlocfilehash: 53abb706bd741e4a7ce548a89dc324dc68376179
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98725430"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-rightcrowd-workforce-management"></a>Руководство по Интеграция единого входа Azure Active Directory с приложением RightCrowd Workforce Management

Из этого учебника вы узнаете, как интегрировать приложение RightCrowd Workforce Management с Azure Active Directory (Azure AD). Интеграция RightCrowd Workforce Management с Azure AD предоставляет следующие возможности:

* Управление доступом к RightCrowd Workforce Management с помощью Azure AD.
* Автоматический вход пользователей в RightCrowd Workforce Management с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка RightCrowd Workforce Management с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.



* RightCrowd Workforce Management поддерживает единый вход, инициированный **поставщиком службы или поставщиком удостоверений**.
* RightCrowd Workforce Management поддерживает **JIT**-подготовку пользователей.


## <a name="adding-rightcrowd-workforce-management-from-the-gallery"></a>Добавление RightCrowd Workforce Management из коллекции

Чтобы настроить интеграцию RightCrowd Workforce Management с Azure AD, нужно добавить RightCrowd Workforce Management из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **RightCrowd Workforce Management**.
1. Выберите **RightCrowd Workforce Management** на панели результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.


## <a name="configure-and-test-azure-ad-sso-for-rightcrowd-workforce-management"></a>Настройка и проверка единого входа Azure AD для RightCrowd Workforce Management

Настройте и проверьте единый вход Azure AD в RightCrowd Workforce Management с помощью тестового пользователя **B.Simon**. Для работы единого входа нужно установить связь между пользователем Azure AD и соответствующим пользователем в RightCrowd Workforce Management.

Чтобы настроить и проверить единый вход Azure AD в RightCrowd Workforce Management, выполните действия из следующих стандартных блоков:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в RightCrowd Workforce Management](#configure-rightcrowd-workforce-management-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя RightCrowd Workforce Management](#create-rightcrowd-workforce-management-test-user)** требуется для того, чтобы в RightCrowd Workforce Management существовал пользователь B.Simon, связанный с представлением этого же пользователя в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **RightCrowd Workforce Management** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **Идентификатор** введите один из следующих URL-адресов: `http://<SUBDOMAIN>.rightcrowdcustomerdomain.com`

    b. В текстовом поле **URL-адрес ответа** введите один из следующих URL-адресов: `https://<SUBDOMAIN>.rightcrowdcustomerdomain.com/RightCrowd/Saml2/Auth/AssertionComsumerService`


1. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    В текстовом поле **URL-адрес входа** введите один из следующих URL-адресов: `http://<SUBDOMAIN>.rightcrowdcustomerdomain.com`

    > [!NOTE]
    > Эти значения приведены для примера. Вместо них необходимо указать фактические значения URL-адреса входа, идентификатора и URL-адреса ответа. Чтобы получить эти значения, [обратитесь к группе поддержки RightCrowd Workforce Management](mailto:info@rightcrowd.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. Выберите команду **Сохранить**.

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите элемент **XML метаданных федерации** и выберите **Скачать**, чтобы скачать сертификат и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

1. Требуемые URL-адреса можно скопировать из раздела **Настройка RightCrowd Workforce Management**.

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

В этом разделе описано, как разрешить пользователю B.Simon использовать единый вход Azure, предоставив такому пользователю доступ к RightCrowd Workforce Management.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **RightCrowd Workforce Management**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-rightcrowd-workforce-management-sso"></a>Настройка единого входа в RightCrowd Workforce Management

Чтобы настроить единый вход на стороне приложения **RightCrowd Workforce Management**, нужно отправить скачанный **XML-файл метаданных федерации** и соответствующие URL-адреса, скопированные на портале Azure, [группе поддержки RightCrowd Workforce Management](mailto:info@rightcrowd.com). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-rightcrowd-workforce-management-test-user"></a>Создание тестового пользователя в RightCrowd Workforce Management

С помощью инструкций из этого раздела вы создадите в RightCrowd Workforce Management пользователя с именем B.Simon. RightCrowd Workforce Management поддерживает JIT-подготовку пользователей, которая включена по умолчанию. В этом разделе никакие действия с вашей стороны не требуются. Если пользователь еще не существует в RightCrowd Workforce Management, он будет создан после аутентификации.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

#### <a name="sp-initiated"></a>Инициация поставщиком услуг:

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в RightCrowd Workforce Management, где можно инициировать поток входа.  

* Перейдите непосредственно по URL-адресу входа в RightCrowd Workforce Management и запустите поток входа.

#### <a name="idp-initiated"></a>Вход, инициированный поставщиком удостоверений

* На портале Azure выберите **Тестировать приложение**, и вы автоматически войдете в приложение RightCrowd Workforce Management, для которого настроен единый вход. 

Вы можете также использовать портал "Мои приложения" корпорации Майкрософт для тестирования приложения в любом режиме. Щелкнув плитку RightCrowd Workforce Management на портале "Мои приложения", вы автоматически перейдете на страницу входа приложения для инициации потока входа (при настройке в режиме поставщика услуг) или в приложение RightCrowd Workforce Management, для которого настроен единый вход (при настройке в режиме поставщика удостоверений). Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Дальнейшие действия

Завершив настройку RightCrowd Workforce Management, вы сможете применить функцию управления сеансом, которая в реальном времени защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).