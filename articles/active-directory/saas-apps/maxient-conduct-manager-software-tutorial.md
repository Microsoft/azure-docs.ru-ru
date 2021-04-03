---
title: Руководство по Интеграция единого входа Azure Active Directory с Maxient Conduct Manager Software | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в Maxient Conduct Manager Software.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/18/2019
ms.author: jeedes
ms.openlocfilehash: 4b57b3fbb338774eb8d66fc4d3c0c817e19aff4c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92458206"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-maxient-conduct-manager-software"></a>Руководство по Интеграция единого входа Azure Active Directory с Maxient Conduct Manager Software

В этом руководстве описано, как интегрировать Maxient Conduct Manager Software с Azure Active Directory (AAD). Интеграция Maxient Conduct Manager Software с Azure AD обеспечивает следующие возможности.

* Использование Azure AD для проверки подлинности пользователей в Maxient Conduct Manager Software.
* Автоматический вход пользователей в Maxient Conduct Manager Software с учетными записями AAD.


Чтобы узнать больше об интеграции приложений SaaS с Azure AD, прочитайте статью [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* подписка Maxient Conduct Manager Software с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В этом руководстве показано, как настроить Azure AD для использования с Maxient Conduct Manager Software.


* Maxient Conduct Manager Software поддерживает единый вход, инициированный **поставщиком услуг и поставщиком удостоверений**.

> [!NOTE]
> Идентификатор этого приложения — фиксированное строковое значение, поэтому в одном клиенте можно настроить только один экземпляр.

## <a name="adding-maxient-conduct-manager-software-from-the-gallery"></a>Добавление Maxient Conduct Manager Software из коллекции

Чтобы настроить интеграцию Maxient Conduct Manager Software с Azure AD, вам потребуется добавить Maxient Conduct Manager Software из коллекции в список управляемых приложений SaaS.

1. Войдите на [портал Azure](https://portal.azure.com) с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Maxient Conduct Manager Software**.
1. Выберите **Maxient Conduct Manager Software** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.


## <a name="configure-and-test-azure-ad-single-sign-on-for-maxient-conduct-manager-software"></a>Настройка и проверка единого входа Azure AD для Maxient Conduct Manager Software

Настройте и проверьте единый вход Azure AD в Maxient Conduct Manager Software. Чтобы обеспечить работу единого входа, необходимо установить подключение между Azure AD и Maxient Conduct Manager Software.

Чтобы настроить и проверить единый вход Azure AD в Maxient Conduct Manager Software, выполните действия в следующих стандартных блоках.

1. **[Настройте единый вход Azure AD](#configure-azure-ad-sso)** , чтобы пользователи могли проходить проверку подлинности в Maxient Conduct Manager Software.
    1. **[Предоставьте всем пользователям возможность использовать Maxient](#assign-all-users-to-be-able-to-authenticate-for-the-maxient-conduct-manager-software)** , чтобы все пользователи в вашем учреждении могли проходить проверку подлинности.
1. **[Протестируйте настройку Azure AD для использования в Maxient](#test-with-maxient)** , чтобы проверить работу конфигурации и правильность выпускаемых атрибутов.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На [портале Azure](https://portal.azure.com/) на странице интеграции с приложением **Maxient Conduct Manager Software** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. В разделе **Базовая конфигурация SAML** приложение предварительно настроено в режиме инициированном **IDP** и ему заданы требуемые URL-адреса в Azure. Пользователь должен сохранить конфигурацию, нажав кнопку **Сохранить**.

1. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://cm.maxient.com/<SCHOOLCODE>`.

    > [!NOTE]
    > Это значение приведено для примера. Вместо него необходимо указать фактический URL-адрес входа. Обратитесь к специалисту по реализации или представителю группы поддержки Maxient, чтобы получить это значение.

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** нажмите кнопку "Копировать", чтобы скопировать **URL-адрес метаданных федерации приложений** и сохранить его на компьютере.  Вам потребуется предоставить специалисту по реализации или представителю группы поддержки Maxient этот URL-адрес.

    ![Ссылка для скачивания сертификата](common/copy-metadataurl.png)

### <a name="assign-all-users-to-be-able-to-authenticate-for-the-maxient-conduct-manager-software"></a>Назначение всем пользователям возможности выполнять проверку подлинности в Maxient Conduct Manager Software

В этом разделе показано, как предоставить всем учетным записям доступ для проверки подлинности с помощью системы Azure в Maxient Conduct Manager Software.  Важно отметить, что этот шаг является **ОБЯЗАТЕЛЬНЫМ** для корректной работы Maxient.  Maxient использует систему Azure AD для *проверки подлинности* пользователей. *Проверка подлинности* пользователей выполняется в системе Maxient для конкретной функции, которую они пытаются выполнить. Maxient не использует атрибуты из вашего каталога для принятия таких решений.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Maxient Conduct Manager Software**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.

   ![Ссылка "Пользователи и группы"](common/users-groups-blade.png)

1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Ссылка "Добавить пользователя"](common/add-assign-user.png)

1. В диалоговом окне **Пользователи и группы** выберите всех пользователей (или соответствующие группы) и **назначьте** им возможность выполнять проверку подлинности в Maxient.

## <a name="test-with-maxient"></a>Тестирование в Maxient 

Если вы не создавали запрос в службу поддержки Maxient, отправьте электронное письмо на адрес [support@maxient.com](mailto:support@maxient.com) с темой "Campus Based Authentication/Azure Setup — \<\<School Name\>\>". В тексте сообщения укажите **URL-адрес метаданных федерации приложений**. Сотрудник Maxient отправит в ответе тестовую ссылку, чтобы проверить правильность выпускаемых атрибутов.  
    
## <a name="additional-resources"></a>Дополнительные ресурсы

- [Список учебников по интеграции приложений SaaS с Azure Active Directory](./tutorial-list.md)

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)

- [Проверьте работу Maxient Conduct Manager Software с AAD](https://aad.portal.azure.com/)