---
title: Обновление образов узлов Azure Kubernetes Service (AKS)
description: Узнайте, как обновить образы на узлах кластера AKS и пулах узлов.
ms.service: container-service
ms.topic: conceptual
ms.date: 11/25/2020
ms.author: jpalma
ms.openlocfilehash: 83d7d48922806334e2b49494fe0ef1d15e1a7a6a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96531485"
---
# <a name="azure-kubernetes-service-aks-node-image-upgrade"></a>Обновление образа узла службы Azure Kubernetes Service (AKS)

AKS поддерживает обновление образов на узле, так что вы будете в курсе последних обновлений операционной системы и среды выполнения. AKS предоставляет один новый образ в неделю с последними обновлениями, поэтому рекомендуется регулярно обновлять образы узла для получения последних компонентов, включая обновления Linux или Windows. В этой статье показано, как обновить образы узлов кластера AKS и как обновить образы пула узлов без обновления версии Kubernetes.

Дополнительные сведения о последних образах, предоставляемых AKS, см. в [заметках о выпуске AKS](https://github.com/Azure/AKS/releases).

Сведения об обновлении версии Kubernetes для кластера см. в статье [Обновление кластера AKS][upgrade-cluster].

> [!NOTE]
> Кластер AKS должен использовать масштабируемые наборы виртуальных машин для узлов.

## <a name="check-if-your-node-pool-is-on-the-latest-node-image"></a>Проверьте, находится ли пул узлов на последнем образе узла

Чтобы увидеть последнюю версию образа узла, доступную для пула узлов, выполните следующую команду: 

```azurecli
az aks nodepool get-upgrades \
    --nodepool-name mynodepool \
    --cluster-name myAKSCluster \
    --resource-group myResourceGroup
```

В выходных данных можно увидеть, как показано в `latestNodeImageVersion` следующем примере:

```output
{
  "id": "/subscriptions/XXXX-XXX-XXX-XXX-XXXXX/resourcegroups/myResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster/agentPools/nodepool1/upgradeProfiles/default",
  "kubernetesVersion": "1.17.11",
  "latestNodeImageVersion": "AKSUbuntu-1604-2020.10.28",
  "name": "default",
  "osType": "Linux",
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.ContainerService/managedClusters/agentPools/upgradeProfiles",
  "upgrades": null
}
```

Итак `nodepool1` , для последнего доступного образа узла это `AKSUbuntu-1604-2020.10.28` . Теперь его можно сравнить с текущей версией образа узла, используемой пулом узлов, выполнив:

```azurecli
az aks nodepool show \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --query nodeImageVersion
```

Пример выходных данных:

```output
"AKSUbuntu-1604-2020.10.08"
```

Поэтому в этом примере можно обновить текущую `AKSUbuntu-1604-2020.10.08` версию образа до последней версии `AKSUbuntu-1604-2020.10.28` . 

## <a name="upgrade-all-nodes-in-all-node-pools"></a>Обновление всех узлов во всех пулах узлов

Обновление образа узла выполняется с помощью `az aks upgrade` . Чтобы обновить образ узла, используйте следующую команду:

```azurecli
az aks upgrade \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-image-only
```

Во время обновления проверьте состояние образов узлов, выполнив следующую `kubectl` команду, чтобы получить метки и отфильтровать сведения о текущем образе узла:

```azurecli
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.labels.kubernetes\.azure\.com\/node-image-version}{"\n"}{end}'
```

После завершения обновления используйте `az aks show` для получения обновленных сведений о пуле узлов. Текущий образ узла отображается в `nodeImageVersion` свойстве.

```azurecli
az aks show \
    --resource-group myResourceGroup \
    --name myAKSCluster
```

## <a name="upgrade-a-specific-node-pool"></a>Обновление определенного пула узлов

Обновление образа в пуле узлов аналогично обновлению образа в кластере.

Чтобы обновить образ ОС пула узлов без выполнения обновления кластера Kubernetes, используйте `--node-image-only` параметр в следующем примере:

```azurecli
az aks nodepool upgrade \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --node-image-only
```

Во время обновления проверьте состояние образов узлов, выполнив следующую `kubectl` команду, чтобы получить метки и отфильтровать сведения о текущем образе узла:

```azurecli
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.labels.kubernetes\.azure\.com\/node-image-version}{"\n"}{end}'
```

После завершения обновления используйте `az aks nodepool show` для получения обновленных сведений о пуле узлов. Текущий образ узла отображается в `nodeImageVersion` свойстве.

```azurecli
az aks nodepool show \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool
```

## <a name="upgrade-node-images-with-node-surge"></a>Обновление образов узлов с пререзкостью узлов

Чтобы ускорить процесс обновления образа узла, можно обновить образы узлов, используя настраиваемое значение всплеска узлов. По умолчанию AKS использует один дополнительный узел для настройки обновлений.

Если вы хотите увеличить скорость обновления, используйте `--max-surge` значение, чтобы настроить число узлов, используемых для обновления, чтобы они выполнялись быстрее. Дополнительные сведения о компромиссах различных `--max-surge` параметров см. в разделе [Настройка обновления всплесков узлов][max-surge].

Следующая команда задает максимальное значение всплеска для выполнения обновления образа узла:

```azurecli
az aks nodepool upgrade \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --max-surge 33% \
    --node-image-only \
    --no-wait
```

Во время обновления проверьте состояние образов узлов, выполнив следующую `kubectl` команду, чтобы получить метки и отфильтровать сведения о текущем образе узла:

```azurecli
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.labels.kubernetes\.azure\.com\/node-image-version}{"\n"}{end}'
```

Используйте `az aks nodepool show` для получения обновленных сведений о пуле узлов. Текущий образ узла отображается в `nodeImageVersion` свойстве.

```azurecli
az aks nodepool show \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool
```

## <a name="next-steps"></a>Дальнейшие действия

- Сведения о последних образах узлов см. в [заметках о выпуске AKS](https://github.com/Azure/AKS/releases) .
- Узнайте, как обновить версию Kubernetes с помощью [обновления кластера AKS][upgrade-cluster].
- [Автоматическое применение обновлений кластера и пула узлов с помощью действий GitHub][github-schedule]
- Узнайте больше о нескольких пулах узлов и об обновлении пулов узлов с помощью [создания нескольких пулов узлов и управления ими][use-multiple-node-pools].

<!-- LINKS - internal -->
[upgrade-cluster]: upgrade-cluster.md
[github-schedule]: node-upgrade-github-actions.md
[use-multiple-node-pools]: use-multiple-node-pools.md
[max-surge]: upgrade-cluster.md#customize-node-surge-upgrade
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
