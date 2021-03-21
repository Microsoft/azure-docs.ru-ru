---
title: Развертывание модели для вывода с помощью GPU
titleSuffix: Azure Machine Learning
description: В этой статье рассказывается, как использовать Машинное обучение Azure для развертывания модели глубокого обучения Tensorflow с поддержкой GPU в качестве веб-службы. запросы на выведение запросов и оценок.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: vaidyas
author: csteegz
ms.reviewer: larryfr
ms.date: 06/17/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, deploy
ms.openlocfilehash: 6797c32ded5c12bbac3fafa1eabd1e6f74d28e07
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102519254"
---
# <a name="deploy-a-deep-learning-model-for-inference-with-gpu"></a>Развертывание модели глубокого обучения для вывода с помощью GPU


В этой статье объясняется, как использовать Машинное обучение Azure для развертывания модели с поддержкой GPU в качестве веб-службы. Сведения в этой статье основаны на развертывании модели в службе Azure Kubernetes Service (AKS). Кластер AKS предоставляет ресурс GPU, который используется моделью для вывода.

Вывод или оценка модели — это этап, в котором для создания прогнозов используется развернутая модель. Использование графических процессоров вместо ЦП обеспечивает преимущества производительности при параллелизуемыеных вычислениях.

> [!IMPORTANT]
> Для развертываний веб-служб вывод GPU поддерживается только в службе Kubernetes Azure. Для вывода с использованием __конвейера машинного обучения__ GPU поддерживаются только в машинное обучение Azureных вычислениях. Дополнительные сведения об использовании конвейеров машинного обучения см. в разделе [учебник. создание конвейера машинное обучение Azure для пакетной оценки](tutorial-pipeline-batch-scoring-classification.md). 

> [!TIP]
> Несмотря на то, что фрагменты кода в этой статье используют модель TensorFlow, эту информацию можно применить к любой платформе машинного обучения, поддерживающей GPU.

> [!NOTE]
> Сведения в этой статье основаны на сведениях в статье [развертывание в службе Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md) . Когда эта статья обычно охватывает развертывание в AKS, в этой статье рассматривается развертывание GPU.

## <a name="prerequisites"></a>Предварительные требования

* Рабочая область машинного обучения Azure. Дополнительные сведения см. в статье [создание машинное обучение Azure рабочей области](how-to-manage-workspace.md).

* Среда разработки Python с установленным пакетом SDK для Машинное обучение Azure. Дополнительные сведения см. в разделе [машинное обучение Azure SDK](/python/api/overview/azure/ml/install).  

