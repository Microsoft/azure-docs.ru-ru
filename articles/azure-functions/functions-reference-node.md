---
title: Справочник разработчика JavaScript по функциям Azure
description: Узнайте, как разрабатывать функции с помощью JavaScript.
ms.assetid: 45dedd78-3ff9-411f-bb4b-16d29a11384c
ms.topic: conceptual
ms.date: 03/07/2021
ms.custom: devx-track-js
ms.openlocfilehash: 971fb2a3239614a708e14c109e567081f1ec9ff6
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102614910"
---
# <a name="azure-functions-javascript-developer-guide"></a>Руководство разработчика JavaScript для Функций Azure

Это краткое описание содержит подробные сведения, помогающие в разработке функций Azure с помощью JavaScript.

В качестве Express.js, Node.js или разработчика JavaScript, если вы не знакомы с функциями Azure, сначала ознакомьтесь с одной из следующих статей:

| Начало работы | Основные понятия| Направляемое обучение |
| -- | -- | -- | 
| <ul><li>[ ФункцияNode.js с использованием Visual Studio Code](./create-first-function-vs-code-node.md)</li><li>[Node.js функции с помощью терминала/командной строки](./create-first-function-cli-node.md)</li><li>[ ФункцияNode.js, использующая портал Azure](functions-create-function-app-portal.md)</li></ul> | <ul><li>[Руководство для разработчиков](functions-reference.md)</li><li>[Сравнение вариантов размещения](functions-scale.md)</li><li>[Функции TypeScript](#typescript)</li><li>[&nbsp;Вопросы производительности](functions-best-practices.md)</li></ul> | <ul><li>[Создание бессерверных приложений.](/learn/paths/create-serverless-applications/)</li><li>[Рефакторинг интерфейсов API Node.js и Express для бессерверных интерфейсов API](/learn/modules/shift-nodejs-express-apis-serverless/)</li></ul> |

## <a name="javascript-function-basics"></a>Основы функций JavaScript

Функция JavaScript (Node.js) является экспортированной `function` , которая выполняется при активации ([триггеры настраиваются в function.json](functions-triggers-bindings.md)). Первым аргументом, передаваемым каждой функции `context` , является объект, который используется для получения и отправки данных привязки, ведения журнала и взаимодействия со средой выполнения.

## <a name="folder-structure"></a>Структура папок

Необходимая структура папок для проекта JavaScript выглядит следующим образом. Это значение по умолчанию можно изменить. Дополнительные сведения см. в разделе о [scriptFile](#using-scriptfile) ниже.

```
FunctionsProject
 | - MyFirstFunction
 | | - index.js
 | | - function.json
 | - MySecondFunction
 | | - index.js
 | | - function.json
 | - SharedCode
 | | - myFirstHelperFunction.js
 | | - mySecondHelperFunction.js
 | - node_modules
 | - host.json
 | - package.json
 | - extensions.csproj
```

В корневой папке проекта существует общий файл [host.json](functions-host-json.md), который может использоваться для настройки приложения-функции. У каждой функции есть папка с собственным файлом кода (JS) и файлом конфигурации привязки (function.json). Имя родительского каталога файла `function.json` всегда является именем этой функции.

Расширения привязки, необходимые в [версии 2.x](functions-versions.md) среды выполнения функций, определены в файле `extensions.csproj` с фактическими файлами библиотеки в папке `bin`. При локальной разработке необходимо [зарегистрировать расширения привязки](./functions-bindings-register.md#extension-bundles). При разработке функций на портале Azure эта регистрация выполняется автоматически.

## <a name="exporting-a-function"></a>Экспорт функции

Функции JavaScript должны быть экспортированы с помощью [`module.exports`](https://nodejs.org/api/modules.html#modules_module_exports) (или [`exports`](https://nodejs.org/api/modules.html#modules_exports) ). Экспортированная функция должна быть функцией JavaScript, которая выполняется при активации.

По умолчанию среда выполнения Функций ищет функцию в файле `index.js`, где `index.js` использует тот же родительский каталог, что и соответствующий файл `function.json`. В стандартном случае экспортированная функция должна быть единственным экземпляром экспорта из соответствующего файла (экспорта с именем `run` или `index`). Чтобы настроить расположение файла и имя экспорта функции, ознакомьтесь с разделом ниже о [настройке точки входа функции](functions-reference-node.md#configure-function-entry-point).

Экспортированной функции передается число аргументов при выполнении. Первый аргумент, который она всегда принимает, — это объект `context`. Если функция является синхронной (не возвращает обещание), необходимо передать `context` объект, так как `context.done` для правильного использования требуется вызов.

```javascript
// You should include context, other arguments are optional
module.exports = function(context, myTrigger, myInput, myOtherInput) {
    // function logic goes here :)
    context.done();
};
```

### <a name="exporting-an-async-function"></a>Экспорт асинхронной функции
При использовании [`async function`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/async_function) объявления или простого JavaScript в [](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) версии 2. x среды выполнения функций нет необходимости явно вызывать [`context.done`](#contextdone-method) обратный вызов, чтобы сообщить о завершении функции. Ваша функция завершается при завершении экспортированной асинхронной функции или обещания. Для функций, предназначенных для среды выполнения версии 1. x, по-прежнему необходимо вызывать, [`context.done`](#contextdone-method) когда выполняется код.

Следующий пример — это простая функция, которая записывает в журнал, что она была запущена, и немедленно завершает выполнение.

```javascript
module.exports = async function (context) {
    context.log('JavaScript trigger function processed a request.');
};
```

При экспорте асинхронной функции вы также можете настроить выходные привязки, чтобы принять значение `return`. Мы рекомендуем выполнять этот шаг, если существует только одна выходная привязка.

Чтобы назначить выходные данные с помощью `return`, измените свойство `name` на `$return` в `function.json`.

```json
{
  "type": "http",
  "direction": "out",
  "name": "$return"
}
```

В этом случае функция должна выглядеть следующим образом:

```javascript
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');
    // You can call and await an async method here
    return {
        body: "Hello, world!"
    };
}
```

## <a name="bindings"></a>Привязки 
В JavaScript [привязки](functions-triggers-bindings.md) настраиваются и определяются в файле function.json функции. Функции взаимодействуют с привязками несколькими способами.

### <a name="inputs"></a>Входные данные
Входные данные в Функциях Azure делятся на две категории: входные данные от триггера и дополнительные входные данных. Привязки триггера и другие привязки для ввода (привязки `direction === "in"`) могут считываться функцией тремя способами.
 - **_[Рекомендуется.]_ Как параметры, передаваемые функции.** Они передаются в функцию в том же порядке, в каком они определены в файле *function.json*. `name`Свойство, определенное в *function.js* , не обязательно должно совпадать с именем параметра, хотя он должен.
 
   ```javascript
   module.exports = async function(context, myTrigger, myInput, myOtherInput) { ... };
   ```
   
 - **Как элементы объекта [`context.bindings`](#contextbindings-property).** Каждый элемент назван по свойству `name`, определенному в файле *function.json*.
 
   ```javascript
   module.exports = async function(context) { 
       context.log("This is myTrigger: " + context.bindings.myTrigger);
       context.log("This is myInput: " + context.bindings.myInput);
       context.log("This is myOtherInput: " + context.bindings.myOtherInput);
   };
   ```
   
 - **В качестве входных данных при использовании объекта JavaScript [`arguments`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments).** Этот способ по сути идентичен передаче входных данных в качестве параметров, но позволяет динамически обрабатывать входные данные.
 
   ```javascript
   module.exports = async function(context) { 
       context.log("This is myTrigger: " + arguments[1]);
       context.log("This is myInput: " + arguments[2]);
       context.log("This is myOtherInput: " + arguments[3]);
   };
   ```

### <a name="outputs"></a>Выходные данные
Выходные данные (привязки `direction === "out"`) могут быть записаны в функцию несколькими способами. Во всех случаях свойство `name` привязки, определенное в файле *function.json*, соответствует имени элемента объекта, записанного в функции. 

Вы можете назначить данные выходным привязкам одним из следующих способов (не объединяя эти методы):

- **_[Рекомендуется, если существует несколько экземпляров выходных данных.]_ Возврат объекта.** При использовании функции возврата Async/Promise можно вернуть объект с назначенными выходными данными. В приведенном ниже примере выходные привязки в файле *function.json* называются httpResponse и queueOutput.

  ```javascript
  module.exports = async function(context) {
      let retMsg = 'Hello, world!';
      return {
          httpResponse: {
              body: retMsg
          },
          queueOutput: retMsg
      };
  };
  ```

  При использовании синхронной функции можно вернуть этот объект с помощью [`context.done`](#contextdone-method) (см. пример).
- **_[Рекомендуется для одного экземпляра выходных данных.]_ Возвращает значение напрямую и с использованием имени привязки $return.** Этот метод работает для функций возврата (асинхронной или функции обещаний). См. пример в разделе [Экспорт асинхронной функции](#exporting-an-async-function). 
- **Присвоение значений для свойства `context.bindings`**. Вы можете присвоить значения непосредственно для context.bindings.

  ```javascript
  module.exports = async function(context) {
      let retMsg = 'Hello, world!';
      context.bindings.httpResponse = {
          body: retMsg
      };
      context.bindings.queueOutput = retMsg;
      return;
  };
  ```

### <a name="bindings-data-type"></a>Тип привязки данных

Чтобы определить тип данных для входной привязки, используйте свойство `dataType` в определении привязки. Например, чтобы прочитать содержимое HTTP-запроса в двоичном формате, используйте тип `binary`:

```json
{
    "type": "httpTrigger",
    "name": "req",
    "direction": "in",
    "dataType": "binary"
}
```

Параметры для `dataType` — это `binary`, `stream` и `string`.

## <a name="context-object"></a>Объект context

Среда выполнения использует `context` объект для передачи данных в функцию и в среду выполнения. Используется для чтения и настройки данных из привязок, а также для записи в журналы `context` объект всегда является первым параметром, передаваемым функции.

Для функций, иссвященных синхронному коду, объект контекста включает `done` обратный вызов, который вызывается при завершении обработки функции. Явный вызов `done` не требуется при написании асинхронного кода; `done` обратный вызов вызывается неявно.

```javascript
module.exports = (context) => {

    // function logic goes here

    context.log("The function has executed.");

    context.done();
};
```

Контекст, переданный в функцию, предоставляет `executionContext` свойство, которое является объектом со следующими свойствами:

| Имя свойства  | Тип  | Описание |
|---------|---------|---------|
| `invocationId` | Строка | Предоставляет уникальный идентификатор для конкретного вызова функции. |
| `functionName` | Строка | Предоставляет имя выполняемой функции. |
| `functionDirectory` | Строка | Предоставляет каталог приложения функций. |

В следующем примере показано, как вернуть `invocationId` .

```javascript
module.exports = (context, req) => {
    context.res = {
        body: context.executionContext.invocationId
    };
    context.done();
};
```

### <a name="contextbindings-property"></a>Свойство context.bindings

```js
context.bindings
```

Возвращает именованный объект, используемый для чтения или назначения данных привязки. Доступ к данным входных и триггеров можно получить, читая свойства в `context.bindings` . Данные выходной привязки могут быть назначены путем добавления данных в `context.bindings`

Например, следующие определения привязки в function.json позволяют вам получить доступ к содержимому очереди из `context.bindings.myInput` и назначить выходные данные очереди, используя `context.bindings.myOutput`.

```json
{
    "type":"queue",
    "direction":"in",
    "name":"myInput"
    ...
},
{
    "type":"queue",
    "direction":"out",
    "name":"myOutput"
    ...
}
```

```javascript
// myInput contains the input data, which may have properties such as "name"
var author = context.bindings.myInput.name;
// Similarly, you can set your output data
context.bindings.myOutput = { 
        some_text: 'hello world', 
        a_number: 1 };
```

Вы можете определить выходные данные привязки с помощью метода `context.done` вместо объекта `context.binding` (см. ниже).

### <a name="contextbindingdata-property"></a>Свойство context.bindingData

```js
context.bindingData
```

Это свойство возвращает именованный объект, содержащий метаданные триггера и данные вызова функции (`invocationId`, `sys.methodName`, `sys.utcNow`, `sys.randGuid`). Пример метаданных триггера см. [по этой ссылке с примером для концентраторов событий](functions-bindings-event-hubs-trigger.md).

### <a name="contextdone-method"></a>Метод context.done

```js
context.done([err],[propertyBag])
```

Позволяет среде выполнения узнать, что ваш код выполнен. Если функция использует [`async function`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/async_function) объявление, вам не нужно использовать `context.done()` . Обратный вызов `context.done` выполняется неявным образом. Асинхронные функции доступны в Node 8 или более поздней версии, для которой требуется среда выполнения Функций версии 2.x.

Если функция не является асинхронной, **необходимо вызвать метод** , `context.done` чтобы сообщить среде выполнения о завершении функции. Если этот метод отсутствует, истекает время ожидания выполнения.

Метод `context.done` позволяет передавать в среду выполнения и определяемые пользователем сообщения об ошибке, и объект JSON, содержащий выходные данные привязки. Свойства, переданные в `context.done`, перезапишут любые значения, заданные для объекта `context.bindings`.

```javascript
// Even though we set myOutput to have:
//  -> text: 'hello world', number: 123
context.bindings.myOutput = { text: 'hello world', number: 123 };
// If we pass an object to the done function...
context.done(null, { myOutput: { text: 'hello there, world', noNumber: true }});
// the done method overwrites the myOutput binding to be: 
//  -> text: 'hello there, world', noNumber: true
```

### <a name="contextlog-method"></a>Метод context.log  

```js
context.log(message)
```

Позволяет записывать данные в журналы функций потоковой передачи на уровне трассировки по умолчанию с другими доступными уровнями ведения журнала. Ведение журнала трассировки подробно описано в следующем разделе. 

## <a name="write-trace-output-to-logs"></a>Запись выходных данных трассировки в журналы

В функциях используются `context.log` методы для записи выходных данных трассировки в журналы и консоль. При вызове `context.log()` сообщение записывается в журналы на уровне трассировки по умолчанию, который является _информационным_ уровнем трассировки. Функции интегрируются с Azure Application Insights для более эффективного захвата журналов приложений функций. Application Insights, часть Azure Monitor, предоставляет средства для сбора, визуального отображения и анализа телеметрии приложений и выходных данных трассировки. Дополнительные сведения см. в статье [мониторинг функций Azure](functions-monitoring.md).

В следующем примере записывается журнал на уровне трассировки info, включая идентификатор вызова:

```javascript
context.log("Something has happened. " + context.invocationId); 
```

Все методы `context.log` поддерживают тот же формат параметров, что и метод Node.js [util.format](https://nodejs.org/api/util.html#util_util_format_format). Просмотрите следующий код, который выполняет запись в журналы функций, используя уровень трассировки по умолчанию:

```javascript
context.log('Node.js HTTP trigger function processed a request. RequestUri=' + req.originalUrl);
context.log('Request Headers = ' + JSON.stringify(req.headers));
```

Этот же код можно записать в таком формате:

```javascript
context.log('Node.js HTTP trigger function processed a request. RequestUri=%s', req.originalUrl);
context.log('Request Headers = ', JSON.stringify(req.headers));
```

> [!NOTE]  
> Не используйте `console.log` для записи выходных данных трассировки. Поскольку выходные данные из `console.log` перехватываются на уровне приложения-функции, оно не привязано к конкретному вызову функции и не отображается в журналах конкретной функции. Кроме того, версия 1. x среды выполнения функций не поддерживает использование `console.log` для записи в консоль.

### <a name="trace-levels"></a>Уровни трассировки

Помимо уровня по умолчанию доступны следующие методы ведения журнала, позволяющие создавать журналы функций на конкретных уровнях трассировки.

| Метод                 | Описание                                |
| ---------------------- | ------------------------------------------ |
| **Ошибка (_сообщение_)**   | Записывает событие уровня ошибки в журналы.   |
| **warn(_message_)**    | Записывает событие уровня предупреждения в журналы. |
| **сведения (_сообщение_)**    | Записывает сообщение в журнал на уровне сведений или более низком.    |
| **verbose(_message_)** | Записывает сообщение в журнал на уровне детализации.           |

В следующем примере тот же журнал записывается на уровне трассировки предупреждений, а не на уровне Info:

```javascript
context.log.warn("Something has happened. " + context.invocationId); 
```

Так как _уровень ошибок_ является наивысшим уровнем трасировки, эта трассировка записывается в выходные данные на всех уровнях трассировки при условии, что ведение журнала включено.

### <a name="configure-the-trace-level-for-logging"></a>Настройка уровня трассировки для ведения журнала

Функции позволяют определить пороговый уровень трассировки для записи в журналы или консоль. Определенные пороговые значения зависят от версии среды выполнения функций.

# <a name="v2x"></a>[v2. x +](#tab/v2)

Чтобы установить пороговое значение для трассировок, записываемых в журналы, используйте `logging.logLevel` свойство в host.jsдля файла. Этот объект JSON позволяет определить пороговое значение по умолчанию для всех функций в приложении-функции, а также задать определенные пороговые значения для отдельных функций. Дополнительные сведения см. в статье [Настройка мониторинга для функций Azure](configure-monitoring.md).

# <a name="v1x"></a>[Версия 1.x](#tab/v1)

Чтобы задать пороговое значение для всех трассировок, записываемых в журналы и консоль, используйте `tracing.consoleLevel` свойство в host.jsдля файла. Этот параметр применяется ко всем функциям в приложении-функции. В следующем примере задается пороговое значение трассировки, чтобы включить подробное ведение журнала:

```json
{
    "tracing": {
        "consoleLevel": "verbose"
    }
}  
```

Значения **consoleLevel** соответствуют именам методов в `context.log`. Чтобы отключить ведение журнала трассировки в консоли, задайте для параметра **consoleLevel** значение _off_. Дополнительные сведения см. [ в разделеhost.jsпо ссылке на v1. x](functions-host-json-v1.md).

---

### <a name="log-custom-telemetry"></a>Регистрация пользовательской телеметрии

По умолчанию функции записывают выходные данные в виде трассировок в Application Insights. Для большего контроля можно использовать [пакет SDK для Application Insights Node.js](https://github.com/microsoft/applicationinsights-node.js) , чтобы отправить пользовательские данные телеметрии в экземпляр Application Insights. 

# <a name="v2x"></a>[v2. x +](#tab/v2)

```javascript
const appInsights = require("applicationinsights");
appInsights.setup();
const client = appInsights.defaultClient;

module.exports = function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    // Use this with 'tagOverrides' to correlate custom telemetry to the parent function invocation.
    var operationIdOverride = {"ai.operation.id":context.traceContext.traceparent};

    client.trackEvent({name: "my custom event", tagOverrides:operationIdOverride, properties: {customProperty2: "custom property value"}});
    client.trackException({exception: new Error("handled exceptions can be logged with this method"), tagOverrides:operationIdOverride});
    client.trackMetric({name: "custom metric", value: 3, tagOverrides:operationIdOverride});
    client.trackTrace({message: "trace message", tagOverrides:operationIdOverride});
    client.trackDependency({target:"http://dbname", name:"select customers proc", data:"SELECT * FROM Customers", duration:231, resultCode:0, success: true, dependencyTypeName: "ZSQL", tagOverrides:operationIdOverride});
    client.trackRequest({name:"GET /customers", url:"http://myserver/customers", duration:309, resultCode:200, success:true, tagOverrides:operationIdOverride});

    context.done();
};
```

# <a name="v1x"></a>[Версия 1.x](#tab/v1)

```javascript
const appInsights = require("applicationinsights");
appInsights.setup();
const client = appInsights.defaultClient;

module.exports = function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    // Use this with 'tagOverrides' to correlate custom telemetry to the parent function invocation.
    var operationIdOverride = {"ai.operation.id":context.operationId};

    client.trackEvent({name: "my custom event", tagOverrides:operationIdOverride, properties: {customProperty2: "custom property value"}});
    client.trackException({exception: new Error("handled exceptions can be logged with this method"), tagOverrides:operationIdOverride});
    client.trackMetric({name: "custom metric", value: 3, tagOverrides:operationIdOverride});
    client.trackTrace({message: "trace message", tagOverrides:operationIdOverride});
    client.trackDependency({target:"http://dbname", name:"select customers proc", data:"SELECT * FROM Customers", duration:231, resultCode:0, success: true, dependencyTypeName: "ZSQL", tagOverrides:operationIdOverride});
    client.trackRequest({name:"GET /customers", url:"http://myserver/customers", duration:309, resultCode:200, success:true, tagOverrides:operationIdOverride});

    context.done();
};
```

---

Параметр `tagOverrides` присваивает параметру `operation_Id` значение идентификатора вызова функции. Этот параметр позволяет сопоставлять все автоматически создаваемые и пользовательские данные телеметрии с конкретным вызовом функции.

## <a name="http-triggers-and-bindings"></a>Триггеры и привязки HTTP

Триггеры HTTP и webhook, а также привязки вывода HTTP используют объекты запроса и ответа для обмена сообщениями HTTP.  

### <a name="request-object"></a>Объект запроса

У объекта (запроса) `context.req` есть следующие свойства:

| Свойство.      | Описание                                                    |
| ------------- | -------------------------------------------------------------- |
| _body_        | Объект, содержащий текст запроса.               |
| _Верхний_     | Объект, содержащий заголовок запроса.                   |
| _Method_      | Метод HTTP, используемый для запроса.                                |
| _оригиналурл_ | URL-адрес запроса.                                        |
| _params_      | Объект, содержащий параметры маршрутизации запроса. |
| _запрос_       | Объект, содержащий параметры запроса.                  |
| _rawBody_     | Текст сообщения в виде строки.                           |


### <a name="response-object"></a>Объект ответа

У объекта (ответа) `context.res` есть следующие свойства:

| Свойство.  | Описание                                               |
| --------- | --------------------------------------------------------- |
| _body_    | Объект, содержащий текст ответа.         |
| _Верхний_ | Объект, содержащий заголовок ответа.             |
| _isRaw_   | Указывает, что форматирование пропускается для ответа.    |
| _status_  | Код состояния HTTP ответа.                     |
| _сеанс_ | Массив объектов cookie HTTP, заданных в ответе. У объекта cookie HTTP есть `name` `value` и другие свойства файлов cookie, такие как `maxAge` или `sameSite` . |

### <a name="accessing-the-request-and-response"></a>Доступ к запросу и ответу 

При работе с триггерами HTTP вы можете получить доступ к объектам запроса и ответа HTTP одним из нескольких способов.

+ **От свойств `req` и `res` объекта `context`.** В этом случае можно использовать обычный шаблон доступа к данным HTTP из объекта context вместо полного шаблона `context.bindings.name`. Следующий пример демонстрирует доступ к объектам `req` и `res` в `context`:

    ```javascript
    // You can access your HTTP request off the context ...
    if(context.req.body.emoji === ':pizza:') context.log('Yay!');
    // and also set your HTTP response
    context.res = { status: 202, body: 'You successfully ordered more coffee!' }; 
    ```

+ **С помощью именованных входных и выходных привязок.** В этом случае триггер и привязки HTTP работают как любая другая привязка. В следующем примере объект ответа задается с помощью именованной привязки `response`: 

    ```json
    {
        "type": "http",
        "direction": "out",
        "name": "response"
    }
    ```
    ```javascript
    context.bindings.response = { status: 201, body: "Insert succeeded." };
    ```
+ **_[Только ответ.]_ Путем вызова `context.res.send(body?: any)`.** HTTP-ответ создается с входными данными `body` в качестве текста ответа. `context.done()` вызывается неявным образом.

+ **_[Только ответ.]_ Путем вызова `context.done()`.** Специальный тип привязки HTTP возвращает ответ, который передается в `context.done()` метод. Следующая привязка вывода HTTP определяет параметр вывода `$return`:

    ```json
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
    ``` 
    ```javascript
     // Define a valid response object.
    res = { status: 201, body: "Insert succeeded." };
    context.done(null, res);   
    ```  

## <a name="scaling-and-concurrency"></a>Масштабирование и параллелизм

По умолчанию функции Azure автоматически отслеживают нагрузку на приложение и при необходимости создают дополнительные экземпляры узлов для Node.js. Функции используют встроенные пороговые значения (недоступные для настройки пользователем) для различных типов триггеров, чтобы решить, когда следует добавлять экземпляры (например, возраст сообщений и размер очереди для QueueTrigger). Дополнительные сведения см. в статье, [посвященной планам с оплатой по мере использования и планам "Премиум"](event-driven-scaling.md).

Такое поведение масштабирования достаточно для многих Node.js приложений. Для приложений, зависящих от ЦП, производительность можно повысить, используя несколько языковых рабочих процессов.

По умолчанию каждый экземпляр узла функций имеет рабочий процесс с одним языком. Вы можете увеличить количество рабочих процессов на узел (до 10) с помощью параметра приложения [FUNCTIONS_WORKER_PROCESS_COUNT](functions-app-settings.md#functions_worker_process_count). Затем Функции Azure пытаются равномерно распределять одновременные вызовы функций между этими рабочими процессами. 

FUNCTIONS_WORKER_PROCESS_COUNT применяется к каждому узлу, создаваемому функциями при масштабировании приложения для удовлетворения потребности. 

## <a name="node-version"></a>Версия узла

В следующей таблице показаны поддерживаемые версии Node.js для каждой основной версии среды выполнения функций по операционной системе:

| Версия службы "Функции" | Версия узла (Windows) | Версия узла (Linux) |
|---|---| --- |
| 3. x (рекомендуется) | `~14` такую<br/>`~12`<br/>`~10` | `node|14` такую<br/>`node|12`<br/>`node|10` |
| 2.x  | `~12`<br/>`~10`<br/>`~8` | `node|10`<br/>`node|8`  |
| 1.x | 6.11.2 (заблокировано средой выполнения) | Н/Д |

Можно просмотреть текущую версию, используемую средой выполнения, записывая `process.version` в журнал любую функцию.

### <a name="setting-the-node-version"></a>Задание версии узла

Для приложений-функций Windows нацеливание на версию в Azure, задав `WEBSITE_NODE_DEFAULT_VERSION` для [параметра приложения](functions-how-to-use-azure-function-app-settings.md#settings) поддерживаемую версию LTS, например `~14` .

Для приложений с функциями Linux выполните следующую команду Azure CLI, чтобы обновить версию узла.

```bash
az functionapp config set --linux-fx-version "node|14" --name "<MY_APP_NAME>" --resource-group "<MY_RESOURCE_GROUP_NAME>"
```

## <a name="dependency-management"></a>Управление зависимостями
Чтобы использовать библиотеки сообщества в коде JavaScript, как показано в следующем примере, необходимо убедиться, что установлены все зависимости приложения-функции в Azure.

```javascript
// Import the underscore.js library
var _ = require('underscore');
var version = process.version; // version === 'v6.5.0'

module.exports = function(context) {
    // Using our imported underscore.js library
    var matched_names = _
        .where(context.bindings.myInput.names, {first: 'Carla'});
```

> [!NOTE]
> Следует определить файл `package.json` в корне вашего приложения-функции. После этого все функции в приложении будут совместно использовать одни и те же кэшированные пакеты, что обеспечивает наилучшую производительность. При возникновении конфликтов версий их можно разрешить, добавив файл `package.json` в папку определенной функции.  

При развертывании приложения-функции из системы управления версиями любой файл `package.json`, присутствующий в вашем репозитории, вызовет `npm install` в своей папке во время развертывания. Но при развертывании с помощью портала или интерфейса командной строки вам придется вручную устанавливать пакеты.

Существует два способа установки пакетов в приложение-функцию. 

### <a name="deploying-with-dependencies"></a>Развертывание с зависимостями
1. Установите все необходимые пакеты в локальной среде, запустив `npm install`.

2. Разверните свой код и убедитесь, что папка `node_modules` включена в развертывание. 


### <a name="using-kudu"></a>С помощью Kudu
1. Перейдите к `https://<function_app_name>.scm.azurewebsites.net`.

2. Щелкните **консоль отладки**  >  **cmd**.

3. Перейдите к `D:\home\site\wwwroot`, а затем перетащите файл package.json в папку **wwwroot** в верхней части страницы.  
    Существуют другие способы передачи файлов в приложение-функцию. Дополнительные сведения см. в разделе [Как обновить файлы приложения-функции](functions-reference.md#fileupdate). 

4. После загрузки файла package.json запустите команду `npm install` в **консоли удаленного выполнения Kudu**.  
    В результате этого действия будут загружены пакеты, указанные в файле package.json, и перезапущено приложение-функция.

## <a name="environment-variables"></a>Переменные среды

Добавьте собственные переменные среды в приложение-функцию, как в локальной, так и в облачной средах, например в рабочие секреты (строки подключения, ключи и конечные точки) или параметры среды (например, переменные профилирования). Доступ к этим параметрам осуществляется с помощью `process.env` в коде функции.

### <a name="in-local-development-environment"></a>В локальной среде разработки

При локальном запуске проект функций включает [ `local.settings.json` файл](./functions-run-local.md), в котором переменные среды хранятся в `Values` объекте. 

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "",
    "FUNCTIONS_WORKER_RUNTIME": "node",
    "translatorTextEndPoint": "https://api.cognitive.microsofttranslator.com/",
    "translatorTextKey": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "languageWorkers__node__arguments": "--prof"
  }
}
```

### <a name="in-azure-cloud-environment"></a>В облачной среде Azure

При запуске в Azure приложение функции позволяет задать использование [параметров приложения](functions-app-settings.md), таких как строки подключения службы, и предоставляет эти параметры в виде переменных среды во время выполнения. 

[!INCLUDE [Function app settings](../../includes/functions-app-settings.md)]

### <a name="access-environment-variables-in-code"></a>Доступ к переменным среды в коде

Получите доступ к параметрам приложения как к переменным среды с помощью `process.env` , как показано здесь во втором и третьем вызовах, `context.log()` где мы заносимся в журнал `AzureWebJobsStorage` `WEBSITE_SITE_NAME` переменных среды и:

```javascript
module.exports = async function (context, myTimer) {

    context.log("AzureWebJobsStorage: " + process.env["AzureWebJobsStorage"]);
    context.log("WEBSITE_SITE_NAME: " + process.env["WEBSITE_SITE_NAME"]);
};
```

## <a name="ecmascript-modules-preview"></a><a name="ecmascript-modules"></a>Модули ECMAScript (Предварительная версия)

> [!NOTE]
> Так как модули ECMAScript в настоящее время помечены как *экспериментальные* в Node.js 14, они доступны в виде предварительной версии функции Node.js 14 функций Azure. До тех пор, пока поддержка Node.js 14 для модулей ECMAScript не будет *стабильной*, ожидать изменения его API или поведения.

Модули [ECMAScript](https://nodejs.org/docs/latest-v14.x/api/esm.html#esm_modules_ecmascript_modules) (модули ES) — это новая официальная система стандартного модуля для Node.js. До сих пор в примерах кода в этой статье используется синтаксис CommonJS. При выполнении функций Azure в Node.js 14 можно выбрать запись функций с помощью синтаксиса ES modules.

Чтобы использовать модули ES в функции, измените имя ее файла, чтобы использовать `.mjs` расширение. Следующий пример файла *index. МЖС* — это функция, активируемая HTTP, которая использует синтаксис ES-модулей для импорта `uuid` библиотеки и возвращения значения.

```js
import { v4 as uuidv4 } from 'uuid';

export default async function (context, req) {
    context.res.body = uuidv4();
};
```

## <a name="configure-function-entry-point"></a>Настройка точки входа функции

Свойства `function.json`, а именно `scriptFile` и `entryPoint`, позволяют настроить расположение и имя экспортированной функции. Эти свойства важны, если JavaScript является транскомпилированным.

### <a name="using-scriptfile"></a>Использование `scriptFile`

По умолчанию функция JavaScript выполняется из файла `index.js`, который расположен в том же родительском каталоге, что и соответствующий файл `function.json`.

`scriptFile` можно использовать для получения структуры папок, которая выглядит следующим образом:

```
FunctionApp
 | - host.json
 | - myNodeFunction
 | | - function.json
 | - lib
 | | - sayHello.js
 | - node_modules
 | | - ... packages ...
 | - package.json
```

Файл `function.json` для `myNodeFunction` должен включать свойство `scriptFile`, указывающее на файл с экспортированной функцией, которую нужно выполнить.

```json
{
  "scriptFile": "../lib/sayHello.js",
  "bindings": [
    ...
  ]
}
```

### <a name="using-entrypoint"></a>Использование `entryPoint`

В `scriptFile` (или `index.js`) функции должны экспортироваться с использованием `module.exports`, чтобы их можно было найти и запустить. По умолчанию функция, которая выполняется при запуске, является единственным экземпляром экспорта (с именем `run` или `index`) из этого файла.

Такой экспорт можно настроить, используя `entryPoint` в файле `function.json`, как показано в следующем примере:

```json
{
  "entryPoint": "logFoo",
  "bindings": [
    ...
  ]
}
```

В среде Функций версии 2.x, которая поддерживает параметр `this` в пользовательских функциях, код этой самой функции может выглядеть следующим образом:

```javascript
class MyObj {
    constructor() {
        this.foo = 1;
    };

    logFoo(context) { 
        context.log("Foo is " + this.foo); 
        context.done(); 
    }
}

const myObj = new MyObj();
module.exports = myObj;
```

В этом примере важно отметить, что хотя экспортируемый объект не гарантирует сохранения состояния между выполнениями.

## <a name="local-debugging"></a>Локальная отладка

При запуске с `--inspect` параметром Node.js процесс прослушивает клиент отладки на указанном порту. В функциях Azure 2. x можно указать аргументы для передачи в процесс Node.js, который выполняет код, добавив переменную среды или параметр приложения `languageWorkers:node:arguments = <args>` . 

Чтобы выполнить отладку локально, добавьте `"languageWorkers:node:arguments": "--inspect=5858"` `Values` в [local.settings.js](./functions-run-local.md#local-settings-file) в файле и подключите отладчик к порту 5858.

При отладке с помощью VS Code `--inspect` параметр автоматически добавляется с использованием `port` значения в launch.jsфайла проекта.

В версии 1. x Настройка `languageWorkers:node:arguments` не будет работать. Порт отладки можно выбрать с помощью [`--nodeDebugPort`](./functions-run-local.md#start) параметра в Azure functions Core Tools.

## <a name="typescript"></a>TypeScript

Если вы используете версию 2. x среды выполнения функций, [функции Azure для Visual Studio Code](./create-first-function-cli-typescript.md) и [Azure functions Core Tools](functions-run-local.md) позволяют создавать приложения-функции с помощью шаблона, поддерживающего проекты приложения-функции TypeScript. Этот шаблон создает `package.json` файлы проекта и, облегчающие превращение `tsconfig.json` , запуск и публикацию функций JavaScript из кода TypeScript с помощью этих средств.

Созданный `.funcignore` файл используется для указания, какие файлы исключаются при публикации проекта в Azure.  

Файлы TypeScript (. TS) преобразуются в файлы JavaScript (JS) в `dist` выходном каталоге. Шаблоны TypeScript используют [ `scriptFile` параметр](#using-scriptfile) в, `function.json` чтобы указать расположение соответствующего JS файла в `dist` папке. Расположение выходных данных задается шаблоном с помощью `outDir` параметра в `tsconfig.json` файле. Если изменить этот параметр или имя папки, среда выполнения не сможет найти код для выполнения.

Способ, которым локально разрабатывается и развертывается в проекте TypeScript, зависит от вашего средства разработки.

### <a name="visual-studio-code"></a>Visual Studio Code

[Функция Azure для расширения Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) позволяет разрабатывать функции с помощью TypeScript. Основные средства являются обязательным требованием к расширению функций Azure.

Чтобы создать приложение-функцию TypeScript в Visual Studio Code, выберите `TypeScript` язык при создании приложения-функции.

При нажатии клавиши **F5** для локального запуска приложения транспилирования выполняется перед инициализацией узла (func.exe). 

При развертывании приложения-функции в Azure с помощью кнопки **развернуть в приложении-функцию.** расширение функций Azure сначала создает готовую к работе сборку файлов JavaScript из исходных файлов TypeScript.

### <a name="azure-functions-core-tools"></a>Azure Functions Core Tools

Существует несколько способов, которым проект TypeScript отличается от проекта JavaScript при использовании основных средств.

#### <a name="create-project"></a>Создание проекта

Чтобы создать проект приложения-функции TypeScript с помощью основных средств, необходимо указать параметр языка TypeScript при создании приложения-функции. Это можно сделать одним из следующих способов:

- Выполните `func init` команду, выберите в `node` качестве стека языка, а затем выберите `typescript` .

- Выполните команду `func init --worker-runtime typescript`.

#### <a name="run-local"></a>Запустить локально

Чтобы запустить код приложения функции локально с помощью основных средств, используйте следующие команды, а не `func host start` : 

```command
npm install
npm start
```

`npm start`Команда эквивалентна следующим командам:

- `npm run build`
- `func extensions install`
- `tsc`
- `func start`

#### <a name="publish-to-azure"></a>Публикация в Azure

Прежде чем использовать [`func azure functionapp publish`] команду для развертывания в Azure, создайте готовую к работе сборку файлов JavaScript из исходных файлов TypeScript. 

Следующие команды подготавливают и публикуют проект TypeScript с помощью основных средств: 

```command
npm run build:production 
func azure functionapp publish <APP_NAME>
```

В этой команде замените `<APP_NAME>` именем приложения функции.

## <a name="considerations-for-javascript-functions"></a>Рекомендации для функций JavaScript

При работе с функциями JavaScript следует помнить о рекомендациях, приведенных в следующих разделах.

### <a name="choose-single-vcpu-app-service-plans"></a>Выбор планов службы приложений для конфигурации с одним виртуальным ЦП

При создании приложения-функции, которое использует план службы приложения, мы советуем выбирать план для конфигурации с одним виртуальным ЦП вместо плана для нескольких виртуальных ЦП. Сейчас служба "Функции Azure" намного эффективнее выполняет функции JavaScript на виртуальных машинах с одним ЦП. Использование более крупных виртуальных машин не приводит к ожидаемому улучшению производительности. При необходимости можно вручную развернуть, добавив несколько экземпляров одновиртуальных цпных виртуальных машин или включив Автомасштабирование. Дополнительные сведения см. в статье [Масштабирование числа экземпляров вручную или автоматически](../azure-monitor/autoscale/autoscale-get-started.md?toc=/azure/app-service/toc.json).

### <a name="cold-start"></a>Холодный запуск

С разработкой Функций Azure в виде бессерверной модели размещения холодные запуски стали реальностью. *Холодный запуск* обозначает, что когда ваше приложение-функция запускается в первый раз после периода отсутствия активности, запуск займет больше времени. Для функций JavaScript, в частности с большими деревьями зависимостей, холодный запуск может оказаться достаточно длительным. Чтобы ускорить процесс холодного запуска, по возможности [выполняйте функции в виде файла пакета](run-functions-from-deployment-package.md). Многие методы развертывания используют эту модель по умолчанию, но если возникают большие временные задержки при холодном запуске и он не выполняется из файла пакета, запуск в виде файла пакета может значительно ускорить процесс.

### <a name="connection-limits"></a>Пределы подключения

При использовании зависящего от службы клиента в приложении "функции Azure" не создавайте новый клиент при каждом вызове функции. Вместо этого создайте в глобальной области один статический клиент. Дополнительные сведения см. [в статье Управление подключениями в функциях Azure](manage-connections.md).

### <a name="use-async-and-await"></a>Использование `async` и `await`

При написании функций Azure в JavaScript следует писать код с помощью `async` `await` ключевых слов и. Написание кода с помощью `async` и `await` вместо обратных вызовов или `.then` и `.catch` с помощью обещания помогает избежать двух распространенных проблем:
 - Создание неперехваченных исключений, которые приводили [к сбою Node.js процесса](https://nodejs.org/api/process.html#process_warning_using_uncaughtexception_correctly), потенциально влияя на выполнение других функций.
 - Непредвиденное поведение, например отсутствующие журналы из context. log, вызванные асинхронными вызовами, которые не ожидаются должным образом.

В приведенном ниже примере асинхронный метод `fs.readFile` вызывается с функцией обратного вызова с ошибками в качестве второго параметра. Этот код вызывает обе проблемы, упомянутые выше. Исключение, которое не было явно перехвачено в правильной области, завершило работу со сбоем всего процесса (#1 проблемы). Вызов `context.done()` за пределами области функции обратного вызова означает, что вызов функции может завершиться до того, как будет прочитан файл (выдача #2). В этом примере вызовет `context.done()` слишком ранние результаты в отсутствующих записях журнала, начиная с `Data from file:` .

```javascript
// NOT RECOMMENDED PATTERN
const fs = require('fs');

module.exports = function (context) {
    fs.readFile('./hello.txt', (err, data) => {
        if (err) {
            context.log.error('ERROR', err);
            // BUG #1: This will result in an uncaught exception that crashes the entire process
            throw err;
        }
        context.log(`Data from file: ${data}`);
        // context.done() should be called here
    });
    // BUG #2: Data is not guaranteed to be read before the Azure Function's invocation ends
    context.done();
}
```

Использование `async` `await` ключевых слов и помогает избежать обеих этих ошибок. [`util.promisify`](https://nodejs.org/api/util.html#util_util_promisify_original)Для включения функций в стиле обратного вызова с ошибками в ожидающие функции следует использовать функцию служебной программы Node.js.

В приведенном ниже примере все необработанные исключения, вызванные во время выполнения функции, не вызывают ошибку при отдельном вызове, вызвавшем исключение. `await`Ключевое слово означает, что следующие шаги `readFileAsync` выполняются только после `readFile` завершения. В `async` и `await` также не требуется вызывать `context.done()` обратный вызов.

```javascript
// Recommended pattern
const fs = require('fs');
const util = require('util');
const readFileAsync = util.promisify(fs.readFile);

module.exports = async function (context) {
    let data;
    try {
        data = await readFileAsync('./hello.txt');
    } catch (err) {
        context.log.error('ERROR', err);
        // This rethrown exception will be handled by the Functions Runtime and will only fail the individual invocation
        throw err;
    }
    context.log(`Data from file: ${data}`);
}
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в следующих ресурсах:

+ [Рекомендации по функциям Azure](functions-best-practices.md)
+ [Справочник разработчика по функциям Azure](functions-reference.md)
+ [Azure Functions triggers and bindings (Триггеры и привязки в Функциях Azure)](functions-triggers-bindings.md)

[' Func Azure functionapp Publish ']: functions-run-local.md#project-file-deployment
