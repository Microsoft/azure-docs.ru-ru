---
title: Пользовательская проверка электронной почты с помощью SendGrid
titleSuffix: Azure AD B2C
description: Узнайте, как интегрироваться с SendGrid, чтобы настроить электронное письмо с проверкой подлинности, отправляемое клиентам при регистрации для использования приложений с поддержкой Azure AD B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/10/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 7fdf1f8a433b6e88aff9a3670ea483f8d72681b4
ms.sourcegitcommit: b572ce40f979ebfb75e1039b95cea7fce1a83452
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "102631206"
---
# <a name="custom-email-verification-with-sendgrid"></a>Пользовательская проверка электронной почты с помощью SendGrid

Используйте настраиваемый адрес электронной почты в Azure Active Directory B2C (Azure AD B2C) для отправки настраиваемых сообщений электронной почты пользователям, которые подписываются на использование ваших приложений. С помощью [дисплайконтролс](display-controls.md) (в настоящее время в предварительной версии) и стороннего поставщика электронной почты SendGrid можно использовать собственный шаблон электронной почты и *адрес и* тему, а также поддержку локализации и настраиваемого одноразового пароля (OTP).

Для проверки пользовательской электронной почты требуется использование стороннего поставщика электронной почты, например [SendGrid](https://sendgrid.com), [маилжет](https://Mailjet.com)или [SparkPost](https://sparkpost.com), настраиваемого REST API или любого поставщика электронной почты на основе HTTP (включая свой собственный). В этой статье описывается настройка решения, использующего SendGrid.

[!INCLUDE [b2c-public-preview-feature](../../includes/active-directory-b2c-public-preview.md)]

## <a name="create-a-sendgrid-account"></a>Создание учетной записи SendGrid

Если у вас его еще нет, начните с настройки учетной записи SendGrid (клиенты Azure могут разблокировать 25 000 бесплатных сообщений электронной почты каждый месяц). Инструкции по установке см. в разделе [Создание учетной записи SendGrid](../sendgrid-dotnet-how-to-send-email.md#create-a-sendgrid-account) статьи [Отправка электронной почты с помощью SendGrid в Azure](../sendgrid-dotnet-how-to-send-email.md).

Обязательно выполните действия, описанные в разделе [Создание ключа API SendGrid](../sendgrid-dotnet-how-to-send-email.md#to-find-your-sendgrid-api-key). Запишите ключ API для использования на следующем шаге.

## <a name="create-azure-ad-b2c-policy-key"></a>Создание ключа политики Azure AD B2C

Затем сохраните ключ API SendGrid в ключе политики Azure AD B2C, чтобы создать ссылку на них.

1. Войдите на [портал Azure](https://portal.azure.com/).
1. Убедитесь, что вы используете каталог, содержащий клиент Azure AD B2C. Выберите фильтр **Каталог и подписка** в верхнем меню, а затем выберите каталог Azure AD B2C.
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
1. На странице "Обзор" выберите **Identity Experience Framework**.
1. Выберите **Ключи политики**, а затем щелкните **Добавить**.
1. В качестве **параметров** выберите **ручной**.
1. Введите **имя** ключа политики. Например, `SendGridSecret`. Префикс `B2C_1A_` будет автоматически добавлен к имени ключа.
1. В поле **Секрет** введите ранее записанный секрет клиента.
1. Для параметра **Использование ключа** выберите **Подпись**.
1. Нажмите кнопку **создания**.

## <a name="create-sendgrid-template"></a>Создание шаблона SendGrid

После создания учетной записи SendGrid и ключа API SendGrid, хранящегося в ключе политики Azure AD B2C, создайте [динамический шаблон транзакций](https://sendgrid.com/docs/ui/sending-email/how-to-send-an-email-with-dynamic-transactional-templates/)SendGrid.

1. На сайте SendGrid откройте страницу [шаблоны транзакций](https://sendgrid.com/dynamic_templates) и выберите **создать шаблон**.
1. Введите уникальное имя шаблона, например `Verification email` , и нажмите кнопку **сохранить**.
1. Чтобы начать редактирование нового шаблона, выберите **Добавить версию**.
1. Выберите **Редактор кода** и **Продолжайте работу**.
1. В редакторе HTML вставьте следующий шаблон HTML или используйте собственный. `{{otp}}`Параметры и `{{email}}` будут заменены динамическими значениями одноразового пароля и адресом электронной почты пользователя.

    ```HTML
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

    <html xmlns="http://www.w3.org/1999/xhtml" dir="ltr" lang="en"><head id="Head1">
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"><title>Contoso demo account email verification code</title><meta name="ROBOTS" content="NOINDEX, NOFOLLOW">
       <style>
           table td {border-collapse:collapse;margin:0;padding:0;}
       </style>
    </head>
    <body dir="ltr" lang="en">
       <table width="100%" cellpadding="0" cellspacing="0" border="0" dir="ltr" lang="en">
            <tr>
               <td valign="top" width="50%"></td>
               <td valign="top">
                  <!-- Email Header -->
                  <table width="640" cellpadding="0" cellspacing="0" border="0" dir="ltr" lang="en" style="border-left:1px solid #e3e3e3;border-right: 1px solid #e3e3e3;">
                   <tr style="background-color: #0072C6;">
                       <td width="1" style="background:#0072C6; border-top:1px solid #e3e3e3;"></td>
                       <td width="24" style="border-top:1px solid #e3e3e3;border-bottom:1px solid #e3e3e3;">&nbsp;</td>
                       <td width="310" valign="middle" style="border-top:1px solid #e3e3e3; border-bottom:1px solid #e3e3e3;padding:12px 0;">
                           <h1 style="line-height:20pt;font-family:Segoe UI Light; font-size:18pt; color:#ffffff; font-weight:normal;">
                            <span id="HeaderPlaceholder_UserVerificationEmailHeader"><font color="#FFFFFF">Verify your email address</font></span>
                           </h1>
                       </td>
                       <td width="24" style="border-top: 1px solid #e3e3e3;border-bottom: 1px solid #e3e3e3;">&nbsp;</td>
                   </tr>
                  </table>
                  <!-- Email Content -->
                  <table width="640" cellpadding="0" cellspacing="0" border="0" dir="ltr" lang="en">
                   <tr>
                       <td width="1" style="background:#e3e3e3;"></td>
                       <td width="24">&nbsp;</td>
                       <td id="PageBody" width="640" valign="top" colspan="2" style="border-bottom:1px solid #e3e3e3;padding:10px 0 20px;border-bottom-style:hidden;">
                           <table cellpadding="0" cellspacing="0" border="0">
                               <tr>
                                   <td width="630" style="font-size:10pt; line-height:13pt; color:#000;">
                                       <table cellpadding="0" cellspacing="0" border="0" width="100%" style="" dir="ltr" lang="en">
                                           <tr>
                                               <td>

       <div style="font-family:'Segoe UI', Tahoma, sans-serif; font-size:14px; color:#333;">
           <span id="BodyPlaceholder_UserVerificationEmailBodySentence1">Thanks for verifying your {{email}} account!</span>
       </div>
       <br>
       <div style="font-family:'Segoe UI', Tahoma, sans-serif; font-size:14px; color:#333; font-weight: bold">
           <span id="BodyPlaceholder_UserVerificationEmailBodySentence2">Your code is: {{otp}}</span>
       </div>
       <br>
       <br>

                                                   <div style="font-family:'Segoe UI', Tahoma, sans-serif; font-size:14px; color:#333;">
                                                   Sincerely,
                                                   </div>
                                                   <div style="font-family:'Segoe UI', Tahoma, sans-serif; font-size:14px; font-style:italic; color:#333;">
                                                       Contoso
                                                   </div>
                                               </td>
                                           </tr>
                                       </table>
                                   </td>
                               </tr>
                           </table>

                       </td>

                       <td width="1">&nbsp;</td>
                       <td width="1"></td>
                       <td width="1">&nbsp;</td>
                       <td width="1" valign="top"></td>
                       <td width="29">&nbsp;</td>
                       <td width="1" style="background:#e3e3e3;"></td>
                   </tr>
                   <tr>
                       <td width="1" style="background:#e3e3e3; border-bottom:1px solid #e3e3e3;"></td>
                       <td width="24" style="border-bottom:1px solid #e3e3e3;">&nbsp;</td>
                       <td id="PageFooterContainer" width="585" valign="top" colspan="6" style="border-bottom:1px solid #e3e3e3;padding:0px;">


                       </td>

                       <td width="29" style="border-bottom:1px solid #e3e3e3;">&nbsp;</td>
                       <td width="1" style="background:#e3e3e3; border-bottom:1px solid #e3e3e3;"></td>
                   </tr>
                  </table>

               </td>
               <td valign="top" width="50%"></td>
           </tr>
       </table>
    <img src="https://mucp.api.account.microsoft.com/m/v2/v?d=AIAACWEPFYXYIUTJIJVV4ST7XLBHVI5MLLYBKJAVXHBDTBHUM5VBSVVPTTVRWDFIXJ5JQTHYOH5TUYIPO4ZAFRFK52UAMIS3UNIPPI7ZJNDZPRXD5VEJBN4H6RO3SPTBS6AJEEAJOUYL4APQX5RJUJOWGPKUABY&amp;i=AIAACL23GD2PFRFEY5YVM2XQLM5YYWMHFDZOCDXUI2B4LM7ETZQO473CVF22PT6WPGR5IIE6TCS6VGEKO5OZIONJWCDMRKWQQVNP5VBYAINF3S7STKYOVDJ4JF2XEW4QQVNHMAPQNHFV3KMR3V3BA4I36B6BO7L4VQUHQOI64EOWPLMG5RB3SIMEDEHPILXTF73ZYD3JT6MYOLAZJG7PJJCAXCZCQOEFVH5VCW2KBQOKRYISWQLRWAT7IINZ3EFGQI2CY2EMK3FQOXM7UI3R7CZ6D73IKDI" width="1" height="1"></body>
    </html>
    ```

1. Разверните **Параметры** слева и для параметра **Тема сообщения электронной почты** введите `{{subject}}` .
1. Выберите **сохранить шаблон**.
1. Вернитесь на страницу **шаблонов транзакций** , щелкнув стрелку назад.
1. Запишите **идентификатор** шаблона, созданного для использования на более позднем этапе. Например, `d-989077fbba9746e89f3f6411f596fb96`. Этот идентификатор указывается при [добавлении преобразования «утверждения](#add-the-claims-transformation)».

## <a name="add-azure-ad-b2c-claim-types"></a>Добавление типов утверждений Azure AD B2C

В политике добавьте следующие типы утверждений в `<ClaimsSchema>` элемент в `<BuildingBlocks>` .

Эти типы утверждений необходимы для создания и проверки адреса электронной почты с помощью кода одноразового пароля (OTP).

```xml
<!-- 
<BuildingBlocks>
  <ClaimsSchema> -->
    <ClaimType Id="Otp">
      <DisplayName>Secondary One-time password</DisplayName>
      <DataType>string</DataType>
    </ClaimType>
    <ClaimType Id="emailRequestBody">
      <DisplayName>SendGrid request body</DisplayName>
      <DataType>string</DataType>
    </ClaimType>
    <ClaimType Id="VerificationCode">
      <DisplayName>Secondary Verification Code</DisplayName>
      <DataType>string</DataType>
      <UserHelpText>Enter your email verification code</UserHelpText>
      <UserInputType>TextBox</UserInputType>
    </ClaimType>
  <!-- 
  </ClaimsSchema>
</BuildingBlocks> -->
```

## <a name="add-the-claims-transformation"></a>Добавление преобразования «утверждения»

Далее необходимо преобразование заявок для вывода строкового утверждения JSON, которое будет основным текстом запроса, отправленного в SendGrid.

Структура объекта JSON определяется идентификаторами в точечной нотации InputParameters и Трансформатионклаимтипес Inputclaim. Числа в точечной нотации подразумевают массивы. Значения берутся из значений Inputclaim и свойств InputParameters "" value ". Дополнительные сведения о преобразованиях утверждений JSON см. в разделе [преобразования утверждений JSON](json-transformations.md).

Добавьте следующее преобразование заявок в `<ClaimsTransformations>` элемент в `<BuildingBlocks>` . Внесите следующие изменения в XML-код преобразования утверждений.

* Обновите `template_id` значение InputParameter, УКАЗАВ идентификатор шаблона транзакций SendGrid, который вы создали ранее в [шаблоне Create SendGrid](#create-sendgrid-template).
* Обновите `from.email` значение адреса. Используйте допустимый адрес электронной почты, чтобы предотвратить пометку сообщения электронной почты как спама.
* Обновите значение `personalizations.0.dynamic_template_data.subject` входного параметра строки темы со строкой темы, подходящей для вашей организации.

```xml
<!-- 
<BuildingBlocks>
  <ClaimsTransformations> -->
    <ClaimsTransformation Id="GenerateEmailRequestBody" TransformationMethod="GenerateJson">
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="personalizations.0.to.0.email" />
        <InputClaim ClaimTypeReferenceId="otp" TransformationClaimType="personalizations.0.dynamic_template_data.otp" />
        <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="personalizations.0.dynamic_template_data.email" />
      </InputClaims>
      <InputParameters>
        <!-- Update the template_id value with the ID of your SendGrid template. -->
        <InputParameter Id="template_id" DataType="string" Value="d-989077fbba9746e89f3f6411f596fb96"/>
        <InputParameter Id="from.email" DataType="string" Value="my_email@mydomain.com"/>
        <!-- Update with a subject line appropriate for your organization. -->
        <InputParameter Id="personalizations.0.dynamic_template_data.subject" DataType="string" Value="Contoso account email verification code"/>
      </InputParameters>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="emailRequestBody" TransformationClaimType="outputClaim"/>
      </OutputClaims>
    </ClaimsTransformation>
  <!--
  </ClaimsTransformations>
</BuildingBlocks> -->
```

## <a name="add-datauri-content-definition"></a>Добавить определение содержимого DataUri

Под преобразованиями утверждений в `<BuildingBlocks>` добавьте следующие [контентдефинитион](contentdefinitions.md) для ссылки на URI данных версии 2.1.0:

```xml
<!--
<BuildingBlocks> -->
  <ContentDefinitions>
   <ContentDefinition Id="api.localaccountsignup">
      <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:2.1.0</DataUri>
    </ContentDefinition>
    <ContentDefinition Id="api.localaccountpasswordreset">
      <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:2.1.0</DataUri>
    </ContentDefinition>
  </ContentDefinitions>
<!--
</BuildingBlocks> -->
```

## <a name="create-a-displaycontrol"></a>Создать тип элемента

Элемент управления отображением проверки используется для проверки адреса электронной почты с помощью кода проверки, отправляемого пользователю.

Этот пример элемента управления отображением настраивается следующим образом:

1. Получение `email` типа утверждения адреса от пользователя.
1. Дождитесь, пока пользователь предоставит `verificationCode` тип утверждения с кодом, отправленным пользователю.
1. Возврат `email` к самостоятельно утвержденному техническому профилю, который содержит ссылку на этот элемент управления отображением.
1. С помощью `SendCode` действия создайте код OTP и отправьте пользователю сообщение электронной почты с кодом OTP.

![Действие отправки кода проверки по электронной почте](media/custom-email-sendgrid/display-control-verification-email-action-01.png)

В разделе определения содержимого, находящиеся в `<BuildingBlocks>` , [](display-controls.md) добавьте в политику следующий элемент списка типа [верификатионконтрол](display-control-verification.md) .

```xml
<!--
<BuildingBlocks> -->
  <DisplayControls>
    <DisplayControl Id="emailVerificationControl" UserInterfaceControlType="VerificationControl">
      <DisplayClaims>
        <DisplayClaim ClaimTypeReferenceId="email" Required="true" />
        <DisplayClaim ClaimTypeReferenceId="verificationCode" ControlClaimType="VerificationCode" Required="true" />
      </DisplayClaims>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="email" />
      </OutputClaims>
      <Actions>
        <Action Id="SendCode">
          <ValidationClaimsExchange>
            <ValidationClaimsExchangeTechnicalProfile TechnicalProfileReferenceId="GenerateOtp" />
            <ValidationClaimsExchangeTechnicalProfile TechnicalProfileReferenceId="SendOtp" />
          </ValidationClaimsExchange>
        </Action>
        <Action Id="VerifyCode">
          <ValidationClaimsExchange>
            <ValidationClaimsExchangeTechnicalProfile TechnicalProfileReferenceId="VerifyOtp" />
          </ValidationClaimsExchange>
        </Action>
      </Actions>
    </DisplayControl>
  </DisplayControls>
<!--
</BuildingBlocks> -->
```

## <a name="add-otp-technical-profiles"></a>Добавление технических профилей OTP

`GenerateOtp`Технический профиль создает код для адреса электронной почты. `VerifyOtp`Технический профиль проверяет код, связанный с адресом электронной почты. Можно изменить конфигурацию формата и срок действия одноразового пароля. Дополнительные сведения о технических профилях OTP см. [в статье Определение технического профиля одноразового пароля](one-time-password-technical-profile.md).

Добавьте в элемент следующие технические профили `<ClaimsProviders>` .

```xml
<!--
<ClaimsProviders> -->
  <ClaimsProvider>
    <DisplayName>One time password technical profiles</DisplayName>
    <TechnicalProfiles>
      <TechnicalProfile Id="GenerateOtp">
        <DisplayName>Generate one time password</DisplayName>
        <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.OneTimePasswordProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
        <Metadata>
          <Item Key="Operation">GenerateCode</Item>
          <Item Key="CodeExpirationInSeconds">1200</Item>
          <Item Key="CodeLength">6</Item>
          <Item Key="CharacterSet">0-9</Item>
          <Item Key="ReuseSameCode">true</Item>
          <Item Key="NumRetryAttempts">5</Item>
        </Metadata>
        <InputClaims>
          <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="identifier" />
        </InputClaims>
        <OutputClaims>
          <OutputClaim ClaimTypeReferenceId="otp" PartnerClaimType="otpGenerated" />
        </OutputClaims>
      </TechnicalProfile>

      <TechnicalProfile Id="VerifyOtp">
        <DisplayName>Verify one time password</DisplayName>
        <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.OneTimePasswordProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
        <Metadata>
          <Item Key="Operation">VerifyCode</Item>
        </Metadata>
        <InputClaims>
          <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="identifier" />
          <InputClaim ClaimTypeReferenceId="verificationCode" PartnerClaimType="otpToVerify" />
        </InputClaims>
      </TechnicalProfile>
     </TechnicalProfiles>
  </ClaimsProvider>
<!--
</ClaimsProviders> -->
```

## <a name="add-a-rest-api-technical-profile"></a>Добавление REST API технического профиля

Этот REST API технический профиль создает содержимое электронной почты (с использованием формата SendGrid). Дополнительные сведения о технических профилях RESTFUL см. [в разделе Определение технического профиля RESTful](restful-technical-profile.md).

Как и в случае с техническими профилями OTP, добавьте в элемент следующие технические профили `<ClaimsProviders>` .

```xml
<ClaimsProvider>
  <DisplayName>RestfulProvider</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="SendOtp">
      <DisplayName>Use SendGrid's email API to send the code the the user</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="ServiceUrl">https://api.sendgrid.com/v3/mail/send</Item>
        <Item Key="AuthenticationType">Bearer</Item>
        <Item Key="SendClaimsIn">Body</Item>
        <Item Key="ClaimUsedForRequestPayload">emailRequestBody</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="BearerAuthenticationToken" StorageReferenceId="B2C_1A_SendGridSecret" />
      </CryptographicKeys>
      <InputClaimsTransformations>
        <InputClaimsTransformation ReferenceId="GenerateEmailRequestBody" />
      </InputClaimsTransformations>
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="emailRequestBody" />
      </InputClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="make-a-reference-to-the-displaycontrol"></a>Создание ссылки на тип элемента

На последнем шаге добавьте ссылку на созданный тип элемента. Замените существующие `LocalAccountSignUpWithLogonEmail` и `LocalAccountDiscoveryUsingEmailAddress` самостоятельно утвержденные технические профили следующим. Если использовалась более ранняя версия политики Azure AD B2C. Эти технические профили используют `DisplayClaims` ссылку на элемент конфигурации.

Дополнительные сведения см. в разделе [самостоятельно утвержденный технический профиль](restful-technical-profile.md) и [тип](display-controls.md)элемента.

```xml
<ClaimsProvider>
  <DisplayName>Local Account</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
      <Metadata>
        <!--OTP validation error messages-->
        <Item Key="UserMessageIfSessionDoesNotExist">You have exceeded the maximum time allowed.</Item>
        <Item Key="UserMessageIfMaxRetryAttempted">You have exceeded the number of retries allowed.</Item>
        <Item Key="UserMessageIfInvalidCode">You have entered the wrong code.</Item>
        <Item Key="UserMessageIfSessionConflict">Cannot verify the code, please try again later.</Item>
      </Metadata>
      <DisplayClaims>
        <DisplayClaim DisplayControlReferenceId="emailVerificationControl" />
        <DisplayClaim ClaimTypeReferenceId="displayName" Required="true" />
        <DisplayClaim ClaimTypeReferenceId="givenName" Required="true" />
        <DisplayClaim ClaimTypeReferenceId="surName" Required="true" />
        <DisplayClaim ClaimTypeReferenceId="newPassword" Required="true" />
        <DisplayClaim ClaimTypeReferenceId="reenterPassword" Required="true" />
      </DisplayClaims>
    </TechnicalProfile>
    <TechnicalProfile Id="LocalAccountDiscoveryUsingEmailAddress">
      <Metadata>
        <!--OTP validation error messages-->
        <Item Key="UserMessageIfSessionDoesNotExist">You have exceeded the maximum time allowed.</Item>
        <Item Key="UserMessageIfMaxRetryAttempted">You have exceeded the number of retries allowed.</Item>
        <Item Key="UserMessageIfInvalidCode">You have entered the wrong code.</Item>
        <Item Key="UserMessageIfSessionConflict">Cannot verify the code, please try again later.</Item>
      </Metadata>
      <DisplayClaims>
        <DisplayClaim DisplayControlReferenceId="emailVerificationControl" />
      </DisplayClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="optional-localize-your-email"></a>Используемых Локализация электронной почты

Чтобы локализовать сообщение электронной почты, необходимо отправить локализованные строки в SendGrid или ваш поставщик электронной почты. Например, можно локализовать тему сообщения электронной почты, текст, сообщение кода или подпись сообщения. Для этого можно использовать преобразование утверждений [жетлокализедстрингстрансформатион](string-transformations.md) , чтобы скопировать локализованные строки в типы утверждений. `GenerateEmailRequestBody`Преобразование «утверждения», которое создает полезные данные JSON, использует входные утверждения, содержащие локализованные строки.

1. В политике определите следующие строковые утверждения: subject, Message, Кодеинтро и Signature.
1. Определите преобразование [жетлокализедстрингстрансформатион](string-transformations.md) Claims, чтобы заменить локализованные строковые значения на утверждения из шага 1.
1. Измените `GenerateEmailRequestBody` Преобразование утверждения, чтобы использовать входные утверждения со следующим фрагментом XML.
1. Обновите шаблон SendGrid, чтобы использовать динамические параметры вместо всех строк, которые будут локализованы Azure AD B2C.

    ```xml
    <ClaimsTransformation Id="GetLocalizedStringsForEmail" TransformationMethod="GetLocalizedStringsTransformation">
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="subject" TransformationClaimType="email_subject" />
        <OutputClaim ClaimTypeReferenceId="message" TransformationClaimType="email_message" />
        <OutputClaim ClaimTypeReferenceId="codeIntro" TransformationClaimType="email_code" />
        <OutputClaim ClaimTypeReferenceId="signature" TransformationClaimType="email_signature" />
      </OutputClaims>
    </ClaimsTransformation>
    <ClaimsTransformation Id="GenerateEmailRequestBody" TransformationMethod="GenerateJson">
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="personalizations.0.to.0.email" />
        <InputClaim ClaimTypeReferenceId="subject" TransformationClaimType="personalizations.0.dynamic_template_data.subject" />
        <InputClaim ClaimTypeReferenceId="otp" TransformationClaimType="personalizations.0.dynamic_template_data.otp" />
        <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="personalizations.0.dynamic_template_data.email" />
        <InputClaim ClaimTypeReferenceId="message" TransformationClaimType="personalizations.0.dynamic_template_data.message" />
        <InputClaim ClaimTypeReferenceId="codeIntro" TransformationClaimType="personalizations.0.dynamic_template_data.codeIntro" />
        <InputClaim ClaimTypeReferenceId="signature" TransformationClaimType="personalizations.0.dynamic_template_data.signature" />
      </InputClaims>
      <InputParameters>
        <InputParameter Id="template_id" DataType="string" Value="d-1234567890" />
        <InputParameter Id="from.email" DataType="string" Value="my_email@mydomain.com" />
      </InputParameters>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="emailRequestBody" TransformationClaimType="outputClaim" />
      </OutputClaims>
    </ClaimsTransformation>
    ```

1. Добавьте следующий элемент [Localization](localization.md) .

    ```xml
    <!--
    <BuildingBlocks> -->
      <Localization Enabled="true">
        <SupportedLanguages DefaultLanguage="en" MergeBehavior="Append">
          <SupportedLanguage>en</SupportedLanguage>
          <SupportedLanguage>es</SupportedLanguage>
        </SupportedLanguages>
        <LocalizedResources Id="api.custom-email.en">
          <LocalizedStrings>
            <!--Email template parameters-->
            <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_subject">Contoso account email verification code</LocalizedString>
            <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_message">Thanks for validating the account</LocalizedString>
            <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_code">Your code is</LocalizedString>
            <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_signature">Sincerely</LocalizedString>
          </LocalizedStrings>
        </LocalizedResources>
        <LocalizedResources Id="api.custom-email.es">
          <LocalizedStrings>
            <!--Email template parameters-->
            <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_subject">Código de verificación del correo electrónico de la cuenta de Contoso</LocalizedString>
            <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_message">Gracias por comprobar la cuenta de </LocalizedString>
            <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_code">Su código es</LocalizedString>
            <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_signature">Sinceramente</LocalizedString>
          </LocalizedStrings>
        </LocalizedResources>
      </Localization>
    <!--
    </BuildingBlocks> -->
    ```

1. Добавьте ссылки на элементы LocalizedResources, обновив элемент [ContentDefinitions](contentdefinitions.md) .

    ```XML
    <!--
    <BuildingBlocks> -->
      <ContentDefinitions>
        <ContentDefinition Id="api.localaccountsignup">
          <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:2.1.0</DataUri>
          <LocalizedResourcesReferences MergeBehavior="Prepend">
            <LocalizedResourcesReference Language="en" LocalizedResourcesReferenceId="api.custom-email.en" />
            <LocalizedResourcesReference Language="es" LocalizedResourcesReferenceId="api.custom-email.es" />
          </LocalizedResourcesReferences>
        </ContentDefinition>
        <ContentDefinition Id="api.localaccountpasswordreset">
          <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:2.1.0</DataUri>
          <LocalizedResourcesReferences MergeBehavior="Prepend">
            <LocalizedResourcesReference Language="en" LocalizedResourcesReferenceId="api.custom-email.en" />
            <LocalizedResourcesReference Language="es" LocalizedResourcesReferenceId="api.custom-email.es" />
          </LocalizedResourcesReferences>
        </ContentDefinition>
      </ContentDefinitions>
    <!--
    </BuildingBlocks> -->
    ```

1. Наконец, добавьте следующее преобразование входных утверждений `LocalAccountSignUpWithLogonEmail` в `LocalAccountDiscoveryUsingEmailAddress` технические профили и.

    ```XML
    <InputClaimsTransformations>
      <InputClaimsTransformation ReferenceId="GetLocalizedStringsForEmail" />
    </InputClaimsTransformations>
    ```
    
## <a name="optional-localize-the-ui"></a>Используемых Локализация пользовательского интерфейса

Элемент Localization обеспечивает поддержку несколько языковых стандартов или языков в политике для путей взаимодействия пользователя. Поддержка локализации в политиках позволяет предоставлять зависящие от языка строки для [элементов пользовательского интерфейса контроля над отображением проверки](localization-string-ids.md#verification-display-control-user-interface-elements)и [один раз в сообщениях об ошибках пароля](localization-string-ids.md#one-time-password-error-messages). Добавьте следующий Локализедстринг в LocalizedResources. 

```XML
<LocalizedResources Id="api.custom-email.en">
  <LocalizedStrings>
    ...
    <!-- Display control UI elements-->
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="intro_msg">Verification is necessary. Please click Send button.</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="success_send_code_msg">Verification code has been sent to your inbox. Please copy it to the input box below.</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="failure_send_code_msg">We are having trouble verifying your email address. Please enter a valid email address and try again.</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="success_verify_code_msg">E-mail address verified. You can now continue.</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="failure_verify_code_msg">We are having trouble verifying your email address. Please try again.</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="but_send_code">Send verification code</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="but_verify_code">Verify code</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="but_send_new_code">Send new code</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="but_change_claims">Change e-mail</LocalizedString>
    <!-- Claims-->
    <LocalizedString ElementType="ClaimType" ElementId="emailVerificationCode" StringId="DisplayName">Verification Code</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="emailVerificationCode" StringId="UserHelpText">Verification code received in the email.</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="emailVerificationCode" StringId="AdminHelpText">Verification code received in the email.</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="DisplayName">Eamil</LocalizedString>
    <!-- Email validation error messages-->
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfSessionDoesNotExist">You have exceeded the maximum time allowed.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfMaxRetryAttempted">You have exceeded the number of retries allowed.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInvalidCode">You have entered the wrong code.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfSessionConflict">Cannot verify the code, please try again later.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfVerificationFailedRetryAllowed">The verification has failed, please try again.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

После добавления локализованных строк удалите метаданные сообщений об ошибках проверки OTP из технических профилей Локалаккаунтсигнупвислогонемаил и Локалаккаунтдисковерюсинжемаиладдресс.

## <a name="next-steps"></a>Дальнейшие действия

Пример пользовательской политики проверки электронной почты можно найти на сайте GitHub:

- [Пользовательская проверка электронной почты — Дисплайконтролс](https://github.com/azure-ad-b2c/samples/tree/master/policies/custom-email-verifcation-displaycontrol)
- Сведения об использовании настраиваемого REST API или поставщика электронной почты SMTP на основе HTTP см. [в разделе Определение технического профиля RESTful в Azure AD B2C настраиваемой политике](restful-technical-profile.md).
