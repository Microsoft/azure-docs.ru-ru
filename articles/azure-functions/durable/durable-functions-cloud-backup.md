---
title: Сценарии развертывания и объединения в устойчивых функциях — Azure
description: Сведения о том, как реализовать сценарии развертывания и объединения в расширении устойчивых функций для Функций Azure.
ms.topic: conceptual
ms.date: 11/02/2019
ms.author: azfuncdf
ms.openlocfilehash: 91128033696af6a56488db7991987f1e384b719e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98027663"
---
# <a name="fan-outfan-in-scenario-in-durable-functions---cloud-backup-example"></a>Сценарии развертывания и объединения в устойчивых функциях. Пример резервного копирования в облако

*Развертывание и объединение* — это шаблон параллельного выполнения нескольких функций с последующим статистическим вычислением результатов. В этой статье описывается пример, использующий [устойчивые функции](durable-functions-overview.md) для реализации сценариев развертывания и объединения. Образец представляет устойчивую функцию, создающую резервную копию всего или некоторого содержимого сайта приложения в службе хранилища Azure.

[!INCLUDE [durable-functions-prerequisites](../../../includes/durable-functions-prerequisites.md)]

## <a name="scenario-overview"></a>Общие сведения о сценарии

В этом образце функции рекурсивно отправляют все файлы в указанном каталоге в хранилище BLOB-объектов и подсчитывают общее число отправленных байтов.

Можно написать одну функцию, которая отвечает за все. Основная проблема, которая может возникнуть, связана с выполнением **масштабируемости**. Выполнение одной функции может выполняться только на одной виртуальной машине, поэтому пропускная способность будет ограничена пропускной способностью одной виртуальной машины. Следующая проблема заключается в **надежности**. В случае сбоя в посередине или если весь процесс занимает более 5 минут, резервное копирование может завершиться сбоем в частично завершенном состоянии. В таком случае его необходимо начать сначала.

Более надежным подходом было бы записать две обычные функции: одна для перечисления файлов и добавления их имен в очередь, а другая для чтения из очереди и отправки файлов в хранилище BLOB-объектов. Этот подход лучше в плане пропускной способности и надежности, но он требует подготавливать очередь и управлять ею. Если требуется выполнить какое-либо действие (например, сообщить об общем количестве отправленных байтов), могут возникнуть значительные проблемы в контексте **управления состоянием** и **координацией**.

Подход к устойчивым функциям обеспечивает получение всех упомянутых преимуществ с очень низкими издержками.

## <a name="the-functions"></a>Функции

В этой статье описаны следующие функции в примере приложения:

* `E2_BackupSiteContent`: [Функция Orchestrator](durable-functions-bindings.md#orchestration-trigger) , которая вызывает `E2_GetFileList` для получения списка файлов для резервного копирования, а затем вызывает `E2_CopyFileToBlob` для резервного копирования каждого файла.
* `E2_GetFileList`: [Функция действия](durable-functions-bindings.md#activity-trigger) , возвращающая список файлов в каталоге.
* `E2_CopyFileToBlob`: Функция действия, которая создает резервную копию одного файла в хранилище BLOB-объектов Azure.

### <a name="e2_backupsitecontent-orchestrator-function"></a>E2_BackupSiteContent Orchestrator, функция

Эта функция оркестратора выполняет следующие задачи:

1. Принимает значение `rootDirectory` в качестве входного параметра.
2. Вызывает функцию для получения рекурсивного списка файлов в `rootDirectory`.
3. Выполняет несколько параллельных вызовов функции для отправки каждого файла в хранилище BLOB-объектов Azure.
4. Ожидает завершения всех передач.
5. Возвращает сумму общего количества байтов, отправленных в хранилище BLOB-объектов Azure.

# <a name="c"></a>[C#](#tab/csharp)

Ниже приведен код, реализующий функцию оркестратора.

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/BackupSiteContent.cs?range=16-42)]

Обратите внимание на строку `await Task.WhenAll(tasks);`. Все отдельные вызовы `E2_CopyFileToBlob` функции *не* ожидают выполнения, что позволяет выполнять их параллельно. При передаче массива задач в `Task.WhenAll` мы получаем задачу, которая не выполнится *до завершения всех операций копирования*. Если вы знакомы с библиотекой параллельных задач (TPL) в .NET, то вы уже знаете об этом. Разница заключается в том, что эти задачи могут выполняться одновременно на нескольких виртуальных машинах, а расширение Устойчивые функции гарантирует устойчивость сквозного выполнения к перезапуску процесса.

Если задача `Task.WhenAll` завершена, это говорит о том, что все вызовы функций завершены и значения возвращены. Каждый вызов `E2_CopyFileToBlob` возвращает число переданных байтов, поэтому, чтобы узнать сумму всех байтов, нужно сложить все возвращаемые значения.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Функция использует стандартный *function.js* для функций Orchestrator.

[!code-json[Main](~/samples-durable-functions/samples/javascript/E2_BackupSiteContent/function.json)]

Ниже приведен код, реализующий функцию оркестратора.

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E2_BackupSiteContent/index.js)]

