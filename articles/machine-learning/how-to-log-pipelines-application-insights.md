---
title: Мониторинг & сбора файлов журналов конвейера
titleSuffix: Azure Machine Learning
description: Добавьте ведение журнала в конвейеры обучения и пакетной оценки и просмотрите записанные результаты в Application Insights.
services: machine-learning
author: NilsPohlmann
ms.author: nilsp
ms.service: machine-learning
ms.subservice: core
ms.date: 08/11/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 7ed27eba66b3d18bed8017934fce85928b961392
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102520053"
---
# <a name="collect-machine-learning-pipeline-log-files-in-application-insights-for-alerts-and-debugging"></a>Получение файлов журнала конвейера машинного обучения в Application Insights для предупреждений и отладки


Библиотеку [опенценсус](https://opencensus.io/quickstart/python/) Python можно использовать для маршрутизации журналов в Application Insights из скриптов. Объединение журналов из конвейеров в одном месте позволяет создавать запросы и диагностировать проблемы. Использование Application Insights позволяет вести мониторинг журналов и сравнивать журналы конвейера во время выполнения.

Если журналы будут находиться в однократном месте, будет представлена история исключений и сообщений об ошибках. Поскольку Application Insights интегрируется с оповещениями Azure, можно также создавать оповещения на основе запросов Application Insights.

## <a name="prerequisites"></a>Предварительные условия

* Выполните действия, чтобы создать рабочую область [машинное обучение Azure](./how-to-manage-workspace.md) и [создать первый конвейер](./how-to-create-machine-learning-pipelines.md) .
* [Настройте среду разработки](./how-to-configure-environment.md) для установки пакета SDK для Машинного обучения Azure.
* Установите пакет [опенценсус Azure Monitor Export](https://pypi.org/project/opencensus-ext-azure/) локально:
  ```python
  pip install opencensus-ext-azure
  ```
* Создание [экземпляра Application Insights](../azure-monitor/app/opencensus-python.md) (этот документ также содержит сведения о получении строки подключения для ресурса).

## <a name="getting-started"></a>Приступая к работе

В этом разделе представлены общие сведения об использовании Опенценсус из конвейера Машинное обучение Azure. Подробное руководство см. в разделе [опенценсус Azure Monitor EXPORTS](https://github.com/census-instrumentation/opencensus-python/tree/master/contrib/opencensus-ext-azure) .

Добавьте Писонскриптстеп в конвейер машинного обучения Azure. Настройте [RunConfiguration](/python/api/azureml-core/azureml.core.runconfiguration) с зависимостью от опенценсус-ext-Azure. Настройте `APPLICATIONINSIGHTS_CONNECTION_STRING` переменную среды.

```python
from azureml.core.conda_dependencies import CondaDependencies
from azureml.core.runconfig import RunConfiguration
from azureml.pipeline.core import Pipeline
from azureml.pipeline.steps import PythonScriptStep

# Connecting to the workspace and compute target not shown

# Add pip dependency on OpenCensus
dependencies = CondaDependencies()
dependencies.add_pip_package("opencensus-ext-azure>=1.0.1")
run_config = RunConfiguration(conda_dependencies=dependencies)

# Add environment variable with Application Insights Connection String
# Replace the value with your own connection string
run_config.environment.environment_variables = {
    "APPLICATIONINSIGHTS_CONNECTION_STRING": 'InstrumentationKey=00000000-0000-0000-0000-000000000000'
}

# Configure step with runconfig
sample_step = PythonScriptStep(
        script_name="sample_step.py",
        compute_target=compute_target,
        runconfig=run_config
)

# Submit new pipeline run
pipeline = Pipeline(workspace=ws, steps=[sample_step])
pipeline.submit(experiment_name="Logging_Experiment")
```

Создайте файл с именем `sample_step.py`. Импортируйте класс Азурелогхандлер, чтобы направить журналы в Application Insights. Также необходимо импортировать библиотеку ведения журнала Python.

```python
from opencensus.ext.azure.log_exporter import AzureLogHandler
import logging
```

Затем добавьте Азурелогхандлер в средство ведения журнала Python.

```python
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)
logger.addHandler(logging.StreamHandler())

# Assumes the environment variable APPLICATIONINSIGHTS_CONNECTION_STRING is already set
logger.addHandler(AzureLogHandler())
logger.warning("I will be sent to Application Insights")
```

## <a name="logging-with-custom-dimensions"></a>Ведение журнала с пользовательскими измерениями
 
По умолчанию журналы, пересылаемые в Application Insights, не имеют достаточного контекста для трассировки выполнения или эксперимента. Чтобы журналы были доступны для диагностики проблем, требуются дополнительные поля. 

Чтобы добавить эти поля, можно добавить пользовательские измерения, чтобы предоставить контекст для сообщения журнала. Одним из примеров является то, что кто-то хочет просматривать журналы в течение нескольких шагов в одном и том же запуске конвейера.

Пользовательские измерения составляют словарь пар "ключ-значение" (хранимых как строковые, строковые). Затем словарь отправляется в Application Insights и отображается как столбец в результатах запроса. Его отдельные измерения можно использовать в качестве [параметров запроса](#additional-helpful-queries).

### <a name="helpful-context-to-include"></a>Полезный контекст для включения

| Поле                          | Причины и примеры                                                                                                                                                                       |
|--------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| parent_run_id                  | Может запрашивать журналы для тех же parent_run_id, чтобы просматривать журналы с течением времени для всех шагов, вместо того чтобы углубляться в каждый отдельный шаг.                                        |
| step_id                        | Может запрашивать журналы для с тем же step_id, чтобы узнать, где возникла ошибка, а не только на отдельном шаге.                                                        |
| step_name                      | Может запрашивать журналы для просмотра производительности по времени. Также помогает найти step_id для последних запусков без погружения в пользовательский интерфейс портала                                          |
| experiment_name                | Может выполнять запросы к журналам для просмотра производительности экспериментов с течением времени. Также помогает найти parent_run_id или step_id для последних запусков без погружения в пользовательский интерфейс портала                   |
| run_url                 | Может напрямую предоставить ссылку на запуск для расследования. |

**Другие полезные поля**

Эти поля могут потребовать дополнительного инструментирования кода и не предоставляются контекстом выполнения.

| Поле                   | Причины и примеры                                                                                                                                                                                                           |
|-------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| build_url и build_version | При использовании CI/CD для развертывания это поле может сопоставлять журналы с версией кода, которая предоставляет логику этапа и конвейера. Эта ссылка может помочь в диагностике проблем или выявлять модели с конкретными характеристиками (значения журнала/метрики). |
| run_type                       | Может различаться между различными типами моделей, а также с циклами обучения и оценки |

### <a name="creating-a-custom-dimensions-dictionary"></a>Создание словаря пользовательских измерений

```python
from azureml.core import Run

run = Run.get_context(allow_offline=False)

custom_dimensions = {
    "parent_run_id": run.parent.id,
    "step_id": run.id,
    "step_name": run.name,
    "experiment_name": run.experiment.name,
    "run_url": run.parent.get_portal_url(),
    "run_type": "training"
}

# Assumes AzureLogHandler was already registered above
logger.info("I will be sent to Application Insights with Custom Dimensions", extra= {"custom_dimensions":custom_dimensions})
```

## <a name="opencensus-python-logging-considerations"></a>Вопросы ведения журнала Опенценсус Python

Опенценсус Азурелогхандлер используется для перенаправления журналов Python в Application Insights. В результате следует учитывать особенности ведения журнала Python. При создании средства ведения журнала оно имеет уровень ведения журнала по умолчанию и отображает журналы, которые больше или равны этому уровню. Хорошим справочником по использованию возможностей ведения журнала Python является [Cookbook ведения журнала](https://docs.python.org/3/howto/logging-cookbook.html).

`APPLICATIONINSIGHTS_CONNECTION_STRING`Для библиотеки опенценсус требуется переменная среды. Рекомендуется задавать эту переменную среды вместо передачи ее в качестве параметра конвейера, чтобы избежать передачи строк соединения в виде открытого текста.

## <a name="querying-logs-in-application-insights"></a>Запрос журналов в Application Insights

Журналы, направляемые в Application Insights, будут отображаться в разделе "трассировки" или "исключения". Не забудьте настроить временное окно, включив в него выполнение конвейера.

![Результат запроса Application Insights](./media/how-to-debug-pipelines-application-insights/traces-application-insights-query.png)

В результате в Application Insights будут показаны сообщение журнала и уровень, путь к файлу и номер строки кода. В нем также будут показаны все пользовательские измерения. На этом рисунке в словаре customDimensions показаны пары "ключ-значение" из предыдущего [примера кода](#creating-a-custom-dimensions-dictionary).

### <a name="additional-helpful-queries"></a>Дополнительные полезные запросы

В некоторых запросах ниже используется "customDimensions. Level". Эти уровни серьезности соответствуют уровню, с которым был первоначально отправлен журнал Python. Дополнительные сведения о запросах см. в разделе [Azure Monitor запросы журналов](/azure/data-explorer/kusto/query/).

| Вариант использования                                                               | Запрос                                                                                              |
|------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|
| Заносить в журнал результаты для конкретного пользовательского измерения, например "parent_run_id" | <pre>traces \| <br>where customDimensions.parent_run_id == '931024c2-3720-11ea-b247-c49deda841c1</pre> |
| Записывать результаты всех обучающих запусков за последние семь дней                     | <pre>traces \| <br>where timestamp > ago(7d) <br>and customDimensions.run_type == 'training'</pre>           |
| Регистрировать результаты с ошибкой Северитилевел за последние семь дней              | <pre>traces \| <br>where timestamp > ago(7d) <br>and customDimensions.Level == 'ERROR'                     |
| Число результатов журнала с Северитилевел ошибкой за последние семь дней     | <pre>traces \| <br>where timestamp > ago(7d) <br>and customDimensions.Level == 'ERROR' \| <br>summarize count()</pre> |

## <a name="next-steps"></a>Дальнейшие действия

После получения журналов в экземпляре Application Insights их можно использовать для установки [предупреждений Azure Monitor](../azure-monitor/alerts/alerts-overview.md#what-you-can-alert-on) на основе результатов запроса.

Вы также можете добавлять результаты из запросов на [панель мониторинга Azure](../azure-monitor/app/tutorial-app-dashboards.md#add-logs-query) для получения дополнительных сведений.