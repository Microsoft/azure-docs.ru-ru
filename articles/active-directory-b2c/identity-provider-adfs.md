---
title: Добавление AD FS в качестве поставщика удостоверений SAML с помощью настраиваемых политик
titleSuffix: Azure AD B2C
description: Настройка AD FS 2016 с помощью протокола SAML и пользовательских политик в Azure Active Directory B2C
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/15/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 292a244a4804f97e8622d6841c33b153af373290
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103489174"
---
# <a name="add-ad-fs-as-a-saml-identity-provider-using-custom-policies-in-azure-active-directory-b2c"></a>Добавление AD FS в качестве поставщика удостоверений SAML с помощью пользовательских политик в Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-user-flow"

[!INCLUDE [active-directory-b2c-limited-to-custom-policy](../../includes/active-directory-b2c-limited-to-custom-policy.md)]

::: zone-end

::: zone pivot="b2c-custom-policy"

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

В этой статье показано, как включить вход для учетной записи AD FS пользователя с помощью [пользовательских политик](custom-policy-overview.md) в Azure Active Directory B2C (Azure AD B2C). Чтобы включить вход, добавьте [поставщик удостоверений SAML](identity-provider-generic-saml.md) в настраиваемую политику.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [active-directory-b2c-customization-prerequisites-custom-policy](../../includes/active-directory-b2c-customization-prerequisites-custom-policy.md)]

## <a name="create-a-self-signed-certificate"></a>Создание самозаверяющего сертификата

[!INCLUDE [active-directory-b2c-create-self-signed-certificate](../../includes/active-directory-b2c-create-self-signed-certificate.md)]

## <a name="create-a-policy-key"></a>Создание ключа политики

Вам необходимо сохранить сертификат в клиенте Azure AD B2C.

