---
title: Создание пользовательского пула образов с помощью коллекции общих образов
description: Пользовательские пулы образов являются эффективным способом настройки узлов вычислений для выполнения рабочих нагрузок пакетной службы.
ms.topic: conceptual
ms.date: 03/04/2021
ms.custom: devx-track-python, devx-track-azurecli
ms.openlocfilehash: 8623c47952540717ae50538fd7b0282c9c8629bb
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102124250"
---
# <a name="use-the-shared-image-gallery-to-create-a-custom-image-pool"></a>Создание пользовательского пула образов с помощью коллекции общих образов

При создании пула в пакетной службе Azure с помощью конфигурации виртуальной машины необходимо указать образ виртуальной машины, предоставляющий операционную систему, для каждого вычислительного узла в пуле. Вы можете создать пул виртуальных машин с поддерживаемым образом Azure Marketplace или создать пользовательский образ с [общим образом из коллекции образов](../virtual-machines/shared-image-galleries.md).

## <a name="benefits-of-the-shared-image-gallery"></a>Преимущества общей коллекции образов

При использовании общей коллекции образов для настраиваемого образа вы можете управлять типом и конфигурацией операционной системы, а также типами дисков данных. Общий образ может содержать данные о приложениях и справочные данные, которые становятся доступными на всех узлах пула пакетной службы сразу после их подготовки.

В зависимости от требований для вашей среды, у вас также может быть несколько версий образа. При использовании версии образа для создания виртуальной машины версия образа используется для создания новых дисков для виртуальной машины.

Общий образ позволяет сэкономить время за счет подготовки вычислительных узлов пула к выполнению рабочей нагрузки пакетной службы. После подготовки можно использовать образ Azure Marketplace и установить программное обеспечение на каждый вычислительный узел, однако использование общего образа, как правило, более эффективно. Кроме того, можно указать несколько реплик для общего образа, поэтому при создании пулов с несколькими виртуальными машинами (более 600 виртуальных машин) вы сэкономите время на создание пула.

Использование общего образа, настроенного для конкретного сценария, дает определенные преимущества.

