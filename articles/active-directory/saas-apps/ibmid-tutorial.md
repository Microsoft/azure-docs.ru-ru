---
title: Руководство. Интеграция единого входа Azure Active Directory с IBMid | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и IBMid.
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
ms.openlocfilehash: 8c1c0131a2771c843cbd636c4a80a8cb7e4d6c08
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101649405"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-ibmid"></a>Руководство. Интеграция единого входа Azure Active Directory с IBMid

В этом руководстве описано, как интегрировать IBMid с Azure Active Directory (Azure AD). Интеграция IBMid с Azure AD обеспечивает следующие возможности:

* Контроль доступа к IBMid с помощью Azure AD.
* Автоматический вход пользователей в IBMid с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка IBMid с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* IBMid поддерживает единый вход, инициированный **поставщиком службы и поставщиком удостоверений**.
* IBMid поддерживает **JIT-подготовку** пользователей.

## <a name="adding-ibmid-from-the-gallery"></a>Добавление IBMid из коллекции

Чтобы настроить интеграцию IBMid с Azure AD, добавьте IBMid из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **IBMid**.
1. Выберите **IBMid** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.


## <a name="configure-and-test-azure-ad-sso-for-ibmid"></a>Настройка и проверка единого входа Azure AD для IBMid

Настройте и проверьте единый вход Azure AD в IBMid с помощью тестового пользователя **B. Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в IBMid.

Чтобы настроить и проверить единый вход Azure AD в IBMid, выполните следующие действия:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в IBMid](#configure-ibmid-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя в IBMid](#create-ibmid-test-user)** нужно для того, чтобы в IBMid существовал пользователь B. Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **IBMid** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **Идентификатор** введите любой из следующих URL-адресов:

    | Идентификатор |
    | ---------- |
    | `https://idaas.iam.ibm.com/idaas/mtfim/sps/idaas/saml20` |
    | `https://ibmlogin.ice.ibmcloud.com/saml/sps/saml20sp/saml20` |
    | `https://prepiam.ice.ibmcloud.com/saml/sps/saml20sp/saml20` |
    |

    а. В текстовом поле **URL-адрес ответа** введите любой из следующих URL-адресов:

    | URL-адрес ответа |
    | ---------- |
    | `https://idaas.iam.ibm.com/idaas/mtfim/sps/idaas/saml20/login` |
    | `https://login.ibm.com/saml/sps/saml20sp/saml20/login` |
    | `https://prepiam.ice.ibmcloud.com/saml/sps/saml20sp/saml20/login` |
    |

1. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    В текстовом поле **URL-адрес входа** введите URL-адрес: `https://myibm.ibm.com/`.


1. Выберите команду **Сохранить**.

1. Для приложения IBMid утверждения SAML должны иметь определенный формат, поэтому следует добавить сопоставления настраиваемых атрибутов в вашу конфигурацию атрибутов токена SAML. На следующем снимке экрана показан список атрибутов по умолчанию.

    ![Изображение](common/default-attributes.png)

1. В дополнение к описанному выше приложение IBMid ожидает в ответе SAML несколько дополнительных атрибутов, которые показаны ниже. Эти атрибуты также заранее заполнены, но вы можете изменить их в соответствии со своими требованиями.
    
    | Имя |  Исходный атрибут|
    | ----------- | --------- |
    | country | user.country |
    | firstName | user.givenname |
    | lastName | user.surname |
    | emailAddress | user.mail |


1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите элемент **XML метаданных федерации** и выберите **Скачать**, чтобы скачать сертификат и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

1. Требуемые URL-адреса можно скопировать в разделе **Настройка IBMid**.

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

В этом разделе описано, как включить единый вход в Azure для пользователя B. Simon, предоставив этому пользователю доступ к IBMid.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **IBMid**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-ibmid-sso"></a>Настройка единого входа в IBMid

Чтобы настроить единый вход на стороне **IBMid**, нужно отправить скачанный **XML-файл метаданных федерации** и соответствующие URL-адреса, скопированные на портале Azure, в [службу поддержки IBMid](mailto:ibmidfd@us.ibm.com). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-ibmid-test-user"></a>Создание тестового пользователя в IBMid

В этом разделе описано, как создать в IBMid пользователя с именем Britta Simon. Приложение IBMid поддерживает JIT-подготовку пользователей, которая включена по умолчанию. В этом разделе никакие действия с вашей стороны не требуются. Если пользователь не существует в IBMid, он создается после проверки подлинности.

## <a name="test-sso"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

#### <a name="sp-initiated"></a>Инициация поставщиком услуг:

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в IBMid, где можно инициировать поток входа.  

* Перейдите по URL-адресу для входа в IBMid и инициируйте поток входа.

#### <a name="idp-initiated"></a>Вход, инициированный поставщиком удостоверений

* Выберите **Тестировать приложение** на портале Azure, и вы автоматически войдете в приложение IBMid, для которого настроен единый вход. 

Вы можете также использовать портал "Мои приложения" корпорации Майкрософт для тестирования приложения в любом режиме. Щелкнув плитку IBMid на портале "Мои приложения", вы будете перенаправлены на страницу входа приложения для инициации потока входа (при настройке в режиме поставщика службы) или автоматически войдете в приложение IBMid, для которого настроен единый вход (при настройке в режиме поставщика удостоверений). Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Дальнейшие действия

После настройки IBMid вы можете применить управление сеансами, которое в реальном времени защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).