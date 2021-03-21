---
title: Развертывание зашифрованной службы для размещения (Предварительная версия)
titleSuffix: Azure Machine Learning
description: Узнайте, как использовать Microsoft SEAL для развертывания зашифрованной службы прогнозирования для классификации изображений
author: luisquintanilla
ms.author: luquinta
ms.date: 07/09/2020
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, devx-track-python, deploy, responsible-ml
ms.openlocfilehash: a2c8b765f109408b8cb08f78a4b3ad2283f35eb9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98223590"
---
# <a name="how-to-deploy-an-encrypted-inferencing-web-service-preview"></a>Развертывание зашифрованной веб-службы для последующего просмотра (Предварительная версия)

Узнайте, как развернуть модель классификации изображений в виде зашифрованной веб-службы для операций вывода с помощью [Экземпляров контейнеров Azure](../container-instances/index.yml) (ACI). Веб-служба представляет собой образ контейнера Docker, который содержит модель и логику оценки.

В этом руководстве вы будете использовать службу "Машинное обучение Azure", чтобы научиться выполнять следующие задачи:

> [!div class="checklist"]
> * Настройка сред
> * Развертывание зашифрованной веб-службы для операций вывода
> * Подготовка тестовых данных
> * Создание зашифрованных прогнозов
> * Очистка ресурсов

ACI — это идеальное решение для тестирования и понимания рабочего процесса развертывания модели. Для масштабируемых рабочих развертываний рекомендуется использовать Службу Azure Kubernetes. Дополнительные сведения см. в статье [Развертывание моделей с помощью Службы машинного обучения Azure](./how-to-deploy-and-where.md).

