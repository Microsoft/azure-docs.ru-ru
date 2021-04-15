---
title: Руководство по интеграции Azure Active Directory с Five9 Plus Adapter (CTI, Contact Center Agents) | Документы Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Five9 Plus Adapter (CTI, Contact Center Agents).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/17/2021
ms.author: jeedes
ms.openlocfilehash: 85e953951d5368dc97312e7810f3c356bda7c6b6
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/02/2021
ms.locfileid: "106218725"
---
# <a name="tutorial-azure-active-directory-integration-with-five9-plus-adapter-cti-contact-center-agents"></a>Руководство по интеграции Azure Active Directory с Five9 Plus Adapter (CTI, Contact Center Agents)

В этом учебнике описано, как интегрировать Five9 Plus Adapter (CTI, Contact Center Agents) с Azure Active Directory (Azure AD). Интеграция Five9 Plus Adapter (CTI, Contact Center Agents) с Azure AD обеспечивает следующие возможности.

* Управление доступом к Five9 Plus Adapter (CTI, Contact Center Agents) с помощью Azure AD.
* Включение автоматического входа пользователей в Five9 Plus Adapter (CTI, Contact Center Agents) (единый вход) с использованием учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с Five9 Plus Adapter (CTI, Contact Center Agents), вам потребуется:

* Подписка Azure AD. (если у вас нет среды Azure AD, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/));
* подписка Five9 Plus Adapter (CTI, Contact Center Agents) с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Five9 Plus Adapter (CTI, Contact Center Agents) поддерживает единый вход, инициируемый **поставщиком удостоверений**.

> [!NOTE]
> Идентификатор этого приложения — фиксированное строковое значение, поэтому в одном клиенте можно настроить только один экземпляр.

## <a name="add-five9-plus-adapter-cti-contact-center-agents-from-the-gallery"></a>Добавление Five9 Plus Adapter (CTI, Contact Center Agents) из коллекции

Чтобы настроить интеграцию Five9 Plus Adapter (CTI, Contact Center Agents) с Azure AD, необходимо добавить Five9 Plus Adapter (CTI, Contact Center Agents) из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** введите **Five9 Plus Adapter (CTI, Contact Center Agents)** в поле поиска.
1. На панели результатов выберите **Five9 Plus Adapter (CTI, Contact Center Agents)** , а затем добавьте приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-five9-plus-adapter-cti-contact-center-agents"></a>Настройка и проверка единого входа Azure AD для Five9 Plus Adapter (CTI, Contact Center Agents)

Настройте и проверьте единый вход Azure AD в Five9 Plus Adapter (CTI, Contact Center Agents) с помощью тестового пользователя **B.Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в приложении Five9 Plus Adapter (CTI, Contact Center Agents).

