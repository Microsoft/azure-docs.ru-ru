---
title: Руководство. Интеграция единого входа Azure Active Directory с CylancePROTECT | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и CylancePROTECT.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/27/2020
ms.author: jeedes
ms.openlocfilehash: 2fd21731513d0b32a96a74a822e38075ad1d8eb2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92454976"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-cylanceprotect"></a>Руководство. Интеграция единого входа Azure Active Directory с CylancePROTECT

В этом руководстве вы узнаете, как интегрировать CylancePROTECT с Azure Active Directory (Azure AD). Интеграция CylancePROTECT с Azure AD обеспечивает следующие возможности.

* Контроль доступа к CylancePROTECT с помощью Azure AD.
* Включение автоматического входа пользователей в CylancePROTECT с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

Чтобы узнать больше об интеграции приложений SaaS с Azure AD, прочитайте статью [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка CylancePROTECT с поддержкой единого входа (SSO).

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* CylancePROTECT поддерживает единый вход инициированного **пакета обновления**.

## <a name="adding-cylanceprotect-from-the-gallery"></a>Добавление CylancePROTECT из коллекции.

Чтобы настроить интеграцию CylancePROTECT с Azure AD, необходимо добавить CylancePROTECT из коллекции в список управляемых приложений SaaS.

1. Войдите на [портал Azure](https://portal.azure.com) с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **CylancePROTECT**.
1. Выберите **CylancePROTECT** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-single-sign-on-for-cylanceprotect"></a>Настройка и проверка единого входа Azure AD для CylancePROTECT

Настройте и проверьте единый вход Azure AD в CylancePROTECT с помощью тестового пользователя **B.Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем на платформе CylancePROTECT.

Чтобы настроить и проверить единый вход AAD в CylancePROTECT, выполните действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    * **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    * **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в CylancePROTECT](#configure-cylanceprotect-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    * **[Создание тестового пользователя CylancePROTECT](#create-cylanceprotect-test-user)** требуется, чтобы в CylancePROTECT существовал пользователь B.Simon, связанный с представлением пользователя в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **CylancePROTECT** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Настройка единого входа с помощью SAML** введите значения для следующих полей:

    а. В текстовом поле **Идентификатор** введите URL-адрес: .
    
    | Регион | Значение URL-адреса |
    |----------|---------|
    | Северо-восток Азиатско-Тихоокеанского региона (APNE1)| `https://login-apne1.cylance.com/EnterpriseLogin/ConsumeSaml`|
    | Юго-восток Азиатско-Тихоокеанского региона (AU) | `https://login-au.cylance.com/EnterpriseLogin/ConsumeSaml` |
    | Центральная Европа (EUC1)|`https://login-euc1.cylance.com/EnterpriseLogin/ConsumeSaml`|
    | Северная Америка|`https://login.cylance.com/EnterpriseLogin/ConsumeSaml`|
    | Южная Америка (SAE1)|`https://login-sae1.cylance.com/EnterpriseLogin/ConsumeSaml`|

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес .
    
    | Регион | Значение URL-адреса |
    |----------|---------|
    | Северо-восток Азиатско-Тихоокеанского региона (APNE1)|`https://login-apne1.cylance.com/EnterpriseLogin/ConsumeSaml`|
    | Юго-восток Азиатско-Тихоокеанского региона (AU)|`https://login-au.cylance.com/EnterpriseLogin/ConsumeSaml`|
    | Центральная Европа (EUC1)|`https://login-euc1.cylance.com/EnterpriseLogin/ConsumeSaml`|
    | Северная Америка|`https://login.cylance.com/EnterpriseLogin/ConsumeSaml`|
    | Южная Америка (SAE1)|`https://login-sae1.cylance.com/EnterpriseLogin/ConsumeSaml`|

1. Приложение CylancePROTECT ожидает проверочные утверждения SAML в определенном формате, который требует добавить настраиваемые сопоставления атрибутов в конфигурацию атрибутов токена SAML. На следующем снимке экрана показан список атрибутов по умолчанию, в котором **nameidentifier** сопоставляется с **user.userprincipalname**. Приложение CylancePROTECT ожидает, что **nameidentifier** будет сопоставляться с **user.mail** и удаляет все остальные утверждения, поэтому щелкните значок **Изменить** и измените сопоставление атрибутов.

    ![Изображение](common/edit-attribute.png)

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите пункт **Сертификат (Base64)** и щелкните **Скачать**, чтобы скачать сертификат. Сохраните этот сертификат на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

1. Требуемый URL-адрес можно скопировать из раздела **Настройка CylancePROTECT**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

> [!NOTE]
> Откройте скачанный расшифрованный сертификат Base64 в текстовом редакторе и скопируйте только текст между **ОТКРЫВАЮЩИМ** и **ЗАКРЫВАЮЩИМ** тегами, чтобы вставить их на портал администрирования Cylance.

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

В этом разделе описано, как включить единый вход в Azure для пользователя B.Simon, предоставив этому пользователю доступ к CylancePROTECT.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **CylancePROTECT**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.

   ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Ссылка "Добавить пользователя"](common/add-assign-user.png)

1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор роли** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-cylanceprotect-sso"></a>Настройка единого входа в CylancePROTECT

Чтобы настроить единый вход на стороне **CylancePROTECT**, нужно отправить скачанный **сертификат (Base64)** и соответствующие URL-адреса, скопированные на портале Azure, в [службу поддержки CylancePROTECT](https://www.cylance.com/en-us/resources/support/support-overview.html). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах. Дополнительные сведения см. в документации по Cylance: [https://support.cylance.com/s/](https://support.cylance.com/s/).

### <a name="create-cylanceprotect-test-user"></a>Создание тестового пользователя CylancePROTECT

Из этого раздела вы узнаете, как создать пользователя Britta Simon в приложении CylancePROTECT. Чтобы добавить пользователей на платформе CylancePROTECT, обратитесь к администратору консоли. Владелец учетной записи Azure Active Directory получит по электронной почте сообщение со ссылкой для активации учетной записи.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку "CylancePROTECT" на панели доступа, вы автоматически войдете в приложение CylancePROTECT, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)

- [Пробное использование CylancePROTECT с Azure AD](https://aad.portal.azure.com/)