- **Использование одних и тех же образов в разных регионах.** Вы можете создавать реплики общих образов в разных регионах, чтобы все пулы использовали один и тот же образ.
- **Настройка операционной системы (ОС)** . Можно настроить конфигурацию диска операционной системы образа.
- **Предварительная установка приложений.** Предварительная установка приложений на диск операционной системы — это более эффективный и безопасный способ, чем установка приложений после подготовки вычислительных узлов с помощью задачи запуска.
- **Копирование больших объемов данных один раз.** Включите статические данные в управляемый общий образ, скопировав их на его диски данных. Это необходимо сделать только один раз, и данные станут доступными в каждом узле пула.
- **Увеличение пулов до больших размеров.** С помощью общей коллекции образов можно создавать большие пулы с настроенными образами вместе с другими репликами общих образов.
- **Более высокая производительность, чем использование только управляемого образа в качестве пользовательского образа.** Для пользовательского пула образов с общим изображением время достижения стабильного состояния составляет менее 25%, а задержка простоя виртуальной машины составляет до 30%.
- **Контроль версий образов и группирование образов для более эффективного управления.** Определение группирования образа содержит сведения о том, с какой целью данный образ был создан и для какой ОС он предназначен, а также информацию о его использовании. Группирование образов позволяет упростить управление образами. Дополнительные сведения см. в разделе [Определения образов](../virtual-machines/shared-image-galleries.md#image-definitions).

## <a name="prerequisites"></a>Предварительные требования

> [!NOTE]
> Необходимо пройти проверку подлинности с помощью Azure AD. Если вы используете shared-key-auth, проверка подлинности завершится сбоем.  

- **Учетная запись пакетной службы Azure.** Чтобы создать учетную запись пакетной службы Azure, обратитесь к примерам быстрого начала работы с пакетной службой с помощью [портала Azure](quick-create-portal.md) или [Azure CLI](quick-create-cli.md).

- **Образ из общей коллекции образов**. Для создания общего образа необходимо иметь ресурс управляемого образа или создать его. Образ должен быть создан на основе моментального снимка дисков ОС виртуальной машины и при необходимости ее подключенных дисков данных.

> [!NOTE]
> Если общий образ находится не в той же подписке, что и учетная запись пакетной службы, необходимо [зарегистрировать поставщик ресурсов Microsoft.BatCH](../azure-resource-manager/management/resource-providers-and-types.md#register-resource-provider) для этой подписки. Две подписки должны находиться в одном клиенте Azure AD.
>
> Образ может находиться в другом регионе, если в нем есть реплики в том же регионе, что и учетная запись пакетной службы.

Если вы используете приложение Azure AD для создания пользовательского пула образов с общим образом галереи образов, этому приложению должна быть предоставлена [Встроенная роль Azure](../role-based-access-control/rbac-and-directory-admin-roles.md#azure-roles) , предоставляющая доступ к общему образу. Вы можете предоставить этот доступ в портал Azure, перейдя к общему изображению, выбрав **Управление доступом (IAM)** и добавив назначение роли для приложения.

## <a name="prepare-a-shared-image"></a>Подготовка общего образа

В Azure можно подготовить общий образ из управляемого образа, который можно создать из:

- моментальных снимков ОС и дисков данных виртуальной машины Azure;
- обобщенной виртуальной машины Azure с управляемыми дисками;
- обобщенного локального виртуального жесткого диска, отправленного в облако.

> [!NOTE]
> Пакетная обработка поддерживает только обобщенные общие образы; для создания пула нельзя использовать специализированный общий образ.

Ниже приведены действия по подготовке виртуальной машины, созданию моментального снимка и созданию образа на основе моментального снимка.

### <a name="prepare-a-vm"></a>Подготовка виртуальной машины

Если для образа создается новая виртуальная машина, для управляемого образа в качестве базового можно использовать собственный образ из Azure Marketplace, поддерживаемый пакетной службой. В качестве базового образа можно использовать только собственные образы Azure Marketplace. Чтобы вывести полный список ссылок на образы из Azure Marketplace, поддерживаемые пакетной службой Azure, см. команду из раздела о [получении списка номеров SKU агента узла](/java/api/com.microsoft.azure.batch.protocol.accounts.listnodeagentskus).

> [!NOTE]
> В качестве базового образа нельзя использовать образы сторонних производителей, для которых предусматриваются дополнительные условия покупки и лицензия. Сведения об образах из Marketplace, см. в руководствах по виртуальным машинам [Linux](../virtual-machines/linux/cli-ps-findimage.md#deploy-an-image-with-marketplace-terms) или [Windows](../virtual-machines/windows/cli-ps-findimage.md#deploy-an-image-with-marketplace-terms).

При создании виртуальных машин следуйте приведенным ниже рекомендациям.

- Убедитесь, что виртуальная машина создается с управляемым диском. Это параметр хранилища по умолчанию при создании виртуальной машины.
- На виртуальной машине не следует устанавливать расширения Azure, например расширение пользовательских скриптов. Если образ содержит предварительно установленное расширение, в Azure могут возникнуть проблемы при развертывании пула пакетной службы.
- При использовании подключенных дисков данных необходимо подключить и отформатировать диски в виртуальной машине, чтобы использовать их.
- Убедитесь, что предоставленный базовый образ операционной системы использует этот временный диск по умолчанию. Сейчас агент узла пакетной службы ожидает этот диск.
- Убедитесь, что диск ОС не зашифрован.
- После запуска виртуальной машины подключитесь к ней с помощью RDP (для Windows) или SSH (для Linux). Установите все необходимое программное обеспечение или скопируйте необходимые данные.
- Для ускорения подготовки пула используйте [параметр кэш диска ReadWrite](../virtual-machines/premium-storage-performance.md#disk-caching) для диска ОС виртуальной машины.

### <a name="create-a-vm-snapshot"></a>Создание моментального снимка виртуальной машины

Моментальный снимок — это полная копия виртуального жесткого диска, доступная только для чтения. Чтобы создать моментальный снимок дисков ОС или дисков данных виртуальной машины, можно использовать портал Azure или средства командной строки. Описание действий и параметров, используемых для создания моментального снимка, см. в руководстве по виртуальным машинам [Linux](../virtual-machines/linux/snapshot-copy-managed-disk.md) или [Windows](../virtual-machines/windows/snapshot-copy-managed-disk.md).

### <a name="create-an-image-from-one-or-more-snapshots"></a>Создание образа на основе одного или нескольких моментальных снимков

Чтобы создать управляемый образ на основе моментального снимка, используйте средства командной строки Azure, такие как команда [az image create](/cli/azure/image). Можно создать образ, указав моментальный снимок диска ОС и при необходимости один или несколько моментальных снимков дисков данных.

### <a name="create-a-shared-image-gallery"></a>Создание Общей коллекции образов.

После успешного создания управляемого образа необходимо создать общую коллекцию образов, чтобы сделать настраиваемый образ доступным. Сведения о создании общей коллекции образов для своих образов см. в разделе [Создание общей коллекции образов с помощью Azure CLI](../virtual-machines/shared-images-cli.md) или [Создания общей коллекции образов с помощью портала Azure](../virtual-machines/linux/shared-images-portal.md).

## <a name="create-a-pool-from-a-shared-image-using-the-azure-cli"></a>Создание пула из общего образа с помощью Azure CLI

Чтобы создать пул из общего образа с помощью Azure CLI, используйте команду `az batch pool create`. Укажите идентификатор общего образа в поле `--image`. Убедитесь, что тип ОС и номер SKU соответствуют версиям, указанным в параметре `--node-agent-sku-id`

> [!NOTE]
> Необходимо пройти проверку подлинности с помощью Azure AD. Если вы используете shared-key-auth, проверка подлинности завершится сбоем.  

```azurecli
az batch pool create \
    --id mypool --vm-size Standard_A1_v2 \
    --target-dedicated-nodes 2 \
    --image "/subscriptions/{sub id}/resourceGroups/{resource group name}/providers/Microsoft.Compute/galleries/{gallery name}/images/{image definition name}/versions/{version id}" \
    --node-agent-sku-id "batch.node.ubuntu 16.04"
```

## <a name="create-a-pool-from-a-shared-image-using-c"></a>Создание пула из общего образа с помощью C#

Кроме того, можно создать пул из общего образа с помощью пакета SDK для C#.

```csharp
private static VirtualMachineConfiguration CreateVirtualMachineConfiguration(ImageReference imageReference)
{
    return new VirtualMachineConfiguration(
        imageReference: imageReference,
        nodeAgentSkuId: "batch.node.windows amd64");
}

private static ImageReference CreateImageReference()
{
    return new ImageReference(
        virtualMachineImageId: "/subscriptions/{sub id}/resourceGroups/{resource group name}/providers/Microsoft.Compute/galleries/{gallery name}/images/{image definition name}/versions/{version id}");
}

private static void CreateBatchPool(BatchClient batchClient, VirtualMachineConfiguration vmConfiguration)
{
    try
    {
        CloudPool pool = batchClient.PoolOperations.CreatePool(
            poolId: PoolId,
            targetDedicatedComputeNodes: PoolNodeCount,
            virtualMachineSize: PoolVMSize,
            virtualMachineConfiguration: vmConfiguration);

        pool.Commit();
    }
    ...
}
```

## <a name="create-a-pool-from-a-shared-image-using-python"></a>Создание пула из общего образа с помощью Python

Вы также можете создать пул из общего образа с помощью пакета SDK для Python: 

```python
# Import the required modules from the
# Azure Batch Client Library for Python
import azure.batch as batch
import azure.batch.models as batchmodels
from azure.common.credentials import ServicePrincipalCredentials

# Specify Batch account and service principal account credentials
account = "{batch-account-name}"
batch_url = "{batch-account-url}"
ad_client_id = "{sp-client-id}"
ad_tenant = "{tenant-id}"
ad_secret = "{sp-secret}"

# Pool settings
pool_id = "LinuxNodesSamplePoolPython"
vm_size = "STANDARD_D2_V3"
node_count = 1

# Initialize the Batch client with Azure AD authentication
creds = ServicePrincipalCredentials(
    client_id=ad_client_id,
    secret=ad_secret,
    tenant=ad_tenant,
    resource="https://batch.core.windows.net/"
)
client = batch.BatchServiceClient(creds, batch_url)

# Configure the start task for the pool
start_task = batchmodels.StartTask(
    command_line="printenv AZ_BATCH_NODE_STARTUP_DIR"
)
start_task.run_elevated = True

# Create an ImageReference which specifies the image from
# Shared Image Gallery to install on the nodes.
ir = batchmodels.ImageReference(
    virtual_machine_image_id="/subscriptions/{sub id}/resourceGroups/{resource group name}/providers/Microsoft.Compute/galleries/{gallery name}/images/{image definition name}/versions/{version id}"
)

# Create the VirtualMachineConfiguration, specifying
# the VM image reference and the Batch node agent to
# be installed on the node.
vmc = batchmodels.VirtualMachineConfiguration(
    image_reference=ir,
    node_agent_sku_id="batch.node.ubuntu 18.04"
)

# Create the unbound pool
new_pool = batchmodels.PoolAddParameter(
    id=pool_id,
    vm_size=vm_size,
    target_dedicated_nodes=node_count,
    virtual_machine_configuration=vmc,
    start_task=start_task
)

# Create pool in the Batch service
client.pool.add(new_pool)
```

## <a name="create-a-pool-from-a-shared-image-using-the-azure-portal"></a>Создание пула из общего образа на портале Azure

Чтобы создать пул из общего образа на портале Azure, выполните указанные ниже действия.

1. Откройте [портал Azure](https://portal.azure.com).
1. Перейдите в раздел **Учетные записи пакетной службы** и выберите свою учетную запись.
1. Выберите **Пулы**, а затем — **Добавить** для создания нового пула.
1. В разделе **Тип образа** выберите **Общая коллекция образов**.
1. Выполните действия в остальных разделах со сведениями об управляемом образе.
1. Щелкните **ОК**.

![Создайте пул из общего образа с помощью портала.](media/batch-sig-images/create-custom-pool.png)

## <a name="considerations-for-large-pools"></a>Рекомендации по созданию крупных пулов

Если вы планируете создать пул из сотен или тысяч виртуальных машин или более с помощью общего образа, используйте следующие рекомендации.

- **Номера реплик общей коллекции образов.**  Для каждого пула, имеющего до 300 экземпляров, рекомендуется разместить по крайней мере одну реплику. Например, при создании пула с 3000 виртуальными машинами необходимо иметь по крайней мере 10 реплик образа. Для повышения производительности рекомендуется хранить больше реплик, чем минимальные требования.

- **Время ожидания для изменения размера.** Если в пуле содержится фиксированное число узлов (если он не выполняет автомасштабирование), увеличьте значение свойства `resizeTimeout` пула в зависимости от размера пула. Для каждых 1000 виртуальных машин рекомендуемое время ожидания изменения размера составляет не менее 15 минут. Например, рекомендуемое время ожидания изменения размера для пула с 2000 виртуальных машин составляет не менее 30 минут.

## <a name="next-steps"></a>Дальнейшие действия

- Подробные сведения о пакетной службе см. в статье [Рабочий процесс и ресурсы пакетной службы](batch-service-workflow-features.md).
- Узнайте подробнее об [общей коллекции образов](../virtual-machines/shared-image-galleries.md).