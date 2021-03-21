---
title: Развертывание моделей, обученных в конструкторе, с помощью студии
titleSuffix: Azure Machine Learning
description: Используйте Машинное обучение Azure Studio для развертывания моделей машинного обучения без написания единой строки кода.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: keli19
author: likebupt
ms.reviewer: peterlu
ms.date: 10/29/2020
ms.topic: conceptual
ms.custom: how-to, deploy, studio, designer
ms.openlocfilehash: 35acfc51ae76fdacef11f03b1fbd91ad58650ae6
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97722629"
---
# <a name="use-the-studio-to-deploy-models-trained-in-the-designer"></a>Развертывание моделей, обученных в конструкторе, с помощью студии

Из этой статьи вы узнаете, как развернуть модель конструктора в качестве конечной точки в режиме реального времени в Машинное обучение Azure Studio.

После регистрации или загрузки можно использовать обученные модели конструктора, как и любые другие модели. Экспортированные модели можно развертывать в таких случаях, как Интернет вещей (IoT) и локальное развертывание.

Развертывание в студии состоит из следующих шагов:

1. Зарегистрируйте обученную модель.
1. Скачайте файл скриптов записи и conda зависимостей для модели.
1. Используемых Настройте скрипт записи.
1. Разверните модель в целевом объекте вычислений.

Можно также развернуть модели непосредственно в конструкторе, чтобы пропустить регистрацию модели и шаги скачивания файлов. Это может быть полезно для быстрого развертывания. Дополнительные сведения см. в разделе [развертывание модели с помощью конструктора](tutorial-designer-automobile-price-deploy.md).

Модели, обученные в конструкторе, также можно развернуть с помощью пакета SDK или интерфейса командной строки (CLI). Дополнительные сведения см. в статье [развертывание существующей модели с помощью машинное обучение Azure](how-to-deploy-existing-model.md).

## <a name="prerequisites"></a>Предварительные условия

* [Рабочая область Машинное обучение Azure](how-to-manage-workspace.md)

* Завершенный конвейер обучения, содержащий один из следующих модулей:
    - [Модуль "Обучение модели"](./algorithm-module-reference/train-model.md)
    - [Обучение модуля модели обнаружения аномалий](./algorithm-module-reference/train-anomaly-detection-model.md)
    - [Обучение модуля модели кластеризации](./algorithm-module-reference/train-clustering-model.md)
    - [Модуль "обучение модели Pytorch"](./algorithm-module-reference/train-pytorch-model.md)
    - [Модуль рекомендации по обучению SVD](./algorithm-module-reference/train-svd-recommender.md)
    - [Модуль модели обучения Vowpal Wabbit](./algorithm-module-reference/train-vowpal-wabbit-model.md)
    - [&ный модуль глубокой модели для всех уровней](./algorithm-module-reference/train-wide-and-deep-recommender.md)

## <a name="register-the-model"></a>Регистрация модели

После завершения обучающего конвейера Зарегистрируйте обученную модель в Машинное обучение Azure рабочей области, чтобы получить доступ к модели в других проектах.

1. Выберите [модуль обучение модели](./algorithm-module-reference/train-model.md).
1. На правой панели выберите вкладку **выходные данные + журналы** .
1. На снимке экрана щелкните значок **зарегистрировать модель** ![ значок шестеренки ](./media/how-to-deploy-model-designer/register-model-icon.png) .

    ![Снимок экрана с правой панелью модуля обучения модели](./media/how-to-deploy-model-designer/train-model-right-pane.png)

1. Введите имя модели, а затем нажмите кнопку **сохранить**.

После регистрации модели ее можно найти на странице ресурсов **модели** в студии.
    
![Снимок экрана зарегистрированной модели на странице ресурсов моделей](./media/how-to-deploy-model-designer/models-asset-page.png)

## <a name="download-the-entry-script-file-and-conda-dependencies-file"></a>Скачайте файл скрипта записи и файл зависимостей conda

Для развертывания модели в Машинное обучение Azure Studio необходимы следующие файлы:

- **Файл скрипта записи** — загружает обученную модель, обрабатывает входные данные из запросов, выполняет вывод в реальном времени и возвращает результат. `score.py`При завершении работы модуля **обучение модели** конструктор автоматически создает файл скрипта записи.

