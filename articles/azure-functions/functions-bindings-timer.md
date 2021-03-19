---
title: Триггеры таймера для службы "Функции Azure"
description: Узнайте, как использовать триггеры с таймерами в функциях Azure.
author: craigshoemaker
ms.assetid: d2f013d1-f458-42ae-baf8-1810138118ac
ms.topic: reference
ms.date: 11/18/2020
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: f99f9d240a0a9220d3b7f57cddd0a4f8ba6b6101
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104595997"
---
# <a name="timer-trigger-for-azure-functions"></a>Триггеры таймера для службы "Функции Azure"

В этой статье описывается, как работать с триггерами таймера в службе "Функции Azure". Триггер таймера позволяет выполнять функцию по расписанию.

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

Сведения о том, как вручную запустить функцию, активируемую с помощью таймера, см. [в разделе ручное выполнение функции, не активируемой HTTP](./functions-manually-run-non-http.md).

## <a name="packages---functions-2x-and-higher"></a>Packages — функции 2. x и более поздних версий

Триггер таймера доступен в пакете NuGet [Microsoft.Azure.WebJobs.Extensions](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions) версии 3.х. Исходный код для пакета находится в репозитории GitHub [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions/Extensions/Timers/).

[!INCLUDE [functions-package-auto](../../includes/functions-package-auto.md)]

## <a name="packages---functions-1x"></a>Пакеты – Функции 1.x

Триггер таймера доступен в пакете NuGet [Microsoft.Azure.WebJobs.Extensions](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions) версии 2.х. Исходный код для пакета находится в репозитории GitHub [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions/Extensions/Timers/).

[!INCLUDE [functions-package-auto](../../includes/functions-package-auto.md)]

## <a name="example"></a>Пример

# <a name="c"></a>[C#](#tab/csharp)

