---
title: Настройка потока входа
titleSuffix: Azure Active Directory B2C
description: Узнайте, как настроить поток входа в Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 01/12/2021
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: c4db7bf989a0062f87a5c3e6303ba64224996554
ms.sourcegitcommit: d59abc5bfad604909a107d05c5dc1b9a193214a8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/14/2021
ms.locfileid: "98224494"
---
# <a name="set-up-a-sign-in-flow-in-azure-active-directory-b2c"></a>Настройка потока входа в Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

## <a name="sign-in-flow-overview"></a>Общие сведения о потоке входа

Политика входа позволяет пользователям: 

* Пользователи могут выполнять вход с помощью локальной учетной записи Azure AD B2C
* Регистрация или вход с помощью учетной записи социальных сетей
* Сброс паролей
* Пользователи не могут зарегистрироваться для использования локальной учетной записи Azure AD B2C. для создания учетной записи Администратор может использовать [MS API Graph](manage-user-accounts-graph-api.md).

![Поток редактирования профиля](./media/add-sign-in-policy/sign-in-user-flow.png)

## <a name="prerequisites"></a>Предварительные требования

Если вы еще не сделали этого, [зарегистрируйте веб-приложение в Azure Active Directory B2C](tutorial-register-applications.md).

::: zone pivot="b2c-user-flow"

## <a name="create-a-sign-in-user-flow"></a>Создание потока пользователя входа

Чтобы добавить политику входа, выполните следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com).
1. Выберите значок **Каталог и подписка** в верхней панели инструментов портала, а затем выберите каталог, содержащий клиент Azure AD B2C.
1. На портале Azure найдите и выберите **Azure AD B2C**.
1. В разделе **Политики** выберите **Потоки пользователей** и щелкните **Создать поток пользователя**.
1. На странице **Создание потока пользователя** выберите поток **входа** пользователя.
1. В разделе **Выбор версии**, выберите элемент **Рекомендуемая** и нажмите кнопку **Создать**. (См. сведения о [версиях потоков пользователей](user-flow-versions.md).)
1. Введите **имя** потока пользователя. Например, *signupsignin1*.
1. Для **поставщиков удостоверений** выберите **Вход по электронной почте**.
1. Для **утверждения приложения** выберите утверждения и атрибуты, которые необходимо отправить в приложение. Например, выберите **Показать больше**, а затем выберите атрибуты и утверждения для **отображаемого имени**, **заданное имя**,  **Фамилия** и **идентификатор объекта пользователя**. Нажмите кнопку **ОК**.
1. Для добавления потока пользователя щелкните **Создать**. Префикс *B2C_1* добавляется к имени автоматически.

### <a name="test-the-user-flow"></a>Тестирование потока пользователя

1. Выберите созданный поток пользователя, чтобы открыть его страницу обзора, а затем щелкните **Выполнить поток пользователя**.
1. В разделе **Приложение** выберите зарегистрированное ранее веб-приложение с именем *webapp1*. В поле **URL-адрес ответа** должно содержаться значение `https://jwt.ms`.
1. Щелкните **Выполнить поток пользователя**.
1. Вы можете войти в систему с помощью созданной учетной записи (с помощью MS API Graph) без ссылки для регистрации. Возвращаемый токен включает выбранные утверждения.

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="remove-the-sign-up-link"></a>Удалить ссылку для регистрации

Технический профиль **SelfAsserted-локалаккаунтсигнин-email** — это [самоподтвержденный](self-asserted-technical-profile.md), который вызывается во время регистрации или входа. Чтобы удалить ссылку для регистрации, установите `setting.showSignupLink` для метаданных значение `false` . Переопределите технические профили SelfAsserted-Локалаккаунтсигнин-email в файле расширения. 

1. Откройте файл расширения для политики, например _`SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`**_.
1. Найдите элемент `ClaimsProviders` . Если такой элемент не существует, добавьте его.
1. Добавьте в элемент следующий поставщик утверждений `ClaimsProviders` :

    ```xml
    <ClaimsProvider>
      <DisplayName>Local Account</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="SelfAsserted-LocalAccountSignin-Email">
          <Metadata>
            <Item Key="setting.showSignupLink">false</Item>
          </Metadata>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

1. В `<BuildingBlocks>` элемент element добавьте следующий [контентдефинитион](contentdefinitions.md) для ссылки на URI версии 1.2.0 или более поздней:

    ```XML
    <ContentDefinitions>
     <ContentDefinition Id="api.localaccountsignup">
        <DataUri>urn:com:microsoft:aad:b2c:elements:contract:unifiedssp:1.2.0</DataUri>
      </ContentDefinition>
    </ContentDefinitions>
    ```

## <a name="update-and-test-your-policy"></a>Обновление и тестирование политики

1. Войдите на [портал Azure](https://portal.azure.com).
1. Убедитесь, что вы используете каталог с клиентом Azure AD, выбрав фильтр **Каталог и подписка** в меню вверху и каталог с вашим клиентом Azure AD.
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Регистрация приложений**.
1. Выберите **Инфраструктура процедур идентификации**.
1. Выберите **Отправка пользовательской политики** и отправьте два файла политики, которые вы изменили ранее.
1. Выберите отправленную политику входа и нажмите кнопку **Запустить сейчас** .
1. Вы можете войти в систему с помощью созданной учетной записи (с помощью MS API Graph) без ссылки для регистрации.

::: zone-end

## <a name="next-steps"></a>Дальнейшие действия

* Добавьте [Вход с помощью поставщика удостоверений в социальных сетях](add-identity-provider.md).
* Настройка [потока сброса пароля](add-password-reset-policy.md).
