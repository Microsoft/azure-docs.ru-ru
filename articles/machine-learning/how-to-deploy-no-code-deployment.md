---
title: Без развертывания кода (Предварительная версия)
titleSuffix: Azure Machine Learning
description: Отсутствие развертывания кода позволяет развертывать модель как веб-службу без необходимости вручную создавать скрипт записи.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: gopalv
author: gvashishtha
ms.date: 07/31/2020
ms.topic: conceptual
ms.custom: deploy
ms.reviewer: larryfr
ms.openlocfilehash: a17126695aa5138d1df7fd17cfaa2f5f75ad1004
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93324927"
---
# <a name="preview-no-code-model-deployment"></a>Образца Развертывание модели без кода

Развертывание модели без кода сейчас находится на этапе предварительной версии и поддерживает следующие платформы машинного обучения:

## <a name="tensorflow-savedmodel-format"></a>Формат Саведмодел TensorFlow
Модели TensorFlow необходимо зарегистрировать в **формате саведмодел** для работы без развертывания модели кода.

Сведения о том, как создать Саведмодел, см. в [этой ссылке](https://www.tensorflow.org/guide/saved_model) .

Поддерживается любая версия TensorFlow, указанная в разделе "Теги" на [TensorFlow, обслуживающем DockerHub](https://registry.hub.docker.com/r/tensorflow/serving/tags).

```python
from azureml.core import Model

model = Model.register(workspace=ws,
                       model_name='flowers',                        # Name of the registered model in your workspace.
                       model_path='./flowers_model',                # Local Tensorflow SavedModel folder to upload and register as a model.
                       model_framework=Model.Framework.TENSORFLOW,  # Framework used to create the model.
                       model_framework_version='1.14.0',            # Version of Tensorflow used to create the model.
                       description='Flowers model')

service_name = 'tensorflow-flower-service'
service = Model.deploy(ws, service_name, [model])
```

## <a name="onnx-models"></a>Модели ONNX

Регистрация и развертывание модели ONNX поддерживаются для любого графа вывода ONNX. Этапы предварительной обработки и выполнения процессов в настоящее время не поддерживаются.

Ниже приведен пример регистрации и развертывания модели MNIST ONNX.

```python
from azureml.core import Model

model = Model.register(workspace=ws,
                       model_name='mnist-sample',                  # Name of the registered model in your workspace.
                       model_path='mnist-model.onnx',              # Local ONNX model to upload and register as a model.
                       model_framework=Model.Framework.ONNX ,      # Framework used to create the model.
                       model_framework_version='1.3',              # Version of ONNX used to create the model.
                       description='Onnx MNIST model')

service_name = 'onnx-mnist-service'
service = Model.deploy(ws, service_name, [model])
```

Чтобы оценить модель, см. статью [использование машинное обучение Azureной модели, развернутой в качестве веб-службы](./how-to-consume-web-service.md). Многие проекты ONNX используют файлы protobuf для сжатия данных для обучения и проверки, что может затруднять знание того, какой формат данных ожидается службой. Разработчик модели должен документировать документацию для разработчиков:

* Формат входных данных (JSON или Binary)
* Форма входных данных и тип (например, массив с плавающей запятой Shape [100,100, 3])
* Сведения о домене (например, для изображения, цветового пространства, порядка компонентов и того, являются ли значения нормализованными)

Если вы используете Pytorch, то [Экспорт моделей из Pytorch в ONNX](https://github.com/onnx/tutorials/blob/master/tutorials/PytorchOnnxExport.ipynb) содержит подробные сведения о преобразовании и ограничениях. 

## <a name="scikit-learn-models"></a>Модели scikit-learn

Не поддерживается развертывание модели кода для всех встроенных типов моделей scikit-учиться.

Ниже приведен пример регистрации и развертывания модели sklearn без дополнительного кода.

```python
from azureml.core import Model
from azureml.core.resource_configuration import ResourceConfiguration

model = Model.register(workspace=ws,
                       model_name='my-sklearn-model',                # Name of the registered model in your workspace.
                       model_path='./sklearn_regression_model.pkl',  # Local file to upload and register as a model.
                       model_framework=Model.Framework.SCIKITLEARN,  # Framework used to create the model.
                       model_framework_version='0.19.1',             # Version of scikit-learn used to create the model.
                       resource_configuration=ResourceConfiguration(cpu=1, memory_in_gb=0.5),
                       description='Ridge regression model to predict diabetes progression.',
                       tags={'area': 'diabetes', 'type': 'regression'})
                       
service_name = 'my-sklearn-service'
service = Model.deploy(ws, service_name, [model])
```

> [!NOTE]
> Модели, поддерживающие predict_proba, будут использовать этот метод по умолчанию. Чтобы переопределить это для использования Predict, можно изменить текст записи следующим образом:

```python
import json


input_payload = json.dumps({
    'data': [
        [ 0.03807591,  0.05068012,  0.06169621, 0.02187235, -0.0442235,
         -0.03482076, -0.04340085, -0.00259226, 0.01990842, -0.01764613]
    ],
    'method': 'predict'  # If you have a classification model, the default behavior is to run 'predict_proba'.
})

output = service.run(input_payload)

print(output)
```

> [!NOTE]
> Эти зависимости включены в предварительно сформированный контейнер вывода scikit-учиться:

```yaml
    - dill
    - azureml-defaults
    - inference-schema[numpy-support]
    - scikit-learn
    - numpy
    - joblib
    - pandas
    - scipy
    - sklearn_pandas
```
## <a name="next-steps"></a>Дальнейшие действия

* [Устранение неполадок при развертывании](how-to-troubleshoot-deployment.md)
* [развертывание в Службе Azure Kubernetes](how-to-deploy-azure-kubernetes-service.md).
* [Создание клиентских приложений для использования веб-служб](how-to-consume-web-service.md)
* [Обновление веб-службы](how-to-deploy-update-web-service.md)
* [Развертывание модели с помощью пользовательского образа DOCKER](how-to-deploy-custom-docker-image.md)
* [Использование TLS для защиты веб-службы с помощью Машинного обучения Azure](how-to-secure-web-service.md).
* [Мониторинг моделей Машинное обучение Azure с помощью Application Insights](how-to-enable-app-insights.md)
* [Сбор данных для моделей в рабочей среде](how-to-enable-data-collection.md)
* [Создание предупреждений и триггеров событий для развертываний моделей](how-to-use-event-grid.md)