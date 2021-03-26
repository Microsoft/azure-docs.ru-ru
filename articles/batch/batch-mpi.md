---
title: Выполняйте приложения с интерфейсом передачи сообщений с помощью задач с несколькими экземплярами
description: Узнайте, как выполнять приложения с интерфейсом передачи сообщений (MPI), используя тип задачи с несколькими экземплярами в пакетной службе Azure.
ms.topic: how-to
ms.date: 03/25/2021
ms.openlocfilehash: 51fc580e0bb31e0e975c53b44887a5889a784eea
ms.sourcegitcommit: 73d80a95e28618f5dfd719647ff37a8ab157a668
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105605677"
---
# <a name="use-multi-instance-tasks-to-run-message-passing-interface-mpi-applications-in-batch"></a>Использование задач с несколькими экземплярами для запуска приложений с интерфейсом передачи сообщений в пакетной службе

Многоэкземплярные задачи позволяют выполнять задачи пакетной службы Azure на нескольких вычислительных узлах одновременно. Эти задачи реализуют в пакетной службе такие сценарии высокопроизводительных вычислений, как приложения интерфейса передачи сообщений (MPI). Из этой статьи вы узнаете, как выполнять задачи с несколькими экземплярами с помощью библиотеки [.NET пакетной службы](/dotnet/api/microsoft.azure.batch).

> [!NOTE]
> Примеры в этой статье посвящены вычислительным узлам Batch .NET, MS-MPI и Windows. Рассматриваемые здесь принципы выполнения многоэкземплярных задач применимы для других платформ и технологий (Python и Intel MPI на узлах Linux, например).

## <a name="multi-instance-task-overview"></a>Общие сведения о задачах с несколькими экземплярами

В пакетной службе каждая задача обычно выполняется на одном вычислительном узле: вы отправляете несколько задач в задание, и пакетная служба планирует выполнение каждой задачи на узле. Тем не менее, если настроить **параметры использования нескольких экземпляров** задачи, вы сообщите пакетной службе вместо этого создать одну основную задачу и несколько подзадач, которые выполняются на нескольких узлах.

:::image type="content" source="media/batch-mpi/batch_mpi_01.png" alt-text="Схема, показывающая общие сведения о параметрах с несколькими экземплярами.":::

При отправке задачи с несколькими экземплярами в задание пакетная служба выполняет определенные действия.

1. Пакетная служба создает одну **основную** задачу и несколько **подзадач** на основе параметров использования нескольких экземпляров. Общее число задач (основная плюс все подзадачи) соответствует количеству **экземпляров** (вычислительных узлов), указанному в параметрах использования нескольких экземпляров.
2. Пакетная служба назначает один из вычислительных узлов **главным** и планирует выполнение основной задачи на нем. Выполнение подзадач она планирует на остальных вычислительных узлах, выделенных для многоэкземплярной задачи, размещая по одной подзадаче на каждом узле.
3. Основная задача и все подзадачи скачивают **общие файлы ресурсов**, указанные в параметрах использования нескольких экземпляров.
4. Потом они выполняют **команду координации** , указанную в параметрах для множественных экземпляров. Команда координации обычно используется для подготовки узлов к выполнению задачи. Это может быть запуск фоновых служб (например, [Microsoft MPI](/message-passing-interface/microsoft-mpi) `smpd.exe` ) и проверка готовности узлов к обработке сообщений между узлами.
5. *После* того, как основная задача и все подзадачи успешно выполнят команду координации, основная задача выполнит **команду приложения** на главном узле. Команда приложения — это командная строка, указанная для многоэкземплярной задачи. Ее выполняет только основная задача. В решении на основе [MS-MPI](/message-passing-interface/microsoft-mpi) это место, где выполняется приложение с поддержкой MPI с помощью `mpiexec.exe` .

> [!NOTE]
> Хотя задача с несколькими экземплярами по функциональности отличается от остальных задач, она не относится к типу таких уникальных задач, как [StartTask](/dotnet/api/microsoft.azure.batch.starttask) или [JobPreparationTask](/dotnet/api/microsoft.azure.batch.jobpreparationtask). Задача с несколькими экземплярами — это всего лишь стандартная задача пакетной службы ([CloudTask](/dotnet/api/microsoft.azure.batch.cloudtask) в .NET пакетной службы) с настроенными параметрами нескольких экземпляров. В этой статье задача такого типа называется **задачей с несколькими экземплярами**.

