---
title: Развертывание модели для использования со службой "Когнитивный поиск"
titleSuffix: Azure Machine Learning
description: Узнайте, как использовать Машинное обучение Azure для развертывания модели для использования с Когнитивный поиск. Модель используется в качестве пользовательского навыка для расширения возможностей поиска.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: cgronlun
author: cjgronlund
ms.reviewer: larryfr
ms.date: 06/11/2020
ms.custom: deploy
ms.openlocfilehash: 9336dbd4d5615a93bbc029ba51b561b18f5a1c15
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102521991"
---
# <a name="deploy-a-model-for-use-with-cognitive-search"></a>Развертывание модели для использования со службой "Когнитивный поиск"


В этой статье объясняется, как использовать Машинное обучение Azure для развертывания модели для использования с [когнитивный Поиск Azure](../search/search-what-is-azure-search.md).

Когнитивный поиск выполняет обработку содержимого по содержимому разнородных, чтобы сделать его запросным для людей или приложений. Этот процесс можно улучшить с помощью модели, развернутой из Машинное обучение Azure.

Машинное обучение Azure может развернуть обученную модель как веб-службу. Затем веб-служба внедряется в Когнитивный поиск _навык_, который становится частью конвейера обработки.

> [!IMPORTANT]
> Сведения в этой статье относятся к развертыванию модели. Он предоставляет сведения о поддерживаемых конфигурациях развертывания, позволяющих использовать модель Когнитивный поиск.
>
> Сведения о настройке Когнитивный поиск для использования развернутой модели см. в руководстве по [сборке и развертыванию пользовательского навыка с помощью машинное обучение Azure](../search/cognitive-search-tutorial-aml-custom-skill.md) .
>
> Пример, на котором основан учебник, см. в разделе [https://github.com/Azure-Samples/azure-search-python-samples/tree/master/AzureML-Custom-Skill](https://github.com/Azure-Samples/azure-search-python-samples/tree/master/AzureML-Custom-Skill) .

При развертывании модели для использования с Azure Когнитивный поиск развертывание должно удовлетворять следующим требованиям.

* Используйте службу Kubernetes Azure для размещения модели для вывода.
* Включите протокол TLS для службы Azure Kubernetes. Протокол TLS используется для защиты обмена данными по протоколу HTTPS между Когнитивный поиск и развернутой моделью.
* Скрипт записи должен использовать `inference_schema` пакет для создания схемы OpenAPI (Swagger) для службы.
* Скрипт записи также должен принимать данные JSON в качестве входных данных и создавать JSON в качестве выходных данных.


## <a name="prerequisites"></a>Предварительные требования

* Рабочая область машинного обучения Azure. Дополнительные сведения см. в статье [создание машинное обучение Azure рабочей области](how-to-manage-workspace.md).

* Среда разработки Python с установленным пакетом SDK для Машинное обучение Azure. Дополнительные сведения см. в разделе [машинное обучение Azure SDK](/python/api/overview/azure/ml/install).  

* Зарегистрированная модель. Если у вас нет модели, используйте пример записной книжки в [https://github.com/Azure-Samples/azure-search-python-samples/tree/master/AzureML-Custom-Skill](https://github.com/Azure-Samples/azure-search-python-samples/tree/master/AzureML-Custom-Skill) .

* Общее представление [о том, как и где развертываются модели](how-to-deploy-and-where.md).

## <a name="connect-to-your-workspace"></a>Подключение к рабочей области

Рабочая область Машинное обучение Azure предоставляет централизованное расположение для работы со всеми артефактами, создаваемыми при использовании Машинное обучение Azure. Рабочая область хранит историю всех учебных запусков, включая журналы, метрики, выходные данные и моментальные снимки сценариев.

Чтобы подключиться к существующей рабочей области, используйте следующий код:

> [!IMPORTANT]
> Этот фрагмент кода ищет конфигурацию рабочей области в текущем или родительском каталоге. Дополнительные сведения см. в статье [создание рабочих областей машинное обучение Azure и управление ими](how-to-manage-workspace.md). Дополнительные сведения о сохранении конфигурации в файле см. в разделе [Создание файла конфигурации рабочей области](how-to-configure-environment.md#workspace).

```python
from azureml.core import Workspace

try:
    # Load the workspace configuration from local cached inffo
    ws = Workspace.from_config()
    print(ws.name, ws.location, ws.resource_group, ws.location, sep='\t')
    print('Library configuration succeeded')
except:
    print('Workspace not found')
```

## <a name="create-a-kubernetes-cluster"></a>Создание кластера Kubernetes

**Оценка времени**: приблизительно 20 минут.

Кластер Kubernetes — это набор экземпляров виртуальных машин (узлов), которые используются для запуска контейнерных приложений.

При развертывании модели из Машинное обучение Azure в службу Azure Kubernetes модель и все ресурсы, необходимые для размещения ее в качестве веб-службы, упаковываются в контейнер DOCKER. Затем этот контейнер развертывается на кластере.

В следующем коде показано, как создать кластер Azure Kubernetes Service (AKS) для рабочей области.

> [!TIP]
> Вы также можете подключить существующую службу Kubernetes Azure к рабочей области Машинное обучение Azure. Дополнительные сведения см. [в статье Развертывание моделей в службе Kubernetes Azure](how-to-deploy-azure-kubernetes-service.md).

> [!IMPORTANT]
> Обратите внимание, что код использует `enable_ssl()` метод, чтобы включить протокол TLS для кластера. Это необходимо при планировании использования развернутой модели из Cognitive Services.

```python
from azureml.core.compute import AksCompute, ComputeTarget
# Create or attach to an AKS inferencing cluster

# Create the provisioning configuration with defaults
prov_config = AksCompute.provisioning_configuration()

# Enable TLS (sometimes called SSL) communications
# Leaf domain label generates a name using the formula
#  "<leaf-domain-label>######.<azure-region>.cloudapp.azure.net"
#  where "######" is a random series of characters
prov_config.enable_ssl(leaf_domain_label = "contoso")

cluster_name = 'amlskills'
# Try to use an existing compute target by that name.
# If one doesn't exist, create one.
try:
    
    aks_target = ComputeTarget(ws, cluster_name)
    print("Attaching to existing cluster")
except Exception as e:
    print("Creating new cluster")
    aks_target = ComputeTarget.create(workspace = ws, 
                                  name = cluster_name, 
                                  provisioning_configuration = prov_config)
    # Wait for the create process to complete
    aks_target.wait_for_completion(show_output = True)
```

> [!IMPORTANT]
> В Azure будет взиматься плата, если существует кластер AKS. Не забудьте удалить кластер AKS после завершения работы с ним.

Дополнительные сведения об использовании AKS с Машинное обучение Azure см. в статье [развертывание в службе Kubernetes Azure](how-to-deploy-azure-kubernetes-service.md).

## <a name="write-the-entry-script"></a>Написание сценария записи

Сценарий записи получает данные, отправленные в веб-службу, передает их в модель и возвращает результаты оценки. Следующий скрипт загружает модель при запуске, а затем использует модель для оценки данных. Этот файл иногда называют `score.py` .

> [!TIP]
> Скрипт относится только к модели. Например, сценарий должен быть уверен, что платформа будет использоваться с моделью, форматами данных и т. д.

> [!IMPORTANT]
> При планировании использования развернутой модели из Cognitive Services Azure необходимо использовать `inference_schema` пакет, чтобы включить создание схем для развертывания. Этот пакет предоставляет декораторы, позволяющие определить формат входных и выходных данных для веб-службы, выполняющей вывод с помощью модели.

```python
from azureml.core.model import Model
from nlp_architect.models.absa.inference.inference import SentimentInference
from spacy.cli.download import download as spacy_download
import traceback
import json
# Inference schema for schema discovery
from inference_schema.schema_decorators import input_schema, output_schema
from inference_schema.parameter_types.numpy_parameter_type import NumpyParameterType
from inference_schema.parameter_types.standard_py_parameter_type import StandardPythonParameterType

def init():
    """
    Set up the ABSA model for Inference  
    """
    global SentInference
    spacy_download('en')
    aspect_lex = Model.get_model_path('hotel_aspect_lex')
    opinion_lex = Model.get_model_path('hotel_opinion_lex') 
    SentInference = SentimentInference(aspect_lex, opinion_lex)

# Use inference schema decorators and sample input/output to
# build the OpenAPI (Swagger) schema for the deployment
standard_sample_input = {'text': 'a sample input record containing some text' }
standard_sample_output = {"sentiment": {"sentence": "This place makes false booking prices, when you get there, they say they do not have the reservation for that day.", 
                                        "terms": [{"text": "hotels", "type": "AS", "polarity": "POS", "score": 1.0, "start": 300, "len": 6}, 
                                                  {"text": "nice", "type": "OP", "polarity": "POS", "score": 1.0, "start": 295, "len": 4}]}}
@input_schema('raw_data', StandardPythonParameterType(standard_sample_input))
@output_schema(StandardPythonParameterType(standard_sample_output))    
def run(raw_data):
    try:
        # Get the value of the 'text' field from the JSON input and perform inference
        input_txt = raw_data["text"]
        doc = SentInference.run(doc=input_txt)
        if doc is None:
            return None
        sentences = doc._sentences
        result = {"sentence": doc._doc_text}
        terms = []
        for sentence in sentences:
            for event in sentence._events:
                for x in event:
                    term = {"text": x._text, "type":x._type.value, "polarity": x._polarity.value, "score": x._score,"start": x._start,"len": x._len }
                    terms.append(term)
        result["terms"] = terms
        print("Success!")
        # Return the results to the client as a JSON document
        return {"sentiment": result}
    except Exception as e:
        result = str(e)
        # return error message back to the client
        print("Failure!")
        print(traceback.format_exc())
        return json.dumps({"error": result, "tb": traceback.format_exc()})
```

Дополнительные сведения о сценариях входа см. [в разделе как и где развертывать](how-to-deploy-and-where.md).

## <a name="define-the-software-environment"></a>Определение программной среды

Класс среды используется для определения зависимостей Python для службы. Он включает зависимости, необходимые как для модели, так и для скрипта записи. В этом примере он устанавливает пакеты из стандартного индекса PyPI, а также из репозитория GitHub. 

```python
from azureml.core.conda_dependencies import CondaDependencies 
from azureml.core import Environment

conda = None
pip = ["azureml-defaults", "azureml-monitoring", 
       "git+https://github.com/NervanaSystems/nlp-architect.git@absa", 'nlp-architect', 'inference-schema',
       "spacy==2.0.18"]

conda_deps = CondaDependencies.create(conda_packages=None, pip_packages=pip)

myenv = Environment(name='myenv')
myenv.python.conda_dependencies = conda_deps
```

Дополнительные сведения о средах см. в статье [создание сред для обучения и развертывания и управление ими](how-to-use-environments.md).

## <a name="define-the-deployment-configuration"></a>Определение конфигурации развертывания

Конфигурация развертывания определяет среду размещения службы Azure Kubernetes, используемую для запуска веб-службы.

> [!TIP]
> Если вы не уверены в том, что требуется использовать память, ЦП или GPU для развертывания, можно воспользоваться профилированием для изучения этих возможностей. Дополнительные сведения см. [в разделе как и где развертывается модель](how-to-deploy-and-where.md).

```python
from azureml.core.model import Model
from azureml.core.webservice import Webservice
from azureml.core.image import ContainerImage
from azureml.core.webservice import AksWebservice, Webservice

# If deploying to a cluster configured for dev/test, ensure that it was created with enough
# cores and memory to handle this deployment configuration. Note that memory is also used by
# things such as dependencies and AML components.

aks_config = AksWebservice.deploy_configuration(autoscale_enabled=True, 
                                                       autoscale_min_replicas=1, 
                                                       autoscale_max_replicas=3, 
                                                       autoscale_refresh_seconds=10, 
                                                       autoscale_target_utilization=70,
                                                       auth_enabled=True, 
                                                       cpu_cores=1, memory_gb=2, 
                                                       scoring_timeout_ms=5000, 
                                                       replica_max_concurrent_requests=2, 
                                                       max_request_wait_time=5000)
```

Дополнительные сведения см. в справочной документации по [AksService.deploy_configuration](/python/api/azureml-core/azureml.core.webservice.akswebservice#deploy-configuration-autoscale-enabled-none--autoscale-min-replicas-none--autoscale-max-replicas-none--autoscale-refresh-seconds-none--autoscale-target-utilization-none--collect-model-data-none--auth-enabled-none--cpu-cores-none--memory-gb-none--enable-app-insights-none--scoring-timeout-ms-none--replica-max-concurrent-requests-none--max-request-wait-time-none--num-replicas-none--primary-key-none--secondary-key-none--tags-none--properties-none--description-none--gpu-cores-none--period-seconds-none--initial-delay-seconds-none--timeout-seconds-none--success-threshold-none--failure-threshold-none--namespace-none--token-auth-enabled-none--compute-target-name-none-).

## <a name="define-the-inference-configuration"></a>Определение конфигурации вывода

Конфигурация вывода указывает на скрипт записи и объект среды:

```python
from azureml.core.model import InferenceConfig
inf_config = InferenceConfig(entry_script='score.py', environment=myenv)
```

Дополнительные сведения см. в справочной документации по [инференцеконфиг](/python/api/azureml-core/azureml.core.model.inferenceconfig).

## <a name="deploy-the-model"></a>Развертывание модели

Разверните модель в кластере AKS и подождите, пока она создаст службу. В этом примере две зарегистрированные модели загружаются из реестра и развертываются в AKS. После развертывания `score.py` файл в развертывании загружает эти модели и использует их для выполнения вывода.

```python
from azureml.core.webservice import AksWebservice, Webservice

c_aspect_lex = Model(ws, 'hotel_aspect_lex')
c_opinion_lex = Model(ws, 'hotel_opinion_lex') 
service_name = "hotel-absa-v2"

aks_service = Model.deploy(workspace=ws,
                           name=service_name,
                           models=[c_aspect_lex, c_opinion_lex],
                           inference_config=inf_config,
                           deployment_config=aks_config,
                           deployment_target=aks_target,
                           overwrite=True)

aks_service.wait_for_deployment(show_output = True)
print(aks_service.state)
```

Дополнительные сведения см. в справочной документации по [модели](/python/api/azureml-core/azureml.core.model.model).

## <a name="issue-a-sample-query-to-your-service"></a>Выдача примера запроса в службу

В следующем примере используются сведения о развертывании, хранящиеся в `aks_service` переменной в предыдущем разделе кода. Она использует эту переменную для получения URL-адреса оценки и маркера проверки подлинности, необходимых для взаимодействия со службой.

```python
import requests
import json

primary, secondary = aks_service.get_keys()

# Test data
input_data = '{"raw_data": {"text": "This is a nice place for a relaxing evening out with friends. The owners seem pretty nice, too. I have been there a few times including last night. Recommend."}}'

# Since authentication was enabled for the deployment, set the authorization header.
headers = {'Content-Type':'application/json',  'Authorization':('Bearer '+ primary)} 

# Send the request and display the results
resp = requests.post(aks_service.scoring_uri, input_data, headers=headers)
print(resp.text)
```

Результат, возвращенный службой, аналогичен следующему JSON:

```json
{"sentiment": {"sentence": "This is a nice place for a relaxing evening out with friends. The owners seem pretty nice, too. I have been there a few times including last night. Recommend.", "terms": [{"text": "place", "type": "AS", "polarity": "POS", "score": 1.0, "start": 15, "len": 5}, {"text": "nice", "type": "OP", "polarity": "POS", "score": 1.0, "start": 10, "len": 4}]}}
```

## <a name="connect-to-cognitive-search"></a>Подключение к Когнитивный поиск

Сведения об использовании этой модели из Когнитивный поиск см. в руководстве по [сборке и развертыванию пользовательского навыка с помощью машинное обучение Azure](../search/cognitive-search-tutorial-aml-custom-skill.md) .

## <a name="clean-up-the-resources"></a>очищать ресурсы.

Если вы создали кластер AKS специально для этого примера, удалите ресурсы после завершения тестирования с помощью Когнитивный поиск.

> [!IMPORTANT]
> Счета Azure выставляются в зависимости от того, как долго развертывается кластер AKS. Не забудьте очистить ее после завершения.

```python
aks_service.delete()
aks_target.delete()
```

## <a name="next-steps"></a>Дальнейшие действия

* [Создание и развертывание пользовательского навыка с помощью Машинное обучение Azure](../search/cognitive-search-tutorial-aml-custom-skill.md)