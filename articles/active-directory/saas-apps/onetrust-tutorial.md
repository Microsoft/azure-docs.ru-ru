---
title: Руководство по интеграции Azure Active Directory с OneTrust Privacy Management Software | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и OneTrust Privacy Management Software.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/13/2021
ms.author: jeedes
ms.openlocfilehash: f0a145b0eb9dd9dbed0927ce825a21d8f47c48ab
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101648447"
---
# <a name="tutorial-azure-active-directory-integration-with-onetrust-privacy-management-software"></a>Руководство по интеграции Azure Active Directory с OneTrust Privacy Management Software

В этом руководстве описано, как интегрировать OneTrust Privacy Management Software с Azure Active Directory (Azure AD). Интеграция OneTrust Privacy Management Software с Azure AD обеспечивает следующие возможности:

* Контроль доступа к OneTrust Privacy Management Software с помощью Azure AD.
* Автоматический вход пользователей в OneTrust Privacy Management Software с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с OneTrust Privacy Management Software, вам потребуется следующее:

* подписка Azure AD; (если у вас нет среды Azure AD, вы можете получить пробную версию на один месяц по [этой ссылке](https://azure.microsoft.com/pricing/free-trial/));
* подписка OneTrust Privacy Management Software с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* OneTrust Privacy Management Software поддерживает единый вход, инициированный **поставщиком услуг** и **поставщиком удостоверений**.

* Приложение OneTrust Privacy Management Software поддерживает **JIT**-подготовку пользователей.

> [!NOTE]
> Идентификатор этого приложения — фиксированное строковое значение, поэтому в одном клиенте можно настроить только один экземпляр.

## <a name="add-onetrust-privacy-management-software-from-the-gallery"></a>Добавление OneTrust Privacy Management Software из коллекции

Чтобы настроить интеграцию OneTrust Privacy Management Software с Azure AD, необходимо добавить OneTrust Privacy Management Software из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **OneTrust Privacy Management Software**.
1. Выберите **OneTrust Privacy Management Software** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-onetrust-privacy-management-software"></a>Настройка и проверка единого входа Azure AD для OneTrust Privacy Management Software

Настройте и проверьте единый вход Azure AD в OneTrust Privacy Management Software с помощью тестового пользователя **B. Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в OneTrust Privacy Management Software.

Чтобы настроить и проверить единый вход Azure AD в OneTrust Privacy Management Software, сделайте следующее.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в OneTrust Privacy Management Software](#configure-onetrust-privacy-management-software-sso)** требуется, чтобы определить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя OneTrust Privacy Management Software](#create-onetrust-privacy-management-software-test-user)** требуется для того, чтобы в OneTrust Privacy Management Software существовал пользователь B. Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

### <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

В этом разделе описано, как включить единый вход Azure AD на портале Azure.
 
1. На портале Azure на странице интеграции с приложением **OneTrust Privacy Management Software** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить эти параметры.

    ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** выполните следующие действия.

    а. В текстовом поле **Идентификатор** введите URL-адрес `https://www.onetrust.com/saml2`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<subdomain>.onetrust.com/auth/consumerservice`.

5. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

     В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://<subdomain>.onetrust.com/auth/login`.

    > [!NOTE]
    > Эти значения приведены для примера. Замените их фактическими значениями URL-адреса ответа и URL-адреса входа. Обратитесь к [группе поддержки клиентов OneTrust Privacy Management Software](mailto:support@onetrust.com), чтобы получить эти значения. Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

6. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

7. Требуемые URL-адреса можно скопировать из раздела **Настройка OneTrust Privacy Management Software**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD 

В этом разделе описано, как на портале Azure создать тестового пользователя с именем B.Simon.
1. Слева на портале Azure выберите **Azure Active Directory** > **Пользователи** > **Все пользователи**.
1. В верхней части экрана выберите **Новый пользователь**.
1. В разделе **Свойства пользователя** выполните следующие действия.
   1. В поле **Имя** введите `B.Simon`.  
   1. В поле **Имя пользователя** введите username@companydomain.extension. Например, `B.Simon@contoso.com`.
   1. Установите флажок **Показать пароль** и запишите значение пароля.
   1. Нажмите кнопку **создания**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как включить единый вход Azure для пользователя B. Simon и предоставить этому пользователю доступ к OneTrust Privacy Management Software.

1. На портале Azure выберите **Корпоративные приложения** > **Все приложения**.
1. Из списка приложений выберите **OneTrust Privacy Management Software**.
1. На странице сводных сведений о приложении откройте раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**. Затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** из списка пользователей. Затем нажмите кнопку **Выбрать** в нижней части экрана.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** выберите **Назначить**.

### <a name="configure-onetrust-privacy-management-software-sso"></a>Настройка единого входа OneTrust Privacy Management Software

Чтобы настроить единый вход на стороне **OneTrust Privacy Management Software**, нужно отправить скачанный файл **XML метаданных федерации** и соответствующие URL-адреса, скопированные на портале Azure, в [группу поддержки OneTrust Privacy Management Software](mailto:support@onetrust.com). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-onetrust-privacy-management-software-test-user"></a>Создание тестового пользователя OneTrust Privacy Management Software

В этом разделе вы создадите в OneTrust Privacy Management Software пользователя с именем Britta Simon. Приложение OneTrust Privacy Management Software поддерживает JIT-подготовку пользователей. Эта функция включена по умолчанию. В этом разделе никакие действия с вашей стороны не требуются. Если пользователь еще не существует в OneTrust Privacy Management Software, он создается после проверки подлинности.

>[!Note]
>Если необходимо вручную создать пользователя, обратитесь к [группе поддержки OneTrust Privacy Management Software](mailto:support@onetrust.com).

### <a name="test-sso"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

#### <a name="sp-initiated"></a>Инициация поставщиком услуг:

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в OneTrust Privacy Management Software, где можно инициировать поток входа.  
 
* Перейдите непосредственно по URL-адресу входа в OneTrust Privacy Management Software и запустите поток входа.

#### <a name="idp-initiated"></a>Вход, инициированный поставщиком удостоверений

* На портале Azure выберите **Тестировать приложение**, и вы автоматически войдете в приложение OneTrust Privacy Management Software, для которого настроен единый вход. 

Вы можете также использовать портал "Мои приложения" корпорации Майкрософт для тестирования приложения в любом режиме. Щелкнув плитку OneTrust Privacy Management Software на портале "Мои приложения", вы автоматически перейдете на страницу входа приложения для инициации потока входа (при настройке в режиме поставщика услуг) или в приложение OneTrust Privacy Management Software, для которого настроен единый вход (при настройке в режиме поставщика удостоверений). Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Дальнейшие действия

После настройки OneTrust Privacy Management Software вы можете применить функцию управления сеансами, которая защищает конфиденциальные данные вашей организации от хищения и несанкционированного доступа в реальном времени. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).