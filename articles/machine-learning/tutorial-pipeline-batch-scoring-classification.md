---
title: Руководство по конвейерам Машинного обучения для пакетной оценки
titleSuffix: Azure Machine Learning
description: В рамках этого руководства вы создадите конвейер машинного обучения для выполнения пакетной оценки. Вы сможете уделить максимум внимания машинному обучению, а не инфраструктуре и автоматизации.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: lobrien
ms.author: laobri
ms.reviewer: laobri
ms.date: 10/13/2020
ms.custom: contperf-fy20q4, devx-track-python
ms.openlocfilehash: 570bfed5ae5fc6fafea36b9ed1f2673a0daae22b
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102521515"
---
# <a name="tutorial-build-an-azure-machine-learning-pipeline-for-batch-scoring"></a>Руководство по Создание конвейеров Машинного обучения Azure для пакетной оценки

Из этого расширенного учебника вы узнаете, как создавать [конвейеры Машинного обучения Azure](concept-ml-pipelines.md) для выполнения заданий пакетной оценки. Конвейеры машинного обучения оптимизируют ваш рабочий процесс за счет ускорения, мобильности и повторного использования, позволяя вам сосредоточиться непосредственно на задачах обучения, а не на сложностях инфраструктуры. После создания и публикации конвейера необходимо настроить конечную точку REST, позволяющую активировать конвейер с любой библиотеки HTTP на любой платформе. 

