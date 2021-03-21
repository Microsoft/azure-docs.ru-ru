---
title: Настройка языка в Azure Active Directory B2C
description: Узнайте, как настроить языковой интерфейс пользователя в Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 12/10/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 418f0797343a64728c4e48084b09bd0e426cec62
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101686416"
---
# <a name="language-customization-in-azure-active-directory-b2c"></a>Настройка языка в Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

Настройка языка в Azure Active Directory B2C (Azure AD B2C) позволяет добавить в поток пользователя различные языки в соответствии с потребностями ваших клиентов. Корпорация Майкрософт предоставляет переводы на [36 языках](#supported-languages). Кроме того, вы можете предоставить собственный перевод на любой язык. Даже если вам требуется только один язык, на странице можно настроить любой текст.

## <a name="how-language-customization-works"></a>Как работает функция настройки языка

Настройка языка позволяет выбрать языки, на которых доступен поток пользователя. Если эта функция включена, вы можете указать параметр строки запроса `ui_locales` в приложении. При вызове Azure AD B2C выполняется перевод страницы на указанный языковой стандарт. Такой тип конфигурации предоставляет вам полный контроль над языками, доступными потоку пользователя, а языковые параметры браузера клиента игнорируются.

Иногда вам может не понадобиться такой уровень контроля за тем, какие языки будут доступны пользователю. Если не указать параметр `ui_locales`, настройка языка будет зависеть от параметров браузера. Вы сможете управлять тем, какие языки будут доступны потоку пользователя, добавив их в качестве поддерживаемых. Если браузер клиента настроен для отображения на языке, который не требуется поддерживать, вместо него будет отображаться язык, выбранный по умолчанию для поддерживаемых языков.

* **язык, указанный в пользовательском интерфейсе**: после включения настройки языка поток пользователя будет переведен на указанный здесь язык.
* **Язык, запрошенный браузером**. Если `ui_locales` параметр не указан, поток пользователя преобразуется в требуемый браузером язык, *Если язык поддерживается*.
* **Язык политики по умолчанию**: Если в браузере не указан язык или он указывает, что он не поддерживается, поток пользователя преобразуется в язык пользовательского потока по умолчанию.

> [!NOTE]
> Если используются пользовательские атрибуты, необходимо предоставить собственный перевод. Дополнительные сведения см. в разделе [Настройка строк](#customize-your-strings).

::: zone pivot="b2c-custom-policy"

Локализация требует трех шагов: 

1. Настройка явного списка поддерживаемых языков
1. Предоставление строк и коллекций для каждого языка
1. Измените [Определение содержимого](contentdefinitions.md) для страницы. 

::: zone-end 

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

::: zone pivot="b2c-user-flow"

## <a name="support-requested-languages-for-ui_locales"></a>Поддержка запрашиваемых языков для ui_locales

Сначала необходимо включить функцию настройки языка для политик, которые были созданы до выхода общедоступного выпуска этой функции. Для политик и потоков пользователей, созданных позже, настройка языка включена по умолчанию.

Включив в потоке пользователя настройку языка, вы сможете управлять языками, доступными потоку пользователя, добавив параметр `ui_locales`.

1. В клиенте Azure AD B2C выберите **Потоки пользователей**.
1. Щелкните поток пользователя, который необходимо включить для переводов.
1. Щелкните **Языки**.
1. Выберите **Включение настройки языка**.

## <a name="select-which-languages-in-your-user-flow-are-enabled"></a>Выбор языков для включения в потоке пользователя

Включите набор языков для потока пользователя, на которые будет выполняться перевод по запросу браузера без параметра `ui_locales`.

1. Убедитесь, что в потоке пользователя включена настройка языка, как описано выше.
1. На странице **Языки** для потока пользователя выберите язык, который требуется поддерживать.
1. На панели свойств переведите переключатель **Включено** в положение **Да**.
1. Выберите **Сохранить** в верхней части панели свойств.

>[!NOTE]
>Если параметр `ui_locales` не указан, страница будет переведена на язык клиентского браузера только в случае включения этой функции.
>

## <a name="customize-your-strings"></a>Настройка строк

Настройка языка позволяет настроить любую строку в информации, передаваемой потоку пользователя.

1. Убедитесь, что в потоке пользователя включена настройка языка, как описано выше.
1. На странице **Языки** для потока пользователя выберите язык, который требуется настроить.
1. В разделе **Page-level-resources files** (Файлы ресурсов на уровне страницы) выберите страницу, которую требуется изменить.
1. Нажмите кнопку **Скачать значения по умолчанию** (или **Скачать переопределения**, если этот язык ранее менялся).

В результате будет создан JSON-файл, который можно использовать для изменения строк.

### <a name="change-any-string-on-the-page"></a>Замена любой строки на странице

1. Откройте скачанный JSON-файл в редакторе JSON.
1. Найдите элемент, требующий замены. Вы можете найти `StringId` искомой строки или найти атрибут `Value`, который вы хотите изменить.
1. Замените атрибут `Value` значением, которое требуется отобразить.
1. В каждой строке, которую необходимо изменить, замените `Override` на `true`.
1. Сохраните файл и отправьте изменения. (Элемент управления отправкой можно найти в том же месте, где вы загружали JSON-файл.)

> [!IMPORTANT]
> Если необходимо переопределить строку, измените значение `Override` на `true`. В противном случае запись не учитывается.

### <a name="change-extension-attributes"></a>Изменение атрибутов расширения

Если вы хотите изменить или добавить в JSON строку пользовательского атрибута, это нужно сделать в следующем формате:

```json
{
  "LocalizedStrings": [
    {
      "ElementType": "ClaimType",
      "ElementId": "extension_<ExtensionAttribute>",
      "StringId": "DisplayName",
      "Override": true,
      "Value": "<ExtensionAttributeValue>"
    }
    [...]
  ]
}
```

Замените `<ExtensionAttribute>` именем пользовательского атрибута.

Замените `<ExtensionAttributeValue>` новой строкой для отображения.

### <a name="provide-a-list-of-values-by-using-localizedcollections"></a>Укажите список значений с помощью LocalizedCollections

Если вы хотите предоставить список значений для ответов, необходимо создать атрибут `LocalizedCollections`. `LocalizedCollections` является массивом пар `Name` и `Value`. Порядок размещения элементов определяет порядок их отображения. Для добавления `LocalizedCollections` используйте следующий формат.

```json
{
  "LocalizedStrings": [...],
  "LocalizedCollections": [
    {
      "ElementType":"ClaimType",
      "ElementId":"<UserAttribute>",
      "TargetCollection":"Restriction",
      "Override": true,
      "Items":[
        {
          "Name":"<Response1>",
          "Value":"<Value1>"
        },
        {
          "Name":"<Response2>",
          "Value":"<Value2>"
        }
      ]
    }
  ]
}
```

* `ElementId` представляет собой пользовательский атрибут, ответом на который является `LocalizedCollections`.
* `Name` является значением, которое видит пользователь.
* `Value` возвращается в утверждении при выборе этого параметра.

### <a name="upload-your-changes"></a>Загрузка изменений

1. Внеся изменения в JSON-файл, вернитесь к клиенту B2C.
1. Щелкните **Потоки пользователей** и выберите поток пользователя, который необходимо включить для переводов.
1. Щелкните **Языки**.
1. Выберите язык, на который требуется выполнить перевод.
1. Выберите страницу, которую необходимо перевести.
1. Щелкните значок папки и выберите JSON-файл для отправки.

Изменения автоматически сохраняются в потоке пользователя.

## <a name="customize-the-page-ui-by-using-language-customization"></a>Настройка пользовательского интерфейса страницы с помощью настройки языка

Существует два способа локализации [HTML-содержимого](customize-ui-with-html.md). Одним из способов является включение [настройка языка](language-customization.md). Включение этой функции позволяет Azure AD B2C пересылать параметр OpenID Connect Connect `ui-locales` в конечную точку. Сервер содержимого может использовать этот параметр для предоставления настроенных страниц HTML на конкретном языке.

Кроме того, можно извлечь содержимое из разных расположений, в зависимости от используемого языкового стандарта. В конечной точке с поддержкой CORS можно настроить структуру папок для размещения содержимого для определенных языков. При использовании шаблона подстановки `{Culture:RFC5646}` запрос будет правильным. Например, предположим, что это URI вашей настраиваемой страницы.

```
https://wingtiptoysb2c.blob.core.windows.net/{Culture:RFC5646}/wingtip/unified.html
```

Страницу можно загрузить на `fr`. Когда страница подгружает содержимое HTML и CSS, она запрашивает его из:

```
https://wingtiptoysb2c.blob.core.windows.net/fr/wingtip/unified.html
```

## <a name="add-custom-languages"></a>Добавление пользовательских языков

Вы также можете добавить языки, на которые корпорация Майкрософт в настоящее время не предоставляет переводы. Необходимо будет предоставить переводы для всех строк в потоке пользователя. Язык и коды языковых стандартов ограничиваются предусмотренными в стандарте ISO 639-1. Например, формат кода локали должен быть "ISO_639-1_code"-"Каунтрикоде" `en-GB` . Дополнительные сведения о форматах идентификатора языкового стандарта см. на https://docs.microsoft.com/openspecs/office_standards/ms-oe376/6c085406-a698-4e12-9d4d-c3b0ee3dbc4a

1. В клиенте Azure AD B2C выберите **Потоки пользователей**.
2. Щелкните поток пользователя, в котором необходимо добавить пользовательские языки, и выберите **Языки**.
3. В верхней части страницы выберите **Добавить пользовательский язык**.
4. В открывшейся области контекста определите язык, для которого предоставляется перевод, введя код допустимого языкового стандарта.
5. Для каждой страницы можно загрузить набор переопределений для английского языка и начать работу над переводами.
6. По завершении операций с JSON-файлами их можно загрузить для каждой страницы.
7. Выберите **Включить**, после чего поток пользователя сможет отображать этот язык для пользователей.
8. Сохраните язык.

>[!IMPORTANT]
>Прежде чем вы сможете сохранить, необходимо включить пользовательские языки или отправить переопределения для них.

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="set-up-the-list-of-supported-languages"></a>Настройка списка поддерживаемых языков

Откройте файл расширения политики. например <em>`SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`**</em>.

1. Найдите элемент [BuildingBlocks](buildingblocks.md). Если такой элемент не существует, добавьте его.
1. Добавьте элемент `Localization` с поддерживаемыми языками: английским (по умолчанию) и испанским.  


```xml
<Localization Enabled="true">
  <SupportedLanguages DefaultLanguage="en" MergeBehavior="ReplaceAll">
    <SupportedLanguage>en</SupportedLanguage>
    <SupportedLanguage>es</SupportedLanguage>
  </SupportedLanguages>
</Localization>
```

## <a name="provide-language-specific-labels"></a>Указание меток языка

Атрибут [LocalizedResources](localization.md#localizedresources) элемента `Localization` содержит список локализованных строк. Элемент локализованных ресурсов содержит идентификатор, который используется для уникальной идентификации локализованных ресурсов. Этот идентификатор используется позже в элементе [определения содержимого](contentdefinitions.md).

Элементы локализованного содержимого следует определить для каждого определения содержимого и языка, поддержка которого требуется. Чтобы настроить единую страницу регистрации или входа на английском и испанском языках, добавьте следующие элементы `LocalizedResources` после элемента `</SupportedLanguages>`.

> [!NOTE]
> В следующем примере мы добавили символ фунта в `#` начале каждой строки, чтобы можно было легко найти локализованные метки на экране.

```xml
<!--Local account sign-up or sign-in page English-->
<Localization Enabled="true">
  ...
  <LocalizedResources Id="api.signuporsignin.en">
    <LocalizedStrings>
      <LocalizedString ElementType="UxElement" StringId="logonIdentifier_email">#Email Address</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="requiredField_email">#Please enter your email</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="logonIdentifier_username">#Username</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="password">#Password</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="createaccount_link">#Sign up now</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="requiredField_username">#Please enter your user name</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="createaccount_intro">#Don't have an account?</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="forgotpassword_link">#Forgot your password?</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="divider_title">#OR</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="cancel_message">#The user has forgotten their password</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="button_signin">#Sign in</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="social_intro">#Sign in with your social account</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="requiredField_password">#Please enter your password</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="invalid_password">#The password you entered is not in the expected format.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="local_intro_username">#Sign in with your user name</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="local_intro_email">#Sign in with your existing account</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="invalid_email">#Please enter a valid email address</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="unknown_error">#We are having trouble signing you in. Please try again later.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="email_pattern">^[a-zA-Z0-9.!#$%&amp;'^_`{}~-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*$</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInvalidPassword">#Your password is incorrect.</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsPrincipalDoesNotExist">#We can't seem to find your account.</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfOldPasswordUsed">#Looks like you used an old password.</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="DefaultMessage">#Invalid username or password.</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfUserAccountDisabled">#Your account has been locked. Contact your support person to unlock it, then try again.</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfUserAccountLocked">#Your account is temporarily locked to prevent unauthorized use. Try again later.</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="AADRequestsThrottled">#There are too many requests at this moment. Please wait for some time and try again.</LocalizedString>
    </LocalizedStrings>
  </LocalizedResources>
  <!--Local account sign-up or sign-in page Spanish-->
  <LocalizedResources Id="api.signuporsignin.es">
    <LocalizedStrings>
      <LocalizedString ElementType="UxElement" StringId="logonIdentifier_email">#Correo electrónico</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="requiredField_email">#Este campo es obligatorio</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="logonIdentifier_username">#Nombre de usuario</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="password">#Contraseña</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="createaccount_link">#Registrarse ahora</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="requiredField_username">#Escriba su nombre de usuario</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="createaccount_intro">#¿No tiene una cuenta?</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="forgotpassword_link">#¿Olvidó su contraseña?</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="divider_title">#O</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="cancel_message">#El usuario ha olvidado su contraseña</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="button_signin">#Iniciar sesión</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="social_intro">#Iniciar sesión con su cuenta de redes sociales</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="requiredField_password">#Escriba su contraseña</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="invalid_password">#La contraseña que ha escrito no está en el formato esperado.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="local_intro_username">#Iniciar sesión con su nombre de usuario</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="local_intro_email">#Iniciar sesión con su cuenta existente</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="invalid_email">#Escriba una dirección de correo electrónico válida</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="unknown_error">#Tenemos problemas para iniciar su sesión. Vuelva a intentarlo más tarde.  </LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="email_pattern">^[a-zA-Z0-9.!#$%&amp;'^_`{}~-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*$</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInvalidPassword">#Su contraseña es incorrecta.</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsPrincipalDoesNotExist">#Parece que no podemos encontrar su cuenta.</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfOldPasswordUsed">#Parece que ha usado una contraseña antigua.</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="DefaultMessage">#El nombre de usuario o la contraseña no son válidos.</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfUserAccountDisabled">#Se bloqueó su cuenta. Póngase en contacto con la persona responsable de soporte técnico para desbloquearla y vuelva a intentarlo.</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfUserAccountLocked">#Su cuenta se bloqueó temporalmente para impedir un uso no autorizado. Vuelva a intentarlo más tarde.</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="AADRequestsThrottled">#Hay demasiadas solicitudes en este momento. Espere un momento y vuelva a intentarlo.</LocalizedString>
    </LocalizedStrings>
  </LocalizedResources>
  <!--Local account sign-up page English-->
  <LocalizedResources Id="api.localaccountsignup.en">
    <LocalizedStrings>
      <LocalizedString ElementType="ClaimType" ElementId="email" StringId="DisplayName">#Email Address</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="email" StringId="UserHelpText">#Email address that can be used to contact you.</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="email" StringId="PatternHelpText">#Please enter a valid email address.</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="newPassword" StringId="DisplayName">#New Password</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="newPassword" StringId="UserHelpText">#Enter new password</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="newPassword" StringId="PatternHelpText">#8-16 characters, containing 3 out of 4 of the following: Lowercase characters, uppercase characters, digits (0-9), and one or more of the following symbols: @ # $ % ^ &amp; * - _ + = [ ] { } | \ : ' , ? / ` ~ " ( ) ; .</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="reenterPassword" StringId="DisplayName">#Confirm New Password</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="reenterPassword" StringId="UserHelpText">#Confirm new password</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="reenterPassword" StringId="PatternHelpText">#8-16 characters, containing 3 out of 4 of the following: Lowercase characters, uppercase characters, digits (0-9), and one or more of the following symbols: @ # $ % ^ &amp; * - _ + = [ ] { } | \ : ' , ? / ` ~ " ( ) ; .</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="displayName" StringId="DisplayName">#Display Name</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="displayName" StringId="UserHelpText">#Your display name.</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="surname" StringId="DisplayName">#Surname</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="surname" StringId="UserHelpText">#Your surname (also known as family name or last name).  </LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="givenName" StringId="DisplayName">#Given Name</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="givenName" StringId="UserHelpText">#Your given name (also known as first name).</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="button_continue">#Create</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="error_fieldIncorrect">#One or more fields are filled out incorrectly. Please check your entries and try again.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="error_passwordEntryMismatch">#The password entry fields do not match. Please enter the same password in both fields and try again.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="error_requiredFieldMissing">#A required field is missing. Please fill out all required fields and try again.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="helplink_text">#What is this?</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="initial_intro">#Please provide the following details.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="preloader_alt">#Please wait</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="required_field">#This information is required.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_but_edit">#Change e-mail</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_but_resend">#Send new code</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_but_send">#Send verification code</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_but_verify">#Verify code</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_fail_code_expired">#That code is expired. Please request a new code.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_fail_no_retry">#You've made too many incorrect attempts. Please try again later.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_fail_retry">#That code is incorrect. Please try again.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_fail_server">#We are having trouble verifying your email address. Please enter a valid email address and try again.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_fail_throttled">#There have been too many requests to verify this email address. Please wait a while, then try again.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_info_msg">#Verification code has been sent to your inbox. Please copy it to the input box below.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_input">#Verification code</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_intro_msg">#Verification is necessary. Please click Send button.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_success_msg">#E-mail address verified. You can now continue.</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="ServiceThrottled">#There are too many requests at this moment. Please wait for some time and try again.</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimNotVerified">#Claim not verified: {0}</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsPrincipalAlreadyExists">#A user with the specified ID already exists. Please choose a different one.</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfIncorrectPattern">#Incorrect pattern for: {0}</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInvalidInput">#{0} has invalid input.</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfMissingRequiredElement">#Missing required element: {0}</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfValidationError">#Error in validation by: {0}</LocalizedString>
    </LocalizedStrings>
  </LocalizedResources>
  <!--Local account sign-up page Spanish-->
  <LocalizedResources Id="api.localaccountsignup.es">
    <LocalizedStrings>
      <LocalizedString ElementType="ClaimType" ElementId="email" StringId="DisplayName">#Dirección de correo electrónico</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="email" StringId="UserHelpText">#Dirección de correo electrónico que puede usarse para ponerse en contacto con usted.</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="email" StringId="PatternHelpText">#Introduzca una dirección de correo electrónico válida.  </LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="newPassword" StringId="DisplayName">#Nueva contraseña</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="newPassword" StringId="UserHelpText">#Escriba la contraseña nueva</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="newPassword" StringId="PatternHelpText">#De 8 a 16 caracteres, que contengan 3 de los 4 tipos siguientes: caracteres en minúsculas, caracteres en mayúsculas, dígitos (0-9) y uno o más de los siguientes símbolos: @ # $ % ^ &amp; * - _ + = [ ] { } | \\ : ' , ? / ` ~ \" ( ) ; .</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="reenterPassword" StringId="DisplayName">#Confirmar nueva contraseña</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="reenterPassword" StringId="UserHelpText">#Confirmar nueva contraseña</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="reenterPassword" StringId="PatternHelpText">#8 a 16 caracteres, que contengan 3 de los 4 tipos siguientes: caracteres en minúsculas, caracteres en mayúsculas, dígitos (0-9) y uno o más de los siguientes símbolos: @ # $ % ^ &amp; * - _ + = [ ] { } | \\ : ' , ? / ` ~ \" ( ) ; .</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="displayName" StringId="DisplayName">#Nombre para mostrar</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="displayName" StringId="UserHelpText">#Su nombre para mostrar.</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="surname" StringId="DisplayName">#Apellido</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="surname" StringId="UserHelpText">#Su apellido.</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="givenName" StringId="DisplayName">#Nombre</LocalizedString>
      <LocalizedString ElementType="ClaimType" ElementId="givenName" StringId="UserHelpText">#Su nombre (también conocido como nombre de pila).</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="button_continue">#Crear</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="error_fieldIncorrect">#Hay uno o varios campos rellenados de forma incorrecta. Compruebe las entradas y vuelva a intentarlo.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="error_passwordEntryMismatch">#Los campos de entrada de contraseña no coinciden. Escriba la misma contraseña en ambos campos y vuelva a intentarlo.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="error_requiredFieldMissing">#Falta un campo obligatorio. Rellene todos los campos necesarios y vuelva a intentarlo.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="helplink_text">#¿Qué es esto?</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="initial_intro">#Proporcione los siguientes detalles.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="preloader_alt">#Espere</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="required_field">#Esta información es obligatoria.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_but_edit">#Cambiar correo electrónico</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_but_resend">#Enviar nuevo código</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_but_send">#Enviar código de comprobación</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_but_verify">#Comprobar código</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_fail_code_expired">#El código ha expirado. Solicite otro nuevo.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_fail_no_retry">#Ha realizado demasiados intentos incorrectos. Vuelva a intentarlo más tarde.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_fail_retry">#Ese código es incorrecto. Inténtelo de nuevo.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_fail_server">#Tenemos problemas para comprobar la dirección de correo electrónico. Escriba una dirección de correo electrónico válida y vuelva a intentarlo.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_fail_throttled">#Ha habido demasiadas solicitudes para comprobar esta dirección de correo electrónico. Espere un poco y vuelva a intentarlo.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_info_msg">#Se ha enviado el código de verificación a su Bandeja de entrada. Cópielo en el siguiente cuadro de entrada.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_input">#Código de verificación</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_intro_msg">#La comprobación es obligatoria. Haga clic en el botón Enviar.</LocalizedString>
      <LocalizedString ElementType="UxElement" StringId="ver_success_msg">#Dirección de correo electrónico comprobada. Puede continuar.</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="ServiceThrottled">#Hay demasiadas solicitudes en este momento. Espere un momento y vuelva a intentarlo.</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimNotVerified">#Reclamación no comprobada: {0}</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsPrincipalAlreadyExists">#Ya existe un usuario con el id. especificado. Elija otro diferente.</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfIncorrectPattern">#Patrón incorrecto para: {0}</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInvalidInput">#{0} tiene una entrada no válida.</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfMissingRequiredElement">#Falta un elemento obligatorio: {0}</LocalizedString>
      <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfValidationError">#Error en la validación de: {0}</LocalizedString>
    </LocalizedStrings>
  </LocalizedResources>
</Localization>
```

## <a name="edit-the-content-definition-with-the-localization"></a>Изменение определения содержимого с помощью локализации

Вставьте содержимое скопированного элемента ContentDefinitions в качестве дочернего элемента BuildingBlocks.

В следующем примере для страницы регистрации или входа добавляются пользовательские строки на английском (en) и испанском (es) языках, включая страницу регистрации локальной учетной записи. Элемент **LocalizedResourcesReferenceId** для каждого **LocalizedResourcesReference** совпадает с именем языкового стандарта, но в качестве идентификатора можно использовать любую строку. Для каждой комбинации языка и страницы укажите соответствующие элементы **LocalizedResources**, созданные ранее.

```xml
<ContentDefinitions>
  <ContentDefinition Id="api.signuporsignin">
    <LocalizedResourcesReferences MergeBehavior="Prepend">
        <LocalizedResourcesReference Language="en" LocalizedResourcesReferenceId="api.signuporsignin.en" />
        <LocalizedResourcesReference Language="es" LocalizedResourcesReferenceId="api.signuporsignin.es" />
    </LocalizedResourcesReferences>
  </ContentDefinition>

  <ContentDefinition Id="api.localaccountsignup">
    <LocalizedResourcesReferences MergeBehavior="Prepend">
        <LocalizedResourcesReference Language="en" LocalizedResourcesReferenceId="api.localaccountsignup.en" />
        <LocalizedResourcesReference Language="es" LocalizedResourcesReferenceId="api.localaccountsignup.es" />
    </LocalizedResourcesReferences>
  </ContentDefinition>
</ContentDefinitions>
```

##  <a name="upload-and-test-your-updated-custom-policy"></a>Передача и проверка обновленной настраиваемой политики

### <a name="upload-the-custom-policy"></a>Передача настраиваемой политики

1. Сохраните файл расширений.
1. Убедитесь, что используете каталог с клиентом Azure AD B2C, выбрав фильтр **Каталог и подписка** в меню вверху и каталог с вашим клиентом.
1. Найдите и выберите **Azure AD B2C**.
1. В разделе **Политики** выберите **Identity Experience Framework**.
1. Выберите **Отправить пользовательскую политику**.
1. Выполните отправку ранее измененного файла расширений.

### <a name="test-the-custom-policy-by-using-run-now"></a>Тестирование настраиваемой политики с помощью параметра **Запустить сейчас**

1. Выберите переданную политику и щелкните **Запустить сейчас**.
1. Отобразится локализованная страница регистрации или входа.
1. Щелкните ссылку для регистрации, и вы увидите локализованную страницу регистрации.
1. Переключите язык браузера по умолчанию на испанский. Или можно добавить параметр строки запроса `ui_locales` в запрос авторизации. Пример: 

```http
https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/B2C_1A_signup_signin/oauth2/v2.0/authorize&client_id=0239a9cc-309c-4d41-12f1-31299feb2e82&nonce=defaultNonce&redirect_uri=https%3A%2F%2Fjwt.ms&scope=openid&response_type=id_token&prompt=login&ui_locales=es
```

::: zone-end


## <a name="additional-information"></a>Дополнительные сведения

::: zone pivot="b2c-user-flow"

### <a name="page-ui-customization-labels-as-overrides"></a>Использование меток настройки пользовательского интерфейса страницы для переопределения параметров страницы

При включении настройки языка все предыдущие изменения меток, использующих настройки пользовательского интерфейса страницы, сохраняются в JSON-файле для английского языка (en). Вы можете продолжить изменять метки и другие текстовые строки, отправив языковые ресурсы в настройках языка.

::: zone-end

### <a name="up-to-date-translations"></a>Актуальные переводы

Корпорация Майкрософт стремится предоставить наиболее актуальные переводы Мы постоянно улучшаем переводы в соответствии с вашими требованиями. Мы выявляем ошибки и изменения в глобальной терминологии и обновляем ее, обеспечивая непрерывную поддержку в потоке пользователя.

### <a name="support-for-right-to-left-languages"></a>Поддержка языков с написанием справа налево

Корпорация Майкрософт сейчас не поддерживает языки с написанием справа налево. Их поддержку можно реализовать, используя пользовательские языковые стандарты и каскадные таблицы стилей для изменения способа отображения строк. При необходимости такой поддержки проголосуйте на [этом форуме для пользователей Azure](https://feedback.azure.com/forums/169401-azure-active-directory/suggestions/19393000-provide-language-support-for-right-to-left-languag).

### <a name="social-identity-provider-translations"></a>Переводы для поставщиков удостоверений в социальных сетях

Корпорация Майкрософт предоставляет параметр `ui_locales` OIDC для имен входа в социальные сети. Однако некоторые поставщики удостоверений в социальных сетях, включая Facebook и Google, не принимают его.

### <a name="browser-behavior"></a>Поведение браузера

И Chrome, и Firefox запрашивают настроенный язык. Если этот язык поддерживается, ему отдается предпочтение над языком, заданным по умолчанию. Сейчас Microsoft Edge не запрашивает язык и сразу отображает язык по умолчанию.

## <a name="supported-languages"></a>Поддерживаемые языки

Azure AD B2C включает поддержку следующих языков. Языки потока пользователей предоставляются Azure AD B2C. Языки уведомлений многофакторной идентификации (MFA) предоставляются [Azure AD MFA](../active-directory/authentication/concept-mfa-howitworks.md).

| Язык              | Код языка | Маршруты пользователей         | Уведомления MFA  |
|-----------------------| :-----------: | :----------------: | :----------------: |
| Арабский                | ar            | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Болгарский             | bg            | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Бенгальский                | bn            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) |
| Каталонский               | ca            | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Чешский                 | cs            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Датский                | da            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Немецкий                | de            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Греческий                 | el            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Английский               | en            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Испанский               | es            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Эстонский              | et            | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Баскский                | eu            | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Финский               | fi            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Французский                | fr            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Галисийский              | gl            | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Гуджарати              | gu            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) |
| Иврит                | he            | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Hindi                 | hi            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Хорватский              | hr            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Венгерский             | hu            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Индонезийский            | идентификатор            | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Итальянский               | it            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Японский              | ja            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Казахский                | kk            | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Каннада               | kn            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) |
| Корейский                | ko            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Литовский            | lt            | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Латышский               | lv            | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Малаялам             | ml            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) |
| Маратхи               | mr            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) |
| Малайский                 | ms            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Норвежский (букмол)      | nb            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) |
| Нидерландский                 | nl            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Норвежский             | Нет            | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Панджаби               | pa            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) |
| Польский                | pl            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Португальский (Бразилия)   | pt-br         | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Португальский (Португалия) | pt-pt         | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Румынский              | ro            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Русский               | ru            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Словацкий                | sk            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Словенский             | sl            | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Сербский — кириллица    | sr-cryl-cs    | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Сербский — латиница       | sr-latn-cs    | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Шведский               | sv            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Тамильский                 | ta            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) |
| Телугу                | te            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) |
| Тайский                  | th            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Турецкий               | tr            | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Украинский             | uk            | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Вьетнамский            | vi            | ![X, указывающее на отсутствие.](./media/user-flow-language-customization/no.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Китайский (упрощенное письмо)  | zh-hans       | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |
| Китайский (традиционное письмо) | zh-hant       | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) | ![Зеленая галочка.](./media/user-flow-language-customization/yes.png) |


## <a name="next-steps"></a>Дальнейшие действия

::: zone pivot="b2c-user-flow"

Дополнительные сведения о настройке пользовательского интерфейса приложения в [Azure Active Directory B2C](customize-ui-with-html.md)см. в этой статье.

::: zone-end 

::: zone pivot="b2c-custom-policy"

- Узнайте больше об элементе [локализации](localization.md) в справочных материалах по IEF.
- Ознакомьтесь со списком [идентификаторов строк локализации](localization-string-ids.md), доступных в Azure AD B2C.

::: zone-end 
