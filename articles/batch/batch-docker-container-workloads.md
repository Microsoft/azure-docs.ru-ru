---
title: Контейнерные рабочие нагрузки
description: Сведения о выполнении и масштабировании приложений из образов контейнера в пакетной службе Azure. Вы можете создать пул вычислительных узлов, которые поддерживают выполнение задач контейнера.
ms.topic: how-to
ms.date: 10/06/2020
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: 9d8776ba8e683cd14c766fead1e7238a6c24d000
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91843453"
---
# <a name="run-container-applications-on-azure-batch"></a>Выполнение контейнерных приложений в пакетной службе Azure

Пакетная служба Azure позволяет выполнять и масштабировать большие количества заданий в пакетной обработке данных в Azure. Такие задачи могут выполняться напрямую на виртуальных машинах (узлах) в пуле пакетной службы. Кроме того, пул пакетной службы можно настроить для выполнения задач в совместимых с Docker контейнерах на узлах. В этой статье показано, как создать пул вычислительных узлов, которые поддерживают выполнение задач контейнера, и как выполнять эти задачи в пуле.

В примерах кода используются пакеты SDK .NET и Python для пакетной службы. Также для создания пулов пакетной службы с включенными контейнерами и выполнения задач в контейнере можно использовать другие пакеты SDK пакетной службы и средства, включая портал Azure.

## <a name="why-use-containers"></a>Зачем нужны контейнеры?

Использование контейнеров предоставляет простой способ запуска задач пакетной службы без необходимости управлять средой и зависимостями. В контейнерах приложения развертываются в качестве упрощенных переносимых самодостаточных единиц, которые могут выполняться в нескольких разных средах. Например, создайте и протестируйте контейнер локально, а затем отправьте образ контейнера в реестр в Azure или другое расположение. Модель развертывания контейнера гарантирует, что среда выполнения приложения всегда правильно установлена и настроена, независимо от того, где размещено приложение. Для решения контейнерных задач в пакетной службе также можно воспользоваться преимуществами функций неконтейнерных задач, включая пакеты приложений и управление файлами ресурсов и выходными файлами.

## <a name="prerequisites"></a>Предварительные требования

Вам нужно иметь представление о контейнере, а также знать, как создать пул пакетной службы и задание.

- **Версии SDK**. Пакеты SDK для пакетной службы поддерживают образы контейнеров в таких версиях:
  - REST API пакетной службы версии 2017-09-01.6.0;
  - пакет SDK для .NET пакетной службы версии 8.0.0;
  - пакет SDK для Python пакетной службы версии 4.0;
  - пакет SDK для Java пакетной службы версии 3.0;
  - пакет SDK для Node.js пакетной службы версии 3.0.

- **Учетные записи**. В своей подписке Azure необходимо создать учетную запись пакетной службы и при необходимости создать учетную запись службы хранилища Azure.

- **Поддерживаемый образ виртуальной машины**. Контейнеры поддерживаются только в пулах, созданных с использованием конфигурации виртуальной машины из образов, описанных в следующем разделе: "Поддерживаемые образы виртуальных машин". Если вы решили предоставить пользовательский образ, ознакомьтесь с рекомендациями в следующем разделе и требованиями в статье [Использование управляемого пользовательского образа для создания пула виртуальных машин](batch-custom-images.md).

Учитывайте следующие ограничения.

- Пакетная служба обеспечивает поддержку RDMA только для контейнеров, выполняемых в пулах Linux.

- Для рабочих нагрузок контейнера Windows мы рекомендуем выбрать размер многоядерного ВМ для пула.

## <a name="supported-virtual-machine-images"></a>Поддерживаемые образы виртуальных машин

