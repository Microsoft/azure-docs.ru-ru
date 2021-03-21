---
title: Создание пула с включенным шифрованием дисков
description: Узнайте, как использовать конфигурацию шифрования дисков для шифрования узлов с помощью ключа, управляемого платформой.
author: pkshultz
ms.topic: how-to
ms.date: 01/27/2021
ms.author: peshultz
ms.custom: references_regions
ms.openlocfilehash: 41fc827459b454e2bcb120a925cdab8fcd46e310
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99055320"
---
# <a name="create-a-pool-with-disk-encryption-enabled"></a>Создание пула с включенным шифрованием дисков

При создании пула пакетной службы Azure с помощью [конфигурации виртуальной машины](nodes-and-pools.md#virtual-machine-configuration)можно зашифровать вычисленные узлы в пуле с помощью ключа, управляемого платформой, указав конфигурацию шифрования диска.

В этой статье объясняется, как создать пул пакетной службы с включенным шифрованием дисков.

> [!IMPORTANT]
> Поддержка шифрования на узле с помощью управляемого платформой ключа в пакетной службе Azure в настоящее время находится в общедоступной предварительной версии для восточной части США, западной части США 2, юго-центральной части США, US Gov (Вирджиния) и US Gov (Аризона) регионов.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="why-use-a-pool-with-disk-encryption-configuration"></a>Зачем использовать пул с конфигурацией шифрования дисков?

С помощью пула пакетной службы вы можете получать доступ к данным и хранить их на временных дисках, а также в ОПЕРАЦИОННЫХ узлах. Шифрование диска на стороне сервера с помощью управляемого платформой ключа обеспечит защиту этих данных с низкой нагрузкой и удобством.

Пакетная обработка применяет одну из этих технологий шифрования дисков на расчетных узлах на основе конфигурации пула и региональной поддержки.

- [Управляемое шифрование диска неактивных с ключами, управляемыми платформой](../virtual-machines/disk-encryption.md#platform-managed-keys)
- [Шифрование на узле с помощью управляемого платформой ключа](../virtual-machines/disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data)
- [Дисковое шифрование Azure](../security/fundamentals/azure-disk-encryption-vms-vmss.md)

Вы не сможете указать, какой метод шифрования будет применяться к узлам в пуле. Вместо этого укажите целевые диски, которые необходимо зашифровать на своих узлах, а Пакетная службы может выбрать соответствующий метод шифрования, гарантируя, что указанные диски будут зашифрованы на кластерном узле.

> [!IMPORTANT]
> При создании пула с [пользовательским образом](batch-sig-images.md)шифрование дисков можно включить только при использовании виртуальных машин Windows.

## <a name="azure-portal"></a>Портал Azure

При создании пула пакетной службы на портал Azure выберите **темпораридиск** или **Осандтемпораридиск** в разделе **Конфигурация шифрования диска**.

:::image type="content" source="media/disk-encryption/portal-view.png" alt-text="Снимок экрана с параметром конфигурации шифрования диска в портал Azure.":::

После создания пула можно увидеть целевые объекты конфигурации шифрования дисков в разделе **Свойства** пула.

:::image type="content" source="media/disk-encryption/configuration-target.png" alt-text="Снимок экрана, показывающий целевые объекты конфигурации шифрования дисков в портал Azure.":::

## <a name="examples"></a>Примеры

В следующих примерах показано, как зашифровать операционную систему и временные диски в пуле пакетной службы с помощью пакета SDK для .NET пакетной службы, пакетной REST API и Azure CLI.

### <a name="batch-net-sdk"></a>Пакет SDK .NET для пакетной службы

```csharp
pool.VirtualMachineConfiguration.DiskEncryptionConfiguration = new DiskEncryptionConfiguration(
    targets: new List<DiskEncryptionTarget> { DiskEncryptionTarget.OsDisk, DiskEncryptionTarget.TemporaryDisk }
    );
```

### <a name="batch-rest-api"></a>REST API пакетной службы

URL-АДРЕС REST API:

```
POST {batchURL}/pools?api-version=2020-03-01.11.0
client-request-id: 00000000-0000-0000-0000-000000000000
```

Тело запроса:

```
"pool": {
    "id": "pool2",
    "vmSize": "standard_a1",
    "virtualMachineConfiguration": {
        "imageReference": {
            "publisher": "Canonical",
            "offer": "UbuntuServer",
            "sku": "18.04-LTS"
        },
        "diskEncryptionConfiguration": {
            "targets": [
                "OsDisk",
                "TemporaryDisk"
            ]
        }
        "nodeAgentSKUId": "batch.node.ubuntu 18.04"
    },
    "resizeTimeout": "PT15M",
    "targetDedicatedNodes": 5,
    "targetLowPriorityNodes": 0,
    "taskSlotsPerNode": 3,
    "enableAutoScale": false,
    "enableInterNodeCommunication": false
}
```

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
az batch pool create \
    --id diskencryptionPool \
    --vm-size Standard_DS1_V2 \
    --target-dedicated-nodes 2 \
    --image canonical:ubuntuserver:18.04-LTS \
    --node-agent-sku-id "batch.node.ubuntu 18.04" \
    --disk-encryption-targets OsDisk TemporaryDisk
```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [шифровании хранилище дисков Azure на стороне сервера](../virtual-machines/disk-encryption.md).
- Подробные сведения о пакетной службе см. в статье [Рабочий процесс и ресурсы пакетной службы](batch-service-workflow-features.md).
