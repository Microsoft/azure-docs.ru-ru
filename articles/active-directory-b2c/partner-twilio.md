---
title: Twilio проверить приложение с помощью Azure Active Directory B2C
titleSuffix: Azure AD B2C
description: Узнайте, как интегрировать пример Интернет-приложения для оплаты в Azure AD B2C с помощью API проверки Twilio. Соблюдайте требования к транзакциям PSD2 (директива служб оплаты 2) с помощью динамической компоновки и надежной проверки подлинности клиента.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 06/08/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 953653a758577ed3d48ca2d81403b4cb363ea294
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95994049"
---
# <a name="integrating-twilio-verify-app-with-azure-active-directory-b2c"></a>Интеграция Twilio проверка приложения с Azure Active Directory B2C

В этом пошаговом руководстве описано, как интегрировать пример Интернет-приложения оплаты в Azure Active Directory B2C (Azure AD B2C) с помощью API проверки Twilio. С помощью Twilio проверить приложение, Azure AD B2C клиенты могут соответствовать требованиям транзакций PSD2 (директивы оплаты 2) с помощью динамической компоновки и надежной проверки подлинности клиента.

## <a name="prerequisites"></a>Предварительные условия

Чтобы приступить к работе, вам потребуется:

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* [Клиент Azure AD B2C](tutorial-create-tenant.md) , связанный с вашей подпиской Azure.
* [Пробная учетная запись](https://www.twilio.com/try-twilio) по адресу Twilio.

## <a name="scenario-description"></a>Описание сценария

Решение Twilio состоит из следующих компонентов:

- [Демонстрационное веб-приложение .NET PSD2](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Twilio-VerifyAPI/source-code/PSD2%20Demo%20App), которое предоставляет возможность входа или регистрации и выполнения фиктивной транзакции с высоким риском.
- Azure AD B2C объединенную [политику входа и регистрации](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Twilio-VerifyAPI/policy).
- Azure AD B2Cная политика, интегрированная с API Twilio для проверки с помощью `id_token_hint` .
- Веб-приложение .NET, которое размещает `.well-known` конечную точку OpenIdConnect, чтобы разрешить проверку `id_token_hint` .


    ![поток twilio](media/partner-twilio/twilio-flow.png)

| Шаг | Описание |
|------|------|
| 1     | Пользователь инициирует вход или регистрацию в демонстрационном приложении PSD2. Пользователь проходит проверку подлинности с помощью Azure AD B2C объединенную политику входа и регистрации. В приложение возвращается маркер. При регистрации номер телефона пользователя проверяется с помощью SMS/Phone и записывается в учетную запись Azure AD B2C.     |
| 2     | Пользователь инициирует транзакцию с высоким риском, например перемещение $50,00. Текущий маркер доступа пользователя оценивается для PolicyId, чтобы определить, прошел ли пользователь проверку подлинности с помощью Step-Up пользовательской политики.     |
| 3     | Приложение записывает значения транзакции и получателя платежа, $50,00 и Джон Петров и создает подписанный маркер. Этот маркер называется `id_token_hint` и содержит утверждение `amount:$500, payee:john doe` . `id_token_hint`Отправляется вместе с запросом к Azure AD B2C пользовательской политике, которая интегрирована с Twilio.     |
| 4     | Azure AD B2C проверяет подпись id_token_hint, проверяя `/.well-known` конечную точку подключения приложений OpenID Connect. После проверки он извлекает утверждения из этого маркера, в особенности `amount` и `payee` . Пользователь увидит страницу для проверки номера мобильного телефона через SMS-сообщение.     |
| 5     | Пользователь запрашивает проверку номера телефона через SMS-сообщение, а Azure AD B2C выполняет запрос REST API в Twilio проверку конечной точки API. Он также отправляет транзакцию `amount` и `payee` как часть процесса PSD2 для создания одноразового секретного кода (OTP). Twilio отправляет сообщение SMS на зарегистрированный номер телефона пользователя.     |
| 6     |  Пользователь вводит OTP, полученный в сообщении SMS, и отправляет его в Azure AD B2C. Azure AD B2C выполняет запрос API с помощью этого OTP, чтобы проверить, правильно ли используется OTP-API. Наконец, в приложение выдается маркер с новым PolicyId, обозначающим, что пользователь выполнил пошаговую проверку подлинности.    |
|      |      |

## <a name="onboard-with-twilio"></a>Подключение с помощью Twilio

1. Получите [пробную учетную запись](https://www.twilio.com/try-twilio) по адресу Twilio.

2. Приобретите номер телефона на сайте Twilio, как описано в [этой статье](https://support.twilio.com/hc/articles/223135247-How-to-Search-for-and-Buy-a-Twilio-Phone-Number-from-Console) .

3. Перейдите к разделу [Проверка API](https://www.twilio.com/console/verify/services) в консоли Twilio и следуйте [инструкциям](https://www.twilio.com/docs/verify/verifying-transactions-psd2) по созданию службы и включению параметра PSD2.  

## <a name="configure-the-psd2-demo-app"></a>Настройка демонстрационного приложения PSD2

1. Откройте решение B2C-WebAPI-DotNet и замените следующие значения собственными значениями для конкретного клиента в web.config:

    ```xml
   <add key="ida:Tenant" value="yourtenant.onmicrosoft.com" />
   <add key="ida:TenantId" value="d6f33888-0000-4c1f-9b50-1590f171fc70" />
   <add key="ida:ClientId" value="6bd98cc8-0000-446a-a05e-b5716ef2651b" />
   <add key="ida:ClientSecret" value="secret" />
   <add key="ida:AadInstance" value="https://yourtenant.b2clogin.com/tfp/{0}/{1}" />
   <add key="ida:RedirectUri" value="https://your hosted psd2 demo app url/" />
   ```

2. В [веб-приложении](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Twilio-VerifyAPI/source-code/PSD2%20Demo%20App) также размещается генератор указания маркера идентификации и конечная точка метаданных.
   - Создайте сертификат для подписи, как описано в описании этого [образца](https://github.com/azure-ad-b2c/samples/tree/master/policies/invite#creating-a-signing-certificate).
   - Обновите следующие строки на основе сертификата в web.config:
   
   ```xml
   <add key="ida:SigningCertThumbprint" value="4F39D6014818082CBB763E5BA5F230E545212E89" />
   <add key="ida:SigningCertAlgorithm" value="RS256" />
   ```

3. Отправьте демонстрационное приложение на выбранный поставщик услуг размещения. В [этом примере](https://github.com/azure-ad-b2c/samples/tree/master/policies/invite#hosting-the-application-in-azure-app-service)представлено руководство по службе приложений Azure, включая инструкции по отправке сертификата.

4. Обновите регистрацию приложения Azure AD B2C, добавив URL-адрес ответа, эквивалентный URL-адресу, по которому размещено приложение.

5. Откройте [файлы политики](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Twilio-VerifyAPI/policy) и замените все экземпляры  `contoso` именем своего клиента.

6. Найдите Twilio REST API технический профиль **Custom-SMS-регистрация**. Обновите  `ServiceURL`   с помощью Twilio AccountSID и номера отсчета на приобретенный номер телефона.

7. Найдите Twilio REST API технические профили, **твилиорестапи-Verify-шаг 1**   и **Твилиорестапи-Verify-шаг 2** и обновите  `ServiceURL`   с помощью Twilio AccountSID.

## <a name="integrate-with-azure-ad-b2c"></a>Интеграция с Azure AD B2C

Добавьте файлы политики в Azure AD B2C:

1. Войдите на [портал Azure](https://portal.azure.com/) с правами глобального администратора клиента Azure AD B2C.

2. Убедитесь, что используете каталог с клиентом Azure AD B2C, выбрав фильтр **Каталог и подписка** в меню вверху и каталог с вашим клиентом.

3. Выберите **Все службы** в левом верхнем углу окна портала Azure, найдите службу **Azure AD B2C** и выберите ее.

4. Перейдите в раздел **Azure AD B2C**  >  **Идентификация**  >  .**ключи политики** инфраструктуры.

5. Добавьте новый раздел с именем **B2cRestTwilioClientId**. Выберите **вручную** и укажите значение параметра Twilio AccountSID.

6. Добавьте новый раздел с именем **B2cRestTwilioClientSecret**. Выберите **вручную** и укажите значение маркера проверки подлинности Twilio.

7. Отправьте все файлы политики в клиент.

8. Настройте строку в преобразовании утверждений Женератеотпмессажеенрол для своего текста SMS-регистрации.

## <a name="test-the-solution"></a>Тестирование решения

* Перейдите к приложению и протестируйте действия входа, регистрации и отправки денег.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в следующих статьях:

- См. [примеры кода интеграции](https://github.com/azure-ad-b2c/samples/tree/master/policies/twilio-mfa-psd2) GitHub для Twilio  

- [Пользовательские политики в AAD B2C](custom-policy-overview.md)

- [Приступая к работе с пользовательскими политиками в AAD B2C](custom-policy-get-started.md?tabs=applications)
