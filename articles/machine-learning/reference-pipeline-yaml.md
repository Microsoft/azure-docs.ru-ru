---
title: YAML конвейера Машинное обучение
titleSuffix: Azure Machine Learning
description: Узнайте, как определить конвейер машинного обучения с помощью файла YAML. Определения конвейера YAML используются с расширением машинного обучения для Azure CLI.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
ms.reviewer: larryfr
ms.author: nilsp
author: NilsPohlmann
ms.date: 07/31/2020
ms.custom: devx-track-python
ms.openlocfilehash: e2b5a3322f633ca8301357c2186d78d3ac437ae2
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102521974"
---
# <a name="define-machine-learning-pipelines-in-yaml"></a>Определение конвейеров машинного обучения в YAML

Узнайте, как определить конвейеры машинного обучения в [YAML](https://yaml.org/). При использовании расширения машинного обучения для Azure CLI многие команды, связанные с конвейером, предполагают файл YAML, определяющий конвейер.

В следующей таблице перечислены, что и сейчас не поддерживается при определении конвейера в YAML.

| Тип шага | Поддержка |
| ----- | :-----: |
| писонскриптстеп | Да |
| параллелрунстеп | Да |
| адластеп | Да |
| азуребатчстеп | Да |
| датабрикксстеп | Да |
| дататрансферстеп | Да |
| аутомлстеп | Нет |
| HyperDriveStep | Нет |
| модулестеп | Да |
| мпистеп | Нет |
| EstimatorStep | Нет |

## <a name="pipeline-definition"></a>Определение конвейера

В определении конвейера используются следующие ключи, которые соответствуют классу [конвейеров](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipeline.pipeline) :

| Ключ YAML | Описание |
| ----- | ----- |
| `name` | Описание конвейера. |
| `parameters` | Параметры для конвейера. |
| `data_reference` | Определяет, как и где следует предоставлять доступ к данным в запуске. |
| `default_compute` | Целевой объект вычислений по умолчанию, в котором выполняются все шаги в конвейере. |
| `steps` | Шаги, используемые в конвейере. |

## <a name="parameters"></a>Параметры

`parameters`Раздел использует следующие ключи, которые соответствуют классу [пипелинепараметер](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelineparameter) :

| Ключ YAML | Описание |
| ---- | ---- |
| `type` | Тип значения параметра. Допустимые типы: `string` , `int` ,, `float` `bool` или `datapath` . |
| `default` | Значение по умолчанию. |

Каждый параметр имеет имя. Например, следующий фрагмент YAML определяет три параметра с именами `NumIterationsParameter` , `DataPathParameter` и `NodeCountParameter` :

```yaml
pipeline:
    name: SamplePipelineFromYaml
    parameters:
        NumIterationsParameter:
            type: int
            default: 40
        DataPathParameter:
            type: datapath
            default:
                datastore: workspaceblobstore
                path_on_datastore: sample2.txt
        NodeCountParameter:
            type: int
            default: 4
```

## <a name="data-reference"></a>Описание данных

`data_references`Раздел использует следующие ключи, которые соответствуют [ссылке](/python/api/azureml-core/azureml.data.data_reference.datareference)на объект.

| Ключ YAML | Описание |
| ----- | ----- |
| `datastore` | Хранилище данных для ссылки. |
| `path_on_datastore` | Относительный путь в резервном хранилище для ссылки на данные. |

Каждая ссылка на данные содержится в ключе. Например, следующий фрагмент YAML определяет ссылку на данные, хранимую в ключе с именем `employee_data` :

```yaml
pipeline:
    name: SamplePipelineFromYaml
    parameters:
        PipelineParam1:
            type: int
            default: 3
    data_references:
        employee_data:
            datastore: adftestadla
            path_on_datastore: "adla_sample/sample_input.csv"
```

## <a name="steps"></a>Шаги

Шаги определяют вычислительную среду, а также файлы для выполнения в среде. Чтобы определить тип шага, используйте `type` ключ:

| Тип шага | Описание |
| ----- | ----- |
| `AdlaStep` | Выполняет скрипт U-SQL с Azure Data Lake Analytics. Соответствует классу [адластеп](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.adlastep) . |
| `AzureBatchStep` | Запускает задания с помощью пакетной службы Azure. Соответствует классу [азуребатчстеп](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.azurebatchstep) . |
| `DatabricsStep` | Добавляет записную книжку, скрипт Python или JAR-файл. Соответствует классу [датабрикксстеп](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.databricksstep) . |
| `DataTransferStep` | Передает данные между вариантами хранения. Соответствует классу [дататрансферстеп](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.datatransferstep) . |
| `PythonScriptStep` | Выполняет скрипт Python. Соответствует классу [писонскриптстеп](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.python_script_step.pythonscriptstep) . |
| `ParallelRunStep` | Запускает скрипт Python для асинхронного и параллельного обработки больших объемов данных. Соответствует классу [параллелрунстеп](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.parallel_run_step.parallelrunstep) . |

### <a name="adla-step"></a>ADLA шаг

| Ключ YAML | Описание |
| ----- | ----- |
| `script_name` | Имя скрипта U-SQL (относительно `source_directory` ). |
| `compute_target` | Целевой объект Azure Data Lake вычислений, используемый для этого шага. |
| `parameters` | [Параметры](#parameters) для конвейера. |
| `inputs` | Входными данными могут быть [инпутпортбиндинг](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.inputportbinding), [Reference](#data-reference), [портдатареференце](/python/api/azureml-pipeline-core/azureml.pipeline.core.portdatareference), [пипелинедата](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata), [DataSet](/python/api/azureml-core/azureml.core.dataset%28class%29), [датасетдефинитион](/python/api/azureml-core/azureml.data.dataset_definition.datasetdefinition)или [пипелинедатасет](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedataset). |
| `outputs` | Выходные данные могут быть либо [пипелинедата](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata) , либо [аутпутпортбиндинг](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.outputportbinding). |
| `source_directory` | Каталог, содержащий скрипт, сборки и т. д. |
| `priority` | Значение приоритета, используемое для текущего задания. |
| `params` | Словарь пар "имя-значение". |
| `degree_of_parallelism` | Степень параллелизма, используемая для этого задания. |
| `runtime_version` | Версия среды выполнения Data Lake Analytics Engine. |
| `allow_reuse` | Определяет, должен ли шаг повторно использовать предыдущие результаты при повторном выполнении с теми же параметрами. |

Следующий пример содержит определение шага ADLA:

```yaml
pipeline:
    name: SamplePipelineFromYaml
    parameters:
        PipelineParam1:
            type: int
            default: 3
    data_references:
        employee_data:
            datastore: adftestadla
            path_on_datastore: "adla_sample/sample_input.csv"
    default_compute: adlacomp
    steps:
        Step1:
            runconfig: "D:\\Yaml\\default_runconfig.yml"
            parameters:
                NUM_ITERATIONS_2:
                    source: PipelineParam1
                NUM_ITERATIONS_1: 7
            type: "AdlaStep"
            name: "MyAdlaStep"
            script_name: "sample_script.usql"
            source_directory: "D:\\scripts\\Adla"
            inputs:
                employee_data:
                    source: employee_data
            outputs:
                OutputData:
                    destination: Output4
                    datastore: adftestadla
                    bind_mode: mount
```

### <a name="azure-batch-step"></a>Шаг пакетной службы Azure

| Ключ YAML | Описание |
| ----- | ----- |
| `compute_target` | Целевой объект вычислений пакетной службы Azure, который будет использоваться для этого шага. |
| `inputs` | Входными данными могут быть [инпутпортбиндинг](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.inputportbinding), [Reference](#data-reference), [портдатареференце](/python/api/azureml-pipeline-core/azureml.pipeline.core.portdatareference), [пипелинедата](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata), [DataSet](/python/api/azureml-core/azureml.core.dataset%28class%29), [датасетдефинитион](/python/api/azureml-core/azureml.data.dataset_definition.datasetdefinition)или [пипелинедатасет](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedataset). |
| `outputs` | Выходные данные могут быть либо [пипелинедата](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata) , либо [аутпутпортбиндинг](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.outputportbinding). |
| `source_directory` | Каталог, содержащий двоичные файлы модуля, исполняемый файл, сборки и т. д. |
| `executable` | Имя команды или исполняемого файла, которые будут запускаться в рамках этого задания. |
| `create_pool` | Логический флаг, указывающий, следует ли создать пул перед выполнением задания. |
| `delete_batch_job_after_finish` | Логический флаг, указывающий, следует ли удалить задание из учетной записи пакетной службы после ее завершения. |
| `delete_batch_pool_after_finish` | Логический флаг, указывающий, следует ли удалить пул после завершения задания. |
| `is_positive_exit_code_failure` | Логический флаг, указывающий, завершается ли задание ошибкой, если задача завершается с положительным кодом. |
| `vm_image_urn` | Если `create_pool` имеет значение `True` , а виртуальная машина использует `VirtualMachineConfiguration` . |
| `pool_id` | Идентификатор пула, в котором будет выполняться задание. |
| `allow_reuse` | Определяет, должен ли шаг повторно использовать предыдущие результаты при повторном выполнении с теми же параметрами. |

В следующем примере содержится определение шага пакетной службы Azure.

```yaml
pipeline:
    name: SamplePipelineFromYaml
    parameters:
        PipelineParam1:
            type: int
            default: 3
    data_references:
        input:
            datastore: workspaceblobstore
            path_on_datastore: "input.txt"
    default_compute: testbatch
    steps:
        Step1:
            runconfig: "D:\\Yaml\\default_runconfig.yml"
            parameters:
                NUM_ITERATIONS_2:
                    source: PipelineParam1
                NUM_ITERATIONS_1: 7
            type: "AzureBatchStep"
            name: "MyAzureBatchStep"
            pool_id: "MyPoolName"
            create_pool: true
            executable: "azurebatch.cmd"
            source_directory: "D:\\scripts\\AureBatch"
            allow_reuse: false
            inputs:
                input:
                    source: input
            outputs:
                output:
                    destination: output
                    datastore: workspaceblobstore
```

### <a name="databricks-step"></a>Шаг "кирпичы"

| Ключ YAML | Описание |
| ----- | ----- |
| `compute_target` | Целевой объект Azure Databricks вычислений, используемый для этого шага. |
| `inputs` | Входными данными могут быть [инпутпортбиндинг](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.inputportbinding), [Reference](#data-reference), [портдатареференце](/python/api/azureml-pipeline-core/azureml.pipeline.core.portdatareference), [пипелинедата](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata), [DataSet](/python/api/azureml-core/azureml.core.dataset%28class%29), [датасетдефинитион](/python/api/azureml-core/azureml.data.dataset_definition.datasetdefinition)или [пипелинедатасет](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedataset). |
| `outputs` | Выходные данные могут быть либо [пипелинедата](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata) , либо [аутпутпортбиндинг](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.outputportbinding). |
| `run_name` | Имя в модулях для этого запуска. |
| `source_directory` | Каталог, содержащий скрипт и другие файлы. |
| `num_workers` | Статическое число рабочих ролей для модуля данных, выполняющих кластер. |
| `runconfig` | Путь к `.runconfig` файлу. Этот файл является YAML представлением класса [RunConfiguration](/python/api/azureml-core/azureml.core.runconfiguration) . Дополнительные сведения о структуре этого файла см. [ в разделеrunconfigschema.json](https://github.com/microsoft/MLOps/blob/b4bdcf8c369d188e83f40be8b748b49821f71cf2/infra-as-code/runconfigschema.json). |
| `allow_reuse` | Определяет, должен ли шаг повторно использовать предыдущие результаты при повторном выполнении с теми же параметрами. |

Следующий пример содержит шаг "кирпичы".

```yaml
pipeline:
    name: SamplePipelineFromYaml
    parameters:
        PipelineParam1:
            type: int
            default: 3
    data_references:
        adls_test_data:
            datastore: adftestadla
            path_on_datastore: "testdata"
        blob_test_data:
            datastore: workspaceblobstore
            path_on_datastore: "dbtest"
    default_compute: mydatabricks
    steps:
        Step1:
            runconfig: "D:\\Yaml\\default_runconfig.yml"
            parameters:
                NUM_ITERATIONS_2:
                    source: PipelineParam1
                NUM_ITERATIONS_1: 7
            type: "DatabricksStep"
            name: "MyDatabrickStep"
            run_name: "DatabricksRun"
            python_script_name: "train-db-local.py"
            source_directory: "D:\\scripts\\Databricks"
            num_workers: 1
            allow_reuse: true
            inputs:
                blob_test_data:
                    source: blob_test_data
            outputs:
                OutputData:
                    destination: Output4
                    datastore: workspaceblobstore
                    bind_mode: mount
```

### <a name="data-transfer-step"></a>Шаг пересылки данных

| Ключ YAML | Описание |
| ----- | ----- |
| `compute_target` | Целевой объект вычислений фабрики данных Azure, который будет использоваться для этого шага. |
| `source_data_reference` | Входное соединение, служащее источником операций по переносу данных. Поддерживаемые значения: [инпутпортбиндинг](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.inputportbinding), [Reference](#data-reference), [портдатареференце](/python/api/azureml-pipeline-core/azureml.pipeline.core.portdatareference), [пипелинедата](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata), [DataSet](/python/api/azureml-core/azureml.core.dataset%28class%29), [датасетдефинитион](/python/api/azureml-core/azureml.data.dataset_definition.datasetdefinition)или [пипелинедатасет](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedataset). |
| `destination_data_reference` | Входное соединение, которое служит в качестве назначения для операций обмена данными. Поддерживаемые значения: [пипелинедата](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata) и [аутпутпортбиндинг](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.outputportbinding). |
| `allow_reuse` | Определяет, должен ли шаг повторно использовать предыдущие результаты при повторном выполнении с теми же параметрами. |

В следующем примере показан шаг перемещения данных.

```yaml
pipeline:
    name: SamplePipelineFromYaml
    parameters:
        PipelineParam1:
            type: int
            default: 3
    data_references:
        adls_test_data:
            datastore: adftestadla
            path_on_datastore: "testdata"
        blob_test_data:
            datastore: workspaceblobstore
            path_on_datastore: "testdata"
    default_compute: adftest
    steps:
        Step1:
            runconfig: "D:\\Yaml\\default_runconfig.yml"
            parameters:
                NUM_ITERATIONS_2:
                    source: PipelineParam1
                NUM_ITERATIONS_1: 7
            type: "DataTransferStep"
            name: "MyDataTransferStep"
            adla_compute_name: adftest
            source_data_reference:
                adls_test_data:
                    source: adls_test_data
            destination_data_reference:
                blob_test_data:
                    source: blob_test_data
```

### <a name="python-script-step"></a>Шаг скрипта Python

| Ключ YAML | Описание |
| ----- | ----- |
| `inputs` | Входными данными могут быть [инпутпортбиндинг](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.inputportbinding), [Reference](#data-reference), [портдатареференце](/python/api/azureml-pipeline-core/azureml.pipeline.core.portdatareference), [пипелинедата](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata), [DataSet](/python/api/azureml-core/azureml.core.dataset%28class%29), [датасетдефинитион](/python/api/azureml-core/azureml.data.dataset_definition.datasetdefinition)или [пипелинедатасет](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedataset). |
| `outputs` | Выходные данные могут быть либо [пипелинедата](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata) , либо [аутпутпортбиндинг](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.outputportbinding). |
| `script_name` | Имя скрипта Python (относительно `source_directory` ). |
| `source_directory` | Каталог, содержащий скрипт, среду Conda и т. д. |
| `runconfig` | Путь к `.runconfig` файлу. Этот файл является YAML представлением класса [RunConfiguration](/python/api/azureml-core/azureml.core.runconfiguration) . Дополнительные сведения о структуре этого файла см. [ в разделеrunconfig.json](https://github.com/microsoft/MLOps/blob/b4bdcf8c369d188e83f40be8b748b49821f71cf2/infra-as-code/runconfigschema.json). |
| `allow_reuse` | Определяет, должен ли шаг повторно использовать предыдущие результаты при повторном выполнении с теми же параметрами. |

Следующий пример содержит шаг сценария Python:

```yaml
pipeline:
    name: SamplePipelineFromYaml
    parameters:
        PipelineParam1:
            type: int
            default: 3
    data_references:
        DataReference1:
            datastore: workspaceblobstore
            path_on_datastore: testfolder/sample.txt
    default_compute: cpu-cluster
    steps:
        Step1:
            runconfig: "D:\\Yaml\\default_runconfig.yml"
            parameters:
                NUM_ITERATIONS_2:
                    source: PipelineParam1
                NUM_ITERATIONS_1: 7
            type: "PythonScriptStep"
            name: "MyPythonScriptStep"
            script_name: "train.py"
            allow_reuse: True
            source_directory: "D:\\scripts\\PythonScript"
            inputs:
                InputData:
                    source: DataReference1
            outputs:
                OutputData:
                    destination: Output4
                    datastore: workspaceblobstore
                    bind_mode: mount
```

### <a name="parallel-run-step"></a>Шаг параллельного выполнения

| Ключ YAML | Описание |
| ----- | ----- |
| `inputs` | Входными данными могут быть [наборы данных](/python/api/azureml-core/azureml.core.dataset%28class%29), [датасетдефинитион](/python/api/azureml-core/azureml.data.dataset_definition.datasetdefinition)или [пипелинедатасет](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedataset). |
| `outputs` | Выходные данные могут быть либо [пипелинедата](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata) , либо [аутпутпортбиндинг](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.outputportbinding). |
| `script_name` | Имя скрипта Python (относительно `source_directory` ). |
| `source_directory` | Каталог, содержащий скрипт, среду Conda и т. д. |
| `parallel_run_config` | Путь к `parallel_run_config.yml` файлу. Этот файл является YAML представлением класса [параллелрунконфиг](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.parallelrunconfig) . |
| `allow_reuse` | Определяет, должен ли шаг повторно использовать предыдущие результаты при повторном выполнении с теми же параметрами. |

Следующий пример содержит шаг параллельного выполнения:

```yaml
pipeline:
    description: SamplePipelineFromYaml
    default_compute: cpu-cluster
    data_references:
        MyMinistInput:
            dataset_name: mnist_sample_data
    parameters:
        PipelineParamTimeout:
            type: int
            default: 600
    steps:        
        Step1:
            parallel_run_config: "yaml/parallel_run_config.yml"
            type: "ParallelRunStep"
            name: "parallel-run-step-1"
            allow_reuse: True
            arguments:
            - "--progress_update_timeout"
            - parameter:timeout_parameter
            - "--side_input"
            - side_input:SideInputData
            parameters:
                timeout_parameter:
                    source: PipelineParamTimeout
            inputs:
                InputData:
                    source: MyMinistInput
            side_inputs:
                SideInputData:
                    source: Output4
                    bind_mode: mount
            outputs:
                OutputDataStep2:
                    destination: Output5
                    datastore: workspaceblobstore
                    bind_mode: mount
```

### <a name="pipeline-with-multiple-steps"></a>Конвейер с несколькими шагами 

| Ключ YAML | Описание |
| ----- | ----- |
| `steps` | Последовательность из одного или нескольких определений Пипелинестеп. Обратите внимание, что `destination` ключи одного шага `outputs` становятся `source` ключами к `inputs` следующему шагу.| 

```yaml
pipeline:
    name: SamplePipelineFromYAML
    description: Sample multistep YAML pipeline
    data_references:
        TitanicDS:
            dataset_name: 'titanic_ds'
            bind_mode: download
    default_compute: cpu-cluster
    steps:
        Dataprep:
            type: "PythonScriptStep"
            name: "DataPrep Step"
            compute: cpu-cluster
            runconfig: ".\\default_runconfig.yml"
            script_name: "prep.py"
            arguments:
            - '--train_path'
            - output:train_path
            - '--test_path'
            - output:test_path
            allow_reuse: True
            inputs:
                titanic_ds:
                    source: TitanicDS
                    bind_mode: download
            outputs:
                train_path:
                    destination: train_csv
                    datastore: workspaceblobstore
                test_path:
                    destination: test_csv
        Training:
            type: "PythonScriptStep"
            name: "Training Step"
            compute: cpu-cluster
            runconfig: ".\\default_runconfig.yml"
            script_name: "train.py"
            arguments:
            - "--train_path"
            - input:train_path
            - "--test_path"
            - input:test_path
            inputs:
                train_path:
                    source: train_csv
                    bind_mode: download
                test_path:
                    source: test_csv
                    bind_mode: download

```

## <a name="schedules"></a>Расписания

При определении расписания для конвейера это может быть либо хранилище данных, либо повторяющееся в зависимости от интервала времени. Ниже приведены ключи, используемые для определения расписания.

| Ключ YAML | Описание |
| ----- | ----- |
| `description` | Описание расписания. |
| `recurrence` | Содержит параметры повторения, если расписание повторяется. |
| `pipeline_parameters` | Все параметры, необходимые для конвейера. |
| `wait_for_provisioning` | Следует ли ожидать завершения подготовки расписания. |
| `wait_timeout` | Количество секунд ожидания до истечения времени ожидания. |
| `datastore_name` | Хранилище данных для отслеживания измененных или добавленных больших двоичных объектов. |
| `polling_interval` | Время в минутах между опросами измененных или добавленных больших двоичных объектов. Значение по умолчанию: 5 минут. Поддерживается только для расписаний хранилища данных. |
| `data_path_parameter_name` | Имя параметра конвейера пути к данным для задания с измененным путем к большому двоичному объекту. Поддерживается только для расписаний хранилища данных. |
| `continue_on_step_failure` | Следует ли продолжать выполнение других шагов в отправленном Пипелинерун, если шаг завершается ошибкой. Если он предоставлен, переопределит `continue_on_step_failure` настройку конвейера.
| `path_on_datastore` | Необязательный параметр. Путь в хранилище данных для отслеживания измененных или добавленных больших двоичных объектов. Путь находится под контейнером для хранилища данных, поэтому фактический путь, за которым отслеживаются мониторы, является контейнером/ `path_on_datastore` . Если нет, отслеживается контейнер хранилища данных. Добавления и изменения, внесенные во вложенную папку, `path_on_datastore` не отслеживаются. Поддерживается только для расписаний хранилища данных. |

Следующий пример содержит определение расписания, запускаемого хранилищем данных:

```yaml
Schedule: 
      description: "Test create with datastore" 
      recurrence: ~ 
      pipeline_parameters: {} 
      wait_for_provisioning: True 
      wait_timeout: 3600 
      datastore_name: "workspaceblobstore" 
      polling_interval: 5 
      data_path_parameter_name: "input_data" 
      continue_on_step_failure: None 
      path_on_datastore: "file/path" 
```

При определении **повторяющегося расписания** используйте следующие ключи в разделе `recurrence` :

| Ключ YAML | Описание |
| ----- | ----- |
| `frequency` | Частота повторения расписания. Допустимые значения: `"Minute"` , `"Hour"` , `"Day"` , `"Week"` или `"Month"` . |
| `interval` | Частота срабатывания расписания. Целочисленное значение — это количество единиц времени, которое нужно ожидать до повторного запуска расписания. |
| `start_time` | Время начала расписания. Формат строки значения — `YYYY-MM-DDThh:mm:ss` . Если время начала не указано, первая рабочая нагрузка выполняется мгновенно, а будущие рабочие нагрузки выполняются на основе расписания. Если время начала в прошлом, то первая рабочая нагрузка будет выполняться при следующем расчете времени выполнения. |
| `time_zone` | Часовой пояс для времени начала. Если часовой пояс не указан, используется время в формате UTC. |
| `hours` | Если `frequency` имеет значение `"Day"` или `"Week"` , можно указать одно или несколько целых чисел от 0 до 23, разделенных запятыми, как часы дня, когда должен выполняться конвейер. `time_of_day` `hours` Можно использовать только или и `minutes` . |
| `minutes` | Если `frequency` имеет значение `"Day"` или `"Week"` , можно указать одно или несколько целых чисел от 0 до 59, разделенных запятыми, как минуты часа, в которые должен выполняться конвейер. `time_of_day` `hours` Можно использовать только или и `minutes` . |
| `time_of_day` | Если `frequency` имеет значение `"Day"` или `"Week"` , можно указать время суток для выполнения расписания. Формат строки значения — `hh:mm` . `time_of_day` `hours` Можно использовать только или и `minutes` . |
| `week_days` | Если `frequency` имеет значение `"Week"` , можно указать один или несколько дней, разделенных запятыми, при запуске расписания. Допустимые значения: `"Monday"` , `"Tuesday"` , `"Wednesday"` ,,, `"Thursday"` `"Friday"` `"Saturday"` и `"Sunday"` . |

Следующий пример содержит определение расписания повторения:

```yaml
Schedule: 
    description: "Test create with recurrence" 
    recurrence: 
        frequency: Week # Can be "Minute", "Hour", "Day", "Week", or "Month". 
        interval: 1 # how often fires 
        start_time: 2019-06-07T10:50:00 
        time_zone: UTC 
        hours: 
        - 1 
        minutes: 
        - 0 
        time_of_day: null 
        week_days: 
        - Friday 
    pipeline_parameters: 
        'a': 1 
    wait_for_provisioning: True 
    wait_timeout: 3600 
    datastore_name: ~ 
    polling_interval: ~ 
    data_path_parameter_name: ~ 
    continue_on_step_failure: None 
    path_on_datastore: ~ 
```

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как [использовать расширение CLI для машинное обучение Azure](reference-azure-machine-learning-cli.md).