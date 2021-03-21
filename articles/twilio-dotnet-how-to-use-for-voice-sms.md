---
title: Как использовать Twilio для поддержки голосовых вызовов и SMS (.NET) | Документация Майкрософт
description: Узнайте, как осуществлять телефонные вызовы и отправку SMS-сообщений с помощью службы Twilio API в Azure. Примеры программного кода написаны в .NET.
services: ''
documentationcenter: .net
author: georgewallace
ms.assetid: 74d4f3c9-f1cb-4968-b744-36b32cd0e834
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 04/24/2015
ms.author: gwallace
ms.custom: devx-track-dotnet
ms.openlocfilehash: 104f969f5e27ef36ad43eb10e19176a4bcfd6648
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96003771"
---
# <a name="how-to-use-twilio-for-voice-and-sms-capabilities-from-azure"></a>Использование Twilio для поддержки голосовых вызовов и SMS в Azure
В этом руководстве показано, как выполнять типовые задачи программирования с помощью службы Twilio API в Azure. Здесь описываются такие сценарии, как телефонный звонок и отправка SMS-сообщения. Дополнительные сведения о Twilio и использовании голоса и SMS в приложениях см. в разделе [дальнейшие действия](#NextSteps) .

## <a name="what-is-twilio"></a><a id="WhatIs"></a>Что такое Twilio?
Twilio создает новые возможности для бизнес-коммуникаций, позволяя разработчикам встраивать в приложения функции голосовой связи, VoIP и обмена сообщениями. Они виртуализируют всю необходимую инфраструктуру в облачной глобальной среде с предоставлением доступа через коммуникационную API платформу Twilio. Приложения отличаются простотой создания и масштабирования. Оцените гибкость работы с оплатой по мере использования и преимущества надежности облака.

**Twilio Voice** позволяет приложениям осуществлять и принимать телефонные вызовы. **Twilio SMS** позволяет приложениям отправлять и принимать SMS-сообщения. **Twilio Client** позволяет выполнять VoIP звонки с любого телефона, планшета или из браузера, а также поддерживает WebRTC.

## <a name="twilio-pricing-and-special-offers"></a><a id="Pricing"></a>Цены и специальные предложения Twilio
Клиентам Azure доступно [специальное предложение](https://www.twilio.com/azure): кредит Twilio в размере 10 долл. США при обновлении учетной записи Twilio. Этот кредит Twilio применяется к любым сценариям использования Twilio (кредит в размере 10 $ позволяет отправить 1000 SMS-сообщений или получать входящие голосовые вызовы продолжительностью до 1000 минут в зависимости от расположения телефонного номера, а также от направления отправки сообщения или совершения звонка). Воспользуйтесь этим Twilioным кредитом и начните с [twilio.com/Azure](https://twilio.com/azure).

Twilio представляет собой службу с повременной оплатой. Стартовые платежи отсутствуют, а учетную запись можно закрыть в любое время. Дополнительные сведения см. в статье [Цены на Twilio](https://www.twilio.com/voice/pricing).

## <a name="concepts"></a><a id="Concepts"></a>Основные понятия
Twilio API — это интерфейс API RESTful, который предоставляет приложениям функции для работы с голосовыми вызовами и SMS. Полный список клиентских библиотек API Twilio, которые доступны на разных языках, см. на [этой странице][twilio_libraries].

Основные аспекты Twilio API: команды Twilio и язык разметки Twilio (TwiML).

### <a name="twilio-verbs"></a><a id="Verbs"></a>Команды Twilio
API использует команды Twilio; Например, команда **&lt; скажите &gt;** указывает Twilio воспроизвести доставить сообщение при вызове.

Ниже приведен список команд Twilio.  Дополнительные сведения о других командах и возможностях см. в [документации по языку разметки Twilio](https://www.twilio.com/docs/api/twiml).

* `<Dial>`: подключение вызывающего абонента к другому телефону.
* `<Gather>`: сбор цифр, введенных на клавиатуре телефона.
* `<Hangup>`: окончание вызова.
* `<Play>`: воспроизведение звукового файла.
* `<Pause>`: бесшумное ожидание в течение указанного времени (в секундах).
* `<Record>`: запись голоса вызывающего абонента и возвращение URL-адреса файла, содержащего запись.
* `<Redirect>`: передача управления для вызова или SMS в TwiML по другому URL-адресу.
* `<Reject>`: отклонение входящего вызова на ваш номер Twilio без выставления счета.
* `<Say>`: преобразование текста в речь при вызове.
* `<Sms>`: отправка SMS-сообщения.

### <a name="twiml"></a>TwiML
TwiML — это набор инструкций на основе XML и с использованием команд Twilio, которые сообщают службе Twilio, как необходимо обработать вызов или SMS.

Например, следующие инструкции TwiML преобразуют текст **Hello World** в речь.

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<Response>
  <Say>Hello World</Say>
</Response>
```

Когда приложение вызывает Twilio API, одним из параметров API является URL-адрес, который возвращает ответ TwiML. Для целей разработки можно использовать URL-адреса из Twilio для предоставления ответов TwiML, используемых приложениями. Также может разместить свои собственные URL-адреса для получения ответов TwiML; другой вариант — использовать объект **TwiMLResponse** .

Дополнительные сведения о командах Twilio, их атрибутах и TwiML см. на странице [TwiML][twiml]. Дополнительные сведения об API Twilio см. на [этой странице][twilio_api].

## <a name="create-a-twilio-account"></a><a id="CreateAccount"></a>Создание учетной записи Twilio
После подготовки к получению учетной записи Twilio зарегистрируйтесь на [странице получения пробной версии Twilio ][try_twilio]. Вы можете начать с бесплатной учетной записи и обновить ее позднее.

При регистрации учетной записи Twilio, вы получите идентификатор учетной записи и маркер проверки подлинности. Эти элементы необходимы для вызовов Twilio API. Чтобы предотвратить несанкционированный доступ к учетной записи, храните маркер проверки подлинности в безопасности. Идентификатор учетной записи и маркер аутентификации отображаются на [странице учетной записи Twilio][twilio_account] в полях **ACCOUNT SID** (Идентификатор безопасности учетной записи) и **AUTH TOKEN** (Маркер аутентификации) соответственно.

## <a name="create-an-azure-application"></a><a id="create_app"></a>Создание приложения Azure
Приложение Azure, на котором размещается приложение с поддержкой Twilio, ничем не отличается от любого другого приложения Azure. Добавьте библиотеку Twilio для .NETи настройте роль так, чтобы она могла использовать такие библиотеки.
Сведения о создании начального проекта Azure с помощью Visual Studio см. на [этой странице][vs_project].

## <a name="configure-your-application-to-use-twilio-libraries"></a><a id="configure_app"></a>Настройка приложения для использования библиотек Twilio
Twilio предоставляет набор вспомогательных библиотек .NET, содержащих различные аспекты Twilio для предоставления простых и легких способов взаимодействия с Twilio REST API и Twilio Client для создания ответов TwiML.

Twilio предоставляет пять библиотек для разработчиков .NET:

| Библиотека | Описание |
| --- | --- |
| Twilio.API | Основная библиотека Twilio, реализующая интерфейс API REST Twilio в виде понятной библиотеки .NET. Эта библиотека доступна для .NET, Silverlight и Windows Phone 7. |
| Twilio.TwiML | Позволяет создавать разметку TwiML удобным для .NET способом. |
| Twilio.MVC | Для разработчиков, использующих ASP.NET MVC, эта библиотека включает в себя TwilioController, TwiML ActionResult и атрибут проверки запроса. |
| Twilio.WebMatrix | Для разработчиков, использующих бесплатное средство разработки WebMatrix от Microsoft, эта библиотека содержит помощники синтаксиса Razor для выполнения различных действий Twilio. |
| Twilio.Client.Capability | Содержит генератор маркера "Возможность" для использования с Twilio Client JavaScript SDK. |

> [!Important]
> Для всех библиотек требуется .NET 3.5, Silverlight 4 и Windows Phone 7 или более поздней версии.

В примерах, приведенных в этом руководстве, используется библиотека Twilio.API.

Библиотеки можно [установить с помощью расширения диспетчера пакетов NuGet](https://www.twilio.com/docs/csharp/install), доступного для всех версий, начиная с Visual Studio 2010 и заканчивая Visual Studio 2015.  Исходный код размещен на веб-сайте [GitHub][twilio_github_repo]. Там же есть ссылка на вики-сайт с полной документацией по использованию этих библиотек.

По умолчанию Microsoft Visual Studio 2010 устанавливает NuGet версии 1.2. Для установки библиотек Twilio требуется версия NuGet 1.6 или выше. Сведения об установке или обновлении NuGet см. в разделе [https://nuget.org/][nuget] .

> [!NOTE]
> Чтобы установить последнюю версию NuGet, сначала необходимо удалить загруженную версию, используя диспетчер расширений Visual Studio. Для этого необходимо запустить Visual Studio от имени администратора. В противном случае кнопка «Удалить» будет неактивна.
>
>

### <a name="to-add-the-twilio-libraries-to-your-visual-studio-project"></a><a id="use_nuget"></a>Чтобы добавить библиотеки Twilio в проект Visual Studio, выполните указанные ниже действия.
1. Откройте решение в Visual Studio.
2. Щелкните правой кнопкой мыши **Ссылки**.
3. Щелкните **Manage NuGet Packages...**
4. Щелкните **Online**(В сети).
5. В поле поиска online введите *twilio*.
6. Щелкните **Install** (Установить) на пакете Twilio.

## <a name="how-to-make-an-outgoing-call"></a><a id="howto_make_call"></a>Практическое руководство. Осуществление исходящего вызова
Далее показано, как осуществлять исходящий вызов с использованием класса **CallResource**. Этот код также использует сайт из Twilio для выдачи ответа на языке разметки Twilio (TwiML). Замените значения телефонных номеров **To** (Кому) и **From** (От) и проверьте номер телефона **From** (От) для учетной записи Twilio перед выполнением кода.

```csharp
// Use your account SID and authentication token instead
// of the placeholders shown here.
const string accountSID = "your_twilio_account";
const string authToken = "your_twilio_authentication_token";

// Initialize the TwilioClient.
TwilioClient.Init(accountSID, authToken);

// Use the Twilio-provided site for the TwiML response.
var url = "https://twimlets.com/message";
url = $"{url}?Message%5B0%5D=Hello%20World";

// Set the call From, To, and URL values to use for the call.
// This sample uses the sandbox number provided by
// Twilio to make the call.
var call = CallResource.Create(
    to: new PhoneNumber("+NNNNNNNNNN"),
    from: new PhoneNumber("NNNNNNNNNN"),
    url: new Uri(url));
    }
```

Дополнительные сведения о параметрах, передаваемых в метод **CallResource.Create**, см. в разделе [https://www.twilio.com/docs/api/rest/making-calls][twilio_rest_making_calls].

Как уже упоминалось, этот код также использует сайт из Twilio для выдачи ответа на языке TwiML. Вместо этого можно использовать собственный веб-сайт для предоставления ответа TwiML. Дополнительные сведения см. в разделе [Практическое руководство. Предоставление ответа TwiML с собственного веб-сайта](#howto_provide_twiml_responses).

## <a name="how-to-send-an-sms-message"></a><a id="howto_send_sms"></a>Практическое руководство. Отправка SMS-сообщения
На следующем снимке экрана показано, как отправить SMS-сообщение с использованием класса **MessageResource**. Номер **from** предоставляется Twilio для пробных учетных записей для отправки SMS-сообщений. Номер **To** (Кому) для учетной записи Twilio необходимо проверить перед выполнением кода.

```csharp
// Use your account SID and authentication token instead
// of the placeholders shown here.
const string accountSID = "your_twilio_account";
const string authToken = "your_twilio_authentication_token";

// Initialize the TwilioClient.
TwilioClient.Init(accountSID, authToken);

try
{
    // Send an SMS message.
    var message = MessageResource.Create(
        to: new PhoneNumber("+12069419717"),
        from: new PhoneNumber("+14155992671"),
        body: "This is my SMS message.");
}
catch (TwilioException ex)
{
    // An exception occurred making the REST call
    Console.WriteLine(ex.Message);
}
```

## <a name="how-to-provide-twiml-responses-from-your-own-website"></a><a id="howto_provide_twiml_responses"></a>Как предоставить TwiML ответы с собственного веб-сайта
Когда приложение инициирует вызов API Twilio (например, с использованием метода **CallResource.Create**), Twilio отправляет ваш запрос на URL-адрес, который должен вернуть ответ TwiML. В примере, показанном в разделе [Практическое руководство. Осуществление исходящего звонка](#howto_make_call), для возврата отклика используется URL-адрес [https://twimlets.com/message][twimlet_message_url], предоставляемый Twilio.

> [!NOTE]
> Хотя TwiML предназначается для использования веб-службами, TwiML можно также просмотреть в браузере. Например, щелкните, [https://twimlets.com/message][twimlet_message_url] чтобы просмотреть пустой `<Response>` элемент; в качестве другого примера щелкните, [https://twimlets.com/message?Message%5B0%5D=Hello%20World](https://twimlets.com/message?Message%5B0%5D=Hello%20World) чтобы просмотреть `<Response>` элемент, содержащий элемент « &lt; скажем» &gt; .
>

Вместо того чтобы использовать URL-адрес, предоставленный Twilio, можно создать собственный URL-адрес для возврата HTTP-ответов. Веб-сайт можно создавать на любом языке, который возвращает HTTP-ответы. В этом разделе предполагается, что URL-адрес будет размещаться из универсального обработчика ASP.NET.

Следующий обработчик ASP.NET создает ответ TwiML **Hello World** на вызов.

```csharp
using System.Text;
using System.Web;

namespace WebRole1
{
    /// <summary>
    /// Summary description for Handler1
    /// </summary>
    public class Handler1 : IHttpHandler
    {
        public void ProcessRequest(HttpContext context)
        {
            const string twiMLResponse =
                "<Response><Say>Hello World.</Say></Response>";

            context.Response.Clear();
            context.Response.ContentType = "text/xml";
            context.Response.ContentEncoding = Encoding.UTF8;
            context.Response.Write(twiMLResponse);
            context.Response.End();
        }

        public bool IsReusable
        {
            get
            {
                return false;
            }
        }
    }
}
```
    
Как видно из приведенного выше примера, ответ TwiML будет представлять собой простой XML-документ. Библиотека Twilio.TwiML содержит классы, которые создадут для вас TwiML. В приведенном ниже примере выдается такой же ответ, как и в предыдущем примере, однако с использованием класса **VoiceResponse**.

```csharp
using System.Web;
using Twilio.TwiML;

namespace WebRole1
{
    /// <summary>
    /// Summary description for Handler1
    /// </summary>
    public class Handler1 : IHttpHandler
    {

        public void ProcessRequest(HttpContext context)
        {
            var twiml = new VoiceResponse();
            twiml.Say("Hello World.");

            context.Response.Clear();
            context.Response.ContentType = "text/xml";
            context.Response.Write(twiml.ToString());
            context.Response.End();
        }

        public bool IsReusable
        {
            get
            {
                return false;
            }
        }
    }
}
```

Дополнительные сведения о TwiML см. в разделе [https://www.twilio.com/docs/api/twiml](https://www.twilio.com/docs/api/twiml) .

После настройки способа предоставления ответов TwiML можно передать этот URL-адрес в метод **CallResource.Create**. Например, если у вас есть веб-приложение MyTwiML, развернутое в облачной службе Azure, а имя обработчика ASP.NET — mytwiml.ashx, то URL-адрес может быть передан в **CallResource.Create**, как показано в следующем примере кода.

```csharp
// This sample uses the sandbox number provided by Twilio to make the call.
// Place the call.
var call = CallResource.Create(
    to: new PhoneNumber("+NNNNNNNNNN"),
    from: new PhoneNumber("NNNNNNNNNN"),
    url: new Uri("http://<your_hosted_service>.cloudapp.net/MyTwiML/mytwiml.ashx"));
    }
```

Дополнительные сведения об использовании Twilio в Azure с ASP.NET см. в статье [Осуществление телефонных звонков с использованием Twilio в веб-роли Azure][howto_phonecall_dotnet].

[!INCLUDE [twilio-additional-services-and-next-steps](../includes/twilio-additional-services-and-next-steps.md)]

[howto_phonecall_dotnet]: partner-twilio-cloud-services-dotnet-phone-call-web-role.md

[twimlet_message_url]: https://twimlets.com/message

[twilio_rest_making_calls]: https://www.twilio.com/docs/api/rest/making-calls

[vs_project]:https://docs.microsoft.com/visualstudio/azure/vs-azure-tools-azure-project-create
[nuget]:https://nuget.org/
[twilio_github_repo]:https://github.com/twilio/twilio-csharp

[twilio_libraries]: https://www.twilio.com/docs/libraries
[twiml]: https://www.twilio.com/docs/api/twiml
[twilio_api]: https://www.twilio.com/api
[try_twilio]: https://www.twilio.com/try-twilio
[twilio_account]:  https://www.twilio.com/console
[verify_phone]: https://www.twilio.com/console/phone-numbers/verified
