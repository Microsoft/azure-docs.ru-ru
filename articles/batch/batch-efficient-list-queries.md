---
title: Создание эффективных запросов на вывод списка
description: 'Сведения о повышении производительности за счет фильтрации запросов, а именно при запросе сведений о ресурсах пакетной службы: пулах, заданиях, задачах и вычислительных узлах.'
ms.topic: how-to
ms.date: 06/18/2020
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: 3a767cc8ae3c8c48e1e40e0735c33fa807ba0015
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88933520"
---
# <a name="create-queries-to-list-batch-resources-efficiently"></a>Эффективное создание запросов на вывод списка ресурсов пакетной службы

Почти все приложения, использующие пакетную службу, должны выполнять определенный тип операций мониторинга или других операций, отправляющих запросы в пакетную службу (часто на регулярной основе). Например, чтобы определить, находятся ли в очереди задачи задания, необходимо получить данные о каждой задаче в задании. Чтобы определить состояние узлов в пуле, нужно получить данные о каждом из этих узлов. В этой статье объясняется, как выполнять такие запросы наиболее эффективно.

Вы можете увеличить производительность приложения пакетной службы Azure, уменьшив объем данных, возвращаемых службой, при запросе заданий, задач, вычислений и других ресурсов с помощью библиотеки [.NET для пакета](/dotnet/api/microsoft.azure.batch) .

> [!NOTE]
> Пакетная служба предоставляет поддержку API для распространенных сценариев выполнения задач в задании и для подсчета ресурсов вычислений в пуле пакетной службы. Вместо того чтобы использовать запрос на вывод списка, вы можете вызвать операции [Получить количество задач](/rest/api/batchservice/job/gettaskcounts) и [Вывести количество узлов в пуле](/rest/api/batchservice/account/listpoolnodecounts). Эти операции более эффективны, чем запросы списка, но возвращают более ограниченные сведения, которые могут не всегда быть актуальными. Дополнительные сведения см. в разделе [подсчет задач и вычисление узлов по состоянию](batch-get-resource-counts.md).

## <a name="specify-a-detail-level"></a>Укажите уровень детализации

В рабочем приложении пакетной службы такие сущности, как задания, задачи и вычислительные узлы, могут исчисляться тысячами. При запросе сведений об этих ресурсах потенциально большой объем данных нужно передать из пакетной службы в приложение по каждому запросу. Ограничение количества элементов и типов сведений, возвращаемых запросом, повышает скорость их обработки и, как следствие, производительность вашего приложения.

Этот фрагмент кода API [.NET пакетной службы](/dotnet/api/microsoft.azure.batch) перечисляет *все* задачи, связанные с заданием, а также выводит *полный набор* их свойств.

```csharp
// Get a collection of all of the tasks and all of their properties for job-001
IPagedEnumerable<CloudTask> allTasks =
    batchClient.JobOperations.ListTasks("job-001");
```

Однако можно выполнить более эффективный запрос списка, применив к запросу уровень детализации. Для этого необходимо передать объект [ODATADetailLevel](/dotnet/api/microsoft.azure.batch.odatadetaillevel) методу [JobOperations.ListTasks](/dotnet/api/microsoft.azure.batch.joboperations). Этот фрагмент кода возвращает только идентификатор, командную строку и информационные свойства вычислительного узла только для завершенных задач:

```csharp
// Configure an ODATADetailLevel specifying a subset of tasks and
// their properties to return
ODATADetailLevel detailLevel = new ODATADetailLevel();
detailLevel.FilterClause = "state eq 'completed'";
detailLevel.SelectClause = "id,commandLine,nodeInfo";

// Supply the ODATADetailLevel to the ListTasks method
IPagedEnumerable<CloudTask> completedTasks =
    batchClient.JobOperations.ListTasks("job-001", detailLevel);
```