1. Войдите на [портал Azure](https://portal.azure.com/).
2. Убедитесь, что вы используете каталог, содержащий клиент Azure AD B2C. Выберите фильтр **Каталог и подписка** в верхнем меню и выберите каталог, который содержит ваш клиент.
3. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
4. На странице "Обзор" выберите **Identity Experience Framework**.
5. Выберите **Ключи политики**, а затем щелкните **Добавить**.
6. Для пункта **Параметры** выберите `Upload`.
7. Введите **имя** ключа политики. Например, `SAMLSigningCert`. Префикс `B2C_1A_` будет автоматически добавлен к имени ключа.
8. Найдите и выберите PFX-файл сертификата с закрытым ключом.
9. Нажмите кнопку **Создать**.

## <a name="add-a-claims-provider"></a>Добавление поставщика утверждений

Если вы хотите, чтобы пользователи вошли с помощью учетной записи AD FS, необходимо определить учетную запись как поставщика утверждений, который Azure AD B2C может взаимодействовать с помощью конечной точки. Конечная точка предоставляет набор утверждений, используемых Azure AD B2C, чтобы проверить, была ли выполнена проверка подлинности определенного пользователя.

Вы можете определить учетную запись AD FS как поставщика утверждений, добавив ее в элемент **клаимспровидерс** в файле расширения политики. Дополнительные сведения см. [в разделе Определение поставщика удостоверений SAML](identity-provider-generic-saml.md).

1. Откройте файл *TrustFrameworkExtensions.xml*.
1. Найдите элемент **ClaimsProviders**. Если он не существует, добавьте его в корневой элемент.
1. Добавьте новый элемент **ClaimsProvider** следующим образом.

    ```xml
    <ClaimsProvider>
      <Domain>contoso.com</Domain>
      <DisplayName>Contoso</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Contoso-SAML2">
          <DisplayName>Contoso</DisplayName>
          <Description>Login with your AD FS account</Description>
          <Protocol Name="SAML2"/>
          <Metadata>
            <Item Key="WantsEncryptedAssertions">false</Item>
            <Item Key="PartnerEntity">https://your-AD-FS-domain/federationmetadata/2007-06/federationmetadata.xml</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="SamlMessageSigning" StorageReferenceId="B2C_1A_SAMLSigningCert"/>
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="userPrincipalName" />
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name"/>
            <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name"/>
            <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email"/>
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name"/>
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="contoso.com" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication"/>
          </OutputClaims>
          <OutputClaimsTransformations>
            <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName"/>
            <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName"/>
            <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId"/>
            <OutputClaimsTransformation ReferenceId="CreateSubjectClaimFromAlternativeSecurityId"/>
          </OutputClaimsTransformations>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-Saml-idp"/>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

1. Замените `your-AD-FS-domain` именем домена AD FS и замените значение исходящего утверждения **IDENTITYPROVIDER** на DNS (произвольное значение, указывающее ваш домен).

1. Откройте раздел `<ClaimsProviders>` и добавьте следующий фрагмент кода XML. Если политика уже содержит технический профиль `SM-Saml-idp`, перейдите к следующему шагу. Дополнительные сведения см. в разделе об [управлении сеансом единого входа](custom-policy-reference-sso.md).

    ```xml
    <ClaimsProvider>
      <DisplayName>Session Management</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="SM-Saml-idp">
          <DisplayName>Session Management Provider</DisplayName>
          <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.SamlSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
          <Metadata>
            <Item Key="IncludeSessionIndex">false</Item>
            <Item Key="RegisterServiceProviders">false</Item>
          </Metadata>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

1. Сохраните файл.

[!INCLUDE [active-directory-b2c-add-identity-provider-to-user-journey](../../includes/active-directory-b2c-add-identity-provider-to-user-journey.md)]

```xml
<OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
  <ClaimsProviderSelections>
    ...
    <ClaimsProviderSelection TargetClaimsExchangeId="ContosoExchange" />
  </ClaimsProviderSelections>
  ...
</OrchestrationStep>

<OrchestrationStep Order="2" Type="ClaimsExchange">
  ...
  <ClaimsExchanges>
    <ClaimsExchange Id="ContosoExchange" TechnicalProfileReferenceId="Contoso-SAML2" />
  </ClaimsExchanges>
</OrchestrationStep>
```

[!INCLUDE [active-directory-b2c-configure-relying-party-policy](../../includes/active-directory-b2c-configure-relying-party-policy-user-journey.md)]

## <a name="configure-an-ad-fs-relying-party-trust"></a>Настройка отношения доверия с проверяющей стороной AD FS

Чтобы использовать AD FS в качестве поставщика удостоверений в Azure AD B2C, необходимо создать AD FS отношения доверия с проверяющей стороной с метаданными SAML Azure AD B2C. В указанном ниже примере показан URL-адрес метаданных SAML, связанных с техническим профилем Azure AD B2C.

```
https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com/your-policy/samlp/metadata?idptp=your-technical-profile
```

При использовании [пользовательского домена](custom-domain.md)используйте следующий формат:

```
https://your-domain-name/your-tenant-name.onmicrosoft.com/your-policy/samlp/metadata?idptp=your-technical-profile
```

Измените следующие значения:

- **имя** клиента с именем клиента, например Your-tenant.onmicrosoft.com.
- **имя вашего домена** с именем пользовательского домена, например Login.contoso.com.
- **your-policy** — именем собственной политики. Например, B2C_1A_signup_signin_adfs.
- **ваш-технический профиль** с именем технического профиля поставщика удостоверений SAML. Например, Contoso-SAML2.

Откройте браузер и перейдите по URL-адресу. Убедитесь, что ввели правильный URL-адрес и что имеете доступ к XML-файлу метаданных. Чтобы добавить новое отношение доверия с проверяющей стороной с помощью оснастки управления AD FS и вручную настроить параметры, выполните следующую процедуру на сервере федерации. Для выполнения этой процедуры требуется как минимум членство в группах **Администраторы** или эквивалент на локальном компьютере.

1. В диспетчер сервера выберите **инструменты**, а затем выберите **AD FS управление**.
2. Выберите **Добавить отношение доверия с проверяющей стороной**.
3. На странице **приветствия** выберите **Поддерживающие утверждения** и нажмите кнопку **Запустить**.
4. На странице **выбора источника данных** выберите параметр **импорта данных о проверяющей стороне, опубликованных в Интернете или локальной сети**, укажите URL-адрес метаданных Azure AD B2C и нажмите кнопку **Далее**.
5. На странице **Указание отображаемого имени** введите **отображаемое имя**. В поле **Примечания** введите описание для этого отношения доверия с проверяющей стороной и нажмите кнопку **Далее**.
6. На странице **Выбрать политику управления доступом** выберите политику и нажмите кнопку **Далее**.
7. На странице **Готовность для добавления отношения доверия** проверьте параметры, а затем нажмите кнопку **Далее**, чтобы сохранить сведения об отношениях доверия с проверяющей стороной.
8. На странице **Готово** щелкните **Закрыть**. При этом автоматически откроется диалоговое окно **Edit Claim Rules** (Изменение правил утверждений).
9. Выберите **Добавить правило**.
10. В разделе **Claim rule template** (Шаблон правила утверждения) выберите **Send LDAP attributes as claims** (Отправка атрибутов LDAP как утверждений).
11. Укажите **имя правила утверждения**. В поле **хранилище атрибутов** выберите **выбрать Active Directory**, добавьте следующие утверждения, а затем нажмите кнопку **Готово** и **ОК**.

    | Атрибут LDAP | Тип исходящего утверждения |
    | -------------- | ------------------- |
    | User-Principal-Name | userPrincipalName |
    | Surname | family_name |
    | Given-Name | given_name |
    | E-Mail-Address | email |
    | Display-Name | name |

    Обратите внимание, что эти имена не отображаются в раскрывающемся списке Тип исходящего утверждения. Их необходимо ввести вручную в. (Раскрывающийся список фактически доступен для редактирования).

12.  В зависимости от типа сертификата может потребоваться задать хэш-алгоритм. В окне свойств отношения доверия с проверяющей стороной (демоверсия B2C) перейдите на вкладку **Дополнительно**, измените **алгоритм SHA** на `SHA-256` и нажмите кнопку **ОК**.
13. В диспетчер сервера выберите **инструменты**, а затем выберите **AD FS управление**.
14. Выберите созданное отношение доверия проверяющей стороны, щелкните **Update from Federation Metadata** (Обновить с метаданных федерации), а затем выберите **Обновить**.

## <a name="test-your-custom-policy"></a>Тестирование настраиваемой политики

1. Войдите на [портал Azure](https://portal.azure.com).
1. Выберите значок **Каталог и подписка** в верхней панели инструментов портала, а затем выберите каталог, содержащий клиент Azure AD B2C.
1. На портале Azure найдите и выберите **Azure AD B2C**.
1. В разделе " **политики**" выберите **инфраструктура процедур идентификации** .
1. Выберите политику проверяющей стороны, например `B2C_1A_signup_signin` .
1. Для **приложения** выберите [ранее зарегистрированное](tutorial-register-applications.md)веб-приложение. В поле **URL-адрес ответа** должно содержаться значение `https://jwt.ms`.
1. Нажмите кнопку **Запустить сейчас** .
1. На странице регистрации или входа выберите **contoso AD FS** для входа с помощью поставщика удостоверений Contoso AD FS.

Если процесс входа прошел успешно, браузер перенаправляется на `https://jwt.ms` , который отображает содержимое маркера, возвращенного Azure AD B2C.

## <a name="troubleshooting-ad-fs-service"></a>Устранение неполадок службы AD FS  

AD FS настроен для использования журнала приложений Windows. При возникновении проблем с настройкой AD FS в качестве поставщика удостоверений SAML с помощью пользовательских политик в Azure AD B2C может потребоваться проверить журнал AD FS событий:

1. На **панели поиска** Windows введите **Просмотр событий**, а затем выберите классическое приложение **Просмотр событий** .
1. Чтобы просмотреть журнал с другого компьютера, щелкните правой кнопкой мыши элемент **Просмотр событий (локальных)** . Выберите пункт **Подключение к другому компьютеру** и заполните поля в диалоговом окне **Выбор компьютера**.
1. В **средстве просмотра событий** откройте **журналы приложений и служб**.
1. Выберите **AD FS**, а затем щелкните **Администратор**. 
1. Чтобы просмотреть дополнительные сведения о событии, дважды щелкните событие.  

### <a name="saml-request-is-not-signed-with-expected-signature-algorithm-event"></a>Запрос SAML не подписан с ожидаемым событием алгоритма подписи

Эта ошибка означает, что запрос SAML, отправленный Azure AD B2C, не подписан с использованием ожидаемого алгоритма подписи, настроенного в AD FS. Например, запрос SAML подписан с помощью алгоритма подписи `rsa-sha256` , но ожидаемым алгоритмом подписи является `rsa-sha1` . Чтобы устранить эту проблему, убедитесь, что для обеих Azure AD B2C и AD FS настроен один и тот же алгоритм подписи.

#### <a name="option-1-set-the-signature-algorithm-in-azure-ad-b2c"></a>Вариант 1. Задание алгоритма подписи в Azure AD B2C  

Вы можете настроить способ подписи запроса SAML в Azure AD B2C. Метаданные [ксмлсигнатуреалгорисм](identity-provider-generic-saml.md) контролирует значение `SigAlg` параметра (строка запроса или параметр POST) в запросе SAML. В следующем примере настраивается Azure AD B2C для использования `rsa-sha256` алгоритма подписи.

```xml
<Metadata>
  <Item Key="WantsEncryptedAssertions">false</Item>
  <Item Key="PartnerEntity">https://your-AD-FS-domain/federationmetadata/2007-06/federationmetadata.xml</Item>
  <Item Key="XmlSignatureAlgorithm">Sha256</Item>
</Metadata>
```

#### <a name="option-2-set-the-signature-algorithm-in-ad-fs"></a>Вариант 2. Задание алгоритма подписи в AD FS 

Кроме того, можно настроить ожидаемый алгоритм подписи запроса SAML в AD FS.

1. В диспетчер сервера выберите **инструменты**, а затем выберите **AD FS управление**.
1. Выберите ранее созданное **отношение доверия с проверяющей стороной** .
1. Выберите **Свойства**, а затем выберите **Предварительные**
1. Настройте **алгоритм SHA** и нажмите кнопку **ОК** , чтобы сохранить изменения.

::: zone-end
