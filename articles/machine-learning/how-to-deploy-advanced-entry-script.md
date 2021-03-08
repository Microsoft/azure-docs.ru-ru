---
title: Скрипт записи автора для расширенных сценариев
titleSuffix: Azure Machine Learning entry script authoring
description: Узнайте, как писать скрипты ввода Машинное обучение Azure для предварительной и последующей обработки во время развертывания.
author: gvashishtha
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.date: 09/17/2020
ms.author: gopalv
ms.reviewer: larryfr
ms.custom: deploy
ms.openlocfilehash: 22b22cf364f4422c358fc71db74465b839ddae45
ms.sourcegitcommit: 6386854467e74d0745c281cc53621af3bb201920
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2021
ms.locfileid: "102454974"
---
# <a name="advanced-entry-script-authoring"></a>Разработка расширенных сценариев ввода

В этой статье показано, как написать скрипты записи для специализированных вариантов использования.

## <a name="prerequisites"></a>Предварительные требования

В этой статье предполагается, что у вас уже есть обученная модель машинного обучения, которую планируется развернуть с помощью Машинное обучение Azure. Дополнительные сведения о развертывании модели см. в [этом руководстве](how-to-deploy-and-where.md).

## <a name="automatically-generate-a-swagger-schema"></a>Автоматическое создание схемы Swagger