## <a name="requirements-for-multi-instance-tasks"></a>Требования к задачам с несколькими экземплярами

При выполнении многоэкземплярной задачи требуется, чтобы в пуле был **включен обмен данными между узлами** и **отключено параллельное выполнение задач**. Чтобы отключить параллельное выполнение задач, присвойте свойству [CloudPool. таскслотсперноде](/dotnet/api/microsoft.azure.batch.cloudpool) значение 1.

> [!NOTE]
> Пакетная служба [ограничит](batch-quota-limit.md#pool-size-limits) размера пула, для которого включен обмен данными между узлами.

В этом фрагменте кода показано, как создать пул для многоэкземплярных задач, используя библиотеку .NET для пакетной службы.

```csharp
CloudPool myCloudPool =
    myBatchClient.PoolOperations.CreatePool(
        poolId: "MultiInstanceSamplePool",
        targetDedicatedComputeNodes: 3
        virtualMachineSize: "standard_d1_v2",
        cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));

// Multi-instance tasks require inter-node communication, and those nodes
// must run only one task at a time.
myCloudPool.InterComputeNodeCommunicationEnabled = true;
myCloudPool.TaskSlotsPerNode = 1;
```

> [!NOTE]
> Если вы попытаетесь запустить задачу с несколькими экземплярами в пуле с отключенным межузловым соединением или со значением *таскслотсперноде* больше 1, задача не будет запланирована — она остается неопределенной в активном состоянии.

### <a name="use-a-starttask-to-install-mpi"></a>Установка MPI с помощью StartTask

Для выполнения приложений MPI с помощью задачи с несколькими экземплярами на вычислительных узлах в пуле сначала необходимо установить реализацию MPI (например, MS-MPI или Intel MPI). Это хорошая возможность воспользоваться задачей [StartTask](/dotnet/api/microsoft.azure.batch.starttask), которая выполняется каждый раз при присоединении узла к пулу или его перезапуске. В этом фрагменте кода создается задача StartTask, указывающая пакет установки MS-MPI в качестве [файла ресурсов](/dotnet/api/microsoft.azure.batch.resourcefile). Командная строка задачи запуска выполняется после скачивания файла ресурсов на узел. В этом случае командная строка выполняет автоматическую установку MS-MPI.

```csharp
// Create a StartTask for the pool which we use for installing MS-MPI on
// the nodes as they join the pool (or when they are restarted).
StartTask startTask = new StartTask
{
    CommandLine = "cmd /c MSMpiSetup.exe -unattend -force",
    ResourceFiles = new List<ResourceFile> { new ResourceFile("https://mystorageaccount.blob.core.windows.net/mycontainer/MSMpiSetup.exe", "MSMpiSetup.exe") },
    UserIdentity = new UserIdentity(new AutoUserSpecification(elevationLevel: ElevationLevel.Admin)),
    WaitForSuccess = true
};
myCloudPool.StartTask = startTask;

// Commit the fully configured pool to the Batch service to actually create
// the pool and its compute nodes.
await myCloudPool.CommitAsync();
```

### <a name="remote-direct-memory-access-rdma"></a>Удаленный доступ к памяти (RDMA)

Если для пула пакетной службы выбран [размер с поддержкой RDMA](../virtual-machines/sizes-hpc.md?toc=/azure/virtual-machines/windows/toc.json), например А9, то приложение MPI может воспользоваться преимуществами сети RDMA Azure с удаленным доступом к памяти, обеспечивающей высокую производительность и низкие задержки.

Найдите размеры, указанные в поле [размеры для виртуальных машин в Azure](../virtual-machines/sizes.md) (для пулов VirtualMachineConfiguration) или [размеры для облачных служб](../cloud-services/cloud-services-sizes-specs.md) (для пулов клаудсервицесконфигуратион).

> [!NOTE]
> Чтобы воспользоваться преимуществами RDMA на [вычислительных узлах Linux](batch-linux-nodes.md), на узлах необходимо использовать **Intel MPI**.

## <a name="create-a-multi-instance-task-with-batch-net"></a>Создание задачи с несколькими экземплярами с помощью .NET для пакетной службы

Теперь после рассмотрения требований к пулу и установки пакета MPI самое время перейти к созданию задачи с несколькими экземплярами. В этом фрагменте кода мы создадим стандартную задачу [CloudTask](/dotnet/api/microsoft.azure.batch.cloudtask), а затем настроим ее свойство [MultiInstanceSettings](/dotnet/api/microsoft.azure.batch.cloudtask). Как упоминалось выше, многоэкземплярная задача не относится к особому типу задач, а является стандартной задачей пакетной службы, настраиваемой с помощью параметров для множественных экземпляров.

```csharp
// Create the multi-instance task. Its command line is the "application command"
// and will be executed *only* by the primary, and only after the primary and
// subtasks execute the CoordinationCommandLine.
CloudTask myMultiInstanceTask = new CloudTask(id: "mymultiinstancetask",
    commandline: "cmd /c mpiexec.exe -wdir %AZ_BATCH_TASK_SHARED_DIR% MyMPIApplication.exe");

// Configure the task's MultiInstanceSettings. The CoordinationCommandLine will be executed by
// the primary and all subtasks.
myMultiInstanceTask.MultiInstanceSettings =
    new MultiInstanceSettings(numberOfNodes) {
    CoordinationCommandLine = @"cmd /c start cmd /c ""%MSMPI_BIN%\smpd.exe"" -d",
    CommonResourceFiles = new List<ResourceFile> {
    new ResourceFile("https://mystorageaccount.blob.core.windows.net/mycontainer/MyMPIApplication.exe",
                     "MyMPIApplication.exe")
    }
};

// Submit the task to the job. Batch will take care of splitting it into subtasks and
// scheduling them for execution on the nodes.
await myBatchClient.JobOperations.AddTaskAsync("mybatchjob", myMultiInstanceTask);
```

## <a name="primary-task-and-subtasks"></a>Основная задача и подзадачи

При создании параметров для множественных экземпляров для задачи нужно указать число вычислительных узлов, которые должны выполнять задачу. При отправке задачи в задание пакетная служба создает одну **основную** задачу и такое количество **подзадач**, которое вместе с основной задачей совпадает с указанным количеством узлов.

Этим задачам присваивается целочисленный идентификатор в диапазоне от 0 до *numberOfInstances* -1. Задача с ИДЕНТИФИКАТОРом 0 является основной задачей, а все остальные идентификаторы являются подзадачами. Например, если для задачи создаются следующие параметры с несколькими экземплярами, то первичная задача будет иметь идентификатор 0, а подзадачи будут иметь идентификаторы от 1 до 9.

```csharp
int numberOfNodes = 10;
myMultiInstanceTask.MultiInstanceSettings = new MultiInstanceSettings(numberOfNodes);
```

### <a name="master-node"></a>Главный узел

Когда вы отправляете многоэкземплярную задачу, пакетная служба назначает один из вычислительных узлов главным и планирует выполнение основной задачи на нем. Выполнение подзадач она планирует на остальных вычислительных узлах, выделенных для многоэкземплярной задачи.

## <a name="coordination-command"></a>команду координации

**Команду координации** выполняют основная задача и подзадачи.

Вызов команды координации блокируется: пакетная служба не выполнит команду приложения, пока команда координации не будет успешно возвращена для всех подзадач. Таким образом, команда координации должна запустить все необходимые фоновые службы. Убедитесь, что они готовы к использованию, а затем выполните выход. К примеру, эта команда координации для решения с использованием MS-MPI версии 7 запускает службу SMPD на узле, а затем завершает работу:

`cmd /c start cmd /c ""%MSMPI_BIN%\smpd.exe"" -d`

Обратите внимание, что в этой команде координации используется `start` . Это необходимо, так как приложение `smpd.exe` не возвращается сразу после выполнения. Если не использовать команду start, то команда координации не возвращается и выполнение команды приложения блокируется.

## <a name="application-command"></a>Команда приложения

Если основная задача и все подзадачи выполнили команду координации, командную строку многоэкземплярной задачи выполняет *только* основная задача. Назовем эту командную строку **командой приложения** , чтобы отличать ее от команды координации.

Для приложений MS-MPI используйте команду приложения, чтобы выполнить приложение с поддержкой MPI с использованием `mpiexec.exe`. В качестве примера ниже приведена команда приложения для решения с использованием MS-MPI версии 7:

`cmd /c ""%MSMPI_BIN%\mpiexec.exe"" -c 1 -wdir %AZ_BATCH_TASK_SHARED_DIR% MyMPIApplication.exe`

> [!NOTE]
> Так как MS-MPI `mpiexec.exe` использует `CCP_NODES` переменную по умолчанию (см. раздел [переменные среды](#environment-variables)), приведенная выше Командная строка приложения исключает ее.

## <a name="environment-variables"></a>Переменные среды

Пакетная служба создает несколько [переменных среды](batch-compute-node-environment-variables.md) для конкретных задач с несколькими экземплярами на вычислительных узлах, выделенных для каждой задачи с несколькими экземплярами. Командные строки координации и приложения могут ссылаться на эти переменные среды, как и сценарии и программы, выполняемые с их помощью.

Приведенные ниже переменные среды создаются пакетной службой для использования многоэкземплярными задачами.

- `CCP_NODES`
- `AZ_BATCH_NODE_LIST`
- `AZ_BATCH_HOST_LIST`
- `AZ_BATCH_MASTER_NODE`
- `AZ_BATCH_TASK_SHARED_DIR`
- `AZ_BATCH_IS_CURRENT_NODE_MASTER`

Полную информацию об этих и других переменных среды вычислительных узлов пакетной службы, включая их содержимое и видимость, доступны в разделе [Переменные среды вычислительных узлов](batch-compute-node-environment-variables.md).

> [!TIP]
> [Пример кода MPI в пакетной службе Linux](https://github.com/Azure-Samples/azure-batch-samples/tree/master/Python/Batch/article_samples/mpi) содержит пример того, как можно использовать несколько переменных среды.

## <a name="resource-files"></a>Файлы ресурсов

Для многоэкземплярных задач есть два набора файлов ресурсов: **общие файлы ресурсов**, которые скачивают *все* задачи (основная задача и подзадачи), и **файлы ресурсов**, указанные непосредственно для многоэкземплярной задачи, которые скачивает *только основная* задача.

В параметрах для задачи с множественными экземплярами можно указать один или несколько **общих файлов ресурсов** . Основная задача и все подзадачи скачивают эти общие файлы ресурсов из [службы хранилища Azure](../storage/common/storage-introduction.md) в **общий каталог задач** каждого узла. Доступ к общему каталогу задачи можно получить из приложения и командной строки координации с помощью переменной среды `AZ_BATCH_TASK_SHARED_DIR` . Пути `AZ_BATCH_TASK_SHARED_DIR` идентичны для каждого узла, выделенного многоэкземплярной задаче, поэтому можно использовать общую команду координации для основной задачи и всех подзадач. Пакетная служба не предоставляет общий доступ к каталогу, как при удаленном доступе, но его можно использовать в качестве подключаемого каталога или общего ресурса, как упоминалось ранее в подсказке к переменным среды.

Файлы ресурсов, указываемые для самой многоэкземплярной задачи, по умолчанию скачиваются в ее рабочий каталог, `AZ_BATCH_TASK_WORKING_DIR`. Как уже упоминалось, в отличие от общих файлов ресурсов, только основная задача скачивает файлы ресурсов, указанные для многоэкземплярной задачи.

> [!IMPORTANT]
> Для создания ссылок на эти каталоги в командной строке всегда используйте переменные среды `AZ_BATCH_TASK_SHARED_DIR` и `AZ_BATCH_TASK_WORKING_DIR`. Не пытайтесь создать пути вручную.

## <a name="task-lifetime"></a>Срок действия задачи

Срок действия основной задачи определяет срок действия всей задачи с несколькими экземплярами. При выходе из основной задачи завершаются все подзадачи. Код выхода основной задачи — это код выхода самой задачи с несколькими экземплярами. На его основе можно определить, как завершилась задача (успешно или со сбоем), т. е. нужно ли выполнять повторные попытки.

Если любая из подзадач завершается неудачно, например, выполняется выход с ненулевым кодом возврата, происходит сбой всей задачи с несколькими экземплярами. Затем задача с несколькими экземплярами завершается и выполняется повторно, пока не будет достигнут предел повтора.

При удалении задачи с несколькими экземплярами пакетная служба удаляет основную задачу и все подзадачи. Все каталоги и файлы подзадач удаляются из вычислительных узлов также, как и при стандартной задаче.

Свойства [TaskConstraints](/dotnet/api/microsoft.azure.batch.taskconstraints) для задачи с несколькими экземплярами (включая [MaxTaskRetryCount](/dotnet/api/microsoft.azure.batch.taskconstraints.maxtaskretrycount), [MaxWallClockTime](/dotnet/api/microsoft.azure.batch.taskconstraints.maxwallclocktime) и [RetentionTime](/dotnet/api/microsoft.azure.batch.taskconstraints.retentiontime)), обрабатываются как свойства для стандартной задачи и применяются к основной задаче и всем подзадачам. Однако если изменить свойство Серетентионтиме после добавления задачи с несколькими экземплярами в задание, это изменение применяется только к основной задаче, а все подзадачи продолжают использовать исходный RetentionTime.

Список последних задач вычислительного узла отражает идентификатор подзадачи, если последняя задача была частью задачи с несколькими экземплярами.

## <a name="obtain-information-about-subtasks"></a>Получение сведений о подзадачах

Чтобы получить сведения о подзадачах с помощью библиотеки .NET пакетной службы, вызовите метод [CloudTask.ListSubtasks](/dotnet/api/microsoft.azure.batch.cloudtask.listsubtasks). Этот метод возвращает сведения о всех подзадачах и вычислительном узле, на котором выполняются задачи. С помощью этих сведений можно определить корневой каталог подзадачи, идентификатор пула, его текущее состояние, код выхода и многое другое. Эти сведения можно использовать в сочетании с методом [PoolOperations.GetNodeFile](/dotnet/api/microsoft.azure.batch.pooloperations.getnodefile), чтобы получить файлы подзадачи. Обратите внимание, что этот метод не возвращает сведения для основной задачи (идентификатор 0).

> [!NOTE]
> Если не указано иное, методы .NET пакетной службы, которые используются для задачи с несколькими экземплярами [CloudTask](/dotnet/api/microsoft.azure.batch.cloudtask), применяются *только* к основной задаче. Например, при вызове метода [CloudTask.ListNodeFiles](/dotnet/api/microsoft.azure.batch.cloudtask.listnodefiles) для задачи с несколькими экземплярами возвращаются только файлы основной задачи.

В следующем фрагменте кода показано, как получить сведения о подзадачах, а также запросить содержимое файла из узлов, на которых выполняются эти подзадачи.

```csharp
// Obtain the job and the multi-instance task from the Batch service
CloudJob boundJob = batchClient.JobOperations.GetJob("mybatchjob");
CloudTask myMultiInstanceTask = boundJob.GetTask("mymultiinstancetask");

// Now obtain the list of subtasks for the task
IPagedEnumerable<SubtaskInformation> subtasks = myMultiInstanceTask.ListSubtasks();

// Asynchronously iterate over the subtasks and print their stdout and stderr
// output if the subtask has completed
await subtasks.ForEachAsync(async (subtask) =>
{
    Console.WriteLine("subtask: {0}", subtask.Id);
    Console.WriteLine("exit code: {0}", subtask.ExitCode);

    if (subtask.State == SubtaskState.Completed)
    {
        ComputeNode node =
            await batchClient.PoolOperations.GetComputeNodeAsync(subtask.ComputeNodeInformation.PoolId,
                                                                 subtask.ComputeNodeInformation.ComputeNodeId);

        NodeFile stdOutFile = await node.GetNodeFileAsync(subtask.ComputeNodeInformation.TaskRootDirectory + "\\" + Constants.StandardOutFileName);
        NodeFile stdErrFile = await node.GetNodeFileAsync(subtask.ComputeNodeInformation.TaskRootDirectory + "\\" + Constants.StandardErrorFileName);
        stdOut = await stdOutFile.ReadAsStringAsync();
        stdErr = await stdErrFile.ReadAsStringAsync();

        Console.WriteLine("node: {0}:", node.Id);
        Console.WriteLine("stdout.txt: {0}", stdOut);
        Console.WriteLine("stderr.txt: {0}", stdErr);
    }
    else
    {
        Console.WriteLine("\tSubtask {0} is in state {1}", subtask.Id, subtask.State);
    }
});
```

## <a name="code-sample"></a>Пример кода

В примере кода [MultiInstanceTasks](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/MultiInstanceTasks) в GitHub демонстрируется использование задачи с несколькими экземплярами для запуска приложения [MS-MPI](/message-passing-interface/microsoft-mpi) на вычислительных узлах пакетной службы. Выполните приведенные ниже действия, чтобы запустить пример.

### <a name="preparation"></a>Подготовка

1. Скачайте [пакет SDK для MS-MPI и распространяемые установщики](/message-passing-interface/microsoft-mpi) и установите их. После установки можно проверить, что переменные среды MS-MPI установлены.
1. Создайте *готовую* сборку программы-примера MPI [MPIHelloWorld](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp/ArticleProjects/MultiInstanceTasks/MPIHelloWorld). Это программа, которую будет выполнять на вычислительных узлах задача с несколькими экземплярами.
1. Создайте ZIP-файл `MPIHelloWorld.exe` (который вы создали на шаге 2) и `MSMpiSetup.exe` (который был скачан на шаге 1). Вы отправите этот ZIP-файл как пакет приложения на следующем шаге.
1. Используйте [портал Azure](https://portal.azure.com) для создания [приложения](batch-application-packages.md) пакетной службы с именем "MPIHelloWorld" и укажите ZIP-файл, созданный на предыдущем шаге, в качестве версии 1.0 пакета приложения. Дополнительные сведения см. в разделе [Передача приложений и управление ими](batch-application-packages.md#upload-and-manage-applications).

> [!TIP]
> Создание *окончательной* версии `MPIHelloWorld.exe` гарантирует, что не нужно включать какие-либо дополнительные зависимости (например, `msvcp140d.dll` или `vcruntime140d.dll` ) в пакет приложения.

### <a name="execution"></a>Выполнение

1. Скачайте [файл Azure-Batch-Samples. zip](https://github.com/Azure/azure-batch-samples/archive/master.zip) из GitHub.
1. Откройте **решение** MultiInstanceTasks в Visual Studio 2019. Файл решения `MultiInstanceTasks.sln` находится в следующем расположении:

    `azure-batch-samples\CSharp\ArticleProjects\MultiInstanceTasks\`
1. Введите данные своей учетной записи пакетной службы и учетной записи хранения в `AccountSettings.settings` в проекте **Microsoft.Azure.Batch.Samples.Common**.
1. **Создайте и запустите** решение MultiInstanceTasks, чтобы выполнить пример приложения MPI на вычислительных узлах в пуле пакетной службы.
1. *Необязательно*: используйте [портал Azure](https://portal.azure.com) или [Batch Explorer](https://azure.github.io/BatchExplorer/), чтобы изучить пример пула, задания и задачи ("MultiInstanceSamplePool", "MultiInstanceSampleJob", "MultiInstanceSampleTask") перед тем, как удалить ресурсы.

> [!TIP]
> Вы можете скачать [Visual Studio Community](https://visualstudio.microsoft.com/vs/community/) бесплатно, если у вас еще нет Visual Studio.

Результат `MultiInstanceTasks.exe` аналогичен приведенному ниже:

```
Creating pool [MultiInstanceSamplePool]...
Creating job [MultiInstanceSampleJob]...
Adding task [MultiInstanceSampleTask] to job [MultiInstanceSampleJob]...
Awaiting task completion, timeout in 00:30:00...

Main task [MultiInstanceSampleTask] is in state [Completed] and ran on compute node [tvm-1219235766_1-20161017t162002z]:
---- stdout.txt ----
Rank 2 received string "Hello world" from Rank 0
Rank 1 received string "Hello world" from Rank 0

---- stderr.txt ----

Main task completed, waiting 00:00:10 for subtasks to complete...

---- Subtask information ----
subtask: 1
        exit code: 0
        node: tvm-1219235766_3-20161017t162002z
        stdout.txt:
        stderr.txt:
subtask: 2
        exit code: 0
        node: tvm-1219235766_2-20161017t162002z
        stdout.txt:
        stderr.txt:

Delete job? [yes] no: yes
Delete pool? [yes] no: yes

Sample complete, hit ENTER to exit...
```

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше о [поддержке MPI для Linux в пакетной службе Azure](https://docs.microsoft.com/archive/blogs/windowshpc/introducing-mpi-support-for-linux-on-azure-batch).
- Узнайте, как [создавать пулы на вычислительных узлах Linux](batch-linux-nodes.md) для использования в решениях MPI пакетной службы Azure.
