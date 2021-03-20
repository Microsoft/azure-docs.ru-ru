---
title: Объяснение в автоматизированном ML (Предварительная версия)
titleSuffix: Azure Machine Learning
description: Узнайте, как получить объяснения о том, как автоматизированная модель ML определяет важность функций и делает прогнозы при использовании пакета SDK для Машинное обучение Azure.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, automl, responsible-ml
ms.author: mithigpe
author: minthigpen
ms.date: 07/09/2020
ms.openlocfilehash: 709c85bed4a028c6c168c79cd9fffd6b7b40cb68
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100008049"
---
# <a name="interpretability-model-explanations-in-automated-machine-learning-preview"></a>Интерпретируемость: пояснения к модели в автоматизированном машинном обучении (предварительная версия)



Из этой статьи вы узнаете, как получить объяснения для автоматизированного машинного обучения (Аутомл) в Машинное обучение Azure. Аутомл помогает понять важность функций создаваемых моделей. 

Все версии пакета SDK после 1.0.85 установлены по `model_explainability=True` умолчанию. В пакете SDK версии 1.0.85 и более ранних версий пользователям необходимо задать `model_explainability=True` в `AutoMLConfig` объекте, чтобы можно было использовать интерпретируемость модели. 

Вы узнаете, как выполнять следующие задачи:

- Выполняйте интерпретацию во время обучения для лучшей модели или любой модели.
- Включите визуализации, чтобы видеть закономерности в данных и объяснениях.
- Реализуйте интерпретируемость во время вывода или оценки.

## <a name="prerequisites"></a>Предварительные условия

- Функции интерпретации. Выполните команду `pip install azureml-interpret` , чтобы получить необходимый пакет.
- Знания о создании экспериментов Аутомл. Дополнительные сведения об использовании пакета SDK для Машинное обучение Azure см. в руководстве по [модели регрессии](tutorial-auto-train-models.md) или в статье [Настройка экспериментов аутомл](how-to-configure-auto-train.md).

## <a name="interpretability-during-training-for-the-best-model"></a>Возможности интерпретации во время обучения для наилучшей модели

Извлеките пояснения из `best_run` , которое включает объяснения как для необработанных, так и для инженерных функций.

> [!NOTE]
> Понятность, объяснение лучшей модели недоступна для экспериментов автоматического обучения, которые рекомендуют следующие алгоритмы в качестве наилучшей модели: 
> * ткнфорекастер
> * аутоарима
> * експонентиалсмусинг
> * Prophet
> * Среднее значение 
> * Алгоритма
> * Среднее значение по сезону 
> * Сезонный Байес

### <a name="download-the-engineered-feature-importances-from-the-best-run"></a>Загрузите более высокую важность функций из лучшего запуска.

Вы можете использовать `ExplanationClient` для скачивания объяснений инженерных функций из хранилища артефактов `best_run` . 

```python
from azureml.interpret import ExplanationClient

client = ExplanationClient.from_run(best_run)
engineered_explanations = client.download_model_explanation(raw=False)
print(engineered_explanations.get_feature_importance_dict())
```

### <a name="download-the-raw-feature-importances-from-the-best-run"></a>Загрузка необработанных характеристик компонента из лучшего запуска

Можно использовать `ExplanationClient` для загрузки необработанных объяснений функций из хранилища артефактов `best_run` .

```python
from azureml.interpret import ExplanationClient

client = ExplanationClient.from_run(best_run)
raw_explanations = client.download_model_explanation(raw=True)
print(raw_explanations.get_feature_importance_dict())
```

## <a name="interpretability-during-training-for-any-model"></a>Возможности интерпретации во время обучения для любой модели 

При вычислении объяснений модели и визуализации их можно не ограничиваться объяснением существующей модели для модели Аутомл. Вы также можете получить описание модели с различными тестовыми данными. Действия, описанные в этом разделе, показывают, как вычислять и визуализировать уровень важности функций, основанных на тестовых данных.

### <a name="retrieve-any-other-automl-model-from-training"></a>Получение любой другой модели Аутомл из обучения

```python
automl_run, fitted_model = local_run.get_output(metric='accuracy')
```

### <a name="set-up-the-model-explanations"></a>Настройка объяснений модели

Используйте `automl_setup_model_explanations` для получения инженерных и необработанных объяснений. `fitted_model`Может создавать следующие элементы:

- Рекомендуемые данные из обученных или тестовых примеров
- Списки имен сконструированных функций
- Искомые классы в столбце с меткой в сценариях классификации

`automl_explainer_setup_obj`Содержит все структуры из списка выше.

```python
from azureml.train.automl.runtime.automl_explain_utilities import automl_setup_model_explanations

automl_explainer_setup_obj = automl_setup_model_explanations(fitted_model, X=X_train, 
                                                             X_test=X_test, y=y_train, 
                                                             task='classification')
```

