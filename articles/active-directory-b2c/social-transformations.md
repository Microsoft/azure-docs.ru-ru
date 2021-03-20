---
title: Примеры преобразования заявок на учетную запись социальных сетей для пользовательских политик
titleSuffix: Azure AD B2C
description: Примеры преобразования заявок на учетную запись социальных сетей (инфраструктура процедур идентификации) для схемы Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/10/2018
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: c60cecb9d6bfeeefc0569a1a57185d13f0c6442f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94953123"
---
# <a name="social-accounts-claims-transformations"></a>Преобразования утверждений учетных записей социальных сетей

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

В Azure Active Directory B2C (Azure AD B2C) идентификаторы учетных записей социальных сетей хранятся в `userIdentities` атрибуте типа утверждения **алтернативесекуритидколлектион** . Каждый элемент в утверждении **alternativeSecurityIdCollection** задает издателя (имя поставщика удостоверений, например facebook.com) и `issuerUserId` — уникальный идентификатор пользователя для издателя.

```json
"userIdentities": [{
    "issuer": "google.com",
    "issuerUserId": "MTA4MTQ2MDgyOTI3MDUyNTYzMjcw"
  },
  {
    "issuer": "facebook.com",
    "issuerUserId": "MTIzNDU="
  }]
```

В этой статье приведены примеры использования преобразований утверждений учетных записей социальных сетей для схемы инфраструктуры процедур идентификации в Azure AD B2C. Дополнительные сведения см. в статье о [преобразовании утверждений](claimstransformations.md).

## <a name="createalternativesecurityid"></a>CreateAlternativeSecurityId

Создает представление JSON свойства alternativeSecurityId пользователя, которое можно использовать в вызовах Azure Active Directory. Дополнительные сведения см. в схеме [AlternativeSecurityId](/graph/api/resources/alternativesecurityid) .

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | ключ | строка | Тип ClaimType, который указывает уникальный идентификатор пользователя, используемый поставщиком удостоверений социальных сетей. |
| InputClaim | identityProvider | строка | Тип ClaimType, который указывает имя поставщика удостоверений учетных записей социальных сетей, например facebook.com. |
| outputClaim | alternativeSecurityId | строка | Тип ClaimType, который создается после вызова ClaimsTransformation. Содержит информацию об удостоверении пользователя учетной записи социальной сети. **issuer** — значение утверждения `identityProvider`. **issuerUserId** — значение утверждения `key` в формате base64. |

Это преобразование утверждений позволяет задать ClaimType для `alternativeSecurityId`. Преобразование используют все технические профили поставщиков удостоверений социальных сетей, например `Facebook-OAUTH`. Следующее преобразование утверждений позволяет получить идентификатор учетной записи пользователя социальной сети и имя поставщика удостоверений. Выходные данные этого технического профиля представляют собой строку в формате JSON, которую можно использовать в службах каталогов Azure AD.

```xml
<ClaimsTransformation Id="CreateAlternativeSecurityId" TransformationMethod="CreateAlternativeSecurityId">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="issuerUserId" TransformationClaimType="key" />
    <InputClaim ClaimTypeReferenceId="identityProvider" TransformationClaimType="identityProvider" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="alternativeSecurityId" TransformationClaimType="alternativeSecurityId" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
    - **key**: 12334
    - **identityProvider**: Facebook.com
- Исходящие утверждения:
    - **alternativeSecurityId**: { "issuer": "facebook.com", "issuerUserId": "MTA4MTQ2MDgyOTI3MDUyNTYzMjcw"}

## <a name="additemtoalternativesecurityidcollection"></a>AddItemToAlternativeSecurityIdCollection

Добавляет `AlternativeSecurityId` к утверждению `alternativeSecurityIdCollection`.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | элемент | строка | Тип ClaimType, добавляемый к исходящему утверждению. |
| InputClaim | коллекция | alternativeSecurityIdCollection | Типы ClaimType, используемые при преобразовании утверждений (если типы доступны в политике). Если это значение указано, при преобразовании утверждений в конец коллекции добавляется `item`. |
| outputClaim | коллекция | alternativeSecurityIdCollection | Типы ClaimType, создаваемые после вызова этого преобразования ClaimsTransformation. Новая коллекция, содержащая как элементы из входного элемента `collection`, так и из `item`. |

В приведенном ниже примере новое удостоверение социальной сети связывается с существующей учетной записью. Чтобы связать новое удостоверение социальной сети с существующей учетной записью, выполните следующие действия:
1. В технических профилях **AAD-UserReadUsingAlternativeSecurityId** и **AAD-UserReadUsingObjectId** выведите утверждение пользователя **alternativeSecurityIds**.
1. Попросите пользователя выполнить вход с помощью одного из поставщиков удостоверений, которые не связаны с этим пользователем.
1. Выполните преобразование утверждений **CreateAlternativeSecurityId**, чтобы создать новый тип утверждения **alternativeSecurityId** с именем `AlternativeSecurityId2`.
1. Вызовите преобразование утверждений **AddItemToAlternativeSecurityIdCollection**, чтобы добавить утверждение **AlternativeSecurityId2** к существующему утверждению **AlternativeSecurityIds**.
1. Сохраните утверждение **alternativeSecurityIds** в учетной записи пользователя.

```xml
<ClaimsTransformation Id="AddAnotherAlternativeSecurityId" TransformationMethod="AddItemToAlternativeSecurityIdCollection">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="AlternativeSecurityId2" TransformationClaimType="item" />
    <InputClaim ClaimTypeReferenceId="AlternativeSecurityIds" TransformationClaimType="collection" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="AlternativeSecurityIds" TransformationClaimType="collection" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
    - **item**: { "issuer": "facebook.com", "issuerUserId": "MTIzNDU=" }
    - **collection**: [ { "issuer": "live.com", "issuerUserId": "MTA4MTQ2MDgyOTI3MDUyNTYzMjcw" } ]
