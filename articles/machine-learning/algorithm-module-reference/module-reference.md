---
title: Справочник по алгоритмам и модулям
description: Сведения о модулях конструктора Машинное обучение Azure, которые можно использовать для создания собственных проектов машинного обучения.
titleSuffix: Azure Machine Learning
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 11/09/2020
ms.openlocfilehash: 89ad9aae7c0d01971bbcfc7e392cb9d455ef85cd
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94376848"
---
# <a name="algorithm--module-reference-for-azure-machine-learning-designer"></a>Ссылка на модуль & алгоритма для конструктора Машинное обучение Azure

Это справочное содержимое содержит технические сведения о каждом из алгоритмов и модулей машинного обучения, доступных в Машинное обучение Azure Designer.

Каждый модуль представляет собой набор кода, который может работать независимо и выполнять задачу машинного обучения с учетом требуемых входных данных. Модуль может содержать определенный алгоритм или выполнять задачу, важную для машинного обучения, например, замена отсутствующего значения или статистический анализ.

Справку по выбору алгоритмов см. в следующих статьях: 
* [Выбор алгоритмов](../how-to-select-algorithms.md)
* [Памятка по алгоритмам Машинного обучения Azure](../algorithm-cheat-sheet.md)

> [!TIP]
> В любом конвейере в конструкторе можно получить сведения о конкретном модуле. Щелкните ссылку **Learn more** (Подробнее) в карточке модуля, наведя указатель мыши на модуль в списке модулей, или на правой панели модуля.

## <a name="data-preparation-modules"></a>Модули подготовки данных


| Функциональность | Описание | Модуль |
| --- |--- | --- |
| Ввод и вывод данных | Перемещение данных из облачных источников в конвейер. Запишите результаты или промежуточные данные в службу хранилища Azure, базу данных SQL или Hive, во время выполнения конвейера или используйте облачное хранилище для обмена данными между конвейерами.  | [Ввод данных вручную](enter-data-manually.md) <br/> [Экспорт данных](export-data.md) <br/> [Импорт данных](import-data.md) |
| Преобразование данных | Операции с данными, которые являются уникальными для машинного обучения, такие как нормализация или группирование данных, сокращение размерности и преобразование данных между разными форматами файлов.| [Добавление столбцов](add-columns.md) <br/> [Добавление строк](add-rows.md) <br/> [Применение математической операции](apply-math-operation.md) <br/> [Применение преобразования SQL](apply-sql-transformation.md) <br/> [Очистка недостающих данных](clean-missing-data.md) <br/> [Обрезка значений](clip-values.md) <br/> [Преобразование в CSV-файл](convert-to-csv.md) <br/> [Преобразование в набор данных](convert-to-dataset.md) <br/> [Преобразование в значения индикатора](convert-to-indicator-values.md) <br/> [Изменение метаданных](edit-metadata.md) <br/> [Группирование данных в ячейки](group-data-into-bins.md) <br/> [Объединение данных](join-data.md) <br/> [Нормализация данных](normalize-data.md) <br/> [Секционирование и выборка](partition-and-sample.md)  <br/> [Удаление дублирующихся строк](remove-duplicate-rows.md) <br/> [SMOTE](smote.md) <br/> [Преобразование "Выбор столбцов"](select-columns-transform.md) <br/> [Выбор столбцов в наборе данных](select-columns-in-dataset.md) <br/> [Split Data](split-data.md) (Разделение данных); |
| Выбор компонентов | Выбор важных и полезных функций для построения аналитической модели. | [Выбор признаков с помощью фильтра](filter-based-feature-selection.md) <br/> [Значение функции перестановки](permutation-feature-importance.md) |
| Статистические функции | Широкий спектр статистических методов, относящихся к обработке и анализу данных. | [Сведение данных](summarize-data.md)|

## <a name="machine-learning-algorithms"></a>Алгоритмы машинного обучения

