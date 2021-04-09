---
title: Руководство по Интеграция Azure Active Directory с Palo Alto Networks (Captive Portal) | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Palo Alto Networks (Captive Portal).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/10/2020
ms.author: jeedes
ms.openlocfilehash: 1b9c9182b8ffc6e9ea2e05fd4863783f111f81dd
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98727154"
---
# <a name="tutorial-azure-active-directory-integration-with-palo-alto-networks-captive-portal"></a>Руководство по Интеграция Azure Active Directory с Palo Alto Networks (Captive Portal)

В этом руководстве описано, как интегрировать Palo Alto Networks (Captive Portal) с Azure Active Directory (Azure AD).
Интеграция Palo Alto Networks (Captive Portal) с Azure AD обеспечивает следующие преимущества.

* Контроль доступа к Palo Alto Networks (Captive Portal) с помощью Azure AD.
* Автоматический вход пользователей в Palo Alto Networks (Captive Portal) (единый вход) с помощью учетных записей Azure AD.
* Вы можете управлять учетными записями централизованно на портале Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы интегрировать Azure AD с Palo Alto Networks (Captive Portal), вам потребуется:

* Подписка Azure Active Directory. Если у вас нет Azure AD, вы можете получить [пробную версию на один месяц](https://azure.microsoft.com/pricing/free-trial/).
* Подписка Palo Alto Networks (Captive Portal) с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Palo Alto Networks (Captive Portal) поддерживает единый вход, инициированный **поставщиком удостоверений**.
* Palo Alto Networks (Captive Portal) поддерживает **JIT**-подготовку пользователей.

## <a name="adding-palo-alto-networks-captive-portal-from-the-gallery"></a>Добавление Palo Alto Networks (Captive Portal) из коллекции

Чтобы настроить интеграцию Palo Alto Networks (Captive Portal) с Azure AD, необходимо добавить это приложение из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** введите **Palo Alto Networks (Captive Portal)** в поле поиска.
1. Выберите **Palo Alto Networks (Captive Portal)** на панели результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso"></a>Настройка и проверка единого входа Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в Palo Alto Networks (Captive Portal) с использованием тестового пользователя **B. Simon**.
Чтобы обеспечить правильную работу единого входа, необходимо установить связь между пользователем Azure AD и соответствующим пользователем Palo Alto Networks (Captive Portal).

Чтобы настроить и проверить единый вход Azure AD в Palo Alto Networks (Captive Portal), сделайте следующее.

1. **[Настройте единый вход Azure AD](#configure-azure-ad-sso)** , чтобы пользователь мог использовать эту функцию.
    * **[Создайте тестового пользователя Azure AD](#create-an-azure-ad-test-user)** , чтобы проверить работу единого входа Azure AD от имени пользователя B. Simon.
    * **[Назначьте тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** , чтобы настроить пользователя B. Simon для использования единого входа Azure AD.
2. **[Настройте единый вход в Palo Alto Networks (Captive Portal)](#configure-palo-alto-networks-captive-portal-sso)** , чтобы настроить параметры единого входа приложения.
    * **[Создайте тестового пользователя Palo Alto Networks (Captive Portal)](#create-a-palo-alto-networks-captive-portal-test-user)** , чтобы в Palo Alto Networks (Captive Portal) существовал пользователь B. Simon, связанный с одноименным пользователем в Azure AD.
3. **[Проверьте единый вход](#test-sso)** , чтобы убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Palo Alto Networks (Captive Portal)** найдите раздел **Управление** и щелкните **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В области **Базовая конфигурация SAML** сделайте следующее.

   1. В поле **Идентификатор** введите URL-адрес по шаблону `https://<customer_firewall_host_name>/SAML20/SP`.

   2. В поле **URL-адрес ответа** введите URL-адрес по шаблону `https://<customer_firewall_host_name>/SAML20/SP/ACS`.

      > [!NOTE]
      > На этом шаге необходимо заменить значения заполнителей фактическим идентификатором и URL-адресами ответа. Чтобы получить эти значения, обратитесь в [группу поддержки клиентов Palo Alto Networks (Captive Portal)](https://support.paloaltonetworks.com/support).

5. В разделе **Сертификат подписи SAML** рядом с **XML метаданных федерации** выберите **Загрузить**. Сохраните скачанный файл на компьютере.

    ![Ссылка для скачивания XML-файла метаданных федерации](common/metadataxml.png)

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

В этом разделе описано, как предоставить пользователю B. Simon доступ к Palo Alto Networks (Captive Portal), чтобы он мог использовать единый вход Azure.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Palo Alto Networks (Captive Portal)** .
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-palo-alto-networks-captive-portal-sso"></a>Настройка единого входа для Palo Alto Networks (Captive Portal)

Настройте единый вход для Palo Alto Networks (Captive Portal), выполнив приведенные ниже действия.

1. В другом окне браузера войдите на веб-сайт Palo Alto Networks в качестве администратора.

2. Выберите вкладку **Device** (Устройство).

    ![Вкладка "Устройство" на веб-сайте Palo Alto Networks](./media/paloaltonetworks-captiveportal-tutorial/tutorial_paloaltoadmin_admin1.png)

3. В меню выберите **SAML Identity Provider** (Поставщик удостоверений SAML), а затем выберите **Импортировать**.

    ![Кнопка "Импортировать"](./media/paloaltonetworks-captiveportal-tutorial/tutorial_paloaltoadmin_admin2.png)

4. В диалоговом окне **SAML Identity Provider Server Profile Import** (Импорт профиля сервера поставщика удостоверений SAML) выполните указанные ниже действия.

    ![Настройка единого входа Palo Alto Networks](./media/paloaltonetworks-captiveportal-tutorial/tutorial_paloaltoadmin_admin3.png)

    1. В поле **Имя профиля** введите имя, например **AzureAD-CaptivePortal**.
    
    2. Рядом с параметром **Identity Provider Metadata** (Метаданные поставщика удостоверений) выберите **Обзор**. Выберите файл Metadata.xml, скачанный с портала Azure.
    
    3. Щелкните **ОК**.

### <a name="create-a-palo-alto-networks-captive-portal-test-user"></a>Создание тестового пользователя в Palo Alto Networks (Captive Portal)

Затем создайте пользователя с именем *Britta Simon* в Palo Alto Networks (Captive Portal). Palo Alto Networks (Captive Portal) поддерживает JIT-подготовку пользователей. Эта функция включена по умолчанию. В этом разделе не нужно выполнять никакие задачи. Если пользователь еще не существует в Palo Alto Networks (Captive Portal), он создается после проверки подлинности.

> [!NOTE]
> Чтобы создать пользователя вручную, обратитесь в [группу поддержки клиентов Palo Alto Networks (Captive Portal)](https://support.paloaltonetworks.com/support).

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов.

* На портале Azure выберите "Тестировать приложение", и вы автоматически войдете в приложение Palo Alto Networks (Captive Portal), для которого настроен единый вход.

* Вы можете использовать портал "Мои приложения" корпорации Майкрософт. Щелкнув плитку Palo Alto Networks (Captive Portal) на портале "Мои приложения", вы автоматически войдете в приложение Palo Alto Networks (Captive Portal), для которого настроили единый вход. Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Дальнейшие действия

После настройки Palo Alto Networks (Captive Portal) вы можете применить функцию управления сеансом, которая в реальном времени защищает конфиденциальные данные вашей организации от кражи и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).