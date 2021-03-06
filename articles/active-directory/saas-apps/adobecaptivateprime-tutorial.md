---
title: Руководство по интеграции Azure Active Directory с Adobe Captivate Prime | Документы Майкрософт
description: Сведения о том, как настроить единый вход между Azure Active Directory и Adobe Captivate Prime.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/31/2021
ms.author: jeedes
ms.openlocfilehash: 87ce580e16d1ca3b90eb66562f06828d775b09ea
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106285690"
---
# <a name="tutorial-azure-active-directory-integration-with-adobe-captivate-prime"></a>Руководство по интеграции Azure Active Directory с Adobe Captivate Prime

В этом руководстве описано, как интегрировать приложение Adobe Captivate Prime с Azure Active Directory (Azure AD). Интеграция Adobe Captivate Prime с Azure AD обеспечивает следующие возможности:

* Контроль доступа к Adobe Captivate Prime с помощью Azure AD.
* Автоматический вход пользователей в Adobe Captivate Prime с учетными записями Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* подписка Adobe Captivate Prime с поддержкой единого входа (SSO).

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Adobe Captivate Prime поддерживает единый вход, инициированный **поставщиком удостоверений**.

> [!NOTE]
> Идентификатор этого приложения — фиксированное строковое значение, поэтому в одном клиенте можно настроить только один экземпляр.

## <a name="add-adobe-captivate-prime-from-the-gallery"></a>Добавление Adobe Captivate Prime из коллекции

Чтобы настроить интеграцию Adobe Captivate Prime с Azure AD, нужно добавить Adobe Captivate Prime из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Adobe Captivate Prime**.
1. Выберите **Adobe Captivate Prime** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-adobe-captivate-prime"></a>Настройка и проверка единого входа Azure AD для Adobe Captivate Prime

Настройте и проверьте единый вход Azure AD в Adobe Captivate Prime с помощью тестового пользователя **B.Simon**. Чтобы обеспечить работу единого входа, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Adobe Captivate Prime.

Чтобы настроить и проверить единый вход Azure AD в Adobe Captivate Prime, выполните следующие действия:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Adobe Captivate Prime](#configure-adobe-captivate-prime-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя Adobe Captivate Prime](#create-adobe-captivate-prime-test-user)** требуется для того, чтобы в Adobe Captivate Prime существовал пользователь B.Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Adobe Captivate Prime** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. На странице **Настройка единого входа с помощью SAML** выполните следующие действия.

    а. В текстовом поле **Идентификатор** введите URL-адрес `https://captivateprime.adobe.com`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес `https://captivateprime.adobe.com/saml/SSO`

5. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** щелкните **Скачать**, чтобы скачать нужный вам **XML метаданных федерации**, и сохраните его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

6. Скопируйте требуемый URL-адрес из раздела **Настройка Adobe Captivate Prime**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

7. Откройте вкладку **Свойства**, скопируйте **URL-адрес пользовательского доступа** и вставьте его в Блокнот.

    ![Ссылка для доступа пользователя](./media/adobecaptivateprime-tutorial/adobe.png)

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

В этом разделе описано, как включить единый вход в Azure для пользователя B.Simon, предоставив этому пользователю доступ к Adobe Captivate Prime.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Adobe Captivate Prime**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-adobe-captivate-prime-sso"></a>Настройка единого входа для Adobe Captivate Prime

Чтобы настроить единый вход на стороне **Adobe Captivate Prime**, нужно передать скачанный файл **XML метаданных федерации**, скопированный **URL-адрес пользовательского доступа** и скопированные соответствующие URL-адреса с портала Azure в [группу поддержки Adobe Captivate Prime](mailto:captivateprimesupport@adobe.com). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-adobe-captivate-prime-test-user"></a>Создание тестового пользователя Adobe Captivate Prime

В этом разделе описано, как создать пользователя Britta Simon в приложении Adobe Captivate Prime. Обратитесь в [группу поддержки Adobe Captivate Prime](mailto:captivateprimesupport@adobe.com), чтобы добавить пользователей на платформу Adobe Captivate Prime. Перед использованием единого входа необходимо создать и активировать пользователей.

## <a name="test-sso"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов.

* На портале Azure выберите "Тестировать приложение", и вы автоматически войдете в приложение Adobe Captivate Prime, для которого настроили единый вход.

* Вы можете использовать портал "Мои приложения" корпорации Майкрософт. Щелкнув элемент "Adobe Captivate Prime" в разделе "Мои приложения", вы автоматически войдете в приложение Adobe Captivate Prime, для которого настроили единый вход. Дополнительные сведения о портале "Мои приложения" см. в [этой статье](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="next-steps"></a>Дальнейшие действия

После настройки Adobe Captivate Prime вы сможете применить функцию управления сеансами, чтобы в реальном времени защищать конфиденциальные данные своей организации от мошенничества и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).
