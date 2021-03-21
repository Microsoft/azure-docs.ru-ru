---
title: Использование автоматизированного ML в конвейерах МАШИНного обучения
titleSuffix: Azure Machine Learning
description: Аутомлстеп позволяет использовать автоматизированное машинное обучение в конвейерах.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: laobri
author: lobrien
manager: cgronlun
ms.date: 02/28/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, automl
ms.openlocfilehash: 0de3c9a7cf464f38a1a12d8bc19451fb1158a5ad
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102520512"
---
# <a name="use-automated-ml-in-an-azure-machine-learning-pipeline-in-python"></a>Использование автоматизированного ML в конвейере Машинное обучение Azure в Python


Функция автоматического выполнения машинного обучения Машинное обучение Azure позволяет обнаруживать высокопроизводительные модели без повторной реализации всех возможных подходов. В сочетании с конвейерами Машинное обучение Azure можно создавать развертываемые рабочие процессы, которые позволяют быстро обнаружить алгоритм, который лучше всего подходит для ваших данных. В этой статье показано, как эффективно присоединиться к этапу подготовки данных к автоматическому этапу ML. С помощью автоматизированного ML можно быстро обнаружить оптимальный алгоритм, который лучше всего подходит для ваших данных, в то же время размещаясь в пути к Млопс и эксплуатации жизненного цикла модели с конвейерами.

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. Если у вас еще нет подписки Azure, создайте бесплатную учетную запись, прежде чем начинать работу. Опробуйте [бесплатную или платную версию Машинного обучения Azure](https://aka.ms/AMLFree) уже сегодня.

* Рабочая область машинного обучения Azure. См. сведения о [создании рабочей области машинного обучения Azure](how-to-manage-workspace.md).  

* Знакомство с [автоматизированными средствами машинного обучения](concept-automated-ml.md) и пакетами SDK для [конвейеров машинного обучения](concept-ml-pipelines.md) Azure.

## <a name="review-automated-mls-central-classes"></a>Обзор центральных классов автоматизированного машинного обучения

Автоматизированное МАШИНное обучение в конвейере представлено `AutoMLStep` объектом. Класс `AutoMLStep` является подклассом `PipelineStep`. Граф `PipelineStep` объектов определяет `Pipeline` .

Существует несколько подклассов `PipelineStep` . Кроме того, в `AutoMLStep` этой статье будет показана `PythonScriptStep` Подготовка данных и другая для регистрации модели.

Предпочтительный способ первоначального перемещения данных _в_ конвейер машинного обучения — с `Dataset` объектами. Чтобы перемещать данные _между_ шагами и возможными сохранение выходных данных из запусков, предпочтительным способом является [`OutputFileDatasetConfig`](/python/api/azureml-core/azureml.data.outputfiledatasetconfig) [`OutputTabularDatasetConfig`](/python/api/azureml-core/azureml.data.output_dataset_config.outputtabulardatasetconfig) объект и. Для использования с `AutoMLStep` `PipelineData` объект должен быть преобразован в `PipelineOutputTabularDataset` объект. Дополнительные сведения см. [в статье входные и выходные данные из конвейеров ML](how-to-move-data-in-out-of-pipelines.md).

`AutoMLStep`Объект настраивается с помощью `AutoMLConfig` объекта. `AutoMLConfig` является гибким классом, как описано в разделе [Настройка автоматизированных экспериментов машинного обучения в Python](./how-to-configure-auto-train.md#configure-your-experiment-settings). 

`Pipeline`Выполняется в `Experiment` . Конвейер `Run` имеет для каждого этапа дочерний элемент `StepRun` . Выходные данные автоматизированного ML `StepRun` — это обучающие показатели и модель с наивысшей производительностью.

Чтобы сделать конкретную задачу, в этой статье создается простой конвейер для задачи классификации. Задача прогнозирует Титанике выживание, но мы не будем обсуждать данные или задачу, за исключением передачи.

## <a name="get-started"></a>Начало работы

### <a name="retrieve-initial-dataset"></a>Получить исходный набор данных

Часто рабочий процесс машинного обучения начинается с уже существующих базовых данных. Это хороший сценарий для зарегистрированного набора данных. Наборы данных видны в рабочей области, поддерживают управление версиями и могут быть интерактивно исследованы. Существует множество способов создания и заполнения набора данных, как описано в разделе [create машинное обучение Azure DataSets](how-to-create-register-datasets.md). Так как мы будем использовать пакет SDK для Python для создания конвейера, используйте пакет SDK для загрузки базовых данных и зарегистрируйте его с именем "titanic_ds".

```python
from azureml.core import Workspace, Dataset

ws = Workspace.from_config()
if not 'titanic_ds' in ws.datasets.keys() :
    # create a TabularDataset from Titanic training data
    web_paths = ['https://dprepdata.blob.core.windows.net/demo/Titanic.csv',
                 'https://dprepdata.blob.core.windows.net/demo/Titanic2.csv']
    titanic_ds = Dataset.Tabular.from_delimited_files(path=web_paths)

    titanic_ds.register(workspace = ws,
                                     name = 'titanic_ds',
                                     description = 'Titanic baseline data',
                                     create_new_version = True)

titanic_ds = Dataset.get_by_name(ws, 'titanic_ds')
```

Сначала код входит в рабочую область Машинное обучение Azure, определенную в **config.json** (описание см. в разделе [Создание файла конфигурации рабочей области](how-to-configure-environment.md#workspace). Если уже нет набора данных с именем `'titanic_ds'` Registered, он создается. Код загружает CSV-данные из Интернета, использует их для создания экземпляра, `TabularDataset` а затем регистрирует набор данных в рабочей области. Наконец, функция `Dataset.get_by_name()` присваивает параметру значение `Dataset` `titanic_ds` . 

### <a name="configure-your-storage-and-compute-target"></a>Настройка хранилища и целевого объекта вычислений

Дополнительные ресурсы, которые понадобятся конвейеру, — хранилище и, как правило, Машинное обучение Azureные ресурсы вычислений. 

```python
from azureml.core import Datastore
from azureml.core.compute import AmlCompute, ComputeTarget

datastore = ws.get_default_datastore()

compute_name = 'cpu-cluster'
if not compute_name in ws.compute_targets :
    print('creating a new compute target...')
    provisioning_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2',
                                                                min_nodes=0,
                                                                max_nodes=1)
    compute_target = ComputeTarget.create(ws, compute_name, provisioning_config)

    compute_target.wait_for_completion(
        show_output=True, min_node_count=None, timeout_in_minutes=20)

    # Show the result
    print(compute_target.get_status().serialize())

compute_target = ws.compute_targets[compute_name]
```

[!INCLUDE [low-pri-note](../../includes/machine-learning-low-pri-vm.md)]

Промежуточные данные между подготовкой данных и автоматическим этапом ML можно хранить в хранилище данных по умолчанию рабочей области, поэтому нам не нужно выполнять больше вызовов `get_default_datastore()` для `Workspace` объекта. 

После этого код проверяет, существует ли целевой объект вычислений AML `'cpu-cluster'` . В противном случае мы указываем, что требуется небольшой вычислительный ресурс на основе ЦП. Если вы планируете использовать функции глубокого обучения ML (например, текстовые Добавление признаков с поддержкой DNN), следует выбрать вычислительную систему с поддержкой поддержки GPU, как описано в статье [размеры виртуальных машин, оптимизированных для GPU](../virtual-machines/sizes-gpu.md). 

Код блокируется до подготовки целевого объекта, а затем выводит некоторые сведения о только что созданном целевом объекте вычислений. Наконец, именованный целевой объект вычислений извлекается из рабочей области и назначается `compute_target` . 

### <a name="configure-the-training-run"></a>Настройка обучающего запуска

Аутомлстеп автоматически настраивает свои зависимости во время отправки задания. Контекст среды выполнения задается путем создания и настройки `RunConfiguration` объекта. Здесь мы устанавливаем целевой объект вычислений.

```python
from azureml.core.runconfig import RunConfiguration

aml_run_config = RunConfiguration()
# Use just-specified compute target ("cpu-cluster")
aml_run_config.target = compute_target
```

## <a name="prepare-data-for-automated-machine-learning"></a>Подготовка данных для автоматического машинного обучения

### <a name="write-the-data-preparation-code"></a>Написание кода подготовки данных

Базовый набор данных Титанике состоит из смешанных числовых и текстовых данных, при этом некоторые значения отсутствуют. Чтобы подготовить его к автоматическому машинному обучению, этап конвейера подготовки данных будет следующим:

- Заполнение отсутствующих данных либо случайными данными, либо категорией, соответствующей "Неизвестный"
- Преобразование данных категорий в целые числа
- Удалить столбцы, которые не планируется использовать
- Разделение данных на обучающие и проверочные наборы
- Запись преобразованных данных в `OutputFileDatasetConfig` выходные пути

```python
%%writefile dataprep.py
from azureml.core import Run

import pandas as pd 
import numpy as np 
import argparse

RANDOM_SEED=42

def prepare_age(df):
    # Fill in missing Age values from distribution of present Age values 
    mean = df["Age"].mean()
    std = df["Age"].std()
    is_null = df["Age"].isnull().sum()
    # compute enough (== is_null().sum()) random numbers between the mean, std
    rand_age = np.random.randint(mean - std, mean + std, size = is_null)
    # fill NaN values in Age column with random values generated
    age_slice = df["Age"].copy()
    age_slice[np.isnan(age_slice)] = rand_age
    df["Age"] = age_slice
    df["Age"] = df["Age"].astype(int)
    
    # Quantize age into 5 classes
    df['Age_Group'] = pd.qcut(df['Age'],5, labels=False)
    df.drop(['Age'], axis=1, inplace=True)
    return df

def prepare_fare(df):
    df['Fare'].fillna(0, inplace=True)
    df['Fare_Group'] = pd.qcut(df['Fare'],5,labels=False)
    df.drop(['Fare'], axis=1, inplace=True)
    return df 

def prepare_genders(df):
    genders = {"male": 0, "female": 1, "unknown": 2}
    df['Sex'] = df['Sex'].map(genders)
    df['Sex'].fillna(2, inplace=True)
    df['Sex'] = df['Sex'].astype(int)
    return df

def prepare_embarked(df):
    df['Embarked'].replace('', 'U', inplace=True)
    df['Embarked'].fillna('U', inplace=True)
    ports = {"S": 0, "C": 1, "Q": 2, "U": 3}
    df['Embarked'] = df['Embarked'].map(ports)
    return df
    
parser = argparse.ArgumentParser()
parser.add_argument('--output_path', dest='output_path', required=True)
args = parser.parse_args()
    
titanic_ds = Run.get_context().input_datasets['titanic_ds']
df = titanic_ds.to_pandas_dataframe().drop(['PassengerId', 'Name', 'Ticket', 'Cabin'], axis=1)
df = prepare_embarked(prepare_genders(prepare_fare(prepare_age(df))))

df.to_csv(os.path.join(args.output_path,"prepped_data.csv"))

print(f"Wrote prepped data to {args.output_path}/prepped_data.csv")
```

Приведенный выше фрагмент кода является полным, но минимальным примером подготовки данных для Титанике данных. Фрагмент кода начинается с Jupyter "Magic Command", чтобы вывести код в файл. Если вы не используете записную книжку Jupyter, удалите эту строку и создайте файл вручную.

Различные `prepare_` функции в приведенном выше фрагменте изменяют соответствующий столбец во входном наборе данных. Эти функции работают с данными после их изменения в `DataFrame` объект Pandas. В каждом случае отсутствующие данные заполняются репрезентативными случайными данными или сведениями о категориях, что означает «неизвестно». Данные о категориях на основе текста сопоставляются с целыми числами. Ненужные столбцы перезаписываются или удаляются. 

После того как код определит функции подготовки данных, код анализирует входной аргумент, который представляет собой путь, по которому нужно записать наши данные.  (Эти значения будут определяться `OutputFileDatasetConfig` объектами, которые будут обсуждаться на следующем шаге.) Код извлекает зарегистрированный `'titanic_cs'` `Dataset` , преобразует его в Pandas `DataFrame` и вызывает различные функции подготовки данных. 

Так как объект `output_path` является каталогом, в вызове `to_csv()` указывается имя файла `prepped_data.csv` .

### <a name="write-the-data-preparation-pipeline-step-pythonscriptstep"></a>Создание этапа конвейера подготовки данных ( `PythonScriptStep` )

Приведенный выше код подготовки данных должен быть связан с `PythonScripStep` объектом, который будет использоваться в конвейере. Путь, по которому записываются выходные данные CSV, создается `OutputFileDatasetConfig` объектом. Ресурсы, подготовленные ранее, такие как `ComputeTarget` , `RunConfig` и, `'titanic_ds' Dataset` используются для завершения спецификации.

```python
from azureml.data import OutputFileDatasetConfig
from azureml.pipeline.steps import PythonScriptStep

prepped_data_path = OutputFileDatasetConfig(name="output_path")

dataprep_step = PythonScriptStep(
    name="dataprep", 
    script_name="dataprep.py", 
    compute_target=compute_target, 
    runconfig=aml_run_config,
    arguments=["--output_path", prepped_data_path],
    inputs=[titanic_ds.as_named_input('titanic_ds')],
    allow_reuse=True
)
```

`prepped_data_path`Объект имеет тип `OutputFileDatasetConfig` , указывающий на каталог.  Обратите внимание, что он указан в `arguments` параметре. Если просмотреть предыдущий шаг, вы увидите, что в коде подготовки данных значение аргумента `'--output_path'` — это путь к каталогу, в котором был ЗАПИСАН CSV-файл. 

## <a name="train-with-automlstep"></a>Обучение с помощью Аутомлстеп

Настройка автоматического этапа конвейера ML выполняется с помощью `AutoMLConfig` класса. Этот гибкий класс описан в статье [Настройка автоматизированных экспериментов машинного обучения в Python](./how-to-configure-auto-train.md). Входные и выходные данные являются единственными аспектами настройки, требующими особого внимания в конвейере машинного обучения. Ниже подробно рассматриваются входные и выходные данные для `AutoMLConfig` в конвейерах. Помимо данных, преимуществом конвейеров машинного обучения является возможность использования различных целевых объектов вычислений для различных этапов. Вы можете использовать более мощный вариант `ComputeTarget` только для автоматизированного процесса машинного обучения. Это так же просто, как назначение более мощным `RunConfiguration` для `AutoMLConfig` `run_configuration` параметра объекта.

### <a name="send-data-to-automlstep"></a>Отправить данные в `AutoMLStep`

В конвейере машинного обучения входные данные должны быть `Dataset` объектом. Самый эффективный способ — предоставить входные данные в виде `OutputTabularDatasetConfig` объектов. Вы создаете объект этого типа с объектом в  `read_delimited_files()` `OutputFileDatasetConfig` , например `prepped_data_path` , `prepped_data_path` объектом.

```python
# type(prepped_data) == OutputTabularDatasetConfig
prepped_data = prepped_data_path.read_delimited_files()
```

Другой вариант — использовать `Dataset` объекты, зарегистрированные в рабочей области:

```python
prepped_data = Dataset.get_by_name(ws, 'Data_prepared')
```

Сравнение двух методов:

| Метод | Преимущества и недостатки | 
|-|-|
|`OutputTabularDatasetConfig`| Более высокая производительность | 
|| Естественный маршрут от `OutputFileDatasetConfig` | 
|| Данные не сохраняются после выполнения конвейера |
|| [Записная книжка с `OutputTabularDatasetConfig` методом](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines/nyc-taxi-data-regression-model-building/nyc-taxi-data-regression-model-building.ipynb) |
| Буферизаци `Dataset` | Снижение производительности. |
| | Может создаваться различными способами. | 
| | Данные сохраняются и видны во всей рабочей области |
| | [Записная книжка с зарегистрированным `Dataset` методом](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/continuous-retraining/auto-ml-continuous-retraining.ipynb)


### <a name="specify-automated-ml-outputs"></a>Указание автоматических выходных данных ML

Выходные данные `AutoMLStep` являются окончательными показателями модели с более высокой производительностью и самой моделью. Чтобы использовать эти выходные данные в дальнейших шагах конвейера, подготавливайте `OutputFileDatasetConfig` объекты для их получения.

```python
from azureml.pipeline.core import TrainingOutput

metrics_data = PipelineData(name='metrics_data',
                            datastore=datastore,
                            pipeline_output_name='metrics_output',
                            training_output=TrainingOutput(type='Metrics'))

model_data = PipelineData(name='best_model_data',
                          datastore=datastore,
                          pipeline_output_name='model_output',
                          training_output=TrainingOutput(type='Model'))
```

В приведенном выше фрагменте кода создаются два `PipelineData` объекта для метрик и выходных данных модели. Каждому из них присваивается имя, которое назначается хранилищу данных по умолчанию, полученному ранее, и связывается с конкретным из `type` `TrainingOutput` `AutoMLStep` . Так как мы присваиваем `pipeline_output_name` эти `PipelineData` объекты, их значения будут доступны не только на этапе отдельного конвейера, но и из конвейера в целом, как будет рассмотрено ниже в разделе «анализ результатов конвейера». 

### <a name="configure-and-create-the-automated-ml-pipeline-step"></a>Настройка и создание этапа автоматического создания конвейера ML

После определения входов и выходов пора создать `AutoMLConfig` и `AutoMLStep` . Сведения о конфигурации будут зависеть от задачи, как описано в разделе [Настройка автоматизированных экспериментов машинного обучения в Python](./how-to-configure-auto-train.md). Для задачи классификации выживания Титанике в следующем фрагменте кода демонстрируется простая конфигурация.

```python
from azureml.train.automl import AutoMLConfig
from azureml.pipeline.steps import AutoMLStep

# Change iterations to a reasonable number (50) to get better accuracy
automl_settings = {
    "iteration_timeout_minutes" : 10,
    "iterations" : 2,
    "experiment_timeout_hours" : 0.25,
    "primary_metric" : 'AUC_weighted'
}

automl_config = AutoMLConfig(task = 'classification',
                             path = '.',
                             debug_log = 'automated_ml_errors.log',
                             compute_target = compute_target,
                             run_configuration = aml_run_config,
                             featurization = 'auto',
                             training_data = prepped_data,
                             label_column_name = 'Survived',
                             **automl_settings)

train_step = AutoMLStep(name='AutoML_Classification',
    automl_config=automl_config,
    passthru_automl_config=False,
    outputs=[metrics_data,model_data],
    enable_default_model_output=False,
    enable_default_metrics_output=False,
    allow_reuse=True)
```
В фрагменте кода показана идиома, обычно используемая с `AutoMLConfig` . Аргументы, которые являются более гибкими (длинных), задаются в отдельном словаре, а значения с меньшей вероятностью изменяются непосредственно в `AutoMLConfig` конструкторе. В этом случае, `automl_settings` укажите краткий запуск: выполнение будет прерываться только после 2 итераций или 15 минут, в зависимости от того, что происходит раньше.

`automl_settings`Словарь передается в `AutoMLConfig` конструктор как кваргс. Другие параметры не являются сложными:

- `task` для этого примера задано значение `classification` . Другие допустимые значения — `regression` и. `forecasting`
- `path` и `debug_log` Опишите путь к проекту и локальный файл, в который будут записываться отладочные данные 
- `compute_target` ранее определено `compute_target` , что в этом примере является недорогым компьютером на основе ЦП. Если вы используете средства глубокого обучения Аутомл, необходимо изменить целевой объект вычислений так, чтобы он был основан на GPU
- Параметру `featurization` задается значение `auto`. Дополнительные сведения см. в разделе [Data Добавление признаков](./how-to-configure-auto-train.md#data-featurization) документа автоматической настройки ml. 
- `label_column_name` Указывает, какой столбец нам нужен для прогнозирования 
- `training_data` задается для `OutputTabularDatasetConfig` объектов, сделанных из выходных файлов этапа подготовки данных 

`AutoMLStep`Сам по себе принимает `AutoMLConfig` и имеет, как выходные данные, `PipelineData` объекты, созданные для хранения метрик и данных модели. 

>[!Important]
> Необходимо задать `enable_default_model_output` и, `enable_default_metrics_output` `True` только если используется  `AutoMLStepRun` .

В этом примере автоматизированный процесс машинного обучения выполнит перекрестные проверки на `training_data` . Число перекрестных проверок можно контролировать с помощью `n_cross_validations` аргумента. Если вы уже разделили обучающие данные как часть шагов подготовки данных, вы можете задать собственное значение `validation_data` `Dataset` .

Иногда можно увидеть использование `X` функций данных и `y` меток данных. Этот метод является устаревшим и должен использоваться `training_data` для ввода. 

## <a name="register-the-model-generated-by-automated-ml"></a>Регистрация модели, созданной автоматическим ML 

Последний шаг в простом конвейере машинного обучения регистрирует созданную модель. Добавив модель в реестр модели рабочей области, она будет доступна на портале, и ее можно будет использовать для управления версиями. Чтобы зарегистрировать модель, напишите другую `PythonScriptStep` , которая принимает `model_data` выходные данные `AutoMLStep` .

### <a name="write-the-code-to-register-the-model"></a>Напишите код для регистрации модели.

Модель регистрируется в `Workspace` . Вы, вероятно, знакомы с использованием `Workspace.from_config()` для входа в рабочую область на локальном компьютере, но существует другой способ получить рабочую область из работающего конвейера ml. `Run.get_context()`Возвращает активный объект `Run` . Этот `run` объект предоставляет доступ ко многим важным объектам, включая `Workspace` использованный здесь.

```python
%%writefile register_model.py
from azureml.core.model import Model, Dataset
from azureml.core.run import Run, _OfflineRun
from azureml.core import Workspace
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("--model_name", required=True)
parser.add_argument("--model_path", required=True)
args = parser.parse_args()

print(f"model_name : {args.model_name}")
print(f"model_path: {args.model_path}")

run = Run.get_context()
ws = Workspace.from_config() if type(run) == _OfflineRun else run.experiment.workspace

model = Model.register(workspace=ws,
                       model_path=args.model_path,
                       model_name=args.model_name)

print("Registered version {0} of model {1}".format(model.version, model.name))
```

### <a name="write-the-pythonscriptstep-code"></a>Написание кода Писонскриптстеп

При регистрации модели `PythonScriptStep` используется `PipelineParameter` для одного из своих аргументов. Параметры конвейера — это аргументы конвейеров, которые могут быть легко заданы во время отправки выполнения. После объявления они передаются как обычные аргументы. 

```python

from azureml.pipeline.core.graph import PipelineParameter

# The model name with which to register the trained model in the workspace.
model_name = PipelineParameter("model_name", default_value="TitanicSurvivalInitial")

register_step = PythonScriptStep(script_name="register_model.py",
                                       name="register_model",
                                       allow_reuse=False,
                                       arguments=["--model_name", model_name, "--model_path", model_data],
                                       inputs=[model_data],
                                       compute_target=compute_target,
                                       runconfig=aml_run_config)
```

## <a name="create-and-run-your-automated-ml-pipeline"></a>Создание и запуск конвейера автоматического обучения

Создание и запуск конвейера, содержащего объект, не `AutoMLStep` отличается от обычного конвейера. 

```python
from azureml.pipeline.core import Pipeline
from azureml.core import Experiment

pipeline = Pipeline(ws, [dataprep_step, train_step, register_step])

experiment = Experiment(workspace=ws, name='titanic_automl')

run = experiment.submit(pipeline, show_output=True)
run.wait_for_completion()
```

Приведенный выше код сочетает подготовку данных, автоматическую процедуру машинного обучения и регистрацию модели в `Pipeline` объекте. Затем он создает `Experiment` объект. `Experiment`Конструктор получит именованный эксперимент, если он существует, или при необходимости создать его. Он отправляет в `Pipeline` `Experiment` , создавая `Run` объект, который будет асинхронно запускать конвейер. `wait_for_completion()`Функция блокируется до завершения выполнения.

### <a name="examine-pipeline-results"></a>Проверка результатов конвейера 

После `run` завершения можно получить `PipelineData` объекты, которым было назначено значение `pipeline_output_name` . Вы можете скачать результаты и загрузить их для дальнейшей обработки.  

```python
metrics_output_port = run.get_pipeline_output('metrics_output')
model_output_port = run.get_pipeline_output('model_output')

metrics_output_port.download('.', show_progress=True)
model_output_port.download('.', show_progress=True)
```

Загруженные файлы записываются в подкаталог `azureml/{run.id}/` . Файл метрик имеет формат JSON и может быть преобразован в кадр данных Pandas для проверки.

Для локальной обработки может потребоваться установить соответствующие пакеты, такие как Pandas, в поле выбора, пакет SDK для AzureML и т. д. В этом примере, скорее всего, лучшая модель, обнаруженная автоматическим ML, будет зависеть от XGBoost.

```bash
!pip install xgboost==0.90
```

```python
import pandas as pd
import json

metrics_filename = metrics_output._path_on_datastore
# metrics_filename = path to downloaded file
with open(metrics_filename) as f:
   metrics_output_result = f.read()
   
deserialized_metrics_output = json.loads(metrics_output_result)
df = pd.DataFrame(deserialized_metrics_output)
df
```

В приведенном выше фрагменте кода показан файл метрик, загружаемый из своего расположения в хранилище данных Azure. Его также можно загрузить из скачанного файла, как показано в примечании. После десериализации и преобразования ее в кадр данных Pandas можно просмотреть подробные метрики для каждой итерации автоматического этапа ML.

Файл модели можно десериализовать в `Model` объект, который можно использовать для получения дополнительных показателей, анализа метрик и т. д. 

```python
import pickle

model_filename = model_output._path_on_datastore
# model_filename = path to downloaded file

with open(model_filename, "rb" ) as f:
    best_model = pickle.load(f)

# ... inferencing code not shown ...
```

Дополнительные сведения о загрузке и работе с существующими моделями см. в разделе [использование существующей модели с машинное обучение Azure](how-to-deploy-existing-model.md).

### <a name="download-the-results-of-an-automated-ml-run"></a>Скачивание результатов автоматического запуска ML

Если вы выполнили следующие инструкции вместе с этой статьей, то у вас будет экземпляр `run` объекта. Но вы также можете получить готовые `Run` объекты из, `Workspace` используя `Experiment` объект.

Рабочая область содержит полную запись всех экспериментов и запусков. Можно использовать портал для поиска и скачивания выходных данных экспериментов или использования кода. Чтобы получить доступ к записям из исторических запусков, используйте Машинное обучение Azure, чтобы найти идентификатор требуемого запуска. С помощью этого идентификатора можно выбрать конкретное значение для `run` `Workspace` и `Experiment` .

```python
# Retrieved from Azure Machine Learning web UI
run_id = 'aaaaaaaa-bbbb-cccc-dddd-0123456789AB'
experiment = ws.experiments['titanic_automl']
run = next(run for run in ex.get_runs() if run.id == run_id)
```

Вам пришлось бы изменить строки в приведенном выше коде на особенности своего исторического выполнения. В приведенном выше фрагменте предполагается, что вы присвоены `ws` значение, соответствующее `Workspace` стандарту `from_config()` . Полученный эксперимент извлекается напрямую, а затем код находит `Run` интересующий его объект, сопоставляя `run.id` значение.

После получения `Run` объекта можно скачать метрики и модель. 

```python
automl_run = next(r for r in run.get_children() if r.name == 'AutoML_Classification')
outputs = automl_run.get_outputs()
metrics = outputs['default_metrics_AutoML_Classification']
model = outputs['default_model_AutoML_Classification']

metrics.get_port_data_reference().download('.')
model.get_port_data_reference().download('.')
```

Каждый `Run` объект содержит `StepRun` объекты, содержащие сведения о выполнении отдельных шагов конвейера. `run`Поиск `StepRun` объекта для `AutoMLStep` . Метрики и модель извлекаются с использованием имен по умолчанию, которые доступны, даже если не передать `PipelineData` объекты в `outputs` параметр `AutoMLStep` . 

Наконец, фактические метрики и модель загружаются на локальный компьютер, как было сказано в разделе "анализ результатов конвейера" выше.

## <a name="next-steps"></a>Дальнейшие действия

- Запустите эту записную книжку Jupyter, где [приведен полный пример автоматизированного ML в конвейере](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines/nyc-taxi-data-regression-model-building/nyc-taxi-data-regression-model-building.ipynb) , который использует регрессию для прогнозирования такси году
- [Создание автоматизированных экспериментов ML без написания кода](how-to-use-automated-ml-for-ml-models.md)
- Изучите различные [записные книжки Jupyter, демонстрирующие автоматический ML](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/automated-machine-learning)
- Ознакомьтесь со сведениями об интеграции конвейера в, чтобы получить [полное млопс](./concept-model-management-and-deployment.md#automate-the-ml-lifecycle) или изучить [репозиторий млопс GitHub](https://github.com/Microsoft/MLOpspython) .