Если, как в примере сценария, задание состоит из тысяч задач, результаты для второго запроса обычно возвращаются быстрее, чем для первого. [Ниже](#efficient-querying-in-batch-net)приведены дополнительные сведения об использовании ODATADetailLevel при получении списка элементов с помощью API пакетной службы для .NET.

> [!IMPORTANT]
> Мы настоятельно рекомендуем всегда передавать объект ODATADetailLevel в вызовы API .NET для получения списков. Это обеспечит максимальную эффективность и производительность приложения. Задание уровня детализации помогает снизить время ответа пакетной службы, повысить эффективность использования сети и минимизировать использование памяти клиентскими приложениями.

## <a name="filter-select-and-expand"></a>Фильтрация, выборка и развертывание

Интерфейсы API [.NET пакетной службы](/dotnet/api/microsoft.azure.batch) и [REST API пакетной службы](/rest/api/batchservice/) позволяют сократить количество возвращаемых в списке элементов и объем сведений, возвращаемых для каждого элемента. Для этого в запросе необходимо указать строки **фильтрации**, **выборки** и **развертывания**.

### <a name="filter"></a>Filter

Строка фильтра (filter) — это выражение, которое уменьшает число возвращаемых элементов. Например, можно вывести список только выполняемых задач для задания или только те из них, которые готовы к выполнению задач.

Строка фильтра состоит из одного или нескольких выражений, каждое из которых состоит из имени свойства, оператора и значения. Для каждого опрашиваемого типа сущности можно указать только определенные свойства, а также только определенные операторы, поддерживаемые каждым свойством.  Несколько выражений можно объединить с помощью логических операторов `and` и `or`.

В этом примере строка фильтра выводит только запущенные задачи отрисовки: `(state eq 'running') and startswith(id, 'renderTask')`.

### <a name="select"></a>Выберите пункт

Строка выборки (select) позволяет ограничить значения свойств, которые требуется вернуть для каждого элемента. Вы указываете список имен свойств, разделенных запятыми, и для элементов в результатах запроса возвращаются только значения этих свойств. Можно указать любое из свойств для запрашиваемого типа сущности.

В этом примере строка выборки указывает, что для каждой задачи необходимо вернуть значения только трех свойств: `id, state, stateTransitionTime`.

### <a name="expand"></a>Разверните

Строка развертывания сокращает количество вызовов API, необходимое для получения определенных сведений. При использовании строки развертывания дополнительные сведения о каждом элементе можно получить с помощью одного вызова API. Вместо того, чтобы сначала получить список сущностей, запросив сведения для каждого элемента в списке, используйте строку развертывания, чтобы получить одну и ту же информацию в одном вызове API, что способствует повышению производительности за счет сокращения вызовов API.

Как и строка выборки, строка развертывания определяет, какие данные нужно включить в результаты запроса списка. Если необходимы все свойства, и строка выборки не указана, для получения статистических данных *следует* использовать строку развертывания. Если для получения подмножества свойств используется строка выборки, то в ней можно указать `stats` и не указывать строку развертывания.

Строки развертывания поддерживаются только при использовании для получения списка заданий, расписаний заданий, задач и пулов. В настоящее время они поддерживают только статистические данные.

Далее приведен пример строки развертывания, которая предписывает, что для каждого элемента в списке необходимо вернуть статистические данные: `stats`.

> [!NOTE]
> При создании какого-либо из трех типов строк запроса (filter, select или expand) необходимо убедиться, что имена свойств и регистр совпадают с аналогичными элементами REST API. Например, при работе с классом [CloudTask](/dotnet/api/microsoft.azure.batch.cloudtask) в .NET необходимо указывать свойство **state**, а не **State**, несмотря на то что в .NET используется свойство [CloudTask.State](/dotnet/api/microsoft.azure.batch.cloudtask.state#Microsoft_Azure_Batch_CloudTask_State). Сопоставление свойств API для .NET и REST см. в таблице ниже.

### <a name="rules-for-filter-select-and-expand-strings"></a>Правила строк фильтрации, выборки и развертывания

- Имена свойств в строках фильтров, SELECT и Expand должны отображаться так же, как и в API-интерфейсе [пакетной](/rest/api/batchservice/) службы, даже при использовании [.NET](/dotnet/api/microsoft.azure.batch) или одного из других пакетов SDK пакетной службы.
- Имена всех свойств чувствительны к регистру, но для значений свойств регистр значения не имеет.
- Строки даты и времени могут иметь один из двух форматов и должны начинаться с `DateTime`.
  
  - Пример формата W3C-DTF: `creationTime gt DateTime'2011-05-08T08:49:37Z'`
  - Пример формата RFC 1123: `creationTime gt DateTime'Sun, 08 May 2011 08:49:37 GMT'`
- Строки с логическими выражениями должны иметь значение `true` или `false`.
- Если задано недопустимое свойство или оператор, отобразится ошибка `400 (Bad Request)` .

## <a name="efficient-querying-in-batch-net"></a>Эффективное выполнение запросов в пакетной службе для .NET

В API [.NET пакетной службы](/dotnet/api/microsoft.azure.batch) для указания строк фильтрации, выборки и развертывания операций списка используется класс [ODATADetailLevel](/dotnet/api/microsoft.azure.batch.odatadetaillevel). Объект ODataDetailLevel имеет три открытых строковых свойства, которые можно указать в конструкторе или установить непосредственно в объекте. Затем объект ODataDetailLevel передается в качестве параметра в различные операции получения списка, такие как [ListPools](/dotnet/api/microsoft.azure.batch.pooloperations), [ListJobs](/dotnet/api/microsoft.azure.batch.joboperations) и [ListTasks](/dotnet/api/microsoft.azure.batch.joboperations).

- [ODATADetailLevel. FilterClause](/dotnet/api/microsoft.azure.batch.odatadetaillevel.filterclause): ограничение числа возвращаемых элементов.
- [ODATADetailLevel. SelectClause](/dotnet/api/microsoft.azure.batch.odatadetaillevel.selectclause): укажите, какие значения свойств возвращаются для каждого элемента.
- [ODATADetailLevel. ExpandClause](/dotnet/api/microsoft.azure.batch.odatadetaillevel.expandclause): получение данных для всех элементов в одном вызове API вместо отдельных вызовов для каждого элемента.

Ниже приведен фрагмент кода, который с помощью API пакетной службы для .NET отправляет эффективный запрос пакетной службе для получения статистики по определенному набору пулов. В этом сценарии у пользователя пакетной службы есть как тестовые, так и рабочие пулы. Идентификаторы тестовых пулов имеют префикс "test", а рабочих пулов — "prod". В приведенном фрагменте *myBatchClient* является правильно инициализированным экземпляром класса [BatchClient](/dotnet/api/microsoft.azure.batch.batchclient) .

```csharp
// First we need an ODATADetailLevel instance on which to set the filter, select,
// and expand clause strings
ODATADetailLevel detailLevel = new ODATADetailLevel();

// We want to pull only the "test" pools, so we limit the number of items returned
// by using a FilterClause and specifying that the pool IDs must start with "test"
detailLevel.FilterClause = "startswith(id, 'test')";

// To further limit the data that crosses the wire, configure the SelectClause to
// limit the properties that are returned on each CloudPool object to only
// CloudPool.Id and CloudPool.Statistics
detailLevel.SelectClause = "id, stats";

// Specify the ExpandClause so that the .NET API pulls the statistics for the
// CloudPools in a single underlying REST API call. Note that we use the pool's
// REST API element name "stats" here as opposed to "Statistics" as it appears in
// the .NET API (CloudPool.Statistics)
detailLevel.ExpandClause = "stats";

// Now get our collection of pools, minimizing the amount of data that is returned
// by specifying the detail level that we configured above
List<CloudPool> testPools =
    await myBatchClient.PoolOperations.ListPools(detailLevel).ToListAsync();
```

> [!TIP]
> Экземпляр [ODATADetailLevel](/dotnet/api/microsoft.azure.batch.odatadetaillevel) с предложениями Select и Expand также можно передать в соответствующие методы Get, чтобы ограничить объем возвращаемых данных, например в метод [PoolOperations.GetPool](/dotnet/api/microsoft.azure.batch.pooloperations.getpool#Microsoft_Azure_Batch_PoolOperations_GetPool_System_String_Microsoft_Azure_Batch_DetailLevel_System_Collections_Generic_IEnumerable_Microsoft_Azure_Batch_BatchClientBehavior__).

## <a name="batch-rest-to-net-api-mappings"></a>Сопоставление API REST и .NET пакетной службы

Имена и регистр свойств в строках фильтрации, выборки и развертывания должны полностью соответствовать аналогичным элементам в REST API. В таблицах ниже приведено сопоставление элементов API в .NET и REST.

### <a name="mappings-for-filter-strings"></a>Сопоставления для строк фильтрации

- **Методы списка .NET**: каждому из .NET-методов API в этом столбце объект [ODATADetailLevel](/dotnet/api/microsoft.azure.batch.odatadetaillevel) передается в качестве параметра.
- **Запросы списка REST**: каждая страница REST API, связанная с этим столбцом, содержит таблицу со свойствами и операциями, разрешенными в строках *фильтрации*. Эти имена свойств и операции используются при создании строки [ODATADetailLevel. FilterClause](/dotnet/api/microsoft.azure.batch.odatadetaillevel.filterclause) .

| Методы списка .NET | Запросы списка REST |
| --- | --- |
| [CertificateOperations.ListCertificates](/dotnet/api/microsoft.azure.batch.certificateoperations) |[Получение списка сертификатов в учетной записи](/rest/api/batchservice/certificate/list) |
| [CloudTask.ListNodeFiles](/dotnet/api/microsoft.azure.batch.cloudtask) |[Получение списка файлов, связанных с задачей](/rest/api/batchservice/file/listfromtask) |
| [JobOperations.ListJobPreparationAndReleaseTaskStatus](/dotnet/api/microsoft.azure.batch.joboperations) |[Получение состояния подготовки задания и задачи деблокирования задания](/rest/api/batchservice/job/listpreparationandreleasetaskstatus) |
| [JobOperations.ListJobs](/dotnet/api/microsoft.azure.batch.joboperations) |[Получение списка заданий в учетной записи](/rest/api/batchservice/job/list) |
| [JobOperations.ListNodeFiles](/dotnet/api/microsoft.azure.batch.joboperations) |[Получение списка файлов в узле](/rest/api/batchservice/file/listfromcomputenode) |
| [JobOperations.ListTasks](/dotnet/api/microsoft.azure.batch.joboperations) |[Получение списка задач, связанных с заданием](/rest/api/batchservice/task/list) |
| [JobScheduleOperations.ListJobSchedules](/dotnet/api/microsoft.azure.batch.jobscheduleoperations) |[Получение расписаний для заданий в учетной записи](/rest/api/batchservice/jobschedule/list) |
| [JobScheduleOperations.ListJobs](/dotnet/api/microsoft.azure.batch.jobscheduleoperations) |[Получение списка заданий, связанных с расписанием задания](/rest/api/batchservice/job/listfromjobschedule) |
| [PoolOperations.ListComputeNodes](/dotnet/api/microsoft.azure.batch.pooloperations) |[Получение списка вычислительных узлов в пуле](/rest/api/batchservice/computenode/list) |
| [PoolOperations.ListPools](/dotnet/api/microsoft.azure.batch.pooloperations) |[Получение списка пулов учетной записи](/rest/api/batchservice/pool/list) |

### <a name="mappings-for-select-strings"></a>Сопоставления для строк выборки

- **Типы пакетной службы для .NET**: типы API пакетной службы для .NET.
- **Сущности REST API**: каждая страница в этом столбце содержит одну таблицу или несколько с именами свойств API REST для соответствующих типов. Эти имена свойств используются при создании строк *выборки* . При создании строки [ODATADetailLevel. SelectClause](/dotnet/api/microsoft.azure.batch.odatadetaillevel.selectclause) используются те же имена свойств.

| Типы пакетной службы для .NET | Сущности REST API |
| --- | --- |
| [Сертификат](/dotnet/api/microsoft.azure.batch.certificate) |[Получение информации о сертификате](/rest/api/batchservice/certificate/get) |
| [CloudJob](/dotnet/api/microsoft.azure.batch.cloudjob) |[Получение информации о задании](/rest/api/batchservice/job/get) |
| [CloudJobSchedule](/dotnet/api/microsoft.azure.batch.cloudjobschedule) |[Получение информации о расписании задания](/rest/api/batchservice/jobschedule/get) |
| [ComputeNode](/dotnet/api/microsoft.azure.batch.computenode) |[Получение информации об узле](/rest/api/batchservice/computenode/get) |
| [CloudPool](/dotnet/api/microsoft.azure.batch.cloudpool) |[Получение информации о пуле](/rest/api/batchservice/pool/get) |
| [CloudTask](/dotnet/api/microsoft.azure.batch.cloudtask) |[Получение информации о задаче](/rest/api/batchservice/task/get) |

## <a name="example-construct-a-filter-string"></a>Пример. Создание строки фильтрации

При создании строки фильтрации для [ODATADetailLevel.FilterClause](/dotnet/api/microsoft.azure.batch.odatadetaillevel.filterclause) обратитесь к таблице, приведенной выше в разделе "Сопоставления для строк фильтрации", чтобы найти страницу документации по REST API, соответствующую операции списка, которую вам необходимо выполнить. В первой многострочной таблице на этой странице содержатся фильтруемые свойства и соответствующие поддерживаемые ими операторы. К примеру, вы хотите получить все задачи с ненулевым кодом возврата. В строке [Вывод задач, связанных с заданием](/rest/api/batchservice/task/list) указана строка с соответствующим параметром и допустимые операторы.

| Свойство | Разрешенные операции | Тип |
|:--- |:--- |:--- |
| `executionInfo/exitCode` |`eq, ge, gt, le , lt` |`Int` |

Соответственно, строка фильтрации для отображения списка всех задач с ненулевым кодом возврата будет выглядеть так:

`(executionInfo/exitCode lt 0) or (executionInfo/exitCode gt 0)`

## <a name="example-construct-a-select-string"></a>Пример. Создание строки выборки

Для создания [ODATADetailLevel.SelectClause](/dotnet/api/microsoft.azure.batch.odatadetaillevel.selectclause) обратитесь к приведенной выше таблице в разделе "Сопоставления для строк выборки" и найдите страницу документации по REST API, которая соответствует типу выводимой сущности. В первой многострочной таблице на этой странице содержатся выбираемые свойства и соответствующие поддерживаемые ими операторы. Например, если вы хотите получить только идентификатор и командную строку для каждой задачи в списке. Эти строки вы найдете в соответствующей таблице [Получение информации о задаче](/rest/api/batchservice/task/get).

| Свойство | Type | Примечания |
|:--- |:--- |:--- |
| `id` |`String` |`The ID of the task.` |
| `commandLine` |`String` |`The command line of the task.` |

Строка выборки для отображения только идентификатора и командной строки для каждой задачи в списке будет выглядеть так:

`id, commandLine`

## <a name="code-samples"></a>Примеры кода

### <a name="efficient-list-queries-code-sample"></a>Пример кода с эффективными запросами списков

Пример проекта [EfficientListQueries](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp/ArticleProjects/EfficientListQueries) на сайте GitHub показывает, насколько эффективный запрос списка может повлиять на производительность приложения. Это консольное приложение C# создает и добавляет большое количество задач в задание. Затем оно несколько раз вызывает метод [JobOperations.ListTasks](/dotnet/api/microsoft.azure.batch.joboperations) и передает в него объекты [ODATADetailLevel](/dotnet/api/microsoft.azure.batch.odatadetaillevel), в которых заданы различные значения свойств для изменения объема возвращаемых данных. Вывод приложения должен быть похож на приведенный ниже:

```
Adding 5000 tasks to job jobEffQuery...
5000 tasks added in 00:00:47.3467587, hit ENTER to query tasks...

4943 tasks retrieved in 00:00:04.3408081 (ExpandClause:  | FilterClause: state eq 'active' | SelectClause: id,state)
0 tasks retrieved in 00:00:00.2662920 (ExpandClause:  | FilterClause: state eq 'running' | SelectClause: id,state)
59 tasks retrieved in 00:00:00.3337760 (ExpandClause:  | FilterClause: state eq 'completed' | SelectClause: id,state)
5000 tasks retrieved in 00:00:04.1429881 (ExpandClause:  | FilterClause:  | SelectClause: id,state)
5000 tasks retrieved in 00:00:15.1016127 (ExpandClause:  | FilterClause:  | SelectClause: id,state,environmentSettings)
5000 tasks retrieved in 00:00:17.0548145 (ExpandClause: stats | FilterClause:  | SelectClause: )

Sample complete, hit ENTER to continue...
```

Как показано в сведениях о затраченном времени, путем ограничения свойств и количества возвращаемых элементов можно значительно уменьшить время отклика на запрос. Этот и другие примеры проектов находятся в репозитории [azure-batch-samples](https://github.com/Azure-Samples/azure-batch-samples) на сервисе GitHub.

### <a name="batchmetrics-library-and-code-sample"></a>Библиотека BatchMetrics и пример кода

Помимо примера кода EfficientListQueries, приведенного выше, пример проекта [BatchMetrics](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp/BatchMetrics) демонстрирует, как эффективно отслеживать ход выполнения заданий пакетной службы Azure с помощью API пакетной службы.

Пример [BatchMetrics](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp/BatchMetrics) включает в себя проект библиотеки классов .NET, который вы можете использовать в собственных проектах, а также простую программу командной строки для проверки и демонстрации использования этой библиотеки.

В примере приложения проекта демонстрируются следующие операции:

1. Выбор определенных атрибутов для скачивания только необходимых свойств.
2. Фильтрация времени смены состояния для скачивания изменений, отправленных с момента последнего запроса.

Например, в библиотеке BatchMetrics используется приведенный ниже метод. Он возвращает объект ODATADetailLevel, указывающий, что для сущностей, к которым обращен запрос, нужно получить только свойства `id` и `state`. Он также указывает, что должны быть возвращены только те сущности, состояние которых изменилось с момента, указанного в значении параметра `DateTime` .

```csharp
internal static ODATADetailLevel OnlyChangedAfter(DateTime time)
{
    return new ODATADetailLevel(
        selectClause: "id, state",
        filterClause: string.Format("stateTransitionTime gt DateTime'{0:o}'", time)
    );
}
```

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [максимально увеличить использование ресурсов пакетной службы Azure с параллельными задачами узла](batch-parallel-node-tasks.md). Некоторые типы рабочих нагрузок могут использовать преимущества параллельных задач на больших (но меньшем) масштабных узлах. Дополнительные сведения о таком сценарии см. [здесь](batch-parallel-node-tasks.md#example-scenario).
- Узнайте, как [отслеживать решения пакетной службы путем инвентаризации задач и узлов по состоянию](batch-get-resource-counts.md) .


[api_net]: /dotnet/api/microsoft.azure.batch
[api_net_listjobs]: /dotnet/api/microsoft.azure.batch.joboperations
[api_rest]: /rest/api/batchservice/
[batch_metrics]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchMetrics
[efficient_query_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/EfficientListQueries
[github_samples]: https://github.com/Azure/azure-batch-samples
[odata]: /dotnet/api/microsoft.azure.batch.odatadetaillevel
[odata_ctor]: /dotnet/api/microsoft.azure.batch.odatadetaillevel
[odata_expand]: /dotnet/api/microsoft.azure.batch.odatadetaillevel
[odata_filter]: /dotnet/api/microsoft.azure.batch.odatadetaillevel
[odata_select]: /dotnet/api/microsoft.azure.batch.odatadetaillevel

[net_list_certs]: /dotnet/api/microsoft.azure.batch.certificateoperations
[net_list_compute_nodes]: /dotnet/api/microsoft.azure.batch.pooloperations
[net_list_job_schedules]: /dotnet/api/microsoft.azure.batch.jobscheduleoperations
[net_list_jobprep_status]: /dotnet/api/microsoft.azure.batch.joboperations
[net_list_jobs]: /dotnet/api/microsoft.azure.batch.joboperations
[net_list_nodefiles]: /dotnet/api/microsoft.azure.batch.joboperations
[net_list_pools]: /dotnet/api/microsoft.azure.batch.pooloperations
[net_list_schedule_jobs]: /dotnet/api/microsoft.azure.batch.jobscheduleoperations
[net_list_task_files]: /dotnet/api/microsoft.azure.batch.cloudtask
[net_list_tasks]: /dotnet/api/microsoft.azure.batch.joboperations

[rest_list_certs]: /rest/api/batchservice/certificate/list
[rest_list_compute_nodes]: /rest/api/batchservice/computenode/list
[rest_list_job_schedules]: /rest/api/batchservice/jobschedule/list
[rest_list_jobprep_status]: /rest/api/batchservice/job/listpreparationandreleasetaskstatus
[rest_list_jobs]: /rest/api/batchservice/job/list
[rest_list_nodefiles]: /rest/api/batchservice/file/listfromcomputenode
[rest_list_pools]: /rest/api/batchservice/pool/list
[rest_list_schedule_jobs]: /rest/api/batchservice/job/listfromjobschedule
[rest_list_task_files]: /rest/api/batchservice/file/listfromtask
[rest_list_tasks]: /rest/api/batchservice/task/list

[rest_get_cert]: /rest/api/batchservice/certificate/get
[rest_get_job]: /rest/api/batchservice/job/get
[rest_get_node]: /rest/api/batchservice/computenode/get
[rest_get_pool]: /rest/api/batchservice/pool/get
[rest_get_schedule]: /rest/api/batchservice/jobschedule/get
[rest_get_task]: /rest/api/batchservice/task/get

[net_cert]: /dotnet/api/microsoft.azure.batch.certificate
[net_job]: /dotnet/api/microsoft.azure.batch.cloudjob
[net_node]: /dotnet/api/microsoft.azure.batch.computenode
[net_pool]: /dotnet/api/microsoft.azure.batch.cloudpool
[net_schedule]: /dotnet/api/microsoft.azure.batch.cloudjobschedule
[net_task]: /dotnet/api/microsoft.azure.batch.cloudtask

[rest_get_task_counts]: /rest/api/batchservice/get-the-task-counts-for-a-job
[rest_get_node_counts]: /rest/api/batchservice/account/listpoolnodecounts
