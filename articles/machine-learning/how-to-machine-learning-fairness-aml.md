---
title: Оценка равноправия моделей ML в Python (Предварительная версия)
titleSuffix: Azure Machine Learning
description: Узнайте, как оценивать и устранять равноправие моделей машинного обучения с помощью Фаирлеарн и пакета SDK для Python Машинное обучение Azure.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: mesameki
author: mesameki
ms.reviewer: luquinta
ms.date: 11/16/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, responsible-ml
ms.openlocfilehash: 70ad5d6d88dabb51e022a1fc5c011341b06f02fd
ms.sourcegitcommit: c8b50a8aa8d9596ee3d4f3905bde94c984fc8aa2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/28/2021
ms.locfileid: "105640683"
---
# <a name="use-azure-machine-learning-with-the-fairlearn-open-source-package-to-assess-the-fairness-of-ml-models-preview"></a>Использование Машинное обучение Azure с пакетом с открытым исходным кодом Фаирлеарн для оценки распределения моделей машинного обучения (Предварительная версия)

В этом пошаговом руководство вы узнаете, как использовать пакет Python с открытым кодом [фаирлеарн](https://fairlearn.github.io/) с машинное обучение Azure для выполнения следующих задач:

* Оцените равноправие прогнозов модели. Дополнительные сведения о равноправии в машинном обучении см. в [статье равноправие в машинном обучении](concept-fairness-ml.md).
* Отправьте, перечислите и скачайте аналитические сведения об оценке равноправия в Машинное обучение Azure Studio.
* Ознакомьтесь с панелью мониторинга оценки равноправия в Машинное обучение Azure Studio, чтобы взаимодействовать с аналитическими сведениями об равноправии моделей.

>[!NOTE]
> Оценка равноправия не является чисто техническим упражнением. **Этот пакет поможет вам оценить равноправие модели машинного обучения, но только вы можете настроить и принять решения относительно того, как работает модель.**  Хотя этот пакет помогает определить количественные метрики для оценки равноправия, разработчики моделей машинного обучения также должны выполнить качественный анализ, чтобы оценить равноправие собственных моделей.

## <a name="azure-machine-learning-fairness-sdk"></a>Пакет SDK для Машинное обучение Azure равноправия 

Пакет SDK для Машинное обучение Azure равноправии `azureml-contrib-fairness` интегрирует пакет Python с открытым кодом ( [фаирлеарн](http://fairlearn.github.io)) в машинное обучение Azure. Чтобы узнать больше об интеграции Фаирлеарн в Машинное обучение Azure, ознакомьтесь с этими [примерами записных книжек](https://github.com/Azure/MachineLearningNotebooks/tree/master/contrib/fairness). Дополнительные сведения о Фаирлеарн см. в [примере руководств](https://fairlearn.org/v0.6.0/auto_examples/) и [примерах записных книжек](https://github.com/fairlearn/fairlearn/tree/master/notebooks). 

Для установки пакетов и используйте следующие команды `azureml-contrib-fairness` `fairlearn` :
```bash
pip install azureml-contrib-fairness
pip install fairlearn==0.4.6
```
Более поздние версии Фаирлеарн также должны работать в следующем примере кода.



## <a name="upload-fairness-insights-for-a-single-model"></a>Отправка сведений об равноправии для одной модели

В следующем примере показано, как использовать пакет равномерного распределения. Мы отправим аналитическую аналитику распределения модели в Машинное обучение Azure и видим панель мониторинга оценки равноправия в Машинное обучение Azure Studio.

1. Обучить образец модели в Jupyter Notebook. 

    Для набора данных используется известный набор данных для взрослых переписей, который мы получаем из Опенмл. Мы надеемся, что у нас есть проблема принятия решения по ссуде с меткой, которая указывает, оплачивается ли человек в прошлом займе. Мы обучить модель для прогнозирования того, что ранее незамеченные лица будут репай ссуду. Такую модель можно использовать для принятия решений о займах.

    ```python
    import copy
    import numpy as np
    import pandas as pd

    from sklearn.compose import ColumnTransformer
    from sklearn.datasets import fetch_openml
    from sklearn.impute import SimpleImputer
    from sklearn.linear_model import LogisticRegression
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler, OneHotEncoder
    from sklearn.compose import make_column_selector as selector
    from sklearn.pipeline import Pipeline
    
    from fairlearn.widget import FairlearnDashboard

    # Load the census dataset
    data = fetch_openml(data_id=1590, as_frame=True)
    X_raw = data.data
    y = (data.target == ">50K") * 1
    
    # (Optional) Separate the "sex" and "race" sensitive features out and drop them from the main data prior to training your model
    X_raw = data.data
    y = (data.target == ">50K") * 1
    A = X_raw[["race", "sex"]]
    X = X_raw.drop(labels=['sex', 'race'],axis = 1)
    
    # Split the data in "train" and "test" sets
    (X_train, X_test, y_train, y_test, A_train, A_test) = train_test_split(
        X_raw, y, A, test_size=0.3, random_state=12345, stratify=y
    )

    # Ensure indices are aligned between X, y and A,
    # after all the slicing and splitting of DataFrames
    # and Series
    X_train = X_train.reset_index(drop=True)
    X_test = X_test.reset_index(drop=True)
    y_train = y_train.reset_index(drop=True)
    y_test = y_test.reset_index(drop=True)
    A_train = A_train.reset_index(drop=True)
    A_test = A_test.reset_index(drop=True)

    # Define a processing pipeline. This happens after the split to avoid data leakage
    numeric_transformer = Pipeline(
        steps=[
            ("impute", SimpleImputer()),
            ("scaler", StandardScaler()),
        ]
    )
    categorical_transformer = Pipeline(
        [
            ("impute", SimpleImputer(strategy="most_frequent")),
            ("ohe", OneHotEncoder(handle_unknown="ignore")),
        ]
    )
    preprocessor = ColumnTransformer(
        transformers=[
            ("num", numeric_transformer, selector(dtype_exclude="category")),
            ("cat", categorical_transformer, selector(dtype_include="category")),
        ]
    )

    # Put an estimator onto the end of the pipeline
    lr_predictor = Pipeline(
        steps=[
            ("preprocessor", copy.deepcopy(preprocessor)),
            (
                "classifier",
                LogisticRegression(solver="liblinear", fit_intercept=True),
            ),
        ]
    )

    # Train the model on the test data
    lr_predictor.fit(X_train, y_train)

    # (Optional) View this model in Fairlearn's fairness dashboard, and see the disparities which appear:
    from fairlearn.widget import FairlearnDashboard
    FairlearnDashboard(sensitive_features=A_test, 
                       sensitive_feature_names=['Race', 'Sex'],
                       y_true=y_test,
                       y_pred={"lr_model": lr_predictor.predict(X_test)})
    ```

2. Войдите в Машинное обучение Azure и зарегистрируйте модель.
   
    Панель мониторинга равноправия может интегрироваться с зарегистрированными или незарегистрированными моделями. Зарегистрируйте модель в Машинное обучение Azure, выполнив следующие действия.
    ```python
    from azureml.core import Workspace, Experiment, Model
    import joblib
    import os
    
    ws = Workspace.from_config()
    ws.get_details()

    os.makedirs('models', exist_ok=True)
    
    # Function to register models into Azure Machine Learning
    def register_model(name, model):
        print("Registering ", name)
        model_path = "models/{0}.pkl".format(name)
        joblib.dump(value=model, filename=model_path)
        registered_model = Model.register(model_path=model_path,
                                        model_name=name,
                                        workspace=ws)
        print("Registered ", registered_model.id)
        return registered_model.id

    # Call the register_model function 
    lr_reg_id = register_model("fairness_logistic_regression", lr_predictor)
    ```

3. Предварительное вычисление метрик равномерного распределения.

    Создание словаря панели мониторинга с помощью `metrics` пакета фаирлеарн. `_create_group_metric_set`Метод имеет аргументы, аналогичные конструктору панели мониторинга, за исключением того, что конфиденциальные функции передаются в виде словаря (чтобы обеспечить доступность имен). При вызове этого метода необходимо также указать тип прогноза (в данном случае это двоичная классификация).

    ```python
    #  Create a dictionary of model(s) you want to assess for fairness 
    sf = { 'Race': A_test.race, 'Sex': A_test.sex}
    ys_pred = { lr_reg_id:lr_predictor.predict(X_test) }
    from fairlearn.metrics._group_metric_set import _create_group_metric_set

    dash_dict = _create_group_metric_set(y_true=y_test,
                                        predictions=ys_pred,
                                        sensitive_features=sf,
                                        prediction_type='binary_classification')
    ```
4. Отправьте предварительно вычисленные метрики распределения.
    
    Теперь импортируйте `azureml.contrib.fairness` пакет, чтобы выполнить отправку:

    ```python
    from azureml.contrib.fairness import upload_dashboard_dictionary, download_dashboard_by_upload_id
    ```
    Создайте эксперимент, запустите и отправьте на него панель мониторинга:
    ```python
    exp = Experiment(ws, "Test_Fairness_Census_Demo")
    print(exp)

    run = exp.start_logging()

    # Upload the dashboard to Azure Machine Learning
    try:
        dashboard_title = "Fairness insights of Logistic Regression Classifier"
        # Set validate_model_ids parameter of upload_dashboard_dictionary to False if you have not registered your model(s)
        upload_id = upload_dashboard_dictionary(run,
                                                dash_dict,
                                                dashboard_name=dashboard_title)
        print("\nUploaded to id: {0}\n".format(upload_id))
        
        # To test the dashboard, you can download it back and ensure it contains the right information
        downloaded_dict = download_dashboard_by_upload_id(run, upload_id)
    finally:
        run.complete()
    ```
5. Проверка панели мониторинга равноправия в Машинное обучение Azure Studio

    Если вы выполните предыдущие действия (отгрузка созданной аналитики распределения ресурсов в Машинное обучение Azure), вы можете просмотреть панель мониторинга равноправие в [машинное обучение Azure Studio](https://ml.azure.com). Эта панель мониторинга — это та же панель мониторинга визуализации, что и в Фаирлеарн, что позволяет анализировать различия между подгруппами конфиденциальных функций (например, от пола до женщина).
    Используйте один из этих путей для доступа к панели мониторинга визуализации в Машинное обучение Azure Studio:

    * **Панель "эксперименты" (Предварительная версия)**
    1. Выберите **эксперименты** на левой панели, чтобы просмотреть список экспериментов, которые вы выполняли на машинное обучение Azure.
    1. Выберите конкретный эксперимент, чтобы просмотреть все запуски в этом эксперименте.
    1. Выберите выполнение, а затем вкладку **равноправие** на панели мониторинга визуализации объяснение.
    1. После размещения на вкладке **равноправие** щелкните **идентификатор равноправия** в меню справа.
    1. Настройте панель мониторинга, выбрав секретный атрибут, метрику производительности и метрику равномерного распределения для земли на странице Оценка равноправия.
    1. Переключите тип диаграммы с одного на другой, чтобы наблюдать за причинами **распределения** и причинами потери **качества обслуживания** .



    [![Выделение панели мониторинга равноправия](./media/how-to-machine-learning-fairness-aml/dashboard-1.png)](./media/how-to-machine-learning-fairness-aml/dashboard-1.png#lightbox)
    
    [![Качество обслуживания панели мониторинга равноправия](./media/how-to-machine-learning-fairness-aml/dashboard-2.png)](./media/how-to-machine-learning-fairness-aml/dashboard-2.png#lightbox)
    * **Панель «модели»**
    1. Если вы зарегистрировали исходную модель, выполнив описанные выше действия, можно выбрать **модели** на левой панели, чтобы просмотреть их.
    1. Выберите модель, а затем вкладку **равноправие** , чтобы просмотреть панель мониторинга визуализации пояснения.

    Дополнительные сведения о панели мониторинга визуализации и содержащихся в ней элементах см. в статье о [пользователе](https://fairlearn.org/v0.6.0/user_guide/assessment.html#fairlearn-dashboard)фаирлеарн.

## <a name="upload-fairness-insights-for-multiple-models"></a>Передача сведений об равноправии для нескольких моделей

Чтобы сравнить несколько моделей и увидеть, как различаются их оценки равноправия, можно передать более одной модели на панель мониторинга визуализации и сравнить компромиссы с их производительностью.

1. Обучение моделей:
    
    Теперь мы создадим второй классификатор, основанный на механизме оценки машинного вектора поддержки, и отправим словарь панели мониторинга равноправия с помощью `metrics` пакета фаирлеарн. Предполагается, что ранее обученная модель по-прежнему доступна.


    ```python
    # Put an SVM predictor onto the preprocessing pipeline
    from sklearn import svm
    svm_predictor = Pipeline(
        steps=[
            ("preprocessor", copy.deepcopy(preprocessor)),
            (
                "classifier",
                svm.SVC(),
            ),
        ]
    )

    # Train your second classification model
    svm_predictor.fit(X_train, y_train)
    ```

2. Регистрация моделей

    Затем следует зарегистрировать обе модели в Машинное обучение Azure. Для удобства храните результаты в словаре, который сопоставляет `id` зарегистрированную модель (строку в `name:version` формате) с самим прогнозом:

    ```python
    model_dict = {}

    lr_reg_id = register_model("fairness_logistic_regression", lr_predictor)
    model_dict[lr_reg_id] = lr_predictor

    svm_reg_id = register_model("fairness_svm", svm_predictor)
    model_dict[svm_reg_id] = svm_predictor
    ```

3. Локальная загрузка панели мониторинга Фаирлеарн

    Перед отправкой ценной информации по равномерному анализу в Машинное обучение Azure можно проверить эти прогнозы на локально вызванной панели мониторинга Фаирлеарн. 



    ```python
    #  Generate models' predictions and load the fairness dashboard locally 
    ys_pred = {}
    for n, p in model_dict.items():
        ys_pred[n] = p.predict(X_test)

    from fairlearn.widget import FairlearnDashboard

    FairlearnDashboard(sensitive_features=A_test, 
                    sensitive_feature_names=['Race', 'Sex'],
                    y_true=y_test.tolist(),
                    y_pred=ys_pred)
    ```

3. Предварительное вычисление метрик равномерного распределения.

    Создание словаря панели мониторинга с помощью `metrics` пакета фаирлеарн.

    ```python
    sf = { 'Race': A_test.race, 'Sex': A_test.sex }

    from fairlearn.metrics._group_metric_set import _create_group_metric_set

    dash_dict = _create_group_metric_set(y_true=Y_test,
                                        predictions=ys_pred,
                                        sensitive_features=sf,
                                        prediction_type='binary_classification')
    ```
4. Отправьте предварительно вычисленные метрики распределения.
    
    Теперь импортируйте `azureml.contrib.fairness` пакет, чтобы выполнить отправку:

    ```python
    from azureml.contrib.fairness import upload_dashboard_dictionary, download_dashboard_by_upload_id
    ```
    Создайте эксперимент, запустите и отправьте на него панель мониторинга:
    ```python
    exp = Experiment(ws, "Compare_Two_Models_Fairness_Census_Demo")
    print(exp)

    run = exp.start_logging()

    # Upload the dashboard to Azure Machine Learning
    try:
        dashboard_title = "Fairness Assessment of Logistic Regression and SVM Classifiers"
        # Set validate_model_ids parameter of upload_dashboard_dictionary to False if you have not registered your model(s)
        upload_id = upload_dashboard_dictionary(run,
                                                dash_dict,
                                                dashboard_name=dashboard_title)
        print("\nUploaded to id: {0}\n".format(upload_id))
        
        # To test the dashboard, you can download it back and ensure it contains the right information
        downloaded_dict = download_dashboard_by_upload_id(run, upload_id)
    finally:
        run.complete()
    ```


    Как и в предыдущем разделе, можно выполнить один из описанных выше путей (через **эксперименты** или **модели**) в машинное обучение Azure Studio, чтобы получить доступ к панели мониторинга визуализации и сравнить две модели с точки зрения равномерности и производительности.


## <a name="upload-unmitigated-and-mitigated-fairness-insights"></a>Отправка нерешенных и сниженных ценных сведений об равноправии

Вы можете использовать [алгоритмы защиты](https://fairlearn.org/v0.6.0/user_guide/mitigation.html)фаирлеарн, сравнить созданные по снижению модели с исходными неминимизированными моделями и перемещаться по компромиссам производительности и равноправию между сравниваемыми моделями.

Пример, демонстрирующий использование алгоритма устранения рисков [поиска в сетке](https://fairlearn.org/v0.6.0/user_guide/mitigation.html#grid-search) (который создает коллекцию проблемных моделей с разными компромиссами и обделами производительности), см. в этом [примере записной книжки](https://github.com/Azure/MachineLearningNotebooks/blob/master/contrib/fairness/fairlearn-azureml-mitigation.ipynb). 

Загрузка нескольких моделей "равноправие аналитики" в одном запуске позволяет сравнивать модели по отношению к равномерности и производительности. Можно щелкнуть любую из моделей, отображаемых на диаграмме сравнения моделей, чтобы просмотреть подробные сведения об равноправии конкретной модели.


[![Панель мониторинга Фаирлеарн для сравнения моделей](./media/how-to-machine-learning-fairness-aml/multi-model-dashboard.png)](./media/how-to-machine-learning-fairness-aml/multi-model-dashboard.png#lightbox)
    

## <a name="next-steps"></a>Дальнейшие шаги

[Дополнительные сведения о равномерном использовании модели](concept-fairness-ml.md)

[Ознакомьтесь с примерами записных книжек Машинное обучение Azure равноправие](https://github.com/Azure/MachineLearningNotebooks/tree/master/contrib/fairness)
