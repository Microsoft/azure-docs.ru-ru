---
title: Типингдна с Azure Active Directory B2C
titleSuffix: Azure AD B2C
description: Узнайте, как интегрировать проверку подлинности Azure AD B2C с Типингдна, чтобы помочь в проверке личности на основе шаблона ввода данных, предоставляет решения для проверки кода, которые принудительно проходят многофакторную проверку подлинности и помогают соблюдать требования к SCA для директивы оплаты 2 (PSD2).
author: gargi-sinha
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 06/25/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 68617d86fda940c5d3752f2389088a8c729aebec
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97108354"
---
# <a name="tutorial-for-configuring-typingdna-with-azure-active-directory-b2c"></a>Руководство по настройке Типингдна с помощью Azure Active Directory B2C

В этом пошаговом руководстве описано, как интегрировать пример Интернет-приложения оплаты в Azure Active Directory B2C с приложением Типингдна. Используя приложение Типингдна, клиенты Azure AD B2C могут соответствовать требованиям к транзакциям [директивы оплаты 2](https://www.typingdna.com/use-cases/sca-strong-customer-authentication) (PSD2) с помощью нажатия клавиши Dynamics и надежной проверки подлинности клиента. Дополнительные сведения о Типингдна см. [здесь](https://www.typingdna.com/).

 Azure AD B2C использует технологии Типингдна для записи пользовательских характеристик и их записи и анализа для ознакомления с каждой проверкой подлинности. Это добавляет уровень защиты, связанный с степенью риска проверки подлинности, и оценивает уровни риска. Azure AD B2C могут вызывать другие механизмы для предоставления дополнительной уверенности, что пользователь является его автором, вызывая Azure AD MFA, принудительную проверку электронной почты или любую другую пользовательскую логику для вашего сценария.

>[!NOTE]
> Этот пример политики основан на [соЦиаландлокалаккаунтсвисмфа](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/SocialAndLocalAccountsWithMfa) Starter Pack.

## <a name="scenario-description"></a>Описание сценария

![Снимок экрана: схема архитектуры Типингдна](./media/partner-typingdna/typingdna-architecture-diagram.png)

### <a name="sign-up"></a>Регистрация

1. Azure AD B2C страницы используют библиотеку JavaScript Типингдна для записи шаблона ввода пользователя. Например, имя пользователя и пароль записываются при регистрации для первоначальной регистрации, а затем при каждом входе в систему для проверки.

2. Когда пользователь отправляет страницу, Библиотека Типингдна вычислит характеристики ввода для пользователя. После этого вставьте данные в скрытое текстовое поле, которое Azure AD B2C отобразить. Это поле скрыто с помощью CSS.  

    [Пример содержит HTML-файлы](https://github.com/azure-ad-b2c/partner-integrations/blob/master/samples/TypingDNA/source-code/selfAssertedSignUp.cshtml) с изменениями JavaScript и CSS, на которые ссылаются `api.selfasserted.tdnasignin` `api.selfasserted.tdnasignup` определения содержимого и. Сведения об размещении HTML-файлов см. в разделе [Размещение содержимого страницы](./customize-ui-with-html.md#hosting-the-page-content) .

3. Azure AD B2C теперь имеет шаблон ввода в контейнере утверждений, когда пользователь отправляет свои учетные данные. Он должен вызвать API (Your) для передачи этих данных в конечную точку REST API Типингдна. Этот API включен в [пример (типингдна-API-Interface)](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/TypingDNA/source-code/TypingDNA-API-Interface).
4. Затем API среднего уровня передает данные шаблона ввода в Типингдна REST API. При регистрации вызывается [Пользовательская конечная точка проверки](https://api.typingdna.com/index.html#api-API_Services-GetUser) , чтобы подтвердить, что пользователь не существовал, а затем вызывается конечная точка [сохранения шаблона](https://api.typingdna.com/index.html#api-API_Services-saveUserPattern) , чтобы сохранить первый шаблон ввода пользователя.

> [!NOTE]
> Все вызовы конечной точки REST API Типингдна отправляют UserId. Это должно быть хэшированное значение. Azure AD B2C использует `HashObjectIdWithEmail` Преобразование утверждений для хэширования сообщения электронной почты с случайными Salt-и секретными данными.  

REST APIные вызовы моделируются `validationTechnicalProfiles` в `LocalAccountSignUpWithLogonEmail-TDNA` :

```xml

<ValidationTechnicalProfiles>

    <ValidationTechnicalProfile ReferenceId="AAD-UserWriteUsingLogonEmail-TDNA" />
    <ValidationTechnicalProfile ReferenceId="REST-TDNA-CheckUser" ContinueOnError="true"/>

    <ValidationTechnicalProfile ReferenceId="REST-TDNA-SaveUser"/>

</ValidationTechnicalProfiles>

```

### <a name="sign-in"></a>Вход

При последующем входе пользовательский шаблон будет вычисляться точно так же, как при регистрации с помощью [ПОЛЬЗОВАТЕЛЬСКОГО HTML](https://github.com/azure-ad-b2c/partner-integrations/blob/master/samples/TypingDNA/source-code/selfAssertedSignIn.cshtml). После того, как профиль ввода находится в Azure AD B2C контейнере утверждений, Azure AD B2C вызовет API для вызова конечной точки REST API в Типингдна. Для подтверждения существования пользователя вызывается конечная точка [пользователя Check](https://api.typingdna.com/index.html#api-API_Services-GetUser) . Затем вызывается проверка конечной точки [шаблона](https://api.typingdna.com/index.html#api-API_Services-verifyTypingPattern) для возврата `net_score` . Это `net_score` указывает, как закрывать шаблон ввода в первоначальной момент при регистрации.

Этот шаблон ввода моделируется `validationTechnicalProfiles` в `SelfAsserted-LocalAccountSignin-Email-TDNA` :

```xml

<ValidationTechnicalProfiles>

    <ValidationTechnicalProfile ReferenceId="login-NonInteractive"/>

    <ValidationTechnicalProfile ReferenceId="REST-TDNA-CheckUser" ContinueOnError="false"/>

    <ValidationTechnicalProfile ReferenceId="REST-TDNA-VerifyUser"/>

    <ValidationTechnicalProfile ReferenceId="REST-TDNA-SaveUser">

        <Preconditions>

            <Precondition Type="ClaimEquals" ExecuteActionsIf="true">

            <Value>saveTypingPattern</Value>

            <Value>False</Value>

            <Action>SkipThisValidationTechnicalProfile</Action>

            </Precondition>

        </Preconditions>

    </ValidationTechnicalProfile>

</ValidationTechnicalProfiles>

```

 Если пользователь получает слишком большой шаблон ввода `net_score` , его можно сохранить с помощью конечной точки типингдна [сохранить шаблон ввода](https://api.typingdna.com/index.html#api-API_Services-saveUserPattern) .  

API должен возвращать утверждение,  `saveTypingPattern` Если вы хотите, чтобы конечная точка шаблона ввода типингдна была вызвана с помощью Azure AD B2C (через API).

Пример в репозитории содержит API (Типингдна-API-Interface), который настроен со следующими свойствами.

- Режим обучения. Если пользователь имеет менее двух сохраненных шаблонов, всегда запрашивает MFA.

- Если у пользователя есть 2-5 шаблонов, а значение меньше `net_score` 50, запросите mfa.

- Если у пользователя есть 5 шаблонов, а значение меньше `net_score` 65, запросите mfa.

Эти пороговые значения должны быть скорректированы в вашем варианте использования.

- После того как API оценит `net_score` , он должен вернуть логическое утверждение B2C- `promptMFA` .

- `promptMFA`Утверждение используется в предварительном условии для условного выполнения Azure AD mfa.

```xml

<OrchestrationStep Order="3" Type="ClaimsExchange">

    <Preconditions>

        <Precondition Type="ClaimsExist" ExecuteActionsIf="true">

            <Value>isActiveMFASession</Value>

            <Action>SkipThisOrchestrationStep</Action>

        </Precondition>

        <Precondition Type="ClaimEquals" ExecuteActionsIf="true">

            <Value>promptMFA</Value>

            <Value>False</Value>

            <Action>SkipThisOrchestrationStep</Action>

        </Precondition>

    </Preconditions>

    <ClaimsExchanges>

        <ClaimsExchange Id="PhoneFactor-Verify" TechnicalProfileReferenceId="PhoneFactor-InputOrVerify" />

    </ClaimsExchanges>

</OrchestrationStep>

```

## <a name="onboard-with-typingdna"></a>Подключение с помощью Типингдна

1. Зарегистрируйтесь для Типингдна [здесь](https://www.typingdna.com/)
2. Войдите на панель мониторинга Типингдна и получите **ключ API** и **секрет API**. Это потребуется в настройке интерфейса API позже

## <a name="integrate-typingdna-with-azure-ad-b2c"></a>Интеграция Типингдна с Azure AD B2C

1. Размещение [интерфейса типингдна-API](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/TypingDNA/source-code/TypingDNA-API-Interface) на выбранном поставщике услуг размещения
2. Замените все экземпляры `apiKey` и `apiSecret` в решении [типингдна-API-Interface](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/TypingDNA/source-code/TypingDNA-API-Interface) на учетные данные из панели мониторинга типингдна.
3. Разместите HTML-файлы поставщика по своему усмотрению, следуя требованиям CORS [здесь](./customize-ui-with-html.md#3-configure-cors) .
4. Замените элементы LoadURI для `api.selfasserted.tdnasignup` `api.selfasserted.tdnasignin` определений содержимого и в `TrustFrameworkExtensions.xml` файле на URI размещенных HTML-файлов соответственно.
5. Создайте ключ политики B2C в разделе Инфраструктура процедур идентификации в колонке Azure AD в **портал Azure**. Используйте `Generate` параметр и назовите этот ключ `tdnaHashedId` .
6. Замена TenantId в файлах политики
7. Замените Сервицеурлс во всех технических профилях Типингдна REST API (RESTFUL-ТДНА-Верифюсер, RESTFUL-ТДНА-Савеусер, RESTFUL-ТДНА-Чеккусер) конечной точкой для [API-интерфейса типингдна-API](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/TypingDNA/source-code/TypingDNA-API-Interface).
8. Отправьте [файлы политики](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/TypingDNA/policy) в клиент.

## <a name="test-the-user-flow"></a>Тестирование потока пользователя

1. Откройте клиент B2C и выберите инфраструктура процедур идентификации.
2. Выберите ранее созданный **поток пользователя**.
3. Выбор **запуска** потока пользователя

    а. **Приложение** — выбор зарегистрированного приложения (пример — JWT)

    b. **URL-адрес ответа** — выберите URL-адрес перенаправления

    c. Выберите **Выполнить поток пользователя**.
  
4. Последовательно выберите процесс регистрации и создайте учетную запись.
5. Выйти
6. Последовательность входа
7. Результирующий результат JWT будет показывать результаты Типингдна

## <a name="live-version"></a>Динамическая версия

• MFA была отключена в этой тестовой версии, но вы видите, что `promptMFA` после проверки подлинности MFA будет выдаваться запрос на указание.

• Зарегистрируйтесь [здесь](https://b2cprod.b2clogin.com/b2cprod.onmicrosoft.com/oauth2/v2.0/authorize?p=B2C_1A_SU_TDNA&client_id=51d907f8-db14-4460-a1fd-27eaeb2a74da&nonce=defaultNonce&redirect_uri=https://jwt.ms/&scope=openid&response_type=id_token&prompt=login) и войдите [здесь](https://b2cprod.b2clogin.com/b2cprod.onmicrosoft.com/oauth2/v2.0/authorize?p=B2C_1A_SI_TDNA&client_id=51d907f8-db14-4460-a1fd-27eaeb2a74da&nonce=defaultNonce&redirect_uri=https://jwt.ms/&scope=openid&response_type=id_token&prompt=login)

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в следующих статьях:

- [Пользовательские политики в AAD B2C](./custom-policy-overview.md)

- [Приступая к работе с пользовательскими политиками в AAD B2C](./custom-policy-get-started.md?tabs=applications)