### <a name="initialize-the-mimic-explainer-for-feature-importance"></a>Инициализация объяснения "имитировать" для важности функции

Чтобы создать пояснения для моделей Аутомл, используйте `MimicWrapper` класс. Мимиквраппер можно инициализировать с помощью следующих параметров:

- Объект настройки пояснения
- Рабочая область
- Суррогатная модель для объяснения `fitted_model` модели аутомл

Мимиквраппер также принимает объект, `automl_run` где будут передаваться инженерные объяснения.

```python
from azureml.interpret import MimicWrapper

# Initialize the Mimic Explainer
explainer = MimicWrapper(ws, automl_explainer_setup_obj.automl_estimator,
                         explainable_model=automl_explainer_setup_obj.surrogate_model, 
                         init_dataset=automl_explainer_setup_obj.X_transform, run=automl_run,
                         features=automl_explainer_setup_obj.engineered_feature_names, 
                         feature_maps=[automl_explainer_setup_obj.feature_map],
                         classes=automl_explainer_setup_obj.classes,
                         explainer_kwargs=automl_explainer_setup_obj.surrogate_model_params)
```

### <a name="use-mimic-explainer-for-computing-and-visualizing-engineered-feature-importance"></a>Использование пояснения к процедуре "имитировать" для вычисления и визуализации важности сконструированных функций

