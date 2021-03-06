---
title: Отслеживание экспериментов машинного обучения с помощью MLflow
titleSuffix: Azure Machine Learning
description: Настройте Млфлов с Машинное обучение Azure для записи метрик и артефактов из моделей машинного обучения и развертывания моделей машинного обучения в качестве веб-службы.
services: machine-learning
author: shivp950
ms.author: shipatel
ms.service: machine-learning
ms.subservice: core
ms.reviewer: nibaccam
ms.date: 12/23/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 02684ba91c207357e15684870a6fa0ceab3e17ff
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102520971"
---
# <a name="train-and-track-ml-models-with-mlflow-and-azure-machine-learning-preview"></a>Обучение и отслеживание моделей машинного обучения с помощью Млфлов и Машинное обучение Azure (Предварительная версия)

Из этой статьи вы узнаете, как включить универсальный код ресурса (URI) отслеживания Млфлов и API ведения журналов, который называется [отслеживанием млфлов](https://mlflow.org/docs/latest/quickstart.html#using-the-tracking-api), чтобы подключиться машинное обучение Azure в качестве серверной части экспериментов млфлов. 

К поддерживаемым возможностям относятся: 

+ Мониторинг и запись метрик и артефактов экспериментов в [рабочей области машинное обучение Azure](./concept-azure-machine-learning-architecture.md#workspace). Если вы уже используете отслеживание MLflow для экспериментов, то эта рабочая область обеспечит централизованное, безопасное и масштабируемое расположение для хранения метрик и моделей обучения.

+ Отправка заданий обучения с помощью [проектов млфлов](https://www.mlflow.org/docs/latest/projects.html) с поддержкой машинное обучение Azure серверной части (Предварительная версия). Задания можно отправлять локально с отслеживанием Машинное обучение Azure или переносить запуски в облако, например с помощью [машинное обучение Azure вычислений](./how-to-create-attach-compute-cluster.md).

+ Отслеживание моделей и управление ими в Млфлов и в реестре моделей Машинное обучение Azure.

[MLflow](https://www.mlflow.org) — это библиотека с открытым кодом для управления жизненным циклом экспериментов машинного обучения. Отслеживание Млфлов — это компонент Млфлов, который регистрирует и отслеживает метрики и артефакты обучающего запуска, независимо от среды эксперимента — локально на компьютере, на удаленном целевом объекте вычислений, виртуальной машине или [кластере Azure Databricks](how-to-use-mlflow-azure-databricks.md). 

>[!NOTE]
> Как библиотека с открытым исходным кодом, Млфлов часто меняются. Таким образом, функции, предоставляемые через Машинное обучение Azure и интеграцию с Млфлов, следует рассматривать как предварительную версию, а не полностью поддерживать корпорацией Майкрософт.

На следующей схеме показано, как с помощью отслеживания MLflow можно следить за метриками выполнения эксперимента и сохранять артефакты модели в рабочей области Машинного обучения Azure.

![Схема взаимодействия MLflow с Машинным обучением Azure](./media/how-to-use-mlflow/mlflow-diagram-track.png)

> [!TIP]
> Сведения в этом документе предназначены главным образом для специалистов по обработке и анализу данных и разработчиков, желающих отслеживать процесс обучения модели. Если вы являетесь администратором, который заинтересован в наблюдении из Машинного обучения Azure за использованием ресурсов и событиями, такими как квоты, завершенные обучающие запуски или завершенные развертывания моделей, ознакомьтесь с разделом [Мониторинг Машинного обучения Azure](monitor-azure-machine-learning.md).

## <a name="compare-mlflow-and-azure-machine-learning-clients"></a>Сравнение клиентов MLflow и Машинного обучения Azure

 В следующей таблице перечислены различные клиенты, которые могут использовать Машинное обучение Azure, и соответствующие им возможности функций.

 Отслеживание MLflow предлагает функции ведения журнала метрик и хранилища артефактов, которые также доступны только при использовании [пакета SDK Python для Машинного обучения Azure](/python/api/overview/azure/ml/intro).

| Функция | Развертывание & отслеживания Млфлов | Пакет SDK Python для Машинного обучения Azure |  Интерфейс командной строки службы "Машинное обучение Azure" | Студия машинного обучения Azure.|
|---|---|---|---|---|
| Управление рабочей областью |   | ✓ | ✓ | ✓ |
| Использование хранилищ данных  |   | ✓ | ✓ | |
| Ведение журнала метрик      | ✓ | ✓ |   | |
| Передача артефактов | ✓ | ✓ |   | |
| Просмотр метрик     | ✓ | ✓ | ✓ | ✓ |
| Управление вычислительными ресурсами   |   | ✓ | ✓ | ✓ |
| Развертывание моделей    | ✓ | ✓ | ✓ | ✓ |
|Мониторинг производительности модели||✓|  |   |
| Определение смещения данных |   | ✓ |   | ✓ |

## <a name="prerequisites"></a>Предварительные требования

* Установите пакет `azureml-mlflow`. 
    * Этот пакет автоматически переносится в пакет `azureml-core` [SDK для машинное обучение Azure Python](/python/api/overview/azure/ml/install), который обеспечивает подключение для млфлов к рабочей области.
* [Создайте рабочую область Машинного обучения Azure](how-to-manage-workspace.md).
    * Узнайте, какие [разрешения доступа требуются для выполнения операций млфлов с рабочей областью](how-to-assign-roles.md#mlflow-operations).

## <a name="track-local-runs"></a>Отслеживание локальных выполнений

Интеграция отслеживания MLflow с Машинным обучением Azure позволяет хранить данные журнала метрик и артефактов локальных выполнений в рабочей области Машинного обучения Azure.

Импортируйте классы `mlflow` и [`Workspace`](/python/api/azureml-core/azureml.core.workspace%28class%29), чтобы получить доступ к универсальному коду ресурса (URI) отслеживания MLflow и настроить рабочую область.

В следующем коде метод `get_mlflow_tracking_uri()` присваивает рабочему пространству `ws` уникальный адрес URI отслеживания, а метод `set_tracking_uri()` указывает этот адрес в качестве URI отслеживания MLflow.

```Python
import mlflow
from azureml.core import Workspace

ws = Workspace.from_config()

mlflow.set_tracking_uri(ws.get_mlflow_tracking_uri())
```

>[!NOTE]
>URI отслеживания действителен до часа. Если вы перезапустили сценарий после некоторого времени простоя, используйте API get_mlflow_tracking_uri, чтобы получить новый универсальный код ресурса (URI).

Задайте имя эксперимента MLflow с помощью `set_experiment()` и запустите обучение с помощью `start_run()`. Затем используйте `log_metric()`, чтобы активировать API ведения журнала MLflow и начать запись в журнал метрик выполнения обучения.

```Python
experiment_name = 'experiment_with_mlflow'
mlflow.set_experiment(experiment_name)

with mlflow.start_run():
    mlflow.log_metric('alpha', 0.03)
```

## <a name="track-remote-runs"></a>Отслеживание удаленных выполнений

Удаленное выполнение позволяет обучать модели на более мощных компьютерах, таких как виртуальные машины с поддержкой GPU или кластеры Вычислительной среды Машинного обучения. Дополнительные сведения о различных параметрах вычислений см. в статье [использование целевых объектов вычислений для обучения модели](how-to-set-up-training-targets.md) .

Интеграция отслеживания MLflow с Машинным обучением Azure позволяет хранить данные журнала метрик и артефактов удаленных выполнений в рабочей области Машинного обучения Azure. При любом запуске с кодом отслеживания Млфлов в нем метрики автоматически регистрируются в рабочей области. 

В следующем примере conda среда включает `mlflow` и `azureml-mlflow` в качестве пакетов PIP. 


```yaml
name: sklearn-example
dependencies:
  - python=3.6.2
  - scikit-learn
  - matplotlib
  - numpy
  - pip:
    - azureml-mlflow
    - numpy
```

В сценарии Настройте среду выполнения вычислений и обучения с помощью [`Environment`](/python/api/azureml-core/azureml.core.environment.environment) класса. Затем создайте конструкцию  [`ScriptRunConfig`](/python/api/azureml-core/azureml.core.script_run_config.scriptrunconfig) с удаленным вычислением в качестве целевого объекта вычислений.

```Python
import mlflow

with mlflow.start_run():
    mlflow.log_metric('example', 1.23)
```

При данной конфигурации выполнения вычислений и обучения для отправки выполнения используется метод `Experiment.submit()`. Этот метод автоматически задает URI отслеживания MLflow и направляет данные ведения журнала из MLflow в рабочую область.

```Python
run = exp.submit(src)
```

## <a name="train-with-mlflow-projects"></a>Обучение с помощью проектов Млфлов

[Проекты млфлов](https://mlflow.org/docs/latest/projects.html) позволяют организовать и описать код, чтобы позволить другим специалистам по обработке и анализу данных (или автоматизированным средствам) запустить его. Проекты Млфлов с Машинное обучение Azure позволяют отслеживанить и управлять обучающими запусками в рабочей области. 

В этом примере показано, как отправлять проекты Млфлов локально с отслеживанием Машинное обучение Azure.

Установите `azureml-mlflow` пакет, чтобы использовать отслеживание млфлов с машинное обучение Azure в локальных экспериментах. Эксперименты можно запускать с помощью Jupyter Notebook или редактора кода.

```shell
pip install azureml-mlflow
```

Импортируйте классы `mlflow` и [`Workspace`](/python/api/azureml-core/azureml.core.workspace%28class%29), чтобы получить доступ к универсальному коду ресурса (URI) отслеживания MLflow и настроить рабочую область.

```Python
import mlflow
from azureml.core import Workspace

ws = Workspace.from_config()

mlflow.set_tracking_uri(ws.get_mlflow_tracking_uri())
```

Задайте имя эксперимента MLflow с помощью `set_experiment()` и запустите обучение с помощью `start_run()`. Затем используйте, `log_metric()` чтобы активировать API ведения журнала млфлов и начать запись в журнал метрик выполнения обучения.

```Python
experiment_name = 'experiment-with-mlflow-projects'
mlflow.set_experiment(experiment_name)
```

Создайте объект конфигурации серверной части для хранения необходимой информации для интеграции, такой как, целевой объект вычислений и используемый тип управляемой среды.

```python
backend_config = {"USE_CONDA": False}
```
Добавьте `azureml-mlflow` пакет в качестве зависимости PIP к файлу конфигурации среды, чтобы отслеживанию метрик и ключевых артефактов в рабочей области. 

``` shell
name: mlflow-example
channels:
  - defaults
  - anaconda
  - conda-forge
dependencies:
  - python=3.6
  - scikit-learn=0.19.1
  - pip
  - pip:
    - mlflow
    - azureml-mlflow
```
Отправьте локальный запуск и убедитесь, что задан параметр `backend = "azureml" ` . С помощью этого параметра можно отправлять запуски локально и получать дополнительную поддержку автоматического отслеживания вывода, файлов журналов, моментальных снимков и ошибок печати в рабочей области. 

Просматривайте свои запуски и метрики в [машинное обучение Azure Studio](overview-what-is-machine-learning-studio.md). 


```python
local_env_run = mlflow.projects.run(uri=".", 
                                    parameters={"alpha":0.3},
                                    backend = "azureml",
                                    use_conda=False,
                                    backend_config = backend_config, 
                                    )

```

## <a name="view-metrics-and-artifacts-in-your-workspace"></a>Просмотр метрик и артефактов в рабочей области

Метрики и артефакты из журнала MLflow хранятся в рабочей области. Чтобы просмотреть их в любое время, перейдите в рабочую область и найдите эксперимент по имени в [Студии машинного обучения Azure](https://ml.azure.com).  Либо выполните приведенный ниже код. 

```python
run.get_metrics()
```

## <a name="manage-models"></a>Модели (управление) 

Зарегистрируйте и отследите модели с помощью [реестра машинное обучение Azure Model](concept-model-management-and-deployment.md#register-package-and-deploy-models-from-anywhere) , который поддерживает реестр модели млфлов. Машинное обучение Azure модели согласовываются со схемой модели Млфлов, что упрощает экспорт и импорт этих моделей в разных рабочих процессах. Млфлов связанные метаданные, такие как, ID запуска, также помечаются зарегистрированной моделью для отслеживания. Пользователи могут отправлять обучающие запуски, регистрировать и развертывать модели, созданные при выполнении Млфлов. 

Если вы хотите развернуть и зарегистрировать готовую к работе модель в течение одного шага, см. статью [развертывание и регистрация моделей млфлов](how-to-deploy-mlflow-models.md).

Чтобы зарегистрировать и просмотреть модель из выполнения, выполните следующие действия.

1. После завершения выполнения вызовите `register_model()` метод.

    ```python
    # the model folder produced from the run is registered. This includes the MLmodel file, model.pkl and the conda.yaml.
    run.register_model(model_name = 'my-model', model_path = 'model')
    ```

1. Просмотрите зарегистрированную модель в рабочей области с помощью [машинное обучение Azure Studio](overview-what-is-machine-learning-studio.md).

    В следующем примере Зарегистрированная модель `my-model` содержит млфлов отслеживание метаданных. 

    ![Register-млфлов — модель](./media/how-to-use-mlflow/registered-mlflow-model.png)

1. Перейдите на вкладку **артефакты** , чтобы просмотреть все файлы модели, которые совпадают со схемой модели млфлов (conda. YAML, млмодел, Model. PKL).

    ![модель-схема](./media/how-to-use-mlflow/mlflow-model-schema.png)

1. Выберите Млмодел, чтобы просмотреть файл Млмодел, созданный при выполнении.

    ![Млмодел-Schema](./media/how-to-use-mlflow/mlmodel-view.png)


## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не планируете использовать зарегистрированные метрики и артефакты в рабочей области, возможность удалить их по отдельности в данный момент недоступна. Вместо этого удалите группу ресурсов, содержащую учетную запись хранения и рабочую область, чтобы не взиматься плата.

1. На портале Azure выберите **Группы ресурсов** в левой части окна.

   ![Удаление ресурсов на портале Azure](./media/how-to-use-mlflow/delete-resources.png)

1. В списке выберите созданную группу ресурсов.

1. Выберите **Удалить группу ресурсов**.

1. Введите имя группы ресурсов. Теперь щелкните **Удалить**.

## <a name="example-notebooks"></a>Примеры записных книжек

В примере использования [MLflow с записными книжками Машинного обучения Azure](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/track-and-monitor-experiments/using-mlflow) демонстрируются и поясняются основные понятия, представленные в этой статье.

> [!NOTE]
> Основанный на сообществе репозиторий примеров с использованием млфлов можно найти по адресу https://github.com/Azure/azureml-examples .

## <a name="next-steps"></a>Дальнейшие действия

* [Развертывание моделей с помощью млфлов](how-to-deploy-mlflow-models.md).
* Отслеживайте рабочие модели для выявления [смещения данных](./how-to-enable-data-collection.md).
* [Track Azure Databricks выполняется с млфлов](how-to-use-mlflow-azure-databricks.md).
* [Управляйте своими моделями](concept-model-management-and-deployment.md).