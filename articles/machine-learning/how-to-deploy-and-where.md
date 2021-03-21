---
title: Развертывание моделей машинного обучения
titleSuffix: Azure Machine Learning
description: Узнайте, как и где развертываются модели машинного обучения. Выполните развертывание в службе "экземпляры контейнеров Azure", службу Kubernetes Azure, Azure IoT Edge и FPGA.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: gopalv
author: gvashishtha
ms.reviewer: larryfr
ms.date: 01/13/2021
ms.topic: conceptual
ms.custom: how-to, devx-track-python, deploy, devx-track-azurecli
adobe-target: true
ms.openlocfilehash: ed397e9f8db721a6baa641fc958af0dda570ce57
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103561946"
---
# <a name="deploy-machine-learning-models-to-azure"></a>Развертывание моделей машинного обучения в Azure

Сведения о развертывании машинного обучения или модели глубокого обучения в качестве веб-службы в облаке Azure. Также можно выполнить развертывание на Azure IoT Edge устройствах.

Последовательность действий всегда одна и та же, независимо от того, где развертывается модель:

1. Зарегистрируйте модель (необязательно, см. ниже).
1. Подготовка конфигурации вывода (если [не используется развертывание без кода](./how-to-deploy-no-code-deployment.md)).
1. Подготовка скрипта записи (если [не используется развертывание без кода](./how-to-deploy-no-code-deployment.md)).
1. Выберите целевой объект вычислений.
1. развертывание модели в целевом объекте вычислений;
1. Протестируйте полученную веб-службу.

Дополнительные сведения об основных понятиях, связанных с рабочим процессом развертывания машинного обучения, см. в статье [Управление моделями с помощью машинное обучение Azure](concept-model-management-and-deployment.md).

## <a name="prerequisites"></a>Предварительные требования

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

