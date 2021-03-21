---
title: Разностная конфиденциальность — Смартноисе (Предварительная версия)
titleSuffix: Azure Machine Learning
description: Узнайте, как применить рекомендации по обеспечению разностной конфиденциальности для Машинное обучение Azure моделей с помощью библиотек с открытым исходным кодом Смартноисе.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, responsible-ml
ms.author: slbird
author: slbird
ms.reviewer: luquinta
ms.date: 01/21/2020
ms.openlocfilehash: 62a002569696da4ef18e7bd967f027eb8247ef65
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98681410"
---
# <a name="use-differential-privacy-in-azure-machine-learning-preview"></a>Использование разностной конфиденциальности в Машинное обучение Azure (Предварительная версия)

Узнайте, как применить рекомендации по обеспечению разностной конфиденциальности для Машинное обучение Azure моделей с помощью библиотек с открытым исходным кодом Python Смартноисе.

Дифференциальная конфиденциальность является образцовым определением конфиденциальности. Системы, которые соответствуют этому определению конфиденциальности, предоставляют надежную защиту от широкого спектра атак с целью реконструкции и переидентификации данных, в том числе атак злоумышленников, располагающих дополнительными сведениями. Узнайте больше о [том, как работает разностная конфиденциальность](./concept-differential-privacy.md).


## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, создайте бесплатную учетную запись, прежде чем начинать работу. Опробуйте [бесплатную или платную версию Машинного обучения Azure](https://aka.ms/AMLFree) уже сегодня.
- [Python 3](https://www.python.org/downloads/);

## <a name="install-smartnoise-python-libraries"></a>Установка библиотек Смартноисе Python

### <a name="standalone-installation"></a>Автономная установка

Эти библиотеки предназначены для работы из распределенных кластеров Spark и могут устанавливаться так же, как и любые другие пакеты.

В приведенных ниже инструкциях предполагается, что команды `python` и `pip` сопоставлены с `python3` и `pip3`.

Используйте PIP для установки [пакетов Python смартноисе](https://pypi.org/project/opendp-smartnoise/).

`pip install opendp-smartnoise`

Чтобы убедиться, что пакеты установлены, откройте командную строку Python и введите следующее.

```python
import opendp.smartnoise.core
import opendp.smartnoise.sql
```

Если импорт выполнен успешно, то библиотеки установлены и готовы к использованию.

### <a name="docker-image-installation"></a>Установка образа DOCKER

Вы также можете использовать пакеты Смартноисе с DOCKER.

Извлеките образ `opendp/smartnoise` для использования библиотек в контейнере Docker, который включает Spark, Jupyter и пример кода.

```sh
docker pull opendp/smartnoise:privacy
```

После извлечения образа запустите сервер Jupyter.

```sh
docker run --rm -p 8989:8989 --name smartnoise-run opendp/smartnoise:privacy
```

Это запустит сервер Jupyter на порте `8989` вашего `localhost` с паролем `pass@word99`. Если вы использовали приведенную выше командную строку для запуска контейнера с именем `smartnoise-privacy`, можете открыть терминал bash на сервере Jupyter, выполнив следующую команду.

```sh
docker exec -it smartnoise-run bash
```

Этот экземпляр Docker очищает все состояния при завершении работы, поэтому вы потеряете все записные книжки, созданные в запущенном экземпляре. Чтобы устранить эту проблему, можно привязать локальную папку к контейнеру при запуске.

```sh
docker run --rm -p 8989:8989 --name smartnoise-run --mount type=bind,source=/Users/your_name/my-notebooks,target=/home/privacy/my-notebooks opendp/smartnoise:privacy
```

Все записные книжки, созданные в папке *my-notebooks*, будут храниться в локальной файловой системе.

## <a name="perform-data-analysis"></a>Выполнение анализа данных

Чтобы подготовить выпуск с дифференциальной конфиденциальностью, необходимо выбрать источник данных, статистический показатель и некоторые параметры конфиденциальности, указывающие уровень защиты конфиденциальности.  

Этот образец ссылается на микроданные для общественного использования (PUMS) в Калифорнии, представляющие анонимные записи о демографических характеристиках граждан.

```python
import os
import sys
import numpy as np
import opendp.smartnoise.core as sn

data_path = os.path.join('.', 'data', 'PUMS_california_demographics_1000', 'data.csv')
var_names = ["age", "sex", "educ", "race", "income", "married", "pid"]
```

В этом примере мы вычисляем среднее значение и дисперсию возраста.  Мы используем общий `epsilon`, равный 1,0 (epsilon — это параметр конфиденциальности, распределяющий наш бюджет конфиденциальности по двум величинам, которые мы хотим вычислить). Дополнительные сведения о [метриках конфиденциальности](concept-differential-privacy.md#differential-privacy-metrics).

```python
with sn.Analysis() as analysis:
    # load data
    data = sn.Dataset(path = data_path, column_names = var_names)

    # get mean of age
    age_mean = sn.dp_mean(data = sn.cast(data['age'], type="FLOAT"),
                          privacy_usage = {'epsilon': .65},
                          data_lower = 0.,
                          data_upper = 100.,
                          data_n = 1000
                         )
    # get variance of age
    age_var = sn.dp_variance(data = sn.cast(data['age'], type="FLOAT"),
                             privacy_usage = {'epsilon': .35},
                             data_lower = 0.,
                             data_upper = 100.,
                             data_n = 1000
                            )
analysis.release()

print("DP mean of age: {0}".format(age_mean.value))
print("DP variance of age: {0}".format(age_var.value))
print("Privacy usage: {0}".format(analysis.privacy_usage))
```

Результаты выглядят примерно так.

```text
DP mean of age: 44.55598845931517
DP variance of age: 231.79044646429134
Privacy usage: approximate {
  epsilon: 1.0
}
```

В этом примере следует обратить внимание на некоторые важные моменты.  Во-первых, объект `Analysis` представляет граф обработки данных.  В этом примере среднее значение и дисперсия вычисляются из одного и того же исходного узла.  Однако можно включить более сложные выражения, объединяющие входы с выходами произвольным образом.

Граф анализа включает метаданные `data_upper` и `data_lower`, указывающие нижнюю и верхнюю границы для возраста.  Эти значения используются для точной калибровки шума, чтобы обеспечить дифференциальную конфиденциальность.  Эти значения также используются в некоторых способах обработки выбросов или отсутствующих значений.

Наконец, граф анализа отслеживает общий потраченный бюджет конфиденциальности.

Библиотеку можно использовать для создания более сложных графов анализа, используя несколько механизмов, видов статистики и служебных функций.

| Статистика    | Механизмы | Техническое оборудование  |
| ------------- |------------|------------|
| Count         | Гауссовский   | Приведение       |
| Гистограмма     | Геометрический  | Прикрепление   |
| Среднее значение          | Лапласовский    | Оцифровывание   |
| Квантили     |            | Filter     |
| SUM           |            | Добавления отсутствующих данных |
| Вариативность/ковариация |      | Преобразование  |

Дополнительные сведения см. в [записной книжке анализ данных](https://github.com/opendifferentialprivacy/smartnoise-samples/blob/master/analysis/basic_data_analysis.ipynb) .

## <a name="approximate-utility-of-differentially-private-releases"></a>Приблизительная полезность выпусков с дифференциальной конфиденциальностью

Так как дифференциальная конфиденциальность работает путем калибровки шума, полезность выпусков может зависеть от риска конфиденциальности.  Как правило, шум, необходимый для защиты каждого отдельного пользователя, становится незначительным по мере увеличения размера образцов, однако кардинальным образом меняет результат для выпусков, ориентированных на отдельного пользователя.  Аналитики могут изучить сведения о точности для выпуска, чтобы определить, насколько он полезен.

```python
with sn.Analysis() as analysis:
    # load data
    data = sn.Dataset(path = data_path, column_names = var_names)

    # get mean of age
    age_mean = sn.dp_mean(data = sn.cast(data['age'], type="FLOAT"),
                          privacy_usage = {'epsilon': .65},
                          data_lower = 0.,
                          data_upper = 100.,
                          data_n = 1000
                         )
analysis.release()

print("Age accuracy is: {0}".format(age_mean.get_accuracy(0.05)))
```

Результат этой операции должен выглядеть примерно так.

```text
Age accuracy is: 0.2995732273553991
```

Этот пример вычисляет среднее значение, приведенное выше, и использует функцию `get_accuracy` для запроса точности `alpha` в размере 0,05. Значение `alpha` в 0,05 представляет интервал 95 %, в котором выпускаемое значение будет находиться в пределах указанной точности примерно 95 % всего времени.  В этом примере указанная точность равна 0,3, что означает, что выпускаемое значение будет находиться в пределах интервала, равного 0,6, примерно 95 % всего времени.  Это значение не следует считать пределом погрешностей, так как выпускаемое значение выходит за пределы указанного диапазона точности, задаваемого `alpha`, а значения за пределами диапазона могут выходить за него в любом направлении.

Аналитики могут запрашивать `get_accuracy` для различных значений `alpha`, чтобы получить более узкие или широкие доверительные интервалы, без дополнительных затрат на конфиденциальность.

## <a name="generate-a-histogram"></a>Создание гистограммы

Встроенная функция `dp_histogram` создает гистограммы дифференциальной конфиденциальности для любого из следующих типов данных.

- Непрерывная переменная, в которой набор чисел должен быть разделен на ячейки.
- Логическая или дихотомическая переменная, которая может принимать только два значения.
- Категориальная переменная, в которой есть отдельные категории, перечисленные в виде строк.

Ниже приведен пример `Analysis` с указанием ячеек для гистограммы непрерывной переменной.

```python
income_edges = list(range(0, 100000, 10000))

with sn.Analysis() as analysis:
    data = sn.Dataset(path = data_path, column_names = var_names)

    income_histogram = sn.dp_histogram(
            sn.cast(data['income'], type='int', lower=0, upper=100),
            edges = income_edges,
            upper = 1000,
            null_value = 150,
            privacy_usage = {'epsilon': 0.5}
        )
```

Так как отдельные пользователи дизъюнктивно разнесены по ячейкам гистограммы, расходы на конфиденциальность начисляются только один раз для каждой гистограммы, даже если гистограмма содержит много ячеек.

Дополнительные сведения о гистограммах см. в [записной книжке о гистограммах](https://github.com/opendifferentialprivacy/smartnoise-samples/blob/master/analysis/histograms.ipynb).

## <a name="generate-a-covariance-matrix"></a>Создание матрицы ковариаций

Смартноисе предлагает три различные `dp_covariance` функции:

- Ковариация между двумя векторами
- Матрица ковариаций
- Матрица кросс-ковариации пары матриц

Ниже приведен пример вычисления скалярной ковариации.

```python
with sn.Analysis() as analysis:
    wn_data = sn.Dataset(path = data_path, column_names = var_names)

    age_income_cov_scalar = sn.dp_covariance(
      left = sn.cast(wn_data['age'], 
      type = "FLOAT"), 
      right = sn.cast(wn_data['income'], 
      type = "FLOAT"), 
      privacy_usage = {'epsilon': 1.0},
      left_lower = 0., 
      left_upper = 100., 
      left_n = 1000, 
      right_lower = 0., 
      right_upper = 500_000.,
      right_n = 1000)
```

Дополнительные сведения см. в [записной книжке ковариации](
https://github.com/opendifferentialprivacy/smartnoise-samples/blob/master/analysis/covariance.ipynb).

## <a name="next-steps"></a>Next Steps

- Ознакомьтесь с [примерами записных книжек смартноисе](https://github.com/opendifferentialprivacy/smartnoise-samples/tree/master/analysis).