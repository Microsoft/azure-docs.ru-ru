---
title: Руководство. Запуск параллельной рабочей нагрузки с помощью API Python
description: Руководство. Обработка медиафайлов параллельно с ffmpeg в пакетной службе с помощью клиентской библиотеки пакетной службы для Python
ms.devlang: python
ms.topic: tutorial
ms.date: 11/29/2018
ms.custom: mvc, devx-track-python
ms.openlocfilehash: 48df2cb6ac103d1f03d2d64fce0d10a8f2f128e2
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105043416"
---
# <a name="tutorial-run-a-parallel-workload-with-azure-batch-using-the-python-api"></a>Руководство по Запуск параллельной рабочей нагрузки с помощью пакета Azure с использованием Python API

Используйте пакетную службу Azure, чтобы эффективно выполнять пакетные задания для крупномасштабных параллельных и высокопроизводительных вычислений (HPC). В этом руководстве рассматривается пример Python для запуска параллельной рабочей нагрузки с помощью пакетной службы Azure. Вы изучите общий рабочий процесс приложения пакетной службы и узнаете, как программно взаимодействовать с ресурсами пакетной службы и службы хранилища. Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Проверка подлинности с помощью учетных записей хранения и пакетной службы.
> * Передача входных файлов в хранилище.
> * Создание пула вычислительных узлов для запуска приложения.
> * Создание задания и задач для обработки входных файлов.
> * Мониторинг выполнения задач.
> * Извлечение выходных файлов

