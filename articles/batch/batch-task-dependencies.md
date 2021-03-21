---
title: Создание зависимостей задач для выполнения задач
description: Создание задач, которые зависят от выполнения других задач, для обработки по модели MapReduce и аналогичных рабочих нагрузок больших данных в пакетной службе Azure.
ms.topic: how-to
ms.date: 12/28/2020
ms.custom: H1Hack27Feb2017, devx-track-csharp
ms.openlocfilehash: ef05a98fffc3c0684ad0fa29f2f9f039b388f5ad
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97803941"
---
# <a name="create-task-dependencies-to-run-tasks-that-depend-on-other-tasks"></a>Создание зависимостей для выполнения задач, которые зависят от других задач

С помощью зависимостей задач пакетной службы вы создаете задачи, запланированные к выполнению на вычислительных узлах после завершения одной или нескольких родительских задач. Например, можно создать задание, которое отрисовывает каждый кадр трехмерного фильма с помощью отдельных параллельных задач. Конечная задача, "задача слияния", объединяет отрисованные кадры в готовый фильм только после успешной отрисовки всех кадров.

Ниже описано несколько ситуаций, в которых удобно использовать зависимости задач. Их можно использовать для:

- рабочих нагрузок MapReduce в облаке;
- заданий, задачи обработки данных которых можно представить в виде направленного ациклического графа (DAG);
- подготовительных и последующих процессов отрисовки, в которых каждая задача должна быть выполнена прежде, чем начнется выполнение следующей задачи;
- любых других заданий, в которых подчиненные задачи зависят от выходных данных вышестоящих задач.

