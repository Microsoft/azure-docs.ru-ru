---
title: Обучение scikit. изучение моделей машинного обучения
titleSuffix: Azure Machine Learning
description: Узнайте, как Машинное обучение Azure позволяет масштабировать задание обучения scikit-учиться с помощью эластичных облачных ресурсов.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: jordane
author: jpe316
ms.date: 09/28/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 807174fdbede2e4631b3ca1df7220904038da4c8
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102518302"
---
# <a name="train-scikit-learn-models-at-scale-with-azure-machine-learning"></a>Обучение scikit. изучение моделей в масштабе с помощью Машинное обучение Azure

В этой статье вы узнаете, как выполнять scikitные сценарии обучения с помощью Машинное обучение Azure.

Примеры сценариев, приведенные в этой статье, используются для классификации изображений диафрагмы для создания модели машинного обучения на основе [набора данных диафрагмы](https://archive.ics.uci.edu/ml/datasets/iris)scikit.

Если вы научитесь изучать модель машинного обучения scikit-учиться с нуля или используете существующую модель в облаке, вы можете использовать Машинное обучение Azure для масштабирования заданий обучения с открытым исходным кодом с помощью эластичных облачных ресурсов. Вы можете создавать, развертывать, выполнять версии и отслеживать модели производственного уровня с помощью Машинное обучение Azure.

## <a name="prerequisites"></a>Предварительные условия

Запустите этот код в любой из этих сред:
 - Вычислительная операция Машинного обучения Azure — загрузка или установка не требуется

    - Выполните инструкции из [учебника Настройка среды и рабочей области](tutorial-1st-experiment-sdk-setup.md)  , чтобы создать выделенный сервер записной книжки, предварительно загруженный с помощью пакета SDK и примера репозитория.
    - В папке обучающие примеры на сервере записной книжки найдите готовую и развернутую записную книжку, перейдя к этому каталогу: **практические советы по использованию azureml > ML-frameworks > scikit-learning > Train-i Parameter-Learning-Deploy-with-sklearn** Folder.

 - Собственный сервер Jupyter Notebook

    - [Установите пакет SDK для машинное обучение Azure](/python/api/overview/azure/ml/install) (>= 1.13.0).
    - [Создайте файл конфигурации рабочей области](how-to-configure-environment.md#workspace).

## <a name="set-up-the-experiment"></a>Настройка эксперимента

В этом разделе выполняется настройка обучающего эксперимента путем загрузки требуемых пакетов Python, инициализации рабочей области, определения среды обучения и подготовки сценария обучения.

### <a name="initialize-a-workspace"></a>Инициализация рабочей области

[Машинное обучение Azure Рабочая область](concept-workspace.md) — это ресурс верхнего уровня для службы. Она предоставляет централизованное расположение для работы со всеми создаваемыми артефактами. В пакете SDK для Python можно получить доступ к артефактам рабочей области, создав [`workspace`](/python/api/azureml-core/azureml.core.workspace.workspace) объект.

Создайте объект рабочей области из `config.json` файла, созданного в [разделе Предварительные требования](#prerequisites).

```Python
from azureml.core import Workspace

ws = Workspace.from_config()
```

### <a name="prepare-scripts"></a>Подготовка скриптов

В этом руководстве обучающий сценарий **train_iris. Корректировка** уже предоставлен [здесь](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/scikit-learn/train-hyperparameter-tune-deploy-with-sklearn/train_iris.py). На практике вы можете использовать любой пользовательский сценарий обучения как есть и запускать его с помощью Azure ML без необходимости изменять код.

Примечания.
- В предоставленном сценарии обучения показано, как регистрировать некоторые метрики в МАШИНном обучении Azure с помощью `Run` объекта в скрипте.
- Предоставленный Скрипт обучения использует примеры данных из  `iris = datasets.load_iris()` функции.  Сведения об использовании и доступе к собственным данным см. в разделе [обучение с наборами](how-to-train-with-datasets.md) данных для обеспечения доступности данных во время обучения.

### <a name="define-your-environment"></a>Определение среды

Чтобы определить [среду](concept-environments.md) машинного обучения Azure, в которой будут инкапсулированы зависимости сценария обучения, можно определить пользовательскую среду или использовать среду, проверенную в машинном обучении Azure.

#### <a name="use-a-curated-environment"></a>Использование проверенной среды
Кроме того, Azure ML предоставляет предварительно созданные и проверенные среды, если вы не хотите определять собственную среду. Дополнительные сведения см. [здесь](resource-curated-environments.md).
Если вы хотите использовать проверенную среду, то вместо этого можно выполнить следующую команду:

```python
from azureml.core import Environment

sklearn_env = Environment.get(workspace=ws, name='AzureML-Tutorial')
```

#### <a name="create-a-custom-environment"></a>Создание пользовательской среды

Вы также можете создать собственную пользовательскую среду. Определите зависимости conda в файле YAML; в этом примере файл называется `conda_dependencies.yml` .

```yaml
dependencies:
  - python=3.6.2
  - scikit-learn
  - numpy
  - pip:
    - azureml-defaults
```

Создайте среду машинного обучения Azure из этой спецификации среды Conda. Среда будет упакована в контейнер DOCKER во время выполнения.
```python
from azureml.core import Environment

sklearn_env = Environment.from_conda_specification(name='sklearn-env', file_path='conda_dependencies.yml')
```

Дополнительные сведения о создании и использовании сред см. [в разделе Создание и использование программных сред в машинное обучение Azure](how-to-use-environments.md).

## <a name="configure-and-submit-your-training-run"></a>Настройка и отправка учебного запуска

### <a name="create-a-scriptrunconfig"></a>Создание Скриптрунконфиг
Создайте объект ScriptRunConfig, чтобы указать сведения о конфигурации для вашего задания обучения, в том числе скрипт обучения, используемую среду и целевой объект вычислений, на котором будет выполняться задание.
Любые аргументы в скрипте обучения передаются через командную строку, если они указаны в `arguments` параметре.

Следующий код настроит объект Скриптрунконфиг, чтобы отправить задание на выполнение на локальном компьютере.

```python
from azureml.core import ScriptRunConfig

src = ScriptRunConfig(source_directory='.',
                      script='train_iris.py',
                      arguments=['--kernel', 'linear', '--penalty', 1.0],
                      environment=sklearn_env)
```

Если вы хотите вместо этого запустить задание в удаленном кластере, можно указать требуемый целевой объект вычислений в `compute_target` параметре скриптрунконфиг.

```python
from azureml.core import ScriptRunConfig

compute_target = ws.compute_targets['<my-cluster-name>']
src = ScriptRunConfig(source_directory='.',
                      script='train_iris.py',
                      arguments=['--kernel', 'linear', '--penalty', 1.0],
                      compute_target=compute_target,
                      environment=sklearn_env)
```

### <a name="submit-your-run"></a>Отправка выполнения
```python
from azureml.core import Experiment

run = Experiment(ws,'Tutorial-TrainIRIS').submit(src)
run.wait_for_completion(show_output=True)
```

> [!WARNING]
> Машинное обучение Azure запускает скрипты обучения, копируя весь исходный каталог. Если у вас есть конфиденциальные данные, которые не нужно передавать, используйте [файл. Ignore](how-to-save-write-experiment-files.md#storage-limits-of-experiment-snapshots) или не включайте его в исходный каталог. Вместо этого получите доступ к данным с помощью [набора данных](how-to-train-with-datasets.md)машинного обучения Azure.

### <a name="what-happens-during-run-execution"></a>Что происходит во время выполнения
При выполнении выполнения он проходит следующие этапы.

- **Подготовка**. образ DOCKER создается в соответствии с определенной средой. Образ отправляется в реестр контейнеров рабочей области и кэшируется для последующего выполнения. Журналы также передаются в журнал выполнения и могут быть просмотрены для отслеживания хода выполнения. Если вместо этого указана проверенная среда, то будет использоваться кэшированный образ, в котором выполняется резервное копирование проверенной среды.

- **Масштабирование**. Кластер пытается выполнить масштабирование, если кластеру Batch AI требуется больше узлов для выполнения выполнения, чем в настоящее время доступно.

- **Выполняется**: все скрипты в папке Script передаются в целевой объект вычислений, хранилища данных подключаются или копируются, и выполняется `script` . Выходные данные из STDOUT и папки **/логс** передаются в журнал выполнения и могут использоваться для наблюдения за выполнением.

- **Пост-обработка**. папка **/Outputs** для выполнения копируется в журнал выполнения.

## <a name="save-and-register-the-model"></a>Сохранение и регистрация модели

После обучения модель можно сохранить и зарегистрировать в рабочей области. Регистрация модели позволяет хранить и редактировать версии моделей в рабочей области для упрощения [развертывания и управления моделями](concept-model-management-and-deployment.md).

Добавьте следующий код в сценарий обучения train_iris. корректировки, чтобы сохранить модель. 

``` Python
import joblib

joblib.dump(svm_model_linear, 'model.joblib')
```

Зарегистрируйте модель в рабочей области с помощью следующего кода. При указании параметров `model_framework` , `model_framework_version` и `resource_configuration` , развертывание модели без кода станет доступным. Развертывание модели без кода позволяет напрямую развернуть модель в качестве веб-службы из зарегистрированной модели, а [`ResourceConfiguration`](/python/api/azureml-core/azureml.core.resource_configuration.resourceconfiguration) объект определяет ресурс вычислений для веб-службы.

```Python
from azureml.core import Model
from azureml.core.resource_configuration import ResourceConfiguration

model = run.register_model(model_name='sklearn-iris', 
                           model_path='outputs/model.joblib',
                           model_framework=Model.Framework.SCIKITLEARN,
                           model_framework_version='0.19.1',
                           resource_configuration=ResourceConfiguration(cpu=1, memory_in_gb=0.5))
```

## <a name="deployment"></a>Развертывание

Только что зарегистрированная модель может быть развернута точно так же, как и любая другая Зарегистрированная модель в МАШИНном обучении Azure. Руководство по развертыванию содержит раздел, посвященный регистрации моделей, но можно сразу перейти к [созданию целевого объекта вычислений](how-to-deploy-and-where.md#choose-a-compute-target) для развертывания, так как у вас уже есть Зарегистрированная модель.

### <a name="preview-no-code-model-deployment"></a>Образца Развертывание модели без кода

Вместо традиционного маршрута развертывания можно также использовать функцию развертывания без кода (Предварительная версия) для scikit — обучение. Развертывание модели без кода поддерживается для всех встроенных типов моделей scikit-учиться. Зарегистрировав модель, как показано выше с помощью `model_framework` параметров, `model_framework_version` и `resource_configuration` , можно просто использовать [`deploy()`](/python/api/azureml-core/azureml.core.model%28class%29#deploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-) статическую функцию для развертывания модели.

```python
web_service = Model.deploy(ws, "scikit-learn-service", [model])
```

Примечание. Эти зависимости включены в предварительно построенный контейнер вывода scikit-учиться.

```yaml
    - azureml-defaults
    - inference-schema[numpy-support]
    - scikit-learn
    - numpy
```

Полное [Описание процесса](how-to-deploy-and-where.md) развертывания в машинное обучение Azure более подробно.


## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы обучили и зарегистрировали модель scikit-учиться и узнали о вариантах развертывания. Дополнительные сведения о Машинное обучение Azure см. в других статьях.

* [Отслеживание метрик выполнения во время обучения](how-to-track-experiments.md)
* [Настройка гиперпараметров](how-to-tune-hyperparameters.md)