Вы можете вызвать `explain()` метод в мимиквраппер с преобразованными примерами тестов, чтобы получить важность признаков для созданных сконструированных функций. Вы также можете войти в [машинное обучение Azure Studio](https://ml.azure.com/) , чтобы просмотреть панель мониторинга значений важности функций созданных сконструированных функций, аутомл феатуризерс.

```python
engineered_explanations = explainer.explain(['local', 'global'], eval_dataset=automl_explainer_setup_obj.X_test_transform)
print(engineered_explanations.get_feature_importance_dict())
```

### <a name="use-mimic-explainer-for-computing-and-visualizing-raw-feature-importance"></a>Использование пояснения к процедуре "имитировать" для вычисления и визуализации необработанных важных компонентов

Вы можете вызвать `explain()` метод в мимиквраппер с преобразованными примерами тестов, чтобы получить важность признаков для необработанных функций. В [машинное обучение Studio](https://ml.azure.com/)можно просмотреть панель мониторинга со значениями важности функций необработанных компонентов.

```python
raw_explanations = explainer.explain(['local', 'global'], get_raw=True,
                                     raw_feature_names=automl_explainer_setup_obj.raw_feature_names,
                                     eval_dataset=automl_explainer_setup_obj.X_test_transform,
                                     raw_eval_dataset=automl_explainer_setup_obj.X_test_raw)
print(raw_explanations.get_feature_importance_dict())
```

## <a name="interpretability-during-inference"></a>Интерпретируемость во время вывода

В этом разделе вы узнаете, как эксплуатацию модель Аутомл с помощью объяснения, которое использовалось для расчета объяснений в предыдущем разделе.

### <a name="register-the-model-and-the-scoring-explainer"></a>Регистрация модели и пояснения к оценкам

Используйте `TreeScoringExplainer` для создания пояснения к оценке, которое будет вычислять значения важности функций во время вывода. Вы инициализируем пояснение к оценке с тем `feature_map` , который был вычислен ранее. 

Сохраните описание оценки, а затем зарегистрируйте модель и описание оценки в службе Управление моделями. Выполните следующий код:

```python
from azureml.interpret.scoring.scoring_explainer import TreeScoringExplainer, save

# Initialize the ScoringExplainer
scoring_explainer = TreeScoringExplainer(explainer.explainer, feature_maps=[automl_explainer_setup_obj.feature_map])

# Pickle scoring explainer locally
save(scoring_explainer, exist_ok=True)

# Register trained automl model present in the 'outputs' folder in the artifacts
original_model = automl_run.register_model(model_name='automl_model', 
                                           model_path='outputs/model.pkl')

# Register scoring explainer
automl_run.upload_file('scoring_explainer.pkl', 'scoring_explainer.pkl')
scoring_explainer_model = automl_run.register_model(model_name='scoring_explainer', model_path='scoring_explainer.pkl')
```

### <a name="create-the-conda-dependencies-for-setting-up-the-service"></a>Создание зависимостей conda для настройки службы

Затем создайте необходимые зависимости среды в контейнере для развернутой модели. Обратите внимание, что azureml — значения по умолчанию с версией >= 1.0.45 должны быть перечислены как зависимость PIP, так как она содержит функции, необходимые для размещения модели в качестве веб-службы.

```python
from azureml.core.conda_dependencies import CondaDependencies

azureml_pip_packages = [
    'azureml-interpret', 'azureml-train-automl', 'azureml-defaults'
]

myenv = CondaDependencies.create(conda_packages=['scikit-learn', 'pandas', 'numpy', 'py-xgboost<=0.80'],
                                 pip_packages=azureml_pip_packages,
                                 pin_sdk_version=True)

with open("myenv.yml","w") as f:
    f.write(myenv.serialize_to_string())

with open("myenv.yml","r") as f:
    print(f.read())

```

### <a name="create-the-scoring-script"></a>Создание скрипта оценки

Напишите сценарий, который загружает модель и создает прогнозы и объяснения на основе нового пакета данных.

```python
%%writefile score.py
import joblib
import pandas as pd
from azureml.core.model import Model
from azureml.train.automl.runtime.automl_explain_utilities import automl_setup_model_explanations


def init():
    global automl_model
    global scoring_explainer

    # Retrieve the path to the model file using the model name
    # Assume original model is named automl_model
    automl_model_path = Model.get_model_path('automl_model')
    scoring_explainer_path = Model.get_model_path('scoring_explainer')

    automl_model = joblib.load(automl_model_path)
    scoring_explainer = joblib.load(scoring_explainer_path)


def run(raw_data):
    data = pd.read_json(raw_data, orient='records')
    # Make prediction
    predictions = automl_model.predict(data)
    # Setup for inferencing explanations
    automl_explainer_setup_obj = automl_setup_model_explanations(automl_model,
                                                                 X_test=data, task='classification')
    # Retrieve model explanations for engineered explanations
    engineered_local_importance_values = scoring_explainer.explain(automl_explainer_setup_obj.X_test_transform)
    # Retrieve model explanations for raw explanations
    raw_local_importance_values = scoring_explainer.explain(automl_explainer_setup_obj.X_test_transform, get_raw=True)
    # You can return any data type as long as it is JSON-serializable
    return {'predictions': predictions.tolist(),
            'engineered_local_importance_values': engineered_local_importance_values,
            'raw_local_importance_values': raw_local_importance_values}
```

### <a name="deploy-the-service"></a>Развертывание службы

Разверните службу с помощью файла conda и файла оценки из предыдущих шагов.

```python
from azureml.core.webservice import Webservice
from azureml.core.webservice import AciWebservice
from azureml.core.model import Model, InferenceConfig
from azureml.core.environment import Environment

aciconfig = AciWebservice.deploy_configuration(cpu_cores=1,
                                               memory_gb=1,
                                               tags={"data": "Bank Marketing",  
                                                     "method" : "local_explanation"},
                                               description='Get local explanations for Bank marketing test data')
myenv = Environment.from_conda_specification(name="myenv", file_path="myenv.yml")
inference_config = InferenceConfig(entry_script="score_local_explain.py", environment=myenv)

# Use configs and models generated above
service = Model.deploy(ws,
                       'model-scoring',
                       [scoring_explainer_model, original_model],
                       inference_config,
                       aciconfig)
service.wait_for_deployment(show_output=True)
```

### <a name="inference-with-test-data"></a>Вывод с тестовыми данными

Вывод с некоторыми тестовыми данными, чтобы увидеть прогнозируемое значение из модели Аутомл, которое в настоящее время поддерживается только в пакете SDK для Машинное обучение Azure. Просмотр важности функций, которые вносят вклад в прогнозируемое значение. 

```python
if service.state == 'Healthy':
    # Serialize the first row of the test data into json
    X_test_json = X_test[:1].to_json(orient='records')
    print(X_test_json)
    # Call the service to get the predictions and the engineered explanations
    output = service.run(X_test_json)
    # Print the predicted value
    print(output['predictions'])
    # Print the engineered feature importances for the predicted value
    print(output['engineered_local_importance_values'])
    # Print the raw feature importances for the predicted value
    print('raw_local_importance_values:\n{}\n'.format(output['raw_local_importance_values']))
```

### <a name="visualize-to-discover-patterns-in-data-and-explanations-at-training-time"></a>Визуализация для обнаружения закономерностей в данных и объяснениях во время обучения

Диаграмму важности функций можно визуализировать в рабочей области в [машинное обучение Studio](https://ml.azure.com). После завершения выполнения Аутомл выберите **Просмотреть сведения о модели** , чтобы просмотреть конкретный запуск. Перейдите на вкладку **пояснения** , чтобы увидеть панель мониторинга визуализации пояснения.

[![Архитектура интерпретации Машинное обучение](./media/how-to-machine-learning-interpretability-automl/automl-explanation.png)](./media/how-to-machine-learning-interpretability-automl/automl-explanation.png#lightbox)

Дополнительные сведения о визуализации панелей мониторинга и конкретных графиках см. в [документации по интерпретированию](how-to-machine-learning-interpretability-aml.md).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о том, как включить пояснения к модели и важность признаков в областях пакета SDK Машинное обучение Azure, отличного от автоматизированного машинного обучения, см. в [статье о концепции, посвященной возможностям интерпретации](how-to-machine-learning-interpretability.md).
