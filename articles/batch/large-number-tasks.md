---
title: Отправка большого количества задач в пакетное задание
description: Узнайте, как эффективно отправлять очень большое количество задач в одном пакетном задании Azure.
ms.topic: how-to
ms.date: 12/30/2020
ms.custom: devx-track-python, devx-track-csharp
ms.openlocfilehash: 08cf92507a4556afbf56c9cb7e2c9c1b3a6c9479
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97831522"
---
# <a name="submit-a-large-number-of-tasks-to-a-batch-job"></a>Отправка большого количества задач в пакетное задание

При выполнении крупномасштабных рабочих нагрузок пакетной службы Azure вам может понадобиться отправить десятки тысяч, сотни тысяч или даже больше задач в одно задание.

В этой статье показано, как отправить большое количество задач с значительно возросшей пропускной способностью для одного пакетного задания. Отправленные задачи поступают в очередь пакетной службы Azure для обработки в пуле, указанном для задания.

## <a name="use-task-collections"></a>Использование коллекций задач

При добавлении большого числа задач используйте соответствующие методы или перегрузки, предоставляемые API пакетной службы, чтобы добавлять задачи как *коллекцию* , а не по одной. Как правило, вы создаете коллекцию задач, определяя задачи по мере выполнения итерации набора входных файлов или параметров для своего задания.

Максимальный размер коллекции задач, которую можно добавить в одном вызове, зависит от используемого API пакетной службы.

### <a name="apis-allowing-collections-of-up-to-100-tasks"></a>Интерфейсы API, позволяющие выполнять коллекции до 100 задач

Эти API-интерфейсы пакетной службы ограничивают сбор до 100 задач. Это ограничение может быть небольшим в зависимости от размера задач (например, если задачи имеют большое количество файлов ресурсов или переменных среды).

- [REST API](/rest/api/batchservice/task/addcollection)
- [API Python](/python/api/azure-batch/azure.batch.operations.TaskOperations)
- [API Node.js](/javascript/api/@azure/batch/task)

При использовании этих API-интерфейсов необходимо предоставить логику для разделения количества задач в соответствии с ограничением коллекции, а также для обработки ошибок и повторных попыток в случае сбоев при добавлении задач. Если коллекция задач слишком велика для добавления, запрос сгенерирует ошибку. После этого нужно повторить попытку с меньшим количеством задач.

### <a name="apis-allowing-collections-of-larger-numbers-of-tasks"></a>Интерфейсы API, разрешающие коллекции большего числа задач

Другие API-интерфейсы пакетной службы поддерживают гораздо большие коллекции задач, ограниченные только объемом памяти, доступной на отправляющем клиенте. Эти API прозрачно управляют разделением коллекции задач на блоки для интерфейсов API нижнего уровня и повторные попытки добавления задач.

