---
title: Руководство по интеграции единого входа Azure Active Directory с IBM Digital Business Automation on Cloud | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и IBM Digital Business Automation on Cloud.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/08/2020
ms.author: jeedes
ms.openlocfilehash: 91a50ed85b5802657a27c5a545de0013937e9da0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92460569"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-ibm-digital-business-automation-on-cloud"></a>Руководство по интеграции единого входа Azure Active Directory с IBM Digital Business Automation on Cloud

В этом учебнике вы узнаете, как интегрировать IBM Digital Business Automation on Cloud с Azure Active Directory (Azure AD). Интеграция IBM Digital Business Automation on Cloud с Azure AD обеспечивает следующие возможности:

* Управление доступом к IBM Digital Business Automation on Cloud с помощью Azure AD.
* Автоматический вход пользователей в IBM Digital Business Automation on Cloud с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

Чтобы узнать больше об интеграции приложений SaaS с Azure AD, прочитайте статью [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка на IBM Digital Business Automation on Cloud с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* IBM Digital Business Automation on Cloud поддерживает единый вход, инициированный **поставщиком услуг и поставщиком удостоверений**.
* После настройки IBM Digital Business Automation on Cloud можно применять элементы управления сеансами, которые защищают от хищения конфиденциальных данных вашей организации и несанкционированного доступа к ним в режиме реального времени. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-ibm-digital-business-automation-on-cloud-from-the-gallery"></a>Добавление IBM Digital Business Automation on Cloud из коллекции

Чтобы настроить интеграцию IBM Digital Business Automation on Cloud с Azure AD, необходимо добавить это решение из коллекции в список управляемых приложений SaaS.

1. Войдите на [портал Azure](https://portal.azure.com) с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **IBM Digital Business Automation on Cloud**.
1. Выберите **IBM Digital Business Automation on Cloud** в области результатов, а затем добавьте приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.


## <a name="configure-and-test-azure-ad-single-sign-on-for-ibm-digital-business-automation-on-cloud"></a>Настройка и проверка единого входа Azure AD для IBM Digital Business Automation on Cloud

Настройте и проверьте единый вход Azure AD в IBM Digital Business Automation on Cloud с помощью тестового пользователя **B. Simon**. Чтобы обеспечить работу единого входа, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в IBM Digital Business Automation on Cloud.

Чтобы настроить и проверить единый вход Azure AD в IBM Digital Business Automation on Cloud, выполните действия из следующих стандартных блоков:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в IBM Digital Business Automation on Cloud](#configure-ibm-digital-business-automation-on-cloud-sso)** необходима, чтобы задать параметры единого входа на стороне приложения.
    1. **[Создание тестового пользователя IBM Digital Business Automation on Cloud](#create-ibm-digital-business-automation-on-cloud-test-user)** требуется для того, чтобы в IBM Digital Business Automation on Cloud существовал пользователь B. Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **IBM Digital Business Automation on Cloud** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. Если у вас есть **файл метаданных поставщика служб**, выполните следующие действия в разделе **Базовая конфигурация SAML**:
    
    а. Щелкните **Отправить файл метаданных**.

    b. Щелкните **значок папки**, выберите файл метаданных и нажмите кнопку **Отправить**.

    c. После успешной отправки файла метаданных значения **Идентификатор** и **URL-адрес ответа** будут заполнены автоматически в текстовом поле в разделе IBM Digital Business Automation on Cloud.

    > [!Note]
    > Если поля **Идентификатор** и **URL-адрес ответа** автоматически не заполняются, введите эти значения вручную в соответствии с поставленной задачей.

    > [!Note]
    > Клиенты могут получить файл метаданных для своей облачной подписки, обратившись в [службу поддержки IBM Digital Business Automation on Cloud](mailto:supportbpmoncloud@us.ibm.com).

1. Если вы хотите настроить приложение в режиме, **инициируемом поставщиком удостоверений**, но у вас нет **файла метаданных, предоставленного поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `https://www.automationcloud.ibm.com/isam/sps/<TENANT>/saml20`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://www.automationcloud.ibm.com/isam/sps/<TENANT>/saml20/login`.

1. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://www.automationcloud.ibm.com/isam/sps/<TENANT>/login`.

    > [!NOTE]
    > Эти значения приведены для примера. Замените их фактическими значениями идентификатора, URL-адреса ответа и URL-адреса входа. Чтобы получить эти значения, обратитесь в [службу поддержки клиентов IBM Digital Business Automation on Cloud](mailto:supportbpmoncloud@us.ibm.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите элемент **XML метаданных федерации** и выберите **Скачать**, чтобы скачать сертификат и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

1. Скопируйте требуемые URL-адреса в разделе **Настройка IBM Digital Business Automation on Cloud**.

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

В этом разделе описано, как включить единый вход Azure для пользователя B. Simon, предоставив этому пользователю доступ к IBM Digital Business Automation on Cloud.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **IBM Digital Business Automation on Cloud**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.

   ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Ссылка "Добавить пользователя"](common/add-assign-user.png)

1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор роли** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-ibm-digital-business-automation-on-cloud-sso"></a>Настройка единого входа в IBM Digital Business Automation on Cloud

Для настройки единого входа на стороне **IBM Digital Business Automation on Cloud** необходимо отправить скачанный **XML-файл метаданных федерации** и соответствующие URL-адреса, скопированные на портале Azure, в [IBM Digital Business Automation on Cloud](mailto:supportbpmoncloud@us.ibm.com). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-ibm-digital-business-automation-on-cloud-test-user"></a>Настройка тестового пользователя в IBM Digital Business Automation on Cloud

В этом разделе описано, как создать пользователя Britta Simon в приложении IBM Digital Business Automation on Cloud. Чтобы добавить пользователей на платформу IBM Digital Business Automation on Cloud обратитесь в [службу поддержки этой платформы](mailto:supportbpmoncloud@us.ibm.com). Перед использованием единого входа необходимо создать и активировать пользователей.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку IBM Digital Business Automation on Cloud на Панели доступа, вы автоматически войдете в решение IBM Digital Business Automation on Cloud, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)

- [Попробуйте использовать IBM Digital Business Automation on Cloud с Azure AD](https://aad.portal.azure.com/)

- [Что такое управление сеансами в Microsoft Cloud App Security?](/cloud-app-security/proxy-intro-aad)

- [Защита IBM Digital Business Automation on Cloud с помощью средств обеспечения расширенной видимости и улучшенных элементов управления](/cloud-app-security/proxy-intro-aad)