---
title: Обновление веб-служб
titleSuffix: Azure Machine Learning
description: Узнайте, как обновить веб-службу, которая уже развернута в Машинное обучение Azure. Вы можете обновить такие параметры, как модель, среда и сценарий входа.
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: larryfr
ms.author: gopalv
author: gvashishtha
ms.date: 07/31/2020
ms.custom: deploy
ms.openlocfilehash: cd8bef53bb8f7c42aa6dbd0bc1e1c67400b8f2a0
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102219086"
---
# <a name="update-a-deployed-web-service"></a>Обновление развернутой веб-службы

Из этой статьи вы узнаете, как обновить веб-службу, развернутую с помощью Машинное обучение Azure.

## <a name="prerequisites"></a>Предварительные требования

В этом учебнике предполагается, что вы уже развернули веб-службу с Машинное обучение Azure. Если вам нужно узнать, как развернуть веб-службу, [выполните следующие действия](how-to-deploy-and-where.md).

## <a name="update-web-service"></a>Обновление веб-службы

Чтобы обновить веб-службу, используйте `update` метод. Можно обновить веб-службу, чтобы использовать новую модель, новый скрипт записи или новые зависимости, которые можно указать в конфигурации вывода. Дополнительные сведения см. в документации по [WebService. Update](/python/api/azureml-core/azureml.core.webservice.webservice.webservice?preserve-view=true&view=azure-ml-py#&preserve-view=trueupdate--args-).

См [. раздел метод обновления службы AKS.](/python/api/azureml-core/azureml.core.webservice.akswebservice?preserve-view=true&view=azure-ml-py#&preserve-view=trueupdate-image-none--autoscale-enabled-none--autoscale-min-replicas-none--autoscale-max-replicas-none--autoscale-refresh-seconds-none--autoscale-target-utilization-none--collect-model-data-none--auth-enabled-none--cpu-cores-none--memory-gb-none--enable-app-insights-none--scoring-timeout-ms-none--replica-max-concurrent-requests-none--max-request-wait-time-none--num-replicas-none--tags-none--properties-none--description-none--models-none--inference-config-none--gpu-cores-none--period-seconds-none--initial-delay-seconds-none--timeout-seconds-none--success-threshold-none--failure-threshold-none--namespace-none--token-auth-enabled-none-)

См [. раздел метод обновления службы ACI.](/python/api/azureml-core/azureml.core.webservice.aci.aciwebservice?preserve-view=true&view=azure-ml-py#&preserve-view=trueupdate-image-none--tags-none--properties-none--description-none--auth-enabled-none--ssl-enabled-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none--enable-app-insights-none--models-none--inference-config-none-)

> [!IMPORTANT]
> При создании новой версии модели необходимо вручную обновить каждую службу, которая будет использовать ее.
>
> Пакет SDK нельзя использовать для обновления веб-службы, опубликованной из конструктора Машинное обучение Azure.

**Использование пакета SDK**

В следующем коде показано, как использовать пакет SDK для обновления сценария модели, среды и входа для веб-службы.

```python
from azureml.core import Environment
from azureml.core.webservice import Webservice
from azureml.core.model import Model, InferenceConfig

# Register new model.
new_model = Model.register(model_path="outputs/sklearn_mnist_model.pkl",
                           model_name="sklearn_mnist",
                           tags={"key": "0.1"},
                           description="test",
                           workspace=ws)

# Use version 3 of the environment.
deploy_env = Environment.get(workspace=ws,name="myenv",version="3")
inference_config = InferenceConfig(entry_script="score.py",
                                   environment=deploy_env)

service_name = 'myservice'
# Retrieve existing service.
service = Webservice(name=service_name, workspace=ws)



# Update to new model(s).
service.update(models=[new_model], inference_config=inference_config)
service.wait_for_deployment(show_output=True)
print(service.state)
print(service.get_logs())
```

**Использование интерфейса командной строки**

Вы также можете обновить веб-службу с помощью интерфейса командной строки ML. В следующем примере демонстрируется регистрация новой модели и обновление веб-службы для использования новой модели.

```azurecli
az ml model register -n sklearn_mnist  --asset-path outputs/sklearn_mnist_model.pkl  --experiment-name myexperiment --output-metadata-file modelinfo.json
az ml service update -n myservice --model-metadata-file modelinfo.json
```

> [!TIP]
> В этом примере документ JSON используется для передачи сведений о модели из команды регистрации в команду Update.
>
> Чтобы обновить службу для использования нового скрипта записи или среды, создайте [файл конфигурации вывода](./reference-azure-machine-learning-cli.md#inference-configuration-schema) и укажите его с помощью `ic` параметра.

Дополнительные сведения см. в документации по [AZ ML Service Update](/cli/azure/ext/azure-cli-ml/ml/service#ext-azure-cli-ml-az-ml-service-update) .

## <a name="next-steps"></a>Дальнейшие действия

* [Устранение неполадок при развертывании](how-to-troubleshoot-deployment.md)
* [развертывание в Службе Azure Kubernetes](how-to-deploy-azure-kubernetes-service.md).
* [Создание клиентских приложений для использования веб-служб](how-to-consume-web-service.md)
* [Развертывание модели с помощью пользовательского образа DOCKER](how-to-deploy-custom-docker-image.md)
* [Использование TLS для защиты веб-службы с помощью Машинного обучения Azure](how-to-secure-web-service.md).
* [Мониторинг моделей Машинное обучение Azure с помощью Application Insights](how-to-enable-app-insights.md)
* [Сбор данных для моделей в рабочей среде](how-to-enable-data-collection.md)
* [Создание предупреждений и триггеров событий для развертываний моделей](how-to-use-event-grid.md)
