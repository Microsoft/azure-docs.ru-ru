---
title: Руководство по интеграции единого входа Azure Active Directory с приложением Count Me In — Operations Dashboard | Документация Майкрософт
description: Сведения о настройке единого входа Azure Active Directory для Count Me In — Operations Dashboard.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/30/2020
ms.author: jeedes
ms.openlocfilehash: 3339516193af6e1ff832ac586f4a81f8799c5b83
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98727698"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-count-me-in---operations-dashboard"></a>Руководство по интеграции единого входа Azure Active Directory с приложением Count Me In — Operations Dashboard

В этом руководстве описано, как интегрировать Count Me In — Operations Dashboard с Azure Active Directory (Azure AD). Интеграция Count Me In — Operations Dashboard с Azure AD предоставляет следующие возможности:

* Контроль доступа к Count Me In — Operations Dashboard с помощью Azure AD.
* Ваши пользователи смогут входить в Count Me In — Operations Dashboard с учетными записями Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Count Me In — Operations Dashboard с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Count Me In — Operations Dashboard поддерживает единый вход, инициируемый **поставщиком служб**.

## <a name="adding-count-me-in---operations-dashboard-from-the-gallery"></a>Добавление Count Me In — Operations Dashboard из коллекции

Чтобы настроить интеграцию Count Me In — Operations Dashboard с Azure AD, необходимо добавить Count Me In — Operations Dashboard из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Count Me In — Operations Dashboard**.
1. Выберите **Count Me In — Operations Dashboard** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-count-me-in---operations-dashboard"></a>Настройка и проверка единого входа Azure AD для Count Me In — Operations Dashboard

Настройте и проверьте единый вход Azure AD в Count Me In — Operations Dashboard с помощью тестового пользователя **B.Simon**. Для обеспечения единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в приложении Count Me In — Operations Dashboard.

Чтобы настроить и проверить единый вход Azure AD в Count Me In — Operations Dashboard, выполните следующие действия:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Count Me In — Operations Dashboard](#configure-count-me-in-operations-dashboard-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя Count Me In — Operations Dashboard](#create-count-me-in-operations-dashboard-test-user)** нужно для того, чтобы в Count Me In — Operations Dashboard существовал пользователь B.Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Count Me In — Operations Dashboard** найдите раздел **Управление** и выберите элемент **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://api-us.localz.io/user/v1/saml/initsso?projectId=<PROJECT_ID>`.

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате: `api-us.localz.io/<PROJECT_ID>`.

    c. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://api-us.localz.io/user/v1/saml/initsso?projectId=<PROJECT_ID>`.

    > [!NOTE]
    > Эти значения приведены для примера. Необходимо обновить эти значения действующим URL-адресом для входа и идентификатором. Обратитесь к [службе поддержке Count Me In — Operations Dashboard](mailto:support@localz.co), чтобы получить эти значения. Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. Приложение Count Me In — Operations Dashboard ожидает проверочные утверждения SAML в определенном формате, поэтому вам нужно добавить сопоставления настраиваемых атрибутов в конфигурацию атрибутов токена SAML. На следующем снимке экрана показан список атрибутов по умолчанию.

    ![Изображение](common/default-attributes.png)

1. Кроме того, приложение Count Me In — Operations Dashboard ожидает несколько дополнительных атрибутов в ответе SAML, как показано ниже. Эти атрибуты также заранее заполнены, но вы можете изменить их в соответствии со своими требованиями.
    
    | Имя |  Исходный атрибут|
    | ----------- | --------- |
    | назначенные роли | user.assignedroles |

    > [!NOTE]
    > Count Me In — Operations Dashboard ожидает, что в приложении пользователям будут назначены роли. Настройте эти роли в Azure AD, чтобы пользователям можно было назначить соответствующие роли. Чтобы понять, как настроить роли в Azure AD, см. [здесь](../develop/howto-add-app-roles-in-azure-ad-apps.md#app-roles-ui--preview).

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите пункт **Сертификат (Base64)** и щелкните **Скачать**, чтобы скачать сертификат. Сохраните этот сертификат на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

1. Требуемые URL-адреса можно скопировать из раздела **Настройка Count Me In — Operations Dashboard**.

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

В этом разделе показано, как включить единый вход в Azure для пользователя B.Simon, предоставив этому пользователю доступ к Count Me In — Operations Dashboard.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Count Me In — Operations Dashboard**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-count-me-in-operations-dashboard-sso"></a>Настройка единого входа в Count Me In — Operations Dashboard

Чтобы настроить единый вход на стороне **Count Me In — Operations Dashboard**, нужно отправить скачанный **сертификат (Base64)** и соответствующие URL-адреса, скопированные на портале Azure, [службе поддержки Count Me In — Operations Dashboard](mailto:support@localz.co). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-count-me-in-operations-dashboard-test-user"></a>Создание тестового пользователя Count Me In — Operations Dashboard

В этом разделе описано, как создать пользователя B.Simon в приложении Count Me In — Operations Dashboard. Обратитесь в [службу поддержки Count Me In — Operations Dashboard](mailto:support@localz.co), чтобы добавить пользователей на платформу Count Me In — Operations Dashboard. Перед использованием единого входа необходимо создать и активировать пользователей.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены на URL-адрес для входа в Count Me In — Operations Dashboard, где можно инициировать поток входа. 

* Перейдите по URL-адресу для входа в приложение Count Me In — Operations Dashboard и инициируйте поток входа.

* Вы можете использовать портал "Мои приложения" корпорации Майкрософт. Теперь при выборе плитки Count Me In — Operations Dashboard в разделе "Мои приложения" вы будете перенаправлены по URL-адресу входа в Count Me In — Operations Dashboard. Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Дальнейшие действия

После настройки Count Me In — Operations Dashboard вы можете применить функцию управления сеансами, которая в режиме реального времени защищает конфиденциальные данные вашей организации от кражи и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).
