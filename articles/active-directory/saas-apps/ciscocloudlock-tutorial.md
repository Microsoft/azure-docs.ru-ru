---
title: Руководство по интеграции Azure Active Directory с The Cloud Security Fabric | Документы Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в The Cloud Security Fabric.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/18/2019
ms.author: jeedes
ms.openlocfilehash: 7025b948615efcf6ace3ca0fb6a2daecdd75c702
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92456043"
---
# <a name="tutorial-integrate-the-cloud-security-fabric-with-azure-active-directory"></a>Руководство. Интеграция Azure Active Directory с The Cloud Security Fabric

В этом руководстве описано, как интегрировать The Cloud Security Fabric с Azure Active Directory (Azure AD). Возможности интеграции The Cloud Security Fabric с Azure AD.

* Контроль с помощью Azure AD доступа к The Cloud Security Fabric.
* Включение автоматического входа пользователей в The Cloud Security Fabric с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

Чтобы узнать больше об интеграции приложений SaaS с Azure AD, прочитайте статью [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* подписка с поддержкой единого входа на The Cloud Security Fabric.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* The Cloud Security Fabric поддерживает единый вход, инициированный **поставщиком услуг**.

## <a name="adding-the-cloud-security-fabric-from-the-gallery"></a>Добавление The Cloud Security Fabric из коллекции

Чтобы настроить интеграцию The Cloud Security Fabric с Azure AD, вам нужно добавить The Cloud Security Fabric из коллекции в список управляемых приложений SaaS.

1. Войдите на [портал Azure](https://portal.azure.com) с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **The Cloud Security Fabric**.
1. Выберите **The Cloud Security Fabric** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.


## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

Настройте и проверьте единый вход Azure AD в The Cloud Security Fabric с помощью тестового пользователя **B.Simon**. Чтобы обеспечить работу единого входа, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в The Cloud Security Fabric.

Чтобы настроить и проверить единый вход Azure AD в The Cloud Security Fabric, выполните действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
2. **[Настройка единого входа в The Cloud Security Fabric](#configure-the-cloud-security-fabric-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.
3. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
4. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
5. **[Создание тестового пользователя The Cloud Security Fabric](#create-the-cloud-security-fabric-test-user)** требуется для того, чтобы в The Cloud Security Fabric существовал пользователь B.Simon, связанный с одноименным пользователем в Azure AD.
6. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

### <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **The Cloud Security Fabric** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

4. В разделе **Базовая конфигурация SAML** выполните приведенные ниже действия.

    а. В текстовом поле **URL-адрес входа** введите URL-адрес: .

      ```http
      https://platform.cloudlock.com
      https://app.cloudlock.com
      ```

    b. В текстовом поле **Идентификатор (сущности)** введите URL-адрес в следующем формате:

      ```http
      https://platform.cloudlock.com/gate/saml/sso/<subdomain>
      https://app.cloudlock.com/gate/saml/sso/<subdomain>
      ```

    > [!NOTE]
    > Значение идентификатора приведено для примера и не является реальным. Вместо него нужно указать фактический идентификатор. Чтобы получить это значение, обратитесь в [службу поддержки The Cloud Security Fabric](mailto:support@cloudlock.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

4. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите элемент **XML метаданных федерации** и нажмите кнопку **Скачать**, чтобы скачать сертификат и сохранить его на компьютере.

    ![Ссылка для скачивания сертификата](common/metadataxml.png)

5. Чтобы изменить параметры в разделе **Подпись** в соответствии со своими требованиями, нажмите кнопку **Изменить**, чтобы открыть диалоговое окно **Сертификат подписи SAML**.

    ![Ответ знака](./media/ciscocloudlock-tutorial/saml.png)

    а. Для параметра **Вариант подписывания** выберите значение **Ответ знака SAML и утверждение**.

    b. Выберите параметр **SHA-256** для **алгоритма подписывания**

    c. Выберите команду **Сохранить**.  

6. Требуемые URL-адреса можно скопировать из раздела **Настройка The Cloud Security Fabric**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

### <a name="configure-the-cloud-security-fabric-sso"></a>Настройка единого входа The Cloud Security Fabric

Чтобы настроить единый вход на стороне **The Cloud Security Fabric**, нужно отправить скачанный **XML-файл метаданных федерации** и соответствующие URL-адреса, скопированные на портале Azure, [группе поддержки The Cloud Security Fabric](mailto:support@cloudlock.com). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.
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

В этом разделе описано, как включить единый вход в Azure для пользователя B.Simon, предоставив этому пользователю доступ к The Cloud Security Fabric.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **The Cloud Security Fabric**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.

   ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Ссылка "Добавить пользователя"](common/add-assign-user.png)

1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор роли** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

### <a name="create-the-cloud-security-fabric-test-user"></a>Создание тестового пользователя The Cloud Security Fabric

В этом разделе описано, как создать пользователя B.Simon в приложении The Cloud Security Fabric. Чтобы добавить пользователей на платформе The Cloud Security Fabric, обратитесь в [службу поддержки The Cloud Security Fabric](mailto:support@cloudlock.com). Перед использованием единого входа необходимо создать и активировать пользователей.

### <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув элемент "The Cloud Security Fabric" на Панели доступа, вы автоматически войдете в приложение The Cloud Security Fabric, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)