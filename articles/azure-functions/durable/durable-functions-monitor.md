---
title: Мониторы в устойчивых функциях — Azure
description: Сведения о том, как реализовать монитор состояния с использованием расширения устойчивых функций для Функций Azure.
ms.topic: conceptual
ms.date: 12/07/2018
ms.author: azfuncdf
ms.openlocfilehash: 8ef32ecfb6f69b71d29578d3b8314f568fd9386a
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102431080"
---
# <a name="monitor-scenario-in-durable-functions---weather-watcher-sample"></a>Сценарий монитора в устойчивых функциях — пример наблюдателя за погодой

Шаблон монитора представляет собой гибкий *повторяющийся* процесс в рабочем процессе. Например, повторение опроса, пока не будут выполнены определенные условия. В этой статье приведен пример использования [устойчивых функций](durable-functions-overview.md) для реализации мониторинга.

## <a name="prerequisites"></a>Предварительные требования

# <a name="c"></a>[C#](#tab/csharp)

* [Выполните инструкции из краткого руководства](durable-functions-create-first-csharp.md)
* [Клонирование или скачивание проекта примеров из GitHub](https://github.com/Azure/azure-functions-durable-extension/tree/main/samples/precompiled)

# <a name="javascript"></a>[JavaScript](#tab/javascript)

* [Выполните инструкции из краткого руководства](quickstart-js-vscode.md)
* [Клонирование или скачивание проекта примеров из GitHub](https://github.com/Azure/azure-functions-durable-extension/tree/main/samples/javascript)

---

## <a name="scenario-overview"></a>Общие сведения о сценарии

Этот пример отслеживает текущие погодные условия местоположения и с помощью SMS оповещает пользователя, когда небо ясное. Для проверки погоды и отправки оповещений можно использовать обычную функцию, активируемую по таймеру. Однако одной из проблем, связанной с этим подходом, является **управление жизненным циклом**. Если необходимо отправить только одно оповещение, монитор должен отключиться после обнаружения ясной погоды. Возможность завершить свое выполнение является одним из преимуществ шаблона мониторинга, среди которых также:

* Мониторы работают с интервалами, а не по расписанию: триггер таймера *запускается* каждый час, а монитор *ожидает* один час между действиями. Действия монитора не будут перекрываться, если это не указано, что может быть важно для длительных задач.
* Мониторы могут иметь динамические интервалы: время ожидания может меняться в зависимости от некоторого состояния.
* Мониторы могут завершить работу, когда какое-либо условие выполняется, или быть завершены другим процессом.
* Мониторы могут принимать параметры. Пример показывает, как один и тот же процесс мониторинга погоды может применяться к любому запрашиваемому местоположению и номеру телефона.
* Мониторы можно масштабировать. Та как каждый монитор является экземпляром оркестрации, можно создавать несколько мониторов без необходимости создавать новые функции или писать новый код.
* Мониторы легко интегрируются в более крупные рабочие процессы. Монитором может быть один раздел более сложной функции оркестрации или [суборкестрация](durable-functions-sub-orchestrations.md).

## <a name="configuration"></a>Конфигурация

### <a name="configuring-twilio-integration"></a>Настройка интеграции Twilio

[!INCLUDE [functions-twilio-integration](../../../includes/functions-twilio-integration.md)]

### <a name="configuring-weather-underground-integration"></a>Настройка интеграции Weather Underground

Этот пример включает в себя использование API Weather Underground для проверки текущих погодных условий для местоположения.

Для начала потребуется учетная запись Weather Underground. Вы можете создать его бесплатно в [https://www.wunderground.com/signup](https://www.wunderground.com/signup) . После этого нужно будет приобрести ключ API. Это можно сделать [https://www.wunderground.com/weather/api](https://www.wunderground.com/weather/api/?MR=1) , посетив, а затем выбрав параметры ключа. Бесплатного плана Stratus Developer достаточно для запуска этого примера.

Получив ключ API, добавьте следующие **параметры приложения** в приложение-функцию.

| Имя параметра приложения | Описание значения |
| - | - |
| **WeatherUndergroundApiKey**  | Ваш ключ API Weather Underground. |

## <a name="the-functions"></a>Функции

В этой статье описаны следующие функции в примере приложения:

* `E3_Monitor`: [Функция Orchestrator](durable-functions-bindings.md#orchestration-trigger) , которая `E3_GetIsClear` периодически вызывает. Она вызывает `E3_SendGoodWeatherAlert`, если `E3_GetIsClear` возвращает значение true.
* `E3_GetIsClear`: [Функция действия](durable-functions-bindings.md#activity-trigger) , которая проверяет текущие условия погоды для расположения.
* `E3_SendGoodWeatherAlert`: функция действия, которая отправляет SMS-сообщение через Twilio.

### <a name="e3_monitor-orchestrator-function"></a>E3_Monitor Orchestrator, функция

# <a name="c"></a>[C#](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/Monitor.cs?range=41-78,97-115)]

Orchestrator требует расположения для отслеживания и номера телефона для отправки сообщения, когда сообщение будет очищаться в месте. Эти данные передаются в Orchestrator как строго типизированный `MonitorRequest` объект.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Функция **E3_Monitor** использует стандартный файл *function.json* для функций оркестратора.

[!code-json[Main](~/samples-durable-functions/samples/javascript/E3_Monitor/function.json)]

Ниже приведен код, реализующий функцию.

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E3_Monitor/index.js)]

---

Эта функция оркестратора выполняет следующие задачи:

1. Получает **MonitorRequest**, состоящий из названия *местоположения* для мониторинга и *номера телефона*, на который будет отправлено SMS-уведомление.
2. Задает срок действия монитора. Для краткости в образце используется жестко заданное значение.
3. Вызывает **E3_GetIsClear**, чтобы определить, ясное ли небо в запрашиваемом местоположении.
4. Если погода хорошая, вызывается **E3_SendGoodWeatherAlert**, чтобы отправить SMS-уведомление на запрошенный номер телефона.
5. Создает устойчивый таймер для возобновления оркестрации во время следующего интервала опроса. Для краткости в образце используется жестко заданное значение.
6. Продолжение выполняется до тех пор, пока текущее время в формате UTC не пройдет время окончания срока действия монитора или не будет отправлено оповещение SMS.

Несколько экземпляров Orchestrator можно запустить одновременно, вызвав функцию Orchestrator несколько раз. Можно указать местоположение для мониторинга и номер телефона для отправки SMS-оповещения. Наконец, обратите внимание, что функция Orchestrator *не* выполняется во время ожидания таймера, поэтому вам не придется начислять за него оплату.
### <a name="e3_getisclear-activity-function"></a>Функция действия E3_GetIsClear

Вспомогательные функции действий, как и другие примеры, являются обычными функциями, которые используют привязку триггера `activityTrigger`. Функция **E3_GetIsClear** получает текущие погодные условия с использованием API Weather Underground и определяет, чистое ли небо.

# <a name="c"></a>[C#](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/Monitor.cs?range=80-85)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Файл *function.json* выглядит следующим образом:

[!code-json[Main](~/samples-durable-functions/samples/javascript/E3_GetIsClear/function.json)]

Ниже приведена сама реализация.

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E3_GetIsClear/index.js)]

---

### <a name="e3_sendgoodweatheralert-activity-function"></a>Функция действия E3_SendGoodWeatherAlert

Функция **E3_SendGoodWeatherAlert** использует привязку Twilio для отправки SMS-сообщения, уведомляющего пользователя, что это подходящее время для прогулки.

# <a name="c"></a>[C#](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/Monitor.cs?range=87-96,140-205)]

> [!NOTE]
> Для `Microsoft.Azure.WebJobs.Extensions.Twilio` запуска примера кода необходимо установить пакет NuGet.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Файл *function.json* выглядит просто:

[!code-json[Main](~/samples-durable-functions/samples/javascript/E3_SendGoodWeatherAlert/function.json)]

Ниже приведен код, который отправляет SMS-сообщение:

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E3_SendGoodWeatherAlert/index.js)]

