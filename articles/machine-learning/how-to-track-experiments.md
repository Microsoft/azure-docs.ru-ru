---
title: Запись экспериментов с машинным обучением и метрик в журнал
titleSuffix: Azure Machine Learning
description: Включение ведения журнала в запусках МАШИНного обучения упрощает мониторинг метрик выполнения в реальном времени и помогает диагностировать ошибки и предупреждения.
services: machine-learning
author: likebupt
ms.author: keli19
ms.reviewer: peterlu
ms.service: machine-learning
ms.subservice: core
ms.date: 07/30/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: 9576730d9c4f8d4d237dce9ce8f207ea14b04f45
ms.sourcegitcommit: 66ce33826d77416dc2e4ba5447eeb387705a6ae5
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/15/2021
ms.locfileid: "103471602"
---
# <a name="enable-logging-in-ml-training-runs"></a>Включение ведения журнала в учебных запусках МАШИНного обучения


Пакет SDK Python для Машинного обучения Azure позволяет вам в реальном времени вести журнал с помощью пакета журналов Python по умолчанию и специализированных функций пакета SDK. Вы можете вести журналы в локальной среде и отправлять их в свою рабочую область на портале.

Журналы помогают вам диагностировать ошибки и предупреждения, а также отслеживать метрики производительности, например параметры и эффективность модели. Из этой статьи вы узнаете, как включить ведение журнала в следующих сценариях:

> [!div class="checklist"]
> * сеансы интерактивного обучения;
> * отправка заданий обучения с помощью ScriptRunConfig;
> * встроенные параметры `logging` Python;
> * ведение журнала из дополнительных источников.


> [!TIP]
> В этой статье демонстрируется, как обеспечить мониторинг для процесса обучения модели. Сведения о том, как обеспечить мониторинг использования ресурсов и событий из Машинного обучения Azure, например квот, завершенных обучающих операций или завершенных развертываний моделей, см. в статье [Мониторинг Машинного обучения Azure](monitor-azure-machine-learning.md).

## <a name="data-types"></a>Типы данных

Вы можете вести журнал по нескольким типам данных, включая скалярные значения, списки, таблицы, изображения, каталоги и т. д. Дополнительные сведения и примеры кода на Python для разных типов данных см. на [странице справки по классу Run](/python/api/azureml-core/azureml.core.run%28class%29).

### <a name="logging-run-metrics"></a>Регистрация метрик запуска 

