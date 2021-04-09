---
title: Руководство по интеграции единого входа Azure Active Directory с VMware Horizon — Unified Access Gateway | Документация Майкрософт
description: Узнайте, как настроить единый вход для Azure Active Directory и VMware Horizon — Unified Access Gateway.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/04/2021
ms.author: jeedes
ms.openlocfilehash: fde57eb3727eda6f810f861102e47a9f5746d1f8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104955624"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-vmware-horizon---unified-access-gateway"></a>Руководство по интеграции единого входа Azure Active Directory с VMware Horizon — Unified Access Gateway

В этом учебнике описано, как интегрировать VMware Horizon — Unified Access Gateway с Azure Active Directory (Azure AD). Интеграция VMware Horizon — Unified Access Gateway с Azure AD обеспечивает следующие возможности.

* Управление доступом к VMware Horizon — Unified Access Gateway с помощью Azure AD.
* Автоматический вход пользователей в VMware Horizon — Unified Access Gateway с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка VMware Horizon — Unified Access Gateway с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* VMware Horizon — Unified Access Gateway поддерживает единый вход, инициированный **поставщиком услуг и поставщиком удостоверений**.

## <a name="add-vmware-horizon---unified-access-gateway-from-the-gallery"></a>Добавление VMware Horizon — Unified Access Gateway из коллекции

Чтобы настроить интеграцию VMware Horizon — Unified Access Gateway с Azure AD, необходимо добавить VMware Horizon — Unified Access Gateway из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **VMware Horizon — Unified Access Gateway**.
1. Выберите **VMware Horizon — Unified Access Gateway** на панели результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.


## <a name="configure-and-test-azure-ad-sso-for-vmware-horizon---unified-access-gateway"></a>Настройка и проверка единого входа Azure AD для VMware Horizon — Unified Access Gateway

Настройте и проверьте единый вход Azure AD в VMware Horizon — Unified Access Gateway с помощью тестового пользователя **B. Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в VMware Horizon — Unified Access Gateway.

Чтобы настроить и проверить единый вход Azure AD в VMware Horizon — Unified Access Gateway, выполните указанные ниже действия.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в VMware Horizon — Unified Access Gateway](#configure-vmware-horizon-unified-access-gateway-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя в VMware Horizon — Unified Access Gateway](#create-vmware-horizon-unified-access-gateway-test-user)** нужно для того, чтобы в VMware Horizon — Unified Access Gateway существовал пользователь B. Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **VMware Horizon — Unified Access Gateway** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `https://<HORIZON_UAG_FQDN>/portal`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<HORIZON_UAG_FQDN>/portal/samlsso`.

1. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://<HORIZON_UAG_FQDN>`.

    > [!NOTE]
    > Эти значения приведены для примера. Замените их фактическими значениями идентификатора, URL-адреса ответа и URL-адреса входа. Чтобы получить эти значения, обратитесь к [группе поддержки VMware Horizon — Unified Access Gateway](mailto:support@vmware.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите элемент **XML метаданных федерации** и выберите **Скачать**, чтобы скачать сертификат и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

1. Требуемые URL-адреса можно скопировать из раздела **Настройка VMware Horizon — Unified Access Gateway**.

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

В этом разделе описано, как включить единый вход Azure для пользователя B. Simon, предоставив этому пользователю доступ к приложению VMware Horizon — Unified Access Gateway.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **VMware Horizon — Unified Access Gateway**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-vmware-horizon-unified-access-gateway-sso"></a>Настройка единого входа VMware Horizon — Unified Access Gateway

Чтобы настроить единый вход на стороне **VMware Horizon — Unified Access Gateway**, нужно отправить скачанный **XML-файл метаданных федерации** и соответствующие URL-адреса, скопированные на портале Azure, в [группу поддержки VMware Horizon — Unified Access Gateway](mailto:support@vmware.com). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-vmware-horizon-unified-access-gateway-test-user"></a>Создание тестового пользователя VMware Horizon — Unified Access Gateway

В этом разделе описано, как создать пользователя B. Simon в приложении VMware Horizon — Unified Access Gateway. Чтобы добавить пользователей на платформу VMware Horizon — Unified Access Gateway, обратитесь в [службу поддержки этой платформы](mailto:support@vmware.com). Перед использованием единого входа необходимо создать и активировать пользователей.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

#### <a name="sp-initiated"></a>Инициация поставщиком услуг:

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в VMware Horizon — Unified Access Gateway, где можно инициировать поток входа.  

* Перейдите по URL-адресу для входа в VMware Horizon — Unified Access Gateway и инициируйте поток входа.

#### <a name="idp-initiated"></a>Вход, инициированный поставщиком удостоверений

* Выберите **Тестировать приложение** на портале Azure. Вы должны автоматически войти в VMware Horizon — Unified Access Gateway, для которого настроили единый вход. 

Вы можете также использовать Панель доступа корпорации Майкрософт для тестирования приложения в любом режиме. Щелкнув элемент VMware Horizon — Unified Access Gateway на Панели доступа, вы автоматически войдете в приложение VMware Horizon — Unified Access Gateway, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="next-steps"></a>Next Steps

После настройки VMware Horizon — Unified Access Gateway вы можете применить функцию управления сеансом, чтобы в реальном времени защищать конфиденциальные данные своей организации от кражи и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).