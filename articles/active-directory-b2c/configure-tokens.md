---
title: Настройка маркеров (Azure Active Directory B2C) | Документация Майкрософт
description: Узнайте, как настроить время существования и параметры совместимости маркера в Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 12/14/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: f22a55a65b7dc4fd4f714d880804e6fd65dbbe46
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101654380"
---
# <a name="configure-tokens-in-azure-active-directory-b2c"></a>Настройка маркеров в Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

Из этой статьи вы узнаете, как настроить [время существования и параметры совместимости маркера](tokens-overview.md) в Azure Active Directory B2C (Azure AD B2C).

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

## <a name="token-lifetime-behavior"></a>Поведение маркера

Время существования маркера можно настроить, включая:

- **Время существования маркера доступа и идентификатора (в минутах)** — время существования токенов носителя OAuth 2,0 и маркеров идентификации. Значение по умолчанию — 60 минут (1 час). Минимальное значение — 5 минут (включительно). Максимальное значение (включительно) — 1 440 минут (24 часа).
- **Время существования маркера обновления (в днях)** — максимальный период времени, до которого маркер обновления может быть использован для получения нового маркера доступа, если приложению была предоставлена `offline_access` область. По умолчанию — 14 дней. Минимальное значение — один день (включительно). Максимальное (включительное) 90 дней.
- **Время существования скользящего окна маркера обновления** — скользящий тип окна токена обновления. `Bounded` Указывает, что маркер обновления можно расширить, как указано в параметре **Длина времени существования (в днях)**. `No expiry` Указывает, что время существования скользящего окна маркера обновления никогда не истекает.
- **Продолжительность существования (в днях)** — по истечении этого периода времени пользователь вынужден выполнить повторную проверку подлинности независимо от срока действия последнего маркера обновления, полученного приложением. Значение должно быть больше или равно значению **времени существования маркера обновления** .

На следующей диаграмме показано поведение скользящего времени существования маркера обновления.

![Время существования маркера обновления](./media/configure-tokens/refresh-token-lifetime.png)