Используйте следующие методы в API ведения журнала, чтобы повлиять на визуализации метрик. Обратите внимание на [ограничения службы](https://docs.microsoft.com/azure/machine-learning/resource-limits-quotas-capacity#metrics) для этих зарегистрированных метрик. 

|Значение в журнале|Пример кода| Формат на портале|
|----|----|----|
|Массив числовых значений| `run.log_list(name='Fibonacci', value=[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89])`|График для одной переменной|
|Одно повторяющееся числовое значение с тем же именем метрики (например, в цикле for)| `for i in tqdm(range(-10, 10)):    run.log(name='Sigmoid', value=1 / (1 + np.exp(-i))) angle = i / 2.0`| График для одной переменной|
|Повторяющиеся строки с двумя числовыми столбцами|`run.log_row(name='Cosine Wave', angle=angle, cos=np.cos(angle))   sines['angle'].append(angle)      sines['sine'].append(np.sin(angle))`|Графики для двух переменных|
|Таблица с двумя числовыми столбцами|`run.log_table(name='Sine Wave', value=sines)`|Графики для двух переменных|
|Изображение журнала|`run.log_image(name='food', path='./breadpudding.jpg', plot=None, description='desert')`|Используйте этот метод для записи файла изображения или Matplotlib построения в запуск. Эти изображения будут отображаться и сравнимы в записи запуска|

### <a name="logging-with-mlflow"></a>Ведение журнала с помощью Млфлов
Используйте Млфловлогжер для записи метрик.

```python
from azureml.core import Run
# connect to the workspace from within your running code
run = Run.get_context()
ws = run.experiment.workspace

# workspace has associated ml-flow-tracking-uri
mlflow_url = ws.get_mlflow_tracking_uri()

#Example: PyTorch Lightning
from pytorch_lightning.loggers import MLFlowLogger

mlf_logger = MLFlowLogger(experiment_name=run.experiment.name, tracking_uri=mlflow_url)
mlf_logger._run_id = run.id
```

## <a name="interactive-logging-session"></a>Интерактивный сеанс ведения журнала

Интерактивные сеансы ведения журнала обычно используются в средах записных книжек. Запустить интерактивный сеанс ведения журнала можно с помощью метода [Experiment.start_logging()](/python/api/azureml-core/azureml.core.experiment%28class%29#start-logging--args----kwargs-). Все метрики, записанные во время сеанса, добавляются в запись о выполнении эксперимента. Метод [run.complete()](/python/api/azureml-core/azureml.core.run%28class%29#complete--set-status-true-) завершает сеансы и помечает операцию как завершенную.

## <a name="scriptrun-logs"></a>Журналы ScriptRun

Из этого раздела вы узнаете, как добавить код ведения журналов в операции выполнения, созданные при настройке с помощью ScriptRunConfig. Вы можете использовать класс [**ScriptRunConfig**](/python/api/azureml-core/azureml.core.scriptrunconfig), чтобы инкапсулировать скрипты и среды для повторяемых операций. Вы также можете использовать этот параметр для отображения визуального мини-приложения Jupyter Notebook для мониторинга.

В этом примере выполняется очистка параметров для альфа-значений и регистрация результатов с помощью метода [run.log()](/python/api/azureml-core/azureml.core.run%28class%29#log-name--value--description----).

1. Создайте скрипт обучения, который включает логику ведения журнала `train.py`.

   [!code-python[](~/MachineLearningNotebooks/how-to-use-azureml/training/train-on-local/train.py)]


1. Отправьте скрипт ```train.py``` для выполнения в среде, управляемой пользователем. Для обучения отправляется вся папка скриптов.

   [! Notebook-Python [] (~/Мачинелеарнингнотебукс/хов-то-усе-азуремл/Траининг/траин-он-локал/траин-он-локал.ипинб? Name = src)]


   [! Записная книжка — Python [] (~/Мачинелеарнингнотебукс/хов-то-усе-азуремл/Траининг/траин-он-локал/траин-он-локал.ипинб? Name = Run)]

    С помощью параметра `show_output` вы можете включить подробное ведение журнала, чтобы просматривать сведения о процессе обучения, а также об удаленных ресурсах или целевых объектах вычислений. Используйте следующий код, чтобы включить подробное ведение журнала при отправке эксперимента.

```python
run = exp.submit(src, show_output=True)
```

Этот же параметр вы можете указать в функции `wait_for_completion` для финального выполнения.

```python
run.wait_for_completion(show_output=True)
```

## <a name="native-python-logging"></a>Собственное ведение журнала Python

Некоторые журналы, создаваемые с помощью пакета SDK, могут содержать сообщения об ошибках с предложением установить значение DEBUG для уровня ведения журнала. Чтобы изменить уровень ведения журнала, добавьте в скрипт приведенный ниже код.

```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

## <a name="additional-logging-sources"></a>Дополнительные источники ведения журнала

Машинное обучение Azure также может включать в журнал данные из других источников во время обучения, например при автоматизированных операциях машинного обучения, или данные из контейнеров Docker, выполняющих задания. Такие журналы не задокументированы, но если у вас возникли проблемы и вы обратились в службу поддержки Майкрософт, ее специалисты могут воспользоваться этими журналами для устранения неполадок.

См. дополнительные сведения о [ведении журналов метрик в конструкторе Машинного обучения Azure](how-to-track-designer-experiments.md).

## <a name="example-notebooks"></a>Примеры записных книжек

Основные понятия, описанные в этой статье, демонстрируют следующие записные книжки:
* [how-to-use-azureml/training/train-on-local](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training/train-on-local)
* [how-to-use-azureml/track-and-monitor-experiments/logging-api](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/track-and-monitor-experiments/logging-api)

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>Дальнейшие действия

Подробные сведения об использовании Машинного обучения Azure см. в следующих статьях:

* Узнайте, как [вести журналы метрик в конструкторе Машинного обучения Azure](how-to-track-designer-experiments.md).

* Пример регистрации оптимальных моделей и их развертывания см. в руководстве [Руководство № 1. Обучение модели классификации изображений с помощью Машинного обучения Azure](tutorial-train-models-with-aml.md).
