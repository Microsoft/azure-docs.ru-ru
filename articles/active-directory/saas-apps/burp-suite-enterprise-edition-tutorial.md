---
title: Руководство по интеграции единого входа Azure Active Directory в Burp Suite Enterprise Edition | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Burp Suite Enterprise Edition.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/09/2020
ms.author: jeedes
ms.openlocfilehash: f071159b8e1a25776fad5e662f4ea18573764a89
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/12/2020
ms.locfileid: "97364498"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-burp-suite-enterprise-edition"></a>Руководство по интеграции единого входа Azure Active Directory в Burp Suite Enterprise Edition

В этом учебнике описано, как интегрировать Burp Suite Enterprise Edition с Azure Active Directory (Azure AD). Интеграция Burp Suite Enterprise Edition с Azure AD обеспечивает следующие возможности:

* Контроль доступа к Burp Suite Enterprise Edition с помощью Azure AD.
* Автоматический вход пользователей в Burp Suite Enterprise Edition с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Burp Suite Enterprise Edition с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.


* Приложение Burp Suite Enterprise Edition поддерживает единый вход, инициированный **поставщиком удостоверений**.

* Приложение Burp Suite Enterprise Edition поддерживает **JIT**-подготовку пользователей.


## <a name="adding-burp-suite-enterprise-edition-from-the-gallery"></a>Добавление Burp Suite Enterprise Edition из коллекции

Чтобы настроить интеграцию приложения Burp Suite Enterprise Edition с Azure AD, нужно добавить Burp Suite Enterprise Edition из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Burp Suite Enterprise Edition**.
1. Выберите **Burp Suite Enterprise Edition** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.


## <a name="configure-and-test-azure-ad-sso-for-burp-suite-enterprise-edition"></a>Настройка и проверка единого входа Azure AD для Burp Suite Enterprise Edition

Настройте и проверьте единый вход Azure AD в приложение Burp Suite Enterprise Edition с помощью тестового пользователя **B. Simon**. Чтобы обеспечить работу единого входа, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Burp Suite Enterprise Edition.

Чтобы настроить и проверить единый вход Azure AD в Burp Suite Enterprise Edition, сделайте следующее:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Burp Suite Enterprise Edition](#configure-burp-suite-enterprise-edition-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя Burp Suite Enterprise Edition](#create-burp-suite-enterprise-edition-test-user)** требуется для того, чтобы в Burp Suite Enterprise Edition существовал пользователь B. Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Burp Suite Enterprise Edition** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Настройка единого входа с помощью SAML** введите значения для следующих полей:

    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `https://<BURPSUITEDOMAIN:PORT>/saml`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<BURPSUITEDOMAIN:PORT>/api-internal/saml/acs`.

    > [!NOTE]
    > Эти значения приведены для примера. Измените их на фактические значения идентификатора и URL-адреса ответа. Чтобы получить эти значения, обратитесь в [группу поддержки клиентов Burp Suite Enterprise Edition](mailto:support@portswigger.net). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. Приложение Burp Suite Enterprise Edition предусматривает использование проверочных утверждений SAML в определенном формате, поэтому вам нужно добавить сопоставления настраиваемых атрибутов в конфигурацию атрибутов токена SAML. На следующем снимке экрана показан список атрибутов по умолчанию.

    ![Изображение](common/edit-attribute.png)

1. Помимо перечисленного выше, приложение Burp Suite Enterprise Edition ожидает несколько дополнительных атрибутов в ответе SAML, которые представлены ниже. Эти атрибуты также заранее заполнены, но вы можете изменить их в соответствии со своими требованиями.


    | Имя | Исходный атрибут|
    | ---------------| --------------- |    
    | Группа | user.groups |

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите пункт **Сертификат (Base64)** и щелкните **Скачать**, чтобы скачать сертификат. Сохраните этот сертификат на компьютере.

    ![Ссылка для скачивания сертификата](common/certificatebase64.png)

1. Скопируйте требуемый URL-адрес из раздела **Настройка Burp Suite Enterprise Edition**.

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

В этом разделе описано, как включить единый вход Azure для пользователя B. Simon, предоставив этому пользователю доступ к Burp Suite Enterprise Edition.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. Из списка приложений выберите **Burp Suite Enterprise Edition**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-burp-suite-enterprise-edition-sso"></a>Настройка единого входа Burp Suite Enterprise Edition

Чтобы настроить единый вход на стороне **Burp Suite Enterprise Edition**, нужно отправить скачанный **сертификат (Base64)** и соответствующие URL-адреса, скопированные на портале Azure, в [группу поддержки Burp Suite Enterprise Edition](mailto:support@portswigger.net). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-burp-suite-enterprise-edition-test-user"></a>Создание тестового пользователя Burp Suite Enterprise Edition

В этом разделе вы создадите в Burp Suite Enterprise Edition пользователя Britta Simon. Приложение Burp Suite Enterprise Edition поддерживает JIT-подготовку пользователей, которая включена по умолчанию. В этом разделе никакие действия с вашей стороны не требуются. Если пользователь еще не существует в Burp Suite Enterprise Edition, он создается после проверки подлинности.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов.

* Выберите "Тестировать приложение" на портале Azure, и вы автоматически войдете в приложение Burp Suite Enterprise Edition, для которого настроен единый вход.

* Вы можете использовать портал "Мои приложения" корпорации Майкрософт. Щелкнув плитку Burp Suite Enterprise Edition в разделе "Мои приложения", вы автоматически войдете в приложение Burp Suite Enterprise Edition, для которого настроен единый вход. Дополнительные сведения о портале "Мои приложения" см. в [этой статье](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).


## <a name="next-steps"></a>Дальнейшие действия

После настройки Burp Suite Enterprise Edition вы можете применить функцию управления сеансом, которая защищает конфиденциальные данные вашей организации от кражи и несанкционированного доступа в режиме реального времени. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).
