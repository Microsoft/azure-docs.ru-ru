---
title: Удаленные целевые объекты вычислений автоматического машинного обучения
titleSuffix: Azure Machine Learning
description: Узнайте, как создавать модели с помощью автоматизированного машинного обучения на Машинное обучение Azure удаленном целевом объекте вычислений с Машинное обучение Azure
services: machine-learning
author: cartacioS
ms.author: sacartac
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.workload: data-services
ms.topic: conceptual
ms.custom: how-to, automl
ms.date: 03/09/2020
ms.openlocfilehash: 402af581e0ca6de9936b78106840a4fe273069d8
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102503459"
---
# <a name="train-models-with-automated-machine-learning-in-the-cloud"></a>Обучение моделей с помощью автоматического машинного обучения в облаке



В службе "Машинное обучение Azure" можно обучить модель для различных типов доступных для управления вычислительных ресурсов. Целью вычислений может быть локальный компьютер или ресурс в облаке.

Вы можете легко масштабировать или масштабировать эксперимент машинного обучения, добавив дополнительные целевые объекты вычислений, например Машинное обучение Azure Compute (Амлкомпуте). AmlCompute — это управляемая вычислительная инфраструктура, которая позволяет пользователю легко создавать одно- и многоузловые вычислительные среды.

Из этой статьи вы узнаете, как создать модель с помощью автоматизированного ML с Амлкомпуте.

## <a name="how-does-remote-differ-from-local"></a>Чем удаленный компьютер отличается от локального?