- **Файл зависимостей Conda** — указывает, от каких пакетов PIP и Conda зависит ваша веб-служба. Конструктор автоматически создает `conda_env.yaml` файл при завершении работы модуля **обучение модели** .

Эти два файла можно скачать на правой панели модуля **обучение модели** :

1. Выберите модуль **Train Model** (Обучение модели).
1. На вкладке **выходы и журналы** выберите папку `trained_model_outputs` .
1. Скачайте `conda_env.yaml` файл и `score.py` файл.

    ![Снимок экрана: скачать файлы для развертывания в правой области](./media/how-to-deploy-model-designer/download-artifacts-in-right-pane.png)

Кроме того, можно загрузить файлы со страницы ресурсов **модели** после регистрации модели.

1. Перейдите на страницу ресурсов **модели** .
1. Выберите модель, которую требуется развернуть.
1. Перейдите на вкладку **артефакты** .
1. Выберите папку `trained_model_outputs`
1. Скачайте `conda_env.yaml` файл и `score.py` файл.  

    ![Снимок экрана: Загрузка файлов для развертывания на странице сведений о модели](./media/how-to-deploy-model-designer/download-artifacts-in-models-page.png)

> [!NOTE]
> `score.py`Файл предоставляет почти те же функциональные возможности, что и модули **оценки моделей** . Тем не менее, некоторые модули, такие как [Оценка SVD, рекомендации](./algorithm-module-reference/score-svd-recommender.md)по [оценке Wide и Deep](./algorithm-module-reference/score-wide-and-deep-recommender.md), а [оценки Vowpal Wabbit](./algorithm-module-reference/score-vowpal-wabbit-model.md) имеют параметры для различных режимов оценки. Эти параметры также можно изменить в скрипте записи.
>
>Дополнительные сведения о настройке параметров в `score.py` файле см. в разделе [Настройка скрипта записи](#configure-the-entry-script).

## <a name="deploy-the-model"></a>Развертывание модели

После загрузки необходимых файлов можно приступать к развертыванию модели.

1. На странице ресурсов **модели** выберите зарегистрированную модель.
1. Нажмите кнопку **развернуть** .
1. В меню Конфигурация введите следующие сведения.

    - Введите имя конечной точки.
    - Выберите, чтобы развернуть модель в [службе Azure Kubernetes](how-to-deploy-azure-kubernetes-service.md) или в [экземпляре контейнера Azure](how-to-deploy-azure-container-instance.md).
    - Отправьте объект `score.py` для **файла скрипта записи**.
    - Отправьте `conda_env.yml` **файл для файла зависимостей Conda**. 

    >[!TIP]
    > В **дополнительном** параметре можно задать емкость ЦП/памяти и другие параметры для развертывания. Эти параметры важны для некоторых моделей, таких как модели PyTorch, которые потребляют значительный объем Мемери (около 4 ГБ).

1. Выберите **развернуть** , чтобы развернуть модель в качестве конечной точки в режиме реального времени.

    ![Снимок экрана: страница "развертывание модели" в активе модели](./media/how-to-deploy-model-designer/deploy-model.png)

## <a name="consume-the-real-time-endpoint"></a>Использование конечной точки в режиме реального времени

После завершения развертывания можно найти конечную точку в режиме реального времени на странице ресурсов **конечных точек** . После этого вы найдете конечную точку RESTFUL, которую клиенты могут использовать для отправки запросов в конечную точку в режиме реального времени. 

> [!NOTE]
> Конструктор также создает образец JSON-файла данных для тестирования, который можно скачать `_samples.json` в папку **trained_model_outputs** .

Используйте следующий пример кода для использования конечной точки в режиме реального времени.

```python

import json
from pathlib import Path
from azureml.core.workspace import Workspace, Webservice
 
service_name = 'YOUR_SERVICE_NAME'
ws = Workspace.get(
    name='WORKSPACE_NAME',
    subscription_id='SUBSCRIPTION_ID',
    resource_group='RESOURCEGROUP_NAME'
)
service = Webservice(ws, service_name)
sample_file_path = '_samples.json'
 
with open(sample_file_path, 'r') as f:
    sample_data = json.load(f)
score_result = service.run(json.dumps(sample_data))
print(f'Inference result = {score_result}')
```

### <a name="consume-computer-vision-related-real-time-endpoints"></a>Использование связанных с компьютерной концепцией конечных точек в реальном времени

При использовании компьютерных концепций, связанных с конечными точками в реальном времени, необходимо преобразовать образы в байты, так как веб-служба принимает только строку в качестве входных данных. Ниже приведен пример кода.

```python
import base64
import json
from copy import deepcopy
from pathlib import Path
from azureml.studio.core.io.image_directory import (IMG_EXTS, image_from_file, image_to_bytes)
from azureml.studio.core.io.transformation_directory import ImageTransformationDirectory

# image path
image_path = Path('YOUR_IMAGE_FILE_PATH')

# provide the same parameter setting as in the training pipeline. Just an example here.
image_transform = [
    # format: (op, args). {} means using default parameter values of torchvision.transforms.
    # See https://pytorch.org/docs/stable/torchvision/transforms.html
    ('Resize', 256),
    ('CenterCrop', 224),
    # ('Pad', 0),
    # ('ColorJitter', {}),
    # ('Grayscale', {}),
    # ('RandomResizedCrop', 256),
    # ('RandomCrop', 224),
    # ('RandomHorizontalFlip', {}),
    # ('RandomVerticalFlip', {}),
    # ('RandomRotation', 0),
    # ('RandomAffine', 0),
    # ('RandomGrayscale', {}),
    # ('RandomPerspective', {}),
]
transform = ImageTransformationDirectory.create(transforms=image_transform).torch_transform

# download _samples.json file under Outputs+logs tab in the right pane of Train Pytorch Model module
sample_file_path = '_samples.json'
with open(sample_file_path, 'r') as f:
    sample_data = json.load(f)

# use first sample item as the default value
default_data = sample_data[0]
data_list = []
for p in image_path.iterdir():
    if p.suffix.lower() in IMG_EXTS:
        data = deepcopy(default_data)
        # convert image to bytes
        data['image'] = base64.b64encode(image_to_bytes(transform(image_from_file(p)))).decode()
        data_list.append(data)

# use data.json as input of consuming the endpoint
data_file_path = 'data.json'
with open(data_file_path, 'w') as f:
    json.dump(data_list, f)
```

## <a name="configure-the-entry-script"></a>Настройка сценария записи

Некоторые модули в конструкторе, такие как [Score SVD](./algorithm-module-reference/score-svd-recommender.md), рекомендация, [Оценка Wide и глубокая рекомендация](./algorithm-module-reference/score-wide-and-deep-recommender.md), а также [Оценка Vowpal Wabbit](./algorithm-module-reference/score-vowpal-wabbit-model.md) , имеют параметры для различных режимов оценки. 

В этом разделе вы узнаете, как обновить эти параметры в файле скрипта записи.

В следующем примере показано, как обновить поведение по умолчанию для обученной **углубленной модели рекомендаций &** . По умолчанию `score.py` файл сообщает веб-службе о необходимости прогнозирования оценок между пользователями и элементами. 

Вы можете изменить файл скрипта записи, чтобы сделать рекомендации по элементам, и вернуть Рекомендуемые элементы, изменив `recommender_prediction_kind` параметр.


```python
import os
import json
from pathlib import Path
from collections import defaultdict
from azureml.studio.core.io.model_directory import ModelDirectory
from azureml.designer.modules.recommendation.dnn.wide_and_deep.score. \
    score_wide_and_deep_recommender import ScoreWideAndDeepRecommenderModule
from azureml.designer.serving.dagengine.utils import decode_nan
from azureml.designer.serving.dagengine.converter import create_dfd_from_dict

model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'trained_model_outputs')
schema_file_path = Path(model_path) / '_schema.json'
with open(schema_file_path) as fp:
    schema_data = json.load(fp)


def init():
    global model
    model = ModelDirectory.load(load_from_dir=model_path)


def run(data):
    data = json.loads(data)
    input_entry = defaultdict(list)
    for row in data:
        for key, val in row.items():
            input_entry[key].append(decode_nan(val))

    data_frame_directory = create_dfd_from_dict(input_entry, schema_data)

    # The parameter names can be inferred from Score Wide and Deep Recommender module parameters:
    # convert the letters to lower cases and replace whitespaces to underscores.
    score_params = dict(
        trained_wide_and_deep_recommendation_model=model,
        dataset_to_score=data_frame_directory,
        training_data=None,
        user_features=None,
        item_features=None,
        ################### Note #################
        # Set 'Recommender prediction kind' parameter to enable item recommendation model
        recommender_prediction_kind='Item Recommendation',
        recommended_item_selection='From All Items',
        maximum_number_of_items_to_recommend_to_a_user=5,
        whether_to_return_the_predicted_ratings_of_the_items_along_with_the_labels='True')
    result_dfd, = ScoreWideAndDeepRecommenderModule().run(**score_params)
    result_df = result_dfd.data
    return json.dumps(result_df.to_dict("list"))
```

Для **широкой & глубоких** и **Vowpal Wabbit** моделей, можно настроить параметр режима оценки с помощью следующих методов.

- Имена параметров — это сочетания символов в нижнем регистре и символы подчеркивания для имен параметров для [оценки Vowpal Wabbit Model](./algorithm-module-reference/score-vowpal-wabbit-model.md) , а также [Расширенная и глубокая рекомендация](./algorithm-module-reference/score-wide-and-deep-recommender.md).
- Значения параметров типа режима — это строки соответствующих имен параметров. В приведенных выше кодах **рекомендуется использовать прогнозирующий тип** , который может иметь значение `'Rating Prediction'` или `'Item Recommendation'` . Другие значения не допускаются.

Для обученной модели, **рекомендуемой для SVD** , имена и значения параметров могут быть менее очевидны, и вы можете найти приведенные ниже таблицы, чтобы решить, как задавать параметры.

| Имя параметра в [подсчете рекомендаций SVD](./algorithm-module-reference/score-svd-recommender.md)                           | Имя параметра в файле скрипта записи |
| ------------------------------------------------------------ | --------------------------------------- |
| Тип прогноза подсистемы                                  | prediction_kind                         |
| Выбор рекомендуемых элементов                                   | recommended_item_selection              |
| Минимальный размер пула рекомендаций для одного пользователя    | min_recommendation_pool_size            |
| Максимальное количество элементов, которые необходимо рекомендовать пользователю               | max_recommended_item_count              |
| Следует ли возвращать прогнозируемые оценки элементов вместе с метками | return_ratings                          |

В следующем коде показано, как задать параметры для рекомендации SVD, в котором используются все шесть параметров, чтобы рекомендовать оценку элементов с присоединенными прогнозируемыми рейтингами.

```python
score_params = dict(
        learner=model,
        test_data=DataTable.from_dfd(data_frame_directory),
        training_data=None,
        # RecommenderPredictionKind has 2 members, 'RatingPrediction' and 'ItemRecommendation'. You
        # can specify prediction_kind parameter with one of them.
        prediction_kind=RecommenderPredictionKind.ItemRecommendation,
        # RecommendedItemSelection has 3 members, 'FromAllItems', 'FromRatedItems', 'FromUndatedItems'.
        # You can specify recommended_item_selection parameter with one of them.
        recommended_item_selection=RecommendedItemSelection.FromRatedItems,
        min_recommendation_pool_size=1,
        max_recommended_item_count=3,
        return_ratings=True,
    )
```


## <a name="next-steps"></a>Дальнейшие действия

* [Обучение модели в конструкторе](tutorial-designer-automobile-price-train-score.md)
* [Развертывание моделей с помощью пакета SDK для Машинное обучение Azure](how-to-deploy-and-where.md)
* [Устранение неполадок при развертывании](how-to-troubleshoot-deployment.md)
* [развертывание в Службе Azure Kubernetes](how-to-deploy-azure-kubernetes-service.md).
* [Создание клиентских приложений для использования веб-служб](how-to-consume-web-service.md)
* [Обновление веб-службы](how-to-deploy-update-web-service.md)