Обратите внимание на строку `yield context.df.Task.all(tasks);`. Все отдельные вызовы `E2_CopyFileToBlob` функции *не* были получены, что позволяет выполнять их параллельно. При передаче массива задач в `context.df.Task.all` мы получаем задачу, которая не выполнится *до завершения всех операций копирования*. Если вы знакомы с [`Promise.all`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all) JavaScript, это не новая возможность. Разница заключается в том, что эти задачи могут выполняться одновременно на нескольких виртуальных машинах, а расширение Устойчивые функции гарантирует устойчивость сквозного выполнения к перезапуску процесса.

> [!NOTE]
> Несмотря на то что задачи концептуально похожи на обещания JavaScript, функции оркестратора должны использовать `context.df.Task.all` и `context.df.Task.any` вместо `Promise.all` и `Promise.race` для управления параллелизацией задач.

После получения от `context.df.Task.all` , мы понимаем, что все вызовы функций завершены и возвращали значения обратно в нас. Каждый вызов `E2_CopyFileToBlob` возвращает число переданных байтов, поэтому, чтобы узнать сумму всех байтов, нужно сложить все возвращаемые значения.

# <a name="python"></a>[Python](#tab/python)

Функция использует стандартный *function.js* для функций Orchestrator.

[!code-json[Main](~/samples-durable-functions-python/samples/fan_in_fan_out/E2_BackupSiteContent/function.json)]

Ниже приведен код, реализующий функцию оркестратора.

[!code-python[Main](~/samples-durable-functions-python/samples/fan_in_fan_out/E2_BackupSiteContent/\_\_init\_\_.py)]

Обратите внимание на строку `yield context.task_all(tasks);`. Все отдельные вызовы `E2_CopyFileToBlob` функции *не* были получены, что позволяет выполнять их параллельно. При передаче массива задач в `context.task_all` мы получаем задачу, которая не выполнится *до завершения всех операций копирования*. Если вы знакомы с [`asyncio.gather`](https://docs.python.org/3/library/asyncio-task.html#asyncio.gather) Python, это не новая возможность. Разница заключается в том, что эти задачи могут выполняться одновременно на нескольких виртуальных машинах, а расширение Устойчивые функции гарантирует устойчивость сквозного выполнения к перезапуску процесса.

> [!NOTE]
> Хотя задачи концептуально похожи на Python аваитаблес, функции Orchestrator должны использовать, а также `yield` API- `context.task_all` `context.task_any` интерфейсы и для управления параллелизации задач.

После получения от `context.task_all` , мы понимаем, что все вызовы функций завершены и возвращали значения обратно в нас. Каждый вызов `E2_CopyFileToBlob` возвращает число переданных байтов, поэтому мы можем вычислить общую сумму байтов, добавив все возвращаемые значения.

---

### <a name="helper-activity-functions"></a>Вспомогательные функции действий

Вспомогательные функции действий, как и другие примеры, являются обычными функциями, которые используют привязку триггера `activityTrigger`.

#### <a name="e2_getfilelist-activity-function"></a>Функция действия E2_GetFileList

# <a name="c"></a>[C#](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/BackupSiteContent.cs?range=44-54)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

*function.jsв* файле для `E2_GetFileList` выглядит следующим образом:

[!code-json[Main](~/samples-durable-functions/samples/javascript/E2_GetFileList/function.json)]

Вот сама реализация:

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E2_GetFileList/index.js)]

Функция использует `readdirp` модуль (версия 2. x) для рекурсивного считывания структуры каталогов.

# <a name="python"></a>[Python](#tab/python)

*function.jsв* файле для `E2_GetFileList` выглядит следующим образом:

[!code-json[Main](~/samples-durable-functions-python/samples/fan_in_fan_out/E2_GetFileList/function.json)]

Вот сама реализация:

[!code-python[Main](~/samples-durable-functions-python/samples/fan_in_fan_out/E2_GetFileList/\_\_init\_\_.py)]

---

> [!NOTE]
> Вы можете спросить, почему нельзя просто поместить этот код непосредственно в функцию оркестратора. Это действие может нарушить одно из основных правил функций оркестратора — они не должны выполнять операции ввода-вывода, включая доступ к локальной файловой системе. Дополнительные сведения см. в разделе [ограничения кода функции Orchestrator](durable-functions-code-constraints.md).

#### <a name="e2_copyfiletoblob-activity-function"></a>Функция действия E2_CopyFileToBlob

# <a name="c"></a>[C#](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/BackupSiteContent.cs?range=56-81)]

> [!NOTE]
> Для `Microsoft.Azure.WebJobs.Extensions.Storage` запуска примера кода необходимо установить пакет NuGet.

