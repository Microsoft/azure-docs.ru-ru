---
title: Руководство по началу работы с машинным обучением — Python
titleSuffix: Azure Machine Learning
description: В этом учебнике описано начало работы с пакетом SDK Машинного обучения Azure для Python, запущенным в вашей личной среде разработки.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: aminsaied
ms.author: amsaied
ms.reviewer: sgilley
ms.date: 02/11/2021
ms.custom: devx-track-python, contperf-fy21q3
adobe-target: true
ms.openlocfilehash: 81b9b5d53e1b61322d25d5dead85429474c1272c
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105935555"
---
# <a name="tutorial-get-started-with-azure-machine-learning-in-your-development-environment-part-1-of-4"></a>Руководство по началу работы с Машинным обучением Azure в среде разработки (часть 1 из 4)

В рамках этой *серии учебников, состоящей из четырех частей*, вы изучите основы Машинного обучения Azure и выполните указанные ниже задачи машинного обучения для Python на основе заданий на облачной платформе Azure. 

В части 1 этой серии учебников вы выполните указанные ниже задачи.

> [!div class="checklist"]
> * Установка пакета SDK Машинного обучения Azure.
> * Настройка структуры каталогов для кода.
> * Создайте рабочую область машинного обучения Azure.
> * Настройка локальной среды разработки.
> * Настройка вычислительного кластера.

> [!NOTE]
> Эта серия руководств посвящена аспектам Машинного обучения Azure, связанным с отправкой **пакетных заданий**, когда код отправляется в облако для запуска в фоновом режиме без вмешательства пользователя. Это удобно при использования завершенных скриптов или кода, которые требуется запускать многократно, или для задач машинного обучения с интенсивным потреблением вычислительных ресурсов. Если вы больше заинтересованы в произвольном рабочем процессе, можно использовать [вычислительный экземпляр Jupyter или RStudio в Машинном обучении Azure](tutorial-1st-experiment-sdk-setup.md).

## <a name="prerequisites"></a>Обязательные условия