По умолчанию выполнение зависимых задач планируются только после успешного завершения родительской задачи. При необходимости можно указать [действие зависимости](#dependency-actions)  , которое будет переопределять поведение по умолчанию и выполнять задачи при сбое родительской задачи.

## <a name="task-dependencies-with-batch-net"></a>Зависимости задач .NET пакетной службы

В этой статье рассматривается настройка зависимостей задач с помощью библиотеки [.NET пакетной службы](/dotnet/api/microsoft.azure.batch). Сначала в этой статье описывается, как [включить зависимость задачи](#enable-task-dependencies) в заданиях, а потом рассматривается, как [настроить задачу с зависимостями](#create-dependent-tasks). Мы также опишем, как указать действие зависимости, запускающее зависимые задачи при сбое родительской задачи. Напоследок рассматриваются поддерживаемые пакетной службой [сценарии использования зависимостей](#dependency-scenarios) .

## <a name="enable-task-dependencies"></a>Включение зависимостей задач

Чтобы использовать зависимости задач в приложении пакетной службы, сначала необходимо настроить задание для использования зависимостей задач. В .NET пакетной службы включите зависимости задач в классе [CloudJob](/dotnet/api/microsoft.azure.batch.cloudjob), задав для свойства [UserTaskDependencies](/dotnet/api/microsoft.azure.batch.cloudjob.usestaskdependencies) значение `true`.

```csharp
CloudJob unboundJob = batchClient.JobOperations.CreateJob( "job001",
    new PoolInformation { PoolId = "pool001" });

// IMPORTANT: This is REQUIRED for using task dependencies.
unboundJob.UsesTaskDependencies = true;
```

Приведенный выше фрагмент кода batchClient представляет собой экземпляр класса [BatchClient](/dotnet/api/microsoft.azure.batch.batchclient).

## <a name="create-dependent-tasks"></a>Создание зависимости задач

Чтобы создать задачу, зависящую от выполнения одной или нескольких родительских задач, можно указать, что эта задача "зависит" от других задач. В пакетной среде .NET настройте свойство [CloudTask. DependsOn](/dotnet/api/microsoft.azure.batch.cloudtask.dependson) с помощью экземпляра класса [TaskDependencies](/dotnet/api/microsoft.azure.batch.taskdependencies) :

```csharp
// Task 'Flowers' depends on completion of both 'Rain' and 'Sun'
// before it is run.
new CloudTask("Flowers", "cmd.exe /c echo Flowers")
{
    DependsOn = TaskDependencies.OnIds("Rain", "Sun")
},
```

Этот фрагмент кода создает зависимую задачу с идентификатором задачи Flowers. Задача Flowers зависит от задач Rain и Sun. Выполнение задачи Flowers будет запланировано на вычислительном узле только после успешного завершения задач Rain и Sun.

> [!NOTE]
> По умолчанию задача считается успешно выполненной, когда она находится в состоянии Выполнено и ее код выхода равен `0`. В пакетной среде .NET это означает, что свойство [CloudTask. State](/dotnet/api/microsoft.azure.batch.cloudtask.state) имеет значение, `Completed` а свойство [TaskExecutionInformation. ExitCode](/dotnet/api/microsoft.azure.batch.taskexecutioninformation.exitcode) CloudTask имеет значение `0` . Чтобы узнать, как это изменить, см. раздел [действия зависимостей](#dependency-actions) .

## <a name="dependency-scenarios"></a>сценарии использования зависимостей

Есть три основных сценария зависимостей задач, которые можно использовать в пакетной службе Azure: "один к одному", "один ко многим" и зависимость диапазона идентификаторов задач. Эти три сценария можно объединить для предоставления четвертого сценария: "многие ко многим".

| Сценарий&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Пример | Иллюстрация |
|:---:| --- | --- |
|  [Один к одному](#one-to-one) |Задача *taskB* зависит от задачи *taskA*. <p/> Выполнение задачи *taskB* не начнется, пока задача *taskA* не будет успешно выполнена. |:::image type="content" source="media/batch-task-dependency/01_one_to_one.png" alt-text="Схема, показывающая сценарий зависимостей задач «один к одному»."::: |
|  [«Один ко многим»](#one-to-many) |Задача *taskC* зависит от задач *taskA* и *taskB*. <p/> Выполнение задачи *taskC* не начнется, пока задачи *taskA* и *taskB* не будут успешно выполнены). |:::image type="content" source="media/batch-task-dependency/02_one_to_many.png" alt-text="Схема, показывающая сценарий зависимости задач «один ко многим»."::: |
|  [Диапазон идентификаторов задач](#task-id-range) |Задача *taskD* зависит от ряда задач. <p/> Выполнение задачи *taskD* не начнется, пока не будут успешно выполнены задачи с идентификаторами от *1* до *10*. |:::image type="content" source="media/batch-task-dependency/03_task_id_range.png" alt-text="Схема, на которой показан сценарий зависимости задачи диапазона ИДЕНТИФИКАТОРов задач."::: |

> [!TIP]
> Вы можете создать связь **многие ко многим**, например, когда каждая задача C, D, E и F зависит от задач A и B. Это удобно, например, в сценариях распараллеленной предварительной обработки, где подчиненные задачи зависят от выходных данных нескольких вышестоящих задач.
> 
> В примерах в этом разделе зависимая задача выполняется только после успешного завершения родительской задачи. Это поведение по умолчанию для зависимой задачи. Зависимую задачу можно запустить после сбоя родительской задачи, указав [действие зависимости](#dependency-actions)  для переопределения поведения по умолчанию.

### <a name="one-to-one"></a>Один к одному

При взаимосвязи "один к одному" задача зависит от успешного выполнения одной родительской задачи. Чтобы создать зависимость, укажите идентификатор одной задачи для статического метода [TaskDependencies. OnId](/dotnet/api/microsoft.azure.batch.taskdependencies.onid) при заполнении свойства [CloudTask. DependsOn](/dotnet/api/microsoft.azure.batch.cloudtask.dependson) .

```csharp
// Task 'taskA' doesn't depend on any other tasks
new CloudTask("taskA", "cmd.exe /c echo taskA"),

// Task 'taskB' depends on completion of task 'taskA'
new CloudTask("taskB", "cmd.exe /c echo taskB")
{
    DependsOn = TaskDependencies.OnId("taskA")
},
```

### <a name="one-to-many"></a>Один ко многим

При взаимосвязи "один ко многим" задача зависит от успешного выполнения нескольких родительских задач. Чтобы создать зависимость, укажите коллекцию идентификаторов задач для статического метода [TaskDependencies. OnIds](/dotnet/api/microsoft.azure.batch.taskdependencies.onids) при заполнении свойства [CloudTask. DependsOn](/dotnet/api/microsoft.azure.batch.cloudtask.dependson) .

```csharp
// 'Rain' and 'Sun' don't depend on any other tasks
new CloudTask("Rain", "cmd.exe /c echo Rain"),
new CloudTask("Sun", "cmd.exe /c echo Sun"),

// Task 'Flowers' depends on completion of both 'Rain' and 'Sun'
// before it is run.
new CloudTask("Flowers", "cmd.exe /c echo Flowers")
{
    DependsOn = TaskDependencies.OnIds("Rain", "Sun")
},
```

### <a name="task-id-range"></a>Диапазон идентификаторов задач

При зависимости от диапазона родительских задач задача зависит от выполнения задач, идентификаторы которых находятся в указанном диапазоне.
Чтобы создать зависимость, укажите идентификаторы первой и последней задач в диапазоне для статического метода [TaskDependencies](/dotnet/api/microsoft.azure.batch.taskdependencies.onidrange) при заполнении свойства [CloudTask. DependsOn](/dotnet/api/microsoft.azure.batch.cloudtask.dependson) .

> [!IMPORTANT]
> Если для зависимостей используются диапазоны идентификаторов задач, для диапазона будут выбираться только задачи с идентификаторами, представляющими целые числа. Например, диапазон `1..10` будет выбирать задачи `3` и `7` , но не `5flamingoes` .
>
> При оценке зависимостей диапазона нули в начале идентификаторов не учитываются. Поэтому задачи с идентификаторами строк `4`, `04` и `004` попадут *в пределы* диапазона, и все они будут рассматриваться как задача `4`. Любая из этих задач, которая завершится первой, выполнит условие зависимости.
>
> Каждая задача в диапазоне должна соответствовать зависимости, как только завершается успешно, так и путем завершения с ошибкой, которая сопоставлена с [действием зависимости](#dependency-actions) , настроенным для **удовлетворения**.

```csharp
// Tasks 1, 2, and 3 don't depend on any other tasks. Because
// we will be using them for a task range dependency, we must
// specify string representations of integers as their ids.
new CloudTask("1", "cmd.exe /c echo 1"),
new CloudTask("2", "cmd.exe /c echo 2"),
new CloudTask("3", "cmd.exe /c echo 3"),

// Task 4 depends on a range of tasks, 1 through 3
new CloudTask("4", "cmd.exe /c echo 4")
{
    // To use a range of tasks, their ids must be integer values.
    // Note that we pass integers as parameters to TaskIdRange,
    // but their ids (above) are string representations of the ids.
    DependsOn = TaskDependencies.OnIdRange(1, 3)
},
```

## <a name="dependency-actions"></a>Действия зависимостей

По умолчанию зависимая задача или набор задач выполняется только после успешного завершения родительской задачи. В некоторых случаях может потребоваться запустить зависимые задачи даже в случае сбоя родительской задачи. Поведение по умолчанию можно переопределить, указав действие зависимости.

Действие зависимости указывает, следует ли запускать зависимую задачу в случае успешного завершения или сбоя родительской задачи. Например предположим, что зависимая задача ожидает данные из восходящей задачи. При сбое восходящей задачи зависимая задача все равно может быть выполнена с использованием старых данных. В этом случае с помощью действия зависимости можно указать, что зависимая задача может быть выполнена в случае сбоя родительской задачи.

Действие зависимости основано на условии выхода родительской задачи. Можно указать действие зависимости для любого из следующих условий выхода:

- Если возникла ошибка предварительной обработки.
- Если возникла ошибка отправки файла. Если задача завершается с кодом выхода, который был задан с помощью свойства **exitCodes** или **exitCodeRanges**, а затем возникает ошибка отправки файла, действие, указанное в коде выхода, имеет более высокий приоритет.
- Когда задача завершается с кодом выхода, определенным в свойстве **ExitCodes**.
- Когда задача завершается с кодом выхода, который попадает в диапазон, заданный свойством **ExitCodeRanges**.
- По умолчанию, если задача завершается с кодом выхода, не указанным в свойстве **ExitCodes** или **ExitCodeRanges**, или если задача завершается с ошибкой предварительной обработки и свойство **PreProcessingError** не задано, или если происходит сбой выполнения задачи с ошибкой отправки файла и свойство **FileUploadError** не задано. 

Дополнительные сведения об этих условиях для .NET см. в разделе класс [екситкондитионс](/dotnet/api/microsoft.azure.batch.exitconditions) .

Чтобы указать действие зависимости в .NET, задайте для свойства [екситоптионс. DependencyAction](/dotnet/api/microsoft.azure.batch.exitoptions.dependencyaction) в качестве условия выхода одно из следующих:

- **Удовлетворение**: указывает, что зависимые задачи могут выполняться, если родительская задача завершается с указанной ошибкой.
- **Блокировать**: указывает, что зависимые задачи не могут выполняться.

По умолчанию для свойства **DependencyAction** заданы значение **Satisfy** для кода выхода 0 и значение **Block** для всех остальных условий выхода.

В следующем фрагменте кода настраивается свойство **DependencyAction** родительской задачи. Если родительская задача завершается ошибкой предварительной обработки или ошибкой с заданным кодом, то зависимая задача блокируется. Если родительская задача завершается с любой другой ошибкой, код которой не равен 0, то зависимая задача выполняется.

```csharp
// Task A is the parent task.
new CloudTask("A", "cmd.exe /c echo A")
{
    // Specify exit conditions for task A and their dependency actions.
    ExitConditions = new ExitConditions
    {
        // If task A exits with a pre-processing error, block any downstream tasks (in this example, task B).
        PreProcessingError = new ExitOptions
        {
            DependencyAction = DependencyAction.Block
        },
        // If task A exits with the specified error codes, block any downstream tasks (in this example, task B).
        ExitCodes = new List<ExitCodeMapping>
        {
            new ExitCodeMapping(10, new ExitOptions() { DependencyAction = DependencyAction.Block }),
            new ExitCodeMapping(20, new ExitOptions() { DependencyAction = DependencyAction.Block })
        },
        // If task A succeeds or fails with any other error, any downstream tasks become eligible to run 
        // (in this example, task B).
        Default = new ExitOptions
        {
            DependencyAction = DependencyAction.Satisfy
        }
    }
},
// Task B depends on task A. Whether it becomes eligible to run depends on how task A exits.
new CloudTask("B", "cmd.exe /c echo B")
{
    DependsOn = TaskDependencies.OnId("A")
},
```

## <a name="code-sample"></a>Пример кода

Пример проекта [TaskDependencies](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/TaskDependencies) в GitHub демонстрирует:

- Включение зависимости задач для задания.
- Создание задач, зависящих от других задач.
- как выполнить эти задачи в пуле вычислительных узлов.

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте о компоненте пакетной службы [пакетов](batch-application-packages.md) , который предоставляет простой способ развертывания и версии приложений, выполняемых задачами на вычислений узлах.
- Сведения о [проверке ошибок для заданий и задач](batch-job-task-error-checking.md).