> [!NOTE]
> >Одностраничные приложения, использующие поток кода авторизации с PKCE, всегда имеют время существования маркера обновления, равное 24 часам, тогда как мобильные приложения, классические приложения и веб-приложения не сталкиваются с этим ограничением. [Узнайте больше о влиянии маркеров обновления в браузере на безопасность](../active-directory/develop/reference-third-party-cookies-spas.md#security-implications-of-refresh-tokens-in-the-browser).


## <a name="configure-token-lifetime"></a>Настройка времени существования токена

::: zone pivot="b2c-user-flow"

Настройка времени существования маркера потока пользователя:

1. Войдите на [портал Azure](https://portal.azure.com).
1. Убедитесь, что вы используете каталог, содержащий клиент Azure AD B2C. Выберите фильтр **Каталог и подписка** в верхнем меню, а затем каталог, содержащий клиент Azure AD B2C.
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
1. Выберите **Потоки пользователей (политики)** .
1. Откройте созданный ранее пользовательский поток.
1. Выберите **Свойства**.
1. В разделе **время существования маркера** настройте свойства в соответствии с потребностями приложения.
1. Выберите команду **Сохранить**.

::: zone-end

::: zone pivot="b2c-custom-policy"

Чтобы изменить параметры совместимости маркеров, необходимо задать метаданные технического профиля [издателя маркера](jwt-issuer-technical-profile.md) в расширении или файл проверяющей стороны политики, для которой вы хотите повлиять. Технический профиль издателя маркера выглядит следующим образом:

```xml
<ClaimsProviders>
  <ClaimsProvider>
    <DisplayName>Token Issuer</DisplayName>
    <TechnicalProfiles>
      <TechnicalProfile Id="JwtIssuer">
        <Metadata>
          <Item Key="token_lifetime_secs">3600</Item>
          <Item Key="id_token_lifetime_secs">3600</Item>
          <Item Key="refresh_token_lifetime_secs">1209600</Item>
          <Item Key="rolling_refresh_token_lifetime_secs">7776000</Item>
          <!--<Item Key="allow_infinite_rolling_refresh_token">true</Item>-->
          <Item Key="IssuanceClaimPattern">AuthorityAndTenantGuid</Item>
          <Item Key="AuthenticationContextReferenceClaimPattern">None</Item>
        </Metadata>
      </TechnicalProfile>
    </TechnicalProfiles>
  </ClaimsProvider>
</ClaimsProviders>
```

В предыдущем примере установлены следующие значения.

- **token_lifetime_secs** — время существования маркера доступа (в секундах). Значение по умолчанию — 3 600 (1 час). Минимальное значение — 300 (5 минут). Максимальное значение — 86 400 (24 часа). 
- время существования маркера **id_token_lifetime_secs** -ID (в секундах). Значение по умолчанию — 3 600 (1 час). Минимальное значение — 300 (5 минут). Максимальное значение — 86 400 (24 часа). 
- **refresh_token_lifetime_secs** Время существования маркера обновления (в секундах). Значение по умолчанию — 120, 9600 (14 дней). Минимальное значение — 86 400 (24 часа). Максимальное значение — 7 776 000 (90 дней). 
- **rolling_refresh_token_lifetime_secs** время существования скользящего окна маркера обновления (в секундах). Значение по умолчанию — 7 776 000 (90 дней). Минимальное значение — 86 400 (24 часа). Максимальное значение — 31 536 000 (365 дней). Если вы не хотите применять скользящее время существования окна, установите значение `allow_infinite_rolling_refresh_token` `true` . 
- **allow_infinite_rolling_refresh_token** время существования скользящего окна маркера обновления никогда не истекает. 

::: zone-end


## <a name="token-compatibility-settings"></a>Параметры маркеров совместимости

Можно настроить совместимость маркеров, в том числе:

- **Утверждение издателя (ISS)** — формат издателя маркера доступа и идентификатора.
- **Утверждение субъекта (** подзапроса) — субъект, о котором маркер утверждает сведения, например пользователя приложения. Это значение является неизменяемым и не может быть переназначено или повторно использовано. Это значение можно использовать для безопасных проверок авторизации, например, когда маркер используется для доступа к ресурсу. По умолчанию утверждение субъекта заполняется идентификатором объекта пользователя в каталоге.
- **Утверждение, представляющее поток пользователя** . это утверждение определяет выполняемый пользователем поток. Возможные значения: `tfp` (по умолчанию) или `acr`.

::: zone pivot="b2c-user-flow"

Чтобы настроить параметры совместимости потока пользователей, выполните следующие действия.

1. Выберите **Потоки пользователей (политики)** .
1. Откройте созданный ранее пользовательский поток.
1. Выберите **Свойства**.
1. В разделе **параметры совместимости токенов** настройте свойства в соответствии с потребностями приложения.
1. Выберите команду **Сохранить**.

::: zone-end

::: zone pivot="b2c-custom-policy"

Чтобы изменить параметры совместимости маркеров, необходимо задать метаданные технического профиля [издателя маркера](jwt-issuer-technical-profile.md) в расширении или файл проверяющей стороны политики, для которой вы хотите повлиять. Технический профиль издателя маркера выглядит следующим образом:

```xml
<ClaimsProviders>
  <ClaimsProvider>
    <DisplayName>Token Issuer</DisplayName>
    <TechnicalProfiles>
      <TechnicalProfile Id="JwtIssuer">
        <Metadata>
          ...
          <Item Key="IssuanceClaimPattern">AuthorityAndTenantGuid</Item>
          <Item Key="AuthenticationContextReferenceClaimPattern">None</Item>
        </Metadata>
      </TechnicalProfile>
    </TechnicalProfiles>
  </ClaimsProvider>
</ClaimsProviders>
```

- **Утверждение издателя (iss).** Утверждение издателя (iss) устанавливается с помощью элемента метаданных **IssuanceClaimPattern**. Применимые значения — `AuthorityAndTenantGuid` и `AuthorityWithTfp`.
- **Утверждение настройки, представляющее идентификатор политики.** Параметры, использующиеся для настройки этого значения, — `TFP` (инфраструктура доверия) и `ACR` (ссылка на контекст проверки подлинности). `TFP` является рекомендуемым значением. Установите значение `None` для **AuthenticationContextReferenceClaimPattern**.

    В элемент **ClaimsSchema** добавьте такой элемент.

    ```xml
    <ClaimType Id="trustFrameworkPolicy">
      <DisplayName>Trust framework policy name</DisplayName>
      <DataType>string</DataType>
    </ClaimType>
    ```

    Добавьте следующий элемент в элемент **OutputClaims**.

    ```xml
    <OutputClaim ClaimTypeReferenceId="trustFrameworkPolicy" Required="true" DefaultValue="{policy}" />
    ```

    Чтобы использовать ACR, удалите элемент **AuthenticationContextReferenceClaimPattern**.

- **Утверждение субъекта (sub).** По умолчанию для этого параметра задано значение ObjectID. Если вы хотите установить значение `Not Supported`, замените эту строку:

    ```xml
    <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub" />
    ```

    на следующую:

    ```xml
    <OutputClaim ClaimTypeReferenceId="sub" />
    ```

::: zone-end

## <a name="provide-optional-claims-to-your-app"></a>Предоставление приложению дополнительных утверждений

Утверждения приложения — это значения, возвращаемые приложению. Обновите поток пользователя, чтобы он содержал нужные утверждения.

::: zone pivot="b2c-user-flow"

1. Выберите **Потоки пользователей (политики)** .
1. Откройте созданный ранее пользовательский поток.
1. Выберите элемент **Утверждения приложения**.
1. Выберите утверждения и атрибуты, которые необходимо отправить обратно в приложение.
1. Выберите команду **Сохранить**.

::: zone-end

::: zone pivot="b2c-custom-policy"

В качестве выходных данных [технического профиля политики проверяющей](relyingparty.md#technicalprofile) стороны используются значения, возвращаемые в приложение. Добавление исходящих утверждений выдает заявки в маркер после успешного пути взаимодействия пользователя и отправляется в приложение. Измените элемент технического профиля в разделе проверяющей стороны, чтобы добавить требуемые утверждения в качестве исходящего утверждения.

1. Откройте файл настраиваемой политики. Например, SignUpOrSignin.xml.
1. Найдите элемент OutputClaims. Добавьте OutputClaim, который необходимо включить в маркер. 
1. Задайте атрибуты исходящего утверждения. 

В следующем примере добавляется `accountBalance` утверждение. Утверждение Аккаунтбаланце отправляется в приложение в виде баланса. 

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="OpenIdConnect" />
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="displayName" />
      <OutputClaim ClaimTypeReferenceId="givenName" />
      <OutputClaim ClaimTypeReferenceId="surname" />
      <OutputClaim ClaimTypeReferenceId="email" />
      <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
      <OutputClaim ClaimTypeReferenceId="identityProvider" />
      <OutputClaim ClaimTypeReferenceId="tenantId" AlwaysUseDefaultValue="true" DefaultValue="{Policy:TenantObjectId}" />
      <!--Add the optional claims here-->
      <OutputClaim ClaimTypeReferenceId="accountBalance" DefaultValue="" PartnerClaimType="balance" />
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" />
  </TechnicalProfile>
</RelyingParty>
```

Элемент OutputClaim содержит следующие атрибуты:

- **Claimtypereferenceid соединяется** — идентификатор типа утверждения, который уже определен в разделе [ClaimsSchema](claimsschema.md) файла политики или родительского файла политики.
- **Партнерклаимтипе** — позволяет изменить имя утверждения в токене. 
- **DefaultValue** — значение по умолчанию. Можно также задать значение по умолчанию для [сопоставителя заявок](claim-resolver-overview.md), например идентификатор клиента.
- **Алвайсуседефаултвалуе** — принудительно использовать значение по умолчанию.

::: zone-end

## <a name="next-steps"></a>Дальнейшие действия

Изучите информацию о том, как [запросить маркеры доступа](access-tokens.md).