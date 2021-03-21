---
title: Обучение и развертывание модели обучения подкреплением (Предварительная версия).
titleSuffix: Azure Machine Learning
description: Узнайте, как использовать Машинное обучение Azure подкреплением Learning (Предварительная версия) для обучения агента RL для воспроизведения Теннисов.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: peterlu
author: peterclu
ms.date: 05/05/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, contperf-fy21q2
ms.openlocfilehash: 4c03016d003978b3c56361595bec7c559205574b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102520886"
---
# <a name="reinforcement-learning-preview-with-azure-machine-learning"></a>Обучение с подкреплением (предварительная версия) в Машинном обучении Azure



> [!NOTE]
> В настоящее время обучение с подкреплением (ОП) в Машинном обучении Azure предлагается в виде предварительной версии функции. На данный момент ее поддерживают только платформы Ray и RLlib.

Из этой статьи вы узнаете, как обучить агент обучения с подкреплением (ОП) играть в видеоигру Pong. Для управления сложностью распределенных RL используется библиотека Python с открытым кодом [Ray рллиб](https://ray.readthedocs.io/en/master/rllib.html) с машинное обучение Azure.

В этой статье раскрываются следующие темы:
> [!div class="checklist"]
> * Настройка эксперимента
> * Определение головного и рабочих узлов
> * Создание оценщика ОП
> * Отправка эксперимента для запуска выполнения
> * Просмотр результатов

В основе этой статьи лежит [пример RLlib Pong](https://aka.ms/azureml-rl-pong), который можно найти в [репозитории GitHub](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/reinforcement-learning/README.md) в записной книжке Машинного обучения Azure.

## <a name="prerequisites"></a>Предварительные требования

Запустите этот код в любой из этих сред. Рекомендуется использовать Машинное обучение Azure вычислительный экземпляр для ускорения запуска. Вы можете быстро клонировать и запустить образцы записных книжек подкреплением на Машинное обучение Azure вычислительном экземпляре.

 - Вычислительная операция Машинного обучения Azure

     - Сведения о том, как клонировать примеры записных книжек, см. в [учебнике по настройке среды и рабочей области](tutorial-1st-experiment-sdk-setup.md).
         - Вместо **учебников** клонируйте папку **how-to-use-azureml**.
     - Запустите записную книжку настройки виртуальной сети, расположенную по адресу `/how-to-use-azureml/reinforcement-learning/setup/devenv_setup.ipynb`, чтобы открыть сетевые порты, используемые для распределенного обучения с подкреплением.
     - Запустите пример записной книжки `/how-to-use-azureml/reinforcement-learning/atari-on-distributed-compute/pong_rllib.ipynb`.
 
 - Собственный сервер Jupyter Notebook

    - Установите [пакет SDK для Машинного обучения Azure](/python/api/overview/azure/ml/install).
    - Установите [пакет SDK обучения с подкреплением для Машинного обучения Azure](/python/api/azureml-contrib-reinforcementlearning/): `pip install --upgrade azureml-contrib-reinforcementlearning`.
    - Создайте [файл конфигурации рабочей области](how-to-configure-environment.md#workspace).
    - Запустите виртуальную сеть, чтобы открыть сетевые порты, используемые для распределенного обучения подкреплением.


## <a name="how-to-train-a-pong-playing-agent"></a>Обучение агента, играющего в Pong

Обучение с подкреплением (ОП) — это подход к машинному обучению, предполагающий обучение на практике. В то время как другие методы машинного обучения предполагают пассивную передачу входных данных и обнаружение в них структур, для ОП используются **агенты обучения**, обеспечивающие активное принятие решений и обучение на собственных результатах.

Агенты обучения учатся играть в Pong в **моделируемой среде**. Для каждого кадра игры они принимают решение о том, нужно ли переместить ракетку вверх или вниз или же оставить ее на месте. Для принятия решения они оценивают состояние игры (RGB-изображение экрана).

Чтобы сообщить агенту о принятии успешного решения, в ОП используется **поощрения**. В этом примере агент получает положительный награду, когда он оценивает точку и отрицательное вознаграждение при оценке точки. После множества итераций агент обучения узнает, как выбрать действие на основе текущего состояния для оптимизации суммы ожидаемых поощрений в будущем. Для выполнения этой оптимизации в RL обычно используются **глубокие нейронные сети** (DNN). 

Обучение завершается, когда средняя оценка поощрений агента в эпохе обучения достигает 18. Это означает, что агент в среднем опережал своего соперника в матчах по крайней мере на 18 очков (максимум 21 очко).

Процесс итерации по моделированию и повторному обучению DNN является дорогостоящим и требует большого количества данных. Одним из способов повышения производительности заданий ОП является **параллелизация**, позволяющая нескольким агентам обучения действовать и обучаться одновременно. Однако управление распределенной средой ОП может оказаться сложной задачей.

Машинное обучение Azure предоставляет платформу, которая позволит справиться с ней за счет горизонтального увеличения масштаба рабочих нагрузок ОП.

## <a name="set-up-the-environment"></a>Настройка среды

Настройте локальную среду RL, выполнив следующие действия.
1. Загрузка требуемых пакетов Python
1. Инициализация рабочей области
1. Создание эксперимента
1. Указание настроенной виртуальной сети.

### <a name="import-libraries"></a>Импорт библиотек

Импортируйте необходимые пакеты Python для выполнения оставшейся части этого примера.

```python
# Azure ML Core imports
import azureml.core
from azureml.core import Workspace
from azureml.core import Experiment
from azureml.core.compute import AmlCompute
from azureml.core.compute import ComputeTarget
from azureml.core.runconfig import EnvironmentDefinition
from azureml.widgets import RunDetails
from azureml.tensorboard import Tensorboard

# Azure ML Reinforcement Learning imports
from azureml.contrib.train.rl import ReinforcementLearningEstimator, Ray
from azureml.contrib.train.rl import WorkerConfiguration
```

### <a name="initialize-a-workspace"></a>Инициализация рабочей области

Инициализируйте объект [рабочей области](concept-workspace.md) из `config.json` файла, созданного в [разделе Предварительные требования](#prerequisites). Если этот код выполняется в вычислительной операции Машинного обучения Azure, файл конфигурации уже создан.

```Python
ws = Workspace.from_config()
```

### <a name="create-a-reinforcement-learning-experiment"></a>Создание эксперимента по обучению с подкреплением

Создайте [эксперимент](/python/api/azureml-core/azureml.core.experiment.experiment), чтобы отслеживать выполнение обучения с подкреплением. В Машинном обучении Azure эксперименты — это логические коллекции связанных попыток, используемые для упорядочения журналов выполнения, прочих журналов, выходных данных и многого другого.

```python
experiment_name='rllib-pong-multi-node'

exp = Experiment(workspace=ws, name=experiment_name)
```

### <a name="specify-a-virtual-network"></a>Задание виртуальной сети

Для заданий ОП, использующих несколько целевых объектов вычислений, необходимо указать виртуальную сеть с открытыми портами, которые позволят рабочим и головным узлам взаимодействовать друг с другом.

Виртуальная сеть может находиться в любой группе ресурсов, но она должна быть в том же регионе, что и рабочая область. Дополнительные сведения о настройке виртуальной сети см. в разделе "предварительные требования" в записной книжке Настройка рабочей области. Здесь следует указать имя виртуальной сети в группе ресурсов.

```python
vnet = 'your_vnet'
```

## <a name="define-head-and-worker-compute-targets"></a>Определение целевых вычислительных узлов (головного и рабочих)

В этом примере используются отдельные целевые объекты вычислений для головных и рабочих узлов. Эти параметры позволяют масштабировать ресурсы вычислений в зависимости от рабочей нагрузки. Задайте количество узлов и размер каждого узла в зависимости от ваших потребностей.

### <a name="head-computing-target"></a>Целевой головной вычислительный узел

Для повышения производительности глубокого обучения можно использовать головной кластер, оснащенный GPU. Головной узел обучает нейронную сеть, которую агент использует для принятия решений. Головной узел также собирает точки данных из рабочих узлов для обучения нейронной сети.

Для вычислений головной узел использует одну [виртуальную машину `STANDARD_NC6`](../virtual-machines/nc-series.md) (ВМ). Он имеет 6 виртуальных ЦП для распределения работы между ними.


```python
from azureml.core.compute import AmlCompute, ComputeTarget

# choose a name for the Ray head cluster
head_compute_name = 'head-gpu'
head_compute_min_nodes = 0
head_compute_max_nodes = 2

# This example uses GPU VM. For using CPU VM, set SKU to STANDARD_D2_V2
head_vm_size = 'STANDARD_NC6'

if head_compute_name in ws.compute_targets:
    head_compute_target = ws.compute_targets[head_compute_name]
    if head_compute_target and type(head_compute_target) is AmlCompute:
        print(f'found head compute target. just use it {head_compute_name}')
else:
    print('creating a new head compute target...')
    provisioning_config = AmlCompute.provisioning_configuration(vm_size = head_vm_size,
                                                                min_nodes = head_compute_min_nodes, 
                                                                max_nodes = head_compute_max_nodes,
                                                                vnet_resourcegroup_name = ws.resource_group,
                                                                vnet_name = vnet_name,
                                                                subnet_name = 'default')

    # create the cluster
    head_compute_target = ComputeTarget.create(ws, head_compute_name, provisioning_config)
    
    # can poll for a minimum number of nodes and for a specific timeout. 
    # if no min node count is provided it will use the scale settings for the cluster
    head_compute_target.wait_for_completion(show_output=True, min_node_count=None, timeout_in_minutes=20)
    
     # For a more detailed view of current AmlCompute status, use get_status()
    print(head_compute_target.get_status().serialize())
```

[!INCLUDE [low-pri-note](../../includes/machine-learning-low-pri-vm.md)]

### <a name="worker-computing-cluster"></a>Рабочий вычислительный кластер

В этом примере для целевого рабочего вычислительного узла используются четыре [виртуальные машины `STANDARD_D2_V2`](../virtual-machines/nc-series.md). Каждый рабочий узел имеет 2 доступных ЦП для всего 8 доступных ЦП.

Для рабочих узлов GPU не требуется, так как они не выполняют глубокое обучение. Рабочие узлы выполняют моделирование игры и собирают данные.

```python
# choose a name for your Ray worker cluster
worker_compute_name = 'worker-cpu'
worker_compute_min_nodes = 0 
worker_compute_max_nodes = 4

# This example uses CPU VM. For using GPU VM, set SKU to STANDARD_NC6
worker_vm_size = 'STANDARD_D2_V2'

# Create the compute target if it hasn't been created already
if worker_compute_name in ws.compute_targets:
    worker_compute_target = ws.compute_targets[worker_compute_name]
    if worker_compute_target and type(worker_compute_target) is AmlCompute:
        print(f'found worker compute target. just use it {worker_compute_name}')
else:
    print('creating a new worker compute target...')
    provisioning_config = AmlCompute.provisioning_configuration(vm_size = worker_vm_size,
                                                                min_nodes = worker_compute_min_nodes, 
                                                                max_nodes = worker_compute_max_nodes,
                                                                vnet_resourcegroup_name = ws.resource_group,
                                                                vnet_name = vnet_name,
                                                                subnet_name = 'default')

    # create the cluster
    worker_compute_target = ComputeTarget.create(ws, worker_compute_name, provisioning_config)
    
    # can poll for a minimum number of nodes and for a specific timeout. 
    # if no min node count is provided it will use the scale settings for the cluster
    worker_compute_target.wait_for_completion(show_output=True, min_node_count=None, timeout_in_minutes=20)
    
     # For a more detailed view of current AmlCompute status, use get_status()
    print(worker_compute_target.get_status().serialize())
```

## <a name="create-a-reinforcement-learning-estimator"></a>Создание оценщика обучения с подкреплением
Используйте [реинфорцементлеарнинжестиматор](/python/api/azureml-contrib-reinforcementlearning/azureml.contrib.train.rl.reinforcementlearningestimator) для отправки учебного задания в машинное обучение Azure.

Машинное обучение Azure использует классы оценщика для инкапсуляции сведений о конфигурации выполнения. Это позволяет указать, как настроить выполнение скрипта. 

### <a name="define-a-worker-configuration"></a>Определение конфигурации рабочего кластера

Объект Воркерконфигуратион сообщает Машинное обучение Azure, как инициализировать рабочий кластер, который запускает скрипт записи.

```python
# Pip packages we will use for both head and worker
pip_packages=["ray[rllib]==0.8.3"] # Latest version of Ray has fixes for isses related to object transfers

# Specify the Ray worker configuration
worker_conf = WorkerConfiguration(
    
    # Azure ML compute cluster to run Ray workers
    compute_target=worker_compute_target, 
    
    # Number of worker nodes
    node_count=4,
    
    # GPU
    use_gpu=False, 
    
    # PIP packages to use
    pip_packages=pip_packages
)
```

### <a name="define-script-parameters"></a>Определение параметров скрипта

Начальный скрипт `pong_rllib.py` принимает список параметров, определяющих способ выполнения задания обучения. Передача этих параметров через оценщик в качестве слоя инкапсуляции упрощает изменение параметров скрипта и запуск конфигураций независимо друг от друга.

Указание правильного `num_workers` приводит к максимально эффективному обеспечению параллелизации. Задайте число рабочих узлов равным количеству доступных ЦП. В этом примере можно использовать следующие вычисления:

Головной узел — это [Standard_NC6](../virtual-machines/nc-series.md) с 6 виртуальными ЦП. Рабочий кластер — это 4 [виртуальные машины Standard_D2_V2](../cloud-services/cloud-services-sizes-specs.md#dv2-series) с двумя ЦП на каждой, всего 8 ЦП. Однако из числа рабочих процессоров необходимо вычесть 1 ЦП, так как один из их нужно выделить для роли головного узла.

6 ЦП + 8 ЦП – 1 головной ЦП = 13 одновременных рабочих процессов. Машинное обучение Azure использует кластеры головного и рабочих узлов, чтобы различать вычислительные ресурсы. Однако луч не различает головные и рабочие процессы, а все ЦП доступны как рабочие потоки.


```python
training_algorithm = "IMPALA"
rl_environment = "PongNoFrameskip-v4"

# Training script parameters
script_params = {
    
    # Training algorithm, IMPALA in this case
    "--run": training_algorithm,
    
    # Environment, Pong in this case
    "--env": rl_environment,
    
    # Add additional single quotes at the both ends of string values as we have spaces in the 
    # string parameters, outermost quotes are not passed to scripts as they are not actually part of string
    # Number of GPUs
    # Number of ray workers
    "--config": '\'{"num_gpus": 1, "num_workers": 13}\'',
    
    # Target episode reward mean to stop the training
    # Total training time in seconds
    "--stop": '\'{"episode_reward_mean": 18, "time_total_s": 3600}\'',
}
```

### <a name="define-the-reinforcement-learning-estimator"></a>Определение оценщика обучения с подкреплением

Для создания оценщика используйте список параметров и объект конфигурации рабочего узла.

```python
# RL estimator
rl_estimator = ReinforcementLearningEstimator(
    
    # Location of source files
    source_directory='files',
    
    # Python script file
    entry_script="pong_rllib.py",
    
    # Parameters to pass to the script file
    # Defined above.
    script_params=script_params,
    
    # The Azure ML compute target set up for Ray head nodes
    compute_target=head_compute_target,
    
    # Pip packages
    pip_packages=pip_packages,
    
    # GPU usage
    use_gpu=True,
    
    # RL framework. Currently must be Ray.
    rl_framework=Ray(),
    
    # Ray worker configuration defined above.
    worker_configuration=worker_conf,
    
    # How long to wait for whole cluster to start
    cluster_coordination_timeout_seconds=3600,
    
    # Maximum time for the whole Ray job to run
    # This will cut off the run after an hour
    max_run_duration_seconds=3600,
    
    # Allow the docker container Ray runs in to make full use
    # of the shared memory available from the host OS.
    shm_size=24*1024*1024*1024
)
```

### <a name="entry-script"></a>Начальный скрипт

[Начальный скрипт](https://aka.ms/azure-rl-pong-script) `pong_rllib.py` обучает нейронную сеть с помощью [среды OpenAI Gym ](https://github.com/openai/gym/) `PongNoFrameSkip-v4`. OpenAI Gym — это стандартизованные интерфейсы для тестирования алгоритмов обучения с подкреплением в классической игре Atari.

В этом примере используется алгоритм обучения, известный как [IMPALA](https://arxiv.org/abs/1802.01561) (Importance Weighted Actor-Learner Architecture — архитектура "Субъект — учащийся" со взвешиванием по важности). IMPALA параллелизует каждый отдельный субъект обучения для масштабирования на множество вычислительных узлов без ущерба для скорости и стабильности.

[Ray Tune](https://ray.readthedocs.io/en/latest/tune.html) выполняет оркестрацию задач рабочего процесса IMPALA.

```python
import ray
import ray.tune as tune
from ray.rllib import train

import os
import sys

from azureml.core import Run
from utils import callbacks

DEFAULT_RAY_ADDRESS = 'localhost:6379'

if __name__ == "__main__":

    # Parse arguments
    train_parser = train.create_parser()

    args = train_parser.parse_args()
    print("Algorithm config:", args.config)

    if args.ray_address is None:
        args.ray_address = DEFAULT_RAY_ADDRESS

    ray.init(address=args.ray_address)

    tune.run(run_or_experiment=args.run,
             config={
                 "env": args.env,
                 "num_gpus": args.config["num_gpus"],
                 "num_workers": args.config["num_workers"],
                 "callbacks": {"on_train_result": callbacks.on_train_result},
                 "sample_batch_size": 50,
                 "train_batch_size": 1000,
                 "num_sgd_iter": 2,
                 "num_data_loader_buffers": 2,
                 "model": {
                    "dim": 42
                 },
             },
             stop=args.stop,
             local_dir='./logs')
```

### <a name="logging-callback-function"></a>Ведение журнала функции обратного вызова


Начальный скрипт использует служебную функцию, чтобы определить [пользовательскую функцию обратного вызова RLlib](https://docs.ray.io/en/latest/rllib-training.html#callbacks-and-custom-metrics) для записи метрик в рабочую область Машинного обучение Azure. Сведения о том, как просмотреть эти метрики, см. в разделе [Мониторинг и просмотр результатов](#monitor-and-view-results).

```python
'''RLlib callbacks module:
    Common callback methods to be passed to RLlib trainer.
'''
from azureml.core import Run

def on_train_result(info):
    '''Callback on train result to record metrics returned by trainer.
    '''
    run = Run.get_context()
    run.log(
        name='episode_reward_mean',
        value=info["result"]["episode_reward_mean"])
    run.log(
        name='episodes_total',
        value=info["result"]["episodes_total"])
```

## <a name="submit-a-run"></a>Отправка запроса на выполнение

Команда [run](/python/api/azureml-core/azureml.core.run%28class%29) обрабатывает журнал выполнения для выполняющихся или завершенных заданий. 

```python
run = exp.submit(config=rl_estimator)
```
> [!NOTE]
> Ее выполнение может занять от 30 до 45 минут.

## <a name="monitor-and-view-results"></a>Мониторинг и просмотр результатов

Используйте мини-приложение Jupyter в Машинном обучении Azure, чтобы просматривать состояние выполнения в режиме реального времени. В мини-приложении показаны два дочерних запуска: один для Head и один для работников. 

```python
from azureml.widgets import RunDetails

RunDetails(run).show()
run.wait_for_completion()
```

1. Дождитесь загрузки мини-приложения.
1. В списке операций выполнения выберите операцию для головного узла.

Выберите **Click here to see the run in Azure Machine Learning studio** (Щелкните здесь, чтобы просмотреть выполнение в студии Машинного обучения Azure), чтобы получить дополнительные сведения о выполнении в студии. Вы можете получить доступ к этим сведениям во время выполнения или после завершения.

![График со сведениями о выполнении для иллюстрации работы мини-приложения](./media/how-to-use-reinforcement-learning/pong-run-details-widget.png)

На графике **episode_reward_mean** показано среднее количество очков, набранных в каждой эпохе обучения. Как можно видеть, сначала агент обучения играл плохо, проигрывая матчи без единого очка (значение reward_mean равно –21). За 100 итераций он научился побеждать компьютер-соперник, набирая в среднем 18 очков.

В журналах дочерней операции выполнения можно просмотреть результаты оценки, записанные в файле driver_log.txt. Возможно, потребуется подождать несколько минут, прежде чем эти метрики станут доступны на странице выполнения.

В ходе небольшой работы вы узнали, как настроить несколько вычислительных ресурсов для обучения агента подкреплением Learning Agent, который очень хорошо воспроизводился на компьютере опппонент.

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как обучить агент обучения с подкреплением с помощью агента обучения IMPALA. Дополнительные примеры см. в [репозитории GitHub в разделе обучения с подкреплением в Машинном обучении Azure ](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/reinforcement-learning/README.md).