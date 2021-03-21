---
title: Мониторинг поведения пользователей с помощью Application Insights
titleSuffix: Azure AD B2C
description: Сведения о том, как включить журналы событий в Application Insights Azure AD B2C пути взаимодействия пользователя.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.date: 01/29/2021
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 92da0b12a3119b048866eef5b18f658916595294
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101645931"
---
# <a name="track-user-behavior-in-azure-ad-b2c-by-using-application-insights"></a>Мониторинг поведения пользователей в Azure AD B2C с помощью Application Insights

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-user-flow"

[!INCLUDE [active-directory-b2c-limited-to-custom-policy](../../includes/active-directory-b2c-limited-to-custom-policy.md)]

::: zone-end

::: zone pivot="b2c-custom-policy"

В Azure Active Directory B2C (Azure AD B2C) данные событий можно отправить непосредственно в [Application Insights](../azure-monitor/app/app-insights-overview.md) с помощью ключа инструментирования, предоставленного для Azure AD B2C. С помощью Application Insights технического профиля можно получить подробные и настраиваемые журналы событий для пути взаимодействия пользователя:

- Получить сведения о поведении пользователей.
- Устранить проблемы в собственных политиках в среде разработки или в рабочей среде.
- Измерить производительность.
- Создать уведомления из Application Insights.

## <a name="overview"></a>Обзор

Чтобы включить пользовательские журналы событий, добавьте Application Insights технический профиль. В техническом профиле вы определяете ключ инструментирования Application Insights, имя события и утверждения для записи. Чтобы опубликовать событие, добавьте технический профиль в качестве этапа оркестрации в [пути взаимодействия пользователя](userjourneys.md).

При использовании Application Insights учитывайте следующее.

- В Application Insights могут быть заданы небольшие задержки, которые обычно меньше пяти минут, прежде чем новые журналы будут доступны.
- Azure AD B2C позволяет выбрать утверждения для записи. Не включайте утверждения с личными данными.
- Для записи сеанса пользователя можно использовать идентификатор корреляции для унификации событий.
- Вызывайте Application Insights технический профиль непосредственно из [пути взаимодействия пользователя](userjourneys.md) или из [ветви](subjourneys.md). Не используйте Application Insights технический профиль в качестве [технического профиля проверки](validation-technical-profile.md).

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [active-directory-b2c-customization-prerequisites-custom-policy](../../includes/active-directory-b2c-customization-prerequisites-custom-policy.md)]

## <a name="create-an-application-insights-resource"></a>Создание ресурса Application Insights

При использовании Application Insights с Azure AD B2C необходимо всего лишь создать ресурс и получить ключ инструментирования. Дополнительные сведения см. [в разделе создание Application Insights ресурса](../azure-monitor/app/create-new-resource.md).

