---
title: Анализ & объяснения моделей машинного обучения в Python (Предварительная версия)
titleSuffix: Azure Machine Learning
description: Узнайте, как получить объяснения о том, как модель машинного обучения определяет важность функций и делает прогнозы при использовании пакета SDK для Машинное обучение Azure.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: mithigpe
author: minthigpen
ms.reviewer: Luis.Quintanilla
ms.date: 07/09/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, responsible-ml
ms.openlocfilehash: fda1bc2ef0a112a8a32ba7c4caebf29028c8cdd7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98222757"
---
# <a name="use-the-interpretability-package-to-explain-ml-models--predictions-in-python-preview"></a>Использование пакета интерпретации для объяснения моделей машинного обучения & прогнозах в Python (Предварительная версия)



В этом пошаговом руководство вы узнаете, как использовать пакет интерпретации пакета SDK для Машинное обучение Azure Python для выполнения следующих задач:


* Объясните все поведение модели или отдельные прогнозы на персональном компьютере локально.

* Включить приемы интерпретации для сконструированных функций.

* Объясните поведение всей модели и отдельных прогнозов в Azure.

* Используйте панель мониторинга визуализации для взаимодействия с пояснениями к модели.

* Разверните пояснение к оценке вместе с моделью, чтобы просмотреть объяснения во время ее формирования.



Дополнительные сведения о поддерживаемых методиках интерпретации и моделях машинного обучения см. [в статье интерпретируемость модели в машинное обучение Azure](how-to-machine-learning-interpretability.md) и [примеры записных книжек](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/explain-model).

## <a name="generate-feature-importance-value-on-your-personal-machine"></a>Создание значения важности функции на личном компьютере 
В следующем примере показано, как использовать пакет интерпретации на персональном компьютере без обращения к службам Azure.

1. Установите пакет `azureml-interpret`.
    ```bash
    pip install azureml-interpret
    ```

2. Обучение образца модели в локальной Jupyter Notebook.

    ```python
    # load breast cancer dataset, a well-known small dataset that comes with scikit-learn
    from sklearn.datasets import load_breast_cancer
    from sklearn import svm
    from sklearn.model_selection import train_test_split
    breast_cancer_data = load_breast_cancer()
    classes = breast_cancer_data.target_names.tolist()
    
    # split data into train and test
    from sklearn.model_selection import train_test_split
    x_train, x_test, y_train, y_test = train_test_split(breast_cancer_data.data,            
                                                        breast_cancer_data.target,  
                                                        test_size=0.2,
                                                        random_state=0)
    clf = svm.SVC(gamma=0.001, C=100., probability=True)
    model = clf.fit(x_train, y_train)
    ```

3. Вызовите объяснение в локальной среде.
   * Чтобы инициализировать объект объяснения, передайте модель и некоторые обучающие данные в конструктор объяснения.
   * Чтобы сделать объяснения и визуализации более информативными, вы можете передать имена функций и имена выходных классов при выполнении классификации.

   В следующих блоках кода показано, как создать экземпляр объекта объяснения с помощью `TabularExplainer` , `MimicExplainer` и `PFIExplainer` локально.
   * `TabularExplainer` вызывает один из трех объяснений ШАП ниже ( `TreeExplainer` , `DeepExplainer` или `KernelExplainer` ).
   * `TabularExplainer` автоматически выбирает наиболее подходящий вариант для вашего варианта использования, но можно напрямую вызвать каждый из трех базовых методов объяснения.

    ```python
    from interpret.ext.blackbox import TabularExplainer

    # "features" and "classes" fields are optional
    explainer = TabularExplainer(model, 
                                 x_train, 
                                 features=breast_cancer_data.feature_names, 
                                 classes=classes)
    ```

    или диспетчер конфигурации служб

    ```python

    from interpret.ext.blackbox import MimicExplainer
    
    # you can use one of the following four interpretable models as a global surrogate to the black box model
    
    from interpret.ext.glassbox import LGBMExplainableModel
    from interpret.ext.glassbox import LinearExplainableModel
    from interpret.ext.glassbox import SGDExplainableModel
    from interpret.ext.glassbox import DecisionTreeExplainableModel

    # "features" and "classes" fields are optional
    # augment_data is optional and if true, oversamples the initialization examples to improve surrogate model accuracy to fit original model.  Useful for high-dimensional data where the number of rows is less than the number of columns.
    # max_num_of_augmentations is optional and defines max number of times we can increase the input data size.
    # LGBMExplainableModel can be replaced with LinearExplainableModel, SGDExplainableModel, or DecisionTreeExplainableModel
    explainer = MimicExplainer(model, 
                               x_train, 
                               LGBMExplainableModel, 
                               augment_data=True, 
                               max_num_of_augmentations=10, 
                               features=breast_cancer_data.feature_names, 
                               classes=classes)
    ```

    или диспетчер конфигурации служб

    ```python
    from interpret.ext.blackbox import PFIExplainer

    # "features" and "classes" fields are optional
    explainer = PFIExplainer(model,
                             features=breast_cancer_data.feature_names, 
                             classes=classes)
    ```

