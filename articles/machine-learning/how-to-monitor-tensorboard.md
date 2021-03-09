---
title: Визуализация экспериментов с помощью TensorBoard
titleSuffix: Azure Machine Learning
description: Запустите TensorBoard для визуализации журналов выполнения экспериментов и выявления потенциальных областей для настройки и повторного обучения гиперпараметров.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: minxia
ms.author: minxia
ms.date: 02/27/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: 2af82734fb9e1571242eec016f36f691411a8f2e
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102518761"
---
# <a name="visualize-experiment-runs-and-metrics-with-tensorboard-and-azure-machine-learning"></a>Визуализируйте запуски и метрики экспериментов с помощью TensorBoard и Машинного обучения Azure


В этой статье вы узнаете, как просматривать запуски и метрики экспериментов в TensorBoard с помощью [пакета `tensorboard`](/python/api/azureml-tensorboard/) в основном пакете SDK Машинного обучения Azure. Изучив запуски экспериментов, вы сможете лучше настроить и переобучить модели машинного обучения.

[TensorBoard](https://www.tensorflow.org/tensorboard/r1/overview) — это набор веб-приложений для проверки и понимания структуры и производительности эксперимента.

Способ запуска TensorBoard с экспериментами Машинного обучения Azure зависит от типа эксперимента.
+ Если ваш эксперимент самостоятельно выводит файлы журналов, которые могут использоваться TensorBoard, например PyTorch, Chain или TensorFlow, то можно [запустить TensorBoard напрямую](#launch-tensorboard) из журнала выполнения эксперимента. 

+ Для экспериментов, которые не выводят файлы, которые понимает TensorBoard, например Scikit-learn или Машинное обучение Azure, используйте [метод `export_to_tensorboard()`](#export), чтобы экспортировать журналы запуска в виде журналов TensorBoard, и запустите TensorBoard из них. 

> [!TIP]
> Сведения в этом документе предназначены главным образом для специалистов по обработке и анализу данных и разработчиков, желающих отслеживать процесс обучения модели. Если вы являетесь администратором, который заинтересован в наблюдении из Машинного обучения Azure за использованием ресурсов и событиями, такими как квоты, завершенные обучающие запуски или завершенные развертывания моделей, см. раздел [Мониторинг Машинного обучения Azure](monitor-azure-machine-learning.md).

## <a name="prerequisites"></a>Предварительные требования

* Чтобы запустить TensorBoard и просмотреть журналы запуска экспериментов, в экспериментах необходимо включить ранее ведение журнала для контроля метрик и производительности.  
* Код в этом документе можно запустить в любой из следующих сред. 
    * Вычислительная операция Машинного обучения Azure — загрузка или установка не требуется
        * Пройдите [руководство по настройке среды и рабочей области](tutorial-1st-experiment-sdk-setup.md), чтобы создать выделенный сервер записных книжек, предварительно загруженный с пакетом SDK и репозиторием с примером.
        * В папке samples на сервере записных книжек найдите две готовые и развернутые записные книжки в следующих каталогах.
            * **как использовать команду azureml > Track-and-Monitoring-эксперименты > tensorboard > Export-Run-History-to-tensorboard > Export-Run-History-to-tensorboard. ipynb**
            * **как использовать-azureml > Track-and-Monitoring-эксперименты > tensorboard > tensorboard > tensorboard. ipynb**
    * Собственный сервер записных книжек Jupyter
       * [Установите пакет SDK для Машинного обучения Azure](/python/api/overview/azure/ml/install) с дополнением `tensorboard`.
        * [Создайте рабочую область Машинного обучения Azure](how-to-manage-workspace.md).  
        * [Создайте файл конфигурации рабочей области](how-to-configure-environment.md#workspace).

## <a name="option-1-directly-view-run-history-in-tensorboard"></a>Вариант 1. Просмотр журнала выполнения непосредственно в TensorBoard

Этот вариант применим для экспериментов, которые изначально выводят файлы журналов в формате TensorBoard, такие как PyTorch, Chainer и TensorFlow. Если в вашем случае это не так, используйте [метод `export_to_tensorboard()`](#export).

В следующем примере кода используется [демонстрационный эксперимент MNIST](https://raw.githubusercontent.com/tensorflow/tensorflow/r1.8/tensorflow/examples/tutorials/mnist/mnist_with_summaries.py) из репозитория TensorFlow в удаленном целевом объекте вычислений среды Машинного обучения Azure. Далее предстоит настроить и запустить запуск для обучения модели TensorFlow, а затем запустить TensorBoard против этого TensorFlow эксперимента.

### <a name="set-experiment-name-and-create-project-folder"></a>Задание имени эксперимента и создание папки проекта

Здесь мы дадим эксперименту имя и создадим для него папку. 
 
```python
from os import path, makedirs
experiment_name = 'tensorboard-demo'

# experiment folder
exp_dir = './sample_projects/' + experiment_name

if not path.exists(exp_dir):
    makedirs(exp_dir)

```

### <a name="download-tensorflow-demo-experiment-code"></a>Загрузка демонстрационного кода эксперимента TensorFlow

В репозитории TensorFlow есть демонстрационная версия MNIST с расширенными инструментами TensorBoard. Для работы с Машинным обучением Azure не нужно вносить никаких изменений в этот демонстрационный код. В следующем коде мы скачиваем код MNIST и сохраняем его во вновь созданную папку эксперимента.

```python
import requests
import os

tf_code = requests.get("https://raw.githubusercontent.com/tensorflow/tensorflow/r1.8/tensorflow/examples/tutorials/mnist/mnist_with_summaries.py")
with open(os.path.join(exp_dir, "mnist_with_summaries.py"), "w") as file:
    file.write(tf_code.text)
```
В файле кода MNIST mnist_with_summaries.py обратите внимание на строки, в которых вызываются методы `tf.summary.scalar()`, `tf.summary.histogram()`, `tf.summary.FileWriter()` и т. д. Эти методы группируют, регистрируют и помечают ключевые метрики экспериментов в журнале выполнения. Метод `tf.summary.FileWriter()` особенно важен, так как он сериализует данные из зарегистрированных вами метрик эксперимента, что позволяет TensorBoard создавать визуализации.

 ### <a name="configure-experiment"></a>Настройка эксперимента

Ниже мы настроим эксперимент и укажем каталоги для журналов и данных. Эти журналы будут отправлены в журнал выполнения, который TensorBoard к ним позже.

> [!Note]
> Для этого примера необходимо установить TensorFlow на локальном компьютере. Кроме того, модуль TensorBoard (то есть модуль, входящий в состав TensorFlow) должен быть доступен для ядра этой записной книжки, так как TensorBoard выполняется именно на локальном компьютере.

```Python
import azureml.core
from azureml.core import Workspace
from azureml.core import Experiment

ws = Workspace.from_config()

# create directories for experiment logs and dataset
logs_dir = os.path.join(os.curdir, "logs")
data_dir = os.path.abspath(os.path.join(os.curdir, "mnist_data"))

if not path.exists(data_dir):
    makedirs(data_dir)

os.environ["TEST_TMPDIR"] = data_dir

# Writing logs to ./logs results in their being uploaded to the run history,
# and thus, made accessible to our TensorBoard instance.
args = ["--log_dir", logs_dir]

# Create an experiment
exp = Experiment(ws, experiment_name)
```

### <a name="create-a-cluster-for-your-experiment"></a>Создание кластера для эксперимента
Для этого эксперимента мы создадим кластер AmlCompute, но эксперименты можно создавать в любой среде, и вы по-прежнему сможете запускать TensorBoard для журналов выполнения экспериментов. 

```Python
from azureml.core.compute import ComputeTarget, AmlCompute

cluster_name = "cpu-cluster"

cts = ws.compute_targets
found = False
if cluster_name in cts and cts[cluster_name].type == 'AmlCompute':
   found = True
   print('Found existing compute target.')
   compute_target = cts[cluster_name]
if not found:
    print('Creating a new compute target...')
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2', 
                                                           max_nodes=4)

    # create the cluster
    compute_target = ComputeTarget.create(ws, cluster_name, compute_config)

compute_target.wait_for_completion(show_output=True, min_node_count=None)

# use get_status() to get a detailed status for the current cluster. 
# print(compute_target.get_status().serialize())
```

[!INCLUDE [low-pri-note](../../includes/machine-learning-low-pri-vm.md)]

### <a name="configure-and-submit-training-run"></a>Настройка и отправка учебного запуска

Настройте задание обучения, создав объект Скриптрунконфиг.

```Python
from azureml.core import ScriptRunConfig
from azureml.core import Environment

# Here we will use the TensorFlow 2.2 curated environment
tf_env = Environment.get(ws, 'AzureML-TensorFlow-2.2-GPU')

src = ScriptRunConfig(source_directory=exp_dir,
                      script='mnist_with_summaries.py',
                      arguments=args,
                      compute_target=compute_target,
                      environment=tf_env)
run = exp.submit(src)
```

### <a name="launch-tensorboard"></a>Запуск TensorBoard

Вы можете запустить TensorBoard во время выполнения или после завершения. В следующем примере создается экземпляр объекта TensorBoard `tb`, который принимает журнал запуска эксперимента, загруженный в `run`, а затем запускает TensorBoard с помощью метода `start()`. 
  
[Конструктор TensorBoard](/python/api/azureml-tensorboard/azureml.tensorboard.tensorboard) принимает массив запусков, поэтому убедитесь, что запуск передается в виде одноэлементного массива.

```python
from azureml.tensorboard import Tensorboard

tb = Tensorboard([run])

# If successful, start() returns a string with the URI of the instance.
tb.start()

# After your job completes, be sure to stop() the streaming otherwise it will continue to run. 
tb.stop()
```

> [!Note]
> Хотя в этом примере используется TensorFlow, TensorBoard можно легко использовать с PyTorch или Chain. TensorFlow должен быть доступен на компьютере с TensorBoard, но не обязательно на компьютере, выполняющем вычисления PyTorch или Chainer. 


<a name="export"></a>

## <a name="option-2-export-history-as-log-to-view-in-tensorboard"></a>Вариант 2. Экспорт журнала для просмотра в TensorBoard

Следующий код настраивает пример эксперимента, начинает процесс ведения журнала с помощью API журнала выполнения Машинного обучения Azure и экспортирует журнал запуска эксперимента в формат, который можно использовать в TensorBoard для визуализации. 

### <a name="set-up-experiment"></a>Настройка эксперимента

Следующий код настраивает новый эксперимент и присваивает каталогу запуска имя `root_run`. 

```python
from azureml.core import Workspace, Experiment
import azureml.core

# set experiment name and run name
ws = Workspace.from_config()
experiment_name = 'export-to-tensorboard'
exp = Experiment(ws, experiment_name)
root_run = exp.start_logging()
```

Здесь мы загружаем набор данных диабетиков — встроенный небольшой набор данных, который поставляется вместе с Scikit-learn, и разбиваем его на наборы для тестов и обучения.

```Python
from sklearn.datasets import load_diabetes
from sklearn.linear_model import Ridge
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import train_test_split
X, y = load_diabetes(return_X_y=True)
columns = ['age', 'gender', 'bmi', 'bp', 's1', 's2', 's3', 's4', 's5', 's6']
x_train, x_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=0)
data = {
    "train":{"x":x_train, "y":y_train},        
    "test":{"x":x_test, "y":y_test}
}
```

### <a name="run-experiment-and-log-metrics"></a>Запуск эксперимента и регистрация метрик

Для этого кода мы обучаем модель линейной регрессии и регистрируем в журнале выполнения ключевые метрики, альфа-коэффициент `alpha` и среднеквадратическую ошибку `mse`.

```Python
from tqdm import tqdm
alphas = [.1, .2, .3, .4, .5, .6 , .7]
# try a bunch of alpha values in a Linear Regression (aka Ridge regression) mode
for alpha in tqdm(alphas):
  # create child runs and fit lines for the resulting models
  with root_run.child_run("alpha" + str(alpha)) as run:
 
   reg = Ridge(alpha=alpha)
   reg.fit(data["train"]["x"], data["train"]["y"])    
 
   preds = reg.predict(data["test"]["x"])
   mse = mean_squared_error(preds, data["test"]["y"])
   # End train and eval

# log alpha, mean_squared_error and feature names in run history
   root_run.log("alpha", alpha)
   root_run.log("mse", mse)
```

### <a name="export-runs-to-tensorboard"></a>Экспорт запусков в TensorBoard

С помощью метода [export_to_tensorboard ()](/python/api/azureml-tensorboard/azureml.tensorboard.export) пакета SDK можно экспортировать журнал выполнения эксперимента Машинного обучения Azure в журналы TensorBoard, чтобы мы могли просматривать их в TensorBoard.  

В следующем коде мы создаем папку `logdir` в текущем рабочем каталоге. В эту папку мы экспортируем журнал выполнения эксперимента и журналы из `root_run`, а затем помечаем запуск как завершенный. 

```Python
from azureml.tensorboard.export import export_to_tensorboard
import os

logdir = 'exportedTBlogs'
log_path = os.path.join(os.getcwd(), logdir)
try:
    os.stat(log_path)
except os.error:
    os.mkdir(log_path)
print(logdir)

# export run history for the project
export_to_tensorboard(root_run, logdir)

root_run.complete()
```

> [!Note]
> Можно также экспортировать в TensorBoard определенный запуск, указав имя запуска: `export_to_tensorboard(run_name, logdir)`

### <a name="start-and-stop-tensorboard"></a>Запуск и остановка TensorBoard
После экспорта журнала выполнения для этого эксперимента можно запустить TensorBoard с помощью метода [start()](/python/api/azureml-tensorboard/azureml.tensorboard.tensorboard#start-start-browser-false-). 

```Python
from azureml.tensorboard import Tensorboard

# The TensorBoard constructor takes an array of runs, so be sure and pass it in as a single-element array here
tb = Tensorboard([], local_root=logdir, port=6006)

# If successful, start() returns a string with the URI of the instance.
tb.start()
```

По завершении убедитесь, что вызвали метод [stop()](/python/api/azureml-tensorboard/azureml.tensorboard.tensorboard#stop--) объекта TensorBoard. В противном случае TensorBoard будет продолжать работать, пока не завершится работа ядра записной книжки. 

```python
tb.stop()
```

## <a name="next-steps"></a>Дальнейшие действия

В этом практическом руководстве вы создали два эксперимента и узнали, как запустить TensorBoard для своих журналов выполнения, чтобы определить области для потенциальной настройки и повторного обучения. 

* Если вы удовлетворены вашей моделью, перейдите к статье [Как развернуть модель](how-to-deploy-and-where.md). 
* Дополнительные сведения о [настройке гиперпараметров](how-to-tune-hyperparameters.md).