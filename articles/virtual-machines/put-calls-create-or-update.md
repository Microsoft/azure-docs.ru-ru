---
title: Вызовы PUT для операций создания или обновления
description: Помещайте вызовы для операций создания или обновления ресурсов вычислений
author: mimckitt
ms.author: mimckitt
ms.reviewer: cynthn
ms.topic: conceptual
ms.service: virtual-machines
ms.date: 08/4/2020
ms.custom: avverma
ms.openlocfilehash: d6ee4179dce905d637e933743ade7452a2484077
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91978566"
---
# <a name="put-calls-for-creation-or-updates-on-compute-resources"></a>Размещение вызовов для создания или обновления ресурсов вычислений

`Microsoft.Compute` ресурсы не поддерживают стандартное определение семантики *http-размещения* . Вместо этого эти ресурсы используют семантику исправления как для команд размещения, так и для ИСПРАВЛЕНИЙ.

При необходимости операции **создания** применяют значения по умолчанию. Однако **обновления** ресурсов, выполненные с помощью инструкции установки или исправления, не применяют свойства по умолчанию. Операции **обновления** применяются к применению тщательной семантики исправлений.

Например, для свойства диска `caching` виртуальной машины по умолчанию `ReadWrite` задано значение, если ресурс является диском ОС.

```json
    "storageProfile": {
      "osDisk": {
        "name": "myVMosdisk",
        "image": {
          "uri": "http://{existing-storage-account-name}.blob.core.windows.net/{existing-container-name}/{existing-generalized-os-image-blob-name}.vhd"
        },
        "osType": "Windows",
        "createOption": "FromImage",
        "caching": "ReadWrite",
        "vhd": {
          "uri": "http://{existing-storage-account-name}.blob.core.windows.net/{existing-container-name}/myDisk.vhd"
        }
      }
    },
```

Однако для операций **обновления** , когда свойство не используется или передается значение *null* , оно остается неизменным и значения по умолчанию не отображаются.

Это важно при отправке операций обновления в ресурс с намерением удалить связь. Если ресурс является `Microsoft.Compute` ресурсом, соответствующее свойство, которое необходимо удалить, должно быть явно вызвано и назначено значение. Чтобы добиться этого, пользователи могут передать пустую строку, например **""**. Это позволит платформе удалить эту связь.

> [!IMPORTANT]
> "Обновление" элемента массива не поддерживается. Вместо этого клиент должен выполнить запрос на размещение или исправление со всем содержимым обновленного массива. Например, чтобы отсоединить диск данных от виртуальной машины, выполните запрос GET для получения текущей модели виртуальной машины, удалите диск, который нужно отсоединить от `properties.storageProfile.dataDisks` , и выполните запрос на размещение с обновленной сущностью виртуальной машины.

## <a name="examples"></a>Примеры

### <a name="correct-payload-to-remove-a-proximity-placement-groups-association"></a>Правильная полезная нагрузка для удаления связи групп размещения с учетом расположения

`
{ "location": "westus", "properties": { "platformFaultDomainCount": 2, "platformUpdateDomainCount": 20, "proximityPlacementGroup": "" } }
`

### <a name="incorrect-payloads-to-remove-a-proximity-placement-groups-association"></a>Неверные полезные данные для удаления связи групп размещения с учетом расположения

`
{ "location": "westus", "properties": { "platformFaultDomainCount": 2, "platformUpdateDomainCount": 20, "proximityPlacementGroup": null } }
`

`
{ "location": "westus", "properties": { "platformFaultDomainCount": 2, "platformUpdateDomainCount": 20 } }
`

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о создании или обновлении вызовов для [виртуальных машин](/rest/api/compute/virtualmachines/createorupdate) и [масштабируемых наборов виртуальных машин](/rest/api/compute/virtualmachinescalesets/createorupdate)