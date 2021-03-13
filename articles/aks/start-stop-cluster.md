---
title: Запуск и завершение службы Kubernetes Azure (AKS)
description: Узнайте, как запустить кластер Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 09/24/2020
author: palma21
ms.openlocfilehash: 87d51f9c1d084faf79c7ec1cf1255a6fb3c8245d
ms.sourcegitcommit: 5f32f03eeb892bf0d023b23bd709e642d1812696
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/12/2021
ms.locfileid: "103201003"
---
# <a name="stop-and-start-an-azure-kubernetes-service-aks-cluster"></a>Останавливает и запускает кластер Azure Kubernetes Service (AKS)

Рабочие нагрузки AKS могут не выполняться непрерывно, например, в кластере разработки, который используется только в рабочее время. Это ведет к тому, что кластер Azure Kubernetes Service (AKS) может простаивать, не выполняя никаких дополнительных системных компонентов. Можно уменьшить объем ресурсов кластера путем [масштабирования всех `User` пулов узлов до 0](scale-cluster.md#scale-user-node-pools-to-0), но [ `System` пул](use-system-pools.md) по-прежнему требуется для запуска системных компонентов во время работы кластера. Чтобы оптимизировать затраты в течение этих периодов, можно полностью отключить (завершить) кластер. Это действие полностью останавливает свою плоскость управления и узлы агентов, позволяя экономить все затраты на вычисление, сохраняя при этом все объекты и состояние кластера, сохраненные для запуска. Затем вы можете выбрать, где вы оставили после выходных или чтобы ваш кластер выполнялся только во время выполнения пакетных заданий.

## <a name="before-you-begin"></a>Перед началом

В этой статье предполагается, что у вас есть кластер AKS. Если вам нужен кластер AKS, обратитесь к краткому руководству по работе с AKS [с помощью Azure CLI][aks-quickstart-cli] или [портала Azure][aks-quickstart-portal].

### <a name="limitations"></a>Ограничения

При использовании функции запуска и окончания кластера действуют следующие ограничения.

- Эта функция поддерживается только для кластеров с поддержкой масштабируемых наборов виртуальных машин.
- Состояние кластера остановленного кластера AKS сохраняется в течение 12 месяцев. Если кластер остановлен более 12 месяцев, состояние кластера восстановить нельзя. Дополнительные сведения см. в разделе [политики поддержки AKS](support-policies.md).
- Вы можете запустить или удалить остановленный кластер AKS. Чтобы выполнить любые операции, такие как масштабирование или обновление, сначала запустите кластер.

## <a name="stop-an-aks-cluster"></a>Завершение кластера AKS

Вы можете использовать `az aks stop` команду, чтобы прерывать работающие узлы кластера AKS и плоскости управления. В следующем примере останавливается кластер с именем *myAKSCluster*:

```azurecli-interactive
az aks stop --name myAKSCluster --resource-group myResourceGroup
```

Вы можете проверить, не остановлен ли кластер, выполнив команду [AZ AKS][az-aks-show] -OUTPUT и убедившись, `powerState` что показанные `Stopped` ниже выходные данные.

```json
{
[...]
  "nodeResourceGroup": "MC_myResourceGroup_myAKSCluster_westus2",
  "powerState":{
    "code":"Stopped"
  },
  "privateFqdn": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
[...]
}
```

Если `provisioningState` показывает `Stopping` , что кластер еще не полностью остановлен.

> [!IMPORTANT]
> Если вы используете [бюджеты прерываний Pod](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) , операция остановки может занять больше времени, так как процесс стока займет большее время.

## <a name="start-an-aks-cluster"></a>Запуск кластера AKS

Вы можете использовать `az aks start` команду, чтобы запустить остановленные узлы кластера AKS и плоскость управления. Кластер перезапускается с предыдущим состоянием плоскости управления и количеством узлов агентов.  
В следующем примере запускается кластер с именем *myAKSCluster*:

```azurecli-interactive
az aks start --name myAKSCluster --resource-group myResourceGroup
```

Вы можете проверить, запущен ли кластер, выполнив команду [AZ AKS Показать][az-aks-show] и убедившись, `powerState` что показанные `Running` ниже выходные данные.

```json
{
[...]
  "nodeResourceGroup": "MC_myResourceGroup_myAKSCluster_westus2",
  "powerState":{
    "code":"Running"
  },
  "privateFqdn": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
[...]
}
```

Если `provisioningState` показывает `Starting` , что кластер еще не был полностью запущен.

## <a name="next-steps"></a>Дальнейшие действия

- Чтобы узнать, как масштабировать `User` Пулы до 0, см. раздел [масштабирование `User` пулов до 0](scale-cluster.md#scale-user-node-pools-to-0).
- Сведения о том, как сократить расходы с помощью экземпляров смесевых, см. в разделе [Добавление пула узлов смесевых цветов в AKS](spot-node-pool.md).
- Дополнительные сведения о политиках поддержки AKS см. в разделе [политики поддержки AKS](support-policies.md).

<!-- LINKS - external -->

<!-- LINKS - internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[az-aks-show]: /cli/azure/aks#az_aks_show
