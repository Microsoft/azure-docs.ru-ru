---
title: Использование Apache Spark в конвейере машинного обучения (Предварительная версия)
titleSuffix: Azure Machine Learning
description: Свяжите рабочую область Azure синапсе Analytics с конвейером машинного обучения Azure, чтобы использовать Apache Spark для обработки данных.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: laobri
author: lobrien
ms.date: 03/04/2021
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: ea7dc30d0aed1350a8c9275d786ea22fa52c77bf
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102203697"
---
# <a name="how-to-use-apache-spark-powered-by-azure-synapse-analytics-in-your-machine-learning-pipeline-preview"></a>Использование Apache Spark (на платформе Azure синапсе Analytics) в конвейере машинного обучения (Предварительная версия)

В этой статье вы узнаете, как использовать пулы Apache Spark, включенные в Azure синапсе Analytics, в качестве целевого объекта вычислений для этапа подготовки данных в конвейере Машинное обучение Azure. Вы узнаете, как один конвейер может использовать ресурсы вычислений, подходящие для конкретного этапа, например для подготовки данных или обучения. Вы увидите, как подготовлены данные для шага Spark и как они передаются следующему шагу. 

## <a name="prerequisites"></a>Предварительные требования

* Создайте [рабочую область Машинного обучения Azure](how-to-manage-workspace.md) для хранения всех ресурсов конвейера.

* [Настройте среду разработки](how-to-configure-environment.md) , чтобы установить пакет sdk для машинное обучение Azure, или используйте [машинное обучение Azure вычислительный экземпляр](concept-compute-instance.md) с уже установленным пакетом SDK.

* Создайте рабочую область Azure синапсе Analytics и пул Apache Spark (см. раздел [Краткое руководство по созданию пула Apache Spark, использующего бессерверный пул, с помощью синапсе Studio](../synapse-analytics/quickstart-create-apache-spark-pool-studio.md)). 

## <a name="link-your-azure-machine-learning-workspace-and-azure-synapse-analytics-workspace"></a>Связывание рабочей области Машинное обучение Azure и рабочей области Azure синапсе Analytics 

Создание и администрирование пулов Apache Spark в рабочей области Azure синапсе Analytics. Чтобы интегрировать пул Apache Spark с рабочей областью Машинное обучение Azure, необходимо создать ссылку на рабочую область Azure синапсе Analytics. 

Пул Apache Spark можно подключить через пользовательский интерфейс Машинное обучение Azure Studio с помощью страницы **связанные службы** . Его также можно выполнить с помощью страницы **вычислений** с параметром **присоединить вычисление** .