В этом примере используется метод шифрования [гомоморфное шифрование](https://github.com/Microsoft/SEAL#homomorphic-encryption). Гомоморфное шифрование позволяет выполнять вычисления с зашифрованными данными без необходимости доступа к секретному ключу (ключу расшифровки). Результаты вычислений шифруются и могут быть прочитаны только владельцем секретного ключа. 

## <a name="prerequisites"></a>Предварительные требования

В этом руководстве предполагается, что у вас есть зарегистрированная в службе "Машинное обучение Azure" модель классификации изображений. В противном случае зарегистрируйте модель с помощью [предварительно обученной модели](https://github.com/Azure/MachineLearningNotebooks/raw/master/tutorials/image-classification-mnist-data/sklearn_mnist_model.pkl) или создайте собственную, выполнив [обучение модели классификации изображений в Машинном обучении Azure](tutorial-train-models-with-aml.md).

## <a name="configure-local-environment"></a>Настройка локальной среды

В записной книжке Jupyter

1. Импортируйте пакеты Python, необходимые для этого примера.

    ```python
    %matplotlib inline
    import numpy as np
    import matplotlib.pyplot as plt

    import azureml.core

    # display the core SDK version number
    print("Azure ML SDK Version: ", azureml.core.VERSION)
    ```

2. Установите библиотеку гомоморфного шифрования для безопасного вывода.

    > [!NOTE]
    > Пакет `encrypted-inference` в настоящее время находится на этапе предварительной версии.

    [`encrypted-inference`](https://pypi.org/project/encrypted-inference) — это библиотека, которая содержит привязки для зашифрованного вывода на основе [Microsoft SEAL](https://github.com/Microsoft/SEAL).

    ```python
    !pip install encrypted-inference==0.9
    ```

## <a name="configure-the-inferencing-environment"></a>Настройка среды для операций вывода

Создайте среду для вывода и добавьте пакет `encrypted-inference` в качестве зависимости conda.

```python
from azureml.core.environment import Environment
from azureml.core.conda_dependencies import CondaDependencies

# to install required packages
env = Environment('tutorial-env')
cd = CondaDependencies.create(pip_packages=['azureml-dataprep[pandas,fuse]>=1.1.14', 'azureml-defaults', 'azure-storage-blob', 'encrypted-inference==0.9'], conda_packages = ['scikit-learn==0.22.1'])

env.python.conda_dependencies = cd

# Register environment to re-use later
env.register(workspace = ws)
```

## <a name="deploy-encrypted-inferencing-web-service"></a>Развертывание зашифрованной веб-службы для операций вывода

Разверните модель в качестве веб-службы, размещенной в ACI.

Чтобы создать правильную среду для ACI, предоставьте следующие компоненты.

* Сценарий оценки, чтобы показать, как использовать модель.
* Файл конфигурации для создания ACI.
* Обученная модель.

### <a name="create-scoring-script"></a>Создание сценария оценки

Создайте сценарий оценки `score.py`, используемый веб-службой для вывода.

В сценарий оценки необходимо включить две обязательные функции.

* Функция `init()`, которая обычно загружает модель в глобальный объект. Эта функция выполняется только один раз при запуске контейнера Docker.
* Функция `run(input_data)` использует модель для прогнозирования значения на основе входных данных. Входные и выходные данные для запуска обычно используют JSON для сериализации и десериализации, но поддерживаются и другие форматы. Функция извлекает открытые ключи для гомоморфного шифрования, которые передаются вызывающей службой.

```python
%%writefile score.py
import json
import os
import pickle
import joblib
from azure.storage.blob import BlobServiceClient, BlobClient, ContainerClient, PublicAccess
from encrypted.inference.eiserver import EIServer

def init():
    global model
    # AZUREML_MODEL_DIR is an environment variable created during deployment.
    # It is the path to the model folder (./azureml-models/$MODEL_NAME/$VERSION)
    # For multiple models, it points to the folder containing all deployed models (./azureml-models)
    model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'sklearn_mnist_model.pkl')
    model = joblib.load(model_path)

    global server
    server = EIServer(model.coef_, model.intercept_, verbose=True)

def run(raw_data):

    json_properties = json.loads(raw_data)

    key_id = json_properties['key_id']
    conn_str = json_properties['conn_str']
    container = json_properties['container']
    data = json_properties['data']

    # download the public keys from blob storage
    blob_service_client = BlobServiceClient.from_connection_string(conn_str=conn_str)
    blob_client = blob_service_client.get_blob_client(container=container, blob=key_id)
    public_keys = blob_client.download_blob().readall()

    result = {}
    # make prediction
    result = server.predict(data, public_keys)

    # you can return any data type as long as it is JSON-serializable
    return result
```

### <a name="create-configuration-file"></a>Создание файла конфигурации

Создайте файл конфигурации развертывания и укажите необходимое для контейнера ACI количество ЦП и объем ОЗУ в гигабайтах. Так как он зависит от модели, для многих моделей по умолчанию обычно бывает достаточно одного ядра и одного ГБ оперативной памяти. Если в дальнейшем вам потребуется больше ресурсов, создайте образ еще раз и повторно разверните службу.

```python
from azureml.core.webservice import AciWebservice

aciconfig = AciWebservice.deploy_configuration(cpu_cores=1, 
                                               memory_gb=1, 
                                               tags={"data": "MNIST",  "method" : "sklearn"}, 
                                               description='Encrypted Predict MNIST with sklearn + SEAL')
```

### <a name="deploy-to-azure-container-instances"></a>Развертывание в службу "Экземпляры контейнеров Azure"

Предполагаемое время выполнения: **2–5 минут**

Настройте изображение и разверните его. Следующий код выполняет указанные далее действия.

1. Создайте объект среды, содержащий зависимости, которые необходимы для модели, с помощью файла среды (`myenv.yml`).
1. Создайте конфигурацию вывода, которая необходима для развертывания модели в виде веб-службы, используя:
   * файла оценки (`score.py`);
   * объект среды, созданный на предыдущем шаге.
1. Разверните модель в контейнере ACI.
1. Получение конечной точки HTTP веб-службы.

```python
%%time
from azureml.core.webservice import Webservice
from azureml.core.model import InferenceConfig
from azureml.core.environment import Environment
from azureml.core import Workspace
from azureml.core.model import Model

ws = Workspace.from_config()
model = Model(ws, 'sklearn_mnist')

myenv = Environment.get(workspace=ws, name="tutorial-env")
inference_config = InferenceConfig(entry_script="score.py", environment=myenv)

service = Model.deploy(workspace=ws,
                       name='sklearn-encrypted-mnist-svc',
                       models=[model],
                       inference_config=inference_config,
                       deployment_config=aciconfig)

service.wait_for_deployment(show_output=True)
```

Получите конечную точку HTTP веб-службы оценки, которая принимает вызовы клиента REST. Доступ к этой конечной точке можно предоставить всем, кто хочет протестировать веб-службу или интегрировать ее в приложение.

```python
print(service.scoring_uri)
```

## <a name="prepare-test-data"></a>Подготовка тестовых данных

1. Скачайте тестовые данные. В нашем случае они сохраняются в каталог с именем *data*.

    ```python
    import os
    from azureml.core import Dataset
    from azureml.opendatasets import MNIST
    
    data_folder = os.path.join(os.getcwd(), 'data')
    os.makedirs(data_folder, exist_ok=True)
    
    mnist_file_dataset = MNIST.get_file_dataset()
    mnist_file_dataset.download(data_folder, overwrite=True)
    ```

1. Загрузите тестовые данные из каталога *data*.

    ```python
    from utils import load_data
    import os
    import glob
    
    data_folder = os.path.join(os.getcwd(), 'data')
    # note we also shrink the intensity values (X) from 0-255 to 0-1. This helps the neural network converge faster
    X_test = load_data(glob.glob(os.path.join(data_folder,"**/t10k-images-idx3-ubyte.gz"), recursive=True)[0], False) / 255.0
    y_test = load_data(glob.glob(os.path.join(data_folder,"**/t10k-labels-idx1-ubyte.gz"), recursive=True)[0], True).reshape(-1)
    ```

## <a name="make-encrypted-predictions"></a>Создание зашифрованных прогнозов

Используйте тестовый набор данных с моделью для получения прогнозов.

Для создания зашифрованных прогнозов выполните следующие действия.

1. Создайте новый клиент на основе гомоморфного шифрования `EILinearRegressionClient` и открытые ключи.

    ```python
    from encrypted.inference.eiclient import EILinearRegressionClient

    # Create a new Encrypted inference client and a new secret key.
    edp = EILinearRegressionClient(verbose=True)

    public_keys_blob, public_keys_data = edp.get_public_keys()
    ```

1. Отправьте созданные открытые ключи гомоморфного шифрования в хранилище больших двоичных объектов рабочей области по умолчанию. Это позволит использовать ключи совместно с сервером вывода.

    ```python
    import azureml.core
    from azureml.core import Workspace, Datastore
    import os

    ws = Workspace.from_config()

    datastore = ws.get_default_datastore()
    container_name=datastore.container_name

    # Create a local file and write the keys to it
    public_keys = open(public_keys_blob, "wb")
    public_keys.write(public_keys_data)
    public_keys.close()

    # Upload the file to blob store
    datastore.upload_files([public_keys_blob])

    # Delete the local file
    os.remove(public_keys_blob)
    ```

1. Зашифруйте тестовые данные.

    ```python
    #choose any one sample from the test data 
    sample_index = 1

    #encrypt the data
    raw_data = edp.encrypt(X_test[sample_index])

    ```

1. Используйте API `run` пакета SDK для вызова службы и предоставления тестового набора данных модели для получения прогнозов. Необходимо отправить строку подключения в хранилище больших двоичных объектов, в которое были отправлены открытые ключи.

    ```python
    import json
    from azureml.core import Webservice

    service = Webservice(ws, 'sklearn-encrypted-mnist-svc')

    #pass the connection string for blob storage to give the server access to the uploaded public keys 
    conn_str_template = 'DefaultEndpointsProtocol={};AccountName={};AccountKey={};EndpointSuffix=core.windows.net'
    conn_str = conn_str_template.format(datastore.protocol, datastore.account_name, datastore.account_key)

    #build the json 
    data = json.dumps({"data": raw_data, "key_id" : public_keys_blob, "conn_str" : conn_str, "container" : container_name })
    data = bytes(data, encoding='ASCII')

    print ('Making an encrypted inference web service call ')
    eresult = service.run(input_data=data)

    print ('Received encrypted inference results')
    ```

1. Используйте клиент для расшифровки результатов.

    ```python
    import numpy as np

    results = edp.decrypt(eresult)

    print ('Decrypted the results ', results)

    #Apply argmax to identify the prediction result
    prediction = np.argmax(results)

    print ( ' Prediction : ', prediction)
    print ( ' Actual Label : ', y_test[sample_index])
    ```

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите веб-службу, созданную в этом примере.

```python
service.delete()
```

Если вы больше не планируете использовать созданные ресурсы Azure, удалите их. Это избавит вас от оплаты неиспользуемых ресурсов, которые продолжат работать. Дополнительные сведения см. в руководстве по [очистке ресурсов](how-to-manage-workspace.md#clean-up-resources).