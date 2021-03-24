---
title: Руководство по Интеграция единого входа Azure Active Directory с доступом к Microsoft Azure с единым входом для Ethidex Compliance Office™ | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory для доступа к Microsoft Azure с единым входом для Ethidex Compliance Office™
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/06/2019
ms.author: jeedes
ms.openlocfilehash: 7ef219ca147fe96fc65f14bbf3ba6a565adc95ec
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92520920"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-ms-azure-sso-access-for-ethidex-compliance-office"></a>Руководство по Интеграция единого входа Azure Active Directory с доступом к Microsoft Azure с единым входом для Ethidex Compliance Office™

В этом руководстве описано, как интегрировать доступ к Microsoft Azure с единым входом для Ethidex Compliance Office™ с Azure Active Directory (Azure AD). Интеграция доступа к Microsoft Azure с единым входом для Ethidex Compliance Office™ с Azure AD дает следующие преимущества:

* Вы можете управлять через AAD правами использования доступа Microsoft Azure с единым входом для Ethidex Compliance Office™.
* Вы можете включить автоматический вход пользователей в доступ к Microsoft Azure с единым входом для Ethidex Compliance Office™ с использованием учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

Чтобы узнать больше об интеграции приложений SaaS с Azure AD, прочитайте статью [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка доступа к Microsoft Azure с единым входом для Ethidex Compliance Office™ с единым входом.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Доступ к Microsoft Azure с единым входом для Ethidex Compliance Office™ поддерживает единый вход, инициированный **поставщиком удостоверений**.

## <a name="adding-ms-azure-sso-access-for-ethidex-compliance-office-from-the-gallery"></a>Добавление доступа к Microsoft Azure с единым входом для Ethidex Compliance Office™ из коллекции

Чтобы настроить интеграцию доступа к Microsoft Azure с единым входом для Ethidex Compliance Office™  с Azure Active Directory, необходимо добавить из коллекции в список управляемых приложений SaaS доступ к Microsoft Azure с единым входом для Ethidex Compliance Office™.

1. Войдите на [портал Azure](https://portal.azure.com) с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** введите **доступ к Microsoft Azure с единым входом для Ethidex Compliance Office™** в поле поиска.
1. Выберите **доступ к Microsoft Azure с единым входом для Ethidex Compliance Office™** из панели результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-single-sign-on-for-ms-azure-sso-access-for-ethidex-compliance-office"></a>Настройка и тестирование единого входа AAD для доступа к Microsoft Azure с единым входом для Ethidex Compliance Office™

Настройте и проверьте единый вход Azure AD в Доступ к Microsoft Azure с единым входом для Ethidex Compliance Office™ с помощью тестового пользователя **B.Simon**. Для обеспечения работы единого входа необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Доступе к Microsoft Azure с единым входом для Ethidex Compliance Office™.

Настройте и проверьте единый вход Azure AD в Доступ к Microsoft Azure с единым входом для Ethidex Compliance Office™, выполнив действия из следующих блоков:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Доступ к Microsoft Azure с единым входом для Ethidex Compliance Office](#configure-ms-azure-sso-access-for-ethidex-compliance-office-sso)** необходима для настройки параметров единого входа на стороне приложения.
    1. **[Создание тестового пользователя Доступа к Microsoft Azure с единым входом для Ethidex Compliance Office™](#create-ms-azure-sso-access-for-ethidex-compliance-office-test-user)** позволяет создать в приложении Доступ к Microsoft Azure с единым входом для Ethidex Compliance Office™  пользователя B.Simon, связанного с представлением этого пользователя в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Доступ к Microsoft Azure с единым входом для Ethidex Compliance Office™**  найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `com.ethidex.prod.<CLIENTID>`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://www.ethidex.com/saml2/sp/acs/<CLIENTID>`.

    > [!NOTE]
    > Эти значения приведены для примера. Измените их на фактические значения идентификатора и URL-адреса ответа. Обратитесь в [службу поддержки доступа с единым входом в MS Azure для Ethidex Compliance Office™](mailto:support@ethidex.com), чтобы получить эти значения. Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. Приложение Доступ к Microsoft Azure с единым входом для Ethidex Compliance Office™ ожидает утверждения SAML в определенном формате, который требует добавить настраиваемые сопоставления атрибутов в вашу конфигурацию атрибутов токена SAML. На следующем снимке экрана показан список атрибутов по умолчанию, в котором **nameidentifier** сопоставляется с **user.userprincipalname**. Приложение Доступ к Microsoft Azure с единым входом для Ethidex Compliance Office™ ожидает, что **nameidentifier** будет сопоставляться с **user.mail**, поэтому необходимо изменить сопоставление атрибутов, щелкнув значок **Изменить**.

    ![Изображение](common/edit-attribute.png)

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите элемент **Сертификат (необработанный)** и щелкните **Скачать**, чтобы скачать сертификат. Сохраните этот сертификат на компьютере.

    ![Ссылка для скачивания сертификата](common/certificateraw.png)

1. Скопируйте требуемые URL-адреса из раздела **Настройка Доступа к Microsoft Azure с единым входом для Ethidex Compliance Office™**.

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

В этом разделе описано, как включить единый вход Azure для пользователя B.Simon, предоставив этому пользователю доступ к Доступу к Microsoft Azure с единым входом для Ethidex Compliance Office™.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Доступ к Microsoft Azure с единым входом для Ethidex Compliance Office™**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.

   ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Ссылка "Добавить пользователя"](common/add-assign-user.png)

1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если ожидается, что в утверждении SAML будет получено какое-либо значение роли, то в диалоговом окне **Выбор роли** нужно выбрать соответствующую роль для пользователя из списка и затем нажать кнопку **Выбрать**, расположенную в нижней части экрана.
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-ms-azure-sso-access-for-ethidex-compliance-office-sso"></a>Настройка единого входа на стороне Доступа к Microsoft Azure с единым входом для Ethidex Compliance Office™

Чтобы настроить единый вход на стороне **Доступа к Microsoft Azure с единым входом для Ethidex Compliance Office™**, необходимо отправить скачанный **сертификат (RAW)** и соответствующие URL-адреса, скопированные на портале Azure, в [Доступ к Microsoft Azure с единым входом для Ethidex Compliance Office™](mailto:support@ethidex.com). Специалисты службы поддержки настроят подключение единого входа SAML на обеих сторонах.

### <a name="create-ms-azure-sso-access-for-ethidex-compliance-office-test-user"></a>Создание тестового пользователя в Доступе к Microsoft Azure с единым входом для Ethidex Compliance Office™

В этом разделе вы создадите пользователя с именем B.Simon в Доступе к Microsoft Azure с единым входом для Ethidex Compliance Office™. Чтобы добавить пользователя на платформу "Доступ к Microsoft Azure с единым входом для Ethidex Compliance Office™", обратитесь в [службу поддержки приложения](mailto:support@ethidex.com). Перед использованием единого входа необходимо создать и активировать пользователей.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув плитку Доступ к Microsoft Azure с единым входом для Ethidex Compliance Office™ на панели доступа, вы автоматически войдете в приложение Доступ к Microsoft Azure с единым входом для Ethidex Compliance Office™, для которого настроили единый вход. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)

- [Тестирование Доступа к Microsoft Azure с единым входом для Ethidex Compliance Office™ с AAD](https://aad.portal.azure.com/)