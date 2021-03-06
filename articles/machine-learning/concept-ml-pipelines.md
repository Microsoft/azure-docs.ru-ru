---
title: Что такое конвейеры машинного обучения
titleSuffix: Azure Machine Learning
description: Узнайте, как конвейеры машинного обучения помогают создавать, оптимизировать и администрировать рабочие процессы машинного обучения.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: laobri
author: lobrien
ms.date: 02/26/2021
ms.custom: devx-track-python
ms.openlocfilehash: 584e421b6beac0e4ecfab5b3e3cb735b8465e1b4
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102503527"
---
# <a name="what-are-azure-machine-learning-pipelines"></a>Что такое Машинное обучение Azure конвейеров?

В этой статье вы узнаете, как конвейер машинного обучения помогает создавать, оптимизировать и администрировать рабочий процесс машинного обучения. 

<a name="compare"></a>
## <a name="which-azure-pipeline-technology-should-i-use"></a>Какую технологию конвейера Azure следует использовать?

Облако Azure предоставляет несколько типов конвейера, каждый из которых имеет другую цель. В следующей таблице перечислены различные конвейеры и их использование.

| Сценарий | Первичный персонаж | Предложение Azure | Предложение OSS | Канонический канал | Преимущества | 
| -------- | --------------- | -------------- | ------------ | -------------- | --------- | 
| Оркестрации модели (машинное обучение) | Специалист по обработке и анализу данных | Конвейеры Машинного обучение Azure | Конвейеры Kubeflow | Модель > данных | Распространение, кэширование, сначала код, повторное использование | 
| Оркестрации данных (подготовка данных) | Инженер данных | [Конвейеры Фабрики данных Azure](../data-factory/concepts-pipelines-activities.md) | Apache Airflow | Данные > данных | Строго типизированное перемещение, действия, ориентированные на данные |
| Согласование кода & приложений (CI/CD) | Разработка и Ops приложения | [Azure Pipelines](https://azure.microsoft.com/services/devops/pipelines/) | Jenkins | Код + модель — > приложения или службы | Поддержка наиболее открытых и гибких действий, очереди утверждений, фазы с ограничением | 

## <a name="what-can-machine-learning-pipelines-do"></a>Что могут выполнять конвейеры машинного обучения?

Конвейер Машинное обучение Azure — это независимый исполняемый рабочий процесс полной задачи машинного обучения. Подзадачи инкапсулируются в последовательность шагов в конвейере. Конвейер Машинное обучение Azure может быть таким же простым, как и тот, который вызывает сценарий Python, поэтому _может_ выполнять практически любые действия. Конвейеры _должны_ сосредоточиться на задачах машинного обучения, таких как:

+ подготовка данных, включая импорт, проверку, очистку, изменение, преобразование, нормализацию и промежуточное хранение;
+ настройка обучения, включая параметризацию аргументов, путей, а также конфигураций ведения журналов и отчетности;
+ Обучение и проверка эффективно и многократно. Эффективность может полагаться на указание конкретных поднаборов данных, различных вычислительных ресурсов оборудования, распределенной обработки и мониторинга хода выполнения.
+ развертывание, включая управление версиями, масштабирование, подготовку и управление доступом;

Независимые шаги позволяют нескольким специалистам по обработке данных работать в одном конвейере одновременно без чрезмерных ресурсов вычислений. Отдельные шаги также упрощают использование различных типов вычислений и размеров для каждого шага.

После создания конвейера часто нужно выполнить более тонкую настройку его цикла обучения. При повторном запуске конвейера выполняется переход к шагам, которые необходимо повторить, например обновленному сценарию обучения. Шаги, которые не нужно повторять, пропускаются. 

С помощью конвейеров вы можете использовать разные аппаратные средства для различных задач. Azure координирует различные используемые [целевые объекты вычислений](concept-azure-machine-learning-architecture.md) , поэтому промежуточные данные легко передаются в подчиненные целевые объекты вычислений.

[Метрики для экспериментов конвейера можно отслеживаниь](./how-to-track-experiments.md) непосредственно в портал Azure или на [целевой странице рабочей области (Предварительная версия)](https://ml.azure.com). После публикации конвейера можно настроить конечную точку RESTFUL, которая позволит перезапустить конвейер из любой платформы или стека.

Вкратце, все сложные задачи жизненного цикла машинного обучения можно использовать с конвейерами. Другие технологии конвейера Azure обладают собственными преимуществами. [Конвейеры фабрики данных Azure](../data-factory/concepts-pipelines-activities.md) предназначен в работе с данными и [Azure pipelines](https://azure.microsoft.com/services/devops/pipelines/) — это подходящий инструмент для непрерывной интеграции и развертывания. Но если ваше внимание уделяется машинному обучению, Машинное обучение Azure конвейеры, скорее всего, будут лучшим выбором для нужд рабочего процесса. 

### <a name="analyzing-dependencies"></a>Анализ зависимостей

Многие экосистемы программирования имеют средства для управления зависимостями ресурсов, библиотек или компиляции. Как правило, эти средства используют метки времени файлов для вычисления зависимостей. При изменении файла только он и его зависимые элементы обновляются (скачаны, перекомпилированы или упакованы). Машинное обучение Azure конвейеры расширяют эту концепцию. Как и традиционные средства сборки, конвейеры вычисляют зависимости между шагами и выполняют только необходимые повторные вычисления. 

Анализ зависимостей в конвейерах Машинное обучение Azure является более сложным, чем простые метки времени. Каждый шаг может выполняться в различных аппаратных и программных средах. Подготовка данных может быть трудоемким процессом, но не нужно работать на оборудовании с мощными графическими процессорами. для некоторых шагов может потребоваться программное обеспечение, предназначенное для конкретной операционной системы, можно использовать распределенное обучение и т. д. 

Машинное обучение Azure автоматически управляет всеми зависимостями между этапами конвейера. Это может привести к ускорению и понижению работы образов DOCKER, подключению и отсоединению вычислений и перемещению данных между действиями в единообразном и автоматическом режиме.

### <a name="coordinating-the-steps-involved"></a>Координация необходимых действий

При создании и запуске `Pipeline` объекта выполняются следующие общие действия.

+ Для каждого шага служба вычисляет требования для:
    + Аппаратные ресурсы для вычислений
    + Ресурсы ОС (образы DOCKER)
    + Ресурсы программного обеспечения (зависимости Conda и Virtualenv)
    + Входные данные 
+ Служба определяет зависимости между шагами, что приводит к динамическому графу выполнения
+ При выполнении каждого узла в графе выполнения:
    + Служба настраивает необходимую аппаратную и программную среду (возможно, повторное использование существующих ресурсов).
    + Этот шаг выполняется, предоставляя сведения о ведении журнала и мониторинге содержащего его `Experiment` объекта.
    + После завершения шага его выходные данные подготавливаются в качестве входных данных для следующего шага и (или) записи в хранилище.
    + Ресурсы, которые больше не нужны, завершаются и отсоединяются

![Шаги конвейера](./media/concept-ml-pipelines/run_an_experiment_as_a_pipeline.png)

## <a name="building-pipelines-with-the-python-sdk"></a>Создание конвейеров с помощью пакета SDK для Python

В [пакете SDK для машинное обучение Azure Python](/python/api/overview/azure/ml/install)конвейер — это объект Python, определенный в `azureml.pipeline.core` модуле. Объект [конвейера](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipeline%28class%29) содержит упорядоченную последовательность из одного или нескольких объектов [пипелинестеп](/python/api/azureml-pipeline-core/azureml.pipeline.core.builder.pipelinestep) . `PipelineStep`Класс является абстрактным, и фактические шаги будут подклассами, такими как [естиматорстеп](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.estimatorstep), [писонскриптстеп](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.pythonscriptstep)или [дататрансферстеп](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.datatransferstep). Класс [модулестеп](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.modulestep) содержит многократно используемую последовательность действий, которые могут совместно использоваться в конвейерах. `Pipeline`Выполняется как часть `Experiment` .

Конвейер машинного обучения Azure связан с рабочей областью Машинное обучение Azure, а этап конвейера связан с целевым объектом вычислений, доступным в этой рабочей области. Дополнительные сведения см. в статьях [создание рабочих областей машинное обучение Azure и управление ими в портал Azure](./how-to-manage-workspace.md) или [целевые объекты вычислений в машинное обучение Azure?](./concept-compute-target.md).

### <a name="a-simple-python-pipeline"></a>Простой конвейер Python

В этом фрагменте кода показаны объекты и вызовы, необходимые для создания и выполнения `Pipeline` :

```python
ws = Workspace.from_config() 
blob_store = Datastore(ws, "workspaceblobstore")
compute_target = ws.compute_targets["STANDARD_NC6"]
experiment = Experiment(ws, 'MyExperiment') 

input_data = Dataset.File.from_files(
    DataPath(datastore, '20newsgroups/20news.pkl'))
prepped_data_path = OutputFileDatasetConfig(name="output_path")

dataprep_step = PythonScriptStep(
    name="prep_data",
    script_name="dataprep.py",
    source_directory="prep_src",
    compute_target=compute_target,
    arguments=["--prepped_data_path", prepped_data_path],
    inputs=[input_dataset.as_named_input('raw_data').as_mount() ]
    )

prepped_data = prepped_data_path.read_delimited_files()

train_step = PythonScriptStep(
    name="train",
    script_name="train.py",
    compute_target=compute_target,
    arguments=["--prepped_data", prepped_data],
    source_directory="train_src"
)
steps = [ dataprep_step, train_step ]

pipeline = Pipeline(workspace=ws, steps=steps)

pipeline_run = experiment.submit(pipeline)
pipeline_run.wait_for_completion()
```

Фрагмент кода начинается с общих Машинное обучение Azure объектов,, a, `Workspace` `Datastore` [ComputeTarget](/python/api/azureml-core/azureml.core.computetarget)и `Experiment` . Затем код создает объекты для хранения `input_data` и `prepped_data_path` . `input_data`Является экземпляром [филедатасет](/python/api/azureml-core/azureml.data.filedataset) , а `prepped_data_path` — экземпляром [аутпутфиледатасетконфиг](/python/api/azureml-core/azureml.data.output_dataset_config.outputfiledatasetconfig). Для `OutputFileDatasetConfig` поведения по умолчанию необходимо скопировать выходные данные в хранилище данных по `workspaceblobstore` пути `/dataset/{run-id}/{output-name}` , где `run-id` — идентификатор выполнения, а `output-name` — автоматически сформированное значение, если оно не указано разработчиком.

Код подготовки данных (не показан) записывает файлы с разделителями в `prepped_data_path` . Эти выходные данные этапа подготовки данных передаются в качестве `prepped_data` этапа обучения. 

Массив `steps` содержит два массива `PythonScriptStep` : `dataprep_step` и `train_step` . Машинное обучение Azure проанализирует зависимость данных и будет `prepped_data` выполняться `dataprep_step` раньше `train_step` . 

Затем код создает экземпляр `Pipeline` самого объекта, передавая в него рабочую область и массив шагов. Вызов начинается с `experiment.submit(pipeline)` запуска конвейера машинного обучения Azure. Вызов `wait_for_completion()` блокируется до завершения конвейера. 

Дополнительные сведения о подключении конвейера к данным см. в статьях [доступ к данным в машинное обучение Azure](concept-data.md) и [Перемещение данных между этапами конвейера ml (Python)](how-to-move-data-in-out-of-pipelines.md). 

## <a name="building-pipelines-with-the-designer"></a>Создание конвейеров с помощью конструктора

Разработчики, предпочитающие визуальную область конструктора, могут использовать конструктор Машинное обучение Azure для создания конвейеров. Доступ к этому средству можно получить с помощью выбора **конструктора** на домашней странице рабочей области.  Конструктор позволяет перетаскивать шаги в область конструктора. 

При визуальном проектировании конвейеров входные и выходные данные шага отображаются как видимые. Можно перетаскивать подключения к данным, что позволяет быстро понять и изменить поток данных конвейера.

![Пример конструктора Машинного обучения Azure](./media/concept-designer/designer-drag-and-drop.gif)

## <a name="key-advantages"></a>Ключевые преимущества

Основные преимущества использования конвейеров для рабочих процессов машинного обучения:

|Ключевое преимущество|Описание|
|:-------:|-----------|
|**Автоматические&nbsp;выполнения**|Планирование выполнения шагов параллельно или в последовательности в надежном и автоматическом режиме. Подготовка и моделирование данных могут быть за последние дни или недели, а Конвейеры позволяют сосредоточиться на других задачах во время выполнения процесса. |
|**Разнородных вычислений**|Используйте несколько конвейеров, которые надежно скоординированы между разнородными и масштабируемыми вычислительными ресурсами и местами хранения. Эффективно используйте доступные вычислительные ресурсы, выполняя отдельные этапы конвейера для различных целевых объектов вычислений, таких как HDInsight, виртуальные машины обработки и анализа данных GPU, а также кирпичы.|
|**Возможность многократного использования**|Создание шаблонов конвейера для конкретных сценариев, таких как переобучение и пакетная оценка. Активация опубликованных конвейеров из внешних систем с помощью простых вызовов RESTFUL.|
|**Отслеживание и управление версиями**|Вместо отслеживания данных и папок результатов вручную при выполнении итераций используйте пакет SDK для конвейеров, чтобы присваивать имена и определять версии источников данных, входных и выходных данных. Вы можете также отдельно управлять сценариями и данными для повышения эффективности.|
| **Модульность** | Отделение областей проблем и изоляция изменений позволяет программному обеспечению развиваться быстрее с более высоким качеством. | 
|**Совместная работа**|Конвейеры позволяют специалистам по обработке и анализу данных совместно работать во всех областях процесса разработки машинного обучения, в то же время одновременно работая с этапами конвейера.|

## <a name="next-steps"></a>Дальнейшие действия

Машинное обучение Azure конвейеры — это мощное средство, которое начинает доставку ценности на стадии раннего развертывания. Значение увеличивается по мере роста группы и проекта. В этой статье объясняется, как указать конвейеры с помощью пакета SDK для Машинное обучение Azure Python и в Azure. Вы увидели простой исходный код и предоставили несколько `PipelineStep` доступных классов. Вы должны иметь представление о том, когда использовать конвейеры Машинное обучение Azure и как они работают в Azure. 

+ Узнайте, как [создать свой первый конвейер](./how-to-create-machine-learning-pipelines.md).

+ Узнайте, как [выполнять пакетные прогнозы для больших данных](tutorial-pipeline-batch-scoring-classification.md ).

+ См. справочную документацию по пакету SDK и [этапам](/python/api/azureml-pipeline-steps/) [конвейера](/python/api/azureml-pipeline-core/) .

+ Ознакомьтесь с примерами записных книжек Jupyter, демонстрирующими [машинное обучение Azure конвейеры](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines). Узнайте, как [запускать записные книжки для изучения этой службы](samples-notebooks.md).