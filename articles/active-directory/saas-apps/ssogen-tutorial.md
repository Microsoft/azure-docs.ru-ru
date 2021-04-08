---
title: Учебник. Интеграция единого входа Azure Active Directory с приложением "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE" | Документация Майкрософт
description: Сведения о том, как настроить единый вход между Azure Active Directory и приложением "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE".
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/11/2021
ms.author: jeedes
ms.openlocfilehash: 9eeafaf0f5fbfaff9394ced0a0623f2fb462ed4d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101647002"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-ssogen---azure-ad-sso-gateway-for-oracle-e-business-suite---ebs-peoplesoft-and-jde"></a>Учебник. Интеграция единого входа Azure Active Directory с приложением "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE"

В этом учебнике описано, как интегрировать приложение "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE" с Azure Active Directory (Azure AD). Интеграция приложения "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE" с Azure Active Directory (Azure AD) предоставляет следующие возможности.

* С помощью AAD вы сможете контролировать доступ к приложению "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE"
* Пользователи смогут автоматически входить в приложение "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE" с учетными записями Azure Active Directory (AAD).
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка на приложение "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE" с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE поддерживает вход, инициированный **поставщиком услуг или поставщиком удостоверений**.

> [!NOTE]
> Идентификатор этого приложения — фиксированное строковое значение, поэтому в одном клиенте можно настроить только один экземпляр.

## <a name="add-ssogen---azure-ad-sso-gateway-for-oracle-e-business-suite---ebs-peoplesoft-and-jde-from-the-gallery"></a>Добавление приложения "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE" из коллекции

Чтобы настроить интеграцию приложения "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE", его нужно добавить из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** введите **Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE** в поле поиска.
1. Выберите **Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE** на панели результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-ssogen---azure-ad-sso-gateway-for-oracle-e-business-suite---ebs-peoplesoft-and-jde"></a>Настройка и проверка единого входа в Azure AD на стороне приложения "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE"

Настройте и проверьте единый вход AAD в приложение "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE", используя тестового пользователя с именем **B.Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в приложении "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE".

Чтобы настроить и проверить единый вход Azure AD для приложения "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE", выполните действия из следующих блоков:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в приложении "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE"](#configure-ssogen---azure-ad-sso-gateway-for-oracle-e-business-suite---ebs-peoplesoft-and-jde-sso)** необходима для настройки параметров единого входа на стороне приложения.
    1. **[Создание тестового пользователя в приложении "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE"](#create-ssogen---azure-ad-sso-gateway-for-oracle-e-business-suite---ebs-peoplesoft-and-jde-test-user)** требуется для того, чтобы в этом приложении существовал пользователь B. Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** введите значения следующих полей.

    В текстовом поле **URL-адрес ответа** введите URL-адрес в следующем формате: `https://<customer_name>.ssogen.com/ssogen/login?client_name=<customer_name>`.

1. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://<customer_name>.ssogen.com/ssogen/login?client_name=<customer_name>`.

    > [!NOTE]
    > Эти значения приведены для примера. Измените их на фактические значения URL-адреса ответа и URL-адреса входа. Чтобы получить эти значения, обратитесь в службу поддержки приложения [Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE](mailto:support@ssogen.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. Приложение "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE" ожидает проверочные утверждения SAML в определенном формате, который требует добавить сопоставления настраиваемых атрибутов в вашу конфигурацию атрибутов токена SAML. На следующем снимке экрана показан список атрибутов по умолчанию, в котором **nameidentifier** сопоставляется с **user.userprincipalname**. Приложение "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE" ожидает, что **nameidentifier** будет сопоставляться с **user.onpremisessamaccountname**, поэтому необходимо изменить сопоставление атрибутов, щелкнув значок **Изменить**.

    ![Изображение](common/edit-attribute.png)

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

В этом разделе описано, как включить единый вход Azure для пользователя B.Simon, предоставив этому пользователю доступ к приложению "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE".

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-ssogen---azure-ad-sso-gateway-for-oracle-e-business-suite---ebs-peoplesoft-and-jde-sso"></a>Настройка приложения "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE"

Чтобы настроить единый вход на стороне приложения **Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE**, изучите предложенную ниже документацию по регистрации единого входа для каждого отдельного приложения.

* Интеграция единого входа AAD для Oracle EBS: [https://www.ssogen.com/oracle-ebs-sso-ldap/](https://www.ssogen.com/oracle-ebs-sso-ldap/)
* Интеграция единого входа AAD для PeopleSoft: [https://www.ssogen.com/peoplesoft-sso/](https://www.ssogen.com/peoplesoft-sso/)
* Интеграция единого входа AAD для JD Edwards: [https://www.ssogen.com/oracle-jde-sso/](https://www.ssogen.com/oracle-jde-sso/)
* Интеграция единого входа AAD для Apache: [https://www.ssogen.com/apache-sso-authentication/](https://www.ssogen.com/apache-sso-authentication/)

### <a name="create-ssogen---azure-ad-sso-gateway-for-oracle-e-business-suite---ebs-peoplesoft-and-jde-test-user"></a>Создание тестового пользователя в приложении "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE"

После успешной проверки подлинности Azure AD отправляет пользовательскому приложению уникальный идентификатор пользователя (идентификатор имени).  Убедитесь, что этот уникальный идентификатор пользователя (идентификатор имени) соответствует записи для этого пользователя в приложении, например, FND_USER.USER_NAME в Oracle EBS 

Обратитесь по адресам [info@ssogen.com](mailto:info@ssogen.com) и [support@ssogen.com](mailto:support@ssogen.com), если у вас возникнут вопросы.

## <a name="test-sso"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

#### <a name="sp-initiated"></a>Инициация поставщиком услуг:

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в приложение "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE", где можно инициировать поток входа.  

* Перейдите по URL-адресу для входа в приложение "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE" и инициируйте поток входа.

#### <a name="idp-initiated"></a>Вход, инициированный поставщиком удостоверений

* Выберите **Тестировать приложение** на портале Azure, и вы автоматически войдете в приложение "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE", для которого настроен единый вход. 

Вы можете также использовать портал "Мои приложения" корпорации Майкрософт для тестирования приложения в любом режиме. Щелкнув плитку приложения "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE" на портале "Мои приложения", вы будете перенаправлены на страницу входа приложения для инициации потока входа (при настройке в режиме поставщика услуг) или автоматически войдете в приложение "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE", для которого настроен единый вход (при настройке в режиме поставщика удостоверений). Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Дальнейшие действия

После настройки приложения "Шлюз единого входа Azure AD SSOGEN для Oracle E-Business Suite (EBS), PeopleSoft и JDE" вы сможете применять средства управления сеансами, которые защищают от хищения и несанкционированного доступа к конфиденциальным данным вашей организации в режиме реального времени. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).