В этом руководстве вы преобразовываете медиафайлы формата MP4 в формат MP3 с помощью инструмента с открытым кодом [ffmpeg](https://ffmpeg.org/). 

[!INCLUDE [quickstarts-free-trial-note.md](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

* [Python 2.7, 3.6 или более поздней версии](https://www.python.org/downloads/)

* Диспетчер пакетов [pip](https://pip.pypa.io/en/stable/installing/).

* учетная запись пакетной службы Azure и связанная учетная запись службы хранилища Azure. Чтобы создать эти учетные записи, см. примеры быстрого начала работы с пакетной службой с помощью [портала Azure](quick-create-portal.md) или [Azure CLI](quick-create-cli.md).

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите на портал Azure по адресу [https://portal.azure.com](https://portal.azure.com).

[!INCLUDE [batch-common-credentials](../../includes/batch-common-credentials.md)] 

## <a name="download-and-run-the-sample"></a>Скачивание и запуск примера приложения

### <a name="download-the-sample"></a>Скачивание примера приложения

[Скачайте или клонируйте пример приложения](https://github.com/Azure-Samples/batch-python-ffmpeg-tutorial) с GitHub. Чтобы клонировать пример репозитория приложения с клиентом Git, выполните следующую команду:

```
git clone https://github.com/Azure-Samples/batch-python-ffmpeg-tutorial.git
```

Перейдите в каталог, в котором содержится файл `batch_python_tutorial_ffmpeg.py`.

В среде Python установите необходимые пакеты с помощью `pip`.

```bash
pip install -r requirements.txt
```

Откройте файл `config.py`. Обновите строки учетных данных учетной записи пакетной службы и учетной записи хранения со значениями, уникальными для ваших учетных записей. Пример:


```Python
_BATCH_ACCOUNT_NAME = 'mybatchaccount'
_BATCH_ACCOUNT_KEY = 'xxxxxxxxxxxxxxxxE+yXrRvJAqT9BlXwwo1CwF+SwAYOxxxxxxxxxxxxxxxx43pXi/gdiATkvbpLRl3x14pcEQ=='
_BATCH_ACCOUNT_URL = 'https://mybatchaccount.mybatchregion.batch.azure.com'
_STORAGE_ACCOUNT_NAME = 'mystorageaccount'
_STORAGE_ACCOUNT_KEY = 'xxxxxxxxxxxxxxxxy4/xxxxxxxxxxxxxxxxfwpbIC5aAWA8wDu+AFXZB827Mt9lybZB1nUcQbQiUrkPtilK5BQ=='
```

### <a name="run-the-app"></a>Запустите приложение

Выполните следующее, чтобы запустить этот сценарий.

```
python batch_python_tutorial_ffmpeg.py
```

Когда вы запустите пример приложения, консоль будет выглядеть так. Во время выполнения может возникнуть пауза на этапе `Monitoring all tasks for 'Completed' state, timeout in 00:30:00...`, когда будут запускаться вычислительные узлы пула. 
   
```
Sample start: 11/28/2018 3:20:21 PM

Container [input] created.
Container [output] created.
Uploading file LowPriVMs-1.mp4 to container [input]...
Uploading file LowPriVMs-2.mp4 to container [input]...
Uploading file LowPriVMs-3.mp4 to container [input]...
Uploading file LowPriVMs-4.mp4 to container [input]...
Uploading file LowPriVMs-5.mp4 to container [input]...
Creating pool [LinuxFFmpegPool]...
Creating job [LinuxFFmpegJob]...
Adding 5 tasks to job [LinuxFFmpegJob]...
Monitoring all tasks for 'Completed' state, timeout in 00:30:00...
Success! All tasks completed successfully within the specified timeout period.
Deleting container [input]....

Sample end: 11/28/2018 3:29:36 PM
Elapsed time: 00:09:14.3418742
```

Перейдите в учетную запись пакетной службы на портале Azure, чтобы отследить пул, вычислительные узлы, задания и задачи. Например, чтобы увидеть тепловую карту вычислительных узлов в вашем пуле, выберите **Пулы** > *LinuxFFmpegPool*.

Во время выполнения задач тепловая карта выглядит следующим образом:

![Тепловая карта пула](./media/tutorial-parallel-python/pool.png)

Обычное время выполнения — **примерно 5 минут**, если для приложения задана конфигурация по умолчанию. Для создания пула потребуется больше всего времени. 

[!INCLUDE [batch-common-tutorial-download](../../includes/batch-common-tutorial-download.md)]

## <a name="review-the-code"></a>Просмотр кода

В следующих разделах мы разобьем пример приложения на действия, выполняемые для обработки рабочей нагрузки в пакетной службе. Во время работы с оставшейся частью статьи сверяйтесь с кодом Python, так как в примере рассматриваются не все строки кода.

### <a name="authenticate-blob-and-batch-clients"></a>Проверка подлинности клиентов больших двоичных объектов и пакетной службы

Для взаимодействия с учетной записью хранения приложение использует пакет [azure-storage-blob](https://pypi.python.org/pypi/azure-storage-blob) для создания объекта [BlockBlobService](/python/api/azure-storage-blob/azure.storage.blob.blockblobservice.blockblobservice).

```python
blob_client = azureblob.BlockBlobService(
    account_name=_STORAGE_ACCOUNT_NAME,
    account_key=_STORAGE_ACCOUNT_KEY)
```

Приложение создает объект [BatchServiceClient](/python/api/azure.batch.batchserviceclient) для создания пулов, заданий и задач в пакетной службе, а также для управления ими. В примере клиент пакетной службы использует проверку подлинности с общим ключом. Пакетная служба Azure также поддерживает проверку подлинности отдельных пользователей или приложений с помощью [Azure Active Directory](batch-aad-auth.md).

```python
credentials = batchauth.SharedKeyCredentials(_BATCH_ACCOUNT_NAME,
                                             _BATCH_ACCOUNT_KEY)

batch_client = batch.BatchServiceClient(
    credentials,
    base_url=_BATCH_ACCOUNT_URL)
```

### <a name="upload-input-files"></a>Передача входных файлов

Приложение использует ссылку `blob_client`, создавая контейнер хранения для входных файлов MP4 и контейнер для выходных данных задач. Затем оно вызывает функцию `upload_file_to_container` для передачи файлов MP4 в локальный каталог `InputFiles` в контейнер. Файлы в хранилище определяются как объекты пакетной службы [ResourceFile](/python/api/azure-batch/azure.batch.models.resourcefile), которые она может впоследствии скачать на вычислительные узлы.

```python
blob_client.create_container(input_container_name, fail_on_exist=False)
blob_client.create_container(output_container_name, fail_on_exist=False)
input_file_paths = []

for folder, subs, files in os.walk(os.path.join(sys.path[0], './InputFiles/')):
    for filename in files:
        if filename.endswith(".mp4"):
            input_file_paths.append(os.path.abspath(
                os.path.join(folder, filename)))

# Upload the input files. This is the collection of files that are to be processed by the tasks.
input_files = [
    upload_file_to_container(blob_client, input_container_name, file_path)
    for file_path in input_file_paths]
```

### <a name="create-a-pool-of-compute-nodes"></a>Создание пула вычислительных узлов

Затем в учетной записи пакетной службы создается пул вычислительных узлов с помощью вызова `create_pool`. Определенная функция использует класс пакетной службы [PoolAddParameter](/python/api/azure-batch/azure.batch.models.pooladdparameter) для настройки количества узлов, размера виртуальной машины и конфигурации пула. Объект [VirtualMachineConfiguration](/python/api/azure-batch/azure.batch.models.virtualmachineconfiguration) указывает [ImageReference](/python/api/azure-batch/azure.batch.models.imagereference) в образе Ubuntu Server 18.04 LTS, опубликованном в Azure Marketplace. Пакетная служба Azure поддерживает широкий спектр образов виртуальной машины в Azure Marketplace, а также пользовательских образов виртуальной машины.

Количество узлов и размер виртуальной машины настраиваются с помощью определенных констант. Пакетная служба Azure поддерживает выделенные узлы и узлы [с низким приоритетом](batch-low-pri-vms.md). Вы можете использовать их в своих пулах. Выделенные узлы зарезервированы для пула. Низкоприоритетные узлы предлагаются по сниженной цене с учетом избыточных ресурсов виртуальной машины в Azure. Эти узлы становятся недоступны, если в Azure недостаточно ресурсов. Пример по умолчанию создает пул, содержащий только 5 низкоприоритетных узлов размером *Standard_A1_v2*. 

Кроме свойств физических узлов эта конфигурация пула включает объект [StartTask](/python/api/azure-batch/azure.batch.models.starttask). Задача StartTask выполняется на каждом узле по мере его присоединения к пулу, а также при каждом перезапуске узла. В этом примере StartTask запускает команды оболочки Bash для установки пакета ffmpeg и зависимостей с узлами.

Метод [pool.add](/python/api/azure-batch/azure.batch.operations.pooloperations) отправляет пул в пакетную службу.

```python
new_pool = batch.models.PoolAddParameter(
    id=pool_id,
    virtual_machine_configuration=batchmodels.VirtualMachineConfiguration(
        image_reference=batchmodels.ImageReference(
            publisher="Canonical",
            offer="UbuntuServer",
            sku="18.04-LTS",
            version="latest"
        ),
        node_agent_sku_id="batch.node.ubuntu 18.04"),
    vm_size=_POOL_VM_SIZE,
    target_dedicated_nodes=_DEDICATED_POOL_NODE_COUNT,
    target_low_priority_nodes=_LOW_PRIORITY_POOL_NODE_COUNT,
    start_task=batchmodels.StartTask(
        command_line="/bin/bash -c \"apt-get update && apt-get install -y ffmpeg\"",
        wait_for_success=True,
        user_identity=batchmodels.UserIdentity(
            auto_user=batchmodels.AutoUserSpecification(
                scope=batchmodels.AutoUserScope.pool,
                elevation_level=batchmodels.ElevationLevel.admin)),
    )
)
batch_service_client.pool.add(new_pool)
```

### <a name="create-a-job"></a>Создание задания

Пакетное задание указывает пул для запуска задач и дополнительные параметры, такие как приоритет и расписание работы. Пример создает задание путем вызова `create_job`. Определенная функция использует класс [JobAddParameter](/python/api/azure-batch/azure.batch.models.jobaddparameter) для создания задания в пуле. Метод [job.add](/python/api/azure-batch/azure.batch.operations.joboperations) отправляет пул в пакетную службу. Изначально у задания нет задач.

```python
job = batch.models.JobAddParameter(
    id=job_id,
    pool_info=batch.models.PoolInformation(pool_id=pool_id))

batch_service_client.job.add(job)
```

### <a name="create-tasks"></a>Создание задач

Приложение создает задачи в задании путем вызова `add_tasks`. Эта определенная функция создает список объектов задачи с помощью класса [TaskAddParameter](/python/api/azure-batch/azure.batch.models.taskaddparameter). Каждая задача запускает ffmpeg для обработки объекта входных данных `resource_files` с помощью параметра `command_line`. ffmpeg был ранее установлен на каждом узле при создании пула. В командной строке выполняется ffmpeg для преобразования каждого входного файла MP4 (видео) в файл MP3 (аудио).

В примере создается объект [OutputFile](/python/api/azure-batch/azure.batch.models.outputfile) для файла MP3 после запуска командной строки. Выходные файлы каждой задачи (в этом случае один) загружаются в контейнер в связанной учетной записи с использованием свойства задачи `output_files`.

Затем приложение добавляет задачи к заданию с помощью метода [task.add_collection](/python/api/azure-batch/azure.batch.operations.taskoperations), который ставит их в очередь для запуска на вычислительных узлах. 

```python
tasks = list()

for idx, input_file in enumerate(input_files):
    input_file_path = input_file.file_path
    output_file_path = "".join((input_file_path).split('.')[:-1]) + '.mp3'
    command = "/bin/bash -c \"ffmpeg -i {} {} \"".format(
        input_file_path, output_file_path)
    tasks.append(batch.models.TaskAddParameter(
        id='Task{}'.format(idx),
        command_line=command,
        resource_files=[input_file],
        output_files=[batchmodels.OutputFile(
            file_pattern=output_file_path,
            destination=batchmodels.OutputFileDestination(
                container=batchmodels.OutputFileBlobContainerDestination(
                    container_url=output_container_sas_url)),
            upload_options=batchmodels.OutputFileUploadOptions(
                upload_condition=batchmodels.OutputFileUploadCondition.task_success))]
    )
    )
batch_service_client.task.add_collection(job_id, tasks)
```

### <a name="monitor-tasks"></a>Мониторинг задач

После добавления задач к заданию пакетная служба автоматически ставит в очередь и назначает их для выполнения на вычислительных узлах в связанном пуле. Пакетная служба обрабатывает постановку задач в очередь, их планирование, повтор и другие задачи администрирования с учетом указанных параметров. 

Есть несколько подходов к отслеживанию выполнения задач. В этом примере функция `wait_for_tasks_to_complete` использует объект [TaskState](/python/api/azure-batch/azure.batch.models.taskstate) для отслеживания определенного состояния задач, в данном случае завершения в течение указанного срока.

```python
while datetime.datetime.now() < timeout_expiration:
    print('.', end='')
    sys.stdout.flush()
    tasks = batch_service_client.task.list(job_id)

    incomplete_tasks = [task for task in tasks if
                        task.state != batchmodels.TaskState.completed]
    if not incomplete_tasks:
        print()
        return True
    else:
        time.sleep(1)
...
```

## <a name="clean-up-resources"></a>Очистка ресурсов

После выполнения задач приложение автоматически удаляет созданный входной контейнер хранилища, а также предоставляет возможность удалить пул и задания пакетной службы. Классы [JobOperations](/python/api/azure-batch/azure.batch.operations.joboperations) и [PoolOperations BatchClient](/python/api/azure-batch/azure.batch.operations.pooloperations) предусматривают методы удаления, которые вызываются, если подтвердить удаление. Вы не оплачиваете задания и задачи, но платите за используемые вычислительные узлы. Поэтому рекомендуется выделять пулы только при необходимости. При удалении пула удаляются все выходные данные задачи на узлах. Однако входные и выходные файлы сохраняются в учетной записи хранения.

Ставшие ненужными группу ресурсов, учетную запись пакетной службы и учетную запись хранения можно удалить. Для этого на портале Azure выберите группу ресурсов для учетной записи пакетной службы и щелкните **Удалить группу ресурсов**.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> * Проверка подлинности с помощью учетных записей хранения и пакетной службы.
> * Передача входных файлов в хранилище.
> * Создание пула вычислительных узлов для запуска приложения.
> * Создание задания и задач для обработки входных файлов.
> * Мониторинг выполнения задач.
> * Извлечение выходных файлов

Дополнительные примеры использования Python API для планирования и обработки рабочих нагрузок пакетной службы см. в примерах на сайте GitHub.

> [!div class="nextstepaction"]
> [Образцы пакетной службы Python](https://github.com/Azure/azure-batch-samples/tree/master/Python/Batch)

