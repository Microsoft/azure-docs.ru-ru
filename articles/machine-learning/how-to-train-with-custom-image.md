---
title: Обучение модели с помощью пользовательского образа DOCKER
titleSuffix: Azure Machine Learning
description: Узнайте, как использовать собственные образы DOCKER или проверенные из Майкрософт для обучения моделей в Машинное обучение Azure.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: sagopal
author: saachigopal
ms.date: 10/20/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: 03400173dd35660d6e7a98500b831a7c6aa4ebd7
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102521176"
---
# <a name="train-a-model-by-using-a-custom-docker-image"></a>Обучение модели с помощью пользовательского образа DOCKER

Из этой статьи вы узнаете, как использовать пользовательский образ DOCKER при обучении моделей с Машинное обучение Azure. Примеры сценариев в этой статье используются для классификации образов Pet путем создания свертки нейронной сети. 

Машинное обучение Azure предоставляет базовый образ DOCKER по умолчанию. Можно также использовать Машинное обучение Azure среды, чтобы указать другой базовый образ, например один из обслуживаемых [машинное обучение Azure базовых образов](https://github.com/Azure/AzureML-Containers) или [пользовательский образ](how-to-deploy-custom-docker-image.md#create-a-custom-base-image). Пользовательские базовые образы позволяют тесно управлять зависимостями и поддерживать более строгий контроль версий компонентов при выполнении заданий обучения.

## <a name="prerequisites"></a>Предварительные условия

Запустите код в любой из этих сред:

* Машинное обучение Azure вычислительного экземпляра (Загрузка или установка не требуется):
  * Выполните инструкции из руководства по [настройке среды и рабочей области](tutorial-1st-experiment-sdk-setup.md) , чтобы создать выделенный сервер записной книжки, предварительно загруженный с помощью пакета SDK и примера репозитория.
  * В [репозитории примеров](https://github.com/Azure/azureml-examples)машинное обучение Azure найдите готовую записную книжку, перейдя к каталогу **Notebooks**  >  **фастаи** Train  >  **-pets-resnet34. ipynb** . 
* Собственный Jupyter Notebook сервер:
  * Создайте [файл конфигурации рабочей области](how-to-configure-environment.md#workspace).
  * Установите [пакет SDK для Машинного обучения Azure](/python/api/overview/azure/ml/install). 
  * Создайте [Реестр контейнеров Azure](../container-registry/index.yml) или другой реестр DOCKER, доступный в Интернете.

## <a name="set-up-a-training-experiment"></a>Настройка обучающего эксперимента

В этом разделе вы настроите обучающий эксперимент, инициализируя рабочую область, определив среду и настроив целевой объект вычислений.

### <a name="initialize-a-workspace"></a>Инициализация рабочей области

[Машинное обучение Azure Рабочая область](concept-workspace.md) — это ресурс верхнего уровня для службы. Она предоставляет централизованное место для работы со всеми создаваемыми артефактами. В пакете SDK для Python можно получить доступ к артефактам рабочей области, создав [`Workspace`](/python/api/azureml-core/azureml.core.workspace.workspace) объект.

Создайте `Workspace` объект на основе config.jsв файле, который вы создали в качестве [необходимого компонента](#prerequisites).

```Python
from azureml.core import Workspace

ws = Workspace.from_config()
```

### <a name="define-your-environment"></a>Определение среды

Создайте `Environment` объект и включите DOCKER.

```python
from azureml.core import Environment

fastai_env = Environment("fastai2")
fastai_env.docker.enabled = True
```

Указанный базовый образ в следующем коде поддерживает библиотеку fast.ai, которая позволяет использовать распределенные возможности глубокого обучения. Дополнительные сведения см. в [репозитории FAST.AI DOCKER Hub](https://hub.docker.com/u/fastdotai). 

Если вы используете пользовательский образ DOCKER, возможно, ваша среда Python уже настроена правильно. В этом случае установите `user_managed_dependencies` флаг для `True` использования встроенной среды Python пользовательского образа. По умолчанию Машинное обучение Azure создает среду Conda с указанными зависимостями. Служба запускает скрипт в этой среде вместо использования библиотек Python, установленных на базовом образе.

```python
fastai_env.docker.base_image = "fastdotai/fastai2:latest"
fastai_env.python.user_managed_dependencies = True
```

#### <a name="use-a-private-container-registry-optional"></a>Использование закрытого реестра контейнеров (необязательно)

Чтобы использовать образ из закрытого реестра контейнеров, который отсутствует в рабочей области, используйте команду, `docker.base_image_registry` чтобы указать адрес репозитория и имя пользователя и пароль:

```python
# Set the container registry information.
fastai_env.docker.base_image_registry.address = "myregistry.azurecr.io"
fastai_env.docker.base_image_registry.username = "username"
fastai_env.docker.base_image_registry.password = "password"
```

#### <a name="use-a-custom-dockerfile-optional"></a>Использование настраиваемого Dockerfile (необязательно)

Также можно использовать пользовательский Dockerfile. Используйте этот подход, если необходимо установить пакеты, отличные от Python, как зависимости. Не забудьте установить базовый образ в `None` .

```python 
# Specify Docker steps as a string. 
dockerfile = r"""
FROM mcr.microsoft.com/azureml/base:intelmpi2018.3-ubuntu16.04
RUN echo "Hello from custom container!"
"""

# Set the base image to None, because the image is defined by Dockerfile.
fastai_env.docker.base_image = None
fastai_env.docker.base_dockerfile = dockerfile

# Alternatively, load the string from a file.
fastai_env.docker.base_image = None
fastai_env.docker.base_dockerfile = "./Dockerfile"
```

>[!IMPORTANT]
> Машинное обучение Azure поддерживает только образы DOCKER, которые предоставляют следующее программное обеспечение:
> * Ubuntu 16,04 или более поздней версии.
> * Conda 4.5. # или более поздней версии.
> * Python 3.5 +.

Дополнительные сведения о создании и управлении средой Машинное обучение Azure см. в разделе [Создание и использование программных сред](how-to-use-environments.md). 

### <a name="create-or-attach-a-compute-target"></a>Создание или присоединение целевого объекта вычислений

Для обучения модели необходимо создать [целевой объект вычислений](concept-azure-machine-learning-architecture.md#compute-targets) . В этом руководстве вы создадите `AmlCompute` ресурс для обучения для учебных вычислений.

Создание `AmlCompute` займет несколько минут. Если `AmlCompute` ресурс уже находится в рабочей области, этот код пропускает процесс создания.

Как и в случае с другими службами Azure, существуют ограничения на определенные ресурсы (например, `AmlCompute` ), связанные со службой машинное обучение Azure. Дополнительные сведения см. в разделе [ограничения по умолчанию и как запросить более высокую квоту](how-to-manage-quotas.md).

```python
from azureml.core.compute import ComputeTarget, AmlCompute
from azureml.core.compute_target import ComputeTargetException

# Choose a name for your cluster.
cluster_name = "gpu-cluster"

try:
    compute_target = ComputeTarget(workspace=ws, name=cluster_name)
    print('Found existing compute target.')
except ComputeTargetException:
    print('Creating a new compute target...')
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_NC6',
                                                           max_nodes=4)

    # Create the cluster.
    compute_target = ComputeTarget.create(ws, cluster_name, compute_config)

    compute_target.wait_for_completion(show_output=True)

# Use get_status() to get a detailed status for the current AmlCompute.
print(compute_target.get_status().serialize())
```

## <a name="configure-your-training-job"></a>Настройка учебного задания

В рамках этого руководства используйте сценарий обучения *Train.py* на [GitHub](https://github.com/Azure/azureml-examples/blob/main/workflows/train/fastai/pets/src/train.py). На практике можно выполнить любой пользовательский сценарий обучения и запустить его, как есть, с Машинное обучение Azure.

Создайте `ScriptRunConfig` ресурс, чтобы настроить выполнение задания в нужном [целевом объекте вычислений](how-to-set-up-training-targets.md).

```python
from azureml.core import ScriptRunConfig

src = ScriptRunConfig(source_directory='fastai-example',
                      script='train.py',
                      compute_target=compute_target,
                      environment=fastai_env)
```

## <a name="submit-your-training-job"></a>Отправка задания обучения

При отправке запуска обучения с помощью `ScriptRunConfig` объекта `submit` метод возвращает объект типа `ScriptRun` . Возвращаемый `ScriptRun` объект предоставляет программный доступ к сведениям о ходе выполнения обучения. 

```python
from azureml.core import Experiment

run = Experiment(ws,'Tutorial-fastai').submit(src)
run.wait_for_completion(show_output=True)
```

> [!WARNING]
> Машинное обучение Azure запускает скрипты обучения, копируя весь исходный каталог. Если у вас есть конфиденциальные данные, которые не нужно передавать, используйте [файл. Ignore](how-to-save-write-experiment-files.md#storage-limits-of-experiment-snapshots) или не включайте его в исходный каталог. Вместо этого получите доступ к данным с помощью [хранилища](/python/api/azureml-core/azureml.data)данных.

## <a name="next-steps"></a>Дальнейшие действия
В этой статье вы обучили модель с помощью пользовательского образа DOCKER. Дополнительные сведения о Машинное обучение Azure см. в следующих статьях:
* [Следите за показателями запуска](how-to-track-experiments.md) во время обучения.
* [Развертывание модели](how-to-deploy-custom-docker-image.md) с помощью пользовательского образа DOCKER.
