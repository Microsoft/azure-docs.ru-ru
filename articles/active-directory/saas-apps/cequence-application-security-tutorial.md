---
title: Руководство по интеграции единого входа Azure Active Directory с Cequence Application Security Platform | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Cequence Application Security Platform.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/25/2020
ms.author: jeedes
ms.openlocfilehash: 967ad66434a09252c1b7afa195facee20279f4dd
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98735300"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-cequence-application-security-platform"></a>Руководство по интеграции единого входа Azure Active Directory с Cequence Application Security Platform

В этом учебнике описано, как интегрировать Cequence Application Security Platform с Azure Active Directory (Azure AD). Интеграция Cequence Application Security Platform с Azure AD обеспечивает следующие возможности:

* Контроль доступа к Cequence Application Security Platform с помощью Azure AD.
* Автоматический вход пользователей в Cequence Application Security Platform с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Cequence Application Security Platform с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Приложение Cequence Application Security Platform поддерживает единый вход, инициируемый **поставщиком услуг**.

* Приложение Cequence Application Security Platform поддерживает **JIT-подготовку** пользователей.


## <a name="adding-cequence-application-security-platform-from-the-gallery"></a>Добавление Cequence Application Security Platform из коллекции

Чтобы настроить интеграцию Cequence Application Security Platform с Azure AD, вам нужно добавить Cequence Application Security Platform из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Cequence Application Security Platform**.
1. Выберите **Cequence Application Security Platform** на панели результатов, а затем добавьте приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.


## <a name="configure-and-test-azure-ad-sso-for-cequence-application-security-platform"></a>Настройка и проверка единого входа Azure AD в Cequence Application Security Platform

Настройте и проверьте единый вход Azure AD в Cequence Application Security Platform с помощью тестового пользователя **B. Simon**. Чтобы обеспечить работу единого входа, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Cequence Application Security Platform.

Чтобы настроить и проверить единый вход Azure AD в Cequence Application Security Platform, сделайте следующее:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Cequence Application Security Platform](#configure-cequence-application-security-platform-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя Cequence Application Security Platform](#create-cequence-application-security-platform-test-user)** требуется для того, чтобы в Cequence Application Security Platform существовал пользователь B. Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Cequence Application Security Platform** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<CUSTOMERNAME>.s.cequence.cloud`.

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `https://<CUSTOMERNAME>.s.cequence.cloud:443/saml/metadata`.

    > [!NOTE]
    > Эти значения приведены для примера. Необходимо обновить эти значения действующим URL-адресом для входа и идентификатором. Чтобы получить эти значения, обратитесь в [группу поддержки клиентов Cequence Application Security Platform](mailto:support@cequence.ai). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. Приложение Cequence Application Security Platform предусматривает использование проверочных утверждений SAML в определенном формате, поэтому вам нужно добавить сопоставления настраиваемых атрибутов в конфигурацию атрибутов токена SAML. На следующем снимке экрана показан список атрибутов по умолчанию.

    ![Изображение](common/default-attributes.png)

1. Помимо перечисленного выше, приложение Cequence Application Security Platform ожидает несколько дополнительных атрибутов в ответе SAML, которые представлены ниже. Эти атрибуты также заранее заполнены, но вы можете изменить их в соответствии со своими требованиями.
    
    | Имя |  Исходный атрибут|
    | --------------- | --------- |
    | Группа | user.groups |

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** нажмите кнопку "Копировать", чтобы скопировать **URL-адрес метаданных федерации приложений** и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/copy-metadataurl.png)
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

В этом разделе описано, как включить единый вход Azure для пользователя B. Simon, предоставив этому пользователю доступ к Cequence Application Security Platform.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Cequence Application Security Platform**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-cequence-application-security-platform-sso"></a>Настройка единого входа в Cequence Application Security Platform

Чтобы настроить единый вход на стороне **Cequence Application Security Platform**, необходимо отправить **URL-адрес метаданных федерации приложений** в [группу поддержки Cequence Application Security Platform](mailto:support@cequence.ai). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-cequence-application-security-platform-test-user"></a>Создание тестового пользователя в Cequence Application Security Platform

Выполнив инструкции из этого раздела, вы создадите в Cequence Application Security Platform пользователя Britta Simon. Приложение Cequence Application Security Platform поддерживает JIT-подготовку пользователей, которая включена по умолчанию. В этом разделе никакие действия с вашей стороны не требуются. Если пользователь еще не существует в Cequence Application Security Platform, он создается после проверки подлинности.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

1. Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в Cequence Application Security Platform, где можно инициировать поток входа. 

2. Перейдите по URL-адресу для входа в Cequence Application Security Platform и инициируйте поток входа.

3. Вы можете использовать Панель доступа (Майкрософт). Щелкнув плитку Cequence Application Security Platform на Панели доступа, вы перейдете по URL-адресу входа в Cequence Application Security Platform. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)


## <a name="next-steps"></a>Дальнейшие действия

После настройки Cequence Application Security Platform вы сможете применить функцию управления сеансами, которая защищает конфиденциальные данные вашей организации от кражи и несанкционированного доступа в режиме реального времени. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).