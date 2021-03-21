---
title: Мониторы в Устойчивые функции (Python) — Azure
description: Узнайте, как реализовать монитор состояния с помощью расширения Устойчивые функции для функций Azure (Python).
author: davidmrdavid
ms.topic: conceptual
ms.date: 12/02/2020
ms.author: azfuncdf
ms.openlocfilehash: 62b3c9bb1c6fd53d9f11227a9d7e774d56859d04
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102434769"
---
# <a name="monitor-scenario-in-durable-functions---github-issue-monitoring-sample"></a>Сценарий мониторинга в Устойчивые функции — пример отслеживания проблем в GitHub

Шаблон монитора представляет собой гибкий повторяющийся процесс в рабочем процессе. Например, повторение опроса, пока не будут выполнены определенные условия. В этой статье приведен пример использования устойчивых функций для реализации мониторинга.

## <a name="prerequisites"></a>Предварительные условия

* [Выполните инструкции из краткого руководства](quickstart-python-vscode.md)
* [Клонирование или скачивание проекта примеров из GitHub](https://github.com/Azure/azure-functions-durable-python/tree/main/samples/)


## <a name="scenario-overview"></a>Общие сведения о сценарии

Этот пример отслеживает количество проблем в репозитории GitHub и предупреждает пользователя о наличии более чем трех открытых проблем. Вы можете использовать обычную функцию, активируемую с помощью таймера, чтобы запрашивать число открытых проблем через регулярные интервалы. Однако одной из проблем, связанной с этим подходом, является **управление жизненным циклом**. Если необходимо отправить только одно оповещение, монитор должен отключиться после обнаружения трех или более проблем. Возможность завершить свое выполнение является одним из преимуществ шаблона мониторинга, среди которых также:

* Мониторы работают с интервалами, а не по расписанию: триггер таймера *запускается* каждый час, а монитор *ожидает* один час между действиями. Действия монитора не будут перекрываться, если это не указано, что может быть важно для длительных задач.
* Мониторы могут иметь динамические интервалы: время ожидания может меняться в зависимости от некоторого состояния.
* Мониторы могут завершить работу, когда какое-либо условие выполняется, или быть завершены другим процессом.
* Мониторы могут принимать параметры. В этом примере показано, как можно применить один и тот же процесс мониторинга репозитория к запрошенному общедоступному репозиторию GitHub и номеру телефона.
* Мониторы можно масштабировать. Та как каждый монитор является экземпляром оркестрации, можно создавать несколько мониторов без необходимости создавать новые функции или писать новый код.
* Мониторы легко интегрируются в более крупные рабочие процессы. Монитором может быть один раздел более сложной функции оркестрации или [суборкестрация](durable-functions-sub-orchestrations.md).

## <a name="configuration"></a>Конфигурация

### <a name="configuring-twilio-integration"></a>Настройка интеграции Twilio

[!INCLUDE [functions-twilio-integration](../../../includes/functions-twilio-integration.md)]

## <a name="the-functions"></a>Функции

В этой статье описаны следующие функции в примере приложения:

* `E3_Monitor`: [Функция Orchestrator](durable-functions-bindings.md#orchestration-trigger) , которая `E3_TooManyOpenIssues` периодически вызывает. Он вызывает `E3_SendAlert` , если возвращаемое значение `E3_TooManyOpenIssues` равно `True` .
* `E3_TooManyOpenIssues`: [Функция действия](durable-functions-bindings.md#activity-trigger) , которая проверяет, имеет ли репозиторий слишком много открытых проблем. В целях демонстрации мы рекомендуем использовать более трех открытых проблем, чтобы они были слишком большими.
* `E3_SendAlert`: функция действия, которая отправляет SMS-сообщение через Twilio.

### <a name="e3_monitor-orchestrator-function"></a>E3_Monitor Orchestrator, функция


Функция **E3_Monitor** использует стандартный файл *function.json* для функций оркестратора.

[!code-json[Main](~/samples-durable-functions-python/samples/monitor/E3_Monitor/function.json)]

Ниже приведен код, реализующий функцию.

[!code-python[Main](~/samples-durable-functions-python/samples/monitor/E3_Monitor/\_\_init\_\_.py)]


Эта функция оркестратора выполняет следующие задачи:

1. Возвращает *репозиторий* для отслеживания и *номер телефона* , на который будет отправлено SMS Notification.
2. Задает срок действия монитора. Для краткости в образце используется жестко заданное значение.
3. Вызывает **E3_TooManyOpenIssues** , чтобы определить, существует ли слишком много открытых проблем в запрошенном репозитории.
4. При наличии слишком большого числа проблем вызывает **E3_SendAlert** для отправки SMS Notification по запрошенному номеру телефона.
5. Создает устойчивый таймер для возобновления оркестрации во время следующего интервала опроса. Для краткости в образце используется жестко заданное значение.
6. Продолжение выполняется до тех пор, пока текущее время в формате UTC не пройдет время окончания срока действия монитора или не будет отправлено оповещение SMS.

Несколько экземпляров Orchestrator можно запустить одновременно, вызвав функцию Orchestrator несколько раз. Можно указать репозиторий для отслеживания и номер телефона для отправки оповещения SMS. Наконец, обратите внимание, что функция Orchestrator *не* выполняется во время ожидания таймера, поэтому вам не придется начислять за него оплату.


### <a name="e3_toomanyopenissues-activity-function"></a>Функция действия E3_TooManyOpenIssues

Вспомогательные функции действий, как и другие примеры, являются обычными функциями, которые используют привязку триггера `activityTrigger`. Функция **E3_TooManyOpenIssues** возвращает список открытых проблем в репозитории и определяет наличие "слишком большого количества" таких элементов: более 3, как в нашем примере.


Файл *function.json* выглядит следующим образом:

[!code-json[Main](~/samples-durable-functions-python/samples/monitor/E3_TooManyOpenIssues/function.json)]

Ниже приведена сама реализация.

[!code-python[Main](~/samples-durable-functions-python/samples/monitor/E3_TooManyOpenIssues/\_\_init\_\_.py)]


### <a name="e3_sendalert-activity-function"></a>Функция действия E3_SendAlert

Функция **E3_SendAlert** использует привязку Twilio для отправки SMS-сообщения, уведомляющее пользователя о наличии по крайней мере 3 открытых проблем, ожидающих разрешения.


Файл *function.json* выглядит просто:

[!code-json[Main](~/samples-durable-functions-python/samples/monitor/E3_TooManyOpenIssues/function.json)]

Ниже приведен код, который отправляет SMS-сообщение:

[!code-python[Main](~/samples-durable-functions-python/samples/monitor/E3_SendAlert/\_\_init\_\_.py)]


## <a name="run-the-sample"></a>Запуск примера

Вам потребуется учетная запись [GitHub](https://github.com/) . Создайте временный общедоступный репозиторий, в который можно открыть проблемы.

С помощью функций, активируемых по HTTP (перечисленных в примере), можно запустить оркестрацию, отправив следующий запрос HTTP POST:

```
POST https://{host}/orchestrators/E3_Monitor
Content-Length: 77
Content-Type: application/json

{ "repo": "<your github handle>/<a new github repo under your user>", "phone": "+1425XXXXXXX" }
```

Например, если имя пользователя GitHub — `foo` , а репозиторий —, то `bar` значение `"repo"` должно быть `"foo/bar"` .

```
HTTP/1.1 202 Accepted
Content-Type: application/json; charset=utf-8
Location: https://{host}/runtime/webhooks/durabletask/instances/f6893f25acf64df2ab53a35c09d52635?taskHub=SampleHubVS&connection=Storage&code={SystemKey}
RetryAfter: 10

{"id": "f6893f25acf64df2ab53a35c09d52635", "statusQueryGetUri": "https://{host}/runtime/webhooks/durabletask/instances/f6893f25acf64df2ab53a35c09d52635?taskHub=SampleHubVS&connection=Storage&code={systemKey}", "sendEventPostUri": "https://{host}/runtime/webhooks/durabletask/instances/f6893f25acf64df2ab53a35c09d52635/raiseEvent/{eventName}?taskHub=SampleHubVS&connection=Storage&code={systemKey}", "terminatePostUri": "https://{host}/runtime/webhooks/durabletask/instances/f6893f25acf64df2ab53a35c09d52635/terminate?reason={text}&taskHub=SampleHubVS&connection=Storage&code={systemKey}"}
```

Экземпляр **E3_Monitor** запускается и запрашивает предоставленный репозиторий на наличие открытых проблем. Если обнаружено по крайней мере три открытых проблемы, вызывается функция действия для отправки предупреждения. в противном случае устанавливается таймер. Когда время таймера истечет, оркестровка возобновится.

Результат действия оркестрации можно просмотреть в журналах функций на портале Функций Azure.

```
[2020-12-04T18:24:30.007Z] Executing 'Functions.HttpStart' (Reason='This function was programmatically 
called via the host APIs.', Id=93772f6b-f4f0-405a-9d7b-be9eb7a38aa6)
[2020-12-04T18:24:30.769Z] Executing 'Functions.E3_Monitor' (Reason='(null)', Id=058e656e-bcb1-418c-95b3-49afcd07bd08)
[2020-12-04T18:24:30.847Z] Started orchestration with ID = '788420bb31754c50acbbc46e12ef4f9c'.
[2020-12-04T18:24:30.932Z] Executed 'Functions.E3_Monitor' (Succeeded, Id=058e656e-bcb1-418c-95b3-49afcd07bd08, Duration=174ms)
[2020-12-04T18:24:30.955Z] Executed 'Functions.HttpStart' (Succeeded, Id=93772f6b-f4f0-405a-9d7b-be9eb7a38aa6, Duration=1028ms)
[2020-12-04T18:24:31.229Z] Executing 'Functions.E3_TooManyOpenIssues' (Reason='(null)', Id=6fd5be5e-7f26-4b0b-98df-c3ac39125da3)
[2020-12-04T18:24:31.772Z] Executed 'Functions.E3_TooManyOpenIssues' (Succeeded, Id=6fd5be5e-7f26-4b0b-98df-c3ac39125da3, Duration=555ms)
[2020-12-04T18:24:40.754Z] Executing 'Functions.E3_Monitor' (Reason='(null)', Id=23915e4c-ddbf-46f9-b3f0-53289ed66082)
[2020-12-04T18:24:40.789Z] Executed 'Functions.E3_Monitor' (Succeeded, Id=23915e4c-ddbf-46f9-b3f0-53289ed66082, Duration=38ms)
(...trimmed...)
```

Согласование будет завершено по истечении времени ожидания или обнаружению более 3 открытых проблем. Можно также использовать `terminate` API внутри другой функции или вызвать веб-перехватчик HTTP Post **терминатепостури** , указанный выше в ответе 202 выше. Чтобы использовать веб-перехватчик, замените на `{text}` причину раннего завершения. URL-адрес HTTP POST будет выглядеть примерно следующим образом:

```
POST https://{host}/runtime/webhooks/durabletask/instances/f6893f25acf64df2ab53a35c09d52635/terminate?reason=Because&taskHub=SampleHubVS&connection=Storage&code={systemKey}
```

## <a name="next-steps"></a>Дальнейшие действия

В этом примере показано, как использовать устойчивые функции для контроля состояния внешнего источника с помощью [устойчивых таймеров](durable-functions-timers.md) и условной логики. В приведенном ниже примере показано, как использовать внешние события и [устойчивые таймеры](durable-functions-timers.md) для обработки действий человека.

> [!div class="nextstepaction"]
> Выполните [пример взаимодействия с пользователем](durable-functions-phone-verification.md)