1. Войдите на [портал Azure](https://portal.azure.com/).
1. Убедитесь, что вы используете каталог с подпиской Azure. Выберите фильтр **каталог и подписка** в верхнем меню и выберите каталог, содержащий подписку Azure. Этот клиент не является клиентом Azure AD B2C.
1. Выберите **создать ресурс** в левом верхнем углу портал Azure, а затем найдите и выберите **Application Insights**.
1. Нажмите кнопку **создания**.
1. В поле **имя** введите имя ресурса.
1. Из списка **Application type** (Тип приложения) выберите **Веб-приложение ASP.NET**.
1. Для **Группы ресурсов** выберите существующую группу или укажите имя, чтобы создать новую.
1. Нажмите кнопку **создания**.
1. Откройте новый ресурс Application Insights, разверните раздел **Essentials** и скопируйте ключ инструментирования.

![Снимок экрана, на котором показан ключ инструментирования на вкладке обзора Application Insights.](./media/analytics-with-application-insights/app-insights.png)

## <a name="define-claims"></a>Определение утверждений

Утверждение предоставляет временное хранилище данных во время выполнения политики Azure AD B2C. Утверждения объявляются в [элементе ClaimsSchema](claimsschema.md).

1. Откройте файл расширения для политики, Файл может выглядеть примерно так `SocialAndLocalAccounts/` **`TrustFrameworkExtensions.xml`** :.
1. Найдите элемент [BuildingBlocks](buildingblocks.md). Если элемент не отображается, добавьте его.
1. Найдите элемент **ClaimsSchema** . Если элемент не отображается, добавьте его.
1. Добавьте следующие утверждения в элемент **ClaimsSchema** :

   ```xml
   <ClaimType Id="EventType">
     <DisplayName>Event type</DisplayName>
     <DataType>string</DataType>
   </ClaimType>
   <ClaimType Id="EventTimestamp">
     <DisplayName>Event timestamp</DisplayName>
     <DataType>string</DataType>
   </ClaimType>
   <ClaimType Id="PolicyId">
     <DisplayName>Policy Id</DisplayName>
     <DataType>string</DataType>
   </ClaimType>
   <ClaimType Id="Culture">
     <DisplayName>Culture ID</DisplayName>
     <DataType>string</DataType>
   </ClaimType>
   <ClaimType Id="CorrelationId">
     <DisplayName>Correlation Id</DisplayName>
     <DataType>string</DataType>
   </ClaimType>
   <ClaimType Id="federatedUser">
     <DisplayName>Federated user</DisplayName>
     <DataType>boolean</DataType>
   </ClaimType>
   <ClaimType Id="parsedDomain">
     <DisplayName>Domain name</DisplayName>
     <DataType>string</DataType>
     <UserHelpText>The domain portion of the email address.</UserHelpText>
   </ClaimType>
   <ClaimType Id="userInLocalDirectory">
     <DisplayName>userInLocalDirectory</DisplayName>
     <DataType>boolean</DataType>
   </ClaimType>
   ```

## <a name="add-new-technical-profiles"></a>Добавление новых технических профилей

Технические профили можно рассматривать как функции в пользовательской политике. Эти функции используют технический подход к [включению профиля](technicalprofiles.md#include-technical-profile) , где технический профиль включает еще один технический профиль и изменяет параметры или добавляет новые функции. В следующей таблице определены технические профили, используемые для открытия сеанса и события POST.

| Технический профиль | Задача |
| ----------------- | -----|
| AppInsights-Common | Общий технический профиль с типичной конфигурацией. Он включает Application Insights ключ инструментирования, коллекцию заявок для записи и режим разработчика. Другие технические профили включают в себя общий технический профиль и добавляют дополнительные заявки, например имя события. |
| AppInsights-SignInRequest | Записывает событие **сигнинрекуест** с набором утверждений при получении запроса на вход. |
| AppInsights-UserSignUp | Записывает событие **UserSignUp** , когда пользователь запускает функцию регистрации в пути регистрации или входа. |
| AppInsights-SignInComplete | Записывает событие **сигнинкомплете** после успешной проверки подлинности, когда маркер был отправлен в приложение проверяющей стороны. |

Откройте файл *TrustFrameworkExtensions.xml* из начального пакета. Добавьте технические профили в элемент **поставщика утверждений** :

```xml
<ClaimsProvider>
  <DisplayName>Application Insights</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="AppInsights-Common">
      <DisplayName>Application Insights</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.Insights.AzureApplicationInsightsProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <!-- The ApplicationInsights instrumentation key, which you use for logging the events -->
        <Item Key="InstrumentationKey">xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx</Item>
        <Item Key="DeveloperMode">false</Item>
        <Item Key="DisableTelemetry ">false</Item>
      </Metadata>
      <InputClaims>
        <!-- Properties of an event are added through the syntax {property:NAME}, where NAME is the property being added to the event. DefaultValue can be either a static value or a value that's resolved by one of the supported DefaultClaimResolvers. -->
        <InputClaim ClaimTypeReferenceId="EventTimestamp" PartnerClaimType="{property:EventTimestamp}" DefaultValue="{Context:DateTimeInUtc}" />
        <InputClaim ClaimTypeReferenceId="tenantId" PartnerClaimType="{property:TenantId}" DefaultValue="{Policy:TrustFrameworkTenantId}" />
        <InputClaim ClaimTypeReferenceId="PolicyId" PartnerClaimType="{property:Policy}" DefaultValue="{Policy:PolicyId}" />
        <InputClaim ClaimTypeReferenceId="CorrelationId" PartnerClaimType="{property:CorrelationId}" DefaultValue="{Context:CorrelationId}" />
        <InputClaim ClaimTypeReferenceId="Culture" PartnerClaimType="{property:Culture}" DefaultValue="{Culture:RFC5646}" />
      </InputClaims>
    </TechnicalProfile>

    <TechnicalProfile Id="AppInsights-SignInRequest">
      <InputClaims>
        <!-- An input claim with a PartnerClaimType="eventName" is required. This is used by the AzureApplicationInsightsProvider to create an event with the specified value. -->
        <InputClaim ClaimTypeReferenceId="EventType" PartnerClaimType="eventName" DefaultValue="SignInRequest" />
      </InputClaims>
      <IncludeTechnicalProfile ReferenceId="AppInsights-Common" />
    </TechnicalProfile>

    <TechnicalProfile Id="AppInsights-UserSignUp">
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="EventType" PartnerClaimType="eventName" DefaultValue="UserSignUp" />
      </InputClaims>
      <IncludeTechnicalProfile ReferenceId="AppInsights-Common" />
    </TechnicalProfile>
    
    <TechnicalProfile Id="AppInsights-SignInComplete">
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="EventType" PartnerClaimType="eventName" DefaultValue="SignInComplete" />
        <InputClaim ClaimTypeReferenceId="federatedUser" PartnerClaimType="{property:FederatedUser}" DefaultValue="false" />
        <InputClaim ClaimTypeReferenceId="parsedDomain" PartnerClaimType="{property:FederationPartner}" DefaultValue="Not Applicable" />
        <InputClaim ClaimTypeReferenceId="identityProvider" PartnerClaimType="{property:IDP}" DefaultValue="Local" />
      </InputClaims>
      <IncludeTechnicalProfile ReferenceId="AppInsights-Common" />
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

> [!IMPORTANT]
> Замените ключ инструментирования в техническом профиле `AppInsights-Common` идентификатором GUID, который предоставляется ресурсом Application Insights.

## <a name="add-the-technical-profiles-as-orchestration-steps"></a>Добавление технических профилей в качестве шагов оркестрации

Добавьте новые шаги оркестрации, которые относятся к техническим профилям.

> [!IMPORTANT]
> После добавления новых шагов оркестрации повторно последовательно пронумеруйте шаги, не пропуская ни одного целого числа от 1 до N.

1. Вызовите метод `AppInsights-SignInRequest` как второй этап оркестрации. Этот шаг отслеживает получение запроса на регистрацию или вход.

   ```xml
   <!-- Track that we have received a sign in request -->
   <OrchestrationStep Order="2" Type="ClaimsExchange">
     <ClaimsExchanges>
       <ClaimsExchange Id="TrackSignInRequest" TechnicalProfileReferenceId="AppInsights-SignInRequest" />
     </ClaimsExchanges>
   </OrchestrationStep>
   ```

1. Перед `SendClaims` этапом оркестрации добавьте новый шаг, который вызывает `AppInsights-UserSignup` . Он срабатывает, когда пользователь нажимает кнопку регистрации в пути регистрации или входа.

   ```xml
   <!-- Handles the user selecting the sign-up link in the local account sign-in page -->
   <OrchestrationStep Order="8" Type="ClaimsExchange">
     <Preconditions>
       <Precondition Type="ClaimsExist" ExecuteActionsIf="false">
         <Value>newUser</Value>
         <Action>SkipThisOrchestrationStep</Action>
       </Precondition>
       <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
         <Value>newUser</Value>
         <Value>false</Value>
         <Action>SkipThisOrchestrationStep</Action>
       </Precondition>
     </Preconditions>
     <ClaimsExchanges>
       <ClaimsExchange Id="TrackUserSignUp" TechnicalProfileReferenceId="AppInsights-UserSignup" />
     </ClaimsExchanges>
   </OrchestrationStep>
   ```

1. После `SendClaims` шага оркестрации вызовите `AppInsights-SignInComplete` . Этот шаг указывает на успешное завершение пути взаимодействия.

   ```xml
   <!-- Track that we have successfully sent a token -->
   <OrchestrationStep Order="10" Type="ClaimsExchange">
     <ClaimsExchanges>
       <ClaimsExchange Id="TrackSignInComplete" TechnicalProfileReferenceId="AppInsights-SignInComplete" />
     </ClaimsExchanges>
   </OrchestrationStep>
   ```

## <a name="upload-your-file-run-the-policy-and-view-events"></a>Отправка файла, выполнение политики и просмотр событий

Сохраните и отправьте файл *TrustFrameworkExtensions.xml*. Затем вызовите политику проверяющей стороны из приложения или используйте **команду Запустить сейчас** в портал Azure. Дождитесь, пока события будут доступны в Application Insights.

1. Откройте **Application Insights** ресурс в клиенте Azure Active Directory.
1. Выберите **Использование**, а затем выберите **события**.
1. Установите для поля **Во время** значение **Последний час**, а для поля **By** (До) — **3 минуты**. Может потребоваться обновить окно, чтобы увидеть результаты.

![Снимок экрана, показывающий Application Insights статистику событий.](./media/analytics-with-application-insights/app-ins-graphic.png)

## <a name="collect-more-data"></a>Сбор дополнительных данных

В соответствии с потребностями бизнеса может потребоваться записать больше заявок. Чтобы добавить утверждение, сначала [Определите утверждение](#define-claims), а затем добавьте утверждение в коллекцию входных утверждений. Утверждения, добавляемые в технический профиль **AppInsights-Common** , отображаются во всех событиях. Утверждения, добавляемые в конкретный технический профиль, отображаются только в этом событии. Элемент входящего утверждения содержит следующие атрибуты:

- **ClaimTypeReferenceId** — это ссылка на тип утверждения.
- **PartnerClaimType** — имя свойства, отображаемое в Azure Insights. Используйте синтаксис `{property:NAME}` , где `NAME` — это свойство, добавляемое к событию.
- **DefaultValue** — это предопределенное значение, которое необходимо записать, например имя события. Если утверждение, используемое в пути взаимодействия пользователя, является пустым, используется значение по умолчанию. Например, `identityProvider` утверждение задается техническими профилями Федерации, такими как Facebook. Если утверждение пустое, оно указывает, что пользователь вошел в локальную учетную запись. Таким же значением по умолчанию задается значение **Local**. Можно также записать [сопоставитель утверждений](claim-resolver-overview.md) с контекстным значением, таким как идентификатор приложения или IP-адрес пользователя.

### <a name="manipulate-claims"></a>Обработка утверждений

[Преобразования входящих утверждений](custom-policy-trust-frameworks.md#manipulating-your-claims) можно использовать для изменения входных утверждений или создания новых перед их отправкой в Application Insights. В следующем примере технический профиль включает `CheckIsAdmin` Преобразование «входные утверждения».

```xml
<TechnicalProfile Id="AppInsights-SignInComplete">
  <InputClaimsTransformations>  
    <InputClaimsTransformation ReferenceId="CheckIsAdmin" />
  </InputClaimsTransformations>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="isAdmin" PartnerClaimType="{property:IsAdmin}"  />
    ...
  </InputClaims>
  <IncludeTechnicalProfile ReferenceId="AppInsights-Common" />
</TechnicalProfile>
```

### <a name="add-events"></a>Добавление событий

Чтобы добавить событие, создайте новый технический профиль, включающий `AppInsights-Common` технический профиль. Затем добавьте новый технический профиль в качестве этапа оркестрации для [пути взаимодействия пользователя](custom-policy-trust-frameworks.md#orchestration-steps). Используйте элемент [предусловия](userjourneys.md#preconditions) , чтобы активировать событие, когда будете готовы. Например, сообщите о событии только в том случае, если пользователи работают с использованием многофакторной проверки подлинности.

```xml
<TechnicalProfile Id="AppInsights-MFA-Completed">
  <InputClaims>
     <InputClaim ClaimTypeReferenceId="EventType" PartnerClaimType="eventName" DefaultValue="MFA-Completed" />
  </InputClaims>
  <IncludeTechnicalProfile ReferenceId="AppInsights-Common" />
</TechnicalProfile>
```

>[!Important]
>При добавлении события к пути взаимодействия пользователя не забывайте последовательно перенумеровать шаги оркестрации.

```xml
<OrchestrationStep Order="8" Type="ClaimsExchange">
  <Precondition Type="ClaimsExist" ExecuteActionsIf="true">
    <Value>isActiveMFASession</Value>
    <Action>SkipThisOrchestrationStep</Action>
    </Precondition>
  </Preconditions>
  <ClaimsExchanges>
    <ClaimsExchange Id="TrackUserMfaCompleted" TechnicalProfileReferenceId="AppInsights-MFA-Completed" />
  </ClaimsExchanges>
</OrchestrationStep>
```

## <a name="enable-developer-mode"></a>Включение режима разработчика

При использовании Application Insights для определения событий можно указать, включен ли режим разработчика. Режим разработчика определяет способ буферизации событий. В среде разработки с минимальным объемом событий включение режима разработчика приводит к тому, что события немедленно отправляются на Application Insights. Значение по умолчанию — `false`. Не включайте режим разработчика в рабочих средах.

Чтобы включить режим разработчика, измените `DeveloperMode` метаданные на `true` в `AppInsights-Common` техническом профиле:

```xml
<TechnicalProfile Id="AppInsights-Common">
  <Metadata>
    ...
    <Item Key="DeveloperMode">true</Item>
  </Metadata>
</TechnicalProfile>
```

## <a name="disable-telemetry"></a>Отключение данных телеметрии

Чтобы отключить журналы Application Insights, измените `DisableTelemetry` метаданные на `true` в `AppInsights-Common` техническом профиле:

```xml
<TechnicalProfile Id="AppInsights-Common">
  <Metadata>
    ...
    <Item Key="DisableTelemetry">true</Item>
  </Metadata>
</TechnicalProfile>
```

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как [создавать настраиваемые панели мониторинга ключевых показателей эффективности с помощью Azure Application Insights](../azure-monitor/app/tutorial-app-dashboards.md).

::: zone-end