Функция использует некоторые дополнительные возможности привязок функций Azure (то есть использование [ `Binder` параметра](../functions-dotnet-class-library.md#binding-at-runtime)), но не нужно беспокоиться об этих деталях в целях этого пошагового руководства.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Файл *unction.json* для `E2_CopyFileToBlob` такой же простой:

[!code-json[Main](~/samples-durable-functions/samples/javascript/E2_CopyFileToBlob/function.json)]

Реализация JavaScript использует [пакет SDK службы хранилища Azure для Node](https://github.com/Azure/azure-storage-node) для отправки файлов в хранилище BLOB-объектов Azure.

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E2_CopyFileToBlob/index.js)]

# <a name="python"></a>[Python](#tab/python)

Файл *unction.json* для `E2_CopyFileToBlob` такой же простой:

[!code-json[Main](~/samples-durable-functions-python/samples/fan_in_fan_out/E2_CopyFileToBlob/function.json)]

Реализация Python использует [пакет SDK службы хранилища Azure для Python](https://github.com/Azure/azure-storage-python) для передачи файлов в хранилище BLOB-объектов Azure.

[!code-python[Main](~/samples-durable-functions-python/samples/fan_in_fan_out/E2_CopyFileToBlob/\_\_init\_\_.py)]

---

Реализация загружает файл с диска и асинхронно выполняет потоковую передачу содержимого в одноименный большой двоичный объект в контейнере резервных копий. В результате возвращается количество байтов, скопированных в хранилище, которое затем используется функцией оркестратора для вычисления общей суммы.

> [!NOTE]
> Это отличный пример перемещения операций ввода-вывода в функцию `activityTrigger`. Можно не только распределить работу между различными компьютерами, но и получить преимущества от создания контрольных точек. Если хост-процесс по какой-либо причине завершится, вы будете знать, какие передачи уже выполнены.

## <a name="run-the-sample"></a>Запуск примера

Вы можете запустить orchestration в Windows, отправив следующий запрос HTTP POST.

```
POST http://{host}/orchestrators/E2_BackupSiteContent
Content-Type: application/json
Content-Length: 20

"D:\\home\\LogFiles"
```

Кроме того, на приложение-функция Linux (в настоящее время Python работает только в Linux для службы приложений) можно запустить оркестрации следующим образом:

```
POST http://{host}/orchestrators/E2_BackupSiteContent
Content-Type: application/json
Content-Length: 20

"/home/site/wwwroot"
```

> [!NOTE]
> Вызываемая функция `HttpStart` работает только с содержимым в формате JSON. По этой причине требуется заголовок `Content-Type: application/json` и путь к каталогу кодируется в виде строки JSON. Кроме того, в предыдущем фрагменте HTTP предполагается, что в файле `host.json` существует запись, которая позволяет удалить префикс `api/` по умолчанию из всех URL-адресов функций для триггеров HTTP. Разметку для этой конфигурации можно найти в файле `host.json` в примерах.

Этот запрос HTTP активирует оркестратор `E2_BackupSiteContent` и передает строку `D:\home\LogFiles` в качестве параметра. В ответе содержится ссылка на получение информации о состоянии операции резервного копирования:

```
HTTP/1.1 202 Accepted
Content-Length: 719
Content-Type: application/json; charset=utf-8
Location: http://{host}/runtime/webhooks/durabletask/instances/b4e9bdcc435d460f8dc008115ff0a8a9?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}

(...trimmed...)
```

В зависимости от количества файлов журнала в приложении-функции, эта операция может занять несколько минут. Информацию об актуальном состоянии можно получить, отправив запрос на URL-адрес в заголовке `Location` предыдущего ответа HTTP 202.

```
GET http://{host}/runtime/webhooks/durabletask/instances/b4e9bdcc435d460f8dc008115ff0a8a9?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}
```

```
HTTP/1.1 202 Accepted
Content-Length: 148
Content-Type: application/json; charset=utf-8
Location: http://{host}/runtime/webhooks/durabletask/instances/b4e9bdcc435d460f8dc008115ff0a8a9?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}

{"runtimeStatus":"Running","input":"D:\\home\\LogFiles","output":null,"createdTime":"2019-06-29T18:50:55Z","lastUpdatedTime":"2019-06-29T18:51:16Z"}
```

В этом случае функция все еще выполняется. Вы можете просмотреть входные данные, сохраненные в состоянии оркестратора, и последнее время обновления. Для опроса состояния завершения операции можно продолжать использовать значения заголовка `Location`. Если состояние — "Завершено", вы увидите HTTP-ответ, как в примере ниже:

```
HTTP/1.1 200 OK
Content-Length: 152
Content-Type: application/json; charset=utf-8

{"runtimeStatus":"Completed","input":"D:\\home\\LogFiles","output":452071,"createdTime":"2019-06-29T18:50:55Z","lastUpdatedTime":"2019-06-29T18:51:26Z"}
```

Теперь оркестрация завершена, и вы можете оценить, сколько времени ушло на ее выполнение. Вы также увидите значение поля `output`, в котором указано, что отправлено примерно 450 КБ данных журналов.

## <a name="next-steps"></a>Дальнейшие действия

В этом примере показано, как реализовать шаблон развертывания и объединения. В следующем примере показано, как реализовать шаблон мониторинга с помощью [устойчивых таймеров](durable-functions-timers.md).

> [!div class="nextstepaction"]
> [Запуск примера мониторинга](durable-functions-monitor.md)