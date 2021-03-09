---
title: Обучение с помощью наборов данных машинного обучения
titleSuffix: Azure Machine Learning
description: Узнайте, как сделать данные доступными для локального или удаленного вычислений для обучения модели с помощью Машинное обучение Azure наборов данных.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: sihhu
author: MayMSFT
manager: cgronlun
ms.reviewer: nibaccam
ms.date: 07/31/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, data4ml
ms.openlocfilehash: 15bad877be00e143ce6f6956a4e1f23378c275c0
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102521787"
---
# <a name="train-models-with-azure-machine-learning-datasets"></a>Обучение моделей с помощью наборов данных Машинное обучение Azure 

Из этой статьи вы узнаете, как работать с [машинное обучение Azureными наборами данных](/python/api/azureml-core/azureml.core.dataset%28class%29) для обучения моделей машинного обучения.  Наборы данных можно использовать в локальном или удаленном целевом объекте вычислений, не беспокоясь о строках подключения или путях к данным. 

Машинное обучение Azure наборы данных обеспечивают простую интеграцию с Машинное обучение Azureными функциями [обучения, такими](/python/api/azureml-train-core/azureml.train.hyperdrive) как [скриптрунконфиг](/python/api/azureml-core/azureml.core.scriptrunconfig), а также [конвейеры и машинное обучение Azure](./how-to-create-machine-learning-pipelines.md).

