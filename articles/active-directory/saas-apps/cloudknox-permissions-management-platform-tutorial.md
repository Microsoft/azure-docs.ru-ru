---
title: Руководство по интеграции единого входа Azure Active Directory с CloudKnox Permissions Management Platform | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и CloudKnox Permissions Management Platform.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/27/2021
ms.author: jeedes
ms.openlocfilehash: 8c1b45d78d4fb61d239fef178cbcb9fa6efd9539
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101645694"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-cloudknox-permissions-management-platform"></a>Руководство по интеграции единого входа Azure Active Directory с CloudKnox Permissions Management Platform

В этом руководстве описано, как интегрировать CloudKnox Permissions Management Platform с Azure Active Directory (AAD). Интеграция CloudKnox Permissions Management Platform с Azure AD обеспечивает следующие возможности.

* С помощью Azure AD вы можете контролировать доступ к CloudKnox Permissions Management Platform.
* Вы можете включить автоматический вход пользователей в CloudKnox Permissions Management Platform с учетными записями AAD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка CloudKnox Permissions Management Platform с поддержкой единого входа

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Приложение CloudKnox Permissions Management Platform поддерживает единый вход, инициированный **поставщиком удостоверений**.

> [!NOTE]
> Идентификатор этого приложения — фиксированное строковое значение, поэтому в одном клиенте можно настроить только один экземпляр.

## <a name="adding-cloudknox-permissions-management-platform-from-the-gallery"></a>Добавление CloudKnox Permissions Management Platform из коллекции

Чтобы настроить интеграцию CloudKnox Permissions Management Platform с Azure AD, вам нужно добавить CloudKnox Permissions Management Platform из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **CloudKnox Permissions Management Platform**.
1. Выберите **CloudKnox Permissions Management Platform** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.


## <a name="configure-and-test-azure-ad-sso-for-cloudknox-permissions-management-platform"></a>Настройка и проверка единого входа Azure AD для CloudKnox Permissions Management Platform

Настройте и проверьте единый вход Azure AD в CloudKnox Permissions Management Platform с помощью тестового пользователя **B.Simon**. Чтобы обеспечить работу единого входа, нужно установить связь между пользователем Azure AD и соответствующим пользователем в CloudKnox Permissions Management Platform.

Чтобы настроить и проверить единый вход Azure AD в CloudKnox Permissions Management Platform, выполните следующие действия:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в CloudKnox Permissions Management Platform](#configure-cloudknox-permissions-management-platform-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя CloudKnox Permissions Management Platform](#create-cloudknox-permissions-management-platform-test-user)** требуется для того, чтобы в CloudKnox Permissions Management Platform существовал пользователь B.Simon, связанный с представлением того же пользователя в AAD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **CloudKnox Permissions Management Platform** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Настройка единого входа с помощью SAML** введите значения для следующих полей:

    В текстовом поле **URL-адрес ответа** введите URL-адрес в следующем формате: `https://app.cloudknox.io/saml/<ID>`.

    > [!NOTE]
    > Значение URL-адреса ответа приведено для примера. Вместо него нужно указать фактический URL-адрес ответа. Чтобы получить это значение, обратитесь в [службу поддержки CloudKnox Permissions Management Platform](mailto:support@cloudknox.io). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. Для приложения CloudKnox Permissions Management Platform утверждения SAML должны иметь определенный формат, поэтому следует добавить настраиваемые сопоставления атрибутов в вашу конфигурацию атрибутов токена SAML. На следующем снимке экрана показан список атрибутов по умолчанию.

    ![Изображение](common/default-attributes.png)

1. В дополнение к описанному выше приложение CloudKnox Permissions Management Platform ожидает несколько дополнительных атрибутов в ответе SAML, как показано ниже. Эти атрибуты также заранее заполнены, но вы можете изменить их в соответствии со своими требованиями.
    
    | Имя |  Атрибут источника|
    | --------------- | --------- |
    | First_Name | user.givenname |
    | Группы | user.groups |
    | Last_Name | user.surname |
    | Email_Address | user.mail |

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите элемент **XML метаданных федерации** и выберите **Скачать**, чтобы скачать сертификат и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

1. Скопируйте требуемый URL-адрес из раздела **Настройка CloudKnox Permissions Management Platform**.

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

В этом разделе описано, как предоставить пользователю B.Simon доступ к приложению CloudKnox Permissions Management Platform, чтобы он мог использовать единый вход Azure.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **CloudKnox Permissions Management Platform**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-cloudknox-permissions-management-platform-sso"></a>Настройка единого входа в CloudKnox Permissions Management Platform

Чтобы настроить единый вход на стороне **CloudKnox Permissions Management Platform**, нужно отправить скачанный **XML-файл метаданных федерации** и соответствующие URL-адреса, скопированные на портале Azure, в [службу поддержки CloudKnox Permissions Management Platform](mailto:support@cloudknox.io). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-cloudknox-permissions-management-platform-test-user"></a>Создание тестового пользователя CloudKnox Permissions Management Platform

В этом разделе вы создадите в CloudKnox Permissions Management Platform пользователя с именем B.Simon. Чтобы добавить пользователей на платформу CloudKnox Permissions Management Platform, обратитесь в [службу поддержки CloudKnox Permissions Management Platform](mailto:support@cloudknox.io). Перед использованием единого входа необходимо создать и активировать пользователей.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов.

* На портале Azure выберите "Тестировать приложение", и вы автоматически войдете в приложение CloudKnox Permissions Management Platform, для которого настроен единый вход.

* Вы можете использовать портал "Мои приложения" корпорации Майкрософт. Щелкнув плитку CloudKnox Permissions Management Platform на портале "Мои приложения", вы автоматически войдете в приложение CloudKnox Permissions Management Platform, для которого настроен единый вход. Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Дальнейшие действия

После настройки CloudKnox Permissions Management Platform вы можете применить функцию управления сеансами, которая в реальном времени защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).