### <a name="explain-the-entire-model-behavior-global-explanation"></a>Объясните все поведение модели (глобальное объяснение) 

Чтобы получить сводные (глобальные) значения важности функций, см. Следующий пример.

```python

# you can use the training data or the test data here, but test data would allow you to use Explanation Exploration
global_explanation = explainer.explain_global(x_test)

# if you used the PFIExplainer in the previous step, use the next line of code instead
# global_explanation = explainer.explain_global(x_train, true_labels=y_train)

# sorted feature importance values and feature names
sorted_global_importance_values = global_explanation.get_ranked_global_values()
sorted_global_importance_names = global_explanation.get_ranked_global_names()
dict(zip(sorted_global_importance_names, sorted_global_importance_values))

# alternatively, you can print out a dictionary that holds the top K feature names and values
global_explanation.get_feature_importance_dict()
```

### <a name="explain-an-individual-prediction-local-explanation"></a>Объясните отдельный прогноз (локальное объяснение)
Получите значения важности отдельных компонентов для различных точек данных, вызвав объяснения для отдельного экземпляра или группы экземпляров.
> [!NOTE]
> `PFIExplainer` не поддерживает локальные объяснения.

```python
# get explanation for the first data point in the test set
local_explanation = explainer.explain_local(x_test[0:5])

# sorted feature importance values and feature names
sorted_local_importance_names = local_explanation.get_ranked_local_names()
sorted_local_importance_values = local_explanation.get_ranked_local_values()
```

### <a name="raw-feature-transformations"></a>Преобразования необработанных функций

Вы можете получить объяснения с точки зрения необработанных, непреобразованных, а не сконструированных функций. Для этого параметра конвейер преобразования компонента передается в объяснение в `train_explain.py` . В противном случае разъяснение содержит объяснения с точки зрения инженерных функций.

