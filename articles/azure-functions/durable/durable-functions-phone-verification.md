---
title: Участие пользователя и время ожидания в устойчивых функциях — Azure
description: Сведения о том, как обрабатывать участие пользователей и время ожидания в расширении устойчивых функций для Функций Azure.
ms.topic: conceptual
ms.date: 12/07/2018
ms.author: azfuncdf
ms.openlocfilehash: dd7f8416b2f4520ec8e94c8608f753f7412afc4d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99627378"
---
# <a name="human-interaction-in-durable-functions---phone-verification-sample"></a>Участие пользователя в устойчивых функциях. Пример проверки номера телефона

В этом примере показано, как создать оркестрацию [устойчивых функций](durable-functions-overview.md), которая включает в себя участие пользователя. Каждый раз, когда человек участвует в автоматизированном процессе, процесс должен отправлять уведомления пользователю и получать ответы асинхронно. Он также должен допускать возможность, что пользователь недоступен. (Именно в этой части время ожидания становится важным.)

В этом примере реализуется система проверки номера телефона на основе SMS. Эти типы потоков часто используются при проверке номера телефона клиента или для многофакторной проверки подлинности. Это мощный пример, поскольку вся реализация выполняется с помощью нескольких небольших функций. Внешнее хранилище данных, например база данных, не требуется.

[!INCLUDE [durable-functions-prerequisites](../../../includes/durable-functions-prerequisites.md)]

## <a name="scenario-overview"></a>Общие сведения о сценарии

Проверка номера телефона используется для подтверждения того, что пользователи вашего приложения не рассылают нежелательную почту и являются теми, за кого себя выдают. Многофакторная проверка подлинности часто применяется для защиты учетных записей пользователей от злоумышленников. Проблема реализации собственной проверки номера телефона состоит в том, что требуется взаимодействие с пользователем с **отслеживанием состояния**. Пользователю обычно предоставляется определенный код (например, 4-значное число), на который он должен отреагировать **в течение приемлемого промежутка времени**.

Обычно в решении "Функции Azure" не отслеживается состояние (как и в других облачных конечных точках на других платформах), поэтому эти типы взаимодействия требуют явного управления состоянием во внешней базе данных или другом постоянном хранилище. Кроме того, взаимодействие должно быть разбито на несколько функций, которые можно согласовать вместе. Например, необходима хотя бы одна функция для выбора кода, его сохранения и отправки на телефон пользователя. Кроме того, вам также требуется другая функция, чтобы получить ответ от пользователя и каким-то образом обратно сопоставить его с исходным вызовом функции, чтобы выполнить проверку кода. Время ожидания также является важным аспектом обеспечения безопасности. Это может оказаться довольно сложным.

Сложность этого сценария значительно снижается, когда вы используете устойчивые функции. Как будет показано в этом примере, с помощью функции оркестратора можно без проблем управлять взаимодействием с отслеживанием состояния без использования любых внешних хранилищ данных. Так как функции оркестратора *устойчивые*, эти интерактивные потоки также очень надежные.

## <a name="configuring-twilio-integration"></a>Настройка интеграции Twilio

[!INCLUDE [functions-twilio-integration](../../../includes/functions-twilio-integration.md)]

## <a name="the-functions"></a>Функции

В этой статье описаны следующие функции, используемые в примере приложения:

