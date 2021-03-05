---
title: Использование групп размещения с учетом расположения для сокращения задержки кластеров службы Azure Kubernetes Service (AKS)
description: Узнайте, как использовать группы размещения с учетом расположения для уменьшения задержки для рабочих нагрузок кластера AKS.
services: container-service
manager: gwallace
ms.topic: article
ms.date: 10/19/2020
ms.openlocfilehash: c0c1bf83bf5a816debe61cab0ceab856bfbd062f
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102181182"
---
# <a name="reduce-latency-with-proximity-placement-groups"></a>Сокращение задержки с помощью групп размещения с учетом расположения

> [!Note]
> При использовании групп размещения с учетом расположения в AKS применяется только к узлам агента. Улучшена скорость между узлом и соответствующим размещенным модулем Pod. Совместное размещение не влияет на размещение плоскости управления кластера.

При развертывании приложения в Azure распределение экземпляров виртуальной машины между регионами или зонами доступности создает задержку в сети, что может повлиять на общую производительность приложения. Группа размещения с учетом расположения — это логическая группировка, используемая для того, чтобы ресурсы вычислений Azure физически размещались близко друг к другу. Для некоторых приложений, таких как игры, инженерные моделирования и торговля с высокой частотой (ХФТ), требуется низкая задержка и задачи, которые быстро выполняются. Для сценариев высокопроизводительных вычислений (HPC) рекомендуется использовать [группы размещения](../virtual-machines/co-location.md#proximity-placement-groups) (ППГ) для пулов узлов кластера.

## <a name="before-you-begin"></a>Перед началом

Для работы с этой статьей требуется Azure CLI версии 2,14 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][azure-cli-install].

### <a name="limitations"></a>Ограничения

* Группа размещения с учетом расположения может сопоставляться только с одной зоной доступности.
* Пул узлов должен использовать масштабируемые наборы виртуальных машин, чтобы связать группу размещения с близкой назначением.
* Пул узлов может связать группу размещения с учетом расположения в пуле узлов только время создания.

## <a name="node-pools-and-proximity-placement-groups"></a>Пулы узлов и группы размещения близости

Первый ресурс, развертываемый с помощью группы размещения с учетом расположения, подключается к определенному центру обработки данных. Дополнительные ресурсы, развернутые с одной и той же группой размещения, находятся в одном центре обработки данных. После того как все ресурсы, использующие группу размещения с учетом расположения, были остановлены (освобождены) или удалены, они больше не будут присоединены.

* Многие пулы узлов можно связать с одной группой размещения близости.
* Пул узлов может быть связан только с одной группой размещения близости.

### <a name="configure-proximity-placement-groups-with-availability-zones"></a>Настройка групп размещения близкого взаимодействия с зонами доступности

> [!NOTE]
> Хотя для групп размещения с близкой близости требуется пул узлов для использования не более чем одной зоны доступности, [базовое соглашение об уровне обслуживания виртуальной машины Azure](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_9/) , равное 99,9%, по-прежнему действует для виртуальных машин в одной зоне.

Группы размещения близкого взаимодействия являются концепцией пула узлов и связаны с каждым пулом отдельных узлов. Использование ресурса ППГ не влияет на доступность плоскости управления AKS. Это может повлиять на способ разработки кластера с зонами. Чтобы обеспечить распределение кластера между несколькими зонами, рекомендуется использовать следующую конструкцию.

* Подготавливает кластер с первым системным пулом, используя 3 зоны, и не сопоставлена группа размещения с учетом расположения. Это гарантирует, что системные модули памяти помещаются в пул выделенных узлов, который будет распределяться по нескольким зонам.
* Добавьте дополнительные пулы узлов пользователей с уникальной зоной и группой размещения с учетом расположения, связанной с каждым пулом. Например, nodepool1 в зоне 1 и PPG1, nodepool2 в зоне 2 и PPG2, nodepool3 в зоне 3 с PPG3. Это гарантирует, что на уровне кластера узлы распределяются по нескольким зонам, а каждый пул отдельных узлов совместно размещается в указанной зоне с выделенным ресурсом ППГ.