Чтобы автоматически создать схему для веб-службы, укажите образец входных и (или) выходных данных в конструкторе для одного из объектов определенного типа. Тип и образец используются для автоматического создания схемы. После этого Машинное обучение Azure создает спецификацию [OpenAPI](https://swagger.io/docs/specification/about/) (Swagger) для веб-службы во время развертывания. 

> [!WARNING]
> Для образца ввода или вывода не следует использовать конфиденциальные или закрытые данные. На странице Swagger для размещения, размещенного на AML, отображаются образцы данных. 

В настоящее время поддерживаются следующие типы:

* `pandas`
* `numpy`
* `pyspark`
* Стандартный объект Python

Чтобы использовать формирование схемы, включите пакет с открытым исходным кодом `inference-schema` 1.1.0 или более поздней версии в файл зависимостей. Дополнительные сведения об этом пакете см. в разделе [https://github.com/Azure/InferenceSchema](https://github.com/Azure/InferenceSchema) . Чтобы создать соответствие Swagger для автоматического использования веб-службы, функция запуска скрипта оценки () должна иметь форму API:
* Первый параметр типа "Стандардписонпараметертипе", именованные **входные данные** и вложенный.
* Необязательный второй параметр типа "Стандардписонпараметертипе" с именем **GlobalParameters**.
* Возвращает словарь типа "Стандардписонпараметертипе" с именем **Results** и Nested.

Определите форматы входных и выходных данных в `input_sample` переменных и `output_sample` , которые представляют форматы запросов и ответов для веб-службы. Используйте эти примеры в декораторах входных и выходных функций `run()` функции. В следующем примере scikit-учиться используется создание схемы.



## <a name="power-bi-compatible-endpoint"></a>Конечная точка, совместимая с Power BI 

В следующем примере показано, как определить форму API в соответствии с инструкциями выше. Этот метод поддерживается для использования развернутой веб-службы из Power BI. (Дополнительные[сведения об использовании веб-службы из Power BI](/power-bi/service-machine-learning-integration).)

```python
import json
import pickle
import numpy as np
import pandas as pd
import azureml.train.automl
from sklearn.externals import joblib
from sklearn.linear_model import Ridge

from inference_schema.schema_decorators import input_schema, output_schema
from inference_schema.parameter_types.standard_py_parameter_type import StandardPythonParameterType
from inference_schema.parameter_types.numpy_parameter_type import NumpyParameterType
from inference_schema.parameter_types.pandas_parameter_type import PandasParameterType


def init():
    global model
    # Replace filename if needed.
    model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'sklearn_regression_model.pkl')
    # Deserialize the model file back into a sklearn model.
    model = joblib.load(model_path)


# providing 3 sample inputs for schema generation
numpy_sample_input = NumpyParameterType(np.array([[1,2,3,4,5,6,7,8,9,10],[10,9,8,7,6,5,4,3,2,1]],dtype='float64'))
pandas_sample_input = PandasParameterType(pd.DataFrame({'name': ['Sarah', 'John'], 'age': [25, 26]}))
standard_sample_input = StandardPythonParameterType(0.0)

# This is a nested input sample, any item wrapped by `ParameterType` will be described by schema
sample_input = StandardPythonParameterType({'input1': numpy_sample_input, 
                                        'input2': pandas_sample_input, 
                                        'input3': standard_sample_input})

sample_global_parameters = StandardPythonParameterType(1.0) # this is optional
sample_output = StandardPythonParameterType([1.0, 1.0])
outputs = StandardPythonParameterType({'Results':sample_output}) # 'Results' is case sensitive

@input_schema('Inputs', sample_input) 
# 'Inputs' is case sensitive

@input_schema('GlobalParameters', sample_global_parameters) 
# this is optional, 'GlobalParameters' is case sensitive

@output_schema(outputs)

def run(Inputs, GlobalParameters): 
    # the parameters here have to match those in decorator, both 'Inputs' and 
    # 'GlobalParameters' here are case sensitive
    try:
        data = Inputs['input1']
        # data will be convert to target format
        assert isinstance(data, np.ndarray)
        result = model.predict(data)
        return result.tolist()
    except Exception as e:
        error = str(e)
        return error
```


## <a name="binary-ie-image-data"></a><a id="binary-data"></a> Двоичные данные (т. е. изображение)

Если модель принимает двоичные данные, например изображение, необходимо изменить `score.py` файл, используемый для развертывания, чтобы принимать необработанные HTTP-запросы. Чтобы принимать необработанные данные, используйте `AMLRequest` класс в скрипте записи и добавьте `@rawhttp` декоратор в `run()` функцию.

Ниже приведен пример `score.py` , который принимает двоичные данные:

```python
from azureml.contrib.services.aml_request import AMLRequest, rawhttp
from azureml.contrib.services.aml_response import AMLResponse
from PIL import Image
import json


def init():
    print("This is init()")
    

@rawhttp
def run(request):
    print("This is run()")
    
    if request.method == 'GET':
        # For this example, just return the URL for GETs.
        respBody = str.encode(request.full_path)
        return AMLResponse(respBody, 200)
    elif request.method == 'POST':
        file_bytes = request.files["image"]
        image = Image.open(file_bytes).convert('RGB')
        # For a real-world solution, you would load the data from reqBody
        # and send it to the model. Then return the response.

        # For demonstration purposes, this example just returns the size of the image as the response..
        return AMLResponse(json.dumps(image.size), 200)
    else:
        return AMLResponse("bad request", 500)
```


> [!IMPORTANT]
> Класс `AMLRequest` находится в пространстве имен `azureml.contrib`. Сущности в этом пространстве имен часто изменяются, так как мы работаем над улучшением службы. Все, что есть в этом пространстве имен, следует рассматривать как предварительную версию, которая не полностью поддерживается корпорацией Майкрософт.
>
> Если необходимо протестировать это в локальной среде разработки, можно установить компоненты с помощью следующей команды:
>
> ```shell
> pip install azureml-contrib-services
> ```

`AMLRequest`Класс позволяет получить доступ только к необработанным данным, отправленным в Score.py, на стороне клиента нет клиентского компонента. С клиента данные отводятся как обычные. Например, следующий код Python считывает файл изображения и отправляет данные:

```python
import requests

uri = service.scoring_uri
image_path = 'test.jpg'
files = {'image': open(image_path, 'rb').read()}
response = requests.post(url, files=files)

print(response.json)
```

<a id="cors"></a>

## <a name="cross-origin-resource-sharing-cors"></a>Общий доступ к ресурсам независимо от источника (CORS)

Совместное использование ресурсов в разных источниках позволяет запрашивать ресурсы на веб-странице из другого домена. CORS работает через заголовки HTTP, отправленные с клиентским запросом и возвращаемые с ответом службы. Дополнительные сведения о CORS и допустимых заголовках см. в разделе [общий доступ к ресурсам между источниками](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) в Википедии.

Чтобы настроить развертывание модели для поддержки CORS, используйте `AMLResponse` класс в скрипте записи. Этот класс позволяет задать заголовки для объекта ответа.

В следующем примере задается `Access-Control-Allow-Origin` заголовок для ответа из скрипта записи:

```python
from azureml.contrib.services.aml_request import AMLRequest, rawhttp
from azureml.contrib.services.aml_response import AMLResponse


def init():
    print("This is init()")

@rawhttp
def run(request):
    print("This is run()")
    print("Request: [{0}]".format(request))
    if request.method == 'GET':
        # For this example, just return the URL for GETs.
        respBody = str.encode(request.full_path)
        return AMLResponse(respBody, 200)
    elif request.method == 'POST':
        reqBody = request.get_data(False)
        # For a real-world solution, you would load the data from reqBody
        # and send it to the model. Then return the response.

        # For demonstration purposes, this example
        # adds a header and returns the request body.
        resp = AMLResponse(reqBody, 200)
        resp.headers['Access-Control-Allow-Origin'] = "http://www.example.com"
        return resp
    else:
        return AMLResponse("bad request", 500)
```

> [!IMPORTANT]
> Класс `AMLResponse` находится в пространстве имен `azureml.contrib`. Сущности в этом пространстве имен часто изменяются, так как мы работаем над улучшением службы. Все, что есть в этом пространстве имен, следует рассматривать как предварительную версию, которая не полностью поддерживается корпорацией Майкрософт.
>
> Если необходимо протестировать это в локальной среде разработки, можно установить компоненты с помощью следующей команды:
>
> ```shell
> pip install azureml-contrib-services
> ```


> [!WARNING]
> Машинное обучение Azure будет маршрутизировать только запросы POST и GET к контейнерам, запускающим службу оценки. Это может вызвать ошибки из-за браузеров, использующих запросы параметров для предварительного рейса запросов CORS.
> 


## <a name="load-registered-models"></a>Загрузка зарегистрированных моделей

Существует два способа определения модели в скрипте записи:
* `AZUREML_MODEL_DIR`: Переменная среды, содержащая путь к расположению модели.
* `Model.get_model_path`: API, возвращающий путь к файлу модели, используя зарегистрированное имя модели.

#### <a name="azureml_model_dir"></a>AZUREML_MODEL_DIR

AZUREML_MODEL_DIR — это переменная среды, созданная во время развертывания службы. Эту переменную среды можно использовать для поиска расположения развернутых моделей.

В следующей таблице описано значение AZUREML_MODEL_DIR в зависимости от числа развернутых моделей:

| Развертывание | Значение переменной среды |
| ----- | ----- |
| Одна модель | Путь к папке, содержащей модель. |
| Несколько моделей | Путь к папке, содержащей все модели. Модели расположены по имени и версии в этой папке ( `$MODEL_NAME/$VERSION` ). |

Во время регистрации и развертывания модели модели помещаются в AZUREML_MODEL_DIR путь, и их исходные имена файлов сохраняются.

Чтобы получить путь к файлу модели в скрипте записи, объедините переменную среды с нужным путем к файлу.

**Пример одной модели**
```python
# Example when the model is a file
model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'sklearn_regression_model.pkl')

# Example when the model is a folder containing a file
file_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'my_model_folder', 'sklearn_regression_model.pkl')
```

**Пример с несколькими моделями**

В этом сценарии в рабочей области регистрируются две модели:

* `my_first_model`: Содержит один файл ( `my_first_model.pkl` ), и существует только одна версия ( `1` ).
* `my_second_model`: Содержит один файл ( `my_second_model.pkl` ), и существуют две версии, `1` и `2` .

При развертывании службы в операции развертывания предоставляются обе модели:

```python
first_model = Model(ws, name="my_first_model", version=1)
second_model = Model(ws, name="my_second_model", version=2)
service = Model.deploy(ws, "myservice", [first_model, second_model], inference_config, deployment_config)
```

В образе DOCKER, на котором размещена служба, `AZUREML_MODEL_DIR` переменная среды содержит каталог, в котором находятся модели.
В этом каталоге каждая из моделей находится в каталоге `MODEL_NAME/VERSION` . Где `MODEL_NAME` — имя зарегистрированной модели, а `VERSION` — версия модели. Файлы, составляющие зарегистрированную модель, хранятся в этих каталогах.

В этом примере пути будут выглядеть так `$AZUREML_MODEL_DIR/my_first_model/1/my_first_model.pkl` : и `$AZUREML_MODEL_DIR/my_second_model/2/my_second_model.pkl` .


```python
# Example when the model is a file, and the deployment contains multiple models
first_model_name = 'my_first_model'
first_model_version = '1'
first_model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), first_model_name, first_model_version, 'my_first_model.pkl')
second_model_name = 'my_second_model'
second_model_version = '2'
second_model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), second_model_name, second_model_version, 'my_second_model.pkl')
```

### <a name="get_model_path"></a>get_model_path

При регистрации модели вы предоставляете имя модели, используемое для управления моделью в реестре. Это имя используется с методом [model.get_model_path ()](/python/api/azureml-core/azureml.core.model.model?preserve-view=true&view=azure-ml-py#&preserve-view=trueget-model-path-model-name--version-none---workspace-none-) для получения пути к файлу модели или файлам в локальной файловой системе. При регистрации папки или коллекции файлов этот API возвращает путь к каталогу, содержащему эти файлы.

При регистрации модели ей присваивается имя. Имя соответствует месту размещения модели: локально или во время развертывания службы.

## <a name="framework-specific-examples"></a>Примеры для конкретной платформы

Дополнительные примеры сценариев записи для конкретных вариантов использования машинного обучения можно найти ниже:

* [PyTorch](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks/pytorch)
* [TensorFlow](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks/tensorflow);
* [Keras](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/keras/train-hyperparameter-tune-deploy-with-keras/train-hyperparameter-tune-deploy-with-keras.ipynb);
* [AutoML](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/automated-machine-learning/classification-bank-marketing-all-features)
* [ONNX](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/deployment/onnx/)

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