* `E4_SmsPhoneVerification`: [Функция Orchestrator](durable-functions-bindings.md#orchestration-trigger) , которая выполняет проверку телефона, включая управление временем ожидания и повторные попытки.
* `E4_SendSmsChallenge`: [Функция действия](durable-functions-bindings.md#activity-trigger) , которая отправляет код через текстовое сообщение.

> [!NOTE]
> `HttpStart`Функция в [примере приложения и краткое руководство](#prerequisites) выступает в качестве [клиента оркестрации](durable-functions-bindings.md#orchestration-client) , запускающего функцию Orchestrator.

### <a name="e4_smsphoneverification-orchestrator-function"></a>E4_SmsPhoneVerification Orchestrator, функция

# <a name="c"></a>[C#](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/PhoneVerification.cs?range=17-70)]

> [!NOTE]
> На первый случай он может быть не очевидным, но эта Orchestrator не нарушает [детерминированное ограничение оркестрации](durable-functions-code-constraints.md). Он детерминирован, так как `CurrentUtcDateTime` свойство используется для вычисления времени окончания срока действия таймера и возвращает одно и то же значение при каждом воспроизведении на этом этапе в коде Orchestrator. Такое поведение важно для того, чтобы обеспечить одинаковый `winner` результат каждого повторяющегося вызова `Task.WhenAny` .

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Функция **E4_SmsPhoneVerification** использует стандартный файл *function.json* для функций оркестратора.

[!code-json[Main](~/samples-durable-functions/samples/javascript/E4_SmsPhoneVerification/function.json)]

Ниже приведен код, реализующий функцию.

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E4_SmsPhoneVerification/index.js)]

> [!NOTE]
> На первый случай он может быть не очевидным, но эта Orchestrator не нарушает [детерминированное ограничение оркестрации](durable-functions-code-constraints.md). Он детерминирован, так как `currentUtcDateTime` свойство используется для вычисления времени окончания срока действия таймера и возвращает одно и то же значение при каждом воспроизведении на этом этапе в коде Orchestrator. Такое поведение важно для того, чтобы обеспечить одинаковый `winner` результат каждого повторяющегося вызова `context.df.Task.any` .

# <a name="python"></a>[Python](#tab/python)

Функция **E4_SmsPhoneVerification** использует стандартный файл *function.json* для функций оркестратора.

[!code-json[Main](~/samples-durable-functions-python/samples/human_interaction/E4_SmsPhoneVerification/function.json)]

Ниже приведен код, реализующий функцию.

[!code-python[Main](~/samples-durable-functions-python/samples/human_interaction/E4_SmsPhoneVerification/\_\_init\_\_.py)]

> [!NOTE]
> На первый случай он может быть не очевидным, но эта Orchestrator не нарушает [детерминированное ограничение оркестрации](durable-functions-code-constraints.md). Он детерминирован, так как `currentUtcDateTime` свойство используется для вычисления времени окончания срока действия таймера и возвращает одно и то же значение при каждом воспроизведении на этом этапе в коде Orchestrator. Такое поведение важно для того, чтобы обеспечить одинаковый `winner` результат каждого повторяющегося вызова `context.df.Task.any` .

---

После запуска функция оркестратора выполняет следующие задачи:

1. Получает номер телефона, на который она будет *отправлять* SMS-уведомление.
2. Вызывает **E4_SendSmsChallenge** для отправки SMS-сообщения пользователю и возвращает обратно ожидаемый 4-значный код запроса.
3. Создает устойчивый таймер, который отсчитывает 90 секунд от текущего времени.
4. Пока выполняется таймер, ожидает событие **SmsChallengeResponse** от пользователя.

Пользователи получают SMS-сообщение с 4-значным кодом. У них 90 секунд для отправки этого же кода из четырех цифр обратно в экземпляр функции Orchestrator для завершения процесса проверки. Если пользователи отправляют неверный код, у них есть еще три попытки (в пределах тех же 90 секунд).

> [!WARNING]
> Если таймеры вам больше не нужны, как показано в примере выше, когда принимается ответ на запрос, [отключите их](durable-functions-timers.md).

## <a name="e4_sendsmschallenge-activity-function"></a>Функция действия E4_SendSmsChallenge

Функция **E4_SendSmsChallenge** использует привязку Twilio для отправки SMS-сообщения с кодом из четырех цифр конечному пользователю.

# <a name="c"></a>[C#](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/PhoneVerification.cs?range=72-89)]

> [!NOTE]
> Для `Microsoft.Azure.WebJobs.Extensions.Twilio` запуска примера кода необходимо установить пакет NuGet.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Файл *function.json* выглядит следующим образом:

[!code-json[Main](~/samples-durable-functions/samples/javascript/E4_SendSmsChallenge/function.json)]

Ниже приведен код, создающий код запроса из четырех цифр и отправляющий SMS-сообщение:

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E4_SendSmsChallenge/index.js)]

