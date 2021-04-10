---
title: Руководство по классификации изображений. Обучение моделей
titleSuffix: Azure Machine Learning
description: Сведения о том, как с помощью Машинного обучения Azure обучить модель классификации изображений, используя Scikit-learn в Jupyter Notebook для Python. Это руководство представляет первую часть серии (всего две части).
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: sdgilley
ms.author: sgilley
ms.date: 09/28/2020
ms.custom: seodec18, devx-track-python
ms.openlocfilehash: 85dea807ee09338e7f0e9e388f6b196fd3beef33
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104588670"
---
# <a name="tutorial-train-image-classification-models-with-mnist-data-and-scikit-learn"></a>Руководство по обучению моделей классификации изображений с использованием данных MNIST и Scikit-learn 


В этом руководстве необходимо обучить модель машинного обучения на удаленных вычислительных ресурсах. Вы будете использовать рабочий процесс обучения и развертывания для Машинного обучения Azure в Jupyter Notebook для Python.  Затем можно использовать записную книжку как шаблон для обучения собственной модели машинного обучения со своими данными. Это руководство представляет собой **первую часть серии, состоящей из двух частей**.  

В руководстве показано, как обучить простую модель регрессии для функции логистики с помощью набора данных [MNIST](http://yann.lecun.com/exdb/mnist/) и [Scikit-learn](https://scikit-learn.org) в Машинном обучении Azure. MNIST — это популярный набор данных, состоящий из 70 000 изображений в оттенках серого. Каждое изображение содержит рукописную цифру размером 28 x 28 пикселей, то есть числа от нуля до девяти. Целью является создание многоклассового классификатора для идентификации цифры, которую отображает данное изображение.

Узнайте, как выполнять следующие действия:

> [!div class="checklist"]
> * настроили среду разработки;
> * Подключение к данным и их проверка.
> * Обучайте простую логистическую регрессию на удаленном кластере.
> * Проверка результатов обучения и регистрация наилучшей модели.

Во [второй части этого руководства](tutorial-deploy-models-with-aml.md) описано, как выбрать и развернуть модель.

Если у вас еще нет подписки Azure, создайте бесплатную учетную запись, прежде чем начинать работу. Опробуйте [бесплатную или платную версию Машинного обучения Azure](https://aka.ms/AMLFree) уже сегодня.

>[!NOTE]
> Код в этой статье протестирован с помощью [пакета SDK для Машинного обучения Azure](/python/api/overview/azure/ml/intro) версии 1.13.0.

## <a name="prerequisites"></a>Предварительные требования

* Пройдите [руководство по созданию первого эксперимента Azure ML](tutorial-1st-experiment-sdk-setup.md), чтобы научиться выполнять такие задачи:
    * Создание рабочей области
    * выполните клонирование записной книжки с учебниками в папку в рабочей области;
    * создание облачной вычислительной операции.

* Откройте записную книжку *img-classification-part1-training.ipynb* в клонированной папке *tutorials/image-classification-mnist-data*. 


Это руководство и дополняющий его файл **utils.py** также доступны на сайте [GitHub](https://github.com/Azure/MachineLearningNotebooks/tree/master/tutorials), если вы хотите использовать их в собственной [локальной среде](how-to-configure-environment.md#local). Выполните `pip install azureml-sdk[notebooks] azureml-opendatasets matplotlib`, чтобы установить зависимости для этого руководства.

> [!Important]
> Оставшаяся часть этой статьи содержит то же содержимое, что и записная книжка.  
>
> Перейдите в записную книжку Jupyter Notebook, чтобы вы могли просматривать его во время выполнения кода. 
> Чтобы выполнить одну ячейку кода в записной книжке, щелкните эту ячейку и нажмите клавиши **SHIFT+ВВОД**. Или запустите всю записную книжку, выбрав **Запустить все** в верхней части панели инструментов.

## <a name="set-up-your-development-environment"></a><a name="start"></a>Настройка среды разработки

Все настройки для работы по разработке можно сделать в записной книжке Python. Настройка включает следующие действия:

* Импорт пакетов Python.
* Подключение к рабочей области, что позволяет локальному компьютеру обмениваться данными с удаленными ресурсами.
* Создание эксперимента, чтобы отслеживать все запуски.
* Создание удаленного целевого объекта вычислений для использования в обучении.

### <a name="import-packages"></a>Импорт пакетов

Импортируйте пакеты Python, которые необходимы вам в этом сеансе. Также отобразите версию пакета SDK Службы машинного обучения Azure.

```python
%matplotlib inline
import numpy as np
import matplotlib.pyplot as plt

import azureml.core
from azureml.core import Workspace

# check core SDK version number
print("Azure ML SDK Version: ", azureml.core.VERSION)
```

### <a name="connect-to-a-workspace"></a>Подключение к рабочей области

В существующей рабочей области создайте объект. `Workspace.from_config()` считывает файл **config.json** и загружает данные в объект с именем `ws`.

```python
# load workspace configuration from the config.json file in the current folder.
ws = Workspace.from_config()
print(ws.name, ws.location, ws.resource_group, sep='\t')
```

>[!NOTE]
> При первом запуске приведенного ниже кода, возможно, потребуется пройти проверку подлинности в рабочей области. Следуйте инструкциям на экране.

### <a name="create-an-experiment"></a>Создание эксперимента

Чтобы отслеживать сведения о выполнении в рабочей области, создайте эксперимент. Рабочая область может содержать несколько экспериментов.

```python
from azureml.core import Experiment
experiment_name = 'Tutorial-sklearn-mnist'

exp = Experiment(workspace=ws, name=experiment_name)
```

### <a name="create-or-attach-an-existing-compute-target"></a>Создание или подключение существующего целевого объекта вычислений

Использование Вычислительной среды Машинного обучения Azure (управляемой службы) позволяет специалистам по обработке и анализу данных обучать модели машинного обучения в кластерах виртуальных машин Azure. Часто используются виртуальные машины с поддержкой GPU. В этом руководстве описано, как создать Вычислительную среду Машинного обучения Azure в качестве среды обучения. Вы предоставите код Python для запуска на этой ВМ позже в учебнике. 

Приведенный ниже код создаст вычислительные кластеры, если они еще не существуют в вашей рабочей области. Он настраивает кластер, который будет масштабироваться до 0, когда не используется, и может масштабироваться до 4 узлов. 

 **Создание целевого объекта вычислений занимает около пяти минут.** Если вычислительный ресурс уже находится в рабочей области, код сразу применяет ее, пропуская процесс создания.

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
    provisioning_config = AmlCompute.provisioning_configuration(vm_size=vm_size,
                                                                min_nodes=compute_min_nodes,
                                                                max_nodes=compute_max_nodes)

    # create the cluster
    compute_target = ComputeTarget.create(
        ws, compute_name, provisioning_config)

    # can poll for a minimum number of nodes and for a specific timeout.
    # if no min node count is provided it will use the scale settings for the cluster
    compute_target.wait_for_completion(
        show_output=True, min_node_count=None, timeout_in_minutes=20)

    # For a more detailed view of current AmlCompute status, use get_status()
    print(compute_target.get_status().serialize())
```

Теперь у вас есть необходимые пакеты и вычислительные ресурсы для обучения модели в облаке. 

## <a name="explore-data"></a>Анализ данных

Прежде чем начинать обучение модели, необходимо понимать, какие данные используются для ее обучения. Из этого раздела вы узнаете, как выполнять следующие действия.

* Скачивание набора данных MNIST.
* Отображение примеров изображений.

### <a name="download-the-mnist-dataset"></a>Скачивание набора данных MNIST

Используйте Открытые наборы данных Azure для получения необработанных файлов данных MNIST. [Открытые наборы данных Azure](../open-datasets/overview-what-are-open-datasets.md) — это проверенные общедоступные наборы данных, которые можно использовать для добавления функций конкретных сценариев в решения машинного обучения для создания более точных моделей. Каждый набор данных использует соответствующий класс (в данном случае — `MNIST`) для получения данных различными способами.

Этот код получает данные в виде объекта `FileDataset`, который является подклассом `Dataset`. `FileDataset` ссылается на один или несколько файлов в любом формате, размещенных в хранилищах данных или доступных по общедоступным URL-адресам. С помощью этого класса вы можете скачивать их или подключать к вычислительной среде, создав ссылку на расположение источника данных. Кроме того, вы регистрируете набор данных в своей рабочей области, чтобы упростить получение данных во время обучения.

Чтобы узнать больше о наборах данных и их использовании в пакете SDK, ознакомьтесь с этим [практическим руководством](how-to-create-register-datasets.md).

```python
from azureml.core import Dataset
from azureml.opendatasets import MNIST

data_folder = os.path.join(os.getcwd(), 'data')
os.makedirs(data_folder, exist_ok=True)

mnist_file_dataset = MNIST.get_file_dataset()
mnist_file_dataset.download(data_folder, overwrite=True)

mnist_file_dataset = mnist_file_dataset.register(workspace=ws,
                                                 name='mnist_opendataset',
                                                 description='training and test dataset',
                                                 create_new_version=True)
```

### <a name="display-some-sample-images"></a>Отображение некоторых примеров изображений

Загрузите сжатые файлы в массивы `numpy`. Затем с помощью `matplotlib` постройте график 30 случайных изображений из набора данных с подписями над ними. На этом шаге потребуется функция `load_data`, которая содержится в файле `utils.py`. Этот файл находится в папке примера. Разместите его в той же папке, где находится эта записная книжка. Функция `load_data` выполняет анализ сжатых файлов, преобразовывая их в массивы numpy.

```python
# make sure utils.py is in the same directory as this code
from utils import load_data
import glob


# note we also shrink the intensity values (X) from 0-255 to 0-1. This helps the model converge faster.
X_train = load_data(glob.glob(os.path.join(data_folder,"**/train-images-idx3-ubyte.gz"), recursive=True)[0], False) / 255.0
X_test = load_data(glob.glob(os.path.join(data_folder,"**/t10k-images-idx3-ubyte.gz"), recursive=True)[0], False) / 255.0
y_train = load_data(glob.glob(os.path.join(data_folder,"**/train-labels-idx1-ubyte.gz"), recursive=True)[0], True).reshape(-1)
y_test = load_data(glob.glob(os.path.join(data_folder,"**/t10k-labels-idx1-ubyte.gz"), recursive=True)[0], True).reshape(-1)


# now let's show some randomly chosen images from the traininng set.
count = 0
sample_size = 30
plt.figure(figsize=(16, 6))
for i in np.random.permutation(X_train.shape[0])[:sample_size]:
    count = count + 1
    plt.subplot(1, sample_size, count)
    plt.axhline('')
    plt.axvline('')
    plt.text(x=10, y=-10, s=y_train[i], fontsize=18)
    plt.imshow(X_train[i].reshape(28, 28), cmap=plt.cm.Greys)
plt.show()
```

Случайный пример изображений приведен ниже.

![Случайный пример изображений](./media/tutorial-train-models-with-aml/digits.png)

Теперь у вас есть представление о том, как выглядят эти изображения, и прогноз ожидаемого результата.

## <a name="train-on-a-remote-cluster"></a>Обучение на удаленном кластере

Для выполнения этой задачи запустите задание в кластере удаленного обучения, который мы настроили ранее.  Чтобы отправить задание, нужно:
* Создание каталога
* Создание сценария обучения
* создать конфигурацию выполнения скрипта;
* отправить задание.

### <a name="create-a-directory"></a>Создание каталога

Создайте каталог для доставки необходимого кода с компьютера на удаленный ресурс.

```python
import os
script_folder = os.path.join(os.getcwd(), "sklearn-mnist")
os.makedirs(script_folder, exist_ok=True)
```

### <a name="create-a-training-script"></a>Создание сценария обучения

Чтобы отправить задание в кластер, необходимо сначала создать сценарий обучения. Для создания сценария обучения выполните следующий код, который называется `train.py`, в каталоге, который был только что создан.

```python
%%writefile $script_folder/train.py

import argparse
import os
import numpy as np
import glob

from sklearn.linear_model import LogisticRegression
import joblib

from azureml.core import Run
from utils import load_data

# let user feed in 2 parameters, the dataset to mount or download, and the regularization rate of the logistic regression model
parser = argparse.ArgumentParser()
parser.add_argument('--data-folder', type=str, dest='data_folder', help='data folder mounting point')
parser.add_argument('--regularization', type=float, dest='reg', default=0.01, help='regularization rate')
args = parser.parse_args()

data_folder = args.data_folder
print('Data folder:', data_folder)

# load train and test set into numpy arrays
# note we scale the pixel intensity values to 0-1 (by dividing it with 255.0) so the model can converge faster.
X_train = load_data(glob.glob(os.path.join(data_folder, '**/train-images-idx3-ubyte.gz'), recursive=True)[0], False) / 255.0
X_test = load_data(glob.glob(os.path.join(data_folder, '**/t10k-images-idx3-ubyte.gz'), recursive=True)[0], False) / 255.0
y_train = load_data(glob.glob(os.path.join(data_folder, '**/train-labels-idx1-ubyte.gz'), recursive=True)[0], True).reshape(-1)
y_test = load_data(glob.glob(os.path.join(data_folder, '**/t10k-labels-idx1-ubyte.gz'), recursive=True)[0], True).reshape(-1)

print(X_train.shape, y_train.shape, X_test.shape, y_test.shape, sep = '\n')

# get hold of the current run
run = Run.get_context()

print('Train a logistic regression model with regularization rate of', args.reg)
clf = LogisticRegression(C=1.0/args.reg, solver="liblinear", multi_class="auto", random_state=42)
clf.fit(X_train, y_train)

print('Predict the test set')
y_hat = clf.predict(X_test)

# calculate accuracy on the prediction
acc = np.average(y_hat == y_test)
print('Accuracy is', acc)

run.log('regularization rate', np.float(args.reg))
run.log('accuracy', np.float(acc))

os.makedirs('outputs', exist_ok=True)
# note file saved in the outputs folder is automatically uploaded into experiment record
joblib.dump(value=clf, filename='outputs/sklearn_mnist_model.pkl')
```

Обратите внимание на то, как сценарий получает данные и сохраняет модели.

+ Скрипт обучения считывает аргумент, чтобы найти каталог с данными. При отправке задания позже вы указываете хранилище данных для этого аргумента: ```parser.add_argument('--data-folder', type=str, dest='data_folder', help='data directory mounting point')```.

+ Скрипт обучения сохраняет модель в каталог с именем **outputs**. Все записи в этом каталоге автоматически передаются в рабочую область. Вы будете обращаться к модели из этого каталога далее в этом руководстве. `joblib.dump(value=clf, filename='outputs/sklearn_mnist_model.pkl')`

+ Для правильной загрузки набора данных учебный скрипт требует файл `utils.py`. Следующий код копирует `utils.py` в `script_folder`, чтобы получить доступ к файлу вместе со сценарием обучения на удаленном ресурсе.

  ```python
  import shutil
  shutil.copy('utils.py', script_folder)
  ```

### <a name="configure-the-training-job"></a>Настройка задания обучения

Создайте объект [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig), чтобы указать сведения о конфигурации для вашего задания обучения, в том числе скрипт обучения, используемую среду и целевой объект вычислений, на котором будет выполняться задание. Настройте ScriptRunConfig, указав следующее:

* Выберите каталог, который содержит скрипт. Все файлы в этом каталоге передаются в узел кластера для выполнения.
* Целевой объект вычисления. В этом примере используется созданный вычислительный кластер Службы машинного обучения Azure.
* Имя скрипта обучения — **train.py**.
* Среду, содержащую библиотеки, необходимые для выполнения скрипта.
* Аргументы, требуемые от скрипта обучения.

В этом руководстве целевой средой является AMLCompute. Все файлы в папке скриптов передаются для выполнения в узлы кластера. Параметр **--data_folder** настраивается на использование хранилища данных.

Сначала создайте среду, содержащую: библиотеку scikit-learn, azureml-dataset-runtime для доступа к набору данных и azureml-defaults с зависимостями для метрик ведения журнала. azureml-defaults также содержат зависимости, необходимые для развертывания модели в качестве веб-службы далее во 2 части руководства.

После определения среды зарегистрируйте ее в рабочей области, чтобы повторно использовать ее во 2 части этого руководства.

```python
from azureml.core.environment import Environment
from azureml.core.conda_dependencies import CondaDependencies

# to install required packages
env = Environment('tutorial-env')
cd = CondaDependencies.create(pip_packages=['azureml-dataset-runtime[pandas,fuse]', 'azureml-defaults'], conda_packages=['scikit-learn==0.22.1'])

env.python.conda_dependencies = cd

# Register environment to re-use later
env.register(workspace=ws)
```

Затем создайте ScriptRunConfig, указав скрипт обучения, целевой объект вычислений и среду.

```python
from azureml.core import ScriptRunConfig

args = ['--data-folder', mnist_file_dataset.as_mount(), '--regularization', 0.5]

src = ScriptRunConfig(source_directory=script_folder,
                      script='train.py', 
                      arguments=args,
                      compute_target=compute_target,
                      environment=env)
```

### <a name="submit-the-job-to-the-cluster"></a>Отправка задания в кластер

Запустите эксперимент, отправив объект ScriptRunConfig:

```python
run = exp.submit(config=src)
run
```

Так как вызов является асинхронным, сразу после запуска задания он возвращает состояние **Preparing** (Подготовка) или **Running** (Выполнение).

## <a name="monitor-a-remote-run"></a>Мониторинг удаленного выполнения

Обычно первый запуск занимает **около 10 минут**. Но для последующих запусков используется тот же образ, если зависимости скрипта не изменяются. Благодаря этому запуск контейнера будет намного быстрее.

Что происходит, пока вы ожидаете завершения?

- **Создание образа**. Будет создан образ Docker, которые соответствует среде Python, указанной средой Машинного обучения Azure. Изображение загружается в рабочую область. Создание и отправка изображений занимает **около пяти минут**.

  Этот этап выполняется один раз для каждой среды Python, а для последующих запусков контейнер помещается в кэш. Во время создания образа журналы будут отправлены в журнал выполнения. С помощью этих журналов вы можете отслеживать ход создания образа.

- **Масштабирование**. Если для выполнения на удаленном кластере требуется больше узлов, чем доступно в текущий момент, дополнительные узлы добавляются автоматически. Масштабирование обычно занимает **около пяти минут.**

- **Выполняется**. На этом этапе необходимые скрипты и файлы отправляются на целевой объект вычислений. Затем подключаются или копируются хранилища данных. После этого выполняется **entry_script**. Пока выполняется задание, стандартный выход **stdout** и каталог **./logs** направляются в журнал выполнения. С помощью этих журналов вы можете отслеживать ход создания образа.

- **Постобработка**. Каталог **./outputs** завершенного выполнения копируется в журнал выполнения в рабочей области, чтобы вы могли обращаться к этим результатам.

Ход выполнения запущенного задания можно контролировать несколькими способами. В этом руководстве используются мини-приложение Jupyter и метод `wait_for_completion`.

### <a name="jupyter-widget"></a>Мини-приложение Jupyter

Отслеживайте ход выполнения с помощью [мини-приложения Jupyter](/python/api/azureml-widgets/azureml.widgets). Как и отправка выполнения, мини-приложение работает асинхронно и в реальном времени предоставляет обновления каждые 10–15 секунд, пока не завершит задание.

```python
from azureml.widgets import RunDetails
RunDetails(run).show()
```

В конце обучения мини-приложение будет выглядеть следующим образом:

![Мини-приложение записной книжки](./media/tutorial-train-models-with-aml/widget.png)

Если необходимо отменить выполнение, вы можете выполнить [эти инструкции](./how-to-manage-runs.md).

### <a name="get-log-results-upon-completion"></a>Получение результатов записи по завершении

Обучение и мониторинг модели происходит в фоновом режиме. Прежде, чем выполнять другой код, дождитесь завершения обучения модели. `wait_for_completion` позволяет узнать, когда завершится обучение модели.

```python
run.wait_for_completion(show_output=False)  # specify True for a verbose log
```

### <a name="display-run-results"></a>Отображение результатов потокового выполнения

Теперь у вас есть модель, которую обучили на удаленном кластере. Извлеките точность модели, выполнив такую команду.

```python
print(run.get_metrics())
```

В выходных данных показано, что точность удаленной модели — 0,9204.

`{'regularization rate': 0.8, 'accuracy': 0.9204}`

В следующем руководстве эта модель рассматривается подробнее.

## <a name="register-model"></a>Регистрация модели

На последнем шаге скрипт обучения записал файл `outputs/sklearn_mnist_model.pkl` в каталог с именем `outputs` на виртуальной машине кластера, где выполнялось задание. Все содержимое специального каталога `outputs` автоматически отправляется в рабочую область. Это содержимое отображается в записи о выполнении эксперимента в рабочей области. Теперь файл модели также доступен в рабочей области.

Вы увидите файлы, связанные с этим выполнением.

```python
print(run.get_file_names())
```

Зарегистрируйте модель в рабочей области, чтобы вы или другие участники смогли позже запросить, проверить и развернуть ее.

```python
# register model
model = run.register_model(model_name='sklearn_mnist',
                           model_path='outputs/sklearn_mnist_model.pkl')
print(model.name, model.id, model.version, sep='\t')
```

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]

Вычислительный кластер Машинного обучения Azure можно удалить. Но для него включено автомасштабирование с минимальным размером, равным нулю. Это означает, что этот ресурс не создает дополнительных расходов, когда он не используется.

```python
# Optionally, delete the Azure Machine Learning Compute cluster
compute_target.delete()
```

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого руководства по Машинному обучению Azure с помощью Python вы выполнили такие задачи:

> [!div class="checklist"]
> * настроили среду разработки;
> * Подключение к данным и их проверка.
> * Обучение нескольких моделей на удаленном кластере с помощью популярной библиотеки машинного обучения scikit-learn.
> * Проверка сведений об обучении и регистрация наилучшей модели.

Теперь вы готовы развернуть зарегистрированную модель с помощью инструкций, представленных в следующей части серии руководств.

> [!div class="nextstepaction"]
> [Руководство 2. Развертывание моделей](tutorial-deploy-models-with-aml.md)
