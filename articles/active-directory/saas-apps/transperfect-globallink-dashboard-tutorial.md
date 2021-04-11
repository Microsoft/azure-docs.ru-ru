---
title: Учебник. Интеграция единого входа Azure Active Directory с TransPerfect GlobalLink Dashboard | Документация Майкрософт
description: Сведения о том, как настроить единый вход между Azure Active Directory и TransPerfect GlobalLink Dashboard.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/12/2021
ms.author: jeedes
ms.openlocfilehash: ad720f5e625d2054a5c79b47c9659d4d10d251e4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104955114"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-transperfect-globallink-dashboard"></a>Учебник. Интеграция единого входа Azure Active Directory с TransPerfect GlobalLink Dashboard

В этом руководстве описано, как интегрировать TransPerfect GlobalLink Dashboard с Azure Active Directory (Azure AD). Интеграция TransPerfect GlobalLink Dashboard с Azure AD предоставляет следующие возможности:

* управление доступом к TransPerfect GlobalLink Dashboard с помощью Azure AD;
* автоматический вход пользователей в TransPerfect GlobalLink Dashboard с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка TransPerfect GlobalLink Dashboard с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* TransPerfect GlobalLink Dashboard поддерживает единый вход, инициированный **поставщиком услуг и поставщиком удостоверений**.
* TransPerfect GlobalLink Dashboard поддерживает **JIT**-подготовку пользователей.

> [!NOTE]
> Идентификатор этого приложения — фиксированное строковое значение, поэтому в одном клиенте можно настроить только один экземпляр.

## <a name="adding-transperfect-globallink-dashboard-from-the-gallery"></a>Добавление TransPerfect GlobalLink Dashboard из коллекции

Чтобы настроить интеграцию TransPerfect GlobalLink Dashboard с Azure AD, необходимо добавить TransPerfect GlobalLink Dashboard из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **TransPerfect GlobalLink Dashboard**.
1. Выберите **TransPerfect GlobalLink Dashboard** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.


## <a name="configure-and-test-azure-ad-sso-for-transperfect-globallink-dashboard"></a>Настройка и проверка единого входа Azure AD для TransPerfect GlobalLink Dashboard

Настройте и проверьте единый вход Azure AD в TransPerfect GlobalLink Dashboard с помощью тестового пользователя **B.Simon**. Для обеспечения единого входа необходимо установить связь между пользователем AAD и соответствующим пользователем в приложении TransPerfect GlobalLink Dashboard.

Чтобы настроить и проверить единый вход Azure AD в TransPerfect GlobalLink Dashboard, выполните следующие действия:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в TransPerfect GlobalLink Dashboard](#configure-transperfect-globallink-dashboard-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя TransPerfect GlobalLink Dashboard](#create-transperfect-globallink-dashboard-test-user)** нужно для того, чтобы в TransPerfect GlobalLink Dashboard существовал пользователь B.Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **TransPerfect GlobalLink Dashboard** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. В разделе **Базовая конфигурация SAML** не нужно выполнять никаких действий, так как приложение уже предварительно интегрировано с Azure.

1. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    В текстовом поле **URL-адрес входа** введите URL-адрес: `https://sso.transperfect.com`.

1. Выберите команду **Сохранить**.

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

В этом разделе описано, как включить единый вход в Azure для пользователя B.Simon, предоставив этому пользователю доступ к TransPerfect GlobalLink Dashboard.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **TransPerfect GlobalLink Dashboard**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-transperfect-globallink-dashboard-sso"></a>Настройка единого входа для TransPerfect GlobalLink Dashboard

Чтобы настроить единый вход на стороне **TransPerfect GlobalLink Dashboard**, необходимо отправить **URL-адрес метаданных федерации приложения** в [службу поддержки TransPerfect GlobalLink Dashboard](mailto:TechOps_Consulting@transperfect.com). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-transperfect-globallink-dashboard-test-user"></a>Создание тестового пользователя TransPerfect GlobalLink Dashboard

В этом разделе описано, как в приложении TransPerfect GlobalLink Dashboard создать пользователя с именем B.Simon. Приложение TransPerfect GlobalLink Dashboard поддерживает JIT-подготовку, которая включена по умолчанию. В этом разделе никакие действия с вашей стороны не требуются. Если пользователь еще не существует в TransPerfect GlobalLink Dashboard, при попытке доступа к приложению TransPerfect GlobalLink Dashboard пользователь будет создан.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

#### <a name="sp-initiated"></a>Инициация поставщиком услуг:

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в TransPerfect GlobalLink Dashboard, где можно инициировать поток входа.  

* Перейдите по URL-адресу для входа в приложение TransPerfect GlobalLink Dashboard и инициируйте поток входа.

#### <a name="idp-initiated"></a>Вход, инициированный поставщиком удостоверений

* На портале Azure выберите **Тестировать приложение**, и вы автоматически войдете в приложение TransPerfect GlobalLink Dashboard, для которого настроили единый вход. 

Вы можете также использовать портал "Мои приложения" корпорации Майкрософт для тестирования приложения в любом режиме. Щелкнув плитку TransPerfect GlobalLink Dashboard на портале "Мои приложения", вы будете перенаправлены на страницу входа приложения для инициации потока входа (при настройке в режиме поставщика услуг) или автоматически войдете в приложение TransPerfect GlobalLink Dashboard, для которого настроен единый вход (при настройке в режиме поставщика удостоверений). Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Дальнейшие действия

После настройки TransPerfect GlobalLink Dashboard вы можете применить функцию управления сеансами, которая в реальном времени защищает конфиденциальные данные вашей организации от кражи и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).