При использовании удаленного целевого объекта вычислений доступны дополнительные функции.  Дополнительные сведения см. в разделе [локальные и удаленные целевые объекты вычислений](concept-automated-ml.md#local-remote).

Руководство «[обучение модели классификации с автоматизированным машинным обучением](tutorial-auto-train-models.md)» рассказывает о том, как использовать локальный компьютер для обучения модели с помощью автоматизированного ml. Рабочий процесс, используемый при локальном обучении, применяется и для удаленных целевых объектов. Чтобы обучить удаленно, сначала необходимо создать удаленный целевой объект вычислений, например Амлкомпуте. Затем необходимо настроить удаленный ресурс и отправить на него код.

В этой статье приведены дополнительные шаги, необходимые для запуска автоматического эксперимента ML на удаленном целевом объекте Амлкомпуте. Объект рабочей области, `ws`, из учебника используется во всем представленном ниже коде.

```python
ws = Workspace.from_config()
```

## <a name="create-resource"></a>Создайте ресурс

Создайте [`AmlCompute`](/python/api/azureml-core/azureml.core.compute.amlcompute%28class%29) целевой объект в рабочей области ( `ws` ), если он еще не существует.

**Оценка времени**. Создание целевого объекта амлкомпуте занимает примерно 5 минут.

```python
from azureml.core.compute import AmlCompute
from azureml.core.compute import ComputeTarget
import os

# choose a name for your cluster
compute_name = os.environ.get("AML_COMPUTE_CLUSTER_NAME", "cpu-cluster")
compute_min_nodes = os.environ.get("AML_COMPUTE_CLUSTER_MIN_NODES", 0)
compute_max_nodes = os.environ.get("AML_COMPUTE_CLUSTER_MAX_NODES", 4)

# This example uses CPU VM. For using GPU VM, set SKU to STANDARD_NC6
vm_size = os.environ.get("AML_COMPUTE_CLUSTER_SKU", "STANDARD_D2_V2")


if compute_name in ws.compute_targets:
    compute_target = ws.compute_targets[compute_name]
    if compute_target and type(compute_target) is AmlCompute:
        print('found compute target. just use it. ' + compute_name)
else:
    print('creating a new compute target...')
    provisioning_config = AmlCompute.provisioning_configuration(vm_size = vm_size,
                                                                min_nodes = compute_min_nodes, 
                                                                max_nodes = compute_max_nodes)

    # create the cluster
    compute_target = ComputeTarget.create(ws, compute_name, provisioning_config)
    
    # can poll for a minimum number of nodes and for a specific timeout. 
    # if no min node count is provided it will use the scale settings for the cluster
    compute_target.wait_for_completion(show_output=True, min_node_count=None, timeout_in_minutes=20)
    
     # For a more detailed view of current AmlCompute status, use get_status()
    print(compute_target.get_status().serialize())
```

Теперь можно использовать объект `compute_target` в качестве целевого объекта удаленных вычислений.

К ограничениям имен кластеров относятся:
+ Должно быть короче 64 символов.
+ Не может содержать ни один из следующих символов: `\` ~ ! @ # $% ^ & * () = + _ [] {} \\ \\ |;: \' \\ ",  < > /?. `

## <a name="access-data-using-tabulardataset-function"></a>Доступ к данным с помощью функции Табулардатасет

Определяется training_data как [`TabularDataset`](/python/api/azureml-core/azureml.data.tabulardataset) и метка, которая передается в автоматизированное машинное обучение в [`AutoMLConfig`](/python/api/azureml-train-automl-client/azureml.train.automl.automlconfig.automlconfig) . По `TabularDataset` `from_delimited_files` умолчанию метод задает `infer_column_types` для значение true, при котором тип столбцов будет автоматически определяться. 

Если вы хотите вручную задать типы столбцов, можно задать `set_column_types` аргумент, чтобы вручную задать тип каждого столбца. В следующем примере кода данные поступают из пакета sklearn.

```python
from sklearn import datasets
from azureml.core.dataset import Dataset
from scipy import sparse
import numpy as np
import pandas as pd
import os

# Create a project_folder if it doesn't exist
if not os.path.isdir('data'):
    os.mkdir('data')
    
if not os.path.exists('project_folder'):
    os.makedirs('project_folder')

X = pd.DataFrame(data_train.data[100:,:])
y = pd.DataFrame(data_train.target[100:])

# merge X and y
label = "digit"
X[label] = y

training_data = X

training_data.to_csv('data/digits.csv')
ds = ws.get_default_datastore()
ds.upload(src_dir='./data', target_path='digitsdata', overwrite=True, show_progress=True)

training_data = Dataset.Tabular.from_delimited_files(path=ds.path('digitsdata/digits.csv'))
```

## <a name="configure-experiment"></a>Настройка эксперимента
Задайте значения для `AutoMLConfig`.  (См. [полный список параметров](how-to-configure-auto-train.md#configure-experiment) и их возможные значения.)

```python
from azureml.train.automl import AutoMLConfig
import time
import logging

automl_settings = {
    "name": "AutoML_Demo_Experiment_{0}".format(time.time()),
    "experiment_timeout_minutes" : 20,
    "enable_early_stopping" : True,
    "iteration_timeout_minutes": 10,
    "n_cross_validations": 5,
    "primary_metric": 'AUC_weighted',
    "max_concurrent_iterations": 10,
}

automl_config = AutoMLConfig(task='classification',
                             debug_log='automl_errors.log',
                             path=project_folder,
                             compute_target=compute_target,
                             training_data=training_data,
                             label_column_name=label,
                             **automl_settings,
                             )
```

## <a name="submit-training-experiment"></a>Отправка обучающего эксперимента

Теперь отправьте конфигурацию для автоматического выбора алгоритма, гиперпараметров и обучения модели.

```python
from azureml.core.experiment import Experiment
experiment = Experiment(ws, 'Tutorial-automl-remote')
remote_run = experiment.submit(automl_config, show_output=True)
```

Результат будет выглядеть примерно так:

```output
Running on remote compute: mydsvmParent Run ID: AutoML_015ffe76-c331-406d-9bfd-0fd42d8ab7f6
***********************************************************************************************
ITERATION: The iteration being evaluated.
PIPELINE:  A summary description of the pipeline being evaluated.
DURATION: Time taken for the current iteration.
METRIC: The result of computing score on the fitted pipeline.
BEST: The best observed score thus far.
***********************************************************************************************

 ITERATION     PIPELINE                               DURATION                METRIC      BEST
         2      Standardize SGD classifier            0:02:36                  0.954     0.954
         7      Normalizer DT                         0:02:22                  0.161     0.954
         0      Scale MaxAbs 1 extra trees            0:02:45                  0.936     0.954
         4      Robust Scaler SGD classifier          0:02:24                  0.867     0.954
         1      Normalizer kNN                        0:02:44                  0.984     0.984
         9      Normalizer extra trees                0:03:15                  0.834     0.984
         5      Robust Scaler DT                      0:02:18                  0.736     0.984
         8      Standardize kNN                       0:02:05                  0.981     0.984
         6      Standardize SVM                       0:02:18                  0.984     0.984
        10      Scale MaxAbs 1 DT                     0:02:18                  0.077     0.984
        11      Standardize SGD classifier            0:02:24                  0.863     0.984
         3      Standardize gradient boosting         0:03:03                  0.971     0.984
        12      Robust Scaler logistic regression     0:02:32                  0.955     0.984
        14      Scale MaxAbs 1 SVM                    0:02:15                  0.989     0.989
        13      Scale MaxAbs 1 gradient boosting      0:02:15                  0.971     0.989
        15      Robust Scaler kNN                     0:02:28                  0.904     0.989
        17      Standardize kNN                       0:02:22                  0.974     0.989
        16      Scale 0/1 gradient boosting           0:02:18                  0.968     0.989
        18      Scale 0/1 extra trees                 0:02:18                  0.828     0.989
        19      Robust Scaler kNN                     0:02:32                  0.983     0.989
```

## <a name="explore-results"></a>Изучите результаты.

Вы можете использовать тот же [мини](/python/api/azureml-widgets/azureml.widgets) -приложение Jupyter, как показано в [учебном курсе](tutorial-auto-train-models.md#explore-the-results) , чтобы просмотреть диаграмму и таблицу результатов.

```python
from azureml.widgets import RunDetails
RunDetails(remote_run).show()
```

Вот как выглядит статическое изображение мини-приложения.  В записной книжке можно щелкнуть любую строку таблицы, чтобы просмотреть свойства прогона и файлы журналов для этого прогона.   Для просмотра графика каждой доступной метрики для каждой итерации можно также использовать раскрывающийся список над графиком.

![таблица мини-приложения](./media/how-to-auto-train-remote/table.png)
![график мини-приложения](./media/how-to-auto-train-remote/plot.png)

Мини-приложение показывает URL-адрес, который можно использовать для просмотра и изучения данных отдельного прогона.  

Если вы не в записной книжке Jupyter, можно отобразить URL-адрес из самого запуска:

```
remote_run.get_portal_url()
```

Та же информация доступна в вашей рабочей области.  Дополнительные сведения об этих результатах см. в разделе [Оценка автоматических результатов машинного обучения](how-to-understand-automated-ml.md).

## <a name="example"></a>Пример

В следующей [записной книжке](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/regression/auto-ml-regression.ipynb) показаны понятия, описанные в этой статье.

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте, [как настроить параметры автоматического обучения](how-to-configure-auto-train.md)
* См. [практические руководства](how-to-machine-learning-interpretability-automl.md) по включению функций интерпретации модели в автоматизированных экспериментах ml.