# <a name="python"></a>[Python](#tab/python)

Файл *function.json* выглядит следующим образом:

[!code-json[Main](~/samples-durable-functions-python/samples/human_interaction/SendSMSChallenge/function.json)]

Ниже приведен код, создающий код запроса из четырех цифр и отправляющий SMS-сообщение:

[!code-python[Main](~/samples-durable-functions-python/samples/human_interaction/SendSMSChallenge/\_\_init\_\_.py)]

---

## <a name="run-the-sample"></a>Запуск примера

С помощью функций, активируемых по HTTP (перечисленных в примере), можно запустить оркестрацию, отправив следующий запрос HTTP POST:

```
POST http://{host}/orchestrators/E4_SmsPhoneVerification
Content-Length: 14
Content-Type: application/json

"+1425XXXXXXX"
```

```
HTTP/1.1 202 Accepted
Content-Length: 695
Content-Type: application/json; charset=utf-8
Location: http://{host}/runtime/webhooks/durabletask/instances/741c65651d4c40cea29acdd5bb47baf1?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}

{"id":"741c65651d4c40cea29acdd5bb47baf1","statusQueryGetUri":"http://{host}/runtime/webhooks/durabletask/instances/741c65651d4c40cea29acdd5bb47baf1?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}","sendEventPostUri":"http://{host}/runtime/webhooks/durabletask/instances/741c65651d4c40cea29acdd5bb47baf1/raiseEvent/{eventName}?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}","terminatePostUri":"http://{host}/runtime/webhooks/durabletask/instances/741c65651d4c40cea29acdd5bb47baf1/terminate?reason={text}&taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}"}
```

Функция оркестратора получает указанный номер телефона и немедленно отправляет на него SMS-сообщение со случайно сгенерированным 4-значным кодом проверки, например *2168*. Функция ожидает ответ в течение 90 секунд.

Чтобы ответить на код, можно использовать [ `RaiseEventAsync` (.NET) или `raiseEvent` (JavaScript)](durable-functions-instance-management.md) в другой функции или вызвать веб-перехватчик HTTP Post **sendEventUrl** , указанный выше в ответе 202 выше, заменив `{eventName}` именем события `SmsChallengeResponse` :

```
POST http://{host}/runtime/webhooks/durabletask/instances/741c65651d4c40cea29acdd5bb47baf1/raiseEvent/SmsChallengeResponse?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}
Content-Length: 4
Content-Type: application/json

2168
```

Если вы отправите это до истечения времени ожидания, оркестрация будет завершена, а для поля `output` будет задано значение `true`, указывающее, что проверка прошла успешно.

```
GET http://{host}/runtime/webhooks/durabletask/instances/741c65651d4c40cea29acdd5bb47baf1?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}
```

```
HTTP/1.1 200 OK
Content-Length: 144
Content-Type: application/json; charset=utf-8

{"runtimeStatus":"Completed","input":"+1425XXXXXXX","output":true,"createdTime":"2017-06-29T19:10:49Z","lastUpdatedTime":"2017-06-29T19:12:23Z"}
```

Если время ожидания истекло или вы ввели неправильный код 4 раза, можно запросить состояние и просмотреть выходные данные функции оркестрации `false`, где указано, что проверка номера телефона завершилась сбоем.

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 145

{"runtimeStatus":"Completed","input":"+1425XXXXXXX","output":false,"createdTime":"2017-06-29T19:20:49Z","lastUpdatedTime":"2017-06-29T19:22:23Z"}
```

## <a name="next-steps"></a>Дальнейшие действия

В этом примере демонстрируются некоторые расширенные возможности Устойчивые функции, особенно `WaitForExternalEvent` и `CreateTimer` API. Вы узнали, как их можно использовать совместно с `Task.WaitAny`, чтобы реализовать надежную систему времени ожидания, которая часто полезна для взаимодействия с реальными людьми. Дополнительные сведения об использовании устойчивых функций см. в серии статей, где подробно рассматриваются определенные темы.

> [!div class="nextstepaction"]
> [Перейти к первой статье из серии](durable-functions-bindings.md)