## <a name="create-a-new-aks-cluster-with-a-proximity-placement-group"></a>Создание нового кластера AKS с группой размещения с учетом расположения

В следующем примере используется команда [AZ Group Create][az-group-create] для создания группы ресурсов с именем *myResourceGroup* в регионе *centralus* . После этого кластер AKS с именем *myAKSCluster* создается с помощью команды [AZ AKS Create][az-aks-create] .

Ускоренная сеть значительно улучшает производительность сети виртуальных машин. В идеале используйте группы размещения близости в сочетании с ускоренной сетью. По умолчанию AKS использует ускоренную сеть на [поддерживаемых экземплярах виртуальных машин](../virtual-network/create-vm-accelerated-networking-cli.md?toc=/azure/virtual-machines/linux/toc.json#limitations-and-constraints), включая большинство виртуальных машин Azure с двумя или более виртуальных ЦП.

Создайте новый кластер AKS с группой размещения с учетом расположения, связанной с первым пулом системных узлов:

```azurecli-interactive
# Create an Azure resource group
az group create --name myResourceGroup --location centralus
```
Выполните следующую команду и сохраните возвращенный идентификатор:

```azurecli-interactive
# Create proximity placement group
az ppg create -n myPPG -g myResourceGroup -l centralus -t standard
```

Команда создает выходные данные, которые включают значение *идентификатора* , необходимое для предстоящих команд CLI:

```output
{
  "availabilitySets": null,
  "colocationStatus": null,
  "id": "/subscriptions/yourSubscriptionID/resourceGroups/myResourceGroup/providers/Microsoft.Compute/proximityPlacementGroups/myPPG",
  "location": "centralus",
  "name": "myPPG",
  "proximityPlacementGroupType": "Standard",
  "resourceGroup": "myResourceGroup",
  "tags": {},
  "type": "Microsoft.Compute/proximityPlacementGroups",
  "virtualMachineScaleSets": null,
  "virtualMachines": null
}
```

Используйте идентификатор ресурса группы размещения близости для значения *миппгресаурцеид* в следующей команде:

```azurecli-interactive
# Create an AKS cluster that uses a proximity placement group for the initial system node pool only. The PPG has no effect on the cluster control plane.
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --ppg myPPGResourceID
```

## <a name="add-a-proximity-placement-group-to-an-existing-cluster"></a>Добавление группы размещения с учетом расположения в существующий кластер

Группу размещения близости можно добавить в существующий кластер, создав новый пул узлов. После этого можно при необходимости перенести существующие рабочие нагрузки в новый пул узлов, а затем удалить исходный пул узлов.

Используйте ту же группу размещения, которая была создана ранее, и это обеспечит, чтобы узлы агента в обоих пулах узлов в кластере AKS физически размещались в одном центре обработки данных.

Используйте идентификатор ресурса из группы размещения с учетом расположения, созданной ранее, и добавьте новый пул узлов с помощью [`az aks nodepool add`][az-aks-nodepool-add] команды:

```azurecli-interactive
# Add a new node pool that uses a proximity placement group, use a --node-count = 1 for testing
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --node-count 1 \
    --ppg myPPGResourceID
```

## <a name="clean-up"></a>Очистка

Чтобы удалить кластер, [`az group delete`][az-group-delete] удалите группу ресурсов AKS с помощью команды:

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о [группах размещения][proximity-placement-groups]с учетом расположения.

<!-- LINKS - Internal -->
[azure-ad-rbac]: azure-ad-rbac.md
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-get-upgrades]: /cli/azure/aks#az-aks-get-upgrades
[az-aks-upgrade]: /cli/azure/aks#az-aks-upgrade
[az-aks-show]: /cli/azure/aks#az-aks-show
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[proximity-placement-groups]: ../virtual-machines/co-location.md#proximity-placement-groups
[az-aks-create]: /cli/azure/aks#az-aks-create
[system-pool]: ./use-system-pools.md
[az-aks-nodepool-add]: /cli/azure/aks/nodepool#az-aks-nodepool-add
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-group-create]: /cli/azure/group#az-group-create
[az-group-delete]: /cli/azure/group#az-group-delete