- Исходящие утверждения:
    - **collection**: [ { "issuer": "live.com", "issuerUserId": "MTA4MTQ2MDgyOTI3MDUyNTYzMjcw" }, { "issuer": "facebook.com", "issuerUserId": "MTIzNDU=" } ]

## <a name="getidentityprovidersfromalternativesecurityidcollectiontransformation"></a>GetIdentityProvidersFromAlternativeSecurityIdCollectionTransformation

Возвращает список издателей из утверждения **alternativeSecurityIdCollection** в новое утверждение **stringCollection**.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | alternativeSecurityIdCollection | alternativeSecurityIdCollection | Тип ClaimType, используемый для получения списка поставщиков удостоверений (издатель). |
| outputClaim | identityProvidersCollection | stringCollection | Типы ClaimType, создаваемые после вызова этого преобразования ClaimsTransformation. Список поставщиков удостоверений, связанных со входящим утверждением alternativeSecurityIdCollection |

Указанное ниже преобразование утверждений позволяет считать утверждение пользователя **alternativeSecurityIds** и извлечь список имен поставщиков удостоверений, связанных с этой учетной записью. Используйте выходную коллекцию **identityProvidersCollection**, чтобы для пользователя отобразился список поставщиков удостоверений, связанный с учетной записью. Или же на странице выбора поставщика удостоверений отфильтруйте список поставщиков удостоверений на основе исходящего утверждения **identityProvidersCollection**. Таким образом, пользователь по выбору может связать с учетной записью новое удостоверение социальной сети, которое с ней еще не связано.

```xml
<ClaimsTransformation Id="ExtractIdentityProviders" TransformationMethod="GetIdentityProvidersFromAlternativeSecurityIdCollectionTransformation">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="alternativeSecurityIds" TransformationClaimType="alternativeSecurityIdCollection" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="identityProviders" TransformationClaimType="identityProvidersCollection" />
  </OutputClaims>
</ClaimsTransformation>
```

- Входящие утверждения:
    - **alternativeSecurityIdCollection**: [ { "issuer": "google.com", "issuerUserId": "MTA4MTQ2MDgyOTI3MDUyNTYzMjcw" }, { "issuer": "facebook.com", "issuerUserId": "MTIzNDU=" } ]
- Исходящие утверждения:
    - **identityProvidersCollection**: [ "facebook.com", "google.com" ]

## <a name="removealternativesecurityidbyidentityprovider"></a>RemoveAlternativeSecurityIdByIdentityProvider

Удаляет **AlternativeSecurityId** из утверждения **alternativeSecurityIdCollection**.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | identityProvider | строка | Тип ClaimType, который содержит имя поставщика удостоверений, удаляемое из коллекции. |
| InputClaim | коллекция | alternativeSecurityIdCollection | Типы ClaimType, используемые в преобразовании утверждений. При преобразовании утверждений identityProvider удаляется из коллекции. |
| outputClaim | коллекция | alternativeSecurityIdCollection | Типы ClaimType, создаваемые после вызова этого преобразования ClaimsTransformation. Новая коллекция после удаления identityProvider из коллекции. |

В приведенном ниже примере удаляется связь одного из удостоверений социальных сетей с существующей учетной записью. Чтобы отменить связь удостоверения социальной сети, выполните следующие действия:
1. В технических профилях **AAD-UserReadUsingAlternativeSecurityId** и **AAD-UserReadUsingObjectId** выведите утверждение пользователя **alternativeSecurityIds**.
2. Попросите пользователя выбрать учетную запись социальной сети, которая будет удалена из списка поставщиков удостоверений, связанных с этим пользователем.
3. Вызовите технический профиль преобразования утверждений, который вызывает преобразование утверждений **RemoveAlternativeSecurityIdByIdentityProvider**, при котором было удалено выбранное удостоверение социальной сети, с использованием имени поставщика удостоверений.
4. Сохранить утверждение **alternativeSecurityIds** в учетной записи пользователя.

```xml
<ClaimsTransformation Id="RemoveAlternativeSecurityIdByIdentityProvider" TransformationMethod="RemoveAlternativeSecurityIdByIdentityProvider">
    <InputClaims>
        <InputClaim ClaimTypeReferenceId="secondIdentityProvider" TransformationClaimType="identityProvider" />
        <InputClaim ClaimTypeReferenceId="AlternativeSecurityIds" TransformationClaimType="collection" />
    </InputClaims>
    <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="AlternativeSecurityIds" TransformationClaimType="collection" />
    </OutputClaims>
</ClaimsTransformation>
</ClaimsTransformations>
```

### <a name="example"></a>Пример

- Входящие утверждения:
    - **identityProvider**: facebook.com
    - **collection**: [ { "issuer": "live.com", "issuerUserId": "MTA4MTQ2MDgyOTI3MDUyNTYzMjcw" }, { "issuer": "facebook.com", "issuerUserId": "MTIzNDU=" } ]
- Исходящие утверждения:
    - **collection**: [ { "issuer": "live.com", "issuerUserId": "MTA4MTQ2MDgyOTI3MDUyNTYzMjcw" } ]