Если вы не готовы сделать данные доступными для обучения модели, но хотите загрузить данные в записную книжку для просмотра данных, см. статью [изучение данных в наборе](how-to-create-register-datasets.md#explore-data)данных. 

## <a name="prerequisites"></a>Предварительные условия

Чтобы создать и обучить наборы данных, вам потребуется:

* Подписка Azure. Если у вас еще нет подписки Azure, создайте бесплатную учетную запись, прежде чем начинать работу. Опробуйте [бесплатную или платную версию Машинного обучения Azure](https://aka.ms/AMLFree) уже сегодня.

* [Рабочая область машинное обучение Azure](how-to-manage-workspace.md).

* [Установленный пакет SDK машинное обучение Azure для Python](/python/api/overview/azure/ml/install) (>= 1.13.0), который включает в себя `azureml-datasets` пакеты.

> [!Note]
> Некоторые классы наборов данных имеют зависимости от пакета [azureml-](/python/api/azureml-dataprep/) DataMarket. Для пользователей Linux эти классы поддерживаются только в следующих дистрибутивах: Red Hat Enterprise Linux, Ubuntu, Fedora и CentOS.

## <a name="consume-datasets-in-machine-learning-training-scripts"></a>Использование наборов данных в сценариях обучения машинного обучения

Если структурированные данные еще не зарегистрированы в качестве набора данных, создайте Табулардатасет и используйте их непосредственно в обучающем скрипте для локального или удаленного эксперимента.

В этом примере вы создаете незарегистрированный [табулардатасет](/python/api/azureml-core/azureml.data.tabulardataset) и указываете его в качестве аргумента скрипта в объекте скриптрунконфиг для обучения. Если вы хотите повторно использовать этот Табулардатасет с другими экспериментами в рабочей области, см. статью [как зарегистрировать наборы данных в рабочей области](how-to-create-register-datasets.md#register-datasets).

### <a name="create-a-tabulardataset"></a>Создание Табулардатасет

Следующий код создает незарегистрированный Табулардатасет из URL-адреса.  

```Python
from azureml.core.dataset import Dataset

web_path ='https://dprepdata.blob.core.windows.net/demo/Titanic.csv'
titanic_ds = Dataset.Tabular.from_delimited_files(path=web_path)
```

Объекты Табулардатасет обеспечивают возможность загрузки данных в Табулардатасет в таблицу данных Pandas или Spark, чтобы вы могли работать с привычными библиотеками подготовки и обучения данных, не покидая записной книжки.

### <a name="access-dataset-in-training-script"></a>Доступ к набору данных в обучающем скрипте

Следующий код настраивает аргумент сценария `--input-data` , который будет указываться при настройке запуска обучения (см. следующий раздел). Когда табличный набор данных передается в качестве значения аргумента, Azure ML разрешит это в идентификатор набора данных, который затем можно использовать для доступа к набору данных в обучающем скрипте (без необходимости жестко кодировать имя или идентификатор набора данных в скрипте). Затем он использует [`to_pandas_dataframe()`](/python/api/azureml-core/azureml.data.tabulardataset#to-pandas-dataframe-on-error--null---out-of-range-datetime--null--) метод для загрузки этого набора данных в кадр данных Pandas для дальнейшего изучения и подготовки перед обучением.

> [!Note]
> Если исходный источник данных содержит NaN, пустые строки или пустые значения, то при использовании `to_pandas_dataframe()` эти значения заменяются значением *null* .

Если необходимо загрузить подготовленные данные в новый набор данных из Pandas данных в памяти, запишите данные в локальный файл, например в Parquet, и создайте новый набор данных из этого файла. Дополнительные сведения о [создании наборов данных](how-to-create-register-datasets.md).

```Python
%%writefile $script_folder/train_titanic.py

import argparse
from azureml.core import Dataset, Run

parser = argparse.ArgumentParser()
parser.add_argument("--input-data", type=str)
args = parser.parse_args()

run = Run.get_context()
ws = run.experiment.workspace

# get the input dataset by ID
dataset = Dataset.get_by_id(ws, id=args.input_data)

# load the TabularDataset to pandas DataFrame
df = dataset.to_pandas_dataframe()
```

### <a name="configure-the-training-run"></a>Настройка обучающего запуска

Объект [скриптрунконфиг](/python/api/azureml-core/azureml.core.scriptrun) используется для настройки и отправки обучающего запуска.

Этот код создает объект Скриптрунконфиг, `src` который указывает

* Каталог скрипта для скриптов. Все файлы в этом каталоге передаются в узел кластера для выполнения.
* Сценарий обучения, *train_titanic. Корректировка*.
* Входной набор данных для обучения, `titanic_ds` как аргумент скрипта. Служба машинного обучения Azure разрешит это для соответствующего идентификатора набора данных при его передаче в скрипт.
* Целевой объект вычислений для выполнения.
* Среда для запуска.

```python
from azureml.core import ScriptRunConfig

src = ScriptRunConfig(source_directory=script_folder,
                      script='train_titanic.py',
                      # pass dataset as an input with friendly name 'titanic'
                      arguments=['--input-data', titanic_ds.as_named_input('titanic')],
                      compute_target=compute_target,
                      environment=myenv)
                             
# Submit the run configuration for your training run
run = experiment.submit(src)
run.wait_for_completion(show_output=True)                             
```

## <a name="mount-files-to-remote-compute-targets"></a>Подключение файлов к удаленным целевым объектам вычислений

Если у вас есть неструктурированные данные, создайте [филедатасет](/python/api/azureml-core/azureml.data.filedataset) и либо подключите, либо Скачайте файлы данных, чтобы сделать их доступными для целей удаленного вычислений. Узнайте, когда использовать [подключение и скачивание](#mount-vs-download) для удаленных экспериментов. 

В следующем примере создается Филедатасет, а набор данных подключается к целевому объекту вычислений путем передачи его в качестве аргумента в обучающий сценарий. 

> [!Note]
> При использовании пользовательского базового образа DOCKER необходимо установить предохранитель через `apt-get install -y fuse` как зависимость для подключения набора данных к работе. Узнайте, как [создать пользовательский образ сборки](how-to-deploy-custom-docker-image.md#build-a-custom-base-image).

### <a name="create-a-filedataset"></a>Создание FileDataset

В следующем примере создается незарегистрированный Филедатасет из URL-адресов. Дополнительные сведения о [создании наборов данных](how-to-create-register-datasets.md) из других источников.

```Python
from azureml.core.dataset import Dataset

web_paths = [
            'http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/train-labels-idx1-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz'
            ]
mnist_ds = Dataset.File.from_files(path = web_paths)
```

### <a name="configure-the-training-run"></a>Настройка обучающего запуска

Рекомендуется передавать набор данных в качестве аргумента при подключении через `arguments` параметр `ScriptRunConfig` конструктора. Это позволит получить путь к данным (точка подключения) в скрипте обучения с помощью аргументов. Таким образом вы сможете использовать один и тот же сценарий обучения для локальной отладки и удаленного обучения на любой облачной платформе.

В следующем примере создается объект Скриптрунконфиг, который передается в Филедатасет с помощью `arguments` . После отправки выполнения файлы данных, на которые ссылается набор данных, `mnist_ds` будут подключены к целевому объекту вычислений.

```python
from azureml.core import ScriptRunConfig

src = ScriptRunConfig(source_directory=script_folder,
                      script='train_mnist.py',
                      # the dataset will be mounted on the remote compute and the mounted path passed as an argument to the script
                      arguments=['--data-folder', mnist_ds.as_mount(), '--regularization', 0.5],
                      compute_target=compute_target,
                      environment=myenv)

# Submit the run configuration for your training run
run = experiment.submit(src)
run.wait_for_completion(show_output=True)
```

### <a name="retrieve-data-in-your-training-script"></a>Получение данных в обучающем скрипте

В следующем коде показано, как получить данные в скрипте.

```Python
%%writefile $script_folder/train_mnist.py

import argparse
import os
import numpy as np
import glob

from utils import load_data

# retrieve the 2 arguments configured through `arguments` in the ScriptRunConfig
parser = argparse.ArgumentParser()
parser.add_argument('--data-folder', type=str, dest='data_folder', help='data folder mounting point')
parser.add_argument('--regularization', type=float, dest='reg', default=0.01, help='regularization rate')
args = parser.parse_args()

data_folder = args.data_folder
print('Data folder:', data_folder)

# get the file paths on the compute
X_train_path = glob.glob(os.path.join(data_folder, '**/train-images-idx3-ubyte.gz'), recursive=True)[0]
X_test_path = glob.glob(os.path.join(data_folder, '**/t10k-images-idx3-ubyte.gz'), recursive=True)[0]
y_train_path = glob.glob(os.path.join(data_folder, '**/train-labels-idx1-ubyte.gz'), recursive=True)[0]
y_test = glob.glob(os.path.join(data_folder, '**/t10k-labels-idx1-ubyte.gz'), recursive=True)[0]

# load train and test set into numpy arrays
X_train = load_data(X_train_path, False) / 255.0
X_test = load_data(X_test_path, False) / 255.0
y_train = load_data(y_train_path, True).reshape(-1)
y_test = load_data(y_test, True).reshape(-1)
```

## <a name="mount-vs-download"></a>Подключение загрузки VS

Подключение или скачивание файлов любого формата поддерживается для наборов данных, созданных из хранилища BLOB-объектов Azure, файлов Azure Azure Data Lake Storage 1-го поколения, Azure Data Lake Storage 2-го поколения, базы данных SQL Azure и базы данных Azure для PostgreSQL. 

При **подключении** набора данных необходимо прикрепить файлы, на которые ссылается набор данных, к каталогу (точке подключения) и сделать его доступным в целевом объекте вычислений. Поддерживается подключение для вычислений на основе Linux, в том числе Машинное обучение Azure вычислений, виртуальных машин и HDInsight. 

При **скачивании** набора данных все файлы, на которые ссылается набор данных, будут скачаны в целевой объект вычислений. Загрузка поддерживается для всех типов вычислений. 

Если сценарий обрабатывает все файлы, на которые ссылается набор данных, а вычислительный диск может соответствовать полному набору данных, то скачивание рекомендуется, чтобы избежать издержек, связанных с потоковой передачей данных из служб хранилища. Если размер данных превышает размер диска вычислений, загрузка невозможна. В этом сценарии рекомендуется монтирование, так как только файлы данных, используемые скриптом, загружаются во время обработки.

Следующий код подключается `dataset` к каталогу Temp по адресу `mounted_path`

```python
import tempfile
mounted_path = tempfile.mkdtemp()

# mount dataset onto the mounted_path of a Linux-based compute
mount_context = dataset.mount(mounted_path)

mount_context.start()

import os
print(os.listdir(mounted_path))
print (mounted_path)
```

## <a name="get-datasets-in-machine-learning-scripts"></a>Получение наборов данных в скриптах машинного обучения

Зарегистрированные наборы данных доступны как локально, так и удаленно в таких кластерах, как Машинное обучение Azure вычислений. Чтобы получить доступ к зарегистрированному набору данных во время экспериментов, используйте следующий код для доступа к рабочей области и получения набора данных, который использовался в ранее отправленном запуске. По умолчанию [`get_by_name()`](/python/api/azureml-core/azureml.core.dataset.dataset#get-by-name-workspace--name--version--latest--) метод в `Dataset` классе возвращает последнюю версию набора данных, зарегистрированную в рабочей области.

```Python
%%writefile $script_folder/train.py

from azureml.core import Dataset, Run

run = Run.get_context()
workspace = run.experiment.workspace

dataset_name = 'titanic_ds'

# Get a dataset by name
titanic_ds = Dataset.get_by_name(workspace=workspace, name=dataset_name)

# Load a TabularDataset into pandas DataFrame
df = titanic_ds.to_pandas_dataframe()
```

## <a name="access-source-code-during-training"></a>Доступ к исходному коду во время обучения

Хранилище BLOB-объектов Azure имеет более высокую пропускную способность, чем общая папка Azure, и может масштабироваться до большого числа параллельно запущенных заданий. По этой причине рекомендуется настроить запуски для использования хранилища BLOB-объектов для передачи файлов исходного кода.

В следующем примере кода в конфигурации запуска указывается хранилище данных BLOB-объектов, которое будет использоваться для передачи исходного кода.

```python 
# workspaceblobstore is the default blob storage
src.run_config.source_directory_data_store = "workspaceblobstore" 
```

## <a name="notebook-examples"></a>Примеры записных книжек

+ В этой статье демонстрируются и развертываются [записные книжки набора данных](https://aka.ms/dataset-tutorial) .
+ См. статью как [параметризе наборы данных в конвейерах машинного обучения](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines/intro-to-pipelines/aml-pipelines-showcasing-dataset-and-pipelineparameter.ipynb).

## <a name="troubleshooting"></a>Диагностика

* **Ошибка инициализации набора данных: время ожидания готовности точки подключения** истекло: 
  * Если у вас нет исходящих правил [группы безопасности сети](../virtual-network/network-security-groups-overview.md) и используется `azureml-sdk>=1.12.0` , обновление `azureml-dataset-runtime` и его зависимости будут актуальными для конкретной дополнительной версии или если вы используете ее в запуске, воссоздайте среду, чтобы она могла установить последнее исправление с исправлением. 
  * Если вы используете `azureml-sdk<1.12.0` , обновите до последней версии.
  * Если у вас есть правила исходящего трафика NSG, убедитесь, что имеется правило исходящего трафика, разрешающее весь трафик для тега службы `AzureResourceMonitor` .

### <a name="overloaded-azurefile-storage"></a>Перегруженное хранилище Азурефиле

Если появляется сообщение об ошибке `Unable to upload project files to working directory in AzureFile because the storage is overloaded` , используйте следующие обходные пути.

Если вы используете общую папку для других рабочих нагрузок, таких как передача данных, рекомендуется использовать большие двоичные объекты, чтобы файловый ресурс можно было использовать для отправки запусков. Вы также можете разделить рабочую нагрузку между двумя разными рабочими областями.

### <a name="passing-data-as-input"></a>Передача данных в качестве входного

*  **Типиррор: FileNotFound: отсутствует такой файл или каталог**: Эта ошибка возникает, если указанный путь к файлу не находится там, где расположен файл. Необходимо убедиться в том, что путь к файлу соответствует месту подключения набора данных к целевому объекту вычислений. Чтобы обеспечить детерминированное состояние, рекомендуется использовать абстрактный путь при подключении набора данных к целевому объекту вычислений. Например, в следующем примере выполняется подключение набора данных в корне файловой системы целевого объекта вычислений `/tmp` . 
    
    ```python
    # Note the leading / in '/tmp/dataset'
    script_params = {
        '--data-folder': dset.as_named_input('dogscats_train').as_mount('/tmp/dataset'),
    } 
    ```

    Если не указать начальную косую черту ("/"), необходимо добавить к рабочему каталогу префикс, например, в `/mnt/batch/.../tmp/dataset` целевом объекте вычислений, чтобы указать, где нужно подключить набор данных.


## <a name="next-steps"></a>Дальнейшие действия

* [Автоматическое обучение моделей машинного обучения](how-to-auto-train-remote.md) с помощью табулардатасетс.

* [Обучение моделей классификации изображений](https://aka.ms/filedataset-samplenotebook) с помощью филедатасетс.

* [Обучение с наборами данных с помощью конвейеров](./how-to-create-machine-learning-pipelines.md).