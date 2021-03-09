---
title: Обучение и развертывание модели TensorFlow
titleSuffix: Azure Machine Learning
description: Узнайте, как Машинное обучение Azure позволяет масштабировать задание обучения TensorFlow с помощью эластичных облачных ресурсов.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: minxia
author: mx-iao
ms.date: 09/28/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: 583f588004f41fc07037e7f5e4ce75538a581c70
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102518336"
---
# <a name="train-tensorflow-models-at-scale-with-azure-machine-learning"></a>Обучение моделей TensorFlow в масштабе с помощью Машинное обучение Azure

Из этой статьи вы узнаете, как выполнять сценарии обучения [TensorFlow](https://www.tensorflow.org/overview) в масштабе с помощью машинное обучение Azure.

В этом примере производится обучение и регистрация модели TensorFlow для классификации рукописных цифр с помощью глубокой нейронной сети (DNN).

Независимо от того, разрабатываете ли вы модель TensorFlow с нуля или используете [существующую модель](how-to-deploy-existing-model.md) в облаке, вы можете использовать машинное обучение Azure для масштабирования заданий обучения с открытым исходным кодом для создания, развертывания, контроля версий и мониторинга моделей производственного уровня.

## <a name="prerequisites"></a>Предварительные условия

Запустите этот код в любой из этих сред:

 - Вычислительная операция Машинного обучения Azure — загрузка или установка не требуется

     - Пройдите [руководство по настройке среды и рабочей области](tutorial-1st-experiment-sdk-setup.md), чтобы создать выделенный сервер записных книжек, предварительно загруженный с пакетом SDK и репозиторием с примером.
    - В папке примеры глубокого обучения на сервере записной книжки найдите готовую и развернутую записную книжку, перейдя к этому каталогу: **практические советы по использованию azureml > ML-frameworks > tensorflow > обучения-параметры-Настройка-Deploy-with-tensorflow** Folder. 
 
 - Собственный сервер Jupyter Notebook

    - [Установите пакет SDK для машинное обучение Azure](/python/api/overview/azure/ml/install) (>= 1.15.0).
    - [Создайте файл конфигурации рабочей области](how-to-configure-environment.md#workspace).
    - [Загрузка примеров файлов скриптов](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks/tensorflow/train-hyperparameter-tune-deploy-with-tensorflow) `tf_mnist.py` перетаскивани `utils.py`
     
    Вы также можете найти завершенную [Jupyter Notebook версию](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/tensorflow/train-hyperparameter-tune-deploy-with-tensorflow/train-hyperparameter-tune-deploy-with-tensorflow.ipynb) этого руководства на странице примеров GitHub. Записная книжка включает в себя развернутые разделы, охватывающие интеллектуальные настройки, развертывание моделей и мини-приложения записных книжек.

## <a name="set-up-the-experiment"></a>Настройка эксперимента

В этом разделе выполняется настройка обучающего эксперимента путем загрузки требуемых пакетов Python, инициализации рабочей области, создания целевого объекта вычислений и определения среды обучения.

### <a name="import-packages"></a>Импорт пакетов

Сначала импортируйте необходимые библиотеки Python.

```Python
import os
import urllib
import shutil
import azureml

from azureml.core import Experiment
from azureml.core import Workspace, Run
from azureml.core import Environment

from azureml.core.compute import ComputeTarget, AmlCompute
from azureml.core.compute_target import ComputeTargetException
```

### <a name="initialize-a-workspace"></a>Инициализация рабочей области

[Машинное обучение Azure Рабочая область](concept-workspace.md) — это ресурс верхнего уровня для службы. Она предоставляет централизованное расположение для работы со всеми создаваемыми артефактами. В пакете SDK для Python можно получить доступ к артефактам рабочей области, создав [`workspace`](/python/api/azureml-core/azureml.core.workspace.workspace) объект.

Создайте объект рабочей области из `config.json` файла, созданного в [разделе Предварительные требования](#prerequisites).

```Python
ws = Workspace.from_config()
```

### <a name="create-a-file-dataset"></a>Создание файлового набора данных

Объект `FileDataset` ссылается на один или несколько файлов в хранилище данных рабочей области или общедоступных URL-адресов. Это могут быть файлы любого формата, и с помощью этого класса вы сможете скачивать их или подключать к вычислительной среде. Создавая `FileDataset`, вы одновременно создаете ссылку на расположение источника данных. Любые преобразования, примененные к такому набору данных, будут сохранены и в исходном наборе данных. Данные хранятся только в исходном расположении, а значит не потребуется лишних расходов на хранение. Дополнительные сведения см. в соответствующем [практическом руководстве](./how-to-create-register-datasets.md) по пакету `Dataset`.

```python
from azureml.core.dataset import Dataset

web_paths = [
            'http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/train-labels-idx1-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz'
            ]
dataset = Dataset.File.from_files(path=web_paths)
```

Используйте `register()` метод, чтобы зарегистрировать набор данных в рабочей области, чтобы предоставить общий доступ другим пользователям, повторно использовать их в различных экспериментах и называть их по имени в обучающем скрипте.

```python
dataset = dataset.register(workspace=ws,
                           name='mnist-dataset',
                           description='training and test dataset',
                           create_new_version=True)

# list the files referenced by dataset
dataset.to_path()
```

### <a name="create-a-compute-target"></a>Создание целевого объекта вычислений

Создайте целевой объект вычислений для выполнения задания TensorFlow. В этом примере создайте кластерный Машинное обучение Azure вычислений с поддержкой GPU.

```Python
cluster_name = "gpu-cluster"

try:
    compute_target = ComputeTarget(workspace=ws, name=cluster_name)
    print('Found existing compute target')
except ComputeTargetException:
    print('Creating a new compute target...')
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_NC6', 
                                                           max_nodes=4)

    compute_target = ComputeTarget.create(ws, cluster_name, compute_config)

    compute_target.wait_for_completion(show_output=True, min_node_count=None, timeout_in_minutes=20)
```

[!INCLUDE [low-pri-note](../../includes/machine-learning-low-pri-vm.md)]

Дополнительные сведения о целевых объектах вычислений см. в статье [что такое целевые показатели вычислений](concept-compute-target.md) .

### <a name="define-your-environment"></a>Определение среды

Чтобы определить [среду](concept-environments.md) машинного обучения Azure, которая инкапсулирует зависимости сценария обучения, можно либо определить пользовательскую среду, либо использовать проверенную среду машинного обучения Azure.

#### <a name="use-a-curated-environment"></a>Использование проверенной среды
Azure ML предоставляет готовые, проверенные среды, если вы не хотите определять собственную среду. В МАШИНном обучении Azure имеется несколько проверенных сред ЦП и GPU для TensorFlow, соответствующих разным версиям TensorFlow. Дополнительные сведения см. [здесь](resource-curated-environments.md).

Если вы хотите использовать проверенную среду, то вместо этого можно выполнить следующую команду:

```python
curated_env_name = 'AzureML-TensorFlow-2.2-GPU'
tf_env = Environment.get(workspace=ws, name=curated_env_name)
```

Чтобы просмотреть пакеты, входящие в проверенную среду, можно записать зависимости conda на диск:
```python
tf_env.save_to_directory(path=curated_env_name)
```

Убедитесь, что проверенная среда включает все зависимости, необходимые для обучающего скрипта. В противном случае необходимо будет изменить окружение, включив недостающие зависимости. Обратите внимание, что при изменении среды потребуется присвоить ей новое имя, так как префикс AzureML зарезервирован для проверенных сред. Если вы изменили файл YAML Dependencies conda, можно создать новое окружение с новым именем, например:
```python
tf_env = Environment.from_conda_specification(name='tensorflow-2.2-gpu', file_path='./conda_dependencies.yml')
```

Если вы изменили проверенный объект среды напрямую, вы можете клонировать эту среду с новым именем:
```python
tf_env = tf_env.clone(new_name='tensorflow-2.2-gpu')
```

#### <a name="create-a-custom-environment"></a>Создание пользовательской среды

Вы также можете создать собственную среду машинного обучения Azure, которая инкапсулирует зависимости обучающего сценария.

Сначала определите зависимости conda в файле YAML. в этом примере файл называется `conda_dependencies.yml` .

```yaml
channels:
- conda-forge
dependencies:
- python=3.6.2
- pip:
  - azureml-defaults
  - tensorflow-gpu==2.2.0
```

Создайте среду машинного обучения Azure из этой спецификации среды conda. Среда будет упакована в контейнер DOCKER во время выполнения.

По умолчанию, если базовый образ не указан, Azure ML будет использовать образ ЦП в `azureml.core.environment.DEFAULT_CPU_IMAGE` качестве базового образа. Поскольку в этом примере выполняется обучение на кластере GPU, необходимо указать базовый образ GPU с необходимыми драйверами и зависимостями GPU. Служба машинного обучения Azure поддерживает набор базовых образов, опубликованных в реестре контейнеров Microsoft (мкр), которые можно использовать. Дополнительные сведения см. в репозитории GitHub [Azure/AzureML-Containers](https://github.com/Azure/AzureML-Containers) .

```python
tf_env = Environment.from_conda_specification(name='tensorflow-2.2-gpu', file_path='./conda_dependencies.yml')

# Specify a GPU base image
tf_env.docker.enabled = True
tf_env.docker.base_image = 'mcr.microsoft.com/azureml/openmpi3.1.2-cuda10.1-cudnn7-ubuntu18.04'
```

> [!TIP]
> При необходимости можно просто захватить все зависимости непосредственно в пользовательском образе DOCKER или Dockerfile и создать среду из нее. Дополнительные сведения см. в разделе [обучение с помощью пользовательского образа](how-to-train-with-custom-image.md).

Дополнительные сведения о создании и использовании сред см. [в разделе Создание и использование программных сред в машинное обучение Azure](how-to-use-environments.md).

## <a name="configure-and-submit-your-training-run"></a>Настройка и отправка учебного запуска

### <a name="create-a-scriptrunconfig"></a>Создание Скриптрунконфиг

Создайте объект [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig), чтобы указать сведения о конфигурации для вашего задания обучения, в том числе скрипт обучения, используемую среду и целевой объект вычислений, на котором будет выполняться задание. Любые аргументы в скрипте обучения передаются через командную строку, если они указаны в `arguments` параметре.

```python
from azureml.core import ScriptRunConfig

args = ['--data-folder', dataset.as_mount(),
        '--batch-size', 64,
        '--first-layer-neurons', 256,
        '--second-layer-neurons', 128,
        '--learning-rate', 0.01]

src = ScriptRunConfig(source_directory=script_folder,
                      script='tf_mnist.py',
                      arguments=args,
                      compute_target=compute_target,
                      environment=tf_env)
```

> [!WARNING]
> Машинное обучение Azure запускает скрипты обучения, копируя весь исходный каталог. Если у вас есть конфиденциальные данные, которые не нужно передавать, используйте [файл. Ignore](how-to-save-write-experiment-files.md#storage-limits-of-experiment-snapshots) или не включайте его в исходный каталог. Вместо этого получите доступ к данным с помощью [набора данных](how-to-train-with-datasets.md)машинного обучения Azure.

Дополнительные сведения о настройке заданий с помощью Скриптрунконфиг см. в статье [Настройка и отправка обучающих запусков](how-to-set-up-training-targets.md).

> [!WARNING]
> Если вы ранее использовали оценщик TensorFlow для настройки заданий обучения TensorFlow, обратите внимание, что средства оценки не рекомендуются к выпуску пакета SDK 1.19.0. С помощью пакета SDK для машинного обучения Azure >= 1.15.0, Скриптрунконфиг является рекомендуемым способом настройки заданий обучения, включая те, которые используют платформы глубокого обучения. Распространенные вопросы по переносу см. в разделе [скриптрунконфиг Guide with Migration](how-to-migrate-from-estimators-to-scriptrunconfig.md).

### <a name="submit-a-run"></a>Отправка запроса на выполнение

[Объект Run](/python/api/azureml-core/azureml.core.run%28class%29) предоставляет интерфейс для журнала выполнения во время выполнения задания и после его завершения.

```Python
run = Experiment(workspace=ws, name='Tutorial-TF-Mnist').submit(src)
run.wait_for_completion(show_output=True)
```
### <a name="what-happens-during-run-execution"></a>Что происходит во время выполнения
При выполнении выполнения он проходит следующие этапы.

- **Подготовка**. образ DOCKER создается в соответствии с определенной средой. Образ отправляется в реестр контейнеров рабочей области и кэшируется для последующего выполнения. Журналы также передаются в журнал выполнения и могут быть просмотрены для отслеживания хода выполнения. Если вместо этого указана проверенная среда, то будет использоваться кэшированный образ, в котором выполняется резервное копирование проверенной среды.

- **Масштабирование**. Кластер пытается выполнить масштабирование, если кластеру Batch AI требуется больше узлов для выполнения выполнения, чем в настоящее время доступно.

- **Выполняется**: все скрипты в папке Script передаются в целевой объект вычислений, хранилища данных подключаются или копируются, и выполняется `script` . Выходные данные из STDOUT и папки **/логс** передаются в журнал выполнения и могут использоваться для наблюдения за выполнением.

- **Пост-обработка**. папка **/Outputs** для выполнения копируется в журнал выполнения.

## <a name="register-or-download-a-model"></a>Регистрация или скачивание модели

После обучения модели ее можно зарегистрировать в рабочей области. Регистрация модели позволяет хранить и редактировать версии моделей в рабочей области для упрощения [развертывания и управления моделями](concept-model-management-and-deployment.md). Необязательно: путем указания параметров `model_framework` , `model_framework_version` и `resource_configuration` , развертывание модели без кода станет доступным. Это позволяет напрямую развернуть модель в качестве веб-службы из зарегистрированной модели, а `ResourceConfiguration` объект определяет ресурс вычислений для веб-службы.

```Python
from azureml.core import Model
from azureml.core.resource_configuration import ResourceConfiguration

model = run.register_model(model_name='tf-mnist', 
                           model_path='outputs/model',
                           model_framework=Model.Framework.TENSORFLOW,
                           model_framework_version='2.0',
                           resource_configuration=ResourceConfiguration(cpu=1, memory_in_gb=0.5))
```

Вы также можете скачать локальную копию модели с помощью объекта Run. В сценарии обучения `tf_mnist.py` объект TensorFlow хранителя сохраняет модель в локальной папке (локальную для целевого объекта вычислений). Для скачивания копии можно использовать объект Run.

```Python
# Create a model folder in the current directory
os.makedirs('./model', exist_ok=True)
run.download_files(prefix='outputs/model', output_directory='./model', append_prefix=False)
```

## <a name="distributed-training"></a>Распределенное обучение

Машинное обучение Azure также поддерживает распределенные задания TensorFlow с несколькими узлами, чтобы можно было масштабировать рабочие нагрузки для обучения. Вы можете легко выполнять распределенные задания TensorFlow, и МАШИНное обучение Azure будет управлять согласованием.

Azure ML поддерживает выполнение распределенных заданий TensorFlow с помощью встроенного API-интерфейса распределенного обучения хоровод и TensorFlow.

### <a name="horovod"></a>Horovod
[Хоровод](https://github.com/uber/horovod) — это платформа с открытым исходным кодом, которая сокращает структуру для распределенного обучения, разработанного Uber. Она предлагает простой путь для написания распределенного кода TensorFlow для обучения.

Обучающий код должен быть оснащен хоровод для распределенного обучения. Дополнительные сведения об использовании хоровод с TensorFlow см. в документации по хоровод:

Дополнительные сведения об использовании хоровод с TensorFlow см. в документации по хоровод:

* [Хоровод с TensorFlow](https://github.com/horovod/horovod/blob/master/docs/tensorflow.rst)
* [Хоровод с API keras TensorFlow](https://github.com/horovod/horovod/blob/master/docs/keras.rst)

Кроме того, убедитесь, что в среде обучения включен пакет **хоровод** . Если вы используете TensorFlow проверенную среду, хоровод уже включен в качестве одной из зависимостей. Если вы используете собственную среду, убедитесь, что включена зависимость хоровод, например:

```yaml
channels:
- conda-forge
dependencies:
- python=3.6.2
- pip:
  - azureml-defaults
  - tensorflow-gpu==2.2.0
  - horovod==0.19.5
```

Чтобы выполнить распределенное задание с использованием MPI или хоровод в МАШИНном обучении Azure, необходимо указать [мпиконфигуратион](/python/api/azureml-core/azureml.core.runconfig.mpiconfiguration) в `distributed_job_config` параметре конструктора скриптрунконфиг. Приведенный ниже код настроит распределенное задание с 2 узлами, выполняющее один процесс на каждом узле. Если вы также хотите запустить несколько процессов на одном узле (т. е. Если номер SKU кластера имеет несколько GPU), дополнительно укажите `process_count_per_node` параметр в мпиконфигуратион (значение по умолчанию — `1` ).

```python
from azureml.core import ScriptRunConfig
from azureml.core.runconfig import MpiConfiguration

src = ScriptRunConfig(source_directory=project_folder,
                      script='tf_horovod_word2vec.py',
                      arguments=['--input_data', dataset.as_mount()],
                      compute_target=compute_target,
                      environment=tf_env,
                      distributed_job_config=MpiConfiguration(node_count=2))
```

Полный учебник по запуску распределенных TensorFlow с помощью хоровод в МАШИНном обучении Azure см. в разделе [Распределенная TensorFlow с хоровод](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/tensorflow/distributed-tensorflow-with-horovod).

### <a name="tfdistribute"></a>TF. распространение

Если вы используете [собственные распределенные TensorFlow](https://www.tensorflow.org/guide/distributed_training) в обучающем коде, например TensorFlow 2. x `tf.distribute.Strategy` , вы также можете запустить распределенное задание с помощью Azure ml. 

Для этого укажите [тенсорфловконфигуратион](/python/api/azureml-core/azureml.core.runconfig.tensorflowconfiguration) в `distributed_job_config` параметре конструктора скриптрунконфиг. Если используется `tf.distribute.experimental.MultiWorkerMirroredStrategy` , укажите `worker_count` в тенсорфловконфигуратион, соответствующем количеству узлов для задания обучения.

```python
import os
from azureml.core import ScriptRunConfig
from azureml.core.runconfig import TensorflowConfiguration

distr_config = TensorflowConfiguration(worker_count=2, parameter_server_count=0)

model_path = os.path.join("./outputs", "keras-model")

src = ScriptRunConfig(source_directory=source_dir,
                      script='train.py',
                      arguments=["--epochs", 30, "--model-dir", model_path],
                      compute_target=compute_target,
                      environment=tf_env,
                      distributed_job_config=distr_config)
```

В TensorFlow `TF_CONFIG` для обучения на нескольких компьютерах требуется переменная среды. МАШИНное обучение Azure настроит и установит `TF_CONFIG` переменную соответствующим образом для каждого рабочего процесса перед выполнением сценария обучения. Вы можете получить доступ `TF_CONFIG` из обучающего сценария, если вам нужно использовать `os.environ['TF_CONFIG']` .

Пример структуры `TF_CONFIG` Set на рабочем узле главного рабочего узла:
```JSON
TF_CONFIG='{
    "cluster": {
        "worker": ["host0:2222", "host1:2222"]
    },
    "task": {"type": "worker", "index": 0},
    "environment": "cloud"
}'
```

Если сценарий обучения использует стратегию сервера параметров для распределенного обучения, т. е. для устаревшего TensorFlow 1. x, необходимо также указать количество серверов параметров, которое будет использоваться в задании, например `distr_config = TensorflowConfiguration(worker_count=2, parameter_server_count=1)` .

## <a name="deploy-a-tensorflow-model"></a>Развертывание модели TensorFlow

Руководство по развертыванию содержит раздел, посвященный регистрации моделей, но можно сразу перейти к [созданию целевого объекта вычислений](how-to-deploy-and-where.md#choose-a-compute-target) для развертывания, так как у вас уже есть Зарегистрированная модель.

### <a name="preview-no-code-model-deployment"></a>Образца Развертывание модели без кода

Вместо традиционного маршрута развертывания можно также использовать функцию развертывания без кода (Предварительная версия) для TensorFlow. Зарегистрировав модель, как показано выше с помощью `model_framework` параметров, `model_framework_version` и `resource_configuration` , можно просто использовать `deploy()` статическую функцию для развертывания модели.

```python
service = Model.deploy(ws, "tensorflow-web-service", [model])
```

Полное [Описание процесса](how-to-deploy-and-where.md) развертывания в машинное обучение Azure более подробно.

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы обучили и зарегистрировали модель TensorFlow и узнали о вариантах развертывания. Дополнительные сведения о Машинное обучение Azure см. в других статьях.

* [Отслеживание метрик выполнения во время обучения](how-to-track-experiments.md)
* [Настройка гиперпараметров](how-to-tune-hyperparameters.md)
* [Эталонная архитектура для распределенного обучения глубокого обучения в Azure](/azure/architecture/reference-architectures/ai/training-deep-learning)
