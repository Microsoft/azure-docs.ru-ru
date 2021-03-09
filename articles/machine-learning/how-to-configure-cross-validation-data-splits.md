---
title: Разбиение данных и перекрестная проверка в автоматизированном машинном обучении
titleSuffix: Azure Machine Learning
description: Узнайте, как настроить Разбиение набора данных и перекрестную проверку для автоматических экспериментов машинного обучения.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, automl
ms.author: cesardl
author: CESARDELATORRE
ms.reviewer: nibaccam
ms.date: 02/23/2021
ms.openlocfilehash: 31d3dc2c2d8194541ba1fe7d0865e6c939d75f73
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102501588"
---
# <a name="configure-data-splits-and-cross-validation-in-automated-machine-learning"></a>Настройка разбиения и перекрестной проверки данных в автоматизированном машинном обучении

В этой статье вы узнаете о различных вариантах настройки обучающих данных и данных проверки, а также параметров перекрестной проверки для автоматизированного машинного обучения, автоматизированного ML и экспериментов.

В Машинное обучение Azure при использовании автоматизированного машинного обучения для построения нескольких моделей МАШИНного обучения каждому дочернему выполнению необходимо проверить связанную модель, вычисляя метрики качества для этой модели, такие как точность или AUC с весовым коэффициентом. Эти метрики рассчитываются путем сравнения прогнозов, производимых с каждой моделью, с реальными метками прошлых наблюдений в данных проверки. [Узнайте больше о том, как рассчитываются метрики на основе типа проверки](#metric-calculation-for-cross-validation-in-machine-learning). 

Автоматические эксперименты ML выполняют проверку модели автоматически. В следующих разделах описано, как можно дополнительно настроить параметры проверки с помощью [пакета SDK для машинное обучение Azure Python](/python/api/overview/azure/ml/). 

Сведения о низком коде или работе без кода см. [в статье Создание автоматических экспериментов машинного обучения в машинное обучение Azure Studio](how-to-use-automated-ml-for-ml-models.md#create-and-run-experiment). 

> [!NOTE]
> В настоящее время Studio поддерживает разбиение данных для обучения и проверки, а также параметры перекрестной проверки, но не поддерживает указание отдельных файлов данных для набора проверки. 

## <a name="prerequisites"></a>Предварительные требования

Для этой статьи требуется:

* Рабочая область машинного обучения Azure. Дополнительные сведения см. в [инструкциях по созданию рабочей области Машинного обучения Azure](how-to-manage-workspace.md).

* Знакомство с настройкой автоматизированного эксперимента машинного обучения с помощью пакета SDK для Машинное обучение Azure. Следуйте указаниям [учебника](tutorial-auto-train-models.md) или [инструкции](how-to-configure-auto-train.md) , чтобы ознакомиться с основными конструктивными шаблонами для экспериментов машинного обучения.

* Анализ данных об обучении и проверке разбивается и перекрестная проверка в качестве концепций машинного обучения. Подробное описание

    * [Сведения об обучении, проверке и тестировании данных в машинном обучении](https://towardsdatascience.com/train-validation-and-test-sets-72cb40cba9e7)

    * [Общие сведения о перекрестной проверке в машинном обучении](https://towardsdatascience.com/understanding-cross-validation-419dbd47e9bd) 

## <a name="default-data-splits-and-cross-validation-in-machine-learning"></a>Разбиение данных по умолчанию и перекрестная проверка в машинном обучении

Используйте объект [аутомлконфиг](/python/api/azureml-train-automl-client/azureml.train.automl.automlconfig.automlconfig) для определения параметров эксперимента и обучения. В следующем фрагменте кода обратите внимание, что определены только обязательные параметры, которые являются параметрами для `n_cross_validation` или `validation_ data` **не** включены.

```python
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/creditcard.csv"

dataset = Dataset.Tabular.from_delimited_files(data)

automl_config = AutoMLConfig(compute_target = aml_remote_compute,
                             task = 'classification',
                             primary_metric = 'AUC_weighted',
                             training_data = dataset,
                             label_column_name = 'Class'
                            )
```

Если параметр или не задан явно `validation_data` `n_cross_validation` , автоматический алгоритм ML применяет методы по умолчанию в зависимости от числа строк, предоставленных в одном наборе данных `training_data` :

|&nbsp;Размер данных для обучения &nbsp;| Метод проверки |
|---|-----|
|**Более &nbsp; &nbsp; 20 000 &nbsp; строк**| Применяется разбиение данных по обучению и проверке. Значение по умолчанию — использовать 10% начального набора данных для обучения в качестве набора проверки. В свою очередь, этот набор проверки используется для вычисления метрик.
|**Меньше &nbsp; &nbsp; 20 000 &nbsp; строк**| Применяется подход перекрестной проверки. Количество сверток по умолчанию зависит от количества строк. <br> **Если набор данных имеет менее 1 000 строк**, то используется 10 сверток. <br> **Если строки находятся в диапазоне от 1 000 до 20 000**, то используются три свертывания.

## <a name="provide-validation-data"></a>Предоставление данных проверки

В этом случае можно начать с одного файла данных и разделить его на обучающие данные и наборы данных проверки. Кроме того, можно указать отдельный файл данных для набора проверки. В любом случае `validation_data` параметр в `AutoMLConfig` объекте назначает, какие данные следует использовать в качестве набора проверки. Этот параметр принимает только наборы данных в форме [машинное обучение Azureного набора данных](how-to-create-register-datasets.md) или Pandas.   

> [!NOTE]
> `validation_size`Параметр не поддерживается в сценариях прогнозирования.

В следующем примере кода явно определяется, какая часть предоставленных данных в `dataset` используется для обучения и проверки.

```python
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/creditcard.csv"

dataset = Dataset.Tabular.from_delimited_files(data)

training_data, validation_data = dataset.random_split(percentage=0.8, seed=1)

automl_config = AutoMLConfig(compute_target = aml_remote_compute,
                             task = 'classification',
                             primary_metric = 'AUC_weighted',
                             training_data = training_data,
                             validation_data = validation_data,
                             label_column_name = 'Class'
                            )
```

## <a name="provide-validation-set-size"></a>Задать размер набора проверки

В этом случае для эксперимента предоставляется только один набор данных. То есть `validation_data` параметр **не** указан, а предоставленный набор данных присваивается  `training_data` параметру.  

В `AutoMLConfig` объекте можно задать `validation_size` параметр для удержания части обучающих данных для проверки. Это означает, что набор проверки будет разделяться автоматическим ML из `training_data` предоставленного начального значения. Это значение должно находиться в диапазоне от 0,0 до 1,0, не включительно (например, 0,2 означает, что 20% данных будут храниться для данных проверки).

> [!NOTE]
> `validation_size`Параметр не поддерживается в сценариях прогнозирования. 

См. Следующий пример кода:

```python
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/creditcard.csv"

dataset = Dataset.Tabular.from_delimited_files(data)

automl_config = AutoMLConfig(compute_target = aml_remote_compute,
                             task = 'classification',
                             primary_metric = 'AUC_weighted',
                             training_data = dataset,
                             validation_size = 0.2,
                             label_column_name = 'Class'
                            )
```

## <a name="k-fold-cross-validation"></a>Перекрестная проверка K раз

Чтобы выполнить перекрестную проверку с помощью k-сгиба, включите `n_cross_validations` параметр и присвойте ему значение. Этот параметр задает количество выполняемых перекрестных проверок на основе того же числа сверток.

> [!NOTE]
> `n_cross_validations`Параметр не поддерживается в сценариях классификации, использующих глубокие нейронные сети.
 
В следующем коде определяется пять сверток для перекрестной проверки. Таким образом, пять различных обученияов, каждое обучение с использованием 4/5 данных и каждая проверка с использованием 1/5 данных с разными контрольными данными каждый раз.

В результате метрики рассчитываются с учетом среднего значения для пяти метрик проверки.

```python
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/creditcard.csv"

dataset = Dataset.Tabular.from_delimited_files(data)

automl_config = AutoMLConfig(compute_target = aml_remote_compute,
                             task = 'classification',
                             primary_metric = 'AUC_weighted',
                             training_data = dataset,
                             n_cross_validations = 5
                             label_column_name = 'Class'
                            )
```
## <a name="monte-carlo-cross-validation"></a>Монте-Карло перекрестная проверка

Чтобы выполнить перекрестную проверку Монте-Карло, включите `validation_size` оба `n_cross_validations` параметра и в `AutoMLConfig` объект. 

Для перекрестной проверки Монте-Карло автоматически ML записывает часть обучающих данных, указанных в `validation_size` параметре, для проверки, а затем назначит остальные данные для обучения. Затем этот процесс повторяется в зависимости от значения, указанного в `n_cross_validations` параметре; в результате каждый раз в случайном порядке создаются новые разбиения на обучение и проверку.

> [!NOTE]
> Перекрестная проверка Монте-Карло не поддерживается в сценариях прогнозирования.

Следующий код определяет, 7 сверток для перекрестной проверки и 20% обучающих данных должны использоваться для проверки. Таким образом, 7 различных обученияов каждое обучение использует 80% данных, и каждая проверка использует 20% данных с разными контрольными данными.

```python
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/creditcard.csv"

dataset = Dataset.Tabular.from_delimited_files(data)

automl_config = AutoMLConfig(compute_target = aml_remote_compute,
                             task = 'classification',
                             primary_metric = 'AUC_weighted',
                             training_data = dataset,
                             n_cross_validations = 7
                             validation_size = 0.2,
                             label_column_name = 'Class'
                            )
```

## <a name="specify-custom-cross-validation-data-folds"></a>Выбор нестандартных данных для перекрестной проверки

Вы также можете предоставить собственные данные перекрестной проверки (ОПС). Это считается более сложным сценарием, поскольку вы указываете, какие столбцы следует разделить и использовать для проверки.  Включите настраиваемые столбцы с разделением КП в обучающие данные и укажите столбцы, заполнив имена столбцов в `cv_split_column_names` параметре. Каждый столбец представляет одну разбивку перекрестной проверки и заполняется целыми значениями 1 или 0, где 1 означает, что строка должна использоваться для обучения, а значение 0 указывает, что строка должна использоваться для проверки.

В следующем фрагменте кода содержатся данные по маркетинговым маркетингу с двумя столбцами разбиения на столбцы "cv1" и "CV2".

```python
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/bankmarketing_with_cv.csv"

dataset = Dataset.Tabular.from_delimited_files(data)

automl_config = AutoMLConfig(compute_target = aml_remote_compute,
                             task = 'classification',
                             primary_metric = 'AUC_weighted',
                             training_data = dataset,
                             label_column_name = 'y',
                             cv_split_column_names = ['cv1', 'cv2']
                            )
```

> [!NOTE]
> Чтобы использовать `cv_split_column_names` с `training_data` и `label_column_name` , обновите пакет SDK для машинное обучение Azure Python версии 1.6.0 или более поздней. Сведения о предыдущих версиях пакета SDK см. в разделе Использование `cv_splits_indices` , но обратите внимание, что он используется `X` только с `y` входными данными набора данных и. 


## <a name="metric-calculation-for-cross-validation-in-machine-learning"></a>Вычисление метрик для перекрестной проверки в машинном обучении

При использовании перекрестной проверки k-сгиба или Монте-Карло, метрики вычисляются по каждой из сверток, а затем суммируются. Статистическая операция представляет собой среднее значение для скалярных метрик и сумму для диаграмм. Метрики, вычисленные во время перекрестной проверки, основываются на всех сверток, и поэтому все образцы из обучающего набора. [Узнайте больше о метриках в автоматизированном машинном обучении](how-to-understand-automated-ml.md).

Если используется настраиваемый набор проверки или автоматически выбранный набор проверки, метрики оценки модели вычисляются только по этому набору проверки, а не к обучающим данным.

## <a name="next-steps"></a>Дальнейшие действия

* [Предотвращение несбалансированных данных и перегонка](concept-manage-ml-pitfalls.md).
* [Руководство. Использование автоматизированного машинного обучения для прогнозирования годуного в такси фрагмента данных](tutorial-auto-train-models.md#split-the-data-into-train-and-test-sets).
* [Автоматическое обучение модели прогнозирования временных рядов](how-to-auto-train-forecast.md).
