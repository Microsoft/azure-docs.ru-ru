---
title: Регистрация и вход с помощью настраиваемых политик для телефона
titleSuffix: Azure AD B2C
description: Отправка одноразовых паролей (OTP) в текстовых сообщениях на телефоны пользователей приложения с помощью пользовательских политик в Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 09/01/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 2600ea3488c643bcf215b058425de42cd439dcff
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98660273"
---
# <a name="set-up-phone-sign-up-and-sign-in-with-custom-policies-in-azure-ad-b2c"></a>Настройка регистрации телефона и входа в систему с помощью пользовательских политик в Azure AD B2C

Регистрация и вход в систему в Azure Active Directory B2C (Azure AD B2C) позволяет пользователям регистрироваться и входить в приложения с помощью одноразового пароля (OTP), отправленного в текстовом сообщении на телефон. Одноразовые пароли могут помочь в минимальном риске пользователей, которые забывают или нарушают свой пароль.

Выполните действия, описанные в этой статье, чтобы использовать пользовательские политики, чтобы пользователи могли регистрироваться и входить в приложения с помощью одноразового пароля, отправленного на свой телефон.

## <a name="pricing"></a>Цены

Одноразовые пароли отправляются пользователям с помощью SMS-сообщений, и вам может потребоваться оплатить каждое отправленное сообщение. Сведения о ценах см. в разделе " **Специальные расходы** " статьи [Azure Active Directory B2C цены](https://azure.microsoft.com/pricing/details/active-directory-b2c/).

## <a name="user-experience-for-phone-sign-up-and-sign-in"></a>Взаимодействие с пользователем при регистрации и входе в телефон

При регистрации и входе с помощью телефона пользователь может зарегистрироваться в приложении, используя номер телефона в качестве основного идентификатора. Процесс регистрации и входа в систему конечного пользователя описан ниже.

> [!NOTE]
> Мы настоятельно рекомендуем включить сведения о согласии в процедуру регистрации и входа, как показано в приведенном ниже образце текста. Этот пример текста предназначен только для информационных целей. Ознакомьтесь с кратким руководством по мониторингу кода на [веб-сайте ктиа](https://www.ctia.org/programs) и обратитесь к своим юридическим или экспертам, чтобы получить рекомендации по окончательному тексту и настройке функций в соответствии с вашими требованиями.
>
> *Указав номер телефона, вы даете согласие на получение одноразового секретного кода, отправленного текстовым сообщением, для помощи при входе в систему для *&lt; вставки &gt; : имя приложения*. Могут применяться стандартные сообщения и скорости передачи данных.*
>
> *&lt;INSERT: ссылка на заявление о конфиденциальности&gt;*<br/>*&lt;INSERT: ссылка на условия предоставления услуг&gt;*

Чтобы добавить собственные сведения о согласии, настройте следующий пример. Включите его в `LocalizedResources` для контентдефинитион, используемого страницей с автоматическим подтверждением, с помощью элемента управления отображением (файл *Phone_Email_Base.xml* в [телефоне для регистрации и входа начальный пакет][starter-pack-phone]):

```xml
<LocalizedResources Id="phoneSignUp.en">        
    <LocalizedStrings>
    <LocalizedString ElementType="DisplayControl" ElementId="phoneControl" StringId="disclaimer_msg_intro">By providing your phone number, you consent to receiving a one-time passcode sent by text message to help you sign into {insert your application name}. Standard message and data rates may apply.</LocalizedString>          
    <LocalizedString ElementType="DisplayControl" ElementId="phoneControl" StringId="disclaimer_link_1_text">Privacy Statement</LocalizedString>                
    <LocalizedString ElementType="DisplayControl" ElementId="phoneControl" StringId="disclaimer_link_1_url">{insert your privacy statement URL}</LocalizedString>          
    <LocalizedString ElementType="DisplayControl" ElementId="phoneControl" StringId="disclaimer_link_2_text">Terms and Conditions</LocalizedString>             
    <LocalizedString ElementType="DisplayControl" ElementId="phoneControl" StringId="disclaimer_link_2_url">{insert your terms and conditions URL}</LocalizedString>          
    <LocalizedString ElementType="UxElement" StringId="initial_intro">Please verify your country code and phone number</LocalizedString>        
    </LocalizedStrings>      
</LocalizedResources>
   ```

### <a name="phone-sign-up-experience"></a>Интерфейс регистрации телефона

Если у пользователя еще нет учетной записи для своего приложения, его можно создать, выбрав ссылку **зарегистрироваться сейчас** . Появится страница регистрации, где пользователь выбирает **страну**, вводит свой номер телефона и выбирает **отправить код**.

![Пользователь начинает регистрацию телефона](media/phone-authentication/phone-signup-start.png)

Одноразовый код проверки отправляется на номер телефона пользователя. Пользователь вводит **код проверки** на странице регистрации, а затем выбирает команду **проверить код**. (Если пользователь не смог получить код, он может выбрать пункт **отправить новый код**.)

![Пользователь проверяет код во время регистрации телефона](media/phone-authentication/phone-signup-verify-code.png)

Пользователь вводит на странице регистрации любые другие запрашиваемые сведения. Например, **Отображаемое имя**, **имя** и **Фамилия** (страна и номер телефона остаются заполненными). Если пользователь хочет использовать другой номер телефона, он может выбрать **изменить номер** для перезапуска регистрации. По завершении пользователь нажмет кнопку **Continue (продолжить**).

![Пользователь предоставляет дополнительные сведения](media/phone-authentication/phone-signup-additional-info.png)

Затем пользователю предлагается ввести электронное сообщение для восстановления. Пользователь вводит свой адрес электронной почты, а затем выбирает **отправить код проверки**. Код отправляется в папку "Входящие" электронной почты пользователя, которую они могут получить и ввести в поле **код проверки** . Затем пользователь выбирает команду **проверить код**. 

После проверки кода пользователь выбирает **создать** , чтобы создать свою учетную запись. Если пользователь хочет использовать другой адрес электронной почты, он может выбрать **изменить электронное письмо**.

![Пользователь создает учетную запись](media/phone-authentication/email-verification.png)

### <a name="phone-sign-in-experience"></a>Интерфейс входа с телефона

Если у пользователя есть существующая учетная запись с номером телефона в качестве идентификатора, пользователь вводит свой номер телефона и выбирает **Continue (продолжить**). Они подтверждают страну и номер телефона, выбирая **Continue (продолжить**) и отправляют одноразовый код проверки на свой телефон. Пользователь вводит код проверки и выбирает **продолжение** входа.

![Взаимодействие с пользователем при входе с телефона](media/phone-authentication/phone-signin-screens.png)

## <a name="deleting-a-user-account"></a>Удаление учетной записи пользователя

В некоторых случаях может потребоваться удалить пользователя и связанные с ним данные из каталога Azure AD B2C. Дополнительные сведения об удалении учетной записи пользователя с помощью портал Azure см. в [этих инструкциях](/microsoft-365/compliance/gdpr-dsr-azure#step-5-delete). 

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-dsr-and-stp-note.md)]



## <a name="prerequisites"></a>Предварительные условия

Перед настройкой OTP вам потребуются следующие ресурсы.

* [клиент Azure AD B2C](tutorial-create-tenant.md);
* [Веб-приложение, зарегистрированное](tutorial-register-applications.md) в клиенте
* [Пользовательские политики](custom-policy-get-started.md) , отправленные в клиент

## <a name="get-the-phone-sign-up--sign-in-starter-pack"></a>Получение телефонного входа & начальный пакет входа в систему

Начните с обновления файлов настраиваемой политики регистрации и входа в телефон для работы с вашим клиентом Azure AD B2C.

1. Найдите [файлы настраиваемой политики регистрации и входа для телефона][starter-pack-phone] в локальном клоне репозитория начального пакета или скачайте их напрямую. Файлы политики XML находятся в следующем каталоге:

    `active-directory-b2c-custom-policy-starterpack/scenarios/`**`phone-number-passwordless`**

1. Замените строку `yourtenant` именем вашего клиента Azure AD B2C в каждом файле. Например, если имя вашего клиента B2C — *contosob2c*, все экземпляры `yourtenant.onmicrosoft.com` должны иметь вид `contosob2c.onmicrosoft.com`.

1. Выполните действия, описанные в разделе [Добавление идентификаторов приложений в пользовательскую политику](custom-policy-get-started.md#add-application-ids-to-the-custom-policy) статьи [Приступая к работе с пользовательскими политиками в Azure Active Directory B2C](custom-policy-get-started.md). В этом случае обновите `/phone-number-passwordless/` **`Phone_Email_Base.xml`** **идентификаторы приложения (клиента)** двух приложений, зарегистрированных при выполнении необходимых условий, *IdentityExperienceFramework* и *ProxyIdentityExperienceFramework*.

## <a name="upload-the-policy-files"></a>Отправка файлов политики

1. Войдите в [портал Azure](https://portal.azure.com) и перейдите к своему клиенту Azure AD B2C.
1. В разделе **Политики** выберите **Identity Experience Framework**.
1. Выберите **Отправить пользовательскую политику**.
1. Отправьте файлы политики в следующем порядке:
    1. *Phone_Email_Base.xml*
    1. *SignUpOrSignInWithPhone.xml*
    1. *SignUpOrSignInWithPhoneOrEmail.xml*
    1. *ProfileEditPhoneOnly.xml*
    1. *ProfileEditPhoneEmail.xml*
    1. *ChangePhoneNumber.xml*
    1. *PasswordResetEmail.xml*

При передаче каждого файла Azure добавляет префикс `B2C_1A_` .

## <a name="test-the-custom-policy"></a>Проверка пользовательской политики

1. В разделе **пользовательские политики** выберите **B2C_1A_SignUpOrSignInWithPhone**.
1. В разделе **Выбор приложения** выберите приложение *APP1* , которое вы зарегистрировали при выполнении необходимых условий.
1. В качестве **URL-адреса ответа выберите** `https://jwt.ms` .
1. Выберите **Запустить сейчас** и зарегистрируйтесь, используя адрес электронной почты или номер телефона.
1. Выберите **выполнить** еще раз и войдите с помощью той же учетной записи, чтобы подтвердить правильность конфигурации.

## <a name="get-user-account-by-phone-number"></a>Получение учетной записи пользователя по номеру телефона

Пользователь, который регистрируется с номером телефона без адреса электронной почты для восстановления, записывается в Azure AD B2C каталог с номером телефона в качестве имени для входа. Чтобы изменить номер телефона, ваша служба технической поддержки или специалисты по поддержке должны сначала найти свою учетную запись, а затем обновить свой номер телефона.

Вы можете найти пользователя по номеру телефона (имени для входа) с помощью [Microsoft Graph](microsoft-graph-operations.md):

```http
GET https://graph.microsoft.com/v1.0/users?$filter=identities/any(c:c/issuerAssignedId eq '+{phone number}' and c/issuer eq '{tenant name}.onmicrosoft.com')
```

Пример:

```http
GET https://graph.microsoft.com/v1.0/users?$filter=identities/any(c:c/issuerAssignedId eq '+450334567890' and c/issuer eq 'contosob2c.onmicrosoft.com')
```

## <a name="next-steps"></a>Дальнейшие действия

Начальный пакет и другие начальные пакеты пользовательской политики регистрации и входа в систему можно найти на сайте GitHub: [Azure-Samples/Active-Directory-B2C-Custom-Policy-starterpack/сценарии/Phone-No — без пароля][starter-pack-phone] . в файлах политики начального пакета используются технические профили многофакторной идентификации и преобразования заявок на телефонные номера.
* [Определение технического профиля многофакторной идентификации Azure AD](multi-factor-auth-technical-profile.md)
* [Определение преобразований заявок на телефонный номер](phone-number-claims-transformations.md)

<!-- LINKS - External -->
[starter-pack]: https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack
[starter-pack-phone]: https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/phone-number-passwordless