* Зарегистрированная модель, использующая GPU.

    * Дополнительные сведения о регистрации моделей см. в разделе [Развертывание моделей](how-to-deploy-and-where.md#registermodel).

    * Сведения о создании и регистрации модели Tensorflow, используемой для создания этого документа, см. в разделе [обучение модели Tensorflow](how-to-train-tensorflow.md).

* Общее представление [о том, как и где развертываются модели](how-to-deploy-and-where.md).

## <a name="connect-to-your-workspace"></a>Подключение к рабочей области

Чтобы подключиться к существующей рабочей области, используйте следующий код:

> [!IMPORTANT]
> Этот фрагмент кода ищет конфигурацию рабочей области в текущем или родительском каталоге. Дополнительные сведения о создании рабочей области см. в статье [Создание рабочих областей машинного обучения Azure и управление ими](how-to-manage-workspace.md).   Дополнительные сведения о сохранении конфигурации в файле см. в разделе [Создание файла конфигурации рабочей области](how-to-configure-environment.md#workspace).

```python
from azureml.core import Workspace

# Connect to the workspace
ws = Workspace.from_config()
```

## <a name="create-a-kubernetes-cluster-with-gpus"></a>Создание кластера Kubernetes с помощью GPU

Служба Azure Kubernetes предоставляет множество различных параметров GPU. Для определения модели можно использовать любой из них. Полный список возможностей и затрат см. [в списке виртуальных машин серии N](https://azure.microsoft.com/pricing/details/virtual-machines/linux/#n-series) .

В следующем коде показано, как создать новый кластер AKS для рабочей области.

```python
from azureml.core.compute import ComputeTarget, AksCompute
from azureml.exceptions import ComputeTargetException

# Choose a name for your cluster
aks_name = "aks-gpu"

# Check to see if the cluster already exists
try:
    aks_target = ComputeTarget(workspace=ws, name=aks_name)
    print('Found existing compute target')
except ComputeTargetException:
    print('Creating a new compute target...')
    # Provision AKS cluster with GPU machine
    prov_config = AksCompute.provisioning_configuration(vm_size="Standard_NC6")

    # Create the cluster
    aks_target = ComputeTarget.create(
        workspace=ws, name=aks_name, provisioning_configuration=prov_config
    )

    aks_target.wait_for_completion(show_output=True)
```

> [!IMPORTANT]
> В Azure будет взиматься плата, если существует кластер AKS. Не забудьте удалить кластер AKS после завершения работы с ним.

Дополнительные сведения об использовании AKS с Машинное обучение Azure см. в статье [развертывание в службе Kubernetes Azure](how-to-deploy-azure-kubernetes-service.md).

## <a name="write-the-entry-script"></a>Написание сценария записи

Сценарий записи получает данные, отправленные в веб-службу, передает их в модель и возвращает результаты оценки. Следующий скрипт загружает модель Tensorflow при запуске, а затем использует модель для оценки данных.

> [!TIP]
> Скрипт относится только к модели. Например, сценарий должен быть уверен, что платформа будет использоваться с моделью, форматами данных и т. д.

```python
import json
import numpy as np
import os
import tensorflow as tf

from azureml.core.model import Model


def init():
    global X, output, sess
    tf.reset_default_graph()
    model_root = os.getenv('AZUREML_MODEL_DIR')
    # the name of the folder in which to look for tensorflow model files
    tf_model_folder = 'model'
    saver = tf.train.import_meta_graph(
        os.path.join(model_root, tf_model_folder, 'mnist-tf.model.meta'))
    X = tf.get_default_graph().get_tensor_by_name("network/X:0")
    output = tf.get_default_graph().get_tensor_by_name("network/output/MatMul:0")

    sess = tf.Session()
    saver.restore(sess, os.path.join(model_root, tf_model_folder, 'mnist-tf.model'))


def run(raw_data):
    data = np.array(json.loads(raw_data)['data'])
    # make prediction
    out = output.eval(session=sess, feed_dict={X: data})
    y_hat = np.argmax(out, axis=1)
    return y_hat.tolist()
```

Этот файл называется `score.py` . Дополнительные сведения о сценариях входа см. [в разделе как и где развертывать](how-to-deploy-and-where.md).

## <a name="define-the-conda-environment"></a>Определение среды conda

Файл среды conda определяет зависимости для службы. Он включает зависимости, необходимые как для модели, так и для скрипта записи. Обратите внимание, что необходимо указать значения azureml-Default с версии >= 1.0.45 в качестве зависимости PIP, так как она содержит функции, необходимые для размещения модели в качестве веб-службы. Следующая YAML определяет среду для модели Tensorflow. Он указывает `tensorflow-gpu` , что будет использовать GPU, используемый в этом развертывании:

```yaml
name: project_environment
dependencies:
  # The python interpreter version.
  # Currently Azure ML only supports 3.5.2 and later.
- python=3.6.2

- pip:
  # You must list azureml-defaults as a pip dependency
  - azureml-defaults>=1.0.45
  - numpy
  - tensorflow-gpu=1.12
channels:
- conda-forge
```

В этом примере файл сохраняется как `myenv.yml` .

## <a name="define-the-deployment-configuration"></a>Определение конфигурации развертывания

> [!IMPORTANT]
> AKS не позволяет использовать в модулях Pod общий доступ к GPU. можно использовать только столько реплик веб-службы с поддержкой GPU, сколько GPU в кластере.

Конфигурация развертывания определяет среду службы Azure Kubernetes, используемую для запуска веб-службы:

```python
from azureml.core.webservice import AksWebservice

gpu_aks_config = AksWebservice.deploy_configuration(autoscale_enabled=False,
                                                    num_replicas=3,
                                                    cpu_cores=2,
                                                    memory_gb=4)
```

Дополнительные сведения см. в справочной документации по [AksService.deploy_configuration](/python/api/azureml-core/azureml.core.webservice.akswebservice#deploy-configuration-autoscale-enabled-none--autoscale-min-replicas-none--autoscale-max-replicas-none--autoscale-refresh-seconds-none--autoscale-target-utilization-none--collect-model-data-none--auth-enabled-none--cpu-cores-none--memory-gb-none--enable-app-insights-none--scoring-timeout-ms-none--replica-max-concurrent-requests-none--max-request-wait-time-none--num-replicas-none--primary-key-none--secondary-key-none--tags-none--properties-none--description-none--gpu-cores-none--period-seconds-none--initial-delay-seconds-none--timeout-seconds-none--success-threshold-none--failure-threshold-none--namespace-none--token-auth-enabled-none--compute-target-name-none-).

## <a name="define-the-inference-configuration"></a>Определение конфигурации вывода

Конфигурация вывода указывает на скрипт записи и объект среды, который использует образ DOCKER с поддержкой GPU. Обратите внимание, что файл YAML, используемый для определения среды, должен содержать идентификаторы azureml-Defaults с Version >= 1.0.45 в качестве зависимости PIP, так как он содержит функции, необходимые для размещения модели в качестве веб-службы.

```python
from azureml.core.model import InferenceConfig
from azureml.core.environment import Environment, DEFAULT_GPU_IMAGE

myenv = Environment.from_conda_specification(name="myenv", file_path="myenv.yml")
myenv.docker.base_image = DEFAULT_GPU_IMAGE
inference_config = InferenceConfig(entry_script="score.py", environment=myenv)
```

Дополнительные сведения о средах см. в статье [создание сред для обучения и развертывания и управление ими](how-to-use-environments.md).
Дополнительные сведения см. в справочной документации по [инференцеконфиг](/python/api/azureml-core/azureml.core.model.inferenceconfig).

## <a name="deploy-the-model"></a>Развертывание модели

Разверните модель в кластере AKS и подождите, пока она создаст службу.

```python
from azureml.core.model import Model

# Name of the web service that is deployed
aks_service_name = 'aks-dnn-mnist'
# Get the registerd model
model = Model(ws, "tf-dnn-mnist")
# Deploy the model
aks_service = Model.deploy(ws,
                           models=[model],
                           inference_config=inference_config,
                           deployment_config=gpu_aks_config,
                           deployment_target=aks_target,
                           name=aks_service_name)

aks_service.wait_for_deployment(show_output=True)
print(aks_service.state)
```

Дополнительные сведения см. в справочной документации по [модели](/python/api/azureml-core/azureml.core.model.model).

## <a name="issue-a-sample-query-to-your-service"></a>Выдача примера запроса в службу

Отправка тестового запроса в развернутую модель. При отправке изображения JPEG в модель она оценивает изображение. Следующий пример кода скачивает тестовые данные, а затем выбирает случайное тестовое изображение для отправки в службу.

```python
# Used to test your webservice
import os
import urllib
import gzip
import numpy as np
import struct
import requests

# load compressed MNIST gz files and return numpy arrays
def load_data(filename, label=False):
    with gzip.open(filename) as gz:
        struct.unpack('I', gz.read(4))
        n_items = struct.unpack('>I', gz.read(4))
        if not label:
            n_rows = struct.unpack('>I', gz.read(4))[0]
            n_cols = struct.unpack('>I', gz.read(4))[0]
            res = np.frombuffer(gz.read(n_items[0] * n_rows * n_cols), dtype=np.uint8)
            res = res.reshape(n_items[0], n_rows * n_cols)
        else:
            res = np.frombuffer(gz.read(n_items[0]), dtype=np.uint8)
            res = res.reshape(n_items[0], 1)
    return res

# one-hot encode a 1-D array
def one_hot_encode(array, num_of_classes):
    return np.eye(num_of_classes)[array.reshape(-1)]

# Download test data
os.makedirs('./data/mnist', exist_ok=True)
urllib.request.urlretrieve('http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz', filename='./data/mnist/test-images.gz')
urllib.request.urlretrieve('http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz', filename='./data/mnist/test-labels.gz')

# Load test data from model training
X_test = load_data('./data/mnist/test-images.gz', False) / 255.0
y_test = load_data('./data/mnist/test-labels.gz', True).reshape(-1)

# send a random row from the test set to score
random_index = np.random.randint(0, len(X_test)-1)
input_data = "{\"data\": [" + str(list(X_test[random_index])) + "]}"

api_key = aks_service.get_keys()[0]
headers = {'Content-Type': 'application/json',
           'Authorization': ('Bearer ' + api_key)}
resp = requests.post(aks_service.scoring_uri, input_data, headers=headers)

print("POST to url", aks_service.scoring_uri)
print("label:", y_test[random_index])
print("prediction:", resp.text)
```

Дополнительные сведения о создании клиентского приложения см. в разделе [Создание клиента для использования развернутой веб-службы](how-to-consume-web-service.md).

## <a name="clean-up-the-resources"></a>очищать ресурсы.

Если вы создали кластер AKS специально для этого примера, удалите ресурсы после завершения работы.

> [!IMPORTANT]
> Счета Azure выставляются в зависимости от того, как долго развертывается кластер AKS. Не забудьте очистить ее после завершения.

```python
aks_service.delete()
aks_target.delete()
```

## <a name="next-steps"></a>Дальнейшие действия

* [Развертывание модели на FPGA](how-to-deploy-fpga-web-service.md)
* [Развертывание модели с помощью ONNX](concept-onnx.md#deploy-onnx-models-in-azure)
* [Обучение моделей Tensorflow DNN](how-to-train-tensorflow.md)