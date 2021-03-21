---
title: TrustFrameworkPolicy — Azure Active Directory B2C | Документация Майкрософт
description: Указание элемента TrustFrameworkPolicy в настраиваемой политике для Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 03/15/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 9bf1cc197a7d6977ccb6ef69e157d9f8a76a58d5
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103470728"
---
# <a name="trustframeworkpolicy"></a>TrustFrameworkPolicy

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Пользовательские политики представлены одним или несколькими XML-файлами, которые ссылаются друг на друга в иерархической цепочке. XML-элементы определяют элементы политики, такие как схема утверждений, преобразования утверждений, определения содержимого, поставщики утверждений, технические профили, путь взаимодействия пользователя и шаги оркестрации. Каждый файл политики определяется внутри элемента верхнего уровня **TrustFrameworkPolicy** в файле политики.

```xml
<TrustFrameworkPolicy
  xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
  xmlns:xsd="https://www.w3.org/2001/XMLSchema"
  xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
  PolicySchemaVersion="0.3.0.0"
  TenantId="yourtenant.onmicrosoft.com"
  PolicyId="B2C_1A_TrustFrameworkBase"
  PublicPolicyUri="http://yourtenant.onmicrosoft.com/B2C_1A_TrustFrameworkBase">
  ...
```


Элемент **TrustFrameworkPolicy** содержит следующие атрибуты:

| Атрибут | Обязательно | Описание |
|---------- | -------- | ----------- |
| PolicySchemaVersion | Да | Версия схемы, которая будет использоваться для выполнения политики. Значение должно быть `0.3.0.0`. |
| TenantObjectId | Нет | Уникальный идентификатор объекта для Azure Active Directory B2C клиента (Azure AD B2C). |
| TenantId | Да | Уникальный идентификатор арендатора, к которому относится эта политика. |
| PolicyId | Да | Уникальный идентификатор политики. Этот идентификатор должен начинаться с префикса *B2C_1A_* |
| PublicPolicyUri | Да | Универсальный код ресурса (URI) для политики, который является сочетанием идентификатора арендатора и идентификатора политики. |
| DeploymentMode | Нет | Возможные значения: `Production` или `Development` . Значение по умолчанию — `Production`. Это свойство используется при отладке политики. Дополнительные сведения см. в разделе [Сбор журналов](troubleshoot-with-application-insights.md). |
| UserJourneyRecorderEndpoint | Нет | Конечная точка, используемая для ведения журнала. Значение должно быть равно, `urn:journeyrecorder:applicationinsights` Если атрибут существует. Дополнительные сведения см. в разделе [Сбор журналов](troubleshoot-with-application-insights.md). |


В следующем примере показано, как указать элемент **TrustFrameworkPolicy**:

``` XML
<TrustFrameworkPolicy
   xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
   xmlns:xsd="https://www.w3.org/2001/XMLSchema"
   xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
   PolicySchemaVersion="0.3.0.0"
   TenantId="yourtenant.onmicrosoft.com"
   PolicyId="B2C_1A_TrustFrameworkBase"
   PublicPolicyUri="http://yourtenant.onmicrosoft.com/B2C_1A_TrustFrameworkBase">
```

Элемент **TrustFrameworkPolicy** содержит следующие элементы:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| басеполици| 0:1| Идентификатор базовой политики. |
| [BuildingBlocks](buildingblocks.md) | 0:1 | Стандартные блоки политики. |
| [ClaimsProviders](claimsproviders.md) | 0:1 | Коллекция поставщиков утверждений. |
| [UserJourneys](userjourneys.md) | 0:1 | Коллекция путей взаимодействия пользователя. |
| [RelyingParty](relyingparty.md) | 0:1 | Определение политики проверяющей стороны. |

Чтобы наследовать политику от другой политики, необходимо объявить элемент **BasePolicy** в элементе **TrustFrameworkPolicy** файла политики. Элемент **BasePolicy** представляет собой ссылку на базовую политику, на основе которой создается эта политика.

В элементе **BasePolicy** содержатся следующие элементы:

| Элемент | Вхождения | Описание |
| ------- | ----------- | --------|
| TenantId | 1:1 | Идентификатор арендатора Azure AD B2C. |
| PolicyId | 1:1 | Идентификатор родительской политики. |


В следующем примере показано, как задать базовую политику. Политика **B2C_1A_TrustFrameworkExtensions** является производной от **B2C_1A_TrustFrameworkBase**.

``` XML
<TrustFrameworkPolicy
   xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
   xmlns:xsd="https://www.w3.org/2001/XMLSchema"
   xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
   PolicySchemaVersion="0.3.0.0"
   TenantId="yourtenant.onmicrosoft.com"
   PolicyId="B2C_1A_TrustFrameworkExtensions"
   PublicPolicyUri="http://yourtenant.onmicrosoft.com/B2C_1A_TrustFrameworkExtensions">

  <BasePolicy>
    <TenantId>yourtenant.onmicrosoft.com</TenantId>
    <PolicyId>B2C_1A_TrustFrameworkBase</PolicyId>
  </BasePolicy>
  ...
</TrustFrameworkPolicy>
```