- [API для .NET](/dotnet/api/microsoft.azure.batch.cloudjob.addtaskasync)
- [API Java](/java/api/com.microsoft.azure.batch.protocol.tasks.addcollectionasync)
- [расширение CLI пакетной службы Azure](batch-cli-templates.md) с шаблонами CLI пакетной службы;
- [расширение пакета SDK для Python](https://pypi.org/project/azure-batch-extensions/).

## <a name="increase-throughput-of-task-submission"></a>Увеличение пропускной способности отправки задачи

Добавление большой коллекции задач в задание может занять некоторое время. Например, Добавление задач 20 000 с помощью API .NET может занять до одной минуты. В зависимости от API пакетной службы и рабочей нагрузки можно повысить пропускную способность задачи, изменив один или несколько следующих элементов.

### <a name="task-size"></a>Размер задачи

Добавление крупных задач занимает больше времени, чем добавление меньших размеров. Чтобы уменьшить размер каждой задачи в коллекции, вы можете упростить командную строку задачи, уменьшить количество переменных среды или более эффективно выполнить требования к выполнению задачи.

Например, вместо использования большого количества файлов ресурсов Установите зависимости задач с помощью [задачи запуска](jobs-and-tasks.md#start-task) в пуле или используйте [пакет приложения](batch-application-packages.md) или [контейнер DOCKER](batch-docker-container-workloads.md).

### <a name="number-of-parallel-operations"></a>Число параллельных операций

В зависимости от API пакетной службы можно увеличить пропускную способность, увеличив максимальное количество параллельных операций, выполняемых клиентом пакетной службы. Настройте этот параметр с помощью свойства [BatchClientParallelOptions.MaxDegreeOfParallelism](/dotnet/api/microsoft.azure.batch.batchclientparalleloptions.maxdegreeofparallelism) в API .NET или параметра `threads` такого метода, как [TaskOperations.add_collection](/python/api/azure-batch/azure.batch.operations.TaskOperations), в расширении пакета SDK для Python в пакетной службе. (Это свойство недоступно в исходном пакете SDK для Python в пакетной службе.)

По умолчанию это свойство имеет значение 1, но его можно задать выше, чтобы повысить пропускную способность операций. Вы компенсируете увеличенную пропускную способность за счет использования пропускной способности сети и производительности ЦП. Пропускная способность задачи увеличивается в 100 раз по сравнению с `MaxDegreeOfParallelism` или `threads`. На практике необходимо задать для числа параллельных операций значение ниже 100.

 Расширение CLI пакетной службы Azure с шаблонами пакетной службы автоматически увеличивает количество параллельных операций на основе количества доступных ядер, но это свойство не настраивается в CLI.

### <a name="http-connection-limits"></a>Ограничения HTTP-подключений

Наличие большого количества параллельных HTTP-подключений может регулировать производительность клиента пакетной службы при добавлении большого количества задач. Некоторые API ограничивают количество HTTP-подключений. Например, при разработке с помощью API .NET свойству [ServicePointManager.DefaultConnectionLimit](/dotnet/api/system.net.servicepointmanager.defaultconnectionlimit) по умолчанию задается значение 2. Мы рекомендуем увеличить значение до числа, близкого к числу параллельных операций или более высокого.

## <a name="example-batch-net"></a>Пример .NET для пакетной службы

В следующих фрагментах кода C# показаны параметры, которые нужно настроить при добавлении большого количества задач с помощью API .NET пакетной службы.

Чтобы увеличить пропускную способность задачи, увеличьте значение свойства [MaxDegreeOfParallelism](/dotnet/api/microsoft.azure.batch.batchclientparalleloptions.maxdegreeofparallelism) в [BatchClient](/dotnet/api/microsoft.azure.batch.batchclient). Пример:

```csharp
BatchClientParallelOptions parallelOptions = new BatchClientParallelOptions()
  {
    MaxDegreeOfParallelism = 15
  };
...
```

Добавьте коллекцию задач в задание, используя соответствующую перегрузку метода [AddTaskAsync](/dotnet/api/microsoft.azure.batch.cloudjob.addtaskasync) или [AddTask](/dotnet/api/microsoft.azure.batch.cloudjob.addtask
). Пример:

```csharp
// Add a list of tasks as a collection
List<CloudTask> tasksToAdd = new List<CloudTask>(); // Populate with your tasks
...
await batchClient.JobOperations.AddTaskAsync(jobId, tasksToAdd, parallelOptions);
```

## <a name="example-batch-cli-extension"></a>Пример расширение CLI пакетной службы

Используя расширения CLI пакетной службы Azure с [шаблонами CLI пакетной службы](batch-cli-templates.md), создайте JSON-файл шаблона задания, который включает в себя [фабрику задач](https://github.com/Azure/azure-batch-cli-extensions/blob/master/doc/taskFactories.md). Фабрика задач настраивает коллекцию связанных задач для задания из одного определения задачи.  

Ниже приведен пример шаблона задания для одномерного задания параметрической очистки с большим количеством задач (в данном случае 250 000). Командная строка задачи — это простая команда `echo`.

```json
{
    "job": {
        "type": "Microsoft.Batch/batchAccounts/jobs",
        "apiVersion": "2016-12-01",
        "properties": {
            "id": "myjob",
            "constraints": {
                "maxWallClockTime": "PT5H",
                "maxTaskRetryCount": 1
            },
            "poolInfo": {
                "poolId": "mypool"
            },
            "taskFactory": {
                "type": "parametricSweep",
                "parameterSets": [
                    {
                        "start": 1,
                        "end": 250000,
                        "step": 1
                    }
                ],
                "repeatTask": {
                    "commandLine": "/bin/bash -c 'echo Hello world from task {0}'",
                    "constraints": {
                        "retentionTime":"PT1H"
                    }
                }
            },
            "onAllTasksComplete": "terminatejob"
        }
    }
}
```

Сведения о запуске задания с помощью шаблона см. в статье [Использование шаблонов интерфейса командной строки для пакетной службы Azure и передачи файлов](batch-cli-templates.md).

## <a name="example-batch-python-sdk-extension"></a>Пример расширение пакета SDK для Python в пакетной службе

Чтобы использовать расширение пакета SDK для Python в пакетной службе Azure, сначала установите пакет SDK для Python и расширение:

```
pip install azure-batch
pip install azure-batch-extensions
```

Настройте клиент `BatchExtensionsClient`, который использует расширение пакета SDK.

```python

client = batch.BatchExtensionsClient(
    base_url=BATCH_ACCOUNT_URL, resource_group=RESOURCE_GROUP_NAME, batch_account=BATCH_ACCOUNT_NAME)
...
```

Создайте коллекцию задач для добавления в задание. Пример:

```python
tasks = list()
# Populate the list with your tasks
...
```

Добавьте коллекцию задач с помощью [task.add_collection](/python/api/azure-batch/azure.batch.operations.TaskOperations). Настройте параметр `threads`, чтобы увеличить количество параллельных операций:

```python
try:
    client.task.add_collection(job_id, threads=100)
except Exception as e:
    raise e
```

Расширение пакета SDK для Python в пакетной службе также поддерживает добавление параметров задачи в задание с помощью спецификации JSON для фабрики задач. Например, настройте параметры задания для параметрического анализа, аналогичного анализу в предыдущем примере шаблона CLI пакетной службы:

```python
parameter_sweep = {
    "job": {
        "type": "Microsoft.Batch/batchAccounts/jobs",
        "apiVersion": "2016-12-01",
        "properties": {
            "id": "myjob",
            "poolInfo": {
                "poolId": "mypool"
            },
            "taskFactory": {
                "type": "parametricSweep",
                "parameterSets": [
                    {
                        "start": 1,
                        "end": 250000,
                        "step": 1
                    }
                ],
                "repeatTask": {
                    "commandLine": "/bin/bash -c 'echo Hello world from task {0}'",
                    "constraints": {
                        "retentionTime": "PT1H"
                    }
                }
            },
            "onAllTasksComplete": "terminatejob"
        }
    }
}
...
job_json = client.job.expand_template(parameter_sweep)
job_parameter = client.job.jobparameter_from_json(job_json)
```

Добавьте параметры задания в задание. Настройте параметр `threads`, чтобы увеличить количество параллельных операций:

```python
try:
    client.job.add(job_parameter, threads=50)
except Exception as e:
    raise e
```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о расширении CLI пакетной службы Azure см. в статье [Использование шаблонов интерфейса командной строки для пакетной службы Azure и передачи файлов](batch-cli-templates.md).
- Ознакомьтесь с дополнительными сведениями о [расширении пакета SDK для Python в пакетной службе](https://pypi.org/project/azure-batch-extensions/).
- Ознакомьтесь с [рекомендациями по пакетной службе Azure](best-practices.md).
