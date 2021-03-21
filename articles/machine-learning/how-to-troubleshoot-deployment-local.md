---
title: Устранение неполадок при развертывании локальной модели
titleSuffix: Azure Machine Learning
description: Используйте развертывание локальной модели в качестве первого шага при устранении ошибок развертывания модели.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: gvashishtha
ms.author: gopalv
ms.reviewer: luquinta
ms.date: 11/25/2020
ms.topic: troubleshooting
ms.custom: devx-track-python, deploy, contperf-fy21q2
ms.openlocfilehash: 69ac47296cb4624de6cdf05ddb3e72973751f631
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102519628"
---
# <a name="troubleshooting-with-a-local-model-deployment"></a>Устранение неполадок при развертывании локальной модели

Используйте развертывание локальной модели в качестве первого шага при устранении неполадок развертывания в службе "экземпляры контейнеров Azure" (ACI) или Azure Kubernetes Service (AKS).  Использование локальной веб-службы упрощает обнаружение и устранение распространенных ошибок развертывания веб-службы DOCKER Машинное обучение Azure.

## <a name="prerequisites"></a>Предварительные требования

* **Подписка Azure**. Попробуйте [бесплатную или платную версию Машинного обучения Azure](https://aka.ms/AMLFree).
* Параметр A (**рекомендуется**) — локальное отладка машинное обучение Azure вычислительного экземпляра
   * Рабочая область машинного обучения Azure с выполняющимся [вычислительным экземпляром](how-to-deploy-local-container-notebook-vm.md)
* Вариант б. Отладка локально на основе вычислений
   * [Пакет SDK для Машинного обучения Azure](/python/api/overview/azure/ml/install).
   * [Интерфейс командной строки Azure](/cli/azure/install-azure-cli).
   * [Расширение CLI для Машинного обучения Azure](reference-azure-machine-learning-cli.md).
   * Запросите рабочую установку DOCKER в локальной системе. 
   * Чтобы проверить установку Docker, выполните команду `docker run hello-world` в терминале или командной строке. Сведения об установке Docker или устранении ошибок Docker см. в [документации Docker](https://docs.docker.com/).

## <a name="debug-locally"></a>Отладка в локальной среде

Пример [локального развертывания](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/deployment/deploy-to-local/register-model-deploy-local.ipynb) можно найти в репозитории  [мачинелеарнингнотебукс](https://github.com/Azure/MachineLearningNotebooks) , чтобы просмотреть пример готового к запуску примера.

> [!WARNING]
> Развертывания локальных веб-служб не поддерживаются для рабочих сред.

Для развертывания в локальной среде измените код, чтобы использовать `LocalWebservice.deploy_configuration()` для создания конфигурации развертывания. Затем используйте `Model.deploy()` для развертывания службы. В следующем примере модель (содержащаяся в переменной моделей) развертывается как локальная веб-служба:

```python
from azureml.core.environment import Environment
from azureml.core.model import InferenceConfig, Model
from azureml.core.webservice import LocalWebservice


# Create inference configuration based on the environment definition and the entry script
myenv = Environment.from_conda_specification(name="env", file_path="myenv.yml")
inference_config = InferenceConfig(entry_script="score.py", environment=myenv)
# Create a local deployment, using port 8890 for the web service endpoint
deployment_config = LocalWebservice.deploy_configuration(port=8890)
# Deploy the service
service = Model.deploy(
    ws, "mymodel", [model], inference_config, deployment_config)
# Wait for the deployment to complete
service.wait_for_deployment(True)
# Display the port that the web service is available on
print(service.port)
```

Если вы определяете собственную спецификацию conda YAML, то List azureml-Defaults версии >= 1.0.45 в качестве зависимости PIP. Этот пакет необходим для размещения модели в качестве веб-службы.

На этом этапе вы можете использовать службу обычным образом. В следующем примере кода демонстрируется отправка данных в службу:

```python
import json

test_sample = json.dumps({'data': [
    [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
    [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]
]})

test_sample = bytes(test_sample, encoding='utf8')

prediction = service.run(input_data=test_sample)
print(prediction)
```

Дополнительные сведения о настройке среды Python см. на [этой странице](how-to-use-environments.md). 

### <a name="update-the-service"></a>Обновление службы

Во время локального тестирования, возможно, нужно будет обновить файл `score.py`, чтобы добавить ведение журнала или попытаться решить любые обнаруженные проблемы. Чтобы перезагрузить изменения в файл `score.py`, используйте `reload()`. Например, следующий код перезагружает сценарий для службы, а затем отправляет в нее данные. Данные оцениваются с помощью обновленного файла `score.py`:

> [!IMPORTANT]
> Метод `reload` доступен лишь для локальных развертываний. Сведения об обновлении развертывания на другом целевом объекте вычислений см. [в разделе Обновление веб-службы](how-to-deploy-update-web-service.md).

```python
service.reload()
print(service.run(input_data=test_sample))
```

> [!NOTE]
> Сценарий перезагружается из расположения, указанного объектом `InferenceConfig`, используемым службой.

Чтобы изменить модель, зависимости Conda или конфигурацию развертывания, используйте метод [update()](/python/api/azureml-core/azureml.core.webservice%28class%29#update--args-). В следующем примере обновляется модель, используемая службой:

```python
service.update([different_model], inference_config, deployment_config)
```

### <a name="delete-the-service"></a>Удаление службы

Чтобы удалить службу, воспользуйтесь методом [delete()](/python/api/azureml-core/azureml.core.webservice%28class%29#delete--).

### <a name="inspect-the-docker-log"></a><a id="dockerlog"></a> Проверка журнала Docker

Можно распечатать подробные сообщения журнала ядра Docker из объекта службы. Вы можете просмотреть журнал для ACI, AKS и локальных развертываний. В следующем примере показано, как вывести данные журналов.

```python
# if you already have the service object handy
print(service.get_logs())

# if you only know the name of the service (note there might be multiple services with the same name but different version number)
print(ws.webservices['mysvc'].get_logs())
```

Если строка `Booting worker with pid: <pid>` в журналах встречается несколько раз, это означает, что недостаточно памяти для запуска рабочей роли.
Эту ошибку можно устранить, увеличив значение `memory_gb` в `deployment_config`

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о развертывании см. в статьях, представленных ниже.

* [Устранение неполадок при удаленных развертываниях](how-to-troubleshoot-deployment.md)
* [Развертывание моделей с помощью Службы машинного обучения Azure](how-to-deploy-and-where.md)
* [Руководство. Обучение и развертывание моделей](tutorial-train-models-with-aml.md)
* [Локальное выполнение экспериментов и их отладка](./how-to-debug-visual-studio-code.md)