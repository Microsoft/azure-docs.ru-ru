---
title: Модель профиля использование памяти и ЦП
titleSuffix: Azure Machine Learning
description: Узнайте, как профилировать модель перед развертыванием. Профилирование определяет использование памяти и ЦП вашей модели.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: gopalv
author: gvashishtha
ms.date: 07/31/2020
ms.topic: conceptual
zone_pivot_groups: aml-control-methods
ms.reviewer: larryfr
ms.custom: deploy
ms.openlocfilehash: b9ae40b3d2673961f9b84ed702f18b25b79b6d0c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93320399"
---
# <a name="profile-your-model-to-determine-resource-utilization"></a>Профилирование модели для определения использования ресурсов

В этой статье показано, как профилировать машинное обучение для моделирования, чтобы определить, сколько ресурсов ЦП и памяти необходимо выделить для модели при ее развертывании в качестве веб-службы.

## <a name="prerequisites"></a>Предварительные условия

В этой статье предполагается, что модель обучена и зарегистрирована с помощью Машинное обучение Azure. Пример обучения и регистрации модели scikit с помощью Машинное обучение Azure см. в этом [руководстве](how-to-train-scikit-learn.md) .

## <a name="limitations"></a>Ограничения

* Профилирование не будет работать, если реестр контейнеров Azure (запись контроля доступа) для рабочей области находится за виртуальной сетью.

## <a name="run-the-profiler"></a>Запуск профилировщика

После регистрации модели и подготовки других компонентов, необходимых для ее развертывания, можно определить ресурсы ЦП и памяти, которые потребуется развернутой службе. Профилирование проверяет службу, которая выполняет модель, и возвращает такие сведения, как загрузка ЦП, использование памяти и задержка ответа. Он также предоставляет рекомендации для ЦП и памяти на основе использования ресурсов.

Для профилирования модели потребуется:
* Зарегистрированная модель.
* Конфигурация вывода, основанная на скрипте записи и определении среды вывода.
* Табличный набор данных с одним столбцом, в котором каждая строка содержит строку, представляющую образец данных запроса.

> [!IMPORTANT]
> На этом этапе поддерживается только Профилирование служб, в которых данные запроса должны быть строками, например: сериализованный JSON, текст, сериализованное изображение строки и т. д. Содержимое каждой строки набора данных (строка) помещается в текст HTTP-запроса и отправляется службе, которая инкапсулирует модель для оценки.

> [!IMPORTANT]
> Мы поддерживаем профилирование до 2 процессоров в ChinaEast2 и Усговаризона регионе.

Ниже приведен пример того, как можно создать входной набор данных для профилирования службы, которая предполагает, что входящие данные запроса должны содержать сериализованный код JSON. В этом случае мы создали набор данных на основе 100 экземпляров одного и того же содержимого данных запроса. В реальных сценариях мы рекомендуем использовать большие наборы данных, содержащие различные входные данные, особенно если использование ресурсов модели зависит от входных данных.

::: zone pivot="py-sdk"

```python
import json
from azureml.core import Datastore
from azureml.core.dataset import Dataset
from azureml.data import dataset_type_definitions

input_json = {'data': [[1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
                       [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]]}
# create a string that can be utf-8 encoded and
# put in the body of the request
serialized_input_json = json.dumps(input_json)
dataset_content = []
for i in range(100):
    dataset_content.append(serialized_input_json)
dataset_content = '\n'.join(dataset_content)
file_name = 'sample_request_data.txt'
f = open(file_name, 'w')
f.write(dataset_content)
f.close()

# upload the txt file created above to the Datastore and create a dataset from it
data_store = Datastore.get_default(ws)
data_store.upload_files(['./' + file_name], target_path='sample_request_data')
datastore_path = [(data_store, 'sample_request_data' +'/' + file_name)]
sample_request_data = Dataset.Tabular.from_delimited_files(
    datastore_path, separator='\n',
    infer_column_types=True,
    header=dataset_type_definitions.PromoteHeadersBehavior.NO_HEADERS)
sample_request_data = sample_request_data.register(workspace=ws,
                                                   name='sample_request_data',
                                                   create_new_version=True)
```

После того как набор данных, содержащий образец данных запроса, готов, создайте конфигурацию определения. Конфигурация вывода основана на score.py и определении среды. В следующем примере показано, как создать конфигурацию вывода и выполнить профилирование.

```python
from azureml.core.model import InferenceConfig, Model
from azureml.core.dataset import Dataset


model = Model(ws, id=model_id)
inference_config = InferenceConfig(entry_script='path-to-score.py',
                                   environment=myenv)
input_dataset = Dataset.get_by_name(workspace=ws, name='sample_request_data')
profile = Model.profile(ws,
            'unique_name',
            [model],
            inference_config,
            input_dataset=input_dataset)

profile.wait_for_completion(True)

# see the result
details = profile.get_details()
```

::: zone-end

::: zone pivot="cli"


Следующая команда показывает, как выполнить профилирование модели с помощью интерфейса командной строки:

```azurecli-interactive
az ml model profile -g <resource-group-name> -w <workspace-name> --inference-config-file <path-to-inf-config.json> -m <model-id> --idi <input-dataset-id> -n <unique-name>
```

> [!TIP]
> Чтобы сохранить данные, возвращаемые при профилировании, используйте теги или свойства модели. Использование тегов или свойств сохраняет данные в модели в реестре модели. В следующих примерах демонстрируется добавление нового тега, `requestedCpu` содержащего `requestedMemoryInGb` сведения и.
>
> ```python
> model.add_tags({'requestedCpu': details['requestedCpu'],
>                 'requestedMemoryInGb': details['requestedMemoryInGb']})
> ```
>
> ```azurecli-interactive
> az ml model profile -g <resource-group-name> -w <workspace-name> --i <model-id> --add-tag requestedCpu=1 --add-tag requestedMemoryInGb=0.5
> ```

::: zone-end

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