Вы также можете подключить пул Apache Spark с помощью пакета SDK (как описано ниже) или с помощью шаблона ARM (см. [Пример шаблона ARM](https://github.com/Azure/azure-quickstart-templates/blob/master/101-machine-learning-linkedservice-create/azuredeploy.json)). 

Вы можете использовать командную строку, чтобы следовать шаблону ARM, добавить связанную службу и подключить пул Apache Spark с помощью следующего кода:

```bash
az deployment group create --name --resource-group <rg_name> --template-file "azuredeploy.json" --parameters @"azuredeploy.parameters.json"
```

> [!Important]
> Чтобы успешно установить связь с рабочей областью Azure синапсе Analytics, необходимо иметь роль владельца в ресурсе рабочей области Azure синапсе Analytics. Проверьте доступ в портал Azure.
> Связанная служба будет получать удостоверение, назначенное системой (САИ) при создании. Необходимо назначить этой службе ссылки роль "синапсе Apache Spark Administrator" из синапсе Studio, чтобы она могла отправить задание Spark (см. раздел [Управление назначениями РОЛЕЙ RBAC синапсе в синапсе Studio](../synapse-analytics/security/how-to-manage-synapse-rbac-role-assignments.md)). Кроме того, необходимо предоставить пользователю Машинное обучение Azure рабочей области роль "участник" от портал Azure управление ресурсами.

## <a name="create-or-retrieve-the-link-between-your-azure-synapse-analytics-workspace-and-your-azure-machine-learning-workspace"></a>Создание или получение связи между рабочей областью Azure синапсе Analytics и рабочей областью Машинное обучение Azure

Связанные службы в рабочей области можно получить с помощью кода, например:

```python
from azureml.core import Workspace, LinkedService, SynapseWorkspaceLinkedServiceConfiguration

ws = Workspace.from_config()

for service in LinkedService.list(ws) : 
    print(f"Service: {service}")

# Retrieve a known linked service
linked_service = LinkedService.get(ws, 'synapselink1')
```

`Workspace.from_config()`Во-первых, обращается к рабочей области машинное обучение Azure с помощью конфигурации в `config.json` (см. [учебник. Приступая к работе с машинное обучение Azure в среде разработки](tutorial-1st-experiment-sdk-setup-local.md)). Затем код выводит на печать все связанные службы, доступные в рабочей области. Наконец, `LinkedService.get()` извлекает связанную службу с именем `'synapselink1'` . 

## <a name="attach-your-apache-spark-pool-as-a-compute-target-for-azure-machine-learning"></a>Подключите пул Apache Spark в качестве целевого объекта вычислений для Машинное обучение Azure

Чтобы использовать пул Apache Spark для управления шагом в конвейере машинного обучения, необходимо присоединить его как `ComputeTarget` для этапа конвейера, как показано в следующем коде.

```python
from azureml.core.compute import SynapseCompute, ComputeTarget

attach_config = SynapseCompute.attach_configuration(
        linked_service = linked_service,
        type="SynapseSpark",
        pool_name="spark01") # This name comes from your Synapse workspace

synapse_compute=ComputeTarget.attach(
        workspace=ws,
        name='link1-spark01',
        attach_configuration=attach_config)

synapse_compute.wait_for_completion()
```

Первым шагом является настройка `SynapseCompute` . `linked_service`Аргумент — это `LinkedService` объект, созданный или полученный на предыдущем шаге. `type`Аргумент должен иметь значение `SynapseSpark` . `pool_name`Аргумент в `SynapseCompute.attach_configuration()` должен соответствовать существующему пулу в рабочей области Azure синапсе Analytics. Дополнительные сведения о создании пула Apache Spark в рабочей области Azure синапсе Analytics см. в разделе [Краткое руководство. Создание бессерверного пула Apache Spark с помощью синапсе Studio](../synapse-analytics/quickstart-create-apache-spark-pool-studio.md). `attach_config` имеет тип `ComputeTargetAttachConfiguration`.

После создания конфигурации можно создать машинное обучение, `ComputeTarget` передав в него `Workspace` имя, и, `ComputeTargetAttachConfiguration` по которому вы хотите ссылаться на вычисление в рабочей области машинного обучения. Вызов `ComputeTarget.attach()` является асинхронным, поэтому образец блокируется до завершения вызова.

## <a name="create-a-synapsesparkstep-that-uses-the-linked-apache-spark-pool"></a>Создание `SynapseSparkStep` , использующее связанный пул Apache Spark

Данные проходят в конвейер машинного обучения с помощью `DatasetConsumptionConfig` объектов, которые могут содержать табличные данные или наборы файлов. Данные часто поступают из файлов в хранилище BLOB-объектов в хранилище данных рабочей области. В следующем коде показан типичный код для создания входных данных для конвейера машинного обучения.

```python
from azureml.core import Dataset

datastore = ws.get_default_datastore()
file_name = 'Titanic.csv'

titanic_tabular_dataset = Dataset.Tabular.from_delimited_files(path=[(datastore, file_name)])
step1_input1 = titanic_tabular_dataset.as_named_input("tabular_input")

# Example only: it wouldn't make sense to duplicate input data, especially one as tabular and the other as files
titanic_file_dataset = Dataset.File.from_files(path=[(datastore, file_name)])
step1_input2 = titanic_file_dataset.as_named_input("file_input").as_hdfs()
```

В приведенном выше коде предполагается, что файл `Titanic.csv` находится в хранилище BLOB-объектов. В коде показано, как считать файл как `TabularDataset` и `FileDataset` . Этот код предназначен только для демонстрационных целей, так как это приведет к путанице при дублировании входных данных или интерпретации одного источника в виде как ресурса, содержащего таблицу, так и файла.

> [!Important]
> Чтобы использовать в `FileDataset` качестве входных данных, ваша `azureml-core` версия должна быть не меньше `1.20.0` . Сведения о том, как это указать с помощью `Environment` класса, обсуждаются ниже.

По завершении шага можно сохранить выходные данные с помощью кода, аналогичного следующему:

```python
from azureml.data import HDFSOutputDatasetConfig
step1_output = HDFSOutputDatasetConfig(destination=(datastore,"test")).register_on_complete(name="registered_dataset")
```

В этом случае данные будут храниться в в `datastore` файле `test` с именем и будут доступны в рабочей области машинного обучения в виде `Dataset` с именем `registered_dataset` .

Помимо данных, этап конвейера может содержать зависимости Python для каждого шага. Отдельные `SynapseSparkStep` объекты могут также указывать точную конфигурацию Apache Spark Azure синапсе. Это показано в следующем коде, который указывает, что `azureml-core` версия пакета должна быть не ниже `1.20.0` . (Как упоминалось ранее, это требование для `azureml-core` использования в `FileDataset` качестве входных данных.)

```python
from azureml.core.environment import Environment
from azureml.pipeline.steps import SynapseSparkStep

env = Environment(name="myenv")
env.python.conda_dependencies.add_pip_package("azureml-core>=1.20.0")

step_1 = SynapseSparkStep(name = 'synapse-spark',
                          file = 'dataprep.py',
                          source_directory="./code", 
                          inputs=[step1_input1, step1_input2],
                          outputs=[step1_output],
                          arguments = ["--tabular_input", step1_input1, 
                                       "--file_input", step1_input2,
                                       "--output_dir", step1_output],
                          compute_target = 'link1-spark01',
                          driver_memory = "7g",
                          driver_cores = 4,
                          executor_memory = "7g",
                          executor_cores = 2,
                          num_executors = 1,
                          environment = env)
```

Приведенный выше код задает один шаг в конвейере машинного обучения Azure. Этот шаг `environment` указывает конкретную `azureml-core` версию и при необходимости может добавить другие зависимости conda или PIP.

`SynapseSparkStep`Будет ZIP-файл и отправлен с локального компьютера в подкаталог `./code` . Этот каталог будет создан повторно на сервере вычислений, а на шаге будет выполнен файл `dataprep.py` из этого каталога. `inputs`И на `outputs` этом шаге были `step1_input1` `step1_input2` `step1_output` рассмотрены объекты, и, описанные выше. Самым простым способом доступа к этим значениям в `dataprep.py` сценарии является связывание их с именем `arguments` .

Следующий набор аргументов для `SynapseSparkStep` элемента управления конструктора Apache Spark. `compute_target`— Это то `'link1-spark01'` , что мы присоединенные к целевому объекту вычислений ранее. Другие параметры указывают память и ядра, которые мы хотим использовать.

В примере записной книжки используется следующий код `dataprep.py` :

```python
import os
import sys
import azureml.core
from pyspark.sql import SparkSession
from azureml.core import Run, Dataset

print(azureml.core.VERSION)
print(os.environ)

import argparse
parser = argparse.ArgumentParser()
parser.add_argument("--tabular_input")
parser.add_argument("--file_input")
parser.add_argument("--output_dir")
args = parser.parse_args()

# use dataset sdk to read tabular dataset
run_context = Run.get_context()
dataset = Dataset.get_by_id(run_context.experiment.workspace,id=args.tabular_input)
sdf = dataset.to_spark_dataframe()
sdf.show()

# use hdfs path to read file dataset
spark= SparkSession.builder.getOrCreate()
sdf = spark.read.option("header", "true").csv(args.file_input)
sdf.show()

sdf.coalesce(1).write\
.option("header", "true")\
.mode("append")\
.csv(args.output_dir)
```

Этот сценарий «подготовки данных» не выполняет никаких преобразований реальных данных, но показывает, как получить данные, преобразовать их в таблицу данных Spark и как выполнить некоторые базовые Apache Spark манипуляции. Выходные данные можно найти в Машинное обучение Azure Studio, открыв дочерний запуск, выбрав вкладку **выходные данные + журналы** и открыв `logs/azureml/driver/stdout` файл, как показано на следующем рисунке.

:::image type="content" source="media/how-to-use-synapsesparkstep/synapsesparkstep-stdout.png" alt-text="Снимок экрана: вкладка &quot;stdout&quot; для дочернего запуска":::

## <a name="use-the-synapsesparkstep-in-a-pipeline"></a>Использование `SynapseSparkStep` в конвейере

Другие шаги в конвейере могут иметь собственные уникальные среды и выполняться на разных вычислительных ресурсах, соответствующих поставленной задаче. Пример записной книжки запускает "шаг обучения" в небольшом кластере ЦП:

```python
from azureml.core.compute import AmlCompute

cpu_cluster_name = "cpucluster"

if cpu_cluster_name in ws.compute_targets:
    cpu_cluster = ComputeTarget(workspace=ws, name=cpu_cluster_name)
    print('Found existing cluster, use it.')
else:
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2', max_nodes=1)
    cpu_cluster = ComputeTarget.create(ws, cpu_cluster_name, compute_config)
    print('Allocating new CPU compute cluster')

cpu_cluster.wait_for_completion(show_output=True)

step2_input = step1_output.as_input("step2_input").as_download()

step_2 = PythonScriptStep(script_name="train.py",
                          arguments=[step2_input],
                          inputs=[step2_input],
                          compute_target=cpu_cluster_name,
                          source_directory="./code",
                          allow_reuse=False)
```

Приведенный выше код создает при необходимости новый ресурс вычислений. Затем `step1_output` результат преобразуется в входные данные для этапа обучения. `as_download()`Параметр означает, что данные будут перемещены на ресурс вычислений, что приведет к более быстрому доступу. Если данные настолько велики, что они не помещаются на локальном жестком диске вычислений, можно использовать `as_mount()` параметр для потоковой передачи данных с помощью файловой системы предохранителя. Это `compute_target` второй шаг `'cpucluster'` , а не `'link1-spark01'` ресурс, использованный на этапе подготовки данных. На этом шаге используется простая программа `train.py` вместо того, что `dataprep.py` использовалось на предыдущем шаге. Подробные сведения см `train.py` . в примере записной книжки.

Определив все шаги, можно создать и запустить конвейер. 

```python
from azureml.pipeline.core import Pipeline

pipeline = Pipeline(workspace=ws, steps=[step_1, step_2])
pipeline_run = pipeline.submit('synapse-pipeline', regenerate_outputs=True)
```

Приведенный выше код создает конвейер, состоящий из этапа подготовки данных на Apache Spark пулах на базе Azure синапсе Analytics ( `step_1` ) и этапа обучения ( `step_2` ). Azure вычисляет диаграмму выполнения, изучая зависимости данных между шагами. В этом случае существует только простая зависимость, которая `step2_input` обязательно требуется `step1_output` .

Вызов метода `pipeline.submit` создает при необходимости эксперимент с именем `synapse-pipeline` и асинхронно начинает выполнение внутри него. Отдельные шаги в конвейере выполняются как дочерние запуски этого главного запуска, и их можно отслеживать и просматривать на странице эксперименты в студии.

## <a name="next-steps"></a>Дальнейшие действия

* [Публикация и мониторинг конвейеров машинного обучения](how-to-deploy-pipelines.md) 
* [Мониторинг Машинного обучения Azure](monitor-azure-machine-learning.md)
* [Использование автоматизированного ML в конвейере Машинное обучение Azure в Python](how-to-use-automlstep-in-pipelines.md)
