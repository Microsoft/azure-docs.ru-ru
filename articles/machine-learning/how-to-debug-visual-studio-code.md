---
title: Интерактивная Отладка с помощью Visual Studio Code
titleSuffix: Azure Machine Learning
description: Отладка Машинное обучение Azure кода, конвейеров и развертываний в интерактивном режиме с помощью Visual Studio Code
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
author: luisquintanilla
ms.author: luquinta
ms.date: 09/30/2020
ms.openlocfilehash: 783b5afdaef369582614cde3525f7968fdb5e567
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102508645"
---
# <a name="interactive-debugging-with-visual-studio-code"></a>Интерактивная Отладка с помощью Visual Studio Code



Узнайте, как выполнять интерактивную отладку Машинное обучение Azure экспериментов, конвейеров и развертываний с помощью Visual Studio Code (VS Code) и [дебугпи](https://github.com/microsoft/debugpy/).

## <a name="run-and-debug-experiments-locally"></a>Запуск и отладка экспериментов в локальной среде

Используйте расширение Машинное обучение Azure для проверки, запуска и отладки экспериментов машинного обучения перед их отправкой в облако.

### <a name="prerequisites"></a>Предварительные требования

* Расширение Машинное обучение Azure VS Code (Предварительная версия). Дополнительные сведения см. в разделе [Настройка расширения Машинное обучение Azure VS Code](tutorial-setup-vscode-extension.md).
* [Docker](https://www.docker.com/get-started)
  * DOCKER Desktop для Mac и Windows
  * Подсистема DOCKER для Linux.
* [Python 3](https://www.python.org/downloads/);

> [!NOTE]
> В Windows не забудьте [настроить DOCKER для использования контейнеров Linux](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers).

> [!TIP]
> Для Windows, хотя и не является обязательным, настоятельно рекомендуется [использовать DOCKER с подсистемой Windows для Linux (WSL) 2](/windows/wsl/tutorials/wsl-containers#install-docker-desktop).

> [!IMPORTANT]
> Перед запуском эксперимента в локальной среде убедитесь, что DOCKER запущен.

### <a name="debug-experiment-locally"></a>Отладка эксперимента локально

1. В VS Code откройте представление расширение Машинное обучение Azure.
1. Разверните узел подписки, содержащий рабочую область. Если у вас ее еще нет, можно [создать машинное обучение Azure рабочую область](how-to-manage-resources-vscode.md#create-a-workspace) с помощью расширения.
1. Разверните узел рабочей области.
1. Щелкните правой кнопкой мыши узел **эксперименты** и выберите **создать эксперимент**. При появлении запроса введите имя для эксперимента.
1. Разверните узел **эксперименты** , щелкните правой кнопкой мыши эксперимент, который необходимо запустить, и выберите команду **запустить эксперимент**.
1. В списке параметров для запуска эксперимента выберите **локально**.
1. **В первый раз используйте только в Windows**. При появлении запроса на разрешение общей папки выберите **Да**. Включение файлового ресурса позволяет DOCKER подключать каталог, содержащий скрипт, к контейнеру. Кроме того, он позволяет DOCKER сохранять журналы и выходные данные из запуска во временном каталоге в системе.
1. Выберите **Да** для отладки эксперимента. В противном случае нажмите кнопку **Нет**. Если выбрать нет, ваш эксперимент будет выполняться локально без подключения к отладчику.
1. Выберите **создать новую конфигурацию запуска** , чтобы создать конфигурацию запуска. Конфигурация запуска определяет скрипт, который требуется запустить, зависимости и наборы данных. Кроме того, если у вас уже есть такая возможность, выберите ее в раскрывающемся списке.
    1. Выберите среду. Вы можете выбрать любой из [машинное обучение Azure проверенного](resource-curated-environments.md) или создать свой собственный.
    1. Укажите имя скрипта, который требуется запустить. Путь задается относительно каталога, открытого в VS Code.
    1. Выберите, следует ли использовать Машинное обучение Azure набор данных. Вы можете создавать [машинное обучение Azure наборы данных](how-to-manage-resources-vscode.md#create-dataset) с помощью расширения.
    1. Дебугпи требуется для подключения отладчика к контейнеру, выполняющему эксперимент. Чтобы добавить дебугпи в качестве зависимости, выберите **Добавить дебугпи**. В противном случае выберите **пропустить**. Не добавляя дебугпи в качестве зависимости, запускает ваш эксперимент без подключения к отладчику.
    1. В редакторе откроется файл конфигурации, содержащий параметры конфигурации запуска. Если вы удовлетворены параметрами, выберите **Отправить эксперимент**. Кроме того, можно открыть палитру команд (**представление > палитра команд**) в строке меню и ввести `Azure ML: Submit experiment` команду в текстовое поле.
1. После отправки эксперимента создается образ DOCKER, содержащий скрипт, и конфигурации, указанные в конфигурации запуска.

    Когда начинается процесс сборки образа DOCKER, содержимое `60_control_log.txt` файлового потока переводится в консоль вывода в VS Code.

    > [!NOTE]
    > При первом создании образа DOCKER может потребоваться несколько минут.

1. После сборки образа появится запрос на запуск отладчика. Задайте точки останова в скрипте и нажмите кнопку **запустить отладчик** , когда будете готовы начать отладку. Это присоединяет отладчик VS Code к контейнеру, выполняющему эксперимент. Кроме того, в расширении Машинное обучение Azure наведите указатель мыши на узел для текущего запуска и щелкните значок воспроизведения, чтобы запустить отладчик.

    > [!IMPORTANT]
    > Для одного эксперимента нельзя использовать несколько сеансов отладки. Однако можно выполнить отладку двух или более экспериментов, используя несколько экземпляров VS Code.

На этом этапе вы сможете пошагово и отлаживать код с помощью VS Code.

Если вы хотите отменить запуск в любой момент, щелкните правой кнопкой мыши узел выполнить и выберите команду **отменить запуск**.

Аналогично удаленному запуску эксперимента можно развернуть узел выполнения, чтобы проверить журналы и выходные данные.

> [!TIP]
> Образы DOCKER, использующие те же зависимости, определенные в вашей среде, используются повторно между запусками. Однако при запуске эксперимента с помощью новой или другой среды создается новый образ. Так как эти образы сохраняются в локальном хранилище, рекомендуется удалить старые или неиспользуемые образы DOCKER. Чтобы удалить образы из системы, используйте [DOCKER CLI](https://docs.docker.com/engine/reference/commandline/rmi/) или [расширение VS Code DOCKER](https://code.visualstudio.com/docs/containers/overview).

## <a name="debug-and-troubleshoot-machine-learning-pipelines"></a>Отладка и устранение неполадок в конвейерах машинного обучения

В некоторых случаях может потребоваться интерактивно отлаживать код Python, используемый в конвейере машинного обучения. С помощью VS Code и дебугпи можно присоединяться к коду, как он выполняется в среде обучения.

### <a name="prerequisites"></a>Предварительные требования

* __Рабочая область машинное обучение Azure__ , настроенная для использования __виртуальной сети Azure__.
* __Конвейер машинное обучение Azure__ , использующий скрипты Python в рамках этапов конвейера. Например, Писонскриптстеп.
* Машинное обучение Azureного кластерного кластера, который находится __в виртуальной сети__ и __используется конвейером для обучения__.
* __Среда разработки__ , которая находится __в виртуальной сети__. Среда разработки может быть одной из следующих:

  * Виртуальная машина Azure в виртуальной сети
  * Вычислительный экземпляр виртуальной машины записной книжки в виртуальной сети
  * Клиентский компьютер, имеющий подключение к виртуальной сети по частной сети либо по VPN, либо через ExpressRoute.

Дополнительные сведения об использовании виртуальной сети Azure с Машинное обучение Azure см. в статье [Общие сведения о изоляции и конфиденциальности виртуальной сети](how-to-network-security-overview.md).

> [!TIP]
> Хотя вы можете работать с Машинное обучение Azureными ресурсами, которые не находятся за виртуальной сетью, рекомендуется использовать виртуальную сеть.

### <a name="how-it-works"></a>Принцип работы

Этапы конвейера машинного обучения запускают скрипты Python. Эти скрипты изменяются для выполнения следующих действий.

1. Регистрировать IP-адрес узла, на котором они выполняются. Используйте IP-адрес для подключения отладчика к сценарию.

2. Запустите компонент отладки дебугпи и дождитесь подключения отладчика.

3. В среде разработки вы отслеживаете журналы, созданные процессом обучения, чтобы найти IP-адрес, на котором выполняется сценарий.

4. Вы указываете VS Code IP-адреса для подключения отладчика к с помощью `launch.json` файла.

5. Вы подключаете отладчик и интерактивно пройдите по сценарию.

### <a name="configure-python-scripts"></a>Настройка скриптов Python

Чтобы включить отладку, внесите следующие изменения в скрипты Python, используемые шагами в конвейере ML:

1. Добавьте следующие операторы импорта:

    ```python
    import argparse
    import os
    import debugpy
    import socket
    from azureml.core import Run
    ```

1. Добавьте следующие аргументы. Эти аргументы позволяют включить отладчик по мере необходимости и задать время ожидания для подключения отладчика:

    ```python
    parser.add_argument('--remote_debug', action='store_true')
    parser.add_argument('--remote_debug_connection_timeout', type=int,
                        default=300,
                        help=f'Defines how much time the AML compute target '
                        f'will await a connection from a debugger client (VSCODE).')
    parser.add_argument('--remote_debug_client_ip', type=str,
                        help=f'Defines IP Address of VS Code client')
    parser.add_argument('--remote_debug_port', type=int,
                        default=5678,
                        help=f'Defines Port of VS Code client')
    ```

1. Добавьте следующие инструкции. Эти инструкции загружают текущий контекст выполнения, чтобы можно было зарегистрировать IP-адрес узла, на котором выполняется код:

    ```python
    global run
    run = Run.get_context()
    ```

1. Добавьте `if` инструкцию, запускающую дебугпи, и дождитесь присоединения отладчика. Если отладчик не подключается до истечения времени ожидания, сценарий продолжится в нормальном режиме. Обязательно замените `HOST` значения, а `PORT` — `listen` собственной функцией.

    ```python
    if args.remote_debug:
        print(f'Timeout for debug connection: {args.remote_debug_connection_timeout}')
        # Log the IP and port
        try:
            ip = args.remote_debug_client_ip
        except:
            print("Need to supply IP address for VS Code client")
        print(f'ip_address: {ip}')
        debugpy.listen(address=(ip, args.remote_debug_port))
        # Wait for the timeout for debugger to attach
        debugpy.wait_for_client()
        print(f'Debugger attached = {debugpy.is_client_connected()}')
    ```

В следующем примере кода Python показан простой `train.py` файл, позволяющий выполнять отладку:

```python
# Copyright (c) Microsoft. All rights reserved.
# Licensed under the MIT license.

import argparse
import os
import debugpy
import socket
from azureml.core import Run

print("In train.py")
print("As a data scientist, this is where I use my training code.")

parser = argparse.ArgumentParser("train")

parser.add_argument("--input_data", type=str, help="input data")
parser.add_argument("--output_train", type=str, help="output_train directory")

# Argument check for remote debugging
parser.add_argument('--remote_debug', action='store_true')
parser.add_argument('--remote_debug_connection_timeout', type=int,
                    default=300,
                    help=f'Defines how much time the AML compute target '
                    f'will await a connection from a debugger client (VSCODE).')
parser.add_argument('--remote_debug_client_ip', type=str,
                    help=f'Defines IP Address of VS Code client')
parser.add_argument('--remote_debug_port', type=int,
                    default=5678,
                    help=f'Defines Port of VS Code client')

# Get run object, so we can find and log the IP of the host instance
global run
run = Run.get_context()

args = parser.parse_args()

# Start debugger if remote_debug is enabled
if args.remote_debug:
    print(f'Timeout for debug connection: {args.remote_debug_connection_timeout}')
    # Log the IP and port
    # ip = socket.gethostbyname(socket.gethostname())
    try:
        ip = args.remote_debug_client_ip
    except:
        print("Need to supply IP address for VS Code client")
    print(f'ip_address: {ip}')
    debugpy.listen(address=(ip, args.remote_debug_port))
    # Wait for the timeout for debugger to attach
    debugpy.wait_for_client()
    print(f'Debugger attached = {debugpy.is_client_connected()}')

print("Argument 1: %s" % args.input_data)
print("Argument 2: %s" % args.output_train)

if not (args.output_train is None):
    os.makedirs(args.output_train, exist_ok=True)
    print("%s created" % args.output_train)
```

### <a name="configure-ml-pipeline"></a>Настройка конвейера машинного обучения

Чтобы предоставить пакеты Python, необходимые для запуска дебугпи и получения контекста выполнения, создайте среду и задайте `pip_packages=['debugpy', 'azureml-sdk==<SDK-VERSION>']` . Измените версию пакета SDK, чтобы она соответствовала используемой. В следующем фрагменте кода показано, как создать среду.

```python
# Use a RunConfiguration to specify some additional requirements for this step.
from azureml.core.runconfig import RunConfiguration
from azureml.core.conda_dependencies import CondaDependencies
from azureml.core.runconfig import DEFAULT_CPU_IMAGE

# create a new runconfig object
run_config = RunConfiguration()

# enable Docker 
run_config.environment.docker.enabled = True

# set Docker base image to the default CPU-based image
run_config.environment.docker.base_image = DEFAULT_CPU_IMAGE

# use conda_dependencies.yml to create a conda environment in the Docker image for execution
run_config.environment.python.user_managed_dependencies = False

# specify CondaDependencies obj
run_config.environment.python.conda_dependencies = CondaDependencies.create(conda_packages=['scikit-learn'],
                                                                           pip_packages=['debugpy', 'azureml-sdk==<SDK-VERSION>'])
```

В разделе [Настройка скриптов Python](#configure-python-scripts) в скрипты, используемые этапами КОНВЕЙЕРа машинного обучения, были добавлены новые аргументы. В следующем фрагменте кода показано, как использовать эти аргументы, чтобы включить отладку для компонента и установить время ожидания. Здесь также показано, как использовать созданную ранее среду, задав `runconfig=run_config` следующие параметры.

```python
# Use RunConfig from a pipeline step
step1 = PythonScriptStep(name="train_step",
                         script_name="train.py",
                         arguments=['--remote_debug', '--remote_debug_connection_timeout', 300,'--remote_debug_client_ip','<VS-CODE-CLIENT-IP>','--remote_debug_port',5678],
                         compute_target=aml_compute,
                         source_directory=source_directory,
                         runconfig=run_config,
                         allow_reuse=False)
```

При выполнении конвейера каждый шаг создает дочерний запуск. Если включена отладка, то измененный сценарий записывает в журнал сведения, аналогичные следующему тексту в `70_driver_log.txt` для дочернего запуска:

```text
Timeout for debug connection: 300
ip_address: 10.3.0.5
```

Сохраните `ip_address` значение. Они будут использоваться в следующем разделе.

> [!TIP]
> Вы также можете найти IP-адрес из журналов выполнения для дочернего выполнения этого шага конвейера. Дополнительные сведения о просмотре этих сведений см. в статье [мониторинг запусков и метрик эксперимента машинного обучения Azure](how-to-track-experiments.md).

### <a name="configure-development-environment"></a>Настройка среды разработки

1. Чтобы установить дебугпи в среде разработки VS Code, используйте следующую команду:

    ```bash
    python -m pip install --upgrade debugpy
    ```

    Дополнительные сведения об использовании дебугпи с VS Code см. в разделе [Удаленная отладка](https://code.visualstudio.com/docs/python/debugging#_debugging-by-attaching-over-a-network-connection).

1. Чтобы настроить VS Code для взаимодействия с Машинное обучение Azure вычислений, в котором работает отладчик, создайте новую конфигурацию отладки:

    1. В VS Code выберите меню __Отладка__, а затем щелкните __Открыть конфигурации__. Откроется файл с именем __launch.json__.

    1. В __launch.jsв__ файле найдите строку, содержащую `"configurations": [` и вставьте следующий текст после него. Измените `"host": "<IP-ADDRESS>"` запись на IP-адрес, возвращенный в журналах из предыдущего раздела. Измените `"localRoot": "${workspaceFolder}/code/step"` запись на локальный каталог, содержащий копию отлаживаемого скрипта:

        ```json
        {
            "name": "Azure Machine Learning Compute: remote debug",
            "type": "python",
            "request": "attach",
            "port": 5678,
            "host": "<IP-ADDRESS>",
            "redirectOutput": true,
            "pathMappings": [
                {
                    "localRoot": "${workspaceFolder}/code/step1",
                    "remoteRoot": "."
                }
            ]
        }
        ```

        > [!IMPORTANT]
        > Если в разделе конфигурации уже есть другие записи, добавьте запятую (,) после вставленного кода.

        > [!TIP]
        > Рекомендуется, особенно для конвейеров, размещать ресурсы для скриптов в отдельных каталогах, чтобы код был важен только для каждого из шагов. В этом примере `localRoot` Пример значения ссылается на `/code/step1` .
        >
        > При отладке нескольких скриптов в разных каталогах создайте отдельный раздел конфигурации для каждого скрипта.

    1. Сохраните файл __launch.json__.

### <a name="connect-the-debugger"></a>Подключение отладчика

1. Откройте VS Code и откройте локальную копию скрипта.
2. Задайте точки останова, где сценарий должен останавливаться после присоединения.
3. Пока в дочернем процессе выполняется сценарий, а в `Timeout for debug connection` журналах отображается, используйте клавишу F5 или выберите __Отладка__. При появлении запроса выберите конфигурацию __удаленной отладки машинное обучение Azure вычисление: Удаленная отладка__ . Можно также выбрать значок отладки на боковой панели, в раскрывающемся меню Отладка выберите __машинное обучение Azure: Удаленная отладка__ , а затем с помощью зеленой стрелки присоединить отладчик.

    На этом этапе VS Code подключается к дебугпи на кластерном узле и останавливается в точке останова, заданной ранее. Теперь вы можете пошагово выполнять код, просматривать переменные и т. д.

    > [!NOTE]
    > Если в журнале отображается запись, то `Debugger attached = False` время ожидания истекает, и скрипт продолжает работу без отладчика. Снова отправьте конвейер и Подключите отладчик после `Timeout for debug connection` сообщения и до истечения времени ожидания.

## <a name="debug-and-troubleshoot-deployments"></a>Отладка и устранение неполадок развертывания

В некоторых случаях может потребоваться интерактивная отладка кода Python, содержащегося в развертывании модели. Например, если начальный сценарий не работает и причину невозможно определить с помощью дополнительного ведения журнала. С помощью VS Code и дебугпи можно присоединяться к коду, выполняющемуся в контейнере DOCKER.

> [!IMPORTANT]
> Этот метод отладки не работает при использовании `Model.deploy()` и `LocalWebservice.deploy_configuration` для развертывания модели в локальной среде. В этом случае необходимо создать образ, используя метод [Model.package()](/python/api/azureml-core/azureml.core.model.model#package-workspace--models--inference-config-none--generate-dockerfile-false-).

Для локальных развертываний веб-службы требуется рабочая установка Docker в локальной системе. Дополнительные сведения об использовании Docker см. в [соответствующей документации](https://docs.docker.com/). Обратите внимание, что при работе с экземплярами вычислений DOCKER уже установлен.

### <a name="configure-development-environment"></a>Настройка среды разработки

1. Чтобы установить дебугпи в локальной среде разработки VS Code, используйте следующую команду:

    ```bash
    python -m pip install --upgrade debugpy
    ```

    Дополнительные сведения об использовании дебугпи с VS Code см. в разделе [Удаленная отладка](https://code.visualstudio.com/docs/python/debugging#_debugging-by-attaching-over-a-network-connection).

1. Чтобы настроить VS Code для взаимодействия с образом Docker, создайте новую конфигурацию отладки:

    1. В VS Code выберите меню __Отладка__ в области __выполнения__ , а затем выберите __Открыть конфигурации__. Откроется файл с именем __launch.json__.

    1. В __launch.jsв__ файле найдите элемент __Configurations__ (строка, которая содержит `"configurations": [` ) и вставьте следующий текст после него. 

        ```json
        {
            "name": "Azure Machine Learning Deployment: Docker Debug",
            "type": "python",
            "request": "attach",
            "connect": {
                "port": 5678,
                "host": "0.0.0.0",
            },
            "pathMappings": [
                {
                    "localRoot": "${workspaceFolder}",
                    "remoteRoot": "/var/azureml-app"
                }
            ]
        }
        ```
        После вставки __launch.js__ файл должен выглядеть следующим образом:
        ```json
        {
        // Use IntelliSense to learn about possible attributes.
        // Hover to view descriptions of existing attributes.
        // For more information, visit: https://go.microsoft.com/fwlink/linkid=830387
        "version": "0.2.0",
        "configurations": [
            {
                "name": "Python: Current File",
                "type": "python",
                "request": "launch",
                "program": "${file}",
                "console": "integratedTerminal"
            },
            {
                "name": "Azure Machine Learning Deployment: Docker Debug",
                "type": "python",
                "request": "attach",
                "connect": {
                    "port": 5678,
                    "host": "0.0.0.0"
                    },
                "pathMappings": [
                    {
                        "localRoot": "${workspaceFolder}",
                        "remoteRoot": "/var/azureml-app"
                    }
                ]
            }
            ]
        }
        ```

        > [!IMPORTANT]
        > Если в разделе конфигурации уже есть другие записи, добавьте запятую ( __,__ ) после вставленного кода.

        Этот раздел подключается к контейнеру DOCKER с помощью порта __5678__.

    1. Сохраните файл __launch.json__.

### <a name="create-an-image-that-includes-debugpy"></a>Создание образа, включающего дебугпи

1. Измените среду conda для развертывания, чтобы она включала дебугпи. В следующем примере демонстрируется добавление с помощью параметра `pip_packages`:

    ```python
    from azureml.core.conda_dependencies import CondaDependencies 


    # Usually a good idea to choose specific version numbers
    # so training is made on same packages as scoring
    myenv = CondaDependencies.create(conda_packages=['numpy==1.15.4',
                                'scikit-learn==0.19.1', 'pandas==0.23.4'],
                                 pip_packages = ['azureml-defaults==1.0.83', 'debugpy'])

    with open("myenv.yml","w") as f:
        f.write(myenv.serialize_to_string())
    ```

1. Чтобы запустить дебугпи и дождаться подключения при запуске службы, добавьте следующее в начало `score.py` файла:

    ```python
    import debugpy
    # Allows other computers to attach to debugpy on this IP address and port.
    debugpy.listen(('0.0.0.0', 5678))
    # Wait 30 seconds for a debugger to attach. If none attaches, the script continues as normal.
    debugpy.wait_for_client()
    print("Debugger attached...")
    ```

1. Создайте образ на основе определения среды и извлеките его в локальный реестр. 

    > [!NOTE]
    > В этом примере предполагается, что `ws` указывает на вашу рабочую область Машинное обучение Azure, а `model` представляет собой развертываемую модель. Файл `myenv.yml` содержит зависимости conda, созданные на шаге 1.

    ```python
    from azureml.core.conda_dependencies import CondaDependencies
    from azureml.core.model import InferenceConfig
    from azureml.core.environment import Environment


    myenv = Environment.from_conda_specification(name="env", file_path="myenv.yml")
    myenv.docker.base_image = None
    myenv.docker.base_dockerfile = "FROM mcr.microsoft.com/azureml/base:intelmpi2018.3-ubuntu16.04"
    inference_config = InferenceConfig(entry_script="score.py", environment=myenv)
    package = Model.package(ws, [model], inference_config)
    package.wait_for_creation(show_output=True)  # Or show_output=False to hide the Docker build logs.
    package.pull()
    ```

    После создания и загрузки образа (этот процесс может занять более 10 минут, поэтому подождите), путь к образу (включая репозиторий, имя и тег, который в данном случае является также его дайджестом) будет отображен в сообщении, похожем на следующее:

    ```text
    Status: Downloaded newer image for myregistry.azurecr.io/package@sha256:<image-digest>
    ```

1. Чтобы упростить работу с изображением локально, можно использовать следующую команду, чтобы добавить тег для этого изображения. Замените `myimagepath` в следующей команде на значение Location из предыдущего шага.

    ```bash
    docker tag myimagepath debug:1
    ```

    На остальных шагах для обозначения расположения локального образа вы можете указывать `debug:1` вместо значения полного пути.

### <a name="debug-the-service"></a>Отладка службы

> [!TIP]
> Если для подключения дебугпи в файле задано время ожидания `score.py` , необходимо подключить VS Code к сеансу отладки до истечения времени ожидания. Запустите VS Code, откройте локальную копию `score.py`, установите точку останова и подготовьте ее к работе, прежде чем выполнять действия, описанные в этом разделе.
>
> Дополнительные сведения об отладке и установке точек останова см. на [этой странице](https://code.visualstudio.com/Docs/editor/debugging).

1. Чтобы запустить контейнер Docker с помощью образа, используйте следующую команду:

    ```bash
    docker run -it --name debug -p 8000:5001 -p 5678:5678 -v <my_local_path_to_score.py>:/var/azureml-app/score.py debug:1 /bin/bash
    ```

    Это присоединяет ваш `score.py` локальный объект к контейнеру. Таким образом, любые изменения, внесенные в редакторе, автоматически отражаются в контейнере.

2. Для лучшего удобства можно переходить к контейнеру с помощью нового интерфейса VS Code. Выберите `Docker` расширение на боковой панели VS Code, найдите созданный локальный контейнер в этой документации `debug:1` . Щелкните этот контейнер правой кнопкой мыши и выберите `"Attach Visual Studio Code"` , затем автоматически откроется новый интерфейс VS Code, и этот интерфейс отобразится внутри созданного контейнера.

    ![Интерфейс VS Code контейнера](./media/how-to-troubleshoot-deployment/container-interface.png)

3. В контейнере выполните следующую команду в оболочке.

    ```bash
    runsvdir /var/runit
    ```
    После этого в оболочке в контейнере можно увидеть следующие выходные данные:

    ![Контейнер для запуска выходных данных консоли](./media/how-to-troubleshoot-deployment/container-run.png)

4. Чтобы присоединить VS Code к дебугпи в контейнере, откройте VS Code и используйте клавишу F5 или выберите __Отладка__. При появлении запроса выберите __развертывание машинное обучение Azure: конфигурация отладки DOCKER__ . Вы также можете выбрать значок " __выполнить__ расширение" на боковой панели, в раскрывающемся меню отладка __машинное обучение Azure развертывание: DOCKER__ , а затем с помощью зеленой стрелки присоединить отладчик.

    ![Значок отладки, кнопка запуска отладки и селектор конфигурации](./media/how-to-troubleshoot-deployment/start-debugging.png)
    
    После нажатия зеленой стрелки и подключения отладчика в контейнере VS Code интерфейс можно увидеть некоторые новые сведения:
    
    ![Сведения о присоединенном отладчике контейнеров](./media/how-to-troubleshoot-deployment/debugger-attached.png)
    
    Кроме того, в основном интерфейсе VS Code доступны следующие возможности:

    ![Точка останова VS Code в score.py](./media/how-to-troubleshoot-deployment/local-debugger.png)

И теперь локальный объект, `score.py` присоединенный к контейнеру, уже остановлен в точках останова, где вы задали значение. На этом этапе VS Code подключается к дебугпи в контейнере DOCKER и останавливает контейнер DOCKER в точке останова, заданной ранее. Теперь вы можете пошагово выполнять код, просматривать переменные и т. д.

Дополнительные сведения об использовании VS Code для отладки Python см. на странице [Отладка кода Python](https://code.visualstudio.com/docs/python/debugging).

### <a name="stop-the-container"></a>Остановка контейнера

Чтобы остановить контейнер, используйте следующую команду:

```bash
docker stop debug
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы настроили VS Code удаленно, вы можете использовать вычислительный экземпляр в качестве удаленного вычислений от VS Code для интерактивной отладки кода. 

Дополнительные сведения об устранении неполадок:

* [Развертывание локальной модели](how-to-troubleshoot-deployment-local.md)
* [Удаленное развертывание модели](how-to-troubleshoot-deployment.md)
* [Конвейеры машинного обучения](how-to-debug-pipelines.md)
* [параллелрунстеп](how-to-debug-parallel-run-step.md)