В следующем примере показана [функция C#](functions-dotnet-class-library.md) , которая выполняется каждый раз, когда в минутах имеется значение, кратное пяти (например, если функция начинается в 18:57:00, следующая производительность будет составлять 19:00:00). [`TimerInfo`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions/Extensions/Timers/TimerInfo.cs)Объект передается в функцию.

```cs
[FunctionName("TimerTriggerCSharp")]
public static void Run([TimerTrigger("0 */5 * * * *")]TimerInfo myTimer, ILogger log)
{
    if (myTimer.IsPastDue)
    {
        log.LogInformation("Timer is running late!");
    }
    log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
}
```

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В следующем примере показаны привязка триггера таймера в файле *function.json* и [функция сценария C#](functions-reference-csharp.md), которая использует эту привязку. Эта функция выполняет запись в журнал, указывая, когда ее вызов выполняется из-за пропущенного запуска по расписанию. [`TimerInfo`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions/Extensions/Timers/TimerInfo.cs)Объект передается в функцию.

Данные привязки в файле *function.json*:

```json
{
    "schedule": "0 */5 * * * *",
    "name": "myTimer",
    "type": "timerTrigger",
    "direction": "in"
}
```

Ниже приведен код скрипта C#.

```csharp
public static void Run(TimerInfo myTimer, ILogger log)
{
    if (myTimer.IsPastDue)
    {
        log.LogInformation("Timer is running late!");
    }
    log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}" );  
}
```

# <a name="java"></a>[Java](#tab/java)

Следующий пример функции активируется и выполняется каждые пять минут. Аннотация `@TimerTrigger` для функции определяет расписание, используя тот же формат строки, что и [выражения CRON](https://en.wikipedia.org/wiki/Cron#CRON_expression).

```java
@FunctionName("keepAlive")
public void keepAlive(
  @TimerTrigger(name = "keepAliveTrigger", schedule = "0 */5 * * * *") String timerInfo,
      ExecutionContext context
 ) {
     // timeInfo is a JSON string, you can deserialize it to an object using your favorite JSON library
     context.getLogger().info("Timer is triggered: " + timerInfo);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В следующем примере показаны привязка триггера таймера в файле *function.json* и [функция JavaScript](functions-reference-node.md), которая использует эту привязку. Эта функция выполняет запись в журнал, указывая, когда ее вызов выполняется из-за пропущенного запуска по расписанию. [Объект Timer](#usage) передается в функцию.

Данные привязки в файле *function.json*:

```json
{
    "schedule": "0 */5 * * * *",
    "name": "myTimer",
    "type": "timerTrigger",
    "direction": "in"
}
```

Ниже показан код JavaScript.

```JavaScript
module.exports = function (context, myTimer) {
    var timeStamp = new Date().toISOString();

    if (myTimer.IsPastDue)
    {
        context.log('Node is running late!');
    }
    context.log('Node timer trigger function ran!', timeStamp);   

    context.done();
};
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В следующем примере показано, как настроить *function.js* для и *run.ps1* файла для триггера таймера в [PowerShell](./functions-reference-powershell.md).

```json
{
  "bindings": [
    {
      "name": "Timer",
      "type": "timerTrigger",
      "direction": "in",
      "schedule": "0 */5 * * * *"
    }
  ]
}
```

```powershell
# Input bindings are passed in via param block.
param($Timer)

# Get the current universal time in the default string format.
$currentUTCtime = (Get-Date).ToUniversalTime()

# The 'IsPastDue' property is 'true' when the current function invocation is later than scheduled.
if ($Timer.IsPastDue) {
    Write-Host "PowerShell timer is running late!"
}

# Write an information log with the current time.
Write-Host "PowerShell timer trigger function ran! TIME: $currentUTCtime"
```

Экземпляр [объекта Timer](#usage) передается в качестве первого аргумента функции.

# <a name="python"></a>[Python](#tab/python)

В следующем примере используется привязка триггера таймера, конфигурация которой описана в *function.js* файле. Фактическая [функция Python](functions-reference-python.md) , использующая привязку, описана в файле *__init__. корректировки* . Объект, переданный в функцию, имеет тип [Azure. functions. тимеррекуест](/python/api/azure-functions/azure.functions.timerrequest). Логика функции записывает в журналы, указывающие, вызван ли текущий вызов из-за пропущенного события расписания.

Данные привязки в файле *function.json*:

```json
{
    "name": "mytimer",
    "type": "timerTrigger",
    "direction": "in",
    "schedule": "0 */5 * * * *"
}
```

Ниже приведен код Python.

```python
import datetime
import logging

import azure.functions as func


def main(mytimer: func.TimerRequest) -> None:
    utc_timestamp = datetime.datetime.utcnow().replace(
        tzinfo=datetime.timezone.utc).isoformat()

    if mytimer.past_due:
        logging.info('The timer is past due!')

    logging.info('Python timer trigger function ran at %s', utc_timestamp)
```

---

## <a name="attributes-and-annotations"></a>Атрибуты и заметки

# <a name="c"></a>[C#](#tab/csharp)

В [библиотеках классов C#](functions-dotnet-class-library.md) используйте [TimerTriggerAttribute](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions/Extensions/Timers/TimerTriggerAttribute.cs).

Конструктор атрибута принимает выражение CRON или `TimeSpan`. Можно использовать, `TimeSpan` только если приложение-функция выполняется в плане службы приложений. `TimeSpan` не поддерживается для функций использования или эластичных баз данных Premium.

В приведенном ниже примере показано выражение CRON.

```csharp
[FunctionName("TimerTriggerCSharp")]
public static void Run([TimerTrigger("0 */5 * * * *")]TimerInfo myTimer, ILogger log)
{
    if (myTimer.IsPastDue)
    {
        log.LogInformation("Timer is running late!");
    }
    log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
}
```

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В скрипте C# атрибуты не поддерживаются.

# <a name="java"></a>[Java](#tab/java)

Аннотация `@TimerTrigger` для функции определяет расписание, используя тот же формат строки, что и [выражения CRON](https://en.wikipedia.org/wiki/Cron#CRON_expression).

```java
@FunctionName("keepAlive")
public void keepAlive(
  @TimerTrigger(name = "keepAliveTrigger", schedule = "0 */5 * * * *") String timerInfo,
      ExecutionContext context
 ) {
     // timeInfo is a JSON string, you can deserialize it to an object using your favorite JSON library
     context.getLogger().info("Timer is triggered: " + timerInfo);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В JavaScript атрибуты не поддерживаются.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В PowerShell не поддерживаются атрибуты.

# <a name="python"></a>[Python](#tab/python)

В Python атрибуты не поддерживаются.

---

## <a name="configuration"></a>Конфигурация

В следующей таблице описываются свойства конфигурации привязки, которые задаются в файле *function.json* и атрибуте `TimerTrigger`.

|свойство function.json | Свойство атрибута |Описание|
|---------|---------|----------------------|
|**type** | Недоступно | Этому свойству необходимо присвоить значение "timerTrigger". Это свойство задается автоматически при создании триггера на портале Azure.|
|**direction** | Недоступно | Для этого свойства необходимо задать значение "in". Это свойство задается автоматически при создании триггера на портале Azure. |
|**name** | Недоступно | Имя переменной, представляющей объект таймера в коде функции. | 
|**Расписание**|**ScheduleExpression**|Значение [выражения CRON](#ncrontab-expressions) или [TimeSpan](#timespan). `TimeSpan` можно использовать только для приложения-функции, которая работает в плане службы приложений. Можно разместить выражение расписания в параметре приложения и задать для этого свойства имя параметра приложения, заключенное в **%** знаки, как показано в следующем примере: "% счедулеаппсеттинг%". |
|**runOnStartup**|**рунонстартуп**|Если настроено значение `true`, функция вызывается при запуске среды выполнения. Например, среда выполнения запускается, когда приложение-функция выходит из спящего режима (в который она перешла из-за отсутствия активности), При перезапуске приложения-функции в связи с изменением функции и при масштабировании приложения-функции. Поэтому **рунонстартуп** следует использовать редко, если всегда имеет значение `true` , особенно в рабочей среде. |
|**усемонитор**|**UseMonitor**|Установите значение `true` или`false`, чтобы указать, следует ли отслеживать расписание. При мониторинге расписания его экземпляры сохраняются, чтобы обеспечить его корректную обработку даже при перезапуске экземпляров приложения-функции. Если значение не задано явным образом, по умолчанию используется `true` Расписание с интервалом повторений больше или равным 1 минуте. Для расписаний, выполняющихся чаще одного раза в минуту, значением по умолчанию является `false`.

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

> [!CAUTION]
> Мы не рекомендуем задавать параметр **runOnStartup** для `true` в рабочей среде. Использование этого параметра заставляет код выполняться в очень непредсказуемое время. В определенных рабочих ситуациях эти дополнительные выполнения могут привести значительно более высоким затратам для приложений, размещенных в планах потребления. Например, при включенном параметре **рунонстартуп** триггер вызывается при каждом масштабировании приложения-функции. Убедитесь, что вы полностью понимаете рабочее поведение своих функций перед включением **runOnStartup** в рабочей среде.

## <a name="usage"></a>Использование

При вызове функции триггера таймера в функцию передается объект Timer. Далее представлен JSON в качестве примера объекта таймера.

```json
{
    "schedule":{
    },
    "scheduleStatus": {
        "last":"2016-10-04T10:15:00+00:00",
        "lastUpdated":"2016-10-04T10:16:00+00:00",
        "next":"2016-10-04T10:20:00+00:00"
    },
    "isPastDue":false
}
```

Значение свойства `isPastDue` — `true`, когда текущая функция вызывается позже запланированного. Например перезапуск приложения-функции может привести к тому, что вызов будет пропущен.

## <a name="ncrontab-expressions"></a>Выражения НКРОНТАБ

Функции Azure используют библиотеку [нкронтаб](https://github.com/atifaziz/NCrontab) для интерпретации выражений нкронтаб. Выражение НКРОНТАБ аналогично выражению CRON, за исключением того, что оно включает в начало в начале использования для точности времени в секундах дополнительное шестое поле.

`{second} {minute} {hour} {day} {month} {day-of-week}`

Каждое поле может принимать значение одного из следующих типов:

|Тип  |Пример  |Когда активируется  |
|---------|---------|---------|
|Определенное значение |<nobr>`0 5 * * * *`</nobr>| Один раз в час дня в минуту 5 каждого часа |
|Все значения (`*`)|<nobr>`0 * 5 * * *`</nobr>| Каждую минуту в час, начиная с часа 5 |
|Диапазон (оператор `-`)|<nobr>`5-7 * * * * *`</nobr>| Три раза в минуту с 5 по 7 в каждую минуту каждого дня |
|Набор значений (оператор `,`)|<nobr>`5,8,10 * * * * *`</nobr>| Три раза в минуту в секунду 5, 8 и 10 в каждую минуту каждого дня |
|Значение интервала (оператор `/`)|<nobr>`0 */5 * * * *`</nobr>| 12 раз в час в секунду 0 каждой 5-минутной недели каждый день; |

[!INCLUDE [functions-cron-expressions-months-days](../../includes/functions-cron-expressions-months-days.md)]

### <a name="ncrontab-examples"></a>Примеры НКРОНТАБ

Ниже приведены некоторые примеры выражений НКРОНТАБ, которые можно использовать для триггера таймера в функциях Azure.

| Пример            | Когда активируется                     |
|--------------------|------------------------------------|
| `0 */5 * * * *`    | через каждые пять минут            |
| `0 0 * * * *`      | через каждый час      |
| `0 0 */2 * * *`    | через каждые 2 часа               |
| `0 0 9-17 * * *`   | один раз в час с 9:00 до 17:00  |
| `0 30 9 * * *`     | каждый день в 9:30               |
| `0 30 9 * * 1-5`   | каждый рабочий день в 9:30           |
| `0 30 9 * Jan Mon` | в 9:30 каждый понедельник в январе |

> [!NOTE]
> Для выражения НКРОНТАБ требуется **шесть форматов полей** . Шестая координата поля — это значение для секунд, которое помещается в начало выражения. В Azure не поддерживаются пять полей с выражениями cron.

### <a name="ncrontab-time-zones"></a>Часовые пояса НКРОНТАБ

Числа в выражении CRON представляют собой время и дату, а не временной интервал. Например, значение 5 в поле `hour` означает 5:00, а не каждые 5 часов.

[!INCLUDE [functions-timezone](../../includes/functions-timezone.md)]

## <a name="timespan"></a>TimeSpan

 `TimeSpan` можно использовать только для приложения-функции, которая работает в плане службы приложений.

В отличие от выражения CRON значение `TimeSpan` задает интервал времени между каждым вызовом функции. Если функция будет выполняться дольше заданного временного интервала, она немедленно будет активирована таймером.

Когда значения выражения `hh` в виде стоки меньше 24, `TimeSpan` будет использовать формат `hh:mm:ss`. Когда первые две цифры больше или равны 24, будет использован следующий формат `dd:hh:mm`. Ниже приводится несколько примеров.

| Пример      | Когда активируется |
|--------------|----------------|
| "01:00:00"   | каждый час     |
| "00:01:00"   | каждую минуту   |
| "25:00:00"   | каждые 25 дней  |
| "1,00:00:00" | Каждый день      |

## <a name="scale-out"></a>Горизонтальное масштабирование

Если приложение-функция будет развернуто на несколько экземпляров, то только один из экземпляров активируемой по таймеру функции выполняется для всех экземпляров. Он не будет повторно запущен, если еще не запущен необработанный вызов.

## <a name="function-apps-sharing-storage"></a>Совместное использование хранилища приложениями-функциями

При совместном использовании учетных записей хранения в приложениях функций, которые не развернуты в службе приложений, может потребоваться явно назначить идентификатор узла каждому приложению.

| Версия службы "Функции" | Параметр                                              |
| ----------------- | ---------------------------------------------------- |
| 2. x (и выше)  | Переменная среды `AzureFunctionsWebHost__hostid`. |
| 1.x               | `id` в *host.js*                                  |

Можно опустить идентифицирующее значение или вручную задать для каждой идентифицирующей конфигурации приложения-функции другое значение.

Триггер таймера использует блокировку хранилища, чтобы гарантировать наличие только одного экземпляра таймера, когда приложение-функция масштабируется до нескольких экземпляров. Если два приложения-функции совместно используют одну и ту же идентифицирующую конфигурацию, и каждый из них использует триггер таймера, запускается только один таймер.

## <a name="retry-behavior"></a>Поведение при повторе

В отличие от триггера очереди триггер таймера не осуществляет повторную попытку после того, как произошла ошибка выполнения функции. Когда функция возвращает ошибку, следующий раз она будет вызвана только по расписанию.

## <a name="manually-invoke-a-timer-trigger"></a>Вызов триггера таймера вручную

Триггер таймера для функций Azure предоставляет веб-перехватчик HTTP, который можно вызвать для запуска функции вручную. Это может быть очень полезно в следующих сценариях.

* Тестирование интеграции
* Переключение слотов в рамках теста состояния или действия прогрева
* Начальное развертывание функции для немедленного заполнения кэша или таблицы уточняющих запросов в базе данных

Дополнительные сведения о вызове функции, активируемой с помощью таймера, см. в разделе [ручное выполнение функции,](./functions-manually-run-non-http.md) которая активируется по протоколу HTTP.

## <a name="troubleshooting"></a>Диагностика

Дополнительные сведения о том, что делать, когда триггер таймера неисправен, см. в разделе [Расследование проблем, когда функции, вызываемые таймером, не срабатывают](https://github.com/Azure/azure-functions-host/wiki/Investigating-and-reporting-issues-with-timer-triggered-functions-not-firing).

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Перейдите к краткому руководству по использованию триггера таймера](functions-create-scheduled-function.md)

> [!div class="nextstepaction"]
> [Основные понятия триггеров и привязок в Функциях Azure](functions-triggers-bindings.md)