---

## <a name="run-the-sample"></a>Запуск примера

С помощью функций, активируемых по HTTP (перечисленных в примере), можно запустить оркестрацию, отправив следующий запрос HTTP POST:

```
POST https://{host}/orchestrators/E3_Monitor
Content-Length: 77
Content-Type: application/json

{ "location": { "city": "Redmond", "state": "WA" }, "phone": "+1425XXXXXXX" }
```

```
HTTP/1.1 202 Accepted
Content-Type: application/json; charset=utf-8
Location: https://{host}/runtime/webhooks/durabletask/instances/f6893f25acf64df2ab53a35c09d52635?taskHub=SampleHubVS&connection=Storage&code={SystemKey}
RetryAfter: 10

{"id": "f6893f25acf64df2ab53a35c09d52635", "statusQueryGetUri": "https://{host}/runtime/webhooks/durabletask/instances/f6893f25acf64df2ab53a35c09d52635?taskHub=SampleHubVS&connection=Storage&code={systemKey}", "sendEventPostUri": "https://{host}/runtime/webhooks/durabletask/instances/f6893f25acf64df2ab53a35c09d52635/raiseEvent/{eventName}?taskHub=SampleHubVS&connection=Storage&code={systemKey}", "terminatePostUri": "https://{host}/runtime/webhooks/durabletask/instances/f6893f25acf64df2ab53a35c09d52635/terminate?reason={text}&taskHub=SampleHubVS&connection=Storage&code={systemKey}"}
```

