---
title: Служба Azure Kubernetes (AKS) с Соглашением об уровне обслуживания с гарантией времени доступности
description: Сведения о варианте Соглашения об уровне обслуживания с гарантией времени доступности для сервера API Azure Kubernetes Service (AKS).
services: container-service
ms.topic: conceptual
ms.date: 01/08/2021
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: fae95e6f3a73d7539f81e1486c4ad021f181aa11
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102176337"
---
# <a name="azure-kubernetes-service-aks-uptime-sla"></a>Соглашение об уровне обслуживания с гарантией времени доступности для Службы Azure Kubernetes (AKS)

Соглашение об уровне обслуживания с гарантией времени доступности — это возможность за дополнительную плату получить более высокий уровень обслуживания кластера. Соглашение об уровне обслуживания с гарантией времени доступности гарантирует 99,95% доступности для конечной точки сервера Kubernetes API любому кластеру, который использует [Зоны доступности][availability-zones], или 99,9% доступности для кластеров, которые не используют Зоны доступности. AKS использует реплики главного узла в доменах обновления и сбоя, чтобы обеспечить соблюдение Соглашения об уровне обслуживания.

Клиенты, которым требуется соглашение об уровне обслуживания для удовлетворения требований соответствия или требуют расширения соглашения об уровне обслуживания для конечных пользователей, должны включить эту функцию. Оно также может быть полезным для клиентов с критически важными рабочими нагрузками. Вариант Соглашения об уровне обслуживания с гарантией времени доступности совместно с Зонами доступности обеспечивает более высокий уровень доступности для сервера API Kubernetes.  

Клиенты по-прежнему смогут создавать неограниченное число бесплатных кластеров с целевым уровнем обслуживания 99,5% и по своему усмотрению выбирать Цель уровня обслуживания или Соглашение об уровне обслуживания с гарантией времени доступности.

> [!Important]
> Для кластеров с блокировкой исходящего трафика необходимо [открыть соответствующие порты](limit-egress-traffic.md).

## <a name="region-availability"></a>Доступность по регионам

* Соглашение об уровне обслуживания доступности доступно в общедоступных регионах и регионах Azure для государственных организаций, где [поддерживается AKS](https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service).
* Соглашение об уровне обслуживания доступности доступно для [частных кластеров AKS][private-clusters] во всех общедоступных регионах, где поддерживается AKS.

## <a name="sla-terms-and-conditions"></a>Условия Соглашения об уровне обслуживания

Соглашение об уровне обслуживания с гарантией времени доступности является платной возможностью и включается отдельно для каждого кластера. Цены на Соглашение об уровне обслуживания с гарантией времени доступности зависят от количества отдельных кластеров, но не от их размеров. Дополнительные сведения см. на странице [Цены на службу Azure Kubernetes (AKS)](https://azure.microsoft.com/pricing/details/kubernetes-service/).

## <a name="before-you-begin"></a>Перед началом

* Установка [Azure CLI](/cli/azure/install-azure-cli) версии 2.8.0 или более поздней

## <a name="creating-a-new-cluster-with-uptime-sla"></a>Создание нового кластера с соглашением об уровне обслуживания для времени работы

> [!NOTE]
> В настоящее время, если вы включили соглашение об уровне обслуживания "время работы", удалить его из кластера невозможно.

Чтобы создать новый кластер, защищенный Соглашением об уровне обслуживания с гарантией времени доступности, используйте Azure CLI.

В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli-interactive
# Create a resource group
az group create --name myResourceGroup --location eastus
```
Используйте [`az aks create`][az-aks-create] команду, чтобы создать кластер AKS. В следующем примере создается кластер *myAKSCluster* с одним узлом. Выполнение этой операции займет несколько минут:

```azurecli-interactive
# Create an AKS cluster with uptime SLA
az aks create --resource-group myResourceGroup --name myAKSCluster --uptime-sla --node-count 1
```
Через несколько минут выполнение команды завершается и отображаются сведения о кластере в формате JSON. В следующем фрагменте кода JSON показан платный уровень номера SKU, указывающий, что кластер включен с соглашением об уровне обслуживания времени работы:

```output
  },
  "sku": {
    "name": "Basic",
    "tier": "Paid"
  },
```

## <a name="modify-an-existing-cluster-to-use-uptime-sla"></a>Изменение существующего кластера для использования соглашения об уровне обслуживания времени работы

При необходимости можно обновить существующие кластеры, чтобы использовать соглашение об уровне обслуживания времени работы.

Если вы создали кластер AKS с предыдущими шагами, удалите группу ресурсов:

```azurecli-interactive
# Delete the existing cluster by deleting the resource group 
az group delete --name myResourceGroup --yes --no-wait
```

Создайте новую группу ресурсов:

```azurecli-interactive
# Create a resource group
az group create --name myResourceGroup --location eastus
```

Создайте новый кластер и не используйте соглашение об уровне обслуживания времени работы:

```azurecli-interactive
# Create a new cluster without uptime SLA
az aks create --resource-group myResourceGroup --name myAKSCluster--node-count 1
```

Используйте [`az aks update`][az-aks-update] команду, чтобы обновить существующий кластер:

```azurecli-interactive
# Update an existing cluster to use Uptime SLA
 az aks update --resource-group myResourceGroup --name myAKSCluster --uptime-sla
 ```

 В следующем фрагменте кода JSON показан платный уровень номера SKU, указывающий, что кластер включен с соглашением об уровне обслуживания времени работы:

 ```output
  },
  "sku": {
    "name": "Basic",
    "tier": "Paid"
  },
  ```

## <a name="clean-up"></a>Очистка

Чтобы избежать расходов, очистите созданные ресурсы. Чтобы удалить кластер, [`az group delete`][az-group-delete] удалите группу ресурсов AKS с помощью команды:

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```


## <a name="next-steps"></a>Дальнейшие действия

Примените [Зоны доступности][availability-zones], чтобы повысить уровень доступности для рабочих нагрузок кластера AKS.

Настройте для кластера [ограничение исходящего трафика](limit-egress-traffic.md).

<!-- LINKS - External -->
[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest
[region-availability]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service

<!-- LINKS - Internal -->
[vm-skus]: ../virtual-machines/sizes.md
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
[faq]: ./faq.md
[availability-zones]: ./availability-zones.md
[az-aks-create]: /cli/azure/aks?#az-aks-create
[limit-egress-traffic]: ./limit-egress-traffic.md
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-aks-update]: /cli/azure/aks#az_aks_update
[az-group-delete]: /cli/azure/group#az-group-delete
[private-clusters]: private-clusters.md
