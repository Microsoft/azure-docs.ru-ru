---
title: Выполнение вычислительных узлов виртуальных машин под управлением Linux в пакетной службе Azure
description: Узнайте, как обрабатывать параллельные расчетные рабочие нагрузки в пулах виртуальных машин Linux в пакетной службе Azure.
ms.topic: how-to
ms.date: 01/21/2021
ms.custom: H1Hack27Feb2017, devx-track-python, devx-track-csharp
ms.openlocfilehash: c711ec0d035b9b59ec7628a51fe3cff26de358bc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98683706"
---
# <a name="provision-linux-compute-nodes-in-batch-pools"></a>Подготовка вычислительных узлов Linux в пулах пакетной службы

Пакетная служба Azure позволяет выполнять параллельные вычислительные рабочие нагрузки на виртуальных машинах Linux и Windows. В этой статье описывается создание пулов вычислительных узлов Linux в пакетной службе с помощью клиентских библиотек [Python](https://pypi.python.org/pypi/azure-batch) и [.NET](/dotnet/api/microsoft.azure.batch). 

## <a name="virtual-machine-configuration"></a>Конфигурация виртуальной машины

При создании пула вычислительных узлов в пакетной службе есть два варианта для выбора размера узла и операционной системы: Cloud Services Configuration (Конфигурация облачных служб) и "Конфигурация виртуальной машины". Пулы [конфигурации виртуальных машин](nodes-and-pools.md#virtual-machine-configuration) состоят из виртуальных машин Azure, которые могут быть созданы из образов Linux или Windows. При создании пула с конфигурацией виртуальной машины необходимо указать [доступный размер вычислений](../virtual-machines/sizes.md), ссылку на образ виртуальной машины, которая будет установлена на узлах, и номер SKU агента узла пакетной службы (программа, которая выполняется на каждом узле и предоставляет интерфейс между узлом и пакетной службой).

### <a name="virtual-machine-image-reference"></a>Ссылка на образ виртуальной машины

Для предоставления вычислительных узлов в конфигурации виртуальной машины пакетная служба использует [масштабируемые наборы виртуальных машин](../virtual-machine-scale-sets/overview.md). Вы можете указать образ из [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/compute?filters=virtual-machine-images&page=1)или [использовать коллекцию общих образов для подготовки пользовательского образа](batch-sig-images.md).

При создании ссылки на образ виртуальной машины необходимо указать следующие свойства.

| **Свойство ссылки на изображение** | **Пример** |
| --- | --- |
| Издатель |Canonical |
| ПРЕДЛОЖЕНИЕ |UbuntuServer |
| номер SKU |18.04-LTS |
| Версия |последняя |

> [!TIP]
> Дополнительные сведения об этих свойствах и о том, как указать образы Marketplace, можно найти в статье [Поиск образов виртуальных машин Linux в Azure Marketplace с помощью Azure CLI](../virtual-machines/linux/cli-ps-findimage.md). Обратите внимание, что некоторые образы Marketplace в настоящее время не совместимы с пакетной службой.

### <a name="list-of-virtual-machine-images"></a>Список образов виртуальных машин

Не все образы Marketplace совместимы с доступными в настоящее время агентами узлов пакетной службы. Чтобы получить список всех поддерживаемых образов виртуальных машин Marketplace для пакетной службы и соответствующих номеров SKU агентов узлов, используйте [list_supported_images](/python/api/azure-batch/azure.batch.operations.AccountOperations#list-supported-images-account-list-supported-images-options-none--custom-headers-none--raw-false----operation-config-) (Python), [листсуппортедимажес](/dotnet/api/microsoft.azure.batch.pooloperations.listsupportedimages) (пакет .NET) или соответствующий API в другом языковом пакете SDK.

### <a name="node-agent-sku"></a>Номер SKU агента узла

[Агент узла пакетной](https://github.com/Azure/Batch/blob/master/changelogs/nodeagent/CHANGELOG.md) службы — это программа, которая выполняется на каждом узле в пуле и предоставляет интерфейс управления и контроля между узлом и пакетной службой. Существуют различные реализации агента узла, известные как номера SKU, для различных операционных систем. По существу, при создании конфигурации виртуальной машины сначала следует указать ссылку на образ виртуальной машины, а затем указать агент узла, который будет установлен на этом образе. Как правило, каждый номер SKU агента узла совместим с несколькими образами виртуальных машин. Вот несколько примеров номеров SKU агентов узлов:

- batch.node.ubuntu 18.04
- batch.node.centos 7
- batch.node.windows amd64

## <a name="create-a-linux-pool-batch-python"></a>Создание пула Linux. Python для пакетной службы

В следующем фрагменте кода показан пример создания пула вычислительных узлов Ubuntu Server с помощью [клиентской библиотеки пакетной службы Microsoft Azure для Python](https://pypi.python.org/pypi/azure-batch). Дополнительные сведения о модуле Python для пакетной службы см. в [справочной документации](/python/api/overview/azure/batch).

В этом фрагменте кода явно создается объект [ImageReference](/python/api/azure-mgmt-batch/azure.mgmt.batch.models.imagereference) и указывается каждое из его свойств (publisher, offer, SKU, version). Однако в рабочем коде рекомендуется использовать метод [list_supported_images](/python/api/azure-batch/azure.batch.operations.AccountOperations#list-supported-images-account-list-supported-images-options-none--custom-headers-none--raw-false----operation-config-) для выбора из доступных комбинаций образа и номера SKU агента узла во время выполнения.

```python
# Import the required modules from the
# Azure Batch Client Library for Python
import azure.batch.batch_service_client as batch
import azure.batch.batch_auth as batchauth
import azure.batch.models as batchmodels

# Specify Batch account credentials
account = "<batch-account-name>"
key = "<batch-account-key>"
batch_url = "<batch-account-url>"

# Pool settings
pool_id = "LinuxNodesSamplePoolPython"
vm_size = "STANDARD_D2_V3"
node_count = 1

# Initialize the Batch client
creds = batchauth.SharedKeyCredentials(account, key)
config = batch.BatchServiceClientConfiguration(creds, batch_url)
client = batch.BatchServiceClient(creds, batch_url)

# Create the unbound pool
new_pool = batchmodels.PoolAddParameter(id=pool_id, vm_size=vm_size)
new_pool.target_dedicated = node_count

# Configure the start task for the pool
start_task = batchmodels.StartTask()
start_task.run_elevated = True
start_task.command_line = "printenv AZ_BATCH_NODE_STARTUP_DIR"
new_pool.start_task = start_task

# Create an ImageReference which specifies the Marketplace
# virtual machine image to install on the nodes
ir = batchmodels.ImageReference(
    publisher="Canonical",
    offer="UbuntuServer",
    sku="18.04-LTS",
    version="latest")

# Create the VirtualMachineConfiguration, specifying
# the VM image reference and the Batch node agent
# to install on the node
vmc = batchmodels.VirtualMachineConfiguration(
    image_reference=ir,
    node_agent_sku_id="batch.node.ubuntu 18.04")

# Assign the virtual machine configuration to the pool
new_pool.virtual_machine_configuration = vmc

# Create pool in the Batch service
client.pool.add(new_pool)
```

Как упоминалось ранее, мы рекомендуем использовать метод [list_supported_images](/python/api/azure-batch/azure.batch.operations.AccountOperations#list-supported-images-account-list-supported-images-options-none--custom-headers-none--raw-false----operation-config-) , чтобы динамически выбирать из поддерживаемых в настоящее время сочетаний образа или агента узла (вместо создания явной [ImageReference](/python/api/azure-mgmt-batch/azure.mgmt.batch.models.imagereference) ). Следующий фрагмент кода Python демонстрирует, как использовать этот метод.

```python
# Get the list of supported images from the Batch service
images = client.account.list_supported_images()

# Obtain the desired image reference
image = None
for img in images:
  if (img.image_reference.publisher.lower() == "canonical" and
        img.image_reference.offer.lower() == "ubuntuserver" and
        img.image_reference.sku.lower() == "18.04-lts"):
    image = img
    break

if image is None:
  raise RuntimeError('invalid image reference for desired configuration')

# Create the VirtualMachineConfiguration, specifying the VM image
# reference and the Batch node agent to be installed on the node
vmc = batchmodels.VirtualMachineConfiguration(
    image_reference=image.image_reference,
    node_agent_sku_id=image.node_agent_sku_id)
```

## <a name="create-a-linux-pool-batch-net"></a>Создание пула Linux. .NET для пакетной службы

В следующем фрагменте кода показан пример создания пула вычислительных узлов Ubuntu Server с помощью клиентской библиотеки [.NET для пакетной службы](https://www.nuget.org/packages/Microsoft.Azure.Batch/). Дополнительные сведения о пакетной службе .NET см. в [справочной документации](/dotnet/api/microsoft.azure.batch).

В следующем фрагменте кода используется метод [PoolOperations. листсуппортедимажес](/dotnet/api/microsoft.azure.batch.pooloperations.listsupportedimages) для выбора из списка поддерживаемых комбинаций образов и номеров SKU агента узла в Marketplace. Этот метод рекомендуется, так как список поддерживаемых сочетаний может изменяться со временем. Чаще всего добавляются поддерживаемые сочетания.

```csharp
// Pool settings
const string poolId = "LinuxNodesSamplePoolDotNet";
const string vmSize = "STANDARD_D2_V3";
const int nodeCount = 1;

// Obtain a collection of all available node agent SKUs.
// This allows us to select from a list of supported
// VM image/node agent combinations.
List<ImageInformation> images =
    batchClient.PoolOperations.ListSupportedImages().ToList();

// Find the appropriate image information
ImageInformation image = null;
foreach (var img in images)
{
    if (img.ImageReference.Publisher == "Canonical" &&
        img.ImageReference.Offer == "UbuntuServer" &&
        img.ImageReference.Sku == "18.04-LTS")
    {
        image = img;
        break;
    }
}

// Create the VirtualMachineConfiguration for use when actually
// creating the pool
VirtualMachineConfiguration virtualMachineConfiguration =
    new VirtualMachineConfiguration(image.ImageReference, image.NodeAgentSkuId);

// Create the unbound pool object using the VirtualMachineConfiguration
// created above
CloudPool pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    virtualMachineSize: vmSize,
    virtualMachineConfiguration: virtualMachineConfiguration,
    targetDedicatedComputeNodes: nodeCount);

// Commit the pool to the Batch service
await pool.CommitAsync();
```

Несмотря на то, что в предыдущем фрагменте используется метод [PoolOperations. истсуппортедимажес](/dotnet/api/microsoft.azure.batch.pooloperations.listsupportedimages) для динамического перечисления и выбора из поддерживаемых сочетаний образов и номеров SKU агента узла (рекомендуется), можно также настроить [ImageReference](/dotnet/api/microsoft.azure.batch.imagereference) явным образом:

```csharp
ImageReference imageReference = new ImageReference(
    publisher: "Canonical",
    offer: "UbuntuServer",
    sku: "18.04-LTS",
    version: "latest");
```

## <a name="connect-to-linux-nodes-using-ssh"></a>Подключение к узлам Linux с помощью SSH

Во время разработки или устранения неполадок может потребоваться войти на узлы в пуле. В отличие от Windows Compute Nodes нельзя использовать протокол удаленного рабочего стола (RDP) для подключения к узлам Linux. Вместо этого пакетная служба включает доступ по протоколу SSH на каждом узле для удаленного подключения.

В следующем фрагменте кода Python создается пользователь на каждом узле пула, который необходим для удаленного подключения. Затем выводятся сведения о подключении SSH для каждого узла.

```python
import datetime
import getpass
import azure.batch.batch_service_client as batch
import azure.batch.batch_auth as batchauth
import azure.batch.models as batchmodels

# Specify your own account credentials
batch_account_name = ''
batch_account_key = ''
batch_account_url = ''

# Specify the ID of an existing pool containing Linux nodes
# currently in the 'idle' state
pool_id = ''

# Specify the username and prompt for a password
username = 'linuxuser'
password = getpass.getpass()

# Create a BatchClient
credentials = batchauth.SharedKeyCredentials(
    batch_account_name,
    batch_account_key
)
batch_client = batch.BatchServiceClient(
    credentials,
    base_url=batch_account_url
)

# Create the user that will be added to each node in the pool
user = batchmodels.ComputeNodeUser(username)
user.password = password
user.is_admin = True
user.expiry_time = \
    (datetime.datetime.today() + datetime.timedelta(days=30)).isoformat()

# Get the list of nodes in the pool
nodes = batch_client.compute_node.list(pool_id)

# Add the user to each node in the pool and print
# the connection information for the node
for node in nodes:
    # Add the user to the node
    batch_client.compute_node.add_user(pool_id, node.id, user)

    # Obtain SSH login information for the node
    login = batch_client.compute_node.get_remote_login_settings(pool_id,
                                                                node.id)

    # Print the connection info for the node
    print("{0} | {1} | {2} | {3}".format(node.id,
                                         node.state,
                                         login.remote_login_ip_address,
                                         login.remote_login_port))
```

Этот код будет выглядеть примерно так, как показано в следующем примере. В этом случае пул содержит четыре узла Linux.

```
Password:
tvm-1219235766_1-20160414t192511z | ComputeNodeState.idle | 13.91.7.57 | 50000
tvm-1219235766_2-20160414t192511z | ComputeNodeState.idle | 13.91.7.57 | 50003
tvm-1219235766_3-20160414t192511z | ComputeNodeState.idle | 13.91.7.57 | 50002
tvm-1219235766_4-20160414t192511z | ComputeNodeState.idle | 13.91.7.57 | 50001
```

При создании пользователя на узле вместо пароля можно указать открытый ключ SSH. В пакете SDK для Python используйте параметр **ssh_public_key** в [ComputeNodeUser](/python/api/azure-batch/azure.batch.models.computenodeuser). В .NET используйте свойство [ComputeNodeUser. SshPublicKey](/dotnet/api/microsoft.azure.batch.computenodeuser.sshpublickey#Microsoft_Azure_Batch_ComputeNodeUser_SshPublicKey) .

## <a name="pricing"></a>Цены

Пакетная служба Azure основана на технологии виртуальных машин Azure и облачных служб Azure. Сама пакетная служба предоставляется бесплатно. Это означает, что плата взимается только за вычислительные ресурсы (и связанные с этим расходы), используемые решениями пакетной службы. При выборе режима **Конфигурация виртуальной машины** плата взимается в зависимости от структуры [цен на виртуальные машины](https://azure.microsoft.com/pricing/details/virtual-machines/).

При развертывании приложений на узлах пакетной службы с помощью [пакетов приложений](batch-application-packages.md) также взимается плата за ресурсы службы хранилища Azure, используемые пакетами приложений.

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с [примерами кода Python](https://github.com/Azure/azure-batch-samples/tree/master/Python/Batch) в [репозитории Azure-Batch-Samples GitHub](https://github.com/Azure/azure-batch-samples) , чтобы узнать, как выполнять общие пакетные операции, такие как создание пулов, заданий и задач. [Файл сведений](https://github.com/Azure/azure-batch-samples/blob/master/Python/Batch/README.md), прилагаемый к примерам кода Python, содержит подробные сведения об установке необходимых пакетов.
- Узнайте об использовании [виртуальных машин с низким приоритетом](batch-low-pri-vms.md) с помощью пакетной службы.