Экземпляр **E3_Monitor** запускается и запрашивает текущие погодные условия для запрашиваемого местоположения. Если погода хорошая, он вызывает функцию действия для отправки оповещения. В противном случае он устанавливает таймер. Когда время таймера истечет, оркестровка возобновится.

Результат действия оркестрации можно просмотреть в журналах функций на портале Функций Azure.

```
2018-03-01T01:14:41.649 Function started (Id=2d5fcadf-275b-4226-a174-f9f943c90cd1)
2018-03-01T01:14:42.741 Started orchestration with ID = '1608200bb2ce4b7face5fc3b8e674f2e'.
2018-03-01T01:14:42.780 Function completed (Success, Id=2d5fcadf-275b-4226-a174-f9f943c90cd1, Duration=1111ms)
2018-03-01T01:14:52.765 Function started (Id=b1b7eb4a-96d3-4f11-a0ff-893e08dd4cfb)
2018-03-01T01:14:52.890 Received monitor request. Location: Redmond, WA. Phone: +1425XXXXXXX.
2018-03-01T01:14:52.895 Instantiating monitor for Redmond, WA. Expires: 3/1/2018 7:14:52 AM.
2018-03-01T01:14:52.909 Checking current weather conditions for Redmond, WA at 3/1/2018 1:14:52 AM.
2018-03-01T01:14:52.954 Function completed (Success, Id=b1b7eb4a-96d3-4f11-a0ff-893e08dd4cfb, Duration=189ms)
2018-03-01T01:14:53.226 Function started (Id=80a4cb26-c4be-46ba-85c8-ea0c6d07d859)
2018-03-01T01:14:53.808 Function completed (Success, Id=80a4cb26-c4be-46ba-85c8-ea0c6d07d859, Duration=582ms)
2018-03-01T01:14:53.967 Function started (Id=561d0c78-ee6e-46cb-b6db-39ef639c9a2c)
2018-03-01T01:14:53.996 Next check for Redmond, WA at 3/1/2018 1:44:53 AM.
2018-03-01T01:14:54.030 Function completed (Success, Id=561d0c78-ee6e-46cb-b6db-39ef639c9a2c, Duration=62ms)
```

Согласование завершается по истечении времени ожидания или при обнаружении Clear небо. Можно также использовать `terminate` API внутри другой функции или вызвать веб-перехватчик HTTP Post **терминатепостури** , указанный выше в ответе 202 выше. Чтобы использовать веб-перехватчик, замените на `{text}` причину раннего завершения. URL-адрес HTTP POST будет выглядеть примерно следующим образом:

```
POST https://{host}/runtime/webhooks/durabletask/instances/f6893f25acf64df2ab53a35c09d52635/terminate?reason=Because&taskHub=SampleHubVS&connection=Storage&code={systemKey}
```

## <a name="next-steps"></a>Дальнейшие действия

В этом примере показано, как использовать устойчивые функции для контроля состояния внешнего источника с помощью [устойчивых таймеров](durable-functions-timers.md) и условной логики. В приведенном ниже примере показано, как использовать внешние события и [устойчивые таймеры](durable-functions-timers.md) для обработки действий человека.

> [!div class="nextstepaction"]
> Выполните [пример взаимодействия с пользователем](durable-functions-phone-verification.md)