- Рабочая область машинного обучения Azure. Дополнительные сведения см. в статье [создание машинное обучение Azure рабочей области](how-to-manage-workspace.md).
- Модель. Если у вас нет обученной модели, можно использовать модель и файлы зависимостей, предоставленные в [этом руководстве](https://aka.ms/azml-deploy-cloud).
- [Расширение интерфейса командной строки Azure (CLI) для службы машинное обучение](reference-azure-machine-learning-cli.md).

# <a name="python"></a>[Python](#tab/python)

- Рабочая область машинного обучения Azure. Дополнительные сведения см. в статье [создание машинное обучение Azure рабочей области](how-to-manage-workspace.md).
- Модель. Если у вас нет обученной модели, можно использовать модель и файлы зависимостей, предоставленные в [этом руководстве](https://aka.ms/azml-deploy-cloud).
- [Пакет средств разработки машинное обучение Azure Software Development Kit (SDK) для Python](/python/api/overview/azure/ml/intro).

---

## <a name="connect-to-your-workspace"></a>Подключение к рабочей области

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

Следуйте указаниям в документации по Azure CLI для [настройки контекста подписки](/cli/azure/manage-azure-subscriptions-azure-cli#change-the-active-subscription).

Затем выполните следующие действия.

```azurecli-interactive
az ml workspace list --resource-group=<my resource group>
```

для просмотра рабочих областей, к которым у вас есть доступ.

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core import Workspace
ws = Workspace.from_config(path=".file-path/ws_config.json")
```

Дополнительные сведения об использовании пакета SDK для подключения к рабочей области см. в документации по [машинное обучение Azure SDK для Python](/python/api/overview/azure/ml/intro#workspace) .


---


## <a name="register-your-model-optional"></a><a id="registermodel"></a> Регистрация модели (необязательно)

Зарегистрированная модель — это логический контейнер для одного или нескольких файлов, составляющих эту модель. Например, если имеется модель, которая хранится в нескольких файлах, можно зарегистрировать их как единую модель в рабочей области. После регистрации файлов можно скачать или развернуть зарегистрированную модель и получить все зарегистрированные файлы.

> [!TIP] 
> Регистрация модели для отслеживания версий рекомендуется, но не является обязательной. Если вы предпочитаете не регистрировать модель, необходимо указать исходный каталог в [инференцеконфиг](/python/api/azureml-core/azureml.core.model.inferenceconfig) или [inferenceconfig.js](./reference-azure-machine-learning-cli.md#inference-configuration-schema) и убедиться, что модель находится в этом исходном каталоге.

> [!TIP]
> При регистрации модели вы предоставляете путь к облачному расположению (из обучающего запуска) или к локальному каталогу. Этот путь предназначен только для поиска файлов для отправки в ходе процесса регистрации. Он не должен соответствовать пути, используемому в скрипте записи. Дополнительные сведения см. [в разделе Поиск файлов модели в скрипте записи](./how-to-deploy-advanced-entry-script.md#load-registered-models).

> [!IMPORTANT]
> При использовании параметра Фильтровать по `Tags` на странице модели машинное обучение Azure Studio вместо использования `TagName : TagValue` клиентов следует использовать `TagName=TagValue` (без пробела).

В следующих примерах показано, как зарегистрировать модель.

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

### <a name="register-a-model-from-an-azure-ml-training-run"></a>Регистрация модели из запуска обучения Azure ML

```azurecli-interactive
az ml model register -n sklearn_mnist  --asset-path outputs/sklearn_mnist_model.pkl  --experiment-name myexperiment --run-id myrunid --tag area=mnist
```

[!INCLUDE [install extension](../../includes/machine-learning-service-install-extension.md)]

`--asset-path`Параметр ссылается на расположение в облаке модели. В этом примере используется путь к одному файлу. Чтобы включить несколько файлов в регистрацию модели, задайте `--asset-path` путь к папке, содержащей файлы.

### <a name="register-a-model-from-a-local-file"></a>Регистрация модели из локального файла

```azurecli-interactive
az ml model register -n onnx_mnist -p mnist/model.onnx
```

Чтобы включить несколько файлов в регистрацию модели, задайте `-p` путь к папке, содержащей файлы.

Дополнительные сведения о `az ml model register` см. в [справочной документации](/cli/azure/ext/azure-cli-ml/ml/model).

# <a name="python"></a>[Python](#tab/python)

### <a name="register-a-model-from-an-azure-ml-training-run"></a>Регистрация модели из запуска обучения Azure ML

  При использовании пакета SDK для обучения модели можно получить либо объект [запуска](/python/api/azureml-core/azureml.core.run.run) , либо объект [аутомлрун](/python/api/azureml-train-automl-client/azureml.train.automl.run.automlrun) в зависимости от того, как была обучена модель. Каждый объект можно использовать для регистрации модели, созданной с помощью запуска эксперимента.

  + Зарегистрировать модель из `azureml.core.Run` объекта:
 
    ```python
    model = run.register_model(model_name='sklearn_mnist',
                               tags={'area': 'mnist'},
                               model_path='outputs/sklearn_mnist_model.pkl')
    print(model.name, model.id, model.version, sep='\t')
    ```

    `model_path`Параметр ссылается на расположение в облаке модели. В этом примере используется путь к одному файлу. Чтобы включить несколько файлов в регистрацию модели, задайте `model_path` путь к папке, содержащей файлы. Дополнительные сведения см. в документации по [Run.register_model](/python/api/azureml-core/azureml.core.run.run#register-model-model-name--model-path-none--tags-none--properties-none--model-framework-none--model-framework-version-none--description-none--datasets-none--sample-input-dataset-none--sample-output-dataset-none--resource-configuration-none----kwargs-) .

  + Зарегистрировать модель из `azureml.train.automl.run.AutoMLRun` объекта:

    ```python
        description = 'My AutoML Model'
        model = run.register_model(description = description,
                                   tags={'area': 'mnist'})

        print(run.model_id)
    ```

    В этом примере `metric` `iteration` Параметры и не указаны, поэтому будет зарегистрирована итерация с лучшей основной метрикой. `model_id`Значение, возвращаемое при выполнении, используется вместо имени модели.

    Дополнительные сведения см. в документации по [AutoMLRun.register_model](/python/api/azureml-train-automl-client/azureml.train.automl.run.automlrun#register-model-model-name-none--description-none--tags-none--iteration-none--metric-none-) .

    Чтобы развернуть зарегистрированную модель из `AutoMLRun` , мы рекомендуем сделать это с помощью [кнопки развертывания одним щелчком в студии машинного обучения Azure](how-to-use-automated-ml-for-ml-models.md#deploy-your-model). 
### <a name="register-a-model-from-a-local-file"></a>Регистрация модели из локального файла

Вы можете зарегистрировать модель, указав локальный путь к модели. Можно указать путь либо к папке, либо к одному файлу. С помощью этого метода можно регистрировать модели, обученные Машинное обучение Azure, а затем загружать их. Этот метод также можно использовать для регистрации моделей, обученных за пределами Машинное обучение Azure.

[!INCLUDE [trusted models](../../includes/machine-learning-service-trusted-model.md)]

+ **Использование пакета SDK и ONNX**

    ```python
    import os
    import urllib.request
    from azureml.core.model import Model
    # Download model
    onnx_model_url = "https://www.cntk.ai/OnnxModels/mnist/opset_7/mnist.tar.gz"
    urllib.request.urlretrieve(onnx_model_url, filename="mnist.tar.gz")
    os.system('tar xvzf mnist.tar.gz')
    # Register model
    model = Model.register(workspace = ws,
                            model_path ="mnist/model.onnx",
                            model_name = "onnx_mnist",
                            tags = {"onnx": "demo"},
                            description = "MNIST image classification CNN from ONNX Model Zoo",)
    ```

  Чтобы включить несколько файлов в регистрацию модели, задайте `model_path` путь к папке, содержащей файлы.

Дополнительные сведения см. в документации по [классу Model](/python/api/azureml-core/azureml.core.model.model).

Дополнительные сведения о работе с моделями, обученными вне Машинное обучение Azure, см. [в разделе Развертывание существующей модели](how-to-deploy-existing-model.md).

---

## <a name="define-an-entry-script"></a>Определение скрипта записи

[!INCLUDE [write entry script](../../includes/machine-learning-entry-script.md)]


## <a name="define-an-inference-configuration"></a>Определение конфигурации вывода


Конфигурация вывода описывает, как настроить веб-службу, содержащую модель. Он используется позже при развертывании модели.

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

Минимальная конфигурация вывода может быть записана следующим образом:

```json
{
    "entryScript": "score.py",
    "sourceDirectory": "./working_dir"
}
```

Это означает, что при развертывании машинного обучения будет использоваться файл `score.py` в `./working_dir` каталоге для обработки входящих запросов.

Более подробное обсуждение конфигураций вывода см. в [этой статье](./reference-azure-machine-learning-cli.md#inference-configuration-schema) . 

# <a name="python"></a>[Python](#tab/python)

В следующем примере демонстрируется:

1. Загрузка [проверенной среды](resource-curated-environments.md) из рабочей области
1. Клонирование среды
1. Указание `scikit-learn` в качестве зависимости.
1. Использование среды для создания Инференцеконфиг

```python
from azureml.core.environment import Environment
from azureml.core.model import InferenceConfig


env = Environment.get(workspace, "AzureML-Minimal").clone(env_name)

for pip_package in ["scikit-learn"]:
    env.python.conda_dependencies.add_pip_package(pip_package)

inference_config = InferenceConfig(entry_script='path-to-score.py',
                                    environment=env)
```

Дополнительные сведения о средах см. в статье [создание сред для обучения и развертывания и управление ими](how-to-use-environments.md).

Дополнительные сведения о конфигурации вывода см. в документации по классу [инференцеконфиг](/python/api/azureml-core/azureml.core.model.inferenceconfig) .

---

> [!TIP] 
> Сведения об использовании пользовательского образа DOCKER с конфигурацией вывода см. в статье [развертывание модели с помощью пользовательского образа DOCKER](how-to-deploy-custom-docker-image.md).

## <a name="choose-a-compute-target"></a>Выбор целевого объекта вычислений

[!INCLUDE [aml-compute-target-deploy](../../includes/aml-compute-target-deploy.md)]

## <a name="define-a-deployment-configuration"></a>Определение конфигурации развертывания

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

Параметры, доступные для конфигурации развертывания, зависят от выбранного целевого объекта вычислений.

[!INCLUDE [aml-local-deploy-config](../../includes/machine-learning-service-local-deploy-config.md)]

Дополнительные сведения см. в [этом справочнике](./reference-azure-machine-learning-cli.md#deployment-configuration-schema).

# <a name="python"></a>[Python](#tab/python)

Перед развертыванием модели необходимо определить конфигурацию развертывания. *Конфигурация развертывания зависит от целевого объекта вычислений, в котором будет размещена веб-служба.* Например, при локальном развертывании модели необходимо указать порт, на котором служба принимает запросы. Конфигурация развертывания не является частью скрипта записи. Он используется для определения характеристик целевого объекта вычислений, в котором будет размещаться скрипт модели и входа.

Вам также может потребоваться создать вычислительный ресурс, если, например, у вас еще нет экземпляра Azure Kubernetes Service (AKS), связанного с вашей рабочей областью.

В следующей таблице приведен пример создания конфигурации развертывания для каждого целевого объекта вычислений.

| Целевой объект вычисления | Пример конфигурации развертывания |
| ----- | ----- |
| Локальная | `deployment_config = LocalWebservice.deploy_configuration(port=8890)` |
| Экземпляры контейнеров Azure | `deployment_config = AciWebservice.deploy_configuration(cpu_cores = 1, memory_gb = 1)` |
| Служба Azure Kubernetes | `deployment_config = AksWebservice.deploy_configuration(cpu_cores = 1, memory_gb = 1)` |

Классы для локальных объектов, экземпляров контейнеров Azure и веб-служб AKS можно импортировать из `azureml.core.webservice` :

```python
from azureml.core.webservice import AciWebservice, AksWebservice, LocalWebservice
```

---

## <a name="deploy-your-machine-learning-model"></a>Развертывание модели машинного обучения

Теперь вы готовы к развертыванию модели. 

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

### <a name="using-a-registered-model"></a>Использование зарегистрированной модели

Если вы зарегистрировали модель в рабочей области Машинное обучение Azure, замените "MyModel: 1" именем модели и ее номером версии.

```azurecli-interactive
az ml model deploy -m mymodel:1 --ic inferenceconfig.json --dc deploymentconfig.json
```

### <a name="using-a-local-model"></a>Использование локальной модели

Если вы предпочитаете не регистрировать модель, можно передать параметр "sourceDirectory" в inferenceconfig.js, чтобы указать локальный каталог, из которого будет обслуживаться модель.

```azurecli-interactive
az ml model deploy --ic inferenceconfig.json --dc deploymentconfig.json
```

# <a name="python"></a>[Python](#tab/python)

В приведенном ниже примере демонстрируется локальное развертывание. Синтаксис будет зависеть от целевого объекта вычислений, выбранного на предыдущем шаге.

```python
from azureml.core.webservice import LocalWebservice, Webservice

deployment_config = LocalWebservice.deploy_configuration(port=8890)
service = Model.deploy(ws, "myservice", [model], inference_config, deployment_config)
service.wait_for_deployment(show_output = True)
print(service.state)
```

Дополнительные сведения см. в документации по [локалвебсервице](/python/api/azureml-core/azureml.core.webservice.local.localwebservice), [model. deploy ()](/python/api/azureml-core/azureml.core.model.model#deploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-)и [WebService](/python/api/azureml-core/azureml.core.webservice.webservice).

---

### <a name="understanding-service-state"></a>Основные сведения о состоянии службы

Во время развертывания модели изменение состояния службы может отображаться, пока оно полностью развернуто.

В следующей таблице описаны различные состояния службы.

| Состояние WebService | Описание | Конечное состояние?
| ----- | ----- | ----- |
| Переход | Служба находится в процессе развертывания. | Нет |
| Unhealthy | Служба была развернута, но сейчас недоступна.  | Нет |
| Непланируемый | В настоящее время служба не может быть развернута из-за нехватки ресурсов. | Нет |
| Сбой | Не удалось выполнить развертывание службы из-за ошибки или сбоя. | Да |
| Работоспособно | Служба работоспособна, и доступна конечная точка. | Да |

> [!TIP]
> При развертывании образы DOCKER для целевых объектов вычислений создаются и загружаются из реестра контейнеров Azure (запись контроля доступа). По умолчанию Машинное обучение Azure создает запись контроля доступа, которая использует уровень служб " *базовый* ". Изменение записи контроля доступа для рабочей области на уровень "Стандартный" или "Премиум" может сократить время, затрачиваемое на сборку и развертывание образов в целевых объектах вычислений. Дополнительные сведения см в статье [Уровни службы Реестра контейнеров Azure](../container-registry/container-registry-skus.md).

> [!NOTE]
> При развертывании модели в службе Kubernetes Azure (AKS) мы рекомендуем включить [Azure Monitor](../azure-monitor/containers/container-insights-enable-existing-clusters.md) для этого кластера. Это поможет понять общую работоспособность кластера и использование ресурсов. Вы также можете найти следующие полезные ресурсы:
>
> * [Проверка событий Работоспособность ресурсов, влияющих на кластер AKS](../aks/aks-resource-health.md)
> * [Диагностика службы Kubernetes Azure](../aks/concepts-diagnostics.md)
>
> Если вы пытаетесь развернуть модель в неработоспособном или перегруженном кластере, то ожидается проблема. Если вам нужна помощь в устранении неполадок с кластером AKS, обратитесь в службу поддержки AKS.

### <a name="batch-inference"></a><a id="azuremlcompute"></a> Вывод пакета
Машинное обучение Azure целевые объекты вычислений создаются и управляются с помощью Машинное обучение Azure. Их можно использовать для прогнозирования пакетной службы из Машинное обучение Azure конвейеров.

Пошаговое руководство по выводу пакетов с помощью Машинное обучение Azure COMPUTE см. [в разделе Выполнение пакетных прогнозов](tutorial-pipeline-batch-scoring-classification.md).

### <a name="iot-edge-inference"></a><a id="iotedge"></a> Вывод IoT Edge
Поддержка развертывания на границе доступна в предварительной версии. Дополнительные сведения см. в разделе [развертывание машинное обучение Azure как модуля IOT Edge](../iot-edge/tutorial-deploy-machine-learning.md).

## <a name="delete-resources"></a>Удаление ресурсов

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

Чтобы удалить развернутую WebService, используйте `az ml service <name of webservice>` .

Чтобы удалить зарегистрированную модель из рабочей области, используйте `az ml model delete <model id>`

Узнайте больше об [удалении WebService](/cli/azure/ext/azure-cli-ml/ml/service#ext-azure-cli-ml-az-ml-service-delete) и [удалении модели](/cli/azure/ext/azure-cli-ml/ml/model#ext-azure-cli-ml-az-ml-model-delete).

# <a name="python"></a>[Python](#tab/python)

Для удаления развернутой веб-службы используйте `service.delete()`.
Чтобы удалить зарегистрированную модель, используйте `model.delete()`.

Дополнительные сведения см. в документации по [WebService. Delete ()](/python/api/azureml-core/azureml.core.webservice%28class%29#delete--) и [model. Delete ()](/python/api/azureml-core/azureml.core.model.model#delete--).

---

## <a name="next-steps"></a>Следующие шаги

* [Устранение неполадок при развертывании](how-to-troubleshoot-deployment.md)
* [развертывание в Службе Azure Kubernetes](how-to-deploy-azure-kubernetes-service.md).
* [Создание клиентских приложений для использования веб-служб](how-to-consume-web-service.md)
* [Обновление веб-службы](how-to-deploy-update-web-service.md)
* [Развертывание модели с помощью пользовательского образа DOCKER](how-to-deploy-custom-docker-image.md)
* [Развертывание с помощью автоматического запуска машинного обучения в Машинное обучение Azure Studio](how-to-use-automated-ml-for-ml-models.md#deploy-your-model)
* [Использование TLS для защиты веб-службы с помощью Машинного обучения Azure](how-to-secure-web-service.md).
* [Мониторинг моделей Машинное обучение Azure с помощью Application Insights](how-to-enable-app-insights.md)
* [Сбор данных для моделей в рабочей среде](how-to-enable-data-collection.md)
* [Создание предупреждений и триггеров событий для развертываний моделей](how-to-use-event-grid.md)