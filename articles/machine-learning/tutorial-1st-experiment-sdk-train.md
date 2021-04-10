---
title: Руководство по обучению первых моделей машинного обучения — Python
titleSuffix: Azure Machine Learning
description: В третьей части серии по началу работы с Машинным обучением Azure описано, как обучить модель машинного обучения.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: aminsaied
ms.author: amsaied
ms.reviewer: sgilley
ms.date: 02/11/2021
ms.custom: devx-track-python, contperf-fy21q3
ms.openlocfilehash: 7e035406ce220728c316aaa2005e0f4d6089b5e2
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105935521"
---
# <a name="tutorial-train-your-first-machine-learning-model-part-3-of-4"></a>Руководство по обучению первых моделей машинного обучения (часть 3 из 4)

В этом учебнике описано, как обучить модель машинного обучения в Машинном обучении Azure.

Этот учебник является *третьей частью серии из четырех учебников*, в рамках которой вы ознакомитесь с основами Машинного обучения Azure и выполните задачи машинного обучения на основе заданий в Azure. Этот учебник создан на основе работы, выполненной в рамках статьи [Часть 1: настройка](tutorial-1st-experiment-sdk-setup-local.md) и [Части 2: Запуск скрипта "Hello World!"](tutorial-1st-experiment-hello-world.md) .

В рамках этого учебника вы выполните следующий шаг, отправив сценарий, который обучает модель машинного обучения. В этом примере показано, как Машинное обучение Azure позволяет легко добиться согласованного поведения между локальной отладкой и удаленными запусками.

Изучив это руководство, вы:

> [!div class="checklist"]
> * Создайте сценарий обучения.
> * используете Conda для определения среды Машинного обучения Azure;
> * Создание скрипта элемента управления
> * Общие сведения о классах Машинного обучения Azure (`Environment`, `Run`, `Metrics`).
> * отправите и запустите свой сценарий обучения;
> * просмотрите выходные данные кода в облаке;
> * научитесь вести журнал метрик в Машинном обучении Azure;
> * просмотрите свои метрики в облаке.

## <a name="prerequisites"></a>Предварительные условия

