---
title: Обучение моделей keras для глубокого обучения
titleSuffix: Azure Machine Learning
description: Узнайте, как обучить и зарегистрировать модель классификации keras глубокой нейронной сети, работающую на TensorFlow с помощью Машинное обучение Azure.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: minxia
author: mx-iao
ms.reviewer: peterlu
ms.date: 09/28/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: 2b4af9dec2bf397ad2766c68d547eeac85a9a9a3
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102518370"
---
# <a name="train-keras-models-at-scale-with-azure-machine-learning"></a>Обучение моделей keras в масштабе с помощью Машинное обучение Azure

В этой статье вы узнаете, как выполнять сценарии обучения keras с Машинное обучение Azure.

В примере кода в этой статье показано, как обучить и зарегистрировать модель классификации keras, созданную с помощью серверной части TensorFlow, с Машинное обучение Azure. Он использует популярный [набор данных MNIST](http://yann.lecun.com/exdb/mnist/) для классификации рукописных цифр с помощью глубокой нейронной сети (DNN), созданной с помощью [библиотеки keras Python](https://keras.io) , работающей поверх [TensorFlow](https://www.tensorflow.org/overview).

Keras — это высокоуровневый API нейронной сети, который может работать над другими популярными DNN платформами для упрощения разработки. С Машинное обучение Azure можно быстро масштабировать задания обучения с помощью эластичных облачных ресурсов. Вы также можете отвестись к обучающим запускам, моделям версий, развертыванию моделей и т. д.

Независимо от того, разрабатываете ли вы модель keras с нуля или используете существующую модель в облаке, Машинное обучение Azure может помочь в создании моделей, готовых для производства.

> [!NOTE]
> Если вы используете API keras **tf. keras** , встроенный в TensorFlow, а не в автономный пакет keras, вместо этого см. статью, чтобы [обучить модели TensorFlow](how-to-train-tensorflow.md).

## <a name="prerequisites"></a>Предварительные условия

Запустите этот код в любой из этих сред:

- Вычислительная операция Машинного обучения Azure — загрузка или установка не требуется

     - Пройдите [руководство по настройке среды и рабочей области](tutorial-1st-experiment-sdk-setup.md), чтобы создать выделенный сервер записных книжек, предварительно загруженный с пакетом SDK и репозиторием с примером.
    - В папке Samples на сервере записной книжки найдите готовую и развернутую записную книжку, перейдя к этому каталогу: **практические советы по использованию azureml > машинного обучения ML-frameworks > keras > Learning-«Parameter-Learning-Deploy-with-keras** ».

 - Собственный сервер Jupyter Notebook

    - [Установите пакет SDK для машинное обучение Azure](/python/api/overview/azure/ml/install) (>= 1.15.0).
    - [Создайте файл конфигурации рабочей области](how-to-configure-environment.md#workspace).
    - [Загрузка примеров файлов скриптов](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks/keras/train-hyperparameter-tune-deploy-with-keras) `keras_mnist.py` перетаскивани `utils.py`

    Вы также можете найти завершенную [Jupyter Notebook версию](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/keras/train-hyperparameter-tune-deploy-with-keras/train-hyperparameter-tune-deploy-with-keras.ipynb) этого руководства на странице примеров GitHub. Записная книжка включает в себя развернутые разделы, охватывающие интеллектуальные настройки, развертывание моделей и мини-приложения записных книжек.

## <a name="set-up-the-experiment"></a>Настройка эксперимента

В этом разделе выполняется настройка обучающего эксперимента путем загрузки требуемых пакетов Python, инициализации рабочей области, создания Филедатасет для входных данных для обучения, создания целевого объекта вычислений и определения среды обучения.

### <a name="import-packages"></a>Импорт пакетов

Сначала импортируйте необходимые библиотеки Python.

```Python
import os
import azureml
from azureml.core import Experiment
from azureml.core import Environment
from azureml.core import Workspace, Run
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

Вы можете использовать `register()` метод для регистрации набора данных в рабочей области, чтобы предоставить общий доступ другим пользователям, повторно использовать их в различных экспериментах и называть их в обучающем скрипте.

```python
dataset = dataset.register(workspace=ws,
                           name='mnist-dataset',
                           description='training and test dataset',
                           create_new_version=True)
```

### <a name="create-a-compute-target"></a>Создание целевого объекта вычислений

Создайте целевой объект вычислений для выполнения задания обучения. В этом примере создайте кластерный Машинное обучение Azure вычислений с поддержкой GPU.

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

Определите [среду](concept-environments.md) машинного обучения Azure, в которой инкапсулируются зависимости сценария обучения.

Сначала определите зависимости conda в файле YAML. в этом примере файл называется `conda_dependencies.yml` .

```yaml
channels:
- conda-forge
dependencies:
- python=3.6.2
- pip:
  - azureml-defaults
  - tensorflow-gpu==2.0.0
  - keras<=2.3.1
  - matplotlib
```

Создайте среду машинного обучения Azure из этой спецификации среды conda. Среда будет упакована в контейнер DOCKER во время выполнения.

По умолчанию, если базовый образ не указан, Azure ML будет использовать образ ЦП в `azureml.core.environment.DEFAULT_CPU_IMAGE` качестве базового образа. Поскольку в этом примере выполняется обучение на кластере GPU, необходимо указать базовый образ GPU с необходимыми драйверами и зависимостями GPU. Служба машинного обучения Azure поддерживает набор базовых образов, опубликованных в реестре контейнеров Microsoft (мкр), которые можно использовать. Дополнительные сведения см. в репозитории GitHub [Azure/AzureML-Containers](https://github.com/Azure/AzureML-Containers) .

```python
keras_env = Environment.from_conda_specification(name='keras-env', file_path='conda_dependencies.yml')

# Specify a GPU base image
keras_env.docker.enabled = True
keras_env.docker.base_image = 'mcr.microsoft.com/azureml/openmpi3.1.2-cuda10.0-cudnn7-ubuntu18.04'
```

Дополнительные сведения о создании и использовании сред см. [в разделе Создание и использование программных сред в машинное обучение Azure](how-to-use-environments.md).

## <a name="configure-and-submit-your-training-run"></a>Настройка и отправка учебного запуска

### <a name="create-a-scriptrunconfig"></a>Создание Скриптрунконфиг
Сначала получите данные из хранилища данных рабочей области с помощью `Dataset` класса.

```python
dataset = Dataset.get_by_name(ws, 'mnist-dataset')

# list the files referenced by mnist-dataset
dataset.to_path()
```

Создайте объект ScriptRunConfig, чтобы указать сведения о конфигурации для вашего задания обучения, в том числе скрипт обучения, используемую среду и целевой объект вычислений, на котором будет выполняться задание.

Любые аргументы в скрипте обучения передаются через командную строку, если они указаны в `arguments` параметре. Датасетконсумптионконфиг для нашего Филедатасет передается в качестве аргумента в обучающий сценарий для `--data-folder` аргумента. МАШИНное обучение Azure разрешит эту Датасетконсумптионконфиг в точке подключения резервного хранилища данных, к которой затем можно получить доступ из обучающего скрипта.

```python
from azureml.core import ScriptRunConfig

args = ['--data-folder', dataset.as_mount(),
        '--batch-size', 50,
        '--first-layer-neurons', 300,
        '--second-layer-neurons', 100,
        '--learning-rate', 0.001]

src = ScriptRunConfig(source_directory=script_folder,
                      script='keras_mnist.py',
                      arguments=args,
                      compute_target=compute_target,
                      environment=keras_env)
```

Дополнительные сведения о настройке заданий с помощью Скриптрунконфиг см. в статье [Настройка и отправка обучающих запусков](how-to-set-up-training-targets.md).

> [!WARNING]
> Если вы ранее использовали оценщик TensorFlow для настройки заданий обучения keras, обратите внимание, что средства оценки не рекомендуются к выпуску пакета SDK 1.19.0. С помощью пакета SDK для машинного обучения Azure >= 1.15.0, Скриптрунконфиг является рекомендуемым способом настройки заданий обучения, включая те, которые используют платформы глубокого обучения. Распространенные вопросы по переносу см. в разделе [скриптрунконфиг Guide with Migration](how-to-migrate-from-estimators-to-scriptrunconfig.md).

### <a name="submit-your-run"></a>Отправка выполнения

[Объект Run](/python/api/azureml-core/azureml.core.run%28class%29) предоставляет интерфейс для журнала выполнения во время выполнения задания и после его завершения.

```Python
run = Experiment(workspace=ws, name='Tutorial-Keras-Minst').submit(src)
run.wait_for_completion(show_output=True)
```

### <a name="what-happens-during-run-execution"></a>Что происходит во время выполнения
При выполнении выполнения он проходит следующие этапы.

- **Подготовка**. образ DOCKER создается в соответствии с определенной средой. Образ отправляется в реестр контейнеров рабочей области и кэшируется для последующего выполнения. Журналы также передаются в журнал выполнения и могут быть просмотрены для отслеживания хода выполнения. Если вместо этого указана проверенная среда, то будет использоваться кэшированный образ, в котором выполняется резервное копирование проверенной среды.

- **Масштабирование**. Кластер пытается выполнить масштабирование, если кластеру Batch AI требуется больше узлов для выполнения выполнения, чем в настоящее время доступно.

- **Выполняется**: все скрипты в папке Script передаются в целевой объект вычислений, хранилища данных подключаются или копируются, и выполняется `script` . Выходные данные из STDOUT и папки **/логс** передаются в журнал выполнения и могут использоваться для наблюдения за выполнением.

- **Пост-обработка**. папка **/Outputs** для выполнения копируется в журнал выполнения.

## <a name="register-the-model"></a>Регистрация модели

После обучения модели ее можно зарегистрировать в рабочей области. Регистрация модели позволяет хранить и редактировать версии моделей в рабочей области для упрощения [развертывания и управления моделями](concept-model-management-and-deployment.md).

```Python
model = run.register_model(model_name='keras-mnist', model_path='outputs/model')
```

> [!TIP]
> Руководство по развертыванию содержит раздел, посвященный регистрации моделей, но можно сразу перейти к [созданию целевого объекта вычислений](how-to-deploy-and-where.md#choose-a-compute-target) для развертывания, так как у вас уже есть Зарегистрированная модель.

Также можно скачать локальную копию модели. Это может быть полезно для выполнения дополнительной проверки модели локально. В скрипте обучения `keras_mnist.py` объект TensorFlow хранителя сохраняет модель в локальной папке (локальную для целевого объекта вычислений). Для скачивания копии из журнала выполнения можно использовать объект Run.

```Python
# Create a model folder in the current directory
os.makedirs('./model', exist_ok=True)

for f in run.get_file_names():
    if f.startswith('outputs/model'):
        output_file_path = os.path.join('./model', f.split('/')[-1])
        print('Downloading from {} to {} ...'.format(f, output_file_path))
        run.download_file(name=f, output_file_path=output_file_path)
```

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы обучили и зарегистрировали модель keras на Машинное обучение Azure. Чтобы узнать, как развернуть модель, перейдите к статье о развертывании модели.

* [Как и где развертываются модели](how-to-deploy-and-where.md)
* [Отслеживание метрик выполнения во время обучения](how-to-track-experiments.md)
* [Настройка гиперпараметров](how-to-tune-hyperparameters.md)
* [Развертывание обученной модели](how-to-deploy-and-where.md)
* [Эталонная архитектура для распределенного обучения глубокого обучения в Azure](/azure/architecture/reference-architectures/ai/training-deep-learning)
