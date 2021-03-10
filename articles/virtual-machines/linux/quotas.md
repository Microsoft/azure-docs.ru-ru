---
title: Квоты на виртуальные ЦП
description: Дополнительные сведения о квотах виртуальных ЦП для Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: quota
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 05/31/2018
ms.author: cynthn
ms.openlocfilehash: 1b9c0d50754d582ca7ada5d0b46c6f998b59d3ae
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102549572"
---
# <a name="check-vcpu-quotas-using-the-azure-cli"></a>Проверьте квоты виртуальных ЦП с помощью Azure CLI

Квоты виртуальных ЦП для виртуальных машин и масштабируемых наборов виртуальных машин разделяются на два уровня для каждой подписки в каждом регионе. Первый уровень — это общие региональные виртуальные ЦП, а второй — ядра для различных семейств размеров виртуальных машин, например виртуальные ЦП серии D. При развертывании новой виртуальной машины ее виртуальные ЦП не должны превышать квоту на виртуальные ЦП для определенного семейства размеров виртуальных машин или квоту на общие региональные виртуальные ЦП. При превышении любой из этих квот развертывание виртуальной машины будет запрещено. Имеется также квота на общее количество виртуальных машин в регионе. Сведения о каждой из этих квот можно просмотреть в разделе **Использование и квоты** на странице **Подписки**[портала Azure](https://portal.azure.com). Кроме того, можно запросить значения с помощью Azure CLI.

> [!NOTE]
> Квота рассчитывается на основе общего числа используемых ядер, как выделенных, так и освобожденных. Если требуются дополнительные ядра, [запросите увеличение квоты](../../azure-portal/supportability/resource-manager-core-quotas-request.md) или удалите виртуальные машины, которые больше не нужны. 


## <a name="check-usage"></a>Проверка использования

Данные об использовании квоты можно проверить с помощью команды [az vm list-usage](/cli/azure/vm).

```azurecli-interactive
az vm list-usage --location "East US" -o table
```

Результат должен выглядеть следующим образом.


```
Name                                CurrentValue    Limit
--------------------------------  --------------  -------
Availability Sets                              0     2000
Total Regional vCPUs                          29      100
Virtual Machines                               7    10000
Virtual Machine Scale Sets                     0     2000
Standard DSv3 Family vCPUs                     8      100
Standard DSv2 Family vCPUs                     3      100
Standard Dv3 Family vCPUs                      2      100
Standard D Family vCPUs                        8      100
Standard Dv2 Family vCPUs                      8      100
Basic A Family vCPUs                           0      100
Standard A0-A7 Family vCPUs                    0      100
Standard A8-A11 Family vCPUs                   0      100
Standard DS Family vCPUs                       0      100
Standard G Family vCPUs                        0      100
Standard GS Family vCPUs                       0      100
Standard F Family vCPUs                        0      100
Standard FS Family vCPUs                       0      100
Standard Storage Managed Disks                 5    10000
Premium Storage Managed Disks                  5    10000
```

## <a name="reserved-vm-instances"></a>Зарезервированные экземпляры виртуальных машин
Зарезервированные экземпляры виртуальных машин, которые ограничиваются одной подпиской и не имеют гибкости размеров ВМ, добавляют новый аспект к квотам на виртуальные ЦП. Эти значения описывают число экземпляров указанного размера, которые можно развернуть в подписке. Они выступают в качестве заполнителя в системе квот, позволяя убедиться, что квота зарезервирована, а также обеспечить развертывание зарезервированных экземпляров Azure в подписке. Например, если в определенной подписке есть 10 зарезервированных экземпляров Standard_D1, лимит использования для них будет составлять 10 единиц. Таким образом в Azure будет обеспечиваться наличие по крайней мере 10 ЦП в рамках квоты на общие региональные виртуальные ЦП, которые будут использоваться для экземпляров Standard_D1, а также наличие хотя бы 10 виртуальных ЦП в рамках квоты на ЦП семейства Standard D, которые будут использоваться для экземпляров Standard_D1.

Если увеличение квоты необходимо для приобретения зарезервированного экземпляра с одной подпиской, вы можете [запросить увеличение квоты](../../azure-portal/supportability/resource-manager-core-quotas-request.md) по подписке.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о выставлении счетов и квотах см. в статье [Подписка Azure, границы, квоты и ограничения службы](../../azure-resource-manager/management/azure-subscription-service-limits.md?toc=/azure/billing/TOC.json).
