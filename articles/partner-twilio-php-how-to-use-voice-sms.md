---
title: Использование Twilio для поддержки голосовых вызовов и SMS (PHP) | Документация Майкрософт
description: Узнайте, как осуществлять телефонные вызовы и отправку SMS-сообщений с помощью службы Twilio API в Azure. Примеры кода написаны на PHP.
documentationcenter: php
services: ''
author: georgewallace
ms.assetid: 007f22e3-ac75-4868-8315-da000c2e0dd0
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: PHP
ms.topic: article
ms.date: 11/25/2014
ms.author: gwallace
ms.openlocfilehash: bf1ab01b39d594002bc5e677ffe6c3049fbb91ce
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95521025"
---
# <a name="how-to-use-twilio-for-voice-and-sms-capabilities-in-php"></a>Использование Twilio для поддержки голосовых возможностей и SMS в PHP
В этом руководстве показано, как выполнять типовые задачи программирования с помощью службы Twilio API в Azure. Здесь описываются такие сценарии, как телефонный звонок и отправка SMS-сообщения. Дополнительные сведения о Twilio и использовании голосовых функций и SMS в приложениях см. в разделе [Дальнейшие действия](#NextSteps).

## <a name="what-is-twilio"></a><a id="WhatIs"></a>Что такое Twilio?
Twilio создает новые возможности для бизнес-коммуникаций, позволяя разработчикам встраивать в приложения функции голосовой связи, VoIP и обмена сообщениями. Они виртуализируют всю необходимую инфраструктуру в облачной глобальной среде с предоставлением доступа через коммуникационную API платформу Twilio. Приложения отличаются простотой создания и масштабирования. Оцените гибкость повременной оплаты, а также преимущества, связанные с надежностью облачных решений.

**Twilio Voice** позволяет приложениям осуществлять и принимать телефонные вызовы. **Twilio SMS** позволяет приложениям отправлять и принимать текстовые сообщения. **Twilio Client** позволяет выполнять VoIP звонки с любого телефона, планшета или из браузера, а также поддерживает WebRTC.

## <a name="twilio-pricing-and-special-offers"></a><a id="Pricing"></a>Цены и специальные предложения Twilio
Клиентам Azure доступно [специальное предложение](https://www.twilio.com/azure): кредит Twilio в размере 10 долл. США при обновлении учетной записи Twilio. Этот кредит Twilio применяется к любым сценариям использования Twilio (кредит в размере 10 $ позволяет отправить 1000 SMS-сообщений или получать входящие голосовые вызовы продолжительностью до 1000 минут в зависимости от расположения телефонного номера, а также от направления отправки сообщения или совершения звонка). Воспользуйтесь этим Twilioным кредитом и начните работу по адресу: [https://ahoy.twilio.com/azure](https://ahoy.twilio.com/azure) .

Twilio представляет собой службу с повременной оплатой. Стартовые платежи отсутствуют, а учетную запись можно закрыть в любое время. Дополнительные сведения см. в статье [Цены на Twilio][twilio_pricing].

## <a name="concepts"></a><a id="Concepts"></a>Основные понятия
Twilio API — это интерфейс API RESTful, который предоставляет приложениям функции для работы с голосовыми вызовами и SMS. Полный список клиентских библиотек API Twilio, которые доступны на разных языках, см. на [этой странице][twilio_libraries].

Основные аспекты Twilio API: команды Twilio и язык разметки Twilio (TwiML).

### <a name="twilio-verbs"></a><a id="Verbs"></a>Команды Twilio
API использует команды Twilio; Например, команда **&lt; скажите &gt;** указывает Twilio воспроизвести доставить сообщение при вызове.

Ниже приведен список команд Twilio. Дополнительные сведения о других командах и возможностях см. в [документации по языку разметки Twilio](https://www.twilio.com/docs/api/twiml).

* **&lt; Набрать номер &gt;**: подключает вызывающий объект к другому телефону.
* **&lt; Сбор &gt; данных**: собирает цифры, указанные на клавиатуре телефона.
* **&lt; Разрыв &gt; соединения**: завершает вызов.
* **&lt; Play &gt;**: воспроизводит звуковой файл.
* **&lt; Пауза &gt;**: ожидание автоматического ожидания в течение указанного числа секунд.
* **&lt; Запись &gt;**: записывает голосовое значение вызывающего объекта и возвращает URL-адрес файла, содержащего запись.
* **&lt; Redirect &gt;**: передает управление вызовом или SMS в TwiML по другому URL-адресу.
* **&lt; Отклонить &gt;**: отклоняет входящий звонок на номер Twilio без выставления счетов
* **&lt; Скажем &gt;**: преобразование текста в речь, выполняемое при вызове.
* **&lt; SMS &gt;**: отправляет сообщение SMS.

### <a name="twiml"></a><a id="TwiML"></a>TwiML
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

## <a name="create-a-php-application"></a><a id="create_app"></a>Создание приложения PHP
Приложение PHP, которое использует службу Twilio и выполняется в Azure, не отличается от других приложений PHP, которые используют службу Twilio. Хотя службы Twilio поддерживают интерфейс REST и могут вызываться из PHP несколькими способами, в этой статье основное внимание уделяется использованию служб Twilio с [библиотекой Twilio для PHP из GitHub][twilio_php]. Дополнительные сведения об использовании библиотеки Twilio для PHP см. в разделе [https://www.twilio.com/docs/libraries/php][twilio_lib_docs] .

Подробные инструкции для создания и развертывания приложения Twilio/PHP в Azure можно найти на [этой странице][howto_phonecall_php].

## <a name="configure-your-application-to-use-twilio-libraries"></a><a id="configure_app"></a>Настройка приложения для использования библиотек Twilio
Настройку приложения для работы с библиотекой Twilio для PHP можно выполнить двумя способами:

1. Скачайте библиотеку Twilio для PHP из GitHub ( [https://github.com/twilio/twilio-php][twilio_php] ) и добавьте в приложение каталог **служб** .
   
    -или-
2. Установите библиотеку Twilio для PHP в виде пакета PEAR. Для установки можно использовать следующие команды:

    ```bash
    $ pear channel-discover twilio.github.com/pear
    $ pear install twilio/Services_Twilio
    ```

После установки библиотеки Twilio для PHP вы можете добавлять в начало PHP-файлов инструкцию **require_once**, задающую ссылку на эту библиотеку:

```php
require_once 'Services/Twilio.php';
```

Дополнительные сведения см. на веб-сайте [https://github.com/twilio/twilio-php/blob/master/README.md][twilio_github_readme].

## <a name="how-to-make-an-outgoing-call"></a><a id="howto_make_call"></a>Практическое руководство. Осуществление исходящего вызова
Ниже показано, как осуществлять исходящий вызов с использованием класса **Services_Twilio**. Этот код также использует сайт из Twilio для выдачи ответа на языке разметки Twilio (TwiML). Замените значения телефонных номеров **From** (От) и **To** (Кому) и проверьте номер телефона **From** (От) для учетной записи Twilio до выполнения кода.

```php
// Include the Twilio PHP library.
require_once 'Services/Twilio.php';

// Library version.
$version = "2010-04-01";

// Set your account ID and authentication token.
$sid = "your_twilio_account_sid";
$token = "your_twilio_authentication_token";

// The number of the phone initiating the call.
$from_number = "NNNNNNNNNNN";

// The number of the phone receiving call.
$to_number = "NNNNNNNNNNN";

// Use the Twilio-provided site for the TwiML response.
$url = "https://twimlets.com/message";

// The phone message text.
$message = "Hello world.";

// Create the call client.
$client = new Services_Twilio($sid, $token, $version);

//Make the call.
try
{
    $call = $client->account->calls->create(
        $from_number, 
        $to_number,
        $url.'?Message='.urlencode($message)
    );
}
catch (Exception $e) 
{
    echo 'Error: ' . $e->getMessage();
}
```

Как уже упоминалось, этот код также использует сайт из Twilio для выдачи ответа на языке TwiML. Можно использовать собственный веб-сайт для предоставления ответа TwiML; дополнительные сведения содержатся в разделе [Предоставление ответов TwiML с собственного веб-сайта](#howto_provide_twiml_responses).

* **Примечание**. сведения об устранении ошибок проверки сертификата TLS/SSL см. в разделе [https://www.twilio.com/docs/api/errors][ssl_validation] 

## <a name="how-to-send-an-sms-message"></a><a id="howto_send_sms"></a>Практическое руководство. Отправка SMS-сообщения
Ниже показано, как отправить SMS-сообщение с использованием класса **Services_Twilio**. С целью отправки SMS-сообщений для пробных учетных записей номер **From** (От) предоставляется Twilio. Номер **To** (Кому) для учетной записи Twilio необходимо проверить перед выполнением кода.

```php
// Include the Twilio PHP library.
require_once 'Services/Twilio.php';

// Library version.
$version = "2010-04-01";

// Set your account ID and authentication token.
$sid = "your_twilio_account_sid";
$token = "your_twilio_authentication_token";


$from_number = "NNNNNNNNNNN"; // With trial account, texts can only be sent from your Twilio number.
$to_number = "NNNNNNNNNNN";
$message = "Hello world.";

// Create the call client.
$client = new Services_Twilio($sid, $token, $version);

// Send the SMS message.
try
{
    $client->$client->account->messages->sendMessage($from_number, $to_number, $message);
}
catch (Exception $e) 
{
    echo 'Error: ' . $e->getMessage();
}
```

## <a name="how-to-provide-twiml-responses-from-your-own-website"></a><a id="howto_provide_twiml_responses"></a>Практическое руководство. Предоставление ответов TwiML с собственного веб-сайта
Когда приложение инициирует вызов API Twilio, Twilio отправляет ваш запрос на URL-адрес, который должен возвратить ответ TwiML. В приведенном выше примере используется URL-адрес, предоставленный Twilio [https://twimlets.com/message][twimlet_message_url] . (Хотя TwiML предназначается для использования службой Twilio, его можно также просмотреть в браузере. Например, щелкните, [https://twimlets.com/message][twimlet_message_url] чтобы просмотреть пустой `<Response>` элемент; в качестве другого примера щелкните, [https://twimlets.com/message?Message%5B0%5D=Hello%20World][twimlet_message_url_hello_world] чтобы просмотреть `<Response>` элемент, содержащий `<Say>` элемент.)

Вместо того чтобы использовать URL-адрес, предоставленный Twilio, можно создать собственный сайт для возврата HTTP-ответов. Веб-сайт можно создавать на любом языке, который возвращает XML-ответы; в этом разделе предполагается, что для создания TwiML будет использоваться язык PHP.

Следующая страница PHP создает ответ TwiML **Hello World** на вызов.

```xml
<?php    
    header("content-type: text/xml");    
    echo "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n";
?>
<Response>    
    <Say>Hello world.</Say>
</Response>
```

Как видно из приведенного выше примера, ответ TwiML будет представлять собой простой XML-документ. Библиотека Twilio для PHP содержит классы, которые создадут для вас TwiML. В следующем примере возвращается аналогичный предыдущему ответ, однако используется класс **Services\_Twilio\_Twiml** из библиотеки Twilio для PHP:

```php
require_once('Services/Twilio.php');

$response = new Services_Twilio_Twiml();
$response->say("Hello world.");
print $response;
```

Дополнительные сведения о TwiML см. в разделе [https://www.twilio.com/docs/api/twiml][twiml_reference] . 

После настройки PHP-страницы для предоставления ответов TwiML используйте URL-адрес PHP-страницы как URL-адрес, передаваемый в метод `Services_Twilio->account->calls->create`. Например, если у вас есть веб-приложение с именем **MyTwiML**, развернутое в размещенной службе Azure, а PHP-страница использует имя **mytwiml.php**, URL-адрес можно передать в класс **Services_Twilio->account->calls->create**, как показано в следующем примере:

```php
require_once 'Services/Twilio.php';

$sid = "your_twilio_account_sid";
$token = "your_twilio_authentication_token";
$from_number = "NNNNNNNNNNN";
$to_number = "NNNNNNNNNNN";
$url = "http://<your_hosted_service>.cloudapp.net/MyTwiML/mytwiml.php";

// The phone message text.
$message = "Hello world.";

$client = new Services_Twilio($sid, $token, "2010-04-01");

try
{
    $call = $client->account->calls->create(
        $from_number, 
        $to_number,
        $url.'?Message='.urlencode($message)
    );
}
catch (Exception $e) 
{
    echo 'Error: ' . $e->getMessage();
}
```

Дополнительные сведения об использовании Twilio в Azure с PHP см. в разделе [Как в Azure выполнить телефонный звонок с помощью Twilio в PHP-приложении][howto_phonecall_php].

## <a name="how-to-use-additional-twilio-services"></a><a id="AdditionalServices"></a>Руководство: использование дополнительных служб Twilio
В дополнение к приведенным примерам, Twilio предлагает веб-интерфейсы API для использования дополнительных функций Twilio в вашем приложении Azure. Дополнительные сведения см. в [документации по интерфейсу API Twilio][twilio_api_documentation].

## <a name="next-steps"></a><a id="NextSteps"></a>Дальнейшие действия
Вы узнали основные сведения о службе Twilio. Для получения дополнительных сведений используйте следующие ссылки.

* [Рекомендации по безопасности Twilio][twilio_security_guidelines]
* [Практические руководства по Twilio и примеры кода][twilio_howtos]
* [Краткие учебники по Twilio][twilio_quickstarts] 
* [Twilio на GitHub][twilio_on_github]
* [Обращение в службу поддержки Twilio][twilio_support]

[twilio_php]: https://github.com/twilio/twilio-php
[twilio_lib_docs]: https://www.twilio.com/docs/libraries/php
[twilio_github_readme]: https://github.com/twilio/twilio-php/blob/master/README.md
[ssl_validation]: https://www.twilio.com/docs/api/errors
[twilio_api_service]: https://api.twilio.com
[howto_phonecall_php]: partner-twilio-php-make-phone-call.md
[twilio_voice_request]: https://www.twilio.com/docs/api/twiml/twilio_request
[twilio_sms_request]: https://www.twilio.com/docs/api/twiml/sms/twilio_request
[misc_role_config_settings]: /previous-versions/azure/hh690945(v=azure.100)
[twimlet_message_url]: https://twimlets.com/message
[twimlet_message_url_hello_world]: https://twimlets.com/message?Message%5B0%5D=Hello%20World
[twiml_reference]: https://www.twilio.com/docs/api/twiml
[twilio_pricing]: https://www.twilio.com/pricing
[special_offer]: https://ahoy.twilio.com/azure
[twilio_libraries]: https://www.twilio.com/docs/libraries
[twiml]: https://www.twilio.com/docs/api/twiml
[twilio_api]: https://www.twilio.com/api
[try_twilio]: https://www.twilio.com/try-twilio
[twilio_account]:  https://www.twilio.com/user/account
[verify_phone]: https://www.twilio.com/user/account/phone-numbers/verified#
[twilio_api_documentation]: https://www.twilio.com/api
[twilio_security_guidelines]: https://www.twilio.com/docs/security
[twilio_howtos]: https://www.twilio.com/docs/howto
[twilio_on_github]: https://github.com/twilio
[twilio_support]: https://www.twilio.com/help/contact
[twilio_quickstarts]: https://www.twilio.com/docs/quickstart