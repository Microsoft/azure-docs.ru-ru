---
title: Отслеживание экспериментов и развертывание моделей
titleSuffix: Azure Data Science Virtual Machine
description: Узнайте, как относить эксперименты с виртуальной машиной для обработки и анализа данных с помощью Машинное обучение Azure и Млфлов.
services: machine-learning
ms.service: data-science-vm
author: samkemp
ms.author: samkemp
ms.topic: conceptual
ms.date: 07/17/2020
ms.openlocfilehash: 3795d531c5c4c543587ab817c05cd1cfeea6be06
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100518544"
---
# <a name="track-experiments-and-deploy-models-in-azure-machine-learning"></a>Отслеживание экспериментов и развертывание моделей в Машинное обучение Azure

Чтобы улучшить процесс создания модели, отслеживайте эксперименты с машинным обучением Azure и метрики выполнения. Из этой статьи вы узнаете, как добавить код ведения журнала в сценарий обучения с помощью API [млфлов](https://mlflow.org/) и отвести эксперимент в машинное обучение Azure.

На следующей схеме показано, как с помощью отслеживания MLflow можно следить за метриками выполнения эксперимента и сохранять артефакты модели в рабочей области Машинного обучения Azure.

![Прослеживание экспериментов](./media/how-to-track-experiments/mlflow-diagram-track.png)

## <a name="prerequisites"></a>Предварительные условия

* Необходимо [подготавливать Рабочая область машинного обучения Azure](../how-to-manage-workspace.md#create-a-workspace)

## <a name="create-a-new-notebook"></a>Создание записной книжки

Пакет SDK для Машинное обучение Azure и Млфлов предварительно установлен на виртуальной машине обработки и анализа данных и доступен в среде **azureml_py36_ \*** conda. В JupyterLab щелкните средство запуска и выберите следующий kernel:

![Выбор ядра](./media/how-to-track-experiments/experiment-tracking-1.png)

## <a name="set-up-the-workspace"></a>Настройка рабочей области

Перейдите к [портал Azure](https://portal.azure.com) и выберите рабочую область, подготовленную в рамках предварительных требований. Вы увидите __файл Download config.json__ (см. ниже). Скачайте конфигурацию и убедитесь, что она хранится в рабочем каталоге на DSVM.

![Получить файл конфигурации](./media/how-to-track-experiments/experiment-tracking-2.png)

В конфигурации содержатся такие сведения, как имя рабочей области, подписка и т. д. Это означает, что вам не нужно жестко кодировать эти параметры.

## <a name="track-dsvm-runs"></a>Отслеживание запусков DSVM

Добавьте следующий код в записную книжку (или скрипт), чтобы задать объект рабочей области AzureML.

```Python
import mlflow
from azureml.core import Workspace

ws = Workspace.from_config()

mlflow.set_tracking_uri(ws.get_mlflow_tracking_uri())
```

>[!NOTE]
URI отслеживания действителен до часа. Если вы перезапустили сценарий после некоторого времени простоя, используйте API get_mlflow_tracking_uri, чтобы получить новый универсальный код ресурса (URI).

### <a name="load-the-data"></a>Загрузка данных

В этом примере используется набор данных по диабету — хорошо известный небольшой набор данных, входящий в состав библиотеки scikit-learn. Эта ячейка загружает набор данных и разделяет его на случайные обучающие и проверочные наборы.

```python
from sklearn.datasets import load_diabetes
from sklearn.linear_model import Ridge
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import train_test_split
from sklearn.externals import joblib

X, y = load_diabetes(return_X_y = True)
columns = ['age', 'gender', 'bmi', 'bp', 's1', 's2', 's3', 's4', 's5', 's6']
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=0)
data = {
    "train":{"X": X_train, "y": y_train},
    "test":{"X": X_test, "y": y_test}
}

print ("Data contains", len(data['train']['X']), "training samples and",len(data['test']['X']), "test samples")
```

### <a name="add-tracking"></a>Добавление функции отслеживания

Добавьте функцию отслеживания эксперимента с помощью пакета SDK для службы "Машинное обучение Azure" и отправьте сохраненную модель в запись о выполнении для эксперимента. Следующий код добавляет журналы и отправляет файл модели в запуск эксперимента. Модель также регистрируется в реестре Машинное обучение Azure Model.

```python
# Get an experiment object from Azure Machine Learning
from azureml.mlflow import register_model
experiment_name = 'experiment_with_mlflow'
mlflow.set_experiment(experiment_name)

with mlflow.start_run():
    # Log the algorithm parameter alpha to the run
    mlflow.log_param('alpha', 0.03)

    # Create, fit, and test the scikit-learn Ridge regression model
    regression_model = Ridge(alpha=0.03)
    regression_model.fit(data['train']['X'], data['train']['y'])
    preds = regression_model.predict(data['test']['X'])

    # Output the Mean Squared Error to the notebook and to the run
    print('Mean Squared Error is', mean_squared_error(data['test']['y'], preds))
    mlflow.log_metric('mse', mean_squared_error(data['test']['y'], preds))

    # Save the model
    model_file_name = 'model.pkl'
    joblib.dump(value = regression_model, filename = model_file_name)

    # upload the model file explicitly into artifacts
    mlflow.log_artifact(model_file_name)
    # register the model
    register_model(mlflow.active_run(), 'diabetes_model', 'model.pkl', model_framework="ScikitLearn")
```

### <a name="view-runs-in-azure-machine-learning"></a>Просмотр запусков в Машинное обучение Azure

Вы можете просмотреть запуск эксперимента в [машинное обучение Azure Studio](https://ml.azure.com). Щелкните __эксперименты__ в меню слева и выберите "experiment_with_mlflow" (или, если вы решили задать имя эксперимента по-другому в приведенном выше фрагменте кода, щелкните имя, которое использовалось):

![Выбор эксперимента](./media/how-to-track-experiments/mlflow-experiments.png)

Должно отобразиться сообщение об ошибке "в журнале в квадрате" (MSE):

![СЦЕНАРИЕВ](./media/how-to-track-experiments/mlflow-experiments-2.png)

Если щелкнуть запуск, вы увидите другие сведения, а также выбранную модель в выходных данных и __журналах__ .

## <a name="deploy-model-in-azure-machine-learning"></a>Развертывание модели в Машинное обучение Azure

В этом разделе показано, как развертывать модели, обученные на DSVM, в Машинное обучение Azure.

### <a name="step-1-create-inference-compute"></a>Шаг 1. Создание вычислений вывода

В меню слева в [AzureML Studio](https://ml.azure.com) щелкните " __вычисления__ ", а затем вкладку " __кластеры вывода__ ". Затем щелкните __+ создать__ , как описано ниже.

![Создание вычислений вывода](./media/how-to-track-experiments/mlflow-experiments-6.png)

В __новой области кластера__ выводятся сведения о заполнении для:

* Имя вычислений
* Служба Kubernetes — выберите создать
* Выберите регион
* Выберите размер виртуальной машины (для целей данного учебника достаточно по умолчанию Standard_D3_v2)
* Назначение кластера — выбор __разработки и тестирования__
* Число узлов должно равняться __1__
* Конфигурация сети — базовый

Затем щелкните __создать__.

![сведения о вычислении](./media/how-to-track-experiments/mlflow-experiments-7.png)

### <a name="step-2-deploy-no-code-inference-service"></a>Шаг 2. Развертывание службы вывода без кода

Когда мы зарегистрировали модель в коде с помощью `register_model` , мы указали платформу как sklearn. Машинное обучение Azure не поддерживает развертывание кода для следующих платформ:

* scikit-learn
* Формат Савемодел Tensorflow
* Формат модели ONNX

Развертывание без кода означает, что можно выполнить развертывание непосредственно из артефакта модели без необходимости указывать какой-либо конкретный Скрипт оценки.

Чтобы развернуть модель диабета, перейдите в меню слева в [машинное обучение Azure Studio](https://ml.azure.com) и выберите __модели__. Затем щелкните зарегистрированную diabetes_model:

![Выбор модели](./media/how-to-track-experiments/mlflow-experiments-3.png)

Затем нажмите кнопку __развернуть__ в области сведения о модели:

![Развернуть](./media/how-to-track-experiments/mlflow-experiments-4.png)

Мы выполним развертывание модели в кластере вывода (служба Azure Kubernetes), созданном на шаге 1. Заполните приведенные ниже сведения, указав имя службы и имя кластера вычислений AKS (созданного на шаге 1). Также рекомендуется увеличить __емкость резерва ЦП__ до 1 (от 0,1) и __емкость резерва памяти__ равна 1 (из 0,5). Вы можете увеличить это увеличение, щелкнув __Дополнительно__ и заполнив подробные сведения. Затем нажмите кнопку __развернуть__.

![сведения о развертывании](./media/how-to-track-experiments/mlflow-experiments-5.png)

### <a name="step-3-consume"></a>Шаг 3. Использование

После успешного развертывания модели вы увидите следующее (для перехода на эту страницу щелкните конечные точки в меню слева, > затем щелкните имя развернутой службы):

![Использование модели](./media/how-to-track-experiments/mlflow-experiments-8.png)

Вы должны увидеть, что состояние развертывания __переходит к__ состоянию __работоспособности__. Кроме того, в этом разделе подробностей представлены конечная точка и URL-адреса Swagger, которые разработчик приложения может использовать для интеграции модели машинного обучения в свои приложения.

Вы можете проверить конечную точку с помощью [POST](https://www.postman.com/)или использовать пакет SDK для AzureML:

```python
from azureml.core import Webservice
import json

# if you called your service differently then change the name below
service = Webservice(ws, name="diabetes-service")

input_payload = json.dumps({
    'data': X_test[0:2].tolist(),
    'method': 'predict'  # If you have a classification model, you can get probabilities by changing this to 'predict_proba'.
})

output = service.run(input_payload)

print(output)
```

### <a name="step-4-clean-up"></a>Шаг 4. Очистка

Удалите вычисление, созданное на шаге 1, чтобы не вычислять непрерывные затраты на вычисления. В меню слева в Машинное обучение Azure Studio щелкните вычисления кластеры вывода > > выберите вычисления > удалить.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о [развертывании моделей в AzureML](../how-to-deploy-and-where.md)