Чтобы создать пул вычислительных узлов виртуальной машины для рабочих нагрузок контейнера, используйте один из поддерживаемых образов Windows или Linux, указанных ниже. Дополнительные сведения о образах Marketplace, совместимых с пакетной службой, см. в разделе [список образов виртуальных машин](batch-linux-nodes.md#list-of-virtual-machine-images).

### <a name="windows-support"></a>Поддержка Windows

Пакетная служба поддерживает образы Windows Server, назначения которых поддерживают контейнеры. Обычно такие имена SKU образа имеют суффикс `-with-containers` или `-with-containers-smalldisk`. Кроме того, [API для перечисления всех поддерживаемых образов в пакетной службе](batch-linux-nodes.md#list-of-virtual-machine-images) будет указывать возможность `DockerCompatible`, если образ поддерживает контейнеры Docker.

Можно также создавать пользовательские образы на основе виртуальных машин, на которых запущено средство Docker в Windows.

### <a name="linux-support"></a>Поддержка Linux

Для рабочих нагрузок контейнера Linux пакетная служба сейчас поддерживает указанные ниже образы Linux, опубликованные разработчиками пакетной службы Microsoft Azure на платформе Azure Marketplace, без необходимости в пользовательском образе.

#### <a name="vm-sizes-without-rdma"></a>Размеры виртуальных машин без RDMA

- Издатель: `microsoft-azure-batch`
  - Предложение: `centos-container`
  - Предложение: `ubuntu-server-container`

#### <a name="vm-sizes-with-rdma"></a>Размеры виртуальных машин с RDMA

- Издатель: `microsoft-azure-batch`
  - Предложение: `centos-container-rdma`
  - Предложение: `ubuntu-server-container-rdma`

Эти образы поддерживаются только в пулах пакетной службы Azure и предназначены для выполнения контейнеров Docker. Они содержат следующее:

- Предустановленная среда выполнения контейнера [Moby](https://github.com/moby/moby), поддерживающая Docker.

- Предустановленные драйверы GPU NVIDIA и среда выполнения контейнера NVIDIA для упрощения развертывания на виртуальных машинах Azure серии N.

- Предварительно установленный или настроенный образ с поддержкой размеров виртуальных машин RDMA InfiniBand для образов с суффиксом `-rdma`. Сейчас эти образы не поддерживают размеры виртуальных машин RDMA/IB SR-IOV.

Вы также можете создавать пользовательские образы на основе виртуальных машин, на которых работает Docker, используя один из дистрибутивов Linux, совместимый с пакетной службой. Если вы решите предоставить собственный пользовательский образ Linux, ознакомьтесь с инструкциями в статье [Использование управляемого пользовательского образа для создания пула виртуальных машин](batch-custom-images.md).

Для поддержки Docker в пользовательском образе установите [Docker Community Edition (CE)](https://www.docker.com/community-edition) или [Docker Enterprise Edition (EE)](https://www.docker.com/enterprise-edition).

Дополнительные рекомендации по использованию пользовательского образа Linux:

- Чтобы воспользоваться высокой производительностью GPU виртуальных машин Azure серии N для пользовательского образа, предварительно установите драйверы NVIDIA. Кроме того, необходимо установить служебную программу подсистемы Docker для GPU NVIDIA — [NVIDIA Docker](https://github.com/NVIDIA/nvidia-docker).

- Для доступа к сети Azure RDMA используйте размер виртуальной машины с поддержкой RDMA. Необходимые драйверы RDMA установлены в образах CentOS HPC и Ubuntu, поддерживаемых пакетной службой. Для выполнения рабочих нагрузок MPI может потребоваться дополнительная настройка. Дополнительные сведения см. в статье [Использование экземпляров с поддержкой RDMA или графического процессора (GPU) в пулах пакетной службы](batch-pool-compute-intensive-sizes.md).

## <a name="container-configuration-for-batch-pool"></a>Конфигурация контейнера для пула пакетной службы

Чтобы включить пул пакетной службы для выполнения рабочих нагрузок контейнера, необходимо указать параметр [ContainerConfiguration](/dotnet/api/microsoft.azure.batch.containerconfiguration) в пуле объекта [VirtualMachineConfiguration](/dotnet/api/microsoft.azure.batch.virtualmachineconfiguration). (Эта статья содержит ссылки на справочники по API пакетной службы для .NET. Соответствующие параметры можно найти в статье об [API пакетной службы для Python](/python/api/overview/azure/batch).)

Можно создать пул с включенным контейнером с предварительно загруженными образами контейнеров или без них, как показано в следующих примерах. Процесс извлечения (или предварительной загрузки) позволяет предварительно загрузить образы контейнеров из центра Docker или другого реестра контейнеров в Интернете. Чтобы обеспечить высокую производительность, используйте [реестр контейнеров Azure](../container-registry/container-registry-intro.md) в одном регионе с учетной записью пакетной службы.

Преимущество предварительного получения образов контейнеров заключается в том, что при первом запуске задач не нужно ожидать скачивания образа контейнера. Конфигурация контейнера извлекает образы контейнеров на виртуальную машину при создании пула. Задачи, выполняющиеся в пуле, могут ссылаться на список образов контейнеров и параметры выполнения контейнера.

### <a name="pool-without-prefetched-container-images"></a>Пул без предварительно полученных образов контейнеров

Чтобы настроить пул с поддержкой контейнера без предварительно выбранных образов контейнеров, определите `ContainerConfiguration` объекты и, `VirtualMachineConfiguration` как показано в следующих примерах. В этих примерах в Marketplace используется сервер Ubuntu для пула контейнеров пакетной службы Azure.

```python
image_ref_to_use = batch.models.ImageReference(
    publisher='microsoft-azure-batch',
    offer='ubuntu-server-container',
    sku='16-04-lts',
    version='latest')

"""
Specify container configuration. This is required even though there are no prefetched images.
"""

container_conf = batch.models.ContainerConfiguration()

new_pool = batch.models.PoolAddParameter(
    id=pool_id,
    virtual_machine_configuration=batch.models.VirtualMachineConfiguration(
        image_reference=image_ref_to_use,
        container_configuration=container_conf,
        node_agent_sku_id='batch.node.ubuntu 16.04'),
    vm_size='STANDARD_D1_V2',
    target_dedicated_nodes=1)
...
```

```csharp
ImageReference imageReference = new ImageReference(
    publisher: "microsoft-azure-batch",
    offer: "ubuntu-server-container",
    sku: "16-04-lts",
    version: "latest");

// Specify container configuration. This is required even though there are no prefetched images.
ContainerConfiguration containerConfig = new ContainerConfiguration();

// VM configuration
VirtualMachineConfiguration virtualMachineConfiguration = new VirtualMachineConfiguration(
    imageReference: imageReference,
    nodeAgentSkuId: "batch.node.ubuntu 16.04");
virtualMachineConfiguration.ContainerConfiguration = containerConfig;

// Create pool
CloudPool pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: 1,
    virtualMachineSize: "STANDARD_D1_V2",
    virtualMachineConfiguration: virtualMachineConfiguration);
```

### <a name="prefetch-images-for-container-configuration"></a>Предварительное получение образов для конфигурации контейнера

Чтобы выполнить предварительную загрузку образов контейнеров в пуле, добавьте список образов контейнеров (`container_image_names` в Python) в `ContainerConfiguration`.

В следующем простом примере на Python показано, как предварительно загрузить стандартный образ контейнера Ubuntu из [центра Docker](https://hub.docker.com).

```python
image_ref_to_use = batch.models.ImageReference(
    publisher='microsoft-azure-batch',
    offer='ubuntu-server-container',
    sku='16-04-lts',
    version='latest')

"""
Specify container configuration, fetching the official Ubuntu container image from Docker Hub.
"""

container_conf = batch.models.ContainerConfiguration(
    container_image_names=['ubuntu'])

new_pool = batch.models.PoolAddParameter(
    id=pool_id,
    virtual_machine_configuration=batch.models.VirtualMachineConfiguration(
        image_reference=image_ref_to_use,
        container_configuration=container_conf,
        node_agent_sku_id='batch.node.ubuntu 16.04'),
    vm_size='STANDARD_D1_V2',
    target_dedicated_nodes=1)
...
```

В следующем примере на C# предполагается, что вы намерены предварительно загрузить образ TensorFlow из [центра Docker](https://hub.docker.com). Данный пример включает задачу запуска, выполняемую на узле виртуальной машины на узлах пула. Можно запустить стартовую задачу в узле, например, для вставки файлового сервера, доступ к которому можно получить из контейнеров.

```csharp
ImageReference imageReference = new ImageReference(
    publisher: "microsoft-azure-batch",
    offer: "ubuntu-server-container",
    sku: "16-04-lts",
    version: "latest");

ContainerRegistry containerRegistry = new ContainerRegistry(
    registryServer: "https://hub.docker.com",
    userName: "UserName",
    password: "YourPassword"                
);

// Specify container configuration, prefetching Docker images
ContainerConfiguration containerConfig = new ContainerConfiguration();
containerConfig.ContainerImageNames = new List<string> { "tensorflow/tensorflow:latest-gpu" };
containerConfig.ContainerRegistries = new List<ContainerRegistry> { containerRegistry };

// VM configuration
VirtualMachineConfiguration virtualMachineConfiguration = new VirtualMachineConfiguration(
    imageReference: imageReference,
    nodeAgentSkuId: "batch.node.ubuntu 16.04");
virtualMachineConfiguration.ContainerConfiguration = containerConfig;

// Set a native host command line start task
StartTask startTaskContainer = new StartTask( commandLine: "<native-host-command-line>" );

// Create pool
CloudPool pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    virtualMachineSize: "Standard_NC6",
    virtualMachineConfiguration: virtualMachineConfiguration);

// Start the task in the pool
pool.StartTask = startTaskContainer;
...
```

### <a name="prefetch-images-from-a-private-container-registry"></a>Предварительно полученные образы из закрытого реестра контейнеров

Можно также выполнять предварительное получение образов контейнеров путем проверки подлинности на сервере закрытого реестра контейнеров. В следующих примерах `ContainerConfiguration` `VirtualMachineConfiguration` объекты и выбирали частный образ TensorFlow из закрытого реестра контейнеров Azure. Здесь используется такая же ссылка на образ, как и в предыдущем примере.

```python
image_ref_to_use = batch.models.ImageReference(
        publisher='microsoft-azure-batch',
        offer='ubuntu-server-container',
        sku='16-04-lts',
        version='latest')

# Specify a container registry
container_registry = batch.models.ContainerRegistry(
        registry_server="myRegistry.azurecr.io",
        user_name="myUsername",
        password="myPassword")

# Create container configuration, prefetching Docker images from the container registry
container_conf = batch.models.ContainerConfiguration(
        container_image_names = ["myRegistry.azurecr.io/samples/myImage"],
        container_registries =[container_registry])

new_pool = batch.models.PoolAddParameter(
            id="myPool",
            virtual_machine_configuration=batch.models.VirtualMachineConfiguration(
                image_reference=image_ref_to_use,
                container_configuration=container_conf,
                node_agent_sku_id='batch.node.ubuntu 16.04'),
            vm_size='STANDARD_D1_V2',
            target_dedicated_nodes=1)
```

```csharp
// Specify a container registry
ContainerRegistry containerRegistry = new ContainerRegistry(
    registryServer: "myContainerRegistry.azurecr.io",
    userName: "myUserName",
    password: "myPassword");

// Create container configuration, prefetching Docker images from the container registry
ContainerConfiguration containerConfig = new ContainerConfiguration();
containerConfig.ContainerImageNames = new List<string> {
        "myContainerRegistry.azurecr.io/tensorflow/tensorflow:latest-gpu" };
containerConfig.ContainerRegistries = new List<ContainerRegistry> { containerRegistry } );

// VM configuration
VirtualMachineConfiguration virtualMachineConfiguration = new VirtualMachineConfiguration(
    imageReference: imageReference,
    nodeAgentSkuId: "batch.node.ubuntu 16.04");
virtualMachineConfiguration.ContainerConfiguration = containerConfig;

// Create pool
CloudPool pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: 4,
    virtualMachineSize: "Standard_NC6",
    virtualMachineConfiguration: virtualMachineConfiguration);
...
```

## <a name="container-settings-for-the-task"></a>Параметры контейнера для задачи

Чтобы выполнить контейнерную задачу в пуле с поддержкой контейнеров, укажите параметры нужного контейнера. Эти параметры включают используемый образ, реестр и параметры выполнения контейнера.

- Используйте свойство `ContainerSettings` в классах задач для настройки параметров конкретного контейнера. Эти параметры определяются классом [TaskContainerSettings](/dotnet/api/microsoft.azure.batch.taskcontainersettings). Обратите внимание, что параметр контейнера `--rm` не требует дополнительный параметр `--runtime`, так как он обрабатывается пакетной службой.

- При запуске задач на образах контейнеров для [задач облака](/dotnet/api/microsoft.azure.batch.cloudtask) и [диспетчера заданий](/dotnet/api/microsoft.azure.batch.cloudjob.jobmanagertask) требуются параметры контейнера. Тем не менее [задача запуска](/dotnet/api/microsoft.azure.batch.starttask), [задача подготовки задания](/dotnet/api/microsoft.azure.batch.cloudjob.jobpreparationtask) и [задача снятия задания](/dotnet/api/microsoft.azure.batch.cloudjob.jobreleasetask) не требуют параметров контейнера (то есть их можно выполнить в контексте контейнера или напрямую на узле).

- Для Windows задачи должны выполняться с параметром [елеватионлевел](/rest/api/batchservice/task/add#elevationlevel) , имеющим значение `admin` . 

- Для Linux Пакетная система будет сопоставлять разрешение пользователя или группы с контейнером. Если доступ к любой папке в контейнере требует разрешения администратора, может потребоваться выполнить задачу как область пула с уровнем прав администратора. Это обеспечит выполнение пакетной службой задачи в качестве корневого в контексте контейнера. В противном случае пользователь без прав администратора может не иметь доступа к этим папкам.

- Для пулов контейнеров с оборудованием с поддержкой GPU Пакетная службы автоматически включит GPU для задач контейнера, поэтому не следует включать `–gpus` аргумент.

### <a name="container-task-command-line"></a>Командная строка контейнерной задачи

При выполнении контейнерной задачи пакетная служба автоматически применяет команду [docker create](https://docs.docker.com/engine/reference/commandline/create/), чтобы создать контейнер на основе образа, указанного в задаче. Затем пакетная служба отслеживает выполнение задачи в контейнере.

Как и для обычных задач пакетной службы, для контейнерной задачи вам нужно указать командную строку. Так как пакетная служба создает контейнер автоматически, в этой командной строке нужно только указать одну или несколько команд для выполнения в контейнере.

Если образ контейнера для задачи пакетной службы настраивается с помощью скрипта [ENTRYPOINT](https://docs.docker.com/engine/reference/builder/#exec-form-entrypoint-example), вы можете задать в командной строке значение ENTRYPOINT по умолчанию или переопределить его:

- Чтобы использовать значение ENTRYPOINT по умолчанию для образа контейнера, в качестве значения командной строки задачи укажите пустую строку `""`.

- Чтобы переопределить значение ENTRYPOINT по умолчанию или присвоить значение, если образ не содержит ENTRYPOINT, задайте для контейнера нужную командную строку, например `/app/myapp` или `/bin/sh -c python myscript.py`.

Необязательный параметр [ContainerRunOptions](/dotnet/api/microsoft.azure.batch.taskcontainersettings.containerrunoptions) содержит дополнительные аргументы для команды `docker create`, которую пакетная служба использует для создания и выполнения контейнера. Например, чтобы задать для контейнера рабочий каталог, укажите параметр `--workdir <directory>`. Дополнительные сведения см. в разделе справки по команде [docker create](https://docs.docker.com/engine/reference/commandline/create/).

### <a name="container-task-working-directory"></a>Рабочий каталог для задач контейнера

Задача пакетной службы Azure выполняется в рабочем каталоге контейнера, который очень похож на каталог, которую пакетная служба настраивает для обычных (не контейнерных) задач. Обратите внимание, что этот рабочий каталог отличается от каталога [WORKDIR](https://docs.docker.com/engine/reference/builder/#workdir), если он настроен в образе, и от рабочего каталога контейнера по умолчанию (`C:\` в контейнере Windows или `/` в контейнере Linux).

Для контейнерной задачи пакетной службы настраивается следующее:

- все каталоги, размещенные на любом уровне в каталоге `AZ_BATCH_NODE_ROOT_DIR` на главном узле (это корневой каталог пакетной службы Azure), сопоставляются в контейнере;
- Все переменные среды задач сопоставляются в контейнере.
- рабочий каталог задачи `AZ_BATCH_TASK_WORKING_DIR` на узле настраивается так же, как для обычной задачи, и сопоставляется в контейнере.

Эти сопоставления позволяют работать с контейнерными задачами так же, как и с обычными. Например, вы можете устанавливать приложения с помощью пакетов приложений, обращаться к файлам ресурсов в службе хранилища Azure, настраивать параметры среды для задачи и сохранять выходные файлы задачи после остановки контейнера.

### <a name="troubleshoot-container-tasks"></a>Устранение неполадок с контейнерными задачами

Если ваша контейнерная задача работает не так, как ожидалось, вам будет полезно получить сведения о конфигурации WORKDIR или ENTRYPOINT для образа контейнера. Чтобы просмотреть конфигурацию, запустите команду [docker image inspect](https://docs.docker.com/engine/reference/commandline/image_inspect/).

Если потребуется, исправьте параметры контейнерной задачи с учетом характеристик образа:

- Используйте в командной строке задачи абсолютный путь. Если для командной строки задачи используется заданное в образе значение ENTRYPOINT по умолчанию, убедитесь, что используется абсолютный путь.
- В параметрах выполнения контейнера для задачи измените рабочую папку, чтобы она соответствовала папке WORKDIR в образе. Например, установите значение `--workdir /app`.

## <a name="container-task-examples"></a>Примеры контейнерных задач

В приведенном ниже фрагменте кода Python показана базовая командная строка, которая выполняется в контейнере, созданном на основе фиктивного образа из центра Docker. Здесь параметр контейнера `--rm` означает удаление контейнера после выполнения задачи, а параметр `--workdir` задает рабочую папку. Эта командная строка переопределяет заданное в контейнере значение ENTRYPOINT, заменяя его простой командой оболочки для создания небольшого файла в рабочей папке задачи на узле.

```python
task_id = 'sampletask'
task_container_settings = batch.models.TaskContainerSettings(
    image_name='myimage',
    container_run_options='--rm --workdir /')
task = batch.models.TaskAddParameter(
    id=task_id,
    command_line='/bin/sh -c \"echo \'hello world\' > $AZ_BATCH_TASK_WORKING_DIR/output.txt\"',
    container_settings=task_container_settings
)
```

В следующем примере на C# показаны основные параметры контейнера для задачи, выполняемой в облаке:

```csharp
// Simple container task command
string cmdLine = "c:\\app\\myApp.exe";

TaskContainerSettings cmdContainerSettings = new TaskContainerSettings (
    imageName: "myimage",
    containerRunOptions: "--rm --workdir c:\\app"
    );

CloudTask containerTask = new CloudTask (
    id: "Task1",
    commandline: cmdLine);
containerTask.ContainerSettings = cmdContainerSettings;
```

## <a name="next-steps"></a>Дальнейшие действия

- Чтобы упростить развертывание рабочих нагрузок контейнера в пакетной службе Azure через [Shipyard рецепты](https://github.com/Azure/batch-shipyard/tree/master/recipes), см. [раздел пакет Shipyard пакета](https://github.com/Azure/batch-shipyard) инструментов.
- Сведения об установке и использовании DOCKER CE в Linux см. в документации по [DOCKER](https://docs.docker.com/engine/installation/) .
- Узнайте, как [использовать управляемый пользовательский образ для создания пула виртуальных машин](batch-custom-images.md).
- Узнайте больше о [проекте Moby](https://mobyproject.org/) — платформе для создания контейнерных систем.