В этом руководстве для классификации изображений без метки используется [Inception-V3](https://arxiv.org/abs/1512.00567), предварительно обученная модель сверточной нейронной сети, реализованная в Tensorflow. 

В этом руководстве выполняются следующие задачи:

> [!div class="checklist"]
> * Настройка рабочей области 
> * Скачивание и сохранение примера данных.
> * Создание объектов DataSet для извлечения и вывода данных.
> * Скачивание, подготовка и регистрация модели в рабочей области
> * Подготовка целевых объектов вычислений и создание сценария оценки.
> * Использование класса `ParallelRunStep` для асинхронной оценки пакетной службы
> * Создание, запуск и публикация конвейера.
> * Включение конечной точки REST для конвейера.

Если у вас еще нет подписки Azure, создайте бесплатную учетную запись, прежде чем начинать работу. Опробуйте [бесплатную или платную версию Машинного обучения Azure](https://aka.ms/AMLFree) уже сегодня.

## <a name="prerequisites"></a>Предварительные требования

* Если у вас еще нет рабочей области Машинного обучения Azure или виртуальной машины записной книжки, выполните инструкции, приведенные в [первой части руководства по установке](tutorial-1st-experiment-sdk-setup.md).
* Завершив настройку, используйте тот же сервер записных книжек, чтобы открыть записную книжку *tutorials/machine-learning-pipelines-advanced/tutorial-pipeline-batch-scoring-classification.ipynb*.

Если вы хотите открыть руководство по установке в собственной [локальной среде](how-to-configure-environment.md#local), доступ к нему можно получить на [GitHub](https://github.com/Azure/MachineLearningNotebooks/tree/master/tutorials). Выполните команду `pip install azureml-sdk[notebooks] azureml-pipeline-core azureml-pipeline-steps pandas requests`, чтобы получить необходимые пакеты.

## <a name="configure-workspace-and-create-a-datastore"></a>Настройка рабочей области и создание хранилища данных

В имеющейся рабочей области Машинного обучения Azure создайте объект.

```python
from azureml.core import Workspace
ws = Workspace.from_config()
```

> [!IMPORTANT]
> Этот фрагмент кода ищет конфигурацию рабочей области в текущем или родительском каталоге. Дополнительные сведения о создании рабочей области см. в статье [Создание рабочих областей машинного обучения Azure и управление ими](how-to-manage-workspace.md). Дополнительные сведения о сохранении конфигурации в файле см. в разделе [Создание файла конфигурации рабочей области](how-to-configure-environment.md#workspace).

## <a name="create-a-datastore-for-sample-images"></a>Создание хранилища данных для образцов изображений

В учетной записи `pipelinedata` извлеките образец общедоступных данных вычисления ImageNet из общедоступного контейнера больших двоичных объектов `sampledata`. Выполните вызов `register_azure_blob_container()`, чтобы предоставить рабочей области `images_datastore` доступ к данным. Затем задайте хранилище данных рабочей области по умолчанию в качестве выходного. Используйте выходное хранилище данных для оценки выходных данных в конвейере.

Дополнительные сведения о доступе к данным см. в [этом разделе](./how-to-access-data.md).

```python
from azureml.core.datastore import Datastore

batchscore_blob = Datastore.register_azure_blob_container(ws, 
                      datastore_name="images_datastore", 
                      container_name="sampledata", 
                      account_name="pipelinedata", 
                      overwrite=True)

def_data_store = ws.get_default_datastore()
```

## <a name="create-dataset-objects"></a>Создание объектов DataSet

При создании конвейера объекты `Dataset` используются для чтения данных из хранилищ данных рабочей области, а объекты `OutputFileDatasetConfig` — для передачи промежуточных данных между шагами конвейера.

> [!Important]
> Пример оценки пакетной службы в этом руководстве использует только один шаг конвейера. В вариантах использования с несколькими шагами типичный поток будет включать следующие шаги:
>
> 1. Использование объектов `Dataset` в качестве *входных данных* для извлечения необработанных данных, выполнение определенных преобразований, а затем *вывод* с объектом `OutputFileDatasetConfig`.
>
> 2. Использование *выходного объекта* `OutputFileDatasetConfig` из предыдущего шага в качестве *входного объекта*. Повторение для последующих шагов.

В этом сценарии создайте объекты `Dataset`, соответствующие каталогам хранилища данных для входных изображений и меток классификации (значения y-test). Кроме того, создайте объект `OutputFileDatasetConfig` для выходных данных пакетной оценки.

```python
from azureml.core.dataset import Dataset
from azureml.data import OutputFileDatasetConfig

input_images = Dataset.File.from_files((batchscore_blob, "batchscoring/images/"))
label_ds = Dataset.File.from_files((batchscore_blob, "batchscoring/labels/"))
output_dir = OutputFileDatasetConfig(name="scores")
```

Зарегистрируйте наборы данных в рабочей области, если они понадобятся вам позже. Это необязательный шаг.

```python

input_images = input_images.register(workspace = ws, name = "input_images")
label_ds = label_ds.register(workspace = ws, name = "label_ds")
```

## <a name="download-and-register-the-model"></a>Скачивание и регистрация модели

Скачайте предварительно обученную модель Tensorflow, чтобы использовать ее для пакетной оценки в конвейере. Сначала создайте локальный каталог, в котором будет храниться модель. Затем скачайте и извлеките модель.

```python
import os
import tarfile
import urllib.request

if not os.path.isdir("models"):
    os.mkdir("models")
    
response = urllib.request.urlretrieve("http://download.tensorflow.org/models/inception_v3_2016_08_28.tar.gz", "model.tar.gz")
tar = tarfile.open("model.tar.gz", "r:gz")
tar.extractall("models")
```

Выполните регистрацию модели в рабочей области, что позволит быстро извлекать ее в процессе конвейера. В статической функции `register()` параметр `model_name` является ключом, который используется для определения расположения модели в пакете SDK.

```python
from azureml.core.model import Model
 
model = Model.register(model_path="models/inception_v3.ckpt",
                       model_name="inception",
                       tags={"pretrained": "inception"},
                       description="Imagenet trained tensorflow inception",
                       workspace=ws)
```

## <a name="create-and-attach-the-remote-compute-target"></a>Создание и вложение удаленного целевого объекта вычислений

Конвейеры машинного обучения невозможно запустить локально, поэтому их можно запускать в облачных ресурсах или *удаленных целевых объектах вычислений*. Удаленный целевой объект вычислений — это повторно используемая виртуальная среда вычислений, в которой выполняются эксперименты и рабочие процессы машинного обучения. 

Выполните приведенный ниже код, чтобы создать целевой объект [`AmlCompute`](/python/api/azureml-core/azureml.core.compute.amlcompute.amlcompute) с поддержкой GPU, а затем вложите его в свою рабочую область. Дополнительные сведения о целевых объектах вычислений см. в [тематической статье](./concept-compute-target.md).


```python
from azureml.core.compute import AmlCompute, ComputeTarget
from azureml.exceptions import ComputeTargetException
compute_name = "gpu-cluster"

# checks to see if compute target already exists in workspace, else create it
try:
    compute_target = ComputeTarget(workspace=ws, name=compute_name)
except ComputeTargetException:
    config = AmlCompute.provisioning_configuration(vm_size="STANDARD_NC6",
                                                   vm_priority="lowpriority", 
                                                   min_nodes=0, 
                                                   max_nodes=1)

    compute_target = ComputeTarget.create(workspace=ws, name=compute_name, provisioning_configuration=config)
    compute_target.wait_for_completion(show_output=True, min_node_count=None, timeout_in_minutes=20)
```

## <a name="write-a-scoring-script"></a>Написание сценария оценки

Для оценки необходимо создать скрипт пакетной оценки под названием `batch_scoring.py`, а затем записать его в текущий каталог. Скрипт принимает входные изображения, применяет модель классификации, а затем выводит прогнозы в файл результатов.

Сценарий `batch_scoring.py` принимает приведенные ниже параметры, которые передаются из класса `ParallelRunStep`, который вы создадите позже:

- `--model_name`: Имя используемой модели.
- `--labels_dir`: Расположение файла `labels.txt`.

Инфраструктура конвейеров использует класс `ArgumentParser` для передачи параметров в шаги конвейера. Например, в следующем коде первый аргумент `--model_name` получает идентификатор свойства `model_name`. В функции `init()` для доступа к этому свойству используется `Model.get_model_path(args.model_name)`.


```python
%%writefile batch_scoring.py

import os
import argparse
import datetime
import time
import tensorflow as tf
from math import ceil
import numpy as np
import shutil
from tensorflow.contrib.slim.python.slim.nets import inception_v3

from azureml.core import Run
from azureml.core.model import Model
from azureml.core.dataset import Dataset

slim = tf.contrib.slim

image_size = 299
num_channel = 3


def get_class_label_dict(labels_dir):
    label = []
    labels_path = os.path.join(labels_dir, 'labels.txt')
    proto_as_ascii_lines = tf.gfile.GFile(labels_path).readlines()
    for l in proto_as_ascii_lines:
        label.append(l.rstrip())
    return label


def init():
    global g_tf_sess, probabilities, label_dict, input_images

    parser = argparse.ArgumentParser(description="Start a tensorflow model serving")
    parser.add_argument('--model_name', dest="model_name", required=True)
    parser.add_argument('--labels_dir', dest="labels_dir", required=True)
    args, _ = parser.parse_known_args()

    label_dict = get_class_label_dict(args.labels_dir)
    classes_num = len(label_dict)

    with slim.arg_scope(inception_v3.inception_v3_arg_scope()):
        input_images = tf.placeholder(tf.float32, [1, image_size, image_size, num_channel])
        logits, _ = inception_v3.inception_v3(input_images,
                                              num_classes=classes_num,
                                              is_training=False)
        probabilities = tf.argmax(logits, 1)

    config = tf.ConfigProto()
    config.gpu_options.allow_growth = True
    g_tf_sess = tf.Session(config=config)
    g_tf_sess.run(tf.global_variables_initializer())
    g_tf_sess.run(tf.local_variables_initializer())

    model_path = Model.get_model_path(args.model_name)
    saver = tf.train.Saver()
    saver.restore(g_tf_sess, model_path)


def file_to_tensor(file_path):
    image_string = tf.read_file(file_path)
    image = tf.image.decode_image(image_string, channels=3)

    image.set_shape([None, None, None])
    image = tf.image.resize_images(image, [image_size, image_size])
    image = tf.divide(tf.subtract(image, [0]), [255])
    image.set_shape([image_size, image_size, num_channel])
    return image


def run(mini_batch):
    result_list = []
    for file_path in mini_batch:
        test_image = file_to_tensor(file_path)
        out = g_tf_sess.run(test_image)
        result = g_tf_sess.run(probabilities, feed_dict={input_images: [out]})
        result_list.append(os.path.basename(file_path) + ": " + label_dict[result[0]])
    return result_list
```

> [!TIP]
> Конвейер в этом руководстве содержит только один шаг и записывает выходные данные в файл. Для многошаговых конвейеров также используется `ArgumentParser`, чтобы определить каталог для записи выходных данных для последующего выполнения действий. Пример передачи данных между несколькими шагами конвейера с использованием конструктивного шаблона `ArgumentParser` см. в [записной книжке](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines/nyc-taxi-data-regression-model-building/nyc-taxi-data-regression-model-building.ipynb).

## <a name="build-the-pipeline"></a>Создание конвейера

Перед запуском конвейера создайте объект, который определяет среду Python и создает зависимости, необходимые скрипту `batch_scoring.py`. Основной зависимостью является TensorFlow, но кроме нее установите `azureml-core` и `azureml-dataprep[fuse]`, требуемые для ParallelRunStep. Кроме того, следует указать поддержку Docker и Docker-GPU.

```python
from azureml.core import Environment
from azureml.core.conda_dependencies import CondaDependencies
from azureml.core.runconfig import DEFAULT_GPU_IMAGE

cd = CondaDependencies.create(pip_packages=["tensorflow-gpu==1.15.2",
                                            "azureml-core", "azureml-dataprep[fuse]"])
env = Environment(name="parallelenv")
env.python.conda_dependencies = cd
env.docker.base_image = DEFAULT_GPU_IMAGE
```

### <a name="create-the-configuration-to-wrap-the-script"></a>Создание конфигурации для заключения скрипта в оболочку

Создайте шаг конвейера, используя сценарий, конфигурацию среды и параметры. Укажите целевой объект вычислений, уже подключенный к вашей рабочей области.

```python
from azureml.pipeline.steps import ParallelRunConfig

parallel_run_config = ParallelRunConfig(
    environment=env,
    entry_script="batch_scoring.py",
    source_directory=".",
    output_action="append_row",
    mini_batch_size="20",
    error_threshold=1,
    compute_target=compute_target,
    process_count_per_node=2,
    node_count=1
)
```

### <a name="create-the-pipeline-step"></a>Создание шага конвейера

Шаг конвейера — это объект, который инкапсулирует все необходимое для запуска конвейера, включая:

* параметры среды и зависимостей;
* вычислительный ресурс для запуска конвейера;
* входные и выходные данные и любые пользовательские параметры;
* ссылку на скрипт или логику пакета SDK для запуска во время выполнения шага.

Несколько классов наследуют от родительского класса [`PipelineStep`](/python/api/azureml-pipeline-core/azureml.pipeline.core.builder.pipelinestep). Для создания шага можно выбрать классы для использования конкретных платформ или стеков. В этом примере используется класс `ParallelRunStep` для определения логики шага с помощью пользовательского скрипта Python. Если аргумент вашего скрипта является вводом или выводом шага, его необходимо определить *в* массиве `arguments`*и* в параметре `input` или `output` соответственно. 

Ссылка на объект в массиве `outputs` становится доступной в качестве *входных данных* для последующего шага конвейера в сценариях с несколькими шагами.

```python
from azureml.pipeline.steps import ParallelRunStep
from datetime import datetime

parallel_step_name = "batchscoring-" + datetime.now().strftime("%Y%m%d%H%M")

label_config = label_ds.as_named_input("labels_input")

batch_score_step = ParallelRunStep(
    name=parallel_step_name,
    inputs=[input_images.as_named_input("input_images")],
    output=output_dir,
    arguments=["--model_name", "inception",
               "--labels_dir", label_config],
    side_inputs=[label_config],
    parallel_run_config=parallel_run_config,
    allow_reuse=False
)
```

Список всех классов для разных типов шагов см. в [этой статье](/python/api/azureml-pipeline-steps/azureml.pipeline.steps).

## <a name="submit-the-pipeline"></a>Отправка конвейера

Теперь запустите конвейер. Сначала создайте объект `Pipeline` с помощью ссылки на вашу рабочую область и созданный шаг конвейера. Параметр `steps` является массивом шагов. В этом случае для пакетной оценки выполняется только один шаг. Чтобы создать конвейеры с несколькими шагами, разместите шаги по порядку в этом массиве.

Затем используйте функцию `Experiment.submit()`, чтобы передать конвейер для выполнения. Функция `wait_for_completion` выводит журналы во время процесса сборки конвейера. В журналах можно просмотреть текущий ход выполнения.

> [!IMPORTANT]
> Первый запуск конвейера занимает примерно *15 минут*. Необходимо скачать все зависимости, создать образ Docker и подготовить и создать среду Python. Повторный запуск конвейера занимает значительно меньше времени, так как эти ресурсы используются повторно, а не создаются. Однако общее время запуска для конвейера зависит от рабочей нагрузки ваших скриптов и процессов, выполняемых в каждом шаге конвейера.

```python
from azureml.core import Experiment
from azureml.pipeline.core import Pipeline

pipeline = Pipeline(workspace=ws, steps=[batch_score_step])
pipeline_run = Experiment(ws, 'Tutorial-Batch-Scoring').submit(pipeline)
pipeline_run.wait_for_completion(show_output=True)
```

### <a name="download-and-review-output"></a>Скачивание и просмотр выходных данных

Запустите приведенный ниже код, чтобы скачать выходной файл, созданный из скрипта `batch_scoring.py`. Затем изучите результаты оценки.

```python
import pandas as pd

batch_run = next(pipeline_run.get_children())
batch_output = batch_run.get_output_data("scores")
batch_output.download(local_path="inception_results")

for root, dirs, files in os.walk("inception_results"):
    for file in files:
        if file.endswith("parallel_run_step.txt"):
            result_file = os.path.join(root, file)

df = pd.read_csv(result_file, delimiter=":", header=None)
df.columns = ["Filename", "Prediction"]
print("Prediction has ", df.shape[0], " rows")
df.head(10)
```

## <a name="publish-and-run-from-a-rest-endpoint"></a>Публикация и запуск из конечной точки REST

Выполните приведенный ниже код, чтобы опубликовать конвейер в рабочей области. В своей рабочей области студии машинного обучения Azure вы можете просмотреть метаданные для конвейера, включая журнал выполнения и продолжительность. Вы также можете запустить в студии конвейер вручную.

Публикация конвейера позволяет конечной точке REST повторно запускать конвейер из любой библиотеки HTTP на любой платформе.

```python
published_pipeline = pipeline_run.publish_pipeline(
    name="Inception_v3_scoring", description="Batch scoring using Inception v3 model", version="1.0")

published_pipeline
```

Чтобы запустить конвейер из конечной точки REST, вам потребуется заголовок проверки подлинности типа OAuth2 Bearer. В следующем примере для иллюстрации используется интерактивная проверка подлинности, но для большинства производственных сценариев, требующих автоматической или автономной проверки подлинности, необходимо использовать проверку подлинности субъекта-службы, как [описано в этой статье](how-to-setup-authentication.md).

Проверка подлинности субъекта-службы включает в себя создание *регистрации приложения* в *Azure Active Directory*. Сначала необходимо создать секрет клиента, а затем предоставить субъекту-службе *доступ роли* к вашей рабочей области машинного обучения. Используйте класс [`ServicePrincipalAuthentication`](/python/api/azureml-core/azureml.core.authentication.serviceprincipalauthentication) для управления потоком проверки подлинности. 

[`InteractiveLoginAuthentication`](/python/api/azureml-core/azureml.core.authentication.interactiveloginauthentication) и `ServicePrincipalAuthentication` наследуются от `AbstractAuthentication`. В обоих случаях используйте функцию [`get_authentication_header()`](/python/api/azureml-core/azureml.core.authentication.abstractauthentication#get-authentication-header--) для извлечения заголовка:

```python
from azureml.core.authentication import InteractiveLoginAuthentication

interactive_auth = InteractiveLoginAuthentication()
auth_header = interactive_auth.get_authentication_header()
```

Получите URL-адрес REST из свойства `endpoint` объекта опубликованного конвейера. Кроме того, URL-адрес REST можно найти в рабочей области в студии машинного обучения Azure. 

Создайте запрос HTTP POST к конечной точке. Укажите заголовок проверки подлинности в запросе. Добавьте объект полезных данных JSON с именем эксперимента.

Выполните запрос на активацию запуска. Включите код для доступа к ключу `Id` из словаря ответов, чтобы получить значение идентификатора запуска.

```python
import requests

rest_endpoint = published_pipeline.endpoint
response = requests.post(rest_endpoint, 
                         headers=auth_header, 
                         json={"ExperimentName": "Tutorial-Batch-Scoring",
                               "ParameterAssignments": {"process_count_per_node": 6}})
run_id = response.json()["Id"]
```

Используйте его для отслеживания состояния нового запуска. Для завершения нового запуска требуется еще 10–15 минут. 

Новый запуск будет выглядеть аналогично конвейеру, который вы запустили ранее в этом руководстве. Вы можете не просматривать все выходные данные.

```python
from azureml.pipeline.core.run import PipelineRun
from azureml.widgets import RunDetails

published_pipeline_run = PipelineRun(ws.experiments["Tutorial-Batch-Scoring"], run_id)
RunDetails(published_pipeline_run).show()
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Не выполняйте инструкции из этого раздела, если вы собираетесь изучать другие руководства по Машинному обучению Azure.

### <a name="stop-the-compute-instance"></a>Остановка вычислительной операции

[!INCLUDE [aml-stop-server](../../includes/aml-stop-server.md)]

### <a name="delete-everything"></a>Удаление всех ресурсов

Если вы не планируете использовать созданные ресурсы, удалите их, чтобы с вас не взималась плата.

1. На портале Azure в меню слева выберите **Группы ресурсов**.
1. В списке групп ресурсов выберите созданную группу ресурсов.
1. Выберите **Удалить группу ресурсов**.
1. Введите имя группы ресурсов. Затем нажмите кнопку **Удалить**.

Вы также можете сохранить группу ресурсов, но удалить одну рабочую область. Отобразите свойства рабочей области и нажмите кнопку **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

Изучив это руководство о конвейерах машинного обучения, вы выполнили следующие задачи:

> [!div class="checklist"]
> * Создание конвейера с зависимостями среды для запуска в удаленном ресурсе вычислений GPU.
> * Создание скрипта оценки для выполнения пакетных прогнозирований с помощью предварительно обученной модели Tensorflow.
> * Публикация конвейера и его включение для запуска с конечной точки REST.

Дополнительные примеры создания конвейеров с помощью пакета SDK для машинного обучения см. в [репозитории записных книжек](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/machine-learning-pipelines).
