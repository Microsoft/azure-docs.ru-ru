---
title: Устранение неполадок ParallelRunStep
titleSuffix: Azure Machine Learning
description: Советы по устранению неполадок при возникновении ошибок с помощью Параллелрунстеп в конвейерах машинного обучения.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: troubleshooting
ms.custom: troubleshooting
ms.reviewer: larryfr, vaidyas, laobri, tracych
ms.author: trmccorm
author: tmccrmck
ms.date: 09/23/2020
ms.openlocfilehash: b5511c8ecc33238e0409b5ee4c1c7a11adddeac5
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102522161"
---
# <a name="troubleshooting-the-parallelrunstep"></a>Устранение неполадок ParallelRunStep

Из этой статьи вы узнаете, как устранять неполадки при получении ошибок с помощью класса [параллелрунстеп](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.parallel_run_step.parallelrunstep) из [пакета SDK машинное обучение Azure](/python/api/overview/azure/ml/intro).

Общие советы по устранению неполадок конвейера см. в разделе [Устранение неполадок конвейеров машинного обучения](how-to-debug-pipelines.md).

## <a name="testing-scripts-locally"></a>Локальное тестирование сценариев

 Ваш Параллелрунстеп выполняется как шаг в конвейерах машинного обучения. Вы можете [протестировать сценарии локально](how-to-debug-visual-studio-code.md#debug-and-troubleshoot-machine-learning-pipelines) в качестве первого шага.

##  <a name="script-requirements"></a>Требования сценария

Скрипт для `ParallelRunStep` *должен содержать* две функции:
- `init()`: Эта функция применяется для всех затратных или повторяющихся операций подготовки к последующему выводу. Например, в ней можно загружать модель в глобальный объект. Эта функция будет вызываться только один раз в начале процесса.
-  `run(mini_batch)`: Эта функция будет выполняться для каждого экземпляра `mini_batch`.
    -  `mini_batch`: `ParallelRunStep` вызывает метод run и передает ему в качестве аргумента список либо Pandas `DataFrame`. Каждая запись в mini_batch содержит одно из следующих значений: путь к файлу для входных данных в формате `FileDataset` или Pandas `DataFrame` для входных данных в формате `TabularDataset`.
    -  `response`: метод run() должен возвращать Pandas `DataFrame` или массив. Для append_row output_action эти возвращаемые элементы добавляются в общий выходной файл. Для summary_only содержимое элементов игнорируется. Для всех выходных действий каждый возвращаемый элемент обозначает один успешный запуск входного элемента во входном мини-пакете. Убедитесь в том, что в результат выполнения включено достаточно данных, чтобы сопоставить входные данные с результатом вывода. Выходные данные будут записаны в выходной файл, и для них не гарантируется правильный порядок. Для сопоставления со входными данными нужно использовать какой-либо ключ.

```python
%%writefile digit_identification.py
# Snippets from a sample script.
# Refer to the accompanying digit_identification.py
# (https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/machine-learning-pipelines/parallel-run)
# for the implementation script.

import os
import numpy as np
import tensorflow as tf
from PIL import Image
from azureml.core import Model


def init():
    global g_tf_sess

    # Pull down the model from the workspace
    model_path = Model.get_model_path("mnist")

    # Construct a graph to execute
    tf.reset_default_graph()
    saver = tf.train.import_meta_graph(os.path.join(model_path, 'mnist-tf.model.meta'))
    g_tf_sess = tf.Session()
    saver.restore(g_tf_sess, os.path.join(model_path, 'mnist-tf.model'))


def run(mini_batch):
    print(f'run method start: {__file__}, run({mini_batch})')
    resultList = []
    in_tensor = g_tf_sess.graph.get_tensor_by_name("network/X:0")
    output = g_tf_sess.graph.get_tensor_by_name("network/output/MatMul:0")

    for image in mini_batch:
        # Prepare each image
        data = Image.open(image)
        np_im = np.array(data).reshape((1, 784))
        # Perform inference
        inference_result = output.eval(feed_dict={in_tensor: np_im}, session=g_tf_sess)
        # Find the best probability, and add it to the result list
        best_result = np.argmax(inference_result)
        resultList.append("{}: {}".format(os.path.basename(image), best_result))

    return resultList
```

Если у вас есть другой файл или папка в том же каталоге, что и скрипт вывода, можно сослаться на него, найдя текущий рабочий каталог.

```python
script_dir = os.path.realpath(os.path.join(__file__, '..',))
file_path = os.path.join(script_dir, "<file_name>")
```

### <a name="parameters-for-parallelrunconfig"></a>Параметры для Параллелрунконфиг

`ParallelRunConfig` — это основная конфигурация для экземпляра `ParallelRunStep` в конвейере Машинного обучения Azure. Он пригодится вам как оболочка скрипта для настройки необходимых параметров, включая перечисленные ниже записи:
- `entry_script`: Локальный путь к пользовательскому скрипту, который будет выполняться параллельно на нескольких узлах. Если присутствует `source_directory`, используйте относительный путь. В противном случае используйте любой путь, доступный на компьютере.
- `mini_batch_size`: Размер мини-пакета, который передается в одном вызове `run()`. (Необязательный параметр; по умолчанию заданы `10` файлов для `FileDataset` и `1MB` для `TabularDataset`.)
    - Для `FileDataset` здесь указывается количество файлов; минимальное допустимое значение — `1`. Несколько файлов можно объединить в один мини-пакет.
    - Для `TabularDataset` здесь указывается размер данных. Примеры допустимых значений: `1024`, `1024KB`, `10MB` и `1GB`. Мы рекомендуем использовать значение `1MB`. Мини-пакет из `TabularDataset` никогда не пересекает границы файлов. Предположим, что у вас есть CSV-файлы с разными размерами в пределах от 100 КБ до 10 МБ. Если задать `mini_batch_size = 1MB`, все файлы с размером меньше 1 МБ будут рассматриваться как один мини-пакет. Файлы с размером, превышающим 1 МБ, будут разбиты на несколько мини-пакетов.
- `error_threshold`: Количество ошибок записи для `TabularDataset` и сбоев чтения файлов для `FileDataset`, которые следует игнорировать во время обработки. Если общее количество ошибок для всего объема входных данных превысит это значение, задание будет прервано. Пороговое количество ошибок применяется к общему объему входных данных, а не к отдельному мини-пакету, которые передаются в метод `run()`. Используется диапазон `[-1, int.max]`. Часть `-1` указывает на то, что следует игнорировать все сбои во время обработки.
- `output_action`: Одно из следующих значений указывает, как будут организованы выходные данные.
    - `summary_only`: Выходные данные сохраняются в пользовательском скрипте. `ParallelRunStep` использует выходные данные только для вычисления порога ошибок.
    - `append_row`: Для всех входных данных в выходной папке будет создан только один файл, куда будут добавляться все выходные данные, разделенные пустой строкой.
- `append_row_file_name`: Чтобы настроить имя выходного файла для append_row output_action (необязательно; значение по умолчанию — `parallel_run_step.txt`).
- `source_directory`: Пути к папкам, которые содержат все файлы для выполнения в целевом объекте вычислений (необязательно).
- `compute_target`: Поддерживается только `AmlCompute`.
- `node_count`: Количество вычислительных узлов, которые будут использоваться для выполнения пользовательского скрипта.
- `process_count_per_node`: Количество процессов на каждом узле. Рекомендуется устанавливать в значение, равное количеству GPU или ЦП на одном узле (необязательно; значение по умолчанию — `1`).
- `environment`: Определение среды Python. Вы можете настроить использование существующей среды Python или временной среды. Также это определение может задавать необходимые зависимости приложения (необязательно).
- `logging_level`: Детализация журнала. Значения уровня детализации в порядке увеличения: `WARNING`, `INFO` и `DEBUG`. (необязательный параметр; по умолчанию используется значение `INFO`.)
- `run_invocation_timeout`: Время вызова метода `run()` в секундах. (необязательный параметр, значение по умолчанию — `60`)
- `run_max_try`: Максимальное число попыток `run()` для mini-batch. Сбой `run()` при возникновении исключения или если при достижении `run_invocation_timeout` ничего не возвращается (необязательно; значение по умолчанию — `3`). 

Можно указать `mini_batch_size`, `node_count`, `process_count_per_node`, `logging_level`, `run_invocation_timeout` и `run_max_try` как `PipelineParameter`, чтобы при повторной отправке запуска конвейера можно было точно настроить значения параметров. В этом примере используется `PipelineParameter` для `mini_batch_size` и `Process_count_per_node`, и эти значения будут изменены при повторной отправке позже. 

### <a name="parameters-for-creating-the-parallelrunstep"></a>Параметры для создания Параллелрунстеп

Создайте ParallelRunStep, используя скрипт, конфигурацию среды и параметры. Укажите целевой объект вычислений, который уже подключен к рабочей области, в качестве целевого объекта для выполнения скрипта вывода. Используйте `ParallelRunStep`, чтобы создать шаг конвейера пакетного вывода, который принимает все следующие параметры.
- `name`: Имя шага со следующими ограничениями на именование: уникальность, от 3 до 32 символов, соответствие регулярному выражению ^\[a-z\]([-a-z0-9]*[a-z0-9])?$.
- `parallel_run_config`: Объект `ParallelRunConfig`, как определено ранее.
- `inputs`: Один или несколько однотипных наборов данных Машинного обучения Azure, которые должны быть секционированы для параллельной обработки.
- `side_inputs`: Один или несколько эталонных данных или наборов данных, используемых в качестве дополнительных входных данных без необходимости секционирования.
- `output`: `OutputFileDatasetConfig` Объект, представляющий путь к каталогу, в котором будут храниться выходные данные.
- `arguments`: Список аргументов, переданных в пользовательский скрипт. Используйте unknown_args, чтобы получить их в начальном сценарии (необязательно).
- `allow_reuse`: Указывает, должен ли шаг повторно использовать предыдущие результаты при запуске с теми же параметрами и входными данными. Если этот параметр имеет значение `False`, то во время выполнения конвейера для этого шага всегда будет создаваться новый запуск. (необязательный параметр; по умолчанию используется значение `True`.)

```python
from azureml.pipeline.steps import ParallelRunStep

parallelrun_step = ParallelRunStep(
    name="predict-digits-mnist",
    parallel_run_config=parallel_run_config,
    inputs=[input_mnist_ds_consumption],
    output=output_dir,
    allow_reuse=True
)
```

## <a name="debugging-scripts-from-remote-context"></a>Отладка сценариев из удаленного контекста

Переход от локальной отладки сценария оценки к отладке сценария оценки в фактическом конвейере может оказаться сложной задачей. Сведения о поиске журналов на портале см. в  [разделе конвейеры машинного обучения статьи Отладка скриптов из удаленного контекста](how-to-debug-pipelines.md). Сведения в этом разделе также применимы к классу ParallelRunStep.

Например, файл журнала `70_driver_log.txt` содержит сведения о контроллере, который запускает код ParallelRunStep.

Из-за того, что задания ParallelRunStep имеют распределенный характер, журналы могут поступать из нескольких разных источников. Однако создается два консолидированных файла, которые предоставляют общие сведения.

- `~/logs/job_progress_overview.txt`: Этот файл содержит общие сведения о количестве мини-пакетов (т. н. задач), созданных до этого момента, и количестве мини-пакетов, обработанных на данный момент. В этом случае отображается результат задачи. Если задание завершилось с ошибкой, отобразится сообщение об ошибке, а также рекомендации по началу устранения неполадок.

- `~/logs/sys/master_role.txt`: Этот файл содержит основной узел (также известный как Orchestrator) для выполняемого задания. Включает создание задач, мониторинг хода выполнения, результат выполнения.

Журналы, созданные на основе начального сценария с помощью вспомогательного метода EntryScript и операторов Print, находятся следующих файлах:

- `~/logs/user/entry_script_log/<ip_address>/<process_name>.log.txt`: Эти файлы являются журналами, записанными из entry_script с помощью вспомогательного приложения Ентрискрипт.

- `~/logs/user/stdout/<ip_address>/<process_name>.stdout.txt`: Эти файлы являются журналами из stdout (например, инструкции Print) entry_script.

- `~/logs/user/stderr/<ip_address>/<process_name>.stderr.txt`: Эти файлы являются журналами из stderr entry_script.

Следующие признаки позволяют быстро распознать ошибки в сценарии.

- `~/logs/user/error.txt`: Этот файл попытается обобщить ошибки в сценарии.

Дополнительные сведения об ошибках в сценарии:

- `~/logs/user/error/`: Содержит полные трассировки стека исключений, возникших при загрузке и выполнении скрипта записи.

Если необходимо в полной мере оценить, как сценарий оценки выполняется на каждом из узлов, просмотрите отдельные журналы процесса для каждого узла. Журналы процесса можно найти в папке `sys/node`. Они сгруппированы по рабочим узлам:

- `~/logs/sys/node/<ip_address>/<process_name>.txt`: Этот файл содержит подробные сведения о каждом мини-пакете по мере его комплектации или завершения рабочими процессами. Для каждого мини-пакета этот файл содержит:

    - IP-адрес и идентификатор рабочего процесса. 
    - Общее число элементов, число успешно обработанных элементов и число элементов, обработка которых завершилась сбоем.
    - Время начала, продолжительность, время обработки и время выполнения метода.

Можно также просмотреть результаты периодических проверок использования ресурсов для каждого узла. Файлы журнала и файлы установки находятся в этой папке:

- `~/logs/perf`: Задайте `--resource_monitor_interval` для изменения интервала проверки в секундах. По умолчанию используется интервал `600` , который составляет примерно 10 минут. Чтобы прерывать наблюдение, задайте значение `0` . Каждая `<ip_address>` Папка включает:

    - `os/`: Сведения обо всех выполняющихся процессах в узле. Одна проверка запускает команду операционной системы и сохраняет результат в файл. В Linux команда имеет значение `ps` . В Windows используйте `tasklist` .
        - `%Y%m%d%H`: Имя вложенной папки — это время в час.
            - `processes_%M`: Файл заканчивается в течение минуты времени проверки.
    - `node_disk_usage.csv`: Подробные сведения об использовании диска узлом.
    - `node_resource_usage.csv`: Общие сведения об использовании ресурсов узла.
    - `processes_resource_usage.csv`: Общие сведения об использовании ресурсов для каждого процесса.

### <a name="how-do-i-log-from-my-user-script-from-a-remote-context"></a>Ведение журнала из пользовательского сценария в удаленном контексте

Параллелрунстеп может запускать несколько процессов на одном узле на основе process_count_per_node. Чтобы упорядочить журналы из каждого процесса на узле и объединить инструкции Print и log, рекомендуется использовать средство ведения журнала Параллелрунстеп, как показано ниже. Вы получаете средство ведения журнала из Ентрискрипт и записываете журналы в папку **журналы/пользователь** на портале.

**Пример начального сценария с использованием средства ведения журнала:**
```python
from azureml_user.parallel_run import EntryScript

def init():
    """ Initialize the node."""
    entry_script = EntryScript()
    logger = entry_script.logger
    logger.debug("This will show up in files under logs/user on the Azure portal.")


def run(mini_batch):
    """ Accept and return the list back."""
    # This class is in singleton pattern and will return same instance as the one in init()
    entry_script = EntryScript()
    logger = entry_script.logger
    logger.debug(f"{__file__}: {mini_batch}.")
    ...

    return mini_batch
```

### <a name="how-could-i-pass-a-side-input-such-as-a-file-or-files-containing-a-lookup-table-to-all-my-workers"></a>Передача всем рабочим ролям сторонних входных данных, таких как файлы, содержащие таблицу подстановки.

Пользователь может передать ссылочные данные в скрипт, используя параметр side_inputs Параллерунстеп. Все наборы данных, предоставленные как side_inputs, будут подключены к каждому рабочему узлу. Пользователь может получить расположение Mount, передав аргумент.

Создайте [набор данных](/python/api/azureml-core/azureml.core.dataset.dataset) , содержащий эталонные данные, и зарегистрируйте его в рабочей области. Передайте его в параметр `side_inputs` вашего `ParallelRunStep`. Кроме того, можно добавить путь в раздел, `arguments` чтобы легко получить доступ к подключенному пути:

```python
label_config = label_ds.as_named_input("labels_input")
batch_score_step = ParallelRunStep(
    name=parallel_step_name,
    inputs=[input_images.as_named_input("input_images")],
    output=output_dir,
    arguments=["--labels_dir", label_config],
    side_inputs=[label_config],
    parallel_run_config=parallel_run_config,
)
```

После этого доступ к нему можно будет получить в сценарии вывода (например, в методе init ()) следующим образом:

```python
parser = argparse.ArgumentParser()
parser.add_argument('--labels_dir', dest="labels_dir", required=True)
args, _ = parser.parse_known_args()

labels_path = args.labels_dir
```

### <a name="how-to-use-input-datasets-with-service-principal-authentication"></a>Как использовать входные наборы данных с проверкой подлинности субъекта-службы?

Пользователь может передавать входные наборы данных с проверкой подлинности субъекта-службы в рабочей области. Использование такого набора данных в Параллелрунстеп требует регистрации набора данных для создания конфигурации Параллелрунстеп.

```python
service_principal = ServicePrincipalAuthentication(
    tenant_id="***",
    service_principal_id="***",
    service_principal_password="***")
 
ws = Workspace(
    subscription_id="***",
    resource_group="***",
    workspace_name="***",
    auth=service_principal
    )
 
default_blob_store = ws.get_default_datastore() # or Datastore(ws, '***datastore-name***') 
ds = Dataset.File.from_files(default_blob_store, '**path***')
registered_ds = ds.register(ws, '***dataset-name***', create_new_version=True)
```

## <a name="next-steps"></a>Дальнейшие действия

* См. следующие [записные книжки Jupyter, демонстрирующие конвейеры машинное обучение Azure](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/machine-learning-pipelines)

* Сведения о пакете [azureml-Pipeline-пошаговые инструкции](/python/api/azureml-pipeline-steps/azureml.pipeline.steps) см. в справочнике по пакету SDK. Просмотрите справочную [документацию](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.parallelrunstep) по классу параллелрунстеп.

* Ознакомьтесь с [дополнительным руководством](tutorial-pipeline-batch-scoring-classification.md) по использованию конвейеров с параллелрунстеп. В этом руководстве показано, как передать другой файл в качестве побочного входа.
