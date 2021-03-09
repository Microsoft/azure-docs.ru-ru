---
title: Высокопроизводительная модель, обслуживающая Тритон (Предварительная версия)
titleSuffix: Azure Machine Learning
description: Узнайте, как развернуть модель с помощью сервера Тритонного вывода NVIDIA в Машинное обучение Azure.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: gopalv
author: gvashishtha
ms.date: 02/16/2020
ms.topic: conceptual
ms.reviewer: larryfr
ms.custom: deploy
ms.openlocfilehash: 2966b685e1904102467bf16994ea781556544047
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102519203"
---
# <a name="high-performance-serving-with-triton-inference-server-preview"></a>Высокопроизводительный обслуживание с помощью сервера вывода Тритон (Предварительная версия) 

Узнайте, как использовать [сервер вывода NVIDIA Тритон](https://aka.ms/nvidia-triton-docs) для повышения производительности веб-службы, используемой для определения модели.

Одним из способов развертывания модели для вывода является веб-служба. Например, развертывание в службу Azure Kubernetes или служба "экземпляры контейнеров Azure". По умолчанию Машинное обучение Azure использует однопотоковую веб-платформу *общего назначения* для развертываний веб-служб.

Тритон — это платформа, *оптимизированная для вывода*. Он обеспечивает более эффективное использование GPU и более экономичного вывода. На стороне сервера он пакетно передает входящие запросы и отправляет эти пакеты для вывода. Пакетная обработка лучше использует ресурсы GPU и является ключевой частью производительности Тритон.

> [!IMPORTANT]
> Использование тритон для развертывания из Машинное обучение Azure в настоящее время находится на __этапе предварительной версии__. Функции предварительной версии могут не быть охвачены службой поддержки клиентов. Дополнительные сведения см. в дополнительных [условиях использования для предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)

> [!TIP]
> Фрагменты кода в этом документе предназначены для наглядных целей и могут не показывать законченное решение. Рабочий пример кода см. в разделе [комплексные примеры Тритон в машинное обучение Azure](https://aka.ms/triton-aml-sample).

## <a name="prerequisites"></a>Предварительные требования

* **Подписка Azure**. Если у вас ее нет, используйте [бесплатную или платную версию Машинного обучения Azure](https://aka.ms/AMLFree).
* Знакомство с [тем, как и где развертывать модель](how-to-deploy-and-where.md) с машинное обучение Azure.
* [Машинное обучение Azure пакет SDK для Python](/python/api/overview/azure/ml/) **или** [Azure CLI](/cli/azure/) и [расширение машинного обучения](reference-azure-machine-learning-cli.md).
* Рабочая установка DOCKER для локального тестирования. Сведения об установке и проверке DOCKER см. в разделе [ориентация и настройка](https://docs.docker.com/get-started/) в документации по DOCKER.

## <a name="architectural-overview"></a>Общие сведения об архитектуре

Прежде чем пытаться использовать Тритон для собственной модели, важно понять, как она работает с Машинное обучение Azure и как она сравнивается с развертыванием по умолчанию.

**Развертывание по умолчанию без Тритон**

* Несколько рабочих ролей [гуникорн](https://gunicorn.org/) запущены для параллельной обработки входящих запросов.
* Эти работники управляют предварительной обработкой, вызовом модели и последующей обработкой. 
* Клиенты используют __URI оценки машинного обучения Azure__. Например, `https://myservice.azureml.net/score`.

:::image type="content" source="./media/how-to-deploy-with-triton/normal-deploy.png" alt-text="Схема архитектуры развертывания &quot;Стандартный&quot;, Тритон":::

**Прямое развертывание с помощью Тритон**

* Запросы переходят непосредственно на сервер Тритон.
* Тритон обрабатывает запросы в пакетах, чтобы обеспечить максимальную загрузку GPU.
* Клиент использует __универсальный код ресурса (URI) Тритон__ для выполнения запросов. Например, `https://myservice.azureml.net/v2/models/${MODEL_NAME}/versions/${MODEL_VERSION}/infer`.

:::image type="content" source="./media/how-to-deploy-with-triton/triton-deploy.png" alt-text="Инференцеконфиг развертывание только с Тритон, без по промежуточного слоя Python":::

**Развертывание конфигурации определения с помощью Тритон**

* Несколько рабочих ролей [гуникорн](https://gunicorn.org/) запущены для параллельной обработки входящих запросов.
* Запросы перенаправляются на **сервер Тритон**. 
* Тритон обрабатывает запросы в пакетах, чтобы обеспечить максимальную загрузку GPU.
* Клиент использует __универсальный код ресурса (URI) оценки машинного обучения Azure__ для выполнения запросов. Например, `https://myservice.azureml.net/score`.

:::image type="content" source="./media/how-to-deploy-with-triton/inference-config-deploy.png" alt-text="Развертывание с использованием Тритон и по промежуточного слоя Python":::

Рабочий процесс для использования Тритон в развертывании модели:

1. Обслуживание модели с помощью Тритон напрямую.
1. Убедитесь, что вы можете отправить запросы к модели, развернутой в Тритон.
1. Используемых Создание слоя по промежуточного слоя Python для выполнения до и после обработки на стороне сервера

## <a name="deploying-triton-without-python-pre--and-post-processing"></a>Развертывание Тритон без выполнения предварительной и последующей обработки Python

Сначала выполните следующие действия, чтобы убедиться, что сервер вывода Тритон может обслуживать модель.

### <a name="optional-define-a-model-config-file"></a>Используемых Определение файла конфигурации модели

Файл конфигурации модели сообщает Тритон, сколько входных данных должно рассчитываться и какие измерения будут такими входными данными. Дополнительные сведения о создании файла конфигурации см. в разделе [Конфигурация модели](https://aka.ms/nvidia-triton-docs) в документации по NVIDIA.

> [!TIP]
> Мы используем `--strict-model-config=false` параметр при запуске сервера вывода Тритон, что означает, что вам не нужно указывать `config.pbtxt` файл для моделей ONNX или TensorFlow.
> 
> Дополнительные сведения об этом параметре см. в разделе [Создание конфигурации модели](https://aka.ms/nvidia-triton-docs) в документации по NVIDIA.

### <a name="use-the-correct-directory-structure"></a>Использовать правильную структуру каталогов

При регистрации модели с Машинное обучение Azure можно зарегистрировать отдельные файлы или структуру каталогов. Для использования Тритон регистрация модели должна быть представлена для структуры каталогов, содержащей каталог с именем `triton` . Общая структура этого каталога:

```bash
models
    - triton
        - model_1
            - model_version
                - model_file
                - config_file
        - model_2
            ...
```

> [!IMPORTANT]
> Эта структура каталогов является репозиторием модели Тритон и требуется для работы ваших моделей с Тритон. Дополнительные сведения см. в статье [Тритон Model репозитория](https://aka.ms/nvidia-triton-docs) в документации по NVIDIA.

### <a name="register-your-triton-model"></a>Регистрация модели Тритон

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

```azurecli-interactive
az ml model register -n my_triton_model -p models --model-framework=Multi
```

Дополнительные сведения о `az ml model register` см. в [справочной документации](/cli/azure/ext/azure-cli-ml/ml/model).

# <a name="python"></a>[Python](#tab/python)


```python

from azureml.core.model import Model

model_path = "models"

model = Model.register(
    model_path=model_path,
    model_name="bidaf-9-tutorial",
    tags={"area": "Natural language processing", "type": "Question-answering"},
    description="Question answering from ONNX model zoo",
    workspace=ws,
    model_framework=Model.Framework.MULTI,  # This line tells us you are registering a Triton model
)

```
Дополнительные сведения см. в документации по [классу Model](/python/api/azureml-core/azureml.core.model.model).

---

### <a name="deploy-your-model"></a>Развертывание модели

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

Если вы используете кластер службы Kubernetes Azure с поддержкой GPU с именем "AKS-GPU", созданный с помощью Машинное обучение Azure, для развертывания модели можно использовать следующую команду.

```azurecli
az ml model deploy -n triton-webservice -m triton_model:1 --dc deploymentconfig.json --compute-target aks-gpu
```

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core.webservice import AksWebservice
from azureml.core.model import InferenceConfig
from random import randint

service_name = "triton-webservice"

config = AksWebservice.deploy_configuration(
    compute_target_name="aks-gpu",
    gpu_cores=1,
    cpu_cores=1,
    memory_gb=4,
    auth_enabled=True,
)

service = Model.deploy(
    workspace=ws,
    name=service_name,
    models=[model],
    deployment_config=config,
    overwrite=True,
)
```
---

[Дополнительные сведения о развертывании моделей см. в этой документации](how-to-deploy-and-where.md).

### <a name="call-into-your-deployed-model"></a>Вызов в развернутой модели

Сначала получите URI оценки и токены носителя.

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)


```azurecli
az ml service show --name=triton-webservice
```
# <a name="python"></a>[Python](#tab/python)

```python
import requests

print(service.scoring_uri)
print(service.get_keys())

```

---

Затем убедитесь, что служба работает, выполнив следующие действия. 

```{bash}
!curl -v $scoring_uri/v2/health/ready -H 'Authorization: Bearer '"$service_key"''
```

Эта команда возвращает сведения, аналогичные приведенным ниже. Примечание `200 OK` . это состояние означает, что веб-сервер работает.

```{bash}
*   Trying 127.0.0.1:8000...
* Connected to localhost (127.0.0.1) port 8000 (#0)
> GET /v2/health/ready HTTP/1.1
> Host: localhost:8000
> User-Agent: curl/7.71.1
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
HTTP/1.1 200 OK
```

После выполнения проверки работоспособности можно создать клиент для отправки данных в Тритон для вывода. Дополнительные сведения о создании клиента см. в [примерах клиента](https://aka.ms/nvidia-client-examples) в документации по NVIDIA. Также доступны [примеры Python на Тритон GitHub](https://aka.ms/nvidia-triton-docs).

Если вы не хотите добавлять предварительную и последующую обработку Python в развернутую веб-службу, это может быть сделано. Если вы хотите добавить эту логику до и после обработки, прочтите статью.

## <a name="optional-re-deploy-with-a-python-entry-script-for-pre--and-post-processing"></a>Используемых Повторное развертывание с помощью скрипта записи Python для предварительной и последующей обработки

Убедившись, что Тритон способен обслуживать модель, можно добавить предварительную и последующей обработку кода, определив _сценарий записи_. Этот файл называется `score.py` . Дополнительные сведения о сценариях входа см. в разделе [Определение записи](how-to-deploy-and-where.md#define-an-entry-script).

Два основных этапа — инициализация HTTP-клиента Тритон в `init()` методе и вызов этого клиента в `run()` функции.

### <a name="initialize-the-triton-client"></a>Инициализация клиента Тритон

Включите в файл код, как в приведенном ниже примере `score.py` . Тритон в Машинное обучение Azure требуется устранить на localhost, порт 8000. В этом случае localhost находится внутри образа DOCKER для этого развертывания, а не для порта на локальном компьютере:

> [!TIP]
> `tritonhttpclient`Пакет PIP включен в проверенную `AzureML-Triton` среду, поэтому нет необходимости указывать его в качестве зависимости PIP.

```python
import tritonhttpclient

def init():
    global triton_client
    triton_client = tritonhttpclient.InferenceServerClient(url="localhost:8000")
```

### <a name="modify-your-scoring-script-to-call-into-triton"></a>Изменение скрипта оценки для вызова Тритон

В следующем примере показано, как динамически запрашивать метаданные для модели.

> [!TIP]
> Вы можете динамически запрашивать метаданные моделей, которые были загружены с помощью Тритон, используя `.get_model_metadata` метод клиента Тритон. Пример использования см. в примере [записной книжке](https://aka.ms/triton-aml-sample) .

```python
input = tritonhttpclient.InferInput(input_name, data.shape, datatype)
input.set_data_from_numpy(data, binary_data=binary_data)

output = tritonhttpclient.InferRequestedOutput(
         output_name, binary_data=binary_data, class_count=class_count)

# Run inference
res = triton_client.infer(model_name,
                          [input]
                          request_id='0',
                          outputs=[output])

```

<a id="redeploy"></a>

### <a name="redeploy-with-an-inference-configuration"></a>Повторное развертывание с конфигурацией вывода

Конфигурация вывода позволяет использовать скрипт записи, а также процесс развертывания Машинное обучение Azure с помощью пакета SDK Python или Azure CLI.

> [!IMPORTANT]
> Необходимо указать `AzureML-Triton` [проверенную среду](./resource-curated-environments.md).
>
> Пример кода Python копирует `AzureML-Triton` в другую среду с именем `My-Triton` . Код Azure CLI также использует эту среду. Дополнительные сведения о клонировании среды см. в справочнике по [среде. Clone ()](/python/api/azureml-core/azureml.core.environment.environment#clone-new-name-) .

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

> [!TIP]
> Дополнительные сведения о создании конфигурации вывода см. в разделе [Схема конфигурации вывода](./reference-azure-machine-learning-cli.md#inference-configuration-schema).

```azurecli
az ml model deploy -n triton-densenet-onnx \
-m densenet_onnx:1 \
--ic inference-config.json \
-e My-Triton --dc deploymentconfig.json \
--overwrite --compute-target=aks-gpu
```

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core.webservice import LocalWebservice
from azureml.core import Environment
from azureml.core.model import InferenceConfig


local_service_name = "triton-bidaf-onnx"
env = Environment.get(ws, "AzureML-Triton").clone("My-Triton")

for pip_package in ["nltk"]:
    env.python.conda_dependencies.add_pip_package(pip_package)

inference_config = InferenceConfig(
    entry_script="score_bidaf.py",  # This entry script is where we dispatch a call to the Triton server
    source_directory=os.path.join("..", "scripts"),
    environment=env
)

local_config = LocalWebservice.deploy_configuration(
    port=6789
)

local_service = Model.deploy(
    workspace=ws,
    name=local_service_name,
    models=[model],
    inference_config=inference_config,
    deployment_config=local_config,
    overwrite=True)

local_service.wait_for_deployment(show_output = True)
print(local_service.state)
# Print the URI you can use to call the local deployment
print(local_service.scoring_uri)
```

---

После завершения развертывания отображается URI оценки. Для этого локального развертывания это будет `http://localhost:6789/score` . При развертывании в облаке можно использовать команду [AZ ML Service показывать](/cli/azure/ext/azure-cli-ml/ml/service#ext_azure_cli_ml_az_ml_service_show) CLI для получения URI оценки.

Сведения о создании клиента, отправляющего запросы вывода в URI оценки, см. в разделе [Использование модели, развернутой в качестве веб-службы](how-to-consume-web-service.md).

### <a name="setting-the-number-of-workers"></a>Задание количества рабочих ролей

Чтобы задать число рабочих ролей в развертывании, задайте переменную среды `WORKER_COUNT` . При наличии объекта [среды](/python/api/azureml-core/azureml.core.environment.environment) с именем `env` можно выполнить следующие действия.

```{py}
env.environment_variables["WORKER_COUNT"] = "1"
```

Это укажет Azure ML, что будет вычислено указанное количество рабочих ролей.


## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы планируете продолжить использование рабочей области Машинное обучение Azure, но хотите избавиться от развернутой службы, используйте один из следующих вариантов:


# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

```azurecli
az ml service delete -n triton-densenet-onnx
```
# <a name="python"></a>[Python](#tab/python)

```python
local_service.delete()
```


---

## <a name="next-steps"></a>Дальнейшие действия

* [Ознакомьтесь с комплексными примерами Тритон в Машинное обучение Azure](https://aka.ms/aml-triton-sample)
* [Примеры клиента Тритон](https://aka.ms/nvidia-client-examples)
* Ознакомьтесь с [документацией по серверу вывода Тритон](https://aka.ms/nvidia-triton-docs)
* [Устранение неполадок при развертывании](how-to-troubleshoot-deployment.md)
* [развертывание в Службе Azure Kubernetes](how-to-deploy-azure-kubernetes-service.md).
* [Обновление веб-службы](how-to-deploy-update-web-service.md)
* [Сбор данных для моделей в рабочей среде](how-to-enable-data-collection.md)