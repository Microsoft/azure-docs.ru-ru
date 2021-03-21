---
title: Azure Active Directory B2C. Идентификаторы строк локализации | Документация Майкрософт
description: Сведения об указании идентификаторов для определения содержимого с идентификатором api.signuporsignin в настраиваемой политике в Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 03/10/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: b60b447d8302b89813ca462c3220603ef926eb26
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102631189"
---
# <a name="localization-string-ids"></a>Идентификаторы строк локализации

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Элемент **Localization** позволяет поддерживать несколько языковых стандартов или языков в политике для путей взаимодействия пользователя. В этой статье предоставлен список идентификаторов локализации, которые можно использовать в политике. Чтобы ознакомиться с локализацией пользовательского интерфейса, обратитесь к статье [Локализация](localization.md).

## <a name="sign-up-or-sign-in-page-elements"></a>Элементы страниц регистрации или входа

Следующие идентификаторы используются для определения содержимого с ИДЕНТИФИКАТОРом `api.signuporsignin` и [самостоятельно подтвержденным техническим профилем](self-asserted-technical-profile.md).

| ID | Значение по умолчанию | Версия макета страницы |
| -- | ------------- | ------ |
| **forgotpassword_link** | Забыли пароль? | `All` |
| **createaccount_intro** | Нет учетной записи? | `All` |
| **button_signin** | Вход | `All` |
| **social_intro** | Войти, используя учетную запись социальной сети | `All` |
| **remember_me** |Оставаться в системе. | `All` |
| **unknown_error** | Возникли проблемы со входом в систему. Повторите попытку позже. | `All` |
| **divider_title** | OR | `All` |
| **local_intro_email** | Войти, используя имеющуюся учетную запись | `< 2.0.0` |
| **logonIdentifier_email** | Электронная почта | `< 2.0.0` |
| **requiredField_email** | Введите свой адрес электронной почты | `< 2.0.0` |
| **invalid_email** | Введите действительный адрес электронной почты | `< 2.0.0` |
| **email_pattern** | ^ [a-zA-Z0-9.! # $% & ' ' \* +/=? ^ \_ \` { \| } ~-] + @ [a-zA-Z0-9-] + (?: \\ . [ a-zA-Z0-9-] +) \* $ |`< 2.0.0` |
| **local_intro_username** | Войти, используя свое имя пользователя | `< 2.0.0` |
| **logonIdentifier_username** | Имя пользователя | `< 2.0.0` |
| **requiredField_username** | Введите свое имя пользователя | `< 2.0.0` |
| **password** | Пароль | `< 2.0.0` |
| **requiredField_password** | Введите пароль | `< 2.0.0` |
| **createaccount_link** | Зарегистрируйтесь сейчас | `< 2.0.0` |
| **cancel_message** | Пользователь забыл свой пароль | `< 2.0.0` |
| **invalid_password** | Введенный пароль не соответствует ожидаемому формату. | `< 2.0.0` |
| **createaccount_one_link** | Зарегистрируйтесь сейчас | `>= 2.0.0` |
| **createaccount_two_links** | Зарегистрироваться с помощью {0} или {1} | `>= 2.0.0` |
| **createaccount_three_links** | Регистрация с помощью {0} , {1} или {2} | `>= 2.0.0` |
| **local_intro_generic** | Войдите с помощью {0} | `>= 2.1.0` |
| **requiredField_generic** | Укажите {0} | `>= 2.1.0` |
| **invalid_generic** | Введите допустимый {0} | `>= 2.1.1` |
| **направляющ** | Вход | `>= 2.1.1` |


> [!NOTE]
> * Заполнители, такие как {0} , будут автоматически заполнены `DisplayName` значением `ClaimType` . 
> * Дополнительные сведения о локализации `ClaimType` см. в статье [Пример регистрации или входа в](#signupsigninexample)систему.

В следующем примере показано использование некоторых элементов пользовательского интерфейса на странице регистрации или входа в систему:

:::image type="content" source="./media/localization-string-ids/localization-susi-2.png" alt-text="Снимок экрана, на котором показаны элементы регистрации или страницы входа U X.":::

### <a name="sign-up-or-sign-in-identity-providers"></a>Поставщики удостоверений регистрации или входа

Идентификаторы поставщиков удостоверений настраиваются в элементе пути взаимодействия пользователя **ClaimsExchange**. Для локализации названия поставщика удостоверений необходимо присвоить **ElementType** значение `ClaimsProvider`, а для **StringId** присваивается идентификатор `ClaimsExchange`.

```xml
<OrchestrationStep Order="2" Type="ClaimsExchange">
  <Preconditions>
    <Precondition Type="ClaimsExist" ExecuteActionsIf="true">
      <Value>objectId</Value>
      <Action>SkipThisOrchestrationStep</Action>
    </Precondition>
  </Preconditions>
  <ClaimsExchanges>
    <ClaimsExchange Id="FacebookExchange" TechnicalProfileReferenceId="Facebook-OAUTH" />
    <ClaimsExchange Id="MicrosoftExchange" TechnicalProfileReferenceId="MSA-OIDC" />
    <ClaimsExchange Id="GoogleExchange" TechnicalProfileReferenceId="Google-OAUTH" />
    <ClaimsExchange Id="SignUpWithLogonEmailExchange" TechnicalProfileReferenceId="LocalAccount" />
  </ClaimsExchanges>
</OrchestrationStep>
```

В следующем примере показана локализация поставщика удостоверений Facebook на арабский язык:

```xml
<LocalizedString ElementType="ClaimsProvider" StringId="FacebookExchange">فيس بوك</LocalizedString>
```

### <a name="sign-up-or-sign-in-error-messages"></a>Сообщения об ошибках при входе или регистрации

| ID | Значение по умолчанию |
| -- | ------------- |
| **UserMessageIfInvalidPassword** | Неправильный пароль. |
| **усермессажеифпассвордекспиред**| Истек срок действия пароля.|
| **UserMessageIfClaimsPrincipalDoesNotExist** | Нам не удается найти вашу учетную запись. |
| **UserMessageIfOldPasswordUsed** | Вероятно, вы используете старый пароль. |
| **DefaultMessage** | Недопустимое имя пользователя или пароль. |
| **UserMessageIfUserAccountDisabled** | Ваша учетная запись заблокирована. Обратитесь к представителю службы поддержки, чтобы разблокировать ее, и повторите попытку. |
| **UserMessageIfUserAccountLocked** | Ваша учетная запись временно заблокирована, чтобы предотвратить ее несанкционированное использование. Повторите попытку позже. |
| **AADRequestsThrottled** | Сейчас слишком много запросов. Подождите и повторите попытку. |

<a name="signupsigninexample"></a>
### <a name="sign-up-or-sign-in-example"></a>Пример регистрации или входа

```xml
<LocalizedResources Id="api.signuporsignin.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="DisplayName">Email Address</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="heading">Sign in</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="social_intro">Sign in with your social account</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="local_intro_generic">Sign in with your {0}</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="requiredField_password">Please enter your password</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="requiredField_generic">Please enter your {0}</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="invalid_generic">Please enter a valid {0}</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="createaccount_one_link">Sign up now</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="createaccount_two_links">Sign up with {0} or {1}</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="createaccount_three_links">Sign up with {0}, {1}, or {2}</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="forgotpassword_link">Forgot your password?</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="button_signin">Sign in</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="divider_title">OR</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="unknown_error">We are having trouble signing you in. Please try again later.</LocalizedString>
    <!-- Uncomment the remember_me only if the keep me signed in is activated. 
    <LocalizedString ElementType="UxElement" StringId="remember_me">Keep me signed in</LocalizedString> -->
    <LocalizedString ElementType="ClaimsProvider" StringId="FacebookExchange">Facebook</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInvalidPassword">Your password is incorrect.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfPasswordExpired">Your password has expired.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsPrincipalDoesNotExist">We can't seem to find your account.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfOldPasswordUsed">Looks like you used an old password.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="DefaultMessage">Invalid username or password.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfUserAccountDisabled">Your account has been locked. Contact your support person to unlock it, then try again.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfUserAccountLocked">Your account is temporarily locked to prevent unauthorized use. Try again later.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="AADRequestsThrottled">There are too many requests at this moment. Please wait for some time and try again.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="sign-up-and-self-asserted-pages-user-interface-elements"></a>Элементы пользовательского интерфейса страниц для входа и самоподтверждения

Ниже приведены идентификаторы для определения содержимого с ИДЕНТИФИКАТОРом `api.localaccountsignup` или любым определением содержимого, которое начинается с `api.selfasserted` , например, и и с `api.selfasserted.profileupdate` `api.localaccountpasswordreset` [самостоятельно подтвержденным техническим профилем](self-asserted-technical-profile.md).

| ID | Значение по умолчанию |
| -- | ------------- |
| **ver_sent** | Код проверки отправлен на следующий адрес: |
| **ver_but_default** | По умолчанию |
| **cancel_message** | Пользователь отменил ввод самостоятельно подтверждаемых данных |
| **preloader_alt** | Подождите |
| **ver_but_send** | Отправить код проверки |
| **alert_yes** | Да |
| **error_fieldIncorrect** | Некоторые поля заполнены неправильно. Проверьте введенные данные и повторите попытку. |
| **year** | Год |
| **verifying_blurb** | Подождите, пока мы обрабатываем ваши данные. |
| **button_cancel** | Отмена |
| **ver_fail_no_retry** | Вы ввели неправильный код слишком много раз. Повторите попытку позже. |
| **month** | Месяц |
| **ver_success_msg** | Адрес электронной почты проверен. Вы можете продолжать работу. |
| **months** | Январь, февраль, март, апрель, май, июнь, июль, август, сентябрь, октябрь, ноябрь, декабрь |
| **ver_fail_server** | Возникли проблемы с проверкой адреса электронной почты. Введите допустимый адрес электронной почты и повторите попытку. |
| **error_requiredFieldMissing** | Отсутствует обязательное поле. Заполните все обязательные поля и повторите попытку. |
| **initial_intro** | Укажите следующие сведения. |
| **ver_but_resend** | Отправить новый код |
| **button_continue** | Создание |
| **error_passwordEntryMismatch** | Пароли в полях не совпадают. Введите одинаковый пароль в оба поля и повторите попытку. |
| **ver_incorrect_format** | Неверный формат. |
| **ver_but_edit** | Изменить адрес электронной почты |
| **ver_but_verify** | Проверить код |
| **alert_no** | нет |
| **ver_info_msg** | Код проверки отправлен на ваш адрес. Скопируйте его в поле ввода ниже. |
| **day** | День |
| **ver_fail_throttled** | Отправлено слишком много запросов на проверку этого адреса электронной почты. Повторите попытку позже. |
| **helplink_text** | Что это такое? |
| **ver_fail_retry** | Этот код неправильный. Повторите попытку. |
| **alert_title** | Отмена ввода своих данных |
| **required_field** | Эта информация обязательна. |
| **alert_message** | Вы действительно хотите отменить ввод своих данных? |
| **ver_intro_msg** | Проверка не требуется. Нажмите кнопку "Отправить". |
| **ver_input** | Код проверки |

### <a name="sign-up-and-self-asserted-pages-error-messages"></a>Сообщения об ошибках на страницах входа и самоподтверждения

| ID | Значение по умолчанию |
| -- | ------------- |
| **UserMessageIfClaimsPrincipalAlreadyExists** | Пользователь с указанным идентификатором уже существует. Выберите другой идентификатор. |
| **UserMessageIfClaimNotVerified** | Утверждение не проверено: {0} |
| **UserMessageIfIncorrectPattern** | Недопустимый шаблон для: {0} |
| **UserMessageIfMissingRequiredElement** | Отсутствует обязательный элемент: {0} |
| **UserMessageIfValidationError** | Ошибка при проверке: {0} |
| **UserMessageIfInvalidInput** | Для {0} указаны недопустимые входные данные. |
| **ServiceThrottled** | Сейчас слишком много запросов. Подождите и повторите попытку. |

В следующем примере показано использование некоторых элементов пользовательского интерфейса на странице регистрации:

![Страница регистрации с именами элементов пользовательского интерфейса с меткой](./media/localization-string-ids/localization-sign-up.png)

В следующем примере показано использование некоторых элементов пользовательского интерфейса на странице регистрации, после того как пользователь щелкнул кнопку отправки кода проверки:

![Элементы пользовательского интерфейса на странице регистрации для проверки адреса электронной почты](./media/localization-string-ids/localization-email-verification.png)

## <a name="sign-up-and-self-asserted-pages-example"></a>Пример регистрации и самостоятельно утвержденных страниц

```xml
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="DisplayName">Email Address</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="UserHelpText">Email address that can be used to contact you.</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="PatternHelpText">Please enter a valid email address.</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="newPassword" StringId="DisplayName">New Password</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="newPassword" StringId="UserHelpText">Enter new password</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="newPassword" StringId="PatternHelpText">8-16 characters, containing 3 out of 4 of the following: Lowercase characters, uppercase characters, digits (0-9), and one or more of the following symbols: @ # $ % ^ &amp; * - _ + = [ ] { } | \ : ' , ? / ` ~ " ( ) ; .</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="reenterPassword" StringId="DisplayName">Confirm New Password</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="reenterPassword" StringId="UserHelpText">Confirm new password</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="reenterPassword" StringId="PatternHelpText">#8-16 characters, containing 3 out of 4 of the following: Lowercase characters, uppercase characters, digits (0-9), and one or more of the following symbols: @ # $ % ^ &amp; * - _ + = [ ] { } | \ : ' , ? / ` ~ " ( ) ; .</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="displayName" StringId="DisplayName">Display Name</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="displayName" StringId="UserHelpText">Your display name.</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="surname" StringId="DisplayName">Surname</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="surname" StringId="UserHelpText">Your surname (also known as family name or last name).</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="givenName" StringId="DisplayName">Given Name</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="givenName" StringId="UserHelpText">Your given name (also known as first name).</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="button_continue">Create</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_fieldIncorrect">One or more fields are filled out incorrectly. Please check your entries and try again.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_passwordEntryMismatch">The password entry fields do not match. Please enter the same password in both fields and try again.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_requiredFieldMissing">A required field is missing. Please fill out all required fields and try again.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="helplink_text">What is this?</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="initial_intro">Please provide the following details.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="preloader_alt">Please wait</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="required_field">This information is required.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_but_edit">Change e-mail</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_but_resend">Send new code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_but_send">Send verification code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_but_verify">Verify code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_fail_code_expired">That code is expired. Please request a new code.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_fail_no_retry">You've made too many incorrect attempts. Please try again later.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_fail_retry">That code is incorrect. Please try again.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_fail_server">We are having trouble verifying your email address. Please enter a valid email address and try again.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_fail_throttled">There have been too many requests to verify this email address. Please wait a while, then try again.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_info_msg">Verification code has been sent to your inbox. Please copy it to the input box below.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_input">Verification code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_intro_msg">Verification is necessary. Please click Send button.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_success_msg">E-mail address verified. You can now continue.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="ServiceThrottled">There are too many requests at this moment. Please wait for some time and try again.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimNotVerified">Claim not verified: {0}</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsPrincipalAlreadyExists">A user with the specified ID already exists. Please choose a different one.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfIncorrectPattern">Incorrect pattern for: {0}</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInvalidInput">{0} has invalid input.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfMissingRequiredElement">Missing required element: {0}</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfValidationError">Error in validation by: {0}</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="phone-factor-authentication-page-user-interface-elements"></a>Элементы пользовательского интерфейса на странице аутентификации по номеру телефона

Ниже приведены идентификаторы для определения содержимого с ИДЕНТИФИКАТОРом `api.phonefactor` и [техническим профилем "телефонный фактор](phone-factor-technical-profile.md)".

| ID | Значение по умолчанию |
| -- | ------------- |
| **button_verify** | Позвонить мне |
| **country_code_label** | код страны; |
| **cancel_message** | Пользователь отменил многофакторную проверку подлинности |
| **text_button_send_second_code** | Отправить новый код |
| **code_pattern** | \\d{6} |
| **intro_mixed** | У нас записан ваш номер. Мы можем отправить код в SMS или позвонить по этому номеру для проверки подлинности. |
| **intro_mixed_p** | У нас записаны ваши номера. Выберите номер, по которому мы можем позвонить или отправить код в SMS для проверки подлинности. |
| **button_verify_code** | Проверить код |
| **requiredField_code** | Введите полученный код проверки |
| **invalid_code** | Введите полученный код из 6 цифр |
| **button_cancel** | Отмена |
| **local_number_input_placeholder_text** | номер телефона. |
| **button_retry** | Повторить попытку |
| **alternative_text** | У меня нет с собой телефона |
| **intro_phone_p** | У нас записаны ваши номера. Выберите номер, по которому мы можем позвонить для проверки подлинности. |
| **intro_phone** | У нас записан ваш номер. Мы позвоним по нему для проверки подлинности. |
| **enter_code_text_intro** | Введите ниже код проверки или  |
| **intro_entry_phone** | Введите ниже номер, по которому мы можем позвонить для проверки подлинности. |
| **intro_entry_sms** | Введите ниже номер, по которому мы можем отправить код в SMS для проверки подлинности. |
| **button_send_code** | Отправить код |
| **invalid_number** | Введите действительный номер телефона |
| **intro_sms** | У нас записан ваш номер. Мы отправим код в SMS для проверки подлинности. |
| **intro_entry_mixed** | Введите ниже номер, по которому мы можем отправить код в SMS или позвонить для проверки подлинности. |
| **number_pattern** | ^\\+(?:[0-9][\\x20-]?){6,14}[0-9]$ |
| **intro_sms_p** |У нас записаны ваши номера. Выберите номер, по которому мы можем отправить код в SMS для проверки подлинности. |
| **requiredField_countryCode** | Выберите код страны |
| **requiredField_number** | Введите номер телефона |
| **country_code_input_placeholder_text** |Страна или регион |
| **number_label** | номер телефона; |
| **error_tryagain** | Указанный номер телефона занят или недоступен. Проверьте номер и повторите попытку. |
| **error_incorrect_code** | Введенный код проверки не совпадает с нашим. Повторите попытку или запросите новый код. |
| **countryList** | См. [список стран](#phone-factor-authentication-page-example). |
| **error_448** | Указанный номер телефона недоступен. |
| **error_449** | Пользователь превысил количество повторных попыток. |
| **verification_code_input_placeholder_text** | Код проверки |

В следующем примере показано использование некоторых элементов пользовательского интерфейса на странице регистрации многофакторной идентификации:

![Элементы пользовательского интерфейса регистрации проверки подлинности с помощью телефонного фактора](./media/localization-string-ids/localization-mfa1.png)

В следующем примере показано использование некоторых элементов пользовательского интерфейса на странице проверки многофакторной идентификации:

![Элементы UX проверки подлинности с помощью телефонного фактора](./media/localization-string-ids/localization-mfa2.png)

## <a name="phone-factor-authentication-page-example"></a>Пример страницы проверки подлинности по телефонным факторам

```xml
<LocalizedResources Id="api.phonefactor.en">
  <LocalizedStrings>
    <LocalizedString ElementType="UxElement" StringId="button_verify">Call Me</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="country_code_label">Country Code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="cancel_message">The user has canceled multi-factor authentication</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="text_button_send_second_code">Send a new code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="code_pattern">\d{6}</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_mixed">We have the following number on record for you. We can send a code via SMS or phone to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_mixed_p">We have the following numbers on record for you. Choose a number that we can phone or send a code via SMS to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="button_verify_code">Verify Code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="requiredField_code">Please enter the verification code you received</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="invalid_code">Please enter the 6-digit code you received</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="button_cancel">Cancel</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="local_number_input_placeholder_text">Phone number</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="button_retry">Retry</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="alternative_text">I don't have my phone</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_phone_p">We have the following numbers on record for you. Choose a number that we can phone to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_phone">We have the following number on record for you. We will phone to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="enter_code_text_intro">Enter your verification code below, or</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_entry_phone">Enter a number below that we can phone to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_entry_sms">Enter a number below that we can send a code via SMS to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="button_send_code">Send Code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="invalid_number">Please enter a valid phone number</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_sms">We have the following number on record for you. We will send a code via SMS to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_entry_mixed">Enter a number below that we can send a code via SMS or phone to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="number_pattern">^\+(?:[0-9][\x20-]?){6,14}[0-9]$</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_sms_p">We have the following numbers on record for you. Choose a number that we can send a code via SMS to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="requiredField_countryCode">Please select your country code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="requiredField_number">Please enter your phone number</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="country_code_input_placeholder_text">Country or region</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="number_label">Phone Number</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_tryagain">The phone number you provided is busy or unavailable. Please check the number and try again.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_incorrect_code">The verification code you have entered does not match our records. Please try again, or request a new code.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="countryList">{"DEFAULT":"Country/Region","AF":"Afghanistan","AX":"Åland Islands","AL":"Albania","DZ":"Algeria","AS":"American Samoa","AD":"Andorra","AO":"Angola","AI":"Anguilla","AQ":"Antarctica","AG":"Antigua and Barbuda","AR":"Argentina","AM":"Armenia","AW":"Aruba","AU":"Australia","AT":"Austria","AZ":"Azerbaijan","BS":"Bahamas","BH":"Bahrain","BD":"Bangladesh","BB":"Barbados","BY":"Belarus","BE":"Belgium","BZ":"Belize","BJ":"Benin","BM":"Bermuda","BT":"Bhutan","BO":"Bolivia","BQ":"Bonaire","BA":"Bosnia and Herzegovina","BW":"Botswana","BV":"Bouvet Island","BR":"Brazil","IO":"British Indian Ocean Territory","VG":"British Virgin Islands","BN":"Brunei","BG":"Bulgaria","BF":"Burkina Faso","BI":"Burundi","CV":"Cabo Verde","KH":"Cambodia","CM":"Cameroon","CA":"Canada","KY":"Cayman Islands","CF":"Central African Republic","TD":"Chad","CL":"Chile","CN":"China","CX":"Christmas Island","CC":"Cocos (Keeling) Islands","CO":"Colombia","KM":"Comoros","CG":"Congo","CD":"Congo (DRC)","CK":"Cook Islands","CR":"Costa Rica","CI":"Côte d'Ivoire","HR":"Croatia","CU":"Cuba","CW":"Curaçao","CY":"Cyprus","CZ":"Czech Republic","DK":"Denmark","DJ":"Djibouti","DM":"Dominica","DO":"Dominican Republic","EC":"Ecuador","EG":"Egypt","SV":"El Salvador","GQ":"Equatorial Guinea","ER":"Eritrea","EE":"Estonia","ET":"Ethiopia","FK":"Falkland Islands","FO":"Faroe Islands","FJ":"Fiji","FI":"Finland","FR":"France","GF":"French Guiana","PF":"French Polynesia","TF":"French Southern Territories","GA":"Gabon","GM":"Gambia","GE":"Georgia","DE":"Germany","GH":"Ghana","GI":"Gibraltar","GR":"Greece","GL":"Greenland","GD":"Grenada","GP":"Guadeloupe","GU":"Guam","GT":"Guatemala","GG":"Guernsey","GN":"Guinea","GW":"Guinea-Bissau","GY":"Guyana","HT":"Haiti","HM":"Heard Island and McDonald Islands","HN":"Honduras","HK":"Hong Kong SAR","HU":"Hungary","IS":"Iceland","IN":"India","ID":"Indonesia","IR":"Iran","IQ":"Iraq","IE":"Ireland","IM":"Isle of Man","IL":"Israel","IT":"Italy","JM":"Jamaica","JP":"Japan","JE":"Jersey","JO":"Jordan","KZ":"Kazakhstan","KE":"Kenya","KI":"Kiribati","KR":"Korea","KW":"Kuwait","KG":"Kyrgyzstan","LA":"Laos","LV":"Latvia","LB":"Lebanon","LS":"Lesotho","LR":"Liberia","LY":"Libya","LI":"Liechtenstein","LT":"Lithuania","LU":"Luxembourg","MO":"Macao SAR","MK":"North Macedonia","MG":"Madagascar","MW":"Malawi","MY":"Malaysia","MV":"Maldives","ML":"Mali","MT":"Malta","MH":"Marshall Islands","MQ":"Martinique","MR":"Mauritania","MU":"Mauritius","YT":"Mayotte","MX":"Mexico","FM":"Micronesia","MD":"Moldova","MC":"Monaco","MN":"Mongolia","ME":"Montenegro","MS":"Montserrat","MA":"Morocco","MZ":"Mozambique","MM":"Myanmar","NA":"Namibia","NR":"Nauru","NP":"Nepal","NL":"Netherlands","NC":"New Caledonia","NZ":"New Zealand","NI":"Nicaragua","NE":"Niger","NG":"Nigeria","NU":"Niue","NF":"Norfolk Island","KP":"North Korea","MP":"Northern Mariana Islands","NO":"Norway","OM":"Oman","PK":"Pakistan","PW":"Palau","PS":"Palestinian Authority","PA":"Panama","PG":"Papua New Guinea","PY":"Paraguay","PE":"Peru","PH":"Philippines","PN":"Pitcairn Islands","PL":"Poland","PT":"Portugal","PR":"Puerto Rico","QA":"Qatar","RE":"Réunion","RO":"Romania","RU":"Russia","RW":"Rwanda","BL":"Saint Barthélemy","KN":"Saint Kitts and Nevis","LC":"Saint Lucia","MF":"Saint Martin","PM":"Saint Pierre and Miquelon","VC":"Saint Vincent and the Grenadines","WS":"Samoa","SM":"San Marino","ST":"São Tomé and Príncipe","SA":"Saudi Arabia","SN":"Senegal","RS":"Serbia","SC":"Seychelles","SL":"Sierra Leone","SG":"Singapore","SX":"Sint Maarten","SK":"Slovakia","SI":"Slovenia","SB":"Solomon Islands","SO":"Somalia","ZA":"South Africa","GS":"South Georgia and South Sandwich Islands","SS":"South Sudan","ES":"Spain","LK":"Sri Lanka","SH":"St Helena, Ascension, Tristan da Cunha","SD":"Sudan","SR":"Suriname","SJ":"Svalbard","SZ":"Swaziland","SE":"Sweden","CH":"Switzerland","SY":"Syria","TW":"Taiwan","TJ":"Tajikistan","TZ":"Tanzania","TH":"Thailand","TL":"Timor-Leste","TG":"Togo","TK":"Tokelau","TO":"Tonga","TT":"Trinidad and Tobago","TN":"Tunisia","TR":"Turkey","TM":"Turkmenistan","TC":"Turks and Caicos Islands","TV":"Tuvalu","UM":"U.S. Outlying Islands","VI":"U.S. Virgin Islands","UG":"Uganda","UA":"Ukraine","AE":"United Arab Emirates","GB":"United Kingdom","US":"United States","UY":"Uruguay","UZ":"Uzbekistan","VU":"Vanuatu","VA":"Vatican City","VE":"Venezuela","VN":"Vietnam","WF":"Wallis and Futuna","YE":"Yemen","ZM":"Zambia","ZW":"Zimbabwe"}</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_448">The phone number you provided is unreachable.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_449">User has exceeded the number of retry attempts.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="verification_code_input_placeholder_text">Verification code</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="strongAuthenticationPhoneNumber" StringId="DisplayName">Phone Number</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>

```

## <a name="verification-display-control-user-interface-elements"></a>Элементы пользовательского интерфейса элемента управления отображением проверки

Ниже приведены идентификаторы для [элемента управления отображением проверки](display-control-verification.md) с [макетом страницы](page-layout.md) 2.1.0 или более поздней версии.

| ID | Значение по умолчанию |
| -- | ------------- |
|intro_msg <sup>*</sup>| Проверка не требуется. Нажмите кнопку "Отправить".|
|success_send_code_msg | Код проверки отправлен. Скопируйте его в поле ввода ниже.|
|failure_send_code_msg | Возникли проблемы с проверкой адреса электронной почты. Введите допустимый адрес электронной почты и повторите попытку.|
|success_verify_code_msg | Адрес электронной почты проверен. Вы можете продолжать работу.|
|failure_verify_code_msg | Возникли проблемы с проверкой адреса электронной почты. Повторите попытку.|
|but_send_code | Отправить код проверки|
|but_verify_code | Проверить код|
|but_send_new_code | Отправить новый код|
|but_change_claims | Изменить адрес электронной почты|

Примечание. `intro_msg` элемент скрыт и не отображается на странице с автоматическим подтверждением. Чтобы сделать его видимым, используйте [HTML-кустомизтион](customize-ui-with-html.md) с каскадные таблицы стилей. Пример:
    
```css
.verificationInfoText div{display: block!important}
```

### <a name="verification-display-control-example"></a>Пример контрольного представления проверки

```xml
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
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
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="verification-display-control-user-interface-elements-deprecated"></a>Элементы пользовательского интерфейса элемента управления отображением проверки (не рекомендуется)

Ниже приведены идентификаторы для [элемента управления отображением проверки](display-control-verification.md) с [разметкой страницы версии](page-layout.md) 2.0.0.

| ID | Значение по умолчанию |
| -- | ------------- |
|verification_control_but_change_claims |Change |
|verification_control_fail_send_code |Отправить код не удалось. Повторите попытку позже. |
|verification_control_fail_verify_code |Проверить код не удалось. Повторите попытку позже. |
|verification_control_but_send_code |Отправить код |
|verification_control_but_send_new_code |Отправить новый код |
|verification_control_but_verify_code |Проверить код |
|verification_control_code_sent| Код проверки отправлен. Скопируйте его в поле ввода ниже. |

### <a name="verification-display-control-example-deprecated"></a>Пример контрольного представления проверки (не рекомендуется)

```xml
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
    <LocalizedString ElementType="UxElement" StringId="verification_control_but_change_claims">Change</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="verification_control_fail_send_code">Failed to send the code, please try again later.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="verification_control_fail_verify_code">Failed to verify the code, please try again later.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="verification_control_but_send_code">Send Code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="verification_control_but_send_new_code">Send New Code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="verification_control_but_verify_code">Verify Code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="verification_control_code_sent">Verification code has been sent. Please copy it to the input box below.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="restful-service-error-messages"></a>Сообщения об ошибках службы RESTful

Ниже приведены идентификаторы сообщений об ошибках [технического профиля службы RESTful](restful-technical-profile.md) .

| ID | Значение по умолчанию |
| -- | ------------- |
|дефаултусермессажеифрекуестфаилед | Не удалось установить подключение к конечной точке службы RESTful. URL-адрес службы RESTful: {0} |
|усермессажеифЦиркуитопен | {0} URL-адрес службы RESTful: {1} |
|усермессажеифднсресолутионфаилед | Не удалось разрешить имя узла конечной точки службы RESTful. URL-адрес службы RESTful: {0} |
|усермессажеифрекуесттимеаут | Не удалось установить подключение к конечной точке службы RESTful через ограничение времени ожидания (в {0} секундах). URL-адрес службы RESTful: {1} |


### <a name="restful-service-example"></a>Пример службы RESTful

```xml
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ErrorMessage" StringId="DefaultUserMessageIfRequestFailed">Failed to establish connection to restful service end point.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfCircuitOpen">Unable to connect to the restful service end point.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfDnsResolutionFailed">Failed to resolve the hostname of the restful service endpoint.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfRequestTimeout">Failed to establish connection to restful service end point within timeout limit.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="azure-ad-mfa-error-messages"></a>Сообщения об ошибках Azure AD MFA

Ниже приведены идентификаторы для сообщения об ошибке [технического профиля Azure AD MFA](multi-factor-auth-technical-profile.md) :

| ID | Значение по умолчанию |
| -- | ------------- |
|UserMessageIfCouldntSendSms | Не удается отправить SMS на телефон, попробуйте использовать другой номер телефона. |
|UserMessageIfInvalidFormat | Неправильный формат номера телефона, внесите исправления и повторите попытку.|
|UserMessageIfMaxAllowedCodeRetryReached | Неправильный код введен слишком много раз. Повторите попытку позже.|
|UserMessageIfServerError | Не удалось использовать службу MFA. Повторите попытку позже.|
|UserMessageIfThrottled | Ваш запрос отрегулирован, повторите попытку позже.|
|UserMessageIfWrongCodeEntered|Указан неправильный код, повторите попытку.|

### <a name="azure-ad-mfa-example"></a>Пример Azure AD MFA

```xml
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfCouldntSendSms">Cannot Send SMS to the phone, please try another phone number.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInvalidFormat">Your phone number is not in a valid format, please correct it and try again.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfMaxAllowedCodeRetryReached">Wrong code entered too many times, please try again later.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfServerError">Cannot use MFA service, please try again later.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfThrottled">Your request has been throttled, please try again later.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfWrongCodeEntered">Wrong code entered, please try again.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="azure-ad-sspr"></a>SSPR в Azure AD

Ниже приведены идентификаторы сообщений об ошибках [технического профиля Azure AD SSPR](aad-sspr-technical-profile.md) :

| ID | Значение по умолчанию |
| -- | ------------- |
|усермессажеифчалленжеекспиред | Срок действия кода истек.|
|усермессажеифинтерналеррор | В службе электронной почты произошла внутренняя ошибка, повторите попытку позже.|
|UserMessageIfThrottled | Вы отправили слишком много запросов, повторите попытку позже.|
|усермессажеифверификатионфаиледноретри | Превышено максимальное число попыток проверки.|
|UserMessageIfVerificationFailedRetryAllowed | Сбой проверки, повторите попытку.|


### <a name="azure-ad-sspr-example"></a>Пример Azure AD SSPR

```XML
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInternalError">We are having trouble verifying your email address. Please try again later.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfThrottled">There have been too many requests to verify this email address. Please wait a while, then try again.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfChallengeExpired">That code is expired. Please request a new code.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfVerificationFailedNoRetry">You've made too many incorrect attempts. Please try again later.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfVerificationFailedRetryAllowed">That code is incorrect. Please try again.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="one-time-password-error-messages"></a>Сообщения об ошибках одноразового пароля

Ниже приведены идентификаторы для сообщения об ошибках [технического профиля одноразового пароля](one-time-password-technical-profile.md):

| ID | Значение по умолчанию |
| -- | ------------- |
|UserMessageIfMaxRetryAttempted |При проверке подлинности превышено максимальное число попыток ввода одноразового пароля |
|UserMessageIfSessionDoesNotExist |Срок действия сеанса проверки пароля истек |
|UserMessageIfSessionConflict |Конфликт сеанса проверки одноразового пароля |
|UserMessageIfInvalidCode |Неправильный пароль, предоставленный для проверки |
|UserMessageIfVerificationFailedRetryAllowed |Этот код неправильный. Повторите попытку. | 

### <a name="one-time-password-example"></a>Пример с одноразовым паролем

```xml
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfSessionDoesNotExist">You have exceeded the maximum time allowed.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfMaxRetryAttempted">You have exceeded the number of retries allowed.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInvalidCode">You have entered the wrong code.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfSessionConflict">Cannot verify the code, please try again later.</LocalizedString>
   <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfVerificationFailedRetryAllowed">That code is incorrect. Please try again.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="claims-transformations-error-messages"></a>Сообщения об ошибках преобразования утверждений

Ниже приведены идентификаторы сообщений об ошибках преобразования утверждений.

| ID | Преобразование утверждений | Значение по умолчанию |
| -- | ------------- |------------- |
|UserMessageIfClaimsTransformationBooleanValueIsNotEqual |[AssertBooleanClaimIsEqualToValue](boolean-transformations.md#assertbooleanclaimisequaltovalue) | Сбой сравнения значений логического утверждения для типа утверждения "inputClaim".| 
|DateTimeGreaterThan |[AssertDateTimeIsGreaterThan](date-transformations.md#assertdatetimeisgreaterthan) | Сбой сравнения значений утверждений: Указанный левый операнд больше правого операнда.|
|UserMessageIfClaimsTransformationStringsAreNotEqual |[AssertStringClaimsAreEqual](string-transformations.md#assertstringclaimsareequal) | Сбой сравнения значений утверждений с использованием StringComparison "OrdinalIgnoreCase".|

### <a name="claims-transformations-example"></a>Пример преобразований утверждений

```xml
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsTransformationBooleanValueIsNotEqual">Your email address hasn't been verified.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="DateTimeGreaterThan">Expiration date must be greater that the current date.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsTransformationStringsAreNotEqual">The email entry fields do not match. Please enter the same email address in both fields and try again.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="next-steps"></a>Следующие шаги

Примеры локализации см. в следующих статьях:

- [Настройка языка с помощью настраиваемой политики в Azure Active Directory B2C](language-customization.md)
- [Настройка языка с помощью потоков пользователя в Azure Active Directory B2C](language-customization.md)
