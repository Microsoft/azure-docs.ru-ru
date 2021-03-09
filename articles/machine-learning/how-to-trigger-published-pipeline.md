---
title: Триггеры Машинное обучение Azure конвейеры
titleSuffix: Azure Machine Learning
description: Активированные Конвейеры позволяют автоматизировать подпрограммы, требующие много времени, такие как обработка данных, обучение и мониторинг.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: laobri
author: lobrien
ms.date: 01/29/2021
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 3ecf4458b052f4fdc0eb2e6e697b0468c71ce9c2
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102519662"
---
# <a name="trigger-machine-learning-pipelines"></a>Активация конвейеров машинного обучения

В этой статье вы узнаете, как программным образом запланировать запуск конвейера в Azure. Расписание можно создать на основе истекшего времени или изменения файловой системы. Расписания на основе времени можно использовать для выполнения регулярных задач, таких как мониторинг смещения данных. Расписания на основе изменений можно использовать для реагирования на нестандартные или непредсказуемые изменения, такие как отправка новых данных или изменение старых данных. После изучения способов создания расписаний вы узнаете, как извлекать и деактивировать их. Наконец, вы узнаете, как использовать другие службы Azure, приложение логики Azure и фабрику данных Azure для выполнения конвейеров. Приложение логики Azure обеспечивает более сложную логику или поведение. Конвейеры фабрики данных Azure позволяют вызывать конвейер машинного обучения как часть более крупного конвейера оркестрации данных.

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. Если у вас нет подписки Azure, создайте [бесплатную учетную запись](https://aka.ms/AMLFree).

* Среда Python, в которой установлен пакет SDK для Машинное обучение Azure для Python. Дополнительные сведения см [. в статьях создание повторно используемых сред для обучения и развертывания с помощью машинное обучение Azure и управление ими.](how-to-use-environments.md)

* Рабочая область Машинное обучение с опубликованным конвейером. Вы можете использовать встроенную [программу создания и запуска конвейеров машинного обучения с помощью пакета SDK для машинное обучение Azure](./how-to-create-machine-learning-pipelines.md).

## <a name="trigger-pipelines-with-azure-machine-learning-sdk-for-python"></a>Активация конвейеров с помощью пакета SDK для Машинное обучение Azure для Python

Чтобы запланировать конвейер, вам потребуется ссылка на рабочую область, идентификатор опубликованного конвейера и имя эксперимента, в котором вы хотите создать расписание. Эти значения можно получить с помощью следующего кода:

```Python
import azureml.core
from azureml.core import Workspace
from azureml.pipeline.core import Pipeline, PublishedPipeline
from azureml.core.experiment import Experiment

ws = Workspace.from_config()

experiments = Experiment.list(ws)
for experiment in experiments:
    print(experiment.name)

published_pipelines = PublishedPipeline.list(ws)
for published_pipeline in  published_pipelines:
    print(f"{published_pipeline.name},'{published_pipeline.id}'")

experiment_name = "MyExperiment" 
pipeline_id = "aaaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee" 
```

## <a name="create-a-schedule"></a>Создание расписания

Чтобы выполнить конвейер на повторяющейся основе, создайте расписание. Объект `Schedule` связывает конвейер, эксперимент и триггер. Триггер может иметь значение `ScheduleRecurrence` , описывающее время ожидания между запусками или путь к хранилищу данных, указывающий каталог для отслеживания изменений. В любом случае вам потребуется идентификатор конвейера и имя эксперимента, в котором создается расписание.

В верхней части файла Python импортируйте `Schedule` `ScheduleRecurrence` классы и:

```python

from azureml.pipeline.core.schedule import ScheduleRecurrence, Schedule
```

### <a name="create-a-time-based-schedule"></a>Создание расписания на основе времени

`ScheduleRecurrence`Конструктор имеет обязательный `frequency` аргумент, который должен быть одной из следующих строк: "Minute", "Hour", "Day", "Week" или "month". Кроме того, требуется целочисленный `interval` аргумент, указывающий, сколько `frequency` единиц должно пройти между запусками расписания. Необязательные аргументы позволяют более подробно узнать о времени запуска, как описано в [документации по пакету SDK для счедулерекурренце](/python/api/azureml-pipeline-core/azureml.pipeline.core.schedule.schedulerecurrence).

Создайте `Schedule` , который начинает выполнение каждые 15 минут:

```python
recurrence = ScheduleRecurrence(frequency="Minute", interval=15)
recurring_schedule = Schedule.create(ws, name="MyRecurringSchedule", 
                            description="Based on time",
                            pipeline_id=pipeline_id, 
                            experiment_name=experiment_name, 
                            recurrence=recurrence)
```

### <a name="create-a-change-based-schedule"></a>Создание расписания на основе изменений

Конвейеры, активируемые изменениями файлов, могут оказаться более эффективными по сравнению с расписаниями, основанными на времени. Если нужно сделать что-то перед изменением файла или при добавлении нового файла в каталог данных, можно предварительно обработать этот файл. Вы можете отслеживать любые изменения в хранилище данных или изменения в определенном каталоге в хранилище данных. При мониторинге определенного каталога изменения в подкаталогах этого каталога _не_ активируют запуск.

Для создания реактивного файла `Schedule` необходимо задать `datastore` параметр в вызове [Schedule. Create](/python/api/azureml-pipeline-core/azureml.pipeline.core.schedule.schedule#create-workspace--name--pipeline-id--experiment-name--recurrence-none--description-none--pipeline-parameters-none--wait-for-provisioning-false--wait-timeout-3600--datastore-none--polling-interval-5--data-path-parameter-name-none--continue-on-step-failure-none--path-on-datastore-none---workflow-provider-none---service-endpoint-none-). Для наблюдения за папкой задайте `path_on_datastore` аргумент.

`polling_interval`Аргумент позволяет указать в минутах частоту проверки хранилища данных на наличие изменений.

Если конвейер был создан с помощью [пути](/python/api/azureml-core/azureml.data.datapath.datapath) [пипелинепараметер](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelineparameter), можно присвоить этой переменной имя измененного файла, задав `data_path_parameter_name` аргумент.

```python
datastore = Datastore(workspace=ws, name="workspaceblobstore")

reactive_schedule = Schedule.create(ws, name="MyReactiveSchedule", description="Based on input file change.",
                            pipeline_id=pipeline_id, experiment_name=experiment_name, datastore=datastore, data_path_parameter_name="input_data")
```

### <a name="optional-arguments-when-creating-a-schedule"></a>Необязательные аргументы при создании расписания

В дополнение к описанным выше аргументам можно задать для `status` аргумента значение, `"Disabled"` чтобы создать неактивное расписание. Наконец, `continue_on_step_failure` позволяет передать логическое значение, которое будет переопределять поведение по умолчанию при сбоях конвейера.

## <a name="view-your-scheduled-pipelines"></a>Просмотр запланированных конвейеров

В веб-браузере перейдите по адресу Машинное обучение Azure. В разделе **конечные точки** панели навигации выберите пункт **конечные точки конвейера**. Откроется список конвейеров, опубликованных в рабочей области.

:::image type="content" source="./media/how-to-trigger-published-pipeline/scheduled-pipelines.png" alt-text="Страница конвейеров AML":::

На этой странице можно просмотреть сводную информацию обо всех конвейерах в рабочей области: имена, описания, состояние и т. д. Выполните детализацию, щелкнув конвейер. На полученной странице имеются дополнительные сведения о конвейере, и вы можете выполнить детализацию для отдельных запусков.

## <a name="deactivate-the-pipeline"></a>Деактивация конвейера

Если у вас есть `Pipeline` Публикация, но она не запланирована, ее можно отключить с помощью:

```python
pipeline = PublishedPipeline.get(ws, id=pipeline_id)
pipeline.disable()
```

Если расписание конвейера запланировано, сначала необходимо отменить его. Получите идентификатор расписания на портале или запустив:

```python
ss = Schedule.list(ws)
for s in ss:
    print(s)
```

После того, как вы `schedule_id` хотите отключить, выполните:

```python
def stop_by_schedule_id(ws, schedule_id):
    s = next(s for s in Schedule.list(ws) if s.id == schedule_id)
    s.disable()
    return s

stop_by_schedule_id(ws, schedule_id)
```

При `Schedule.list(ws)` повторном запуске необходимо получить пустой список.

## <a name="use-azure-logic-apps-for-complex-triggers"></a>Использование Azure Logic Apps для сложных триггеров 

Более сложные правила триггера или поведение можно создать с помощью [приложения логики Azure](../logic-apps/logic-apps-overview.md).

Чтобы использовать приложение логики Azure для активации конвейера Машинное обучение, требуется конечная точка RESTFUL для опубликованного конвейера Машинное обучение. [Создайте и опубликуйте свой конвейер](./how-to-create-machine-learning-pipelines.md). Затем найдите конечную точку RESTFUL `PublishedPipeline` , используя идентификатор конвейера:

```python
# You can find the pipeline ID in Azure Machine Learning studio

published_pipeline = PublishedPipeline.get(ws, id="<pipeline-id-here>")
published_pipeline.endpoint 
```

## <a name="create-a-logic-app"></a>Создание приложения логики

Теперь создайте экземпляр [приложения логики Azure](../logic-apps/logic-apps-overview.md) . При желании [Используйте среду службы интеграции (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment.md) и [Настройте управляемый клиентом ключ](../logic-apps/customer-managed-keys-integration-service-environment.md) для использования приложением логики.

После подготовки приложения логики выполните следующие действия, чтобы настроить триггер для конвейера.

1. Чтобы предоставить приложению доступ к рабочая область машинного обучения Azure, [Создайте управляемое удостоверение, назначенное системой](../logic-apps/create-managed-service-identity.md) .

1. Перейдите в представление конструктора приложений логики и выберите шаблон Пустое приложение логики. 
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="media/how-to-trigger-published-pipeline/blank-template.png" alt-text="пустой шаблон.":::

1. В конструкторе найдите BLOB- **объект**. Выберите **при добавлении или изменении большого двоичного объекта (только свойства)** и добавьте этот триггер в приложение логики.
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="media/how-to-trigger-published-pipeline/add-trigger.png" alt-text="Добавление триггера":::

1. Укажите сведения о подключении для учетной записи хранения BLOB-объектов, которую вы хотите отслеживать для добавления или изменения больших двоичных объектов. Выберите контейнер для отслеживания. 
 
    Выберите **интервал** и **частоту** опроса обновлений, которые работают за вас.  

    > [!NOTE]
    > Этот триггер будет отслеживать выбранный контейнер, но не будет отслеживать вложенные папки.

1. Добавьте действие HTTP, которое будет выполняться при обнаружении нового или измененного большого двоичного объекта. Выберите **+ новый шаг**, затем найдите и выберите действие HTTP.

  > [!div class="mx-imgBorder"]
  > :::image type="content" source="media/how-to-trigger-published-pipeline/search-http.png" alt-text="Поиск действия HTTP":::

  Для настройки действия используйте следующие параметры:

  | Параметр | Значение | 
  |---|---|
  | Действие HTTP | POST |
  | URI |Конечная точка для опубликованного конвейера, которую вы нашли в качестве [необходимого компонента](#prerequisites) |
  | Режим проверки подлинности | Управляемое удостоверение |

1. Настройте расписание, чтобы задать значение любого [пути PipelineParameters](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines/intro-to-pipelines/aml-pipelines-showcasing-datapath-and-pipelineparameter.ipynb) , который вы имеете в наличии:

    ```json
    {
      "DataPathAssignments": {
        "input_datapath": {
          "DataStoreName": "<datastore-name>",
          "RelativePath": "@{triggerBody()?['Name']}" 
        }
      },
      "ExperimentName": "MyRestPipeline",
      "ParameterAssignments": {
        "input_string": "sample_string3"
      },
      "RunSource": "SDK"
    }
    ```

    Используйте `DataStoreName` добавленные в рабочую область в качестве [необходимого компонента](#prerequisites).
     
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="media/how-to-trigger-published-pipeline/http-settings.png" alt-text="Параметры HTTP":::

1. Выберите **сохранить** , и ваше расписание теперь готово.

> [!IMPORTANT]
> Если вы используете управление доступом на основе ролей Azure (Azure RBAC) для управления доступом к конвейеру, [Задайте разрешения для сценария конвейера (обучение или оценка)](how-to-assign-roles.md#common-scenarios).

## <a name="call-machine-learning-pipelines-from-azure-data-factory-pipelines"></a>Вызов конвейеров машинного обучения из конвейеров фабрики данных Azure

В конвейере фабрики данных Azure действие *машинное обучение выполнение конвейера* выполняет конвейер машинное обучение Azure. Это действие можно найти на странице создания фабрики данных в категории *машинное обучение* .

:::image type="content" source="media/how-to-trigger-published-pipeline/azure-data-factory-pipeline-activity.png" alt-text="Снимок экрана, показывающий действие конвейера ML в среде разработки фабрики данных Azure":::

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы использовали пакет SDK для Машинное обучение Azure для Python, чтобы запланировать конвейер двумя разными способами. Одно расписание повторяется на основе затраченного времени. Другое расписание выполняется при изменении файла на указанном `Datastore` или в каталоге в этом хранилище. Вы узнали, как использовать портал для проверки конвейера и отдельных запусков. Вы узнали, как отключить расписание, чтобы остановить выполнение конвейера. Наконец, вы создали приложение логики Azure для активации конвейера. 

Дополнительные сведения см. в разделе:

> [!div class="nextstepaction"]
> [Использование конвейеров Машинного обучения Azure для пакетной оценки](tutorial-pipeline-batch-scoring-classification.md)

* Дополнительные сведения о [конвейерах](concept-ml-pipelines.md)
* Дополнительные сведения об [исследовании машинное обучение Azure с помощью Jupyter](samples-notebooks.md)