Формат поддерживаемых преобразований такой же, как описано в [sklearn-Pandas](https://github.com/scikit-learn-contrib/sklearn-pandas). Как правило, все преобразования поддерживаются при условии, что они работают с одним столбцом, чтобы сделать их более простыми.

Ознакомьтесь с описанием необработанных функций с помощью `sklearn.compose.ColumnTransformer` или со списком заданных кортежей трансформатора. В следующем примере используется `sklearn.compose.ColumnTransformer` .

```python
from sklearn.compose import ColumnTransformer

numeric_transformer = Pipeline(steps=[
    ('imputer', SimpleImputer(strategy='median')),
    ('scaler', StandardScaler())])

categorical_transformer = Pipeline(steps=[
    ('imputer', SimpleImputer(strategy='constant', fill_value='missing')),
    ('onehot', OneHotEncoder(handle_unknown='ignore'))])

preprocessor = ColumnTransformer(
    transformers=[
        ('num', numeric_transformer, numeric_features),
        ('cat', categorical_transformer, categorical_features)])

# append classifier to preprocessing pipeline.
# now we have a full prediction pipeline.
clf = Pipeline(steps=[('preprocessor', preprocessor),
                      ('classifier', LogisticRegression(solver='lbfgs'))])


# clf.steps[-1][1] returns the trained classification model
# pass transformation as an input to create the explanation object
# "features" and "classes" fields are optional
tabular_explainer = TabularExplainer(clf.steps[-1][1],
                                     initialization_examples=x_train,
                                     features=dataset_feature_names,
                                     classes=dataset_classes,
                                     transformations=preprocessor)
```

Если вы хотите выполнить пример со списком заданных кортежей трансформатора, используйте следующий код:

```python
from sklearn.pipeline import Pipeline
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.linear_model import LogisticRegression
from sklearn_pandas import DataFrameMapper

# assume that we have created two arrays, numerical and categorical, which holds the numerical and categorical feature names

numeric_transformations = [([f], Pipeline(steps=[('imputer', SimpleImputer(
    strategy='median')), ('scaler', StandardScaler())])) for f in numerical]

categorical_transformations = [([f], OneHotEncoder(
    handle_unknown='ignore', sparse=False)) for f in categorical]

transformations = numeric_transformations + categorical_transformations

# append model to preprocessing pipeline.
# now we have a full prediction pipeline.
clf = Pipeline(steps=[('preprocessor', DataFrameMapper(transformations)),
                      ('classifier', LogisticRegression(solver='lbfgs'))])

# clf.steps[-1][1] returns the trained classification model
# pass transformation as an input to create the explanation object
# "features" and "classes" fields are optional
tabular_explainer = TabularExplainer(clf.steps[-1][1],
                                     initialization_examples=x_train,
                                     features=dataset_feature_names,
                                     classes=dataset_classes,
                                     transformations=transformations)
```

## <a name="generate-feature-importance-values-via-remote-runs"></a>Создание значений важности функций с помощью удаленных запусков

В следующем примере показано, как можно использовать `ExplanationClient` класс, чтобы разрешить интерпретируемость модели для удаленных запусков. Он концептуально напоминает локальный процесс, за исключением:

* Используйте `ExplanationClient` в удаленном запуске, чтобы передать контекст интерпретации.
* Скачайте контекст позже в локальной среде.

1. Установите пакет `azureml-interpret`.
    ```bash
    pip install azureml-interpret
    ```
1. Создание обучающего скрипта в локальной Jupyter Notebook. Например, `train_explain.py`.

    ```python
    from azureml.interpret import ExplanationClient
    from azureml.core.run import Run
    from interpret.ext.blackbox import TabularExplainer

    run = Run.get_context()
    client = ExplanationClient.from_run(run)

    # write code to get and split your data into train and test sets here
    # write code to train your model here 

    # explain predictions on your local machine
    # "features" and "classes" fields are optional
    explainer = TabularExplainer(model, 
                                 x_train, 
                                 features=feature_names, 
                                 classes=classes)

    # explain overall model predictions (global explanation)
    global_explanation = explainer.explain_global(x_test)
    
    # uploading global model explanation data for storage or visualization in webUX
    # the explanation can then be downloaded on any compute
    # multiple explanations can be uploaded
    client.upload_model_explanation(global_explanation, comment='global explanation: all features')
    # or you can only upload the explanation object with the top k feature info
    #client.upload_model_explanation(global_explanation, top_k=2, comment='global explanation: Only top 2 features')
    ```

1. Настройте Машинное обучение Azure вычислений в качестве целевого объекта вычислений и отправьте обучающий запуск. Инструкции см. в статье [Создание и управление машинное обучение Azure вычислений для кластеров](how-to-create-attach-compute-cluster.md) . Вы также можете найти [примеры записных книжек](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/explain-model/azure-integration/remote-explanation) .

1. Скачайте пояснения в локальную Jupyter Notebook.

    ```python
    from azureml.interpret import ExplanationClient
    
    client = ExplanationClient.from_run(run)
    
    # get model explanation data
    explanation = client.download_model_explanation()
    # or only get the top k (e.g., 4) most important features with their importance values
    explanation = client.download_model_explanation(top_k=4)
    
    global_importance_values = explanation.get_ranked_global_values()
    global_importance_names = explanation.get_ranked_global_names()
    print('global importance values: {}'.format(global_importance_values))
    print('global importance names: {}'.format(global_importance_names))
    ```


## <a name="visualizations"></a>Визуализации

После скачивания объяснений в локальную Jupyter Notebook можно использовать панель мониторинга визуализации для понимания и интерпретации модели. Чтобы загрузить мини-приложение панели мониторинга визуализации в Jupyter Notebook, используйте следующий код:

```python
from interpret_community.widget import ExplanationDashboard

ExplanationDashboard(global_explanation, model, datasetX=x_test)
```

Визуализация поддерживает объяснения как для сконструированных, так и для необработанных функций. Необработанные объяснения основаны на возможностях исходного набора данных, а инженерные объяснения основаны на возможностях набора данных с примененным проектированием признаков.

При попытке интерпретировать модель с учетом исходного набора данных рекомендуется использовать необработанные объяснения, так как каждый уровень важности каждого компонента будет соответствовать столбцу из исходного набора данных. Один из сценариев, в котором могут быть полезны инженерные объяснения, может быть полезен при анализе влияния отдельных категорий на функцию категоризации. Если к функции категоризации применяется Однофункциональная кодировка, то в итоговых инженерных объяснениях будет использоваться другое значение важности для каждой категории, по одной на одну и ту же функцию. Это может быть полезно при ограничении части набора данных, наиболее информативной для модели.

> [!NOTE]
> Инженерные и необработанные объяснения вычисляются последовательно. Сначала сконструированное пояснение создается на основе модели и конвейера Добавление признаков. Затем необработанное описание создается на основе этого инженерного объяснения путем агрегирования важности сконструированных функций, поступилных из одной необработанной функции.

### <a name="create-edit-and-view-dataset-cohorts"></a>Создание, изменение и Просмотр набора данных когорты

На верхней ленте показана общая статистика по модели и данным. Вы можете выполнить срез и кость данных в когорты набора данных, или подгруппы, чтобы исследовать или сравнивать производительность модели и пояснения между этими определенными подгруппами. Сравнивая статистику набора данных и пояснения между этими подгруппами, можно получить представление о том, почему возможны ошибки в одной группе и в другой.

[![Создание, изменение и просмотр когорты набора данных](./media/how-to-machine-learning-interpretability-aml/dataset-cohorts.gif)](./media/how-to-machine-learning-interpretability-aml/dataset-cohorts.gif#lightbox)

### <a name="understand-entire-model-behavior-global-explanation"></a>Понимание поведения модели (глобальное объяснение) 

Первые три вкладки панели мониторинга объяснение обеспечивают общий анализ обученной модели, а также ее прогнозы и объяснения.

#### <a name="model-performance"></a>Производительность модели
Оцените производительность модели, изучив распределение прогнозируемых значений и значений метрик производительности модели. Вы можете дополнительно исследовать модель, изучив сравнительный анализ его производительности в разных когорты или подгруппах набора данных. Выберите Фильтры вдоль значения y и значения x, чтобы вырезание по разным измерениям. Просмотр метрик, таких как точность, точность, отзыв, ложный положительный коэффициент (зарегистрированном) и ложная Отрицательная частота (ФНР).

[![Вкладка «производительность модели» в визуализации пояснения](./media/how-to-machine-learning-interpretability-aml/model-performance.gif)](./media/how-to-machine-learning-interpretability-aml/model-performance.gif#lightbox)

#### <a name="dataset-explorer"></a>Обозреватель наборов данных
Изучите статистику набора данных, выбрав разные фильтры вдоль X, Y и цветовых осей для среза данных по разным измерениям. Создайте набор данных когорты выше для анализа статистики набора данных с помощью фильтров, таких как прогнозируемый результат, функции набора данных и группы ошибок. Используйте значок шестеренки в правом верхнем углу диаграммы, чтобы изменить типы диаграмм.

[![Вкладка «Обозреватель наборов данных» в визуализации пояснения](./media/how-to-machine-learning-interpretability-aml/dataset-explorer.gif)](./media/how-to-machine-learning-interpretability-aml/dataset-explorer.gif#lightbox)

#### <a name="aggregate-feature-importance"></a>Совокупное важность функций
Ознакомьтесь с важнейшими функциями, которые влияют на общие прогнозы модели (также известное как глобальное объяснение). Используйте ползунок для отображения значений важности функции по убыванию. Выберите до трех когорты, чтобы увидеть значения важности их компонентов рядом. Щелкните любую панель функций на диаграмме, чтобы увидеть, как значения выбранного компонента влияют на прогнозируемую модель в схеме зависимостей ниже.

[![Общая вкладка «важность функций» в визуализации пояснения](./media/how-to-machine-learning-interpretability-aml/aggregate-feature-importance.gif)](./media/how-to-machine-learning-interpretability-aml/aggregate-feature-importance.gif#lightbox)

### <a name="understand-individual-predictions-local-explanation"></a>Общие сведения о конкретных прогнозах (локальное объяснение) 

Четвертая вкладка вкладки пояснения позволяет детализировать отдельные точки данных и их индивидуальные характеристики. Вы можете загрузить график важности отдельных компонентов для любой точки данных, щелкнув любую из отдельных точек данных на главной точечной диаграмме или выбрав определенную точку в мастере панели справа.

|Графическое представления|Описание|
|----|-----------|
|Важность отдельных компонентов|Показывает важные функции верхнего уровня для отдельного прогноза. Помогает проиллюстрировать локальное поведение базовой модели на определенной точке данных.|
|Анализ What-If|Позволяет изменять значения компонентов выбранной реальной точки данных и отслеживать итоговые изменения в прогнозируемых значениях, создавая гипотетическую точку данных с новыми значениями функций.|
|Отдельное условное ожидание (ICE)|Позволяет изменять значение компонента с минимального значения на максимальное значение. Помогает проиллюстрировать изменение прогноза точки данных при изменении компонента.|

[![Важность отдельных компонентов и вкладка "что если" в объяснении панели мониторинга](./media/how-to-machine-learning-interpretability-aml/individual-tab.gif)](./media/how-to-machine-learning-interpretability-aml/individual-tab.gif#lightbox)

> [!NOTE]
> Эти пояснения основаны на многих приближениях и не являются «причинами» прогнозов. Без тщательной математической надежности вывода причин мы не рекомендуем пользователям принимать реальные решения на основе пертурбатионсного средства What-If. Это средство в основном предназначено для понимания модели и отладки.

### <a name="visualization-in-azure-machine-learning-studio"></a>Визуализация в Машинное обучение Azure Studio

Если вы завершите действия по [удаленной интерпретации](how-to-machine-learning-interpretability-aml.md#generate-feature-importance-values-via-remote-runs) (Отправка созданного пояснения в журнал машинное обучение Azure выполнения), вы можете просмотреть панель мониторинга визуализации в [машинное обучение Azure Studio](https://ml.azure.com). Эта панель мониторинга представляет собой более простую версию панели мониторинга визуализации, описанную выше. What-If создания и построения точек ввода-отображения отключены, так как в Машинное обучение Azure Studio нет активных вычислений, которые могут выполнять вычисления в реальном времени.

Если доступны наборы данных, глобальные и локальные объяснения, данные заполняют все вкладки. Если доступно только глобальное объяснение, вкладка "важность отдельных функций" будет отключена.

Используйте один из этих путей для доступа к панели мониторинга визуализации в Машинное обучение Azure Studio:

* Панель " **эксперименты** " (Предварительная версия)
  1. Выберите **эксперименты** на левой панели, чтобы просмотреть список экспериментов, которые вы выполняли на машинное обучение Azure.
  1. Выберите конкретный эксперимент, чтобы просмотреть все запуски в этом эксперименте.
  1. Выберите выполнение, а затем на вкладке **пояснения** на панели мониторинга визуализации объяснение.

   [![Панель мониторинга визуализации с общей важностью функций в AzureML Studio в экспериментах](./media/how-to-machine-learning-interpretability-aml/model-explanation-dashboard-aml-studio.png)](./media/how-to-machine-learning-interpretability-aml/model-explanation-dashboard-aml-studio.png#lightbox)

* Панель « **модели** »
  1. Если вы зарегистрировали исходную модель, выполнив действия, описанные в разделе [Развертывание моделей с помощью машинное обучение Azure](./how-to-deploy-and-where.md), можно выбрать **модели** в левой области, чтобы просмотреть их.
  1. Выберите модель, а затем на вкладке **пояснения** Просмотрите панель мониторинга визуализации пояснения.

## <a name="interpretability-at-inference-time"></a>Возможные интерпретации во время вывода

Вы можете развернуть пояснение вместе с исходной моделью и использовать его во время вывода, чтобы предоставить отдельные значения важности функции (локальное объяснение) для любой новой точки. Мы также предлагаем более легкие оценки для повышения производительности интерпретации во время вывода, которые в настоящее время поддерживаются только в Машинное обучение Azure SDK. Процесс развертывания пояснения к более светлой оценке аналогичен развертыванию модели и включает следующие шаги:

1. Создайте объект пояснения. Например, можно использовать `TabularExplainer` :

   ```python
    from interpret.ext.blackbox import TabularExplainer


   explainer = TabularExplainer(model, 
                                initialization_examples=x_train, 
                                features=dataset_feature_names, 
                                classes=dataset_classes, 
                                transformations=transformations)
   ```

1. Создайте объяснение оценки с помощью объекта пояснения.

   ```python
   from azureml.interpret.scoring.scoring_explainer import KernelScoringExplainer, save

   # create a lightweight explainer at scoring time
   scoring_explainer = KernelScoringExplainer(explainer)

   # pickle scoring explainer
   # pickle scoring explainer locally
   OUTPUT_DIR = 'my_directory'
   save(scoring_explainer, directory=OUTPUT_DIR, exist_ok=True)
   ```

1. Настройка и регистрация образа, использующего модель пояснения к оценкам.

   ```python
   # register explainer model using the path from ScoringExplainer.save - could be done on remote compute
   # scoring_explainer.pkl is the filename on disk, while my_scoring_explainer.pkl will be the filename in cloud storage
   run.upload_file('my_scoring_explainer.pkl', os.path.join(OUTPUT_DIR, 'scoring_explainer.pkl'))
   
   scoring_explainer_model = run.register_model(model_name='my_scoring_explainer', 
                                                model_path='my_scoring_explainer.pkl')
   print(scoring_explainer_model.name, scoring_explainer_model.id, scoring_explainer_model.version, sep = '\t')
   ```

1. В качестве дополнительного шага можно получить описание оценки из облака и протестировать объяснения.

   ```python
   from azureml.interpret.scoring.scoring_explainer import load

   # retrieve the scoring explainer model from cloud"
   scoring_explainer_model = Model(ws, 'my_scoring_explainer')
   scoring_explainer_model_path = scoring_explainer_model.download(target_dir=os.getcwd(), exist_ok=True)

   # load scoring explainer from disk
   scoring_explainer = load(scoring_explainer_model_path)

   # test scoring explainer locally
   preds = scoring_explainer.explain(x_test)
   print(preds)
   ```

1. Разверните образ в целевом объекте вычислений, выполнив следующие действия.

   1. При необходимости Зарегистрируйте исходную модель прогноза, выполнив действия, описанные в разделе [Развертывание моделей с помощью машинное обучение Azure](./how-to-deploy-and-where.md).

   1. Создайте файл оценки.

         ```python
         %%writefile score.py
         import json
         import numpy as np
         import pandas as pd
         import os
         import pickle
         from sklearn.externals import joblib
         from sklearn.linear_model import LogisticRegression
         from azureml.core.model import Model
          
         def init():
         
            global original_model
            global scoring_model
             
            # retrieve the path to the model file using the model name
            # assume original model is named original_prediction_model
            original_model_path = Model.get_model_path('original_prediction_model')
            scoring_explainer_path = Model.get_model_path('my_scoring_explainer')

            original_model = joblib.load(original_model_path)
            scoring_explainer = joblib.load(scoring_explainer_path)

         def run(raw_data):
            # get predictions and explanations for each data point
            data = pd.read_json(raw_data)
            # make prediction
            predictions = original_model.predict(data)
            # retrieve model explanations
            local_importance_values = scoring_explainer.explain(data)
            # you can return any data type as long as it is JSON-serializable
            return {'predictions': predictions.tolist(), 'local_importance_values': local_importance_values}
         ```
   1. Определите конфигурацию развертывания.

         Эта конфигурация зависит от требований модели. В следующем примере определяется конфигурация, использующая одно ядро ЦП и один ГБ памяти.

         ```python
         from azureml.core.webservice import AciWebservice

          aciconfig = AciWebservice.deploy_configuration(cpu_cores=1,
                                                    memory_gb=1,
                                                    tags={"data": "NAME_OF_THE_DATASET",
                                                          "method" : "local_explanation"},
                                                    description='Get local explanations for NAME_OF_THE_PROBLEM')
         ```

   1. Создайте файл с зависимостями окружения.

         ```python
         from azureml.core.conda_dependencies import CondaDependencies

         # WARNING: to install this, g++ needs to be available on the Docker image and is not by default (look at the next cell)

         azureml_pip_packages = ['azureml-defaults', 'azureml-core', 'azureml-telemetry', 'azureml-interpret']
 

         # specify CondaDependencies obj
         myenv = CondaDependencies.create(conda_packages=['scikit-learn', 'pandas'],
                                          pip_packages=['sklearn-pandas'] + azureml_pip_packages,
                                          pin_sdk_version=False)


         with open("myenv.yml","w") as f:
            f.write(myenv.serialize_to_string())

         with open("myenv.yml","r") as f:
            print(f.read())
         ```

   1. Создайте пользовательский dockerfile с установленным параметром g + +.

         ```python
         %%writefile dockerfile
         RUN apt-get update && apt-get install -y g++
         ```

   1. Разверните созданный образ.
   
         Этот процесс занимает примерно пять минут.

         ```python
         from azureml.core.webservice import Webservice
         from azureml.core.image import ContainerImage

         # use the custom scoring, docker, and conda files we created above
         image_config = ContainerImage.image_configuration(execution_script="score.py",
                                                         docker_file="dockerfile",
                                                         runtime="python",
                                                         conda_file="myenv.yml")

         # use configs and models generated above
         service = Webservice.deploy_from_model(workspace=ws,
                                             name='model-scoring-service',
                                             deployment_config=aciconfig,
                                             models=[scoring_explainer_model, original_model],
                                             image_config=image_config)

         service.wait_for_deployment(show_output=True)
         ```

1. Протестируйте развертывание.

    ```python
    import requests

    # create data to test service with
    examples = x_list[:4]
    input_data = examples.to_json()

    headers = {'Content-Type':'application/json'}

    # send request to service
    resp = requests.post(service.scoring_uri, input_data, headers=headers)

    print("POST to url", service.scoring_uri)
    # can covert back to Python objects from json string if desired
    print("prediction:", resp.text)
    ```

1. Очистка.

   Для удаления развернутой веб-службы используйте `service.delete()`.

## <a name="troubleshooting"></a>Устранение неполадок

* **Разреженные данные не поддерживаются**: более подробное описание модели — это немедленная работа с большим количеством функций, поэтому сейчас мы не поддерживаем формат разреженных данных. Кроме того, при работе с большими наборами данных и большим количеством функций будут возникать общие проблемы с памятью. 

* **Модели прогнозирования не поддерживаются в объяснениях моделей**: интерпретируемость, объяснение наилучшей модели, недоступна для экспериментов по прогнозированию аутомл, которые рекомендуют следующие алгоритмы в качестве наилучшей модели: Ткнфорекастер, Аутоарима, Профет, Експонентиалсмусинг, Average, Байеса среднего и сезонного Байеса. Прогноз Аутомл содержит модели регрессии, которые поддерживают объяснения. Однако на панели мониторинга объяснение вкладка "важность отдельных компонентов" не поддерживается для прогнозирования из-за сложности в конвейерах данных.

* **Локальное объяснение для индекса данных**. панель мониторинга "объяснение" не поддерживает присвоить локальные значения важности идентификатору строки из исходного набора данных проверки, если этот набор данных больше 5000 точек данных в качестве панели мониторинга случайным образом довнсамплес данные. Однако на вкладке важность отдельных функций отображаются значения необработанных наборов данных для каждого объекта, переданного на панель мониторинга. Пользователи могут сопоставлять локальные значения с исходным набором данных с помощью соответствующих значений функций необработанных наборов данных. Если размер набора данных проверки менее 5000 образцов, `index` функция в AzureML Studio будет соответствовать индексу в проверочном наборе данных.

* **Графики "что если/" не поддерживаются в Studio**: What-If и отдельные графики условного определения (ICE) не поддерживаются в машинное обучение Azure Studio на вкладке "объяснения", поскольку для передаваемого объяснения требуется активное вычисление для пересчета прогнозов и вероятностей пертурбедных функций. Сейчас она поддерживается в записных книжках Jupyter при запуске в качестве мини-приложения с помощью пакета SDK.


## <a name="next-steps"></a>Дальнейшие действия

[Дополнительные сведения о интерпретируемости модели](how-to-machine-learning-interpretability.md)

[Ознакомьтесь с примерами Машинное обучение Azure интерпретируемые записные книжки](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/explain-model)