Чтобы настроить и проверить единый вход Azure AD с Five9 Plus Adapter (CTI, Contact Center Agents), выполните следующие действия.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Five9 Plus Adapter (CTI, Contact Center Agents)](#configure-five9-plus-adapter-cti-contact-center-agents-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя Five9 Plus Adapter (CTI, Contact Center Agents)](#create-five9-plus-adapter-cti-contact-center-agents-test-user)** требуется для того, чтобы в Five9 Plus Adapter (CTI, Contact Center Agents) существовал пользователь B.Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Five9 Plus Adapter (CTI, Contact Center Agents)** найдите раздел **Управление** и щелкните **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. На странице **Настройка единого входа с помощью SAML** выполните следующие действия.

    а. В текстовом поле **Идентификатор** введите любой из следующих URL-адресов:
    
    |    Среда      |       URL-адрес      |
    | :-- | :-- |
    | Для "Five9 Plus Adapter for Microsoft Dynamics CRM" | `https://app.five9.com/appsvcs/saml/metadata/alias/msdc` |
    | Для "Five9 Plus Adapter for Zendesk" | `https://app.five9.com/appsvcs/saml/metadata/alias/zd` |
    | Для "Five9 Plus Adapter for Agent Desktop Toolkit" | `https://app.five9.com/appsvcs/saml/metadata/alias/adt` |

    b. В текстовом поле **URL-адрес ответа** введите любой из следующих URL-адресов:

    |      Среда     |      URL-адрес      |
    | :--                  | :--           |
    | Для "Five9 Plus Adapter for Microsoft Dynamics CRM" | `https://app.five9.com/appsvcs/saml/SSO/alias/msdc` |
    | Для "Five9 Plus Adapter for Zendesk" | `https://app.five9.com/appsvcs/saml/SSO/alias/zd` |
    | Для "Five9 Plus Adapter for Agent Desktop Toolkit" | `https://app.five9.com/appsvcs/saml/SSO/alias/adt` |

6. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Загрузить**, чтобы загрузить требуемый **сертификат (Base64)** из предложенных вариантов, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

7. Требуемые URL-адреса можно скопировать в разделе **Настройка Five9 Plus Adapter (CTI, Contact Center Agents)**.

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

В этом разделе описано, как разрешить пользователю B.Simon использовать единый вход Azure, предоставив этому пользователю доступ к Five9 Plus Adapter (CTI, Contact Center Agents).

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Five9 Plus Adapter (CTI, Contact Center Agents)**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-five9-plus-adapter-cti-contact-center-agents-sso"></a>Настройка единого входа в Five9 Plus Adapter (CTI, Contact Center Agents)

1. Чтобы настроить единый вход на стороне **Five9 Plus Adapter (CTI, Contact Center Agents)**, нужно отправить скачанный **сертификат (Base64)** и соответствующие URL-адреса, которые в скопировали, [группе поддержки Five9 Plus Adapter (CTI, Contact Center Agents)](https://www.five9.com/about/contact). Для дальнейшей настройки единого входа выполните следующие действия в соответствии с адаптером:

    a. Руководство по администрированию "Five9 Plus Adapter for Agent Desktop Toolkit": [https://webapps.five9.com/assets/files/for_customers/documentation/integrations/agent-desktop-toolkit/plus-agent-desktop-toolkit-administrators-guide.pdf](https://webapps.five9.com/assets/files/for_customers/documentation/integrations/agent-desktop-toolkit/plus-agent-desktop-toolkit-administrators-guide.pdf)
    
    b. Руководство по администрированию "Five9 Plus Adapter for Microsoft Dynamics CRM": [https://webapps.five9.com/assets/files/for_customers/documentation/integrations/microsoft/microsoft-administrators-guide.pdf](https://webapps.five9.com/assets/files/for_customers/documentation/integrations/microsoft/microsoft-administrators-guide.pdf)
    
    c. Руководство по администрированию "Five9 Plus Adapter for Zendesk": [https://webapps.five9.com/assets/files/for_customers/documentation/integrations/zendesk/zendesk-plus-administrators-guide.pdf](https://webapps.five9.com/assets/files/for_customers/documentation/integrations/zendesk/zendesk-plus-administrators-guide.pdf)

### <a name="create-five9-plus-adapter-cti-contact-center-agents-test-user"></a>Создание тестового пользователя Five9 Plus Adapter (CTI, Contact Center Agents)

В этом разделе вы создадите тестового пользователя по имени Britta Simon в Five9 Plus Adapter (CTI, Contact Center Agents). Чтобы добавить пользователей в Five9 Plus Adapter (CTI, Contact Center Agents), обратитесь в [службу поддержки Five9 Plus Adapter (CTI, Contact Center Agents)](https://www.five9.com/about/contact). Перед использованием единого входа необходимо создать и активировать пользователей. 

## <a name="test-sso"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов.

* Нажмите "Тестировать приложение" на портале Azure, и вы автоматически войдете в Five9 Plus Adapter (CTI, Contact Center Agents), для которого настроен единый вход.

* Вы можете использовать портал "Мои приложения" корпорации Майкрософт. Щелкнув элемент "Five9 Plus Adapter (CTI, Contact Center Agents)" на портале "Мои приложения", вы автоматически войдете в приложение Five9 Plus Adapter (CTI, Contact Center Agents), для которого настроили единый вход. Дополнительные сведения о портале "Мои приложения" см. в [этой статье](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="next-steps"></a>Дальнейшие действия

После настройки Five9 Plus Adapter (CTI, Contact Center Agents) вы можете применить функцию управления сеансом, которая защищает от хищения конфиденциальных данных вашей организации и несанкционированного доступа к ним в реальном времени. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).