| Функциональность | Описание | Модуль |
| --- |--- | --- |
| Регрессия | Прогнозирование значения. | [Регрессия с помощью увеличивающегося дерева принятия решений](boosted-decision-tree-regression.md) <br/> [Регрессия с использованием модели леса принятия решений](decision-forest-regression.md) <br/> [Быстрая квантильная регрессия леса](fast-forest-quantile-regression.md)  <br/> [Линейная регрессия](linear-regression.md)  <br/> [Регрессия нейронной сети](neural-network-regression.md)  <br/> [Регрессия Пуассона](poisson-regression.md)  <br/>|
| Кластеризация | Группирование данных.| [Кластеризация методом K-средних](k-means-clustering.md)
| Классификация | Прогнозирование класса.  Выбор одного из двоичных (для двух классов) или мультиклассовых алгоритмов.| [Мультиклассовое увеличивающееся дерево принятия решений](multiclass-boosted-decision-tree.md) <br/> [Мультиклассовый лес принятия решений](multiclass-decision-forest.md) <br/> [Мультиклассовая регрессионная логистическая модель](multiclass-logistic-regression.md)  <br/> [Мультиклассовая нейронная сеть](multiclass-neural-network.md) <br/> [Многоклассовый классификатор "один — все"](one-vs-all-multiclass.md) <br/> [Многоклассовый классификатор "один — один"](one-vs-one-multiclass.md) <br/>[Двухклассовое усредненное восприятие](two-class-averaged-perceptron.md) <br/>  [Two-Class Boosted Decision Tree](two-class-boosted-decision-tree.md) (Двухклассовое увеличивающееся дерево принятия решений);  <br/> [Двухклассовый лес принятия решений](two-class-decision-forest.md) <br/>  [Двухклассовая регрессионная логистическая модель](two-class-logistic-regression.md) <br/> [Двухклассовая нейронная сеть](two-class-neural-network.md) <br/> [Двухклассовый метод опорных векторов](two-class-support-vector-machine.md) | 

## <a name="modules-for-building-and-evaluating-models"></a>Модули для создания и оценки моделей

| Функциональность | Описание | Модуль |
| --- |--- | --- |
| Обучение модели | Обработка данных с помощью алгоритма. |  [Обучение модели кластеризации](train-clustering-model.md) <br/> [Train Model](train-model.md) (Обучение модели); <br/> [Обучение модели Pytorch](train-pytorch-model.md) <br/> [Настройка гиперпараметров модели](tune-model-hyperparameters.md) |
| Анализ и оценка модели | Измерение точности обученной модели. | [Применение преобразования](apply-transformation.md) <br/> [Назначение данных в кластеры](assign-data-to-clusters.md) <br/> [Перекрестная проверка модели](cross-validate-model.md) <br/> [Анализ модели](evaluate-model.md) <br/> [Оценка моделей изображений](score-image-model.md) <br/> [Оценка модели](score-model.md) |
| Язык Python | Внедрение кода в модуль для интеграции Python с вашим конвейером. | [Создание модели Python](create-python-model.md) <br/> [Выполнение скриптов Python](execute-python-script.md) |
| Язык R | Внедрение кода в модуль для интеграции R с вашим конвейером. | [Выполнение скрипта R](execute-r-script.md) |
| Анализ текста | Специализированные вычислительные средства для работы со структурированным и неструктурированным текстом. |  [Преобразование слов в векторы](convert-word-to-vector.md) <br/> [Извлечение N-грамм из текста](extract-n-gram-features-from-text.md) <br/> [Хэширование признаков](feature-hashing.md) <br/> [Предварительная обработка текста](preprocess-text.md) <br/> [Латентное распределение Дирихле (LDA)](latent-dirichlet-allocation.md) <br/> [Оценка модели Vowpal Wabbit](score-vowpal-wabbit-model.md) <br/> [Обучение модели Vowpal Wabbit](train-vowpal-wabbit-model.md)|
| Компьютерное зрение | Модули, связанные с предварительной обработкой данных образа и распознаванием изображений. |  [Применение преобразования изображений](apply-image-transformation.md) <br/> [Преобразование в каталог изображений](convert-to-image-directory.md) <br/> [Инициализация преобразования изображений](init-image-transformation.md) <br/> [Разделение каталога изображений](split-image-directory.md) <br/> [DenseNet](densenet.md) <br/> [ResNet](resnet.md) |
| Рекомендация | Построение моделей рекомендаций. | [Оценка рекомендателя](evaluate-recommender.md) <br/> [Оценка рекомендателя SVD](score-svd-recommender.md) <br/> [Оценка Wide and Deep Recommender](score-wide-and-deep-recommender.md)<br/> [Обучение рекомендателя SVD](train-SVD-recommender.md) <br/> [Обучение Wide and Deep Recommender](train-wide-and-deep-recommender.md)|
| Обнаружение аномалий | Построение моделей обнаружения аномалий. | [Обнаружение аномалий на основе анализа первичных компонентов](pca-based-anomaly-detection.md) <br/> [Обучение модели обнаружения аномалий](train-anomaly-detection-model.md) |


## <a name="web-service"></a>Веб-служба

Ознакомьтесь со сведениями о [модулях веб-служб](web-service-input-output.md), необходимых для вывода в реальном времени в конструкторе Машинного обучения Azure.

## <a name="error-messages"></a>Сообщения об ошибках

Прочитайте о [сообщениях об ошибках и кодах исключений](designer-error-codes.md), которые могут возникнуть при использовании модулей в конструкторе Машинного обучения Azure.

## <a name="next-steps"></a>Дальнейшие действия

* [Руководство. Прогнозирование цен на автомобили с помощью конструктора](../tutorial-designer-automobile-price-train-score.md)