- [Anaconda](https://www.anaconda.com/download/) или [Miniconda](https://www.anaconda.com/download/) для управления виртуальными средами Python и установки пакетов.
- Завершение [части 1](tutorial-1st-experiment-sdk-setup-local.md) и [части 2](tutorial-1st-experiment-hello-world.md) в серии руководств.

## <a name="create-training-scripts"></a>Создание сценариев обучения

Сначала определите архитектуру нейронной сети в файле `model.py`. Весь обучающий код будет перемещен в подкаталог `src`, включая `model.py`.

Приведенный ниже код взят из [этого вводного примера](https://pytorch.org/tutorials/beginner/blitz/cifar10_tutorial.html) из PyTorch. Обратите внимание, что концепции Машинного обучения Azure применяются к любому коду машинного обучения, а не только к PyTorch.

:::code language="python" source="~/MachineLearningNotebooks/tutorials/get-started-day1/IDE-users/src/model.py":::

Далее определите сценарий обучения. Этот сценарий скачивает набор данных CIFAR10 с помощью API-интерфейсов PyTorch `torchvision.dataset`, настраивает сеть, определенную в `model.py`, и обучает ее для двух эпох, используя стандартный SGD и потери с перекрестной энтропией.

Создайте сценарий `train.py` в подкаталоге `src`:

:::code language="python" source="~/MachineLearningNotebooks/tutorials/get-started-day1/IDE-users/src/train.py":::

Теперь у вас есть следующая структура каталогов:

:::image type="content" source="media/tutorial-1st-experiment-sdk-train/directory-structure.png" alt-text="Структура каталогов с файлом train.py в подкаталоге src":::


> [!div class="nextstepaction"]
> [Созданы скрипты обучения](?success=create-scripts#environment) [Возникла проблема](https://www.research.net/r/7CTJQQN?issue=create-scripts)

## <a name="create-a-new-python-environment"></a><a name="environment"></a> Создание новой среды Python

В скрытом каталоге `.azureml` создайте файл с именем `pytorch-env.yml`:

:::code language="yml" source="~/MachineLearningNotebooks/tutorials/get-started-day1/IDE-users/environments/pytorch-env.yml":::

Эта среда имеет все зависимости, необходимые для вашей модели и сценария обучения. Обратите внимание на отсутствие зависимости от пакета SDK Машинного обучения Azure для Python.

> [!div class="nextstepaction"]
> [Создан файл среды](?success=create-env-file#test-local) [Возникла проблема](https://www.research.net/r/7CTJQQN?issue=create-env-file)

## <a name="test-locally"></a><a name="test-local"></a> Локальное тестирование

В окне терминала или окна запроса Anaconda используйте следующий код, чтобы протестировать скрипт локально в новой среде.  

```bash
conda deactivate                                # If you are still using the tutorial environment, exit it
conda env create -f .azureml/pytorch-env.yml    # create the new Conda environment
conda activate pytorch-env                      # activate new Conda environment
python src/train.py                             # train model
```

После выполнения этого скрипта данные будут загружены в каталог с именем `tutorial/data`.

> [!div class="nextstepaction"]
> [Код запущен локально](?success=test-local#create-local) [Возникла проблема](https://www.research.net/r/7CTJQQN?issue=test-local)

## <a name="create-the-control-script"></a><a name="create-local"></a> Создание скрипта элемента управления

Разница между следующим скриптом элемента управления и тем, который использовался для отправки сообщения "Hello World!" Вы добавляете несколько дополнительных строк, чтобы задать среду.

Создайте файл Python в каталоге `tutorial` с именем `04-run-pytorch.py`:

```python
# 04-run-pytorch.py
from azureml.core import Workspace
from azureml.core import Experiment
from azureml.core import Environment
from azureml.core import ScriptRunConfig

if __name__ == "__main__":
    ws = Workspace.from_config()
    experiment = Experiment(workspace=ws, name='day1-experiment-train')
    config = ScriptRunConfig(source_directory='./src',
                             script='train.py',
                             compute_target='cpu-cluster')

    # set up pytorch environment
    env = Environment.from_conda_specification(
        name='pytorch-env',
        file_path='./.azureml/pytorch-env.yml'
    )
    config.run_config.environment = env

    run = experiment.submit(config)

    aml_url = run.get_portal_url()
    print(aml_url)
```    
    
### <a name="understand-the-code-changes"></a>Изучение изменений кода

:::row:::
   :::column span="":::
      `env = ...`
   :::column-end:::
   :::column span="2":::
      Машинное обучение Azure реализует концепцию [среды](/python/api/azureml-core/azureml.core.environment.environment), чтобы представить воспроизводимую версионную среду Python для выполнения экспериментов. Вы можете легко создать среду из локальной среды Conda или PIP.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      `config.run_config.environment = env`
   :::column-end:::
   :::column span="2":::
      Добавление среды в [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig).
   :::column-end:::
:::row-end:::

> [!div class="nextstepaction"]
> [Мной создан скрипт управления](?success=control-script#submit) [Возникла проблема](https://www.research.net/r/7CTJQQN?issue=control-script)


## <a name="submit-the-run-to-azure-machine-learning"></a><a name="submit"></a> Отправка выполнения в Машинное обучение Azure

Переключитесь обратно в *учебную* среду с установленным пакетом SDK Машинного обучения Azure для Python. Так как обучающий код не выполняется на компьютере, PyTorch устанавливать не нужно.  Но вам потребуется `azureml-sdk` (находится в *учебной* среде).

```bash
conda deactivate
conda activate tutorial
python 04-run-pytorch.py
```

>[!NOTE] 
> При первом запуске этого скрипта Машинное обучение Azure создаст образ Docker из вашей среды PyTorch. Все выполнение может занять от 5 до 10 минут. 
>
> Журналы сборки Docker можно просмотреть в Студии машинного обучения Azure: Перейдите по ссылке в Студию, выберите вкладку **Выходные данные и журналы**, а затем выберите `20_image_build_log.txt`.
>
> Этот образ будет повторно использован в будущих запусках для их ускорения.

После сборки образа выберите `70_driver_log.txt`, чтобы просмотреть выходные данные вашего сценария обучения.

```txt
Downloading https://www.cs.toronto.edu/~kriz/cifar-10-python.tar.gz to ./data/cifar-10-python.tar.gz
...
Files already downloaded and verified
epoch=1, batch= 2000: loss 2.19
epoch=1, batch= 4000: loss 1.82
epoch=1, batch= 6000: loss 1.66
epoch=1, batch= 8000: loss 1.58
epoch=1, batch=10000: loss 1.52
epoch=1, batch=12000: loss 1.47
epoch=2, batch= 2000: loss 1.39
epoch=2, batch= 4000: loss 1.38
epoch=2, batch= 6000: loss 1.37
epoch=2, batch= 8000: loss 1.33
epoch=2, batch=10000: loss 1.31
epoch=2, batch=12000: loss 1.27
Finished Training
```

> [!WARNING]
> Если отображается сообщение об ошибке `Your total snapshot size exceeds the limit`, каталог `data` указан в значении `source_directory`, используемом в `ScriptRunConfig`.
>
> Перемещение `data` за пределы `src`.

Среды можно регистрировать в рабочей области с помощью `env.register(ws)`. Затем моно легко предоставлять к ним доступ, использовать повторно и структурировать по версиям. Среды упрощают воспроизведение предыдущих результатов и совместную работу с командой.

Машинное обучение Azure также поддерживает коллекцию курируемых сред. Эти среды охватывают общие сценарии машинного обучения и поддерживаются кэшированными образами Docker. Кэшированные образы Docker ускоряют первое удаленное выполнение.

Если коротко, использование зарегистрированных сред позволяет экономить время. Дополнительные сведения см. в статье [Использование сред](./how-to-use-environments.md).

> [!div class="nextstepaction"]
> [Выполнение отправлено](?success=test-w-environment#log) [Возникла проблема](https://www.research.net/r/7CTJQQN?issue=test-w-environment)

## <a name="log-training-metrics"></a><a name="log"></a> Ведение журнала метрик обучения

Теперь, настроив обучение модели в Машинном обучении Azure, начните отслеживать некоторые метрики производительности.

Текущий сценарий обучения выводит метрики в терминал. Машинное обучение Azure предоставляет механизм для ведения журнала метрик с дополнительными функциональными возможностями. Добавив несколько строк кода, вы получаете возможность визуализировать метрики в студии и сравнивать метрики из нескольких запусков.

### <a name="modify-trainpy-to-include-logging"></a>Изменение `train.py` для ведения журнала

Измените скрипт `train.py`, чтобы добавить еще две строки кода:

:::code language="python" source="~/MachineLearningNotebooks/tutorials/get-started-day1/code/pytorch-cifar10-train-with-logging/train.py":::


#### <a name="understand-the-additional-two-lines-of-code"></a>Сведения о дополнительных двух строках кода

В `train.py` вы обращаетесь к объекту run из _самого_ сценария обучения с помощью метода `Run.get_context()` и используете его для ведения журнала метрик:

```python
# in train.py
run = Run.get_context()

...

run.log('loss', loss)
```

Метрики в Машинном обучении Azure:

- Организованы по экспериментам и выполняются так, чтобы можно было легко отследить и сравнить метрики.
- Оснащены пользовательским интерфейсом, что позволяет визуализировать эффективность обучения в студии.
- Предназначены для масштабирования, поэтому вы можете использовать эти преимущества даже при выполнении сотен экспериментов.

> [!div class="nextstepaction"]
> [Файл train.py изменен](?success=modify-train#log) [Возникла проблема](https://www.research.net/r/7CTJQQN?issue=modify-train)

### <a name="update-the-conda-environment-file"></a>Обновление файла среды Conda

Сценарий `train.py` только что принял новую зависимость от `azureml.core`. Обновите `pytorch-env.yml`, чтобы отразить это изменение:

:::code language="python" source="~/MachineLearningNotebooks/tutorials/get-started-day1/configuration/pytorch-aml-env.yml":::

> [!div class="nextstepaction"]
> [Файл среды обновлен](?success=update-environment#submit-again) [Возникла проблема](https://www.research.net/r/7CTJQQN?issue=update-environment)

### <a name="submit-the-run-to-azure-machine-learning"></a><a name="submit-again"></a> Отправка выполнения в Машинное обучение Azure
Отправьте этот сценарий еще раз:

```bash
python 04-run-pytorch.py
```

На этот раз при посещении студии перейдите на вкладку **Метрики**, на которой теперь можно увидеть обновления в реальном времени для потерь обучения модели.

:::image type="content" source="media/tutorial-1st-experiment-sdk-train/logging-metrics.png" alt-text="Диаграмма потерь обучения на вкладке метрик":::

> [!div class="nextstepaction"]
> [Выполнение отправлено мною повторно](?success=resubmit-with-logging#next-steps) [Возникла проблема](https://www.research.net/r/7CTJQQN?issue=resubmit-with-logging)

## <a name="next-steps"></a>Следующие шаги

В этом сеансе вы обновили базовый сценарий Hello World! на более реалистичный сценарий обучения, для выполнения которого требуется определенная среда Python. Вы узнали, как использовать локальную среду Conda в облаке со средами Машинного обучение Azure. И наконец, вы узнали, как с помощью нескольких строк кода можно вести журнал метрик в Машинном обучении Azure.

Существуют и другие способы создания сред Машинного обучения Azure, в том числе из [файла requirements.txt для pip](/python/api/azureml-core/azureml.core.environment.environment#from-pip-requirements-name--file-path-) или даже из [имеющейся локальной среды Conda](/python/api/azureml-core/azureml.core.environment.environment#from-existing-conda-environment-name--conda-environment-name-).

В следующий раз вы узнаете, как работать с данными в Машинном обучении Azure путем передачи набора данных CIFAR10 в Azure.

> [!div class="nextstepaction"]
> [Руководство. Предоставление своих данных](tutorial-1st-experiment-bring-data.md)

>[!NOTE] 
> Если вы хотите на этом завершить серию учебников, не переходя к следующему шагу, [очистите ресурсы](tutorial-1st-experiment-bring-data.md#clean-up-resources).
