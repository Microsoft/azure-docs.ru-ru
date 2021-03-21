---
title: Использование виртуальных узлов
titleSuffix: Azure Kubernetes Service
description: Общие сведения о том, как использовать виртуальный узел с Azure Kubernetes Services (AKS)
services: container-service
ms.topic: conceptual
ms.date: 02/17/2021
ms.custom: references_regions
ms.openlocfilehash: 3bba1155ec57db67968aec95d1d3386fc6cda006
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100634453"
---
# <a name="create-and-configure-an-azure-kubernetes-services-aks-cluster-to-use-virtual-nodes"></a>Создание и настройка кластера Azure Kubernetes Services (AKS) для использования виртуальных узлов

Чтобы быстро масштабировать рабочие нагрузки приложений в кластере AKS, можно использовать виртуальные узлы. Виртуальные узлы дают возможность быстро подготовить pod и оплачивать только время их выполнения (посекундно). Вам не придется ждать, пока средство автомасштабирования кластера Kubernetes развернет вычислительные узлы для виртуальных машин, на которых можно будет запустить дополнительные pod. Виртуальные узлы поддерживаются только для контейнеров pod и узлов Linux.

Надстройка виртуальных узлов для AKS основана на [виртуальной Kubelet][virtual-kubelet-repo]проекта с открытым исходным кодом.

В этой статье приводятся общие сведения о доступности регионов и требованиях к сети для использования виртуальных узлов, а также об известных ограничениях.

## <a name="regional-availability"></a>Доступность в регионах

Все регионы, где ACI поддерживает номера SKU виртуальной сети, поддерживаются для развертываний виртуальных узлов.

Для доступных номеров SKU ЦП и памяти в каждом регионе проверьте [доступность ресурсов для экземпляров контейнеров Azure для экземпляров контейнеров Azure в регионах Azure — группы контейнеров Linux](../container-instances/container-instances-region-availability.md#linux-container-groups) .

## <a name="network-requirements"></a>Требования к сети

Виртуальные узлы обеспечивают сетевое взаимодействие контейнеров pod, которые выполняются в Экземплярах контейнеров Azure (ACI) и кластере AKS. Для обеспечения этого взаимодействия создается подсеть виртуальной сети и назначаются делегированные разрешения. Виртуальные узлы работают только с кластерами AKS, созданными с помощью *Advanced* Networking (Azure CNI). По умолчанию кластеры AKS создаются с помощью *базовых* сетей (кубенет).

Для модулей Pod, работающих в службе "экземпляры контейнеров Azure" (ACI), требуется доступ к конечной точке сервера API AKS, чтобы настроить сеть.

## <a name="known-limitations"></a>Известные ограничения

Функциональные возможности виртуальных узлов сильно зависят от набора функций ACI. В дополнение к [квотам и ограничениям для экземпляров контейнеров Azure](../container-instances/container-instances-quotas.md)следующие сценарии пока не поддерживаются виртуальными узлами:

* Использование субъекта-службы для извлечения образов ACR. [Обходной путь](https://github.com/virtual-kubelet/azure-aci/blob/master/README.md#private-registry) — использование [секретов Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/#create-a-secret-by-providing-credentials-on-the-command-line).
* [Ограничения виртуальной сети](../container-instances/container-instances-vnet.md), включая пиринг виртуальных сетей, политики сети Kubernetes и передача исходящего трафика в Интернет при использовании групп безопасности сети.
* Контейнеры инициализации.
* [Псевдонимы узлов](https://kubernetes.io/docs/concepts/services-networking/add-entries-to-pod-etc-hosts-with-host-aliases/).
* [Аргументы](../container-instances/container-instances-exec.md#restrictions) для exec в ACI.
* [Daemonset](concepts-clusters-workloads.md#statefulsets-and-daemonsets) не будет выполнять развертывание модулей Pod на виртуальных узлах
* Виртуальные узлы поддерживают планирование контейнеров pod для Linux. Чтобы запланировать контейнеры Windows Server в ACI, можно вручную установить поставщик [Virtual Kubelet ACI](https://github.com/virtual-kubelet/azure-aci) с открытым кодом.
* Для виртуальных узлов требуются кластеры AKS с сетью CNI Azure.
* Использование диапазонов IP-адресов, прошедших авторизацию сервера API, для AKS.
* Общий доступ к файлам Azure для подключения к общему ресурсу поддерживается для [общего назначения версии v1](../storage/common/storage-account-overview.md#types-of-storage-accounts). Следуйте инструкциям по подключению [тома с файловым ресурсом Azure](azure-files-volume.md) .
* Использование IPv6 не поддерживается.

## <a name="next-steps"></a>Дальнейшие действия

Настройте виртуальные узлы для кластеров:

- [Создание виртуальных узлов с помощью Azure CLI](virtual-nodes-cli.md)
- [Создание виртуальных узлов в Службе Azure Kubernetes (AKS) с помощью портала](virtual-nodes-portal.md)

Виртуальные узлы — это зачастую лишь один из компонентов, необходимых для масштабирования решения в AKS. Дополнительные сведения о таких решениях см. в следующих статьях:

- [Масштабирование приложений в Службе Azure Kubernetes (AKS)][aks-hpa]
- [Автомасштабирование кластера с помощью службы Azure Kubernetes (AKS)][aks-cluster-autoscaler]
- [Пример автомасштабирования для виртуальных узлов][virtual-node-autoscale]
- [Дополнительные сведения о библиотеке Virtual Kubelet с открытым кодом][virtual-kubelet-repo]

<!-- LINKS - external -->
[aks-hpa]: tutorial-kubernetes-scale.md
[aks-cluster-autoscaler]: ./cluster-autoscaler.md
[virtual-node-autoscale]: https://github.com/Azure-Samples/virtual-node-autoscale
[virtual-kubelet-repo]: https://github.com/virtual-kubelet/virtual-kubelet
