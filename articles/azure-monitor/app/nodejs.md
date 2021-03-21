---
title: Мониторинг служб Node.js с помощью Azure Application Insights | Документация Майкрософт
description: Используйте Application Insights для мониторинга производительности и диагностики проблем в службах Node.js.
ms.topic: conceptual
ms.date: 06/01/2020
ms.custom: devx-track-js
ms.openlocfilehash: 9eba74bcf2b4b047b3ed881342db4f7b1011f928
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100585761"
---
# <a name="monitor-your-nodejs-services-and-apps-with-application-insights"></a>Мониторинг служб и приложений Node.js с помощью Application Insights

[Application Insights](./app-insights-overview.md) отслеживает серверные службы и компоненты после развертывания, помогая находить и быстро диагностировать производительность и другие проблемы. Вы можете использовать Application Insights для служб Node.js, размещенных в вашем центре обработки данных, виртуальных машинах Azure и веб-приложениях, а также в других общедоступных облаках.

Для получения, хранения и анализа данных мониторинга включите пакет SDK в ваш код, а затем настройте соответствующий ресурс Application Insights в Azure. Пакет SDK отправляет данные в этот ресурс для дальнейшего анализа и исследования.

Пакет SDK для Node.js может автоматически отслеживать входящие и исходящие HTTP-запросы, исключения и некоторые системные метрики. Начиная с версии 0,20, пакет SDK также может отслеживать некоторые распространенные [сторонние пакеты](https://github.com/microsoft/node-diagnostic-channel/tree/master/src/diagnostic-channel-publishers#currently-supported-modules), такие как MongoDB, MySQL и Redis. Все события, связанные с входящим HTTP-запросом, коррелируют для быстрого устранения неполадок.

Вы можете инструментировать вручную и отслеживать дополнительные аспекты приложения и системы с помощью API TelemetryClient. С дополнительными сведениями о API TelemetryClient можно ознакомиться далее в этой статье.

## <a name="get-started"></a>Начало работы

Выполните указанные ниже задачи, чтобы настроить мониторинг для приложения или службы.

### <a name="prerequisites"></a>Предварительные условия

Чтобы начать, у вас должна быть подписка Azure. Вы можете [создать ее бесплатно][azure-free-offer]. Если у вашей организации уже есть подписка Azure, администратору необходимо выполнить [эти инструкции][add-aad-user], чтобы добавить вас в эту подписку.

[azure-free-offer]: https://azure.microsoft.com/free/
[add-aad-user]: ../../active-directory/fundamentals/add-users-azure-active-directory.md

### <a name="set-up-an-application-insights-resource"></a><a name="resource"></a> Настройка ресурса Application Insights

1. Войдите на [портал Azure][portal].
2. [Создание ресурса Application Insights](create-new-resource.md)

### <a name="set-up-the-nodejs-sdk"></a><a name="sdk"></a> Настройка пакета SDK для Node.js

Включите пакет SDK в приложение, чтобы он мог собирать данные.

> [!IMPORTANT]
> Новые регионы Azure **должны** использовать строки подключения вместо ключей инструментирования. [Строка подключения](./sdk-connection-string.md?tabs=nodejs) определяет ресурс, с которым необходимо связать данные телеметрии. Она также позволяет изменить конечные точки, которые ресурс будет использовать в качестве назначения для данных телеметрии. Необходимо скопировать строку подключения и добавить ее в код приложения или переменную среды.

1. Скопируйте ключ инструментирования ресурса (также называемый *iKey*) из созданного ресурса. Application Insights использует ikey для сопоставления данных с ресурсом Azure. Прежде чем пакет SDK сможет использовать ikey, необходимо указать ikey в переменной среды или в коде.  

   ![Копирование ключа инструментирования](./media/nodejs/instrumentation-key-001.png)

2. Добавьте библиотеку пакета SDK для Node.js в зависимости от приложения с помощью файла package.json. Из корневой папки приложения выполните следующую команду:

   ```bash
   npm install applicationinsights --save
   ```

    > [!NOTE]
    > Если вы используете TypeScript, не устанавливайте отдельные пакеты "Ввод". Пакет NPM содержит встроенные вводимые элементы.

3. Загрузите явным образом библиотеку в код. Так как пакет SDK внедряет инструментирование во многие другие библиотеки, вам необходимо загрузить библиотеку как можно раньше, даже до выполнения других инструкций `require`.

   ```javascript
   let appInsights = require('applicationinsights');
   ```
4.  Вы также можете предоставить iKey через переменную среды `APPINSIGHTS_INSTRUMENTATIONKEY` вместо передачи ее вручную в  `setup()` или `new appInsights.TelemetryClient()` . Это позволит хранить ключи ikey вне выделенного исходного кода, и вы можете указать различные ключи ikey для разных сред. Для настройки вызова вручную `appInsights.setup('[your ikey]');` .

    Дополнительные параметры конфигурации приведены в указанных ниже разделах.

    Вы можете использовать пакет SDK, не отправляя данные телеметрии. Для этого задайте `appInsights.defaultClient.config.disableAppInsights = true`.

5. Запустите автоматическую сбор и отправку данных путем вызова `appInsights.start();` .

### <a name="monitor-your-app"></a><a name="monitor"></a> Мониторинг приложения

Пакет SDK автоматически собирает данные телеметрии о среде выполнения Node.js и некоторых распространенных модулях сторонних разработчиков. Используйте приложение для создания этих данных.

Затем на [портале Azure][portal] перейдите к ресурсу Application Insights, созданному ранее. На **временной шкале обзора** просмотрите первые несколько точек данных. Для просмотра более подробных данных выбирайте различные компоненты на диаграммах.

Чтобы просмотреть топологию, обнаруженную для приложения, можно использовать [схему приложений](app-map.md).

#### <a name="no-data"></a>Нет данных

Так как пакет SDK упаковывает данные для передачи, возможна некоторая задержка отображения элементов на портале. Если данные в ресурсе не отображаются, попробуйте сделать следующее:

* Продолжайте использовать приложение. Выполните дополнительные действия, чтобы создать больше данных телеметрии.
* Нажмите кнопку **обновления** в представлении ресурса на портале. Диаграммы периодически обновляются, но если нажать эту кнопку, обновление будет выполнено сразу же.
* [Необходимые порты для исходящего трафика](./ip-addresses.md) должны быть открыты.
* Найти отдельные события можно с помощью функции [поиска](./diagnostic-search.md).
* Проверьте [вопросы и ответы][FAQ].

## <a name="basic-usage"></a>Основное использование

Для встроенной коллекции HTTP-запросов, популярных событий сторонних библиотек, необработанных исключений и системных метрик:

```javascript

let appInsights = require("applicationinsights");
appInsights.setup("[your ikey]").start();

```

> [!NOTE]
> Если ключ инструментирования задан в переменной среды `APPINSIGHTS_INSTRUMENTATIONKEY` , `.setup()` можно вызвать без аргументов. Это позволяет легко использовать разные ключи инструментирования iKey для разных сред.

Перед загрузкой других пакетов загрузите библиотеку Application Insights, `require("applicationinsights")` как можно раньше в скриптах. Это необходимо для того, чтобы библиотека Application Insights могла подготовить последующие пакеты для отслеживания. При возникновении конфликтов с другими библиотеками, выполняющими аналогичные подготовительные действия, попробуйте загрузить библиотеку Application Insights.

В связи с тем, как JavaScript обрабатывает обратные вызовы, требуется дополнительная работа, чтобы отложить запрос между внешними зависимостями и более поздними обратными вызовами. По умолчанию это дополнительное отслеживание включено. отключите его, вызвав `setAutoDependencyCorrelation(false)` , как описано в разделе [конфигурации](#sdk-configuration) ниже.

## <a name="migrating-from-versions-prior-to-022"></a>Миграция с версий до 0,22

Существуют критические изменения между выпусками до версии 0,22 и после. Эти изменения предназначены для обеспечения согласованности с другими Application Insightsми пакетами SDK и позволяют использовать будущие возможности расширения.

В общем случае можно выполнить миграцию следующим образом:

- Замените ссылки на `appInsights.client` `appInsights.defaultClient` .
- Заменить `appInsights.getClient()` ссылки на `new appInsights.TelemetryClient()`
- Замените все аргументы на методы Client. Track * одним объектом, содержащим именованные свойства, в качестве аргументов. См. подсказку встроенного типа интегрированной среды разработки или [телеметритипес](https://github.com/Microsoft/ApplicationInsights-node.js/tree/develop/Declarations/Contracts/TelemetryTypes) для каждого типа телеметрии, кроме объекта.

При доступе к функциям конфигурации пакета SDK без их объединения в можно `appInsights.setup()` найти эти функции в `appInsights.Configurations` (например, `appInsights.Configuration.setAutoCollectDependencies(true)` ). Ознакомьтесь с изменениями конфигурации по умолчанию в следующем разделе.

## <a name="sdk-configuration"></a>Конфигурация пакета SDK

`appInsights`Объект предоставляет ряд методов настройки. Они перечислены в следующем фрагменте кода со значениями по умолчанию.

```javascript
let appInsights = require("applicationinsights");
appInsights.setup("<instrumentation_key>")
    .setAutoDependencyCorrelation(true)
    .setAutoCollectRequests(true)
    .setAutoCollectPerformance(true, true)
    .setAutoCollectExceptions(true)
    .setAutoCollectDependencies(true)
    .setAutoCollectConsole(true)
    .setUseDiskRetryCaching(true)
    .setSendLiveMetrics(false)
    .setDistributedTracingMode(appInsights.DistributedTracingModes.AI)
    .start();
```

Чтобы полностью сопоставить события в службе, задайте `.setAutoDependencyCorrelation(true)`. Благодаря этому пакет SDK может отслеживать контекст в асинхронных обратных вызовах в службе Node.js.

Для получения подробных сведений об этих элементах управления и дополнительных вторичных аргументах изучите их описания в подсказках встроенного типа интегрированной среды разработки или в [applicationinsights. TS.](https://github.com/microsoft/ApplicationInsights-node.js/blob/develop/applicationinsights.ts)

> [!NOTE]
>  По умолчанию `setAutoCollectConsole` настроено *исключение* вызовов `console.log` (и других методов консоли). Будут собираться только вызовы поддерживаемых средств ведения журнала сторонних разработчиков (например, Winston и бунян). Это поведение можно изменить для включения вызовов методов с `console` помощью `setAutoCollectConsole(true, true)` .

### <a name="sampling"></a>Дискретизация

По умолчанию пакет SDK будет отсылать все собранные данные в службу Application Insights. Если собираются данные большого объема, может потребоваться включить выборку для уменьшения объема отправляемых данных. `samplingPercentage`Для этого задайте поле в `config` объекте клиента. Установка значения `samplingPercentage` 100 (по умолчанию) означает, что будут отправлены все данные, а 0 означает, что ничего не будет отправлено.

Если используется автоматическая корреляция, все данные, связанные с одним запросом, будут включены или исключены как единое целое.

Добавьте следующий код, чтобы включить выборку:

```javascript
const appInsights = require("applicationinsights");
appInsights.setup("<instrumentation_key>");
appInsights.defaultClient.config.samplingPercentage = 33; // 33% of all telemetry will be sent to Application Insights
appInsights.start();
```

### <a name="multiple-roles-for-multi-components-applications"></a>Несколько ролей для приложений с несколькими компонентами

Если приложение состоит из нескольких компонентов, которые вы хотите инструментировать для работы с одним и тем же ключом инструментирования, и все они по-прежнему видят эти компоненты в виде отдельных единиц на портале, как если бы они использовали отдельные ключи инструментирования (например, в виде отдельных узлов на схеме приложения), может потребоваться вручную настроить поле RoleName, чтобы отличать данные телеметрии одного компонента от других компонентов Application Insights, отправляя

Чтобы задать поле RoleName, используйте следующую команду:

```javascript
const appInsights = require("applicationinsights");
appInsights.setup("<instrumentation_key>");
appInsights.defaultClient.context.tags[appInsights.defaultClient.context.keys.cloudRole] = "MyRoleName";
appInsights.start();
```

### <a name="automatic-third-party-instrumentation"></a>Автоматическое инструментирование сторонних разработчиков

Для наблюдения за контекстом между асинхронными вызовами некоторые изменения необходимы в сторонних библиотеках, таких как MongoDB и Redis. По умолчанию Application Insights будет использовать [`diagnostic-channel-publishers`](https://github.com/Microsoft/node-diagnostic-channel/tree/master/src/diagnostic-channel-publishers) для обновления некоторых из этих библиотек. Это можно отключить, задав `APPLICATION_INSIGHTS_NO_DIAGNOSTIC_CHANNEL` переменную среды.

> [!NOTE]
> Если задать эту переменную среды, события могут быть неправильно связаны с правильной операцией.

 Отдельные обезьяны — исправления можно отключить, задав `APPLICATION_INSIGHTS_NO_PATCH_MODULES` для переменной среды разделенный запятыми список пакетов для отключения (например, `APPLICATION_INSIGHTS_NO_PATCH_MODULES=console,redis` ), чтобы избежать исправления `console` `redis` пакетов и.

В настоящее время существует девять пакетов, которые инструментированы: `bunyan` ,, `console` ,, `mongodb` `mongodb-core` `mysql` , `redis` , `winston` , `pg` и `pg-pool` . Сведения о том, какие версии этих пакетов исправлены, см. в [файле сведений диагностики-Channel-Publishers](https://github.com/Microsoft/node-diagnostic-channel/blob/master/src/diagnostic-channel-publishers/README.md) .

`bunyan`Исправления, `winston` и `console` будут создавать Application Insights события трассировки в зависимости от того `setAutoCollectConsole` , включено ли. Остальное создаст Application Insights события зависимости в зависимости от того `setAutoCollectDependencies` , включено ли.

### <a name="live-metrics"></a>Интерактивные метрики

Чтобы включить отправку динамических метрик из приложения в Azure, используйте `setSendLiveMetrics(true)` . Фильтрация динамических метрик на портале сейчас не поддерживается.

### <a name="extended-metrics"></a>Расширенные метрики

> [!NOTE]
> Возможность отправки расширенных собственных метрик была добавлена в версии 1.4.0

Чтобы включить отправку расширенных собственных метрик из приложения в Azure, установите отдельный пакет собственных метрик. Пакет SDK будет автоматически загружаться при установке и начинать сбор Node.js собственных метрик.

```bash
npm install applicationinsights-native-metrics
```

В настоящее время пакет машинных метрик выполняет Автосбор времени ЦП для сборки мусора, тактов цикла событий и использования кучи.

- **Сборка мусора**: количество времени ЦП, затраченного на каждый тип сборки мусора, и количество вхождений каждого типа.
- **Цикл событий**: сколько тактов произошло и сколько времени ЦП было затрачено.
- **Куча и не куча**: объем используемой памяти приложения находится в куче или не в куче.

### <a name="distributed-tracing-modes"></a>Распределенные режимы трассировки

По умолчанию пакет SDK будет отсылать заголовки, понятные другим приложениям и службам, которые инструментированы с помощью пакета SDK для Application Insights. При необходимости можно включить отправку и получение заголовков [контекста трассировки W3C](https://github.com/w3c/trace-context) в дополнение к существующим заголовкам AI, чтобы не нарушать корреляцию ни с одной из существующих устаревших служб. Включение заголовков W3C позволит приложению сопоставляться с другими службами, не оснащенными Application Insights, но применяя этот стандарт W3C.

```Javascript
const appInsights = require("applicationinsights");
appInsights
  .setup("<your ikey>")
  .setDistributedTracingMode(appInsights.DistributedTracingModes.AI_AND_W3C)
  .start()
```

## <a name="telemetryclient-api"></a>API TelemetryClient

Полное описание API TelemetryClient см. в статье [API Application Insights для пользовательских событий и метрик](./api-custom-events-metrics.md).

Вы можете отслеживать любой запрос, событие, метрику или исключение с помощью пакета SDK для Node.js для Application Insights. В примере кода ниже приведены некоторые API, которые можно использовать.

```javascript
let appInsights = require("applicationinsights");
appInsights.setup().start(); // assuming ikey in env var. start() can be omitted to disable any non-custom data
let client = appInsights.defaultClient;
client.trackEvent({name: "my custom event", properties: {customProperty: "custom property value"}});
client.trackException({exception: new Error("handled exceptions can be logged with this method")});
client.trackMetric({name: "custom metric", value: 3});
client.trackTrace({message: "trace message"});
client.trackDependency({target:"http://dbname", name:"select customers proc", data:"SELECT * FROM Customers", duration:231, resultCode:0, success: true, dependencyTypeName: "ZSQL"});
client.trackRequest({name:"GET /customers", url:"http://myserver/customers", duration:309, resultCode:200, success:true});

let http = require("http");
http.createServer( (req, res) => {
  client.trackNodeHttpRequest({request: req, response: res}); // Place at the beginning of your request handler
});
```

### <a name="track-your-dependencies"></a>Отслеживание зависимостей

С помощью следующего кода можно отслеживать зависимости:

```javascript
let appInsights = require("applicationinsights");
let client = new appInsights.TelemetryClient();

var success = false;
let startTime = Date.now();
// execute dependency call here....
let duration = Date.now() - startTime;
success = true;

client.trackDependency({target:"http://dbname", name:"select customers proc", data:"SELECT * FROM Customers", duration:duration, resultCode:0, success: true, dependencyTypeName: "ZSQL"});;
```

Пример служебной программы `trackMetric` , с помощью которого можно определить, как долго выполняется планирование циклов событий:  

```javascript
function startMeasuringEventLoop() {
  var startTime = process.hrtime();
  var sampleSum = 0;
  var sampleCount = 0;

  // Measure event loop scheduling delay
  setInterval(() => {
    var elapsed = process.hrtime(startTime);
    startTime = process.hrtime();
    sampleSum += elapsed[0] * 1e9 + elapsed[1];
    sampleCount++;
  }, 0);

  // Report custom metric every second
  setInterval(() => {
    var samples = sampleSum;
    var count = sampleCount;
    sampleSum = 0;
    sampleCount = 0;

    if (count > 0) {
      var avgNs = samples / count;
      var avgMs = Math.round(avgNs / 1e6);
      client.trackMetric({name: "Event Loop Delay", value: avgMs});
    }
  }, 1000);
}
```

### <a name="add-a-custom-property-to-all-events"></a>Добавление пользовательского свойства во все события

С помощью следующего кода можно добавить пользовательское свойство во все события:

```javascript
appInsights.defaultClient.commonProperties = {
  environment: process.env.SOME_ENV_VARIABLE
};
```

### <a name="track-http-get-requests"></a>Отслеживание HTTP-запросов GET

Используйте следующий код для ручной трассировки HTTP-запросов GET:

> [!NOTE]
> По умолчанию все запросы отправляются. Чтобы отключить автоматическую коллекцию, вызовите. Сетаутоколлектрекуестс (false) перед вызовом Start ().

```javascript
appInsights.defaultClient.trackRequest({name:"GET /customers", url:"http://myserver/customers", duration:309, resultCode:200, success:true});
```

Кроме того, можно отслеживание запросов с помощью `trackNodeHttpRequest` метода:

```javascript
var server = http.createServer((req, res) => {
  if ( req.method === "GET" ) {
      appInsights.defaultClient.trackNodeHttpRequest({request:req, response:res});
  }
  // other work here....
  res.end();
});
```

### <a name="track-server-startup-time"></a>Отслеживание времени запуска сервера

С помощью следующего кода можно отслеживать время запуска сервера:

```javascript
let start = Date.now();
server.on("listening", () => {
  let duration = Date.now() - start;
  appInsights.defaultClient.trackMetric({name: "server startup time", value: duration});
});
```

### <a name="flush"></a>Очистка

По умолчанию данные телеметрии передаются в буфер на 15 секунд перед отправкой на сервер приема. Если приложение имеет короткий срок существования (например, средство CLI), то при завершении работы приложения может потребоваться вручную очистить буферизованную телеметрию `appInsights.defaultClient.flush()` .

Если пакет SDK обнаруживает сбой приложения, он вызывает сброс для вас, `appInsights.defaultClient.flush({ isAppCrashing: true })` . При использовании параметра flush `isAppCrashing` предполагается, что приложение находится в ненормальном состоянии, а не подходит для отправки данных телеметрии. Вместо этого пакет SDK сохраняет все буферизованные данные телеметрии в [постоянное хранилище](./data-retention-privacy.md#nodejs) и позволяет завершить работу приложения. При повторном запуске приложения будет предпринята попытка отправить данные телеметрии, сохраненные в постоянное хранилище.

### <a name="preprocess-data-with-telemetry-processors"></a>Предварительная обработка данных с помощью обработчиков телеметрии

Вы можете обрабатывать и фильтровать собранные данные перед их отправкой для хранения с помощью *процессоров телеметрии*. Обработчики данных телеметрии вызываются по одному в том порядке, в котором они были добавлены перед отправкой элемента телеметрии в облако.

```javascript
public addTelemetryProcessor(telemetryProcessor: (envelope: Contracts.Envelope, context: { http.RequestOptions, http.ClientRequest, http.ClientResponse, correlationContext }) => boolean)
```

Если обработчик данных телеметрии возвращает значение false, этот элемент телеметрии не будет отправлен.

Все обработчики телеметрии получают данные телеметрии и конверт для проверки и изменения. Они также получают объект контекста. Содержимое этого объекта определяется `contextObjects` параметром при вызове метода Track для отслеживания телеметрических данных вручную. Для автоматически собираемых данных телеметрии этот объект заполняется доступными сведениями о запросе и содержимым постоянного запроса, предоставленным `appInsights.getCorrelationContext()` (если включена автоматическая корреляция зависимостей).

Тип TypeScript для обработчика данных телеметрии:

```javascript
telemetryProcessor: (envelope: ContractsModule.Contracts.Envelope, context: { http.RequestOptions, http.ClientRequest, http.ClientResponse, correlationContext }) => boolean;
```

Например, процессор, который удаляет данные трассировки стеков из исключений, может быть написан и добавлен следующим образом:

```javascript
function removeStackTraces ( envelope, context ) {
  if (envelope.data.baseType === "Microsoft.ApplicationInsights.ExceptionData") {
    var data = envelope.data.baseData;
    if (data.exceptions && data.exceptions.length > 0) {
      for (var i = 0; i < data.exceptions.length; i++) {
        var exception = data.exceptions[i];
        exception.parsedStack = null;
        exception.hasFullStack = false;
      }
    }
  }
  return true;
}

appInsights.defaultClient.addTelemetryProcessor(removeStackTraces);
```

## <a name="use-multiple-instrumentation-keys"></a>Использование нескольких ключей инструментирования

Можно создать несколько Application Insights ресурсов и отправить каждому из них разные данные, используя соответствующие ключи инструментирования ("iKey").

 Пример:

```javascript
let appInsights = require("applicationinsights");

// configure auto-collection under one ikey
appInsights.setup("_ikey-A_").start();

// track some events manually under another ikey
let otherClient = new appInsights.TelemetryClient("_ikey-B_");
otherClient.trackEvent({name: "my custom event"});
```

## <a name="advanced-configuration-options"></a>Расширенные параметры конфигурации

Клиентский объект содержит `config` свойство с множеством необязательных параметров для расширенных сценариев. Их можно задать следующим образом:

```javascript
client.config.PROPERTYNAME = VALUE;
```

Эти свойства зависят от клиента, поэтому их можно настроить `appInsights.defaultClient` отдельно от клиентов, созданных с помощью `new appInsights.TelemetryClient()` .

| Свойство.                        | Описание                                                                                                |
| ------------------------------- |------------------------------------------------------------------------------------------------------------|
| instrumentationKey              | Идентификатор ресурса Application Insights.                                                      |
| endpointUrl                     | Конечная точка приема, в которую отправляются полезные данные телеметрии.                                                      |
| куиккпулсехост                  | Узел Live Metrics Stream для отправки данных телеметрии динамических метрик.                                            |
| проксихттпурл                    | Прокси-сервер для HTTP-трафика пакета SDK (необязательно, по умолчанию извлеченный из `http_proxy` переменной среды).     |
| проксихттпсурл                   | Прокси-сервер для трафика HTTPS пакета SDK (необязательно, по умолчанию извлеченный из `https_proxy` переменной среды).   |
| хттпажент                       | HTTP. Агент, используемый для HTTP-трафика пакета SDK (необязательный, неопределенный по умолчанию).                                   |
| хттпсажент                      | HTTPS. Агент, используемый для трафика HTTPS пакета SDK (необязательно, значение по умолчанию не определено).                                 |
| maxBatchSize                    | Максимальное число элементов телеметрии, включаемых в полезные данные в конечную точку приема (по умолчанию `250` ).   |
| максбатчинтервалмс              | Максимальное время ожидания в maxBatchSize полезных данных (по умолчанию `15000` ).               |
| дисаблеаппинсигхтс              | Флаг, указывающий, отключена ли передача телеметрии (по умолчанию `false` ).                                 |
| самплингперцентаже              | Процент элементов телеметрии, которые должны быть переданы (по умолчанию `100` ).                      |
| коррелатионидретринтервалмс    | Время ожидания перед повторной попыткой получения идентификатора для корреляции между компонентами (по умолчанию `30000` ).     |
| коррелатионхеадерексклудеддомаинс| Список доменов, исключаемых из вставки заголовка корреляции между компонентами (по умолчанию см. [config. TS](https://github.com/Microsoft/ApplicationInsights-node.js/blob/develop/Library/Config.ts)).|

## <a name="next-steps"></a>Дальнейшие действия

* [Навигация и панели мониторинга на портале Application Insights](./overview-dashboard.md)
* [Знакомство с аналитикой в Application Insights](../logs/log-analytics-tutorial.md)

<!--references-->

[portal]: https://portal.azure.com/
[FAQ]: ../faq.md