- Подписка Azure. Если у вас еще нет подписки Azure, создайте бесплатную учетную запись, прежде чем начинать работу. Попробуйте [Машинное обучение Azure](https://aka.ms/AMLFree).
- [Anaconda](https://www.anaconda.com/download/) или [Miniconda](https://www.anaconda.com/download/) для управления виртуальными средами Python и установки пакетов.  
- Если вы ранее не использовали Conda, см. статью [Начало работы с Conda](https://conda.io/projects/conda/en/latest/user-guide/getting-started.html).

## <a name="install-the-azure-machine-learning-sdk"></a>Установка пакета SDK Машинного обучения Azure.

В этом руководстве предполагается использование пакета SDK Машинного обучения Azure для Python. Чтобы избежать проблем с зависимостями Python, создайте изолированную среду. В этой серии руководств для создания такой среды предполагается использование Conda. Если вы предпочитаете другие решения, такие как `venv`, `virtualenv` или Docker, убедитесь, что используется версия Python не ниже 3.5 и не выше 3.9.

Проверьте, есть ли в системе установка Conda:
    
```bash
conda --version
```
    
Если эта команда возвращает ошибку `conda not found`, [скачайте и установите Miniconda](https://docs.conda.io/en/latest/miniconda.html). 

После установки Conda используйте терминал или окно командной строки Anaconda для создания новой среды:

```bash
conda create -n tutorial python=3.8
```

Затем установите пакет SDK Машинного обучения Azure в созданную среду Conda:

```bash
conda activate tutorial
pip install azureml-core
```
    
> [!NOTE]
> Завершение установки пакета SDK Машинного обучения Azure занимает примерно 2 минуты.
>
> Если возникает ошибка времени ожидания, воспользуйтесь `pip install --default-timeout=100 azureml-core`.


> [!div class="nextstepaction"]
> [Мной установлен пакет SDK](?success=install-sdk#dir) [Возникла проблема](https://www.research.net/r/7C8Z3DN?issue=install-sdk)

## <a name="create-a-directory-structure-for-code"></a><a name="dir"></a>Создание структуры каталогов для кода

Для работы с этим руководством рекомендуется настроить следующую простую структуру каталогов:

:::image type="content" source="media/tutorial-1st-experiment-sdk-local/directory-structure-1.png" alt-text="Структура каталога: верхний уровень для учебника с подкаталогом .azureml":::


- `tutorial`: каталог верхнего уровня проекта.
- `.azureml`: скрытый подкаталог для хранения файлов конфигурации Машинного обучения Azure.

Например, чтобы создать их в окне bash:

```bash
mkdir tutorial
cd tutorial
mkdir .azureml
```

> [!TIP]
> Чтобы создать или просмотреть структуру в окне с графическим представлением, сначала включите возможность просмотра и создания скрытых файлов и папок:
>
> * В окне поиска компьютера Mac используйте клавиши **CMD+SHIFT+.** для переключения отображения скрытых файлов и папок.  
> * В проводнике Windows 10 [включите просмотр скрытых файлов и папок](https://support.microsoft.com/en-us/windows/view-hidden-files-and-folders-in-windows-10-97fbc472-c603-9d90-91d0-1166d1d9f4b5). 
> * В графическом интерфейсе Linux используйте **CTRL+H** или меню **Представление** и установите флажок **Show hidden files** (Показать скрытые файлы).




> [!div class="nextstepaction"]
> [Мной создан каталог](?success=create-dir#workspace) [Возникла проблема](https://www.research.net/r/7C8Z3DN?issue=create-dir)

## <a name="create-an-azure-machine-learning-workspace"></a><a name="workspace"></a>Создание рабочей области машинного обучения Azure

Рабочая область — это ресурс верхнего уровня для Машинного обучения Azure и централизованное расположение для:

- управления ресурсами (например, вычислительными ресурсами);
- хранения ресурсов, таких как записные книжки, среды, наборы данных, конвейеры, модели и конечные точки;
- совместной работы с другими участниками команды.

В каталоге верхнего уровня `tutorial` добавьте новый файл Python с именем `01-create-workspace.py`, используя следующий код. Задайте параметры (имя, идентификатор подписки, группа ресурсов и [расположение](https://azure.microsoft.com/global-infrastructure/services/?products=machine-learning-service)) согласно своим предпочтениям.

Вы можете выполнить код в интерактивном сеансе или с помощью файла Python.

>[!NOTE]
> Если вы используете локальную среду разработки (например, свой компьютер), вам будет предложено пройти проверку подлинности в рабочей области с помощью *кода устройства* при первом запуске следующего кода. Следуйте инструкциям на экране.

```python
# tutorial/01-create-workspace.py
from azureml.core import Workspace

ws = Workspace.create(name='<my_workspace_name>', # provide a name for your workspace
                      subscription_id='<azure-subscription-id>', # provide your subscription ID
                      resource_group='<myresourcegroup>', # provide a resource group name
                      create_resource_group=True,
                      location='<NAME_OF_REGION>') # For example: 'westeurope' or 'eastus2' or 'westus2' or 'southeastasia'.

# write out the workspace details to a configuration file: .azureml/config.json
ws.write_config(path='.azureml')
```

В окне с активированной средой Conda *tutorial1* запустите этот код из каталога `tutorial`.

```bash
cd <path/to/tutorial>
python ./01-create-workspace.py
```

> [!TIP]
> Если выполнение этого кода приводит к ошибке с сообщением, что у вас нет доступа к подписке, см. раздел [Создание рабочей области](how-to-manage-workspace.md?tab=python#create-multi-tenant), чтобы получить сведения о параметрах проверки подлинности.


После успешного запуска *01-create-workspace.py* структура папок будет примерно такой:

:::image type="content" source="media/tutorial-1st-experiment-sdk-local/directory-structure-2.png" alt-text="Файл config.json появляется в подкаталоге .azureml после запуска 01-create-workspace.py":::

Файл `.azureml/config.json` содержит метаданные, необходимые для подключения к рабочей области Машинного обучения Azure. А именно, он содержит идентификатор подписки, группу ресурсов и имя рабочей области. 

> [!NOTE]
> Содержимое `config.json` не является секретным. Вы можете использовать эти сведения совместно.
>
> Для взаимодействия с рабочей областью Машинного обучения Azure также необходимо пройти проверку подлинности.

> [!div class="nextstepaction"]
> [Мной создана рабочая область](?success=create-workspace#cluster) [Возникла проблема](https://www.research.net/r/7C8Z3DN?issue=create-workspace)

## <a name="create-an-azure-machine-learning-compute-cluster"></a><a name="cluster"></a> Создание вычислительного кластера Машинного обучения Azure

Создайте скрипт Python в каталоге верхнего уровня `tutorial` с именем `02-create-compute.py`. Заполните его следующим кодом, чтобы создать кластер Машинного обучения Azure, который будет автоматически масштабироваться от нуля до четырех узлов:

```python
# tutorial/02-create-compute.py
from azureml.core import Workspace
from azureml.core.compute import ComputeTarget, AmlCompute
from azureml.core.compute_target import ComputeTargetException

ws = Workspace.from_config() # This automatically looks for a directory .azureml

# Choose a name for your CPU cluster
cpu_cluster_name = "cpu-cluster"

# Verify that the cluster does not exist already
try:
    cpu_cluster = ComputeTarget(workspace=ws, name=cpu_cluster_name)
    print('Found existing cluster, use it.')
except ComputeTargetException:
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2',
                                                           idle_seconds_before_scaledown=2400,
                                                           min_nodes=0,
                                                           max_nodes=4)
    cpu_cluster = ComputeTarget.create(ws, cpu_cluster_name, compute_config)

cpu_cluster.wait_for_completion(show_output=True)
```

В окне с активированной средой Conda *tutorial1* выполните файл Python:

```bash
python ./02-create-compute.py
```


> [!NOTE]
> После создания кластера для него будут подготовлены 0 узлов. За кластер *не* взимается плата, пока вы не отправите задание. Если этот кластер бездействует на протяжении 2400 секунд (40 минут), его размер уменьшается.

Теперь структура папок будет выглядеть следующим образом:

:::image type="content" source="media/tutorial-1st-experiment-sdk-local/directory-structure-3.png" alt-text="Добавление 02-create-compute.py в каталог для учебника":::

> [!div class="nextstepaction"]
> [Мной создан вычислительный кластер](?success=create-compute-cluster#next-steps) [Возникла проблема](https://www.research.net/r/7C8Z3DN?issue=create-compute-cluster)

## <a name="view-in-the-studio"></a>Просмотр в студии

Войдите в [Студию машинного обучения Azure](https://ml.azure.com), чтобы просмотреть созданные рабочую область и вычислительный экземпляр.

1. Выберите **подписку**, которую вы использовали для создания рабочей области.
1. Выберите созданную **рабочую область машинного обучения**: *tutorial-ws*.
1. После загрузки рабочей области слева выберите **Вычисления**.
1. Вверху выберите вкладку **Вычислительные кластеры**.

:::image type="content" source="media/tutorial-1st-experiment-sdk-local/compute-instance-in-studio.png" alt-text="Снимок экрана: просмотр вычислительного экземпляра в рабочей области.":::

В этом представлении отображается подготовленный вычислительный кластер, а также количество бездействующих, занятых и неподготовленных узлов.  Так как вы еще не использовали кластер, все узлы являются неподготовленными.

## <a name="next-steps"></a>Следующие шаги

В этом учебнике по установке вы выполнили следующие задачи:

- создали рабочую область Машинного обучения Azure;
- настройка локальной среды разработки;
- создание вычислительного кластера Машинного обучения Azure.

Другие разделы этого руководства посвящены следующему:

* Часть 2. Выполнение кода в облаке с помощью пакета SDK Машинного обучения Azure для Python.
* Часть 3. Управление средой Python, используемой для обучения модели.
* Часть 4. Отправка данных в Azure и их использование для обучения.

Перейдите к следующему руководству, чтобы ознакомиться с инструкциями по отправке скрипта в вычислительный кластер Машинного обучения Azure.

> [!div class="nextstepaction"]
> [Учебник по выполнению скрипта "Hello world!" в Azure](tutorial-1st-experiment-hello-world.md)
