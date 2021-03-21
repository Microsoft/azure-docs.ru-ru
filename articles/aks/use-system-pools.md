---
title: Использование пулов системных узлов в службе Kubernetes Azure (AKS)
description: Узнайте, как создавать пулы системных узлов и управлять ими в службе Kubernetes Azure (AKS).
services: container-service
ms.topic: article
ms.date: 06/18/2020
ms.author: mlearned
ms.custom: fasttrack-edit, devx-track-azurecli
ms.openlocfilehash: 9c53cb53517c4696a1bb47c2cb72335979d58d3a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102178836"
---
# <a name="manage-system-node-pools-in-azure-kubernetes-service-aks"></a>Управление пулами системных узлов в службе Kubernetes Azure (AKS)

В службе Azure Kubernetes Service (AKS) узлы одной и той же конфигурации группируются в *Пулы узлов*. Пулы узлов содержат базовые виртуальные машины, на которых работают ваши приложения. Пулы системных узлов и пулы узлов пользователей — это два разных режима пула узлов для кластеров AKS. Пулы системных узлов служат основной целью размещения критически важных системных модулей, таких как `CoreDNS` и `metrics-server` . Пулы узлов пользователей служат основной целью размещения модулей Pod приложения. Однако модули приложений можно запланировать в пулах системных узлов, если в кластере AKS требуется только один пул. Каждый кластер AKS должен содержать по крайней мере один пул системных узлов, содержащий по крайней мере один узел.

> [!Important]
> При запуске одного пула системных узлов для кластера AKS в рабочей среде мы рекомендуем использовать по крайней мере три узла для пула узлов.

## <a name="before-you-begin"></a>Перед началом

* Необходимо установить или настроить Azure CLI версии 2.3.1 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][install-azure-cli].

## <a name="limitations"></a>Ограничения

При создании кластеров AKS, поддерживающих пулы системных узлов, и управлении ими действуют следующие ограничения.

* См. раздел [квоты, ограничения размера виртуальной машины и доступность регионов в службе Azure Kubernetes (AKS)][quotas-skus-regions].
* Кластер AKS должен быть построен с масштабируемыми наборами виртуальных машин в качестве типа виртуальной машины и балансировщика нагрузки *стандартного* SKU.
* Имя пула узлов может содержать только буквы в нижнем регистре и должно начинаться с буквы в нижнем регистре. Для пулов узлов Linux длина должна составлять от 1 до 12 символов. Для пулов узлов Windows длина должна составлять от 1 до 6 символов.
* Для установки режима пула узлов необходимо использовать API версии 2020-03-01 или более поздней. Кластеры, созданные в версиях API старше 2020-03-01, содержат только пулы пользовательских узлов, но их можно перенести для включения пулов системных узлов, следуя [шагам обновления режима пула](#update-existing-cluster-system-and-user-node-pools).
* Режим пула узлов является обязательным свойством и должен быть задан явно при использовании шаблонов ARM или прямых вызовов API.

## <a name="system-and-user-node-pools"></a>Пулы системных и пользовательских узлов

Для пула системных узлов AKS автоматически присваивает метке **kubernetes.Azure.com/mode: System** узлам. Это приводит к тому, что AKS предпочитает планирование системных модулей в пулах узлов, содержащих эту метку. Эта метка не препятствует планированию модулей Pod приложения на пулах системных узлов. Однако мы рекомендуем изолировать критически важные системные модули из модулей Pod приложения, чтобы предотвратить случайное уничтожение системных модулей в ненастроенных или мошеннических приложениях. Это поведение можно реализовать, создав выделенный пул системных узлов. Используйте `CriticalAddonsOnly=true:NoSchedule` таинт, чтобы запретить планирование модулей приложений для пулов системных узлов.

Пулы системных узлов имеют следующие ограничения.

* Пулы систем osType должны быть Linux.
* Пулы узлов пользователей osType могут быть Linux или Windows.
* Системные пулы должны содержать по крайней мере один узел, а пулы узлов пользователей могут содержать ноль или более узлов.
* Для пулов системных узлов требуется номер SKU виртуальной машины не менее 2 виртуальных ЦП и 4 ГБ памяти.
* Пулы системных узлов должны поддерживать по крайней мере 30 модулей Pod, как описано в [формуле минимального и максимального значений для модулей][maximum-pods]Pod.
* Для пулов смесевых узлов требуются пулы узлов пользователей.
* Добавление дополнительного пула узлов системы или изменение пула узлов, который является пулом системных узлов, *не* приводит к автоматическому перемещению системных модулей. Системные модули могут продолжать работать в том же пуле узлов, даже если вы переменяете его на пул узлов пользователей. При удалении или уменьшении уровня пула узлов, в котором запущены системные модули, ранее созданные пулом системных узлов, эти системные модули повторно развертываются с предпочтительным расписанием для нового пула узлов системы.

Вы можете выполнять следующие операции с пулами узлов:

* Создание выделенного пула системных узлов (предпочтительное планирование системных модулей для пулов узлов `mode:system` )
* Измените пул узлов системы так, чтобы он был пулом узлов пользователей при наличии другого пула узлов системы, который будет использоваться в кластере AKS.
* Измените пул узлов пользователей, чтобы он был пулом системных узлов.
* Удаление пулов узлов пользователей.
* Пулы системных узлов можно удалить при условии, что в кластере AKS имеется другой пул узлов системы.
* Кластер AKS может иметь несколько пулов системных узлов и требует по крайней мере одного пула системных узлов.
* Если вы хотите изменить различные неизменяемые параметры для существующих пулов узлов, можно создать новые пулы узлов, чтобы заменить их. Одним из примеров является добавление нового пула узлов с новым параметром Максподс и удаление старого пула узлов.

## <a name="create-a-new-aks-cluster-with-a-system-node-pool"></a>Создание нового кластера AKS с пулом системных узлов

При создании нового кластера AKS автоматически создается пул системных узлов с одним узлом. Исходный пул узлов по умолчанию имеет режим System. При создании пулов узлов с помощью `az aks nodepool add` эти пулы узлов являются пулами пользовательских узлов, если только вы не укажете параметр mode явным образом.

В следующем примере создается группа ресурсов с именем *myResourceGroup* в регионе *eastus*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Используйте команду [az aks create][az-aks-create], чтобы создать кластер AKS. В следующем примере создается кластер с именем *myAKSCluster* с одним выделенным системным пулом, содержащим один узел. Для рабочих нагрузок убедитесь, что вы используете пулы системных узлов по крайней мере на трех узлах. Выполнение этой операции займет несколько минут.

```azurecli-interactive
# Create a new AKS cluster with a single system pool
az aks create -g myResourceGroup --name myAKSCluster --node-count 1 --generate-ssh-keys
```

## <a name="add-a-dedicated-system-node-pool-to-an-existing-aks-cluster"></a>Добавление выделенного пула системных узлов в существующий кластер AKS

> [!Important]
> После создания пула узлов изменить таинтс узла через интерфейс командной строки невозможно.

В существующие кластеры AKS можно добавить один или несколько пулов системных узлов. Рекомендуется запланировать модули для приложений на пулах узлов пользователей и выделить пулы системных узлов только для критических системных модулей. Это предотвращает случайное удаление системных модулей из незаконных приложений. Применяйте это поведение с `CriticalAddonsOnly=true:NoSchedule` [таинт][aks-taints] для пулов системных узлов. 

Следующая команда добавляет выделенный пул узлов системы типов Mode со счетчиком по умолчанию для трех узлов.

```azurecli-interactive
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name systempool \
    --node-count 3 \
    --node-taints CriticalAddonsOnly=true:NoSchedule \
    --mode System
```
## <a name="show-details-for-your-node-pool"></a>Отображение сведений о пуле узлов

Сведения о пуле узлов можно проверить с помощью следующей команды.  

```azurecli-interactive
az aks nodepool show -g myResourceGroup --cluster-name myAKSCluster -n systempool
```

Для пулов узлов системы определен режим **системы** типов, а для пулов узлов пользователей определен режим типа **User** . Для системного пула убедитесь, что для параметра таинт задано значение `CriticalAddonsOnly=true:NoSchedule` , что позволит предотвратить планирование модулей приложений из существ, запланированных в этом пуле узлов.

```output
{
  "agentPoolType": "VirtualMachineScaleSets",
  "availabilityZones": null,
  "count": 1,
  "enableAutoScaling": null,
  "enableNodePublicIp": false,
  "id": "/subscriptions/yourSubscriptionId/resourcegroups/myResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster/agentPools/systempool",
  "maxCount": null,
  "maxPods": 110,
  "minCount": null,
  "mode": "System",
  "name": "systempool",
  "nodeImageVersion": "AKSUbuntu-1604-2020.06.30",
  "nodeLabels": {},
  "nodeTaints": [
    "CriticalAddonsOnly=true:NoSchedule"
  ],
  "orchestratorVersion": "1.16.10",
  "osDiskSizeGb": 128,
  "osType": "Linux",
  "provisioningState": "Failed",
  "proximityPlacementGroupId": null,
  "resourceGroup": "myResourceGroup",
  "scaleSetEvictionPolicy": null,
  "scaleSetPriority": null,
  "spotMaxPrice": null,
  "tags": null,
  "type": "Microsoft.ContainerService/managedClusters/agentPools",
  "upgradeSettings": {
    "maxSurge": null
  },
  "vmSize": "Standard_DS2_v2",
  "vnetSubnetId": null
}
```

## <a name="update-existing-cluster-system-and-user-node-pools"></a>Обновление существующей системы кластера и пользовательских пулов узлов

> [!NOTE]
> Для установки режима пула системных узлов необходимо использовать API версии 2020-03-01 или более поздней. В результате кластеры, созданные в версиях API старше 2020-03-01, содержат только пулы пользовательских узлов. Для получения функциональных возможностей и преимуществ пула системных узлов в старых кластерах обновите режим существующих пулов узлов, используя следующие команды в последней версии Azure CLI.

Можно изменить режимы для пулов узлов и пользователей. Пул системных узлов можно изменить на пул пользователей, только если в кластере AKS уже существует другой пул узлов системы.

Эта команда изменяет пул системных узлов на пул узлов пользователей.

```azurecli-interactive
az aks nodepool update -g myResourceGroup --cluster-name myAKSCluster -n mynodepool --mode user
```

Эта команда изменяет пул узлов пользователей на пул системных узлов.

```azurecli-interactive
az aks nodepool update -g myResourceGroup --cluster-name myAKSCluster -n mynodepool --mode system
```

## <a name="delete-a-system-node-pool"></a>Удаление пула системных узлов

> [!Note]
> Чтобы использовать пулы системных узлов в кластерах AKS до версии API 2020-03-02, добавьте новый пул узлов системы, а затем удалите исходный пул узлов по умолчанию.

Необходимо иметь по крайней мере два пула системных узлов в кластере AKS, прежде чем можно будет удалить один из них.

```azurecli-interactive
az aks nodepool delete -g myResourceGroup --cluster-name myAKSCluster -n mynodepool
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить кластер, используйте команду [AZ Group Delete][az-group-delete] , чтобы удалить группу ресурсов AKS:

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```



## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как создавать пулы узлов системы в кластере AKS и управлять ими. Дополнительные сведения об использовании нескольких пулов узлов см. в статье [Использование пулов с несколькими][use-multiple-node-pools]узлами.

<!-- EXTERNAL LINKS -->
[kubernetes-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-taint]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#taint
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubernetes-labels]: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/
[kubernetes-label-syntax]: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#syntax-and-character-set

<!-- INTERNAL LINKS -->
[aks-taints]: use-multiple-node-pools.md#setting-nodepool-taints
[aks-windows]: windows-container-cli.md
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-nodepool-add]: /cli/azure/aks/nodepool#az-aks-nodepool-add
[az-aks-nodepool-list]: /cli/azure/aks/nodepool#az-aks-nodepool-list
[az-aks-nodepool-update]: /cli/azure/aks/nodepool#az-aks-nodepool-update
[az-aks-nodepool-upgrade]: /cli/azure/aks/nodepool#az-aks-nodepool-upgrade
[az-aks-nodepool-scale]: /cli/azure/aks/nodepool#az-aks-nodepool-scale
[az-aks-nodepool-delete]: /cli/azure/aks/nodepool#az-aks-nodepool-delete
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-group-create]: /cli/azure/group#az-group-create
[az-group-delete]: /cli/azure/group#az-group-delete
[az-group-deployment-create]: /cli/azure/group/deployment#az-group-deployment-create
[gpu-cluster]: gpu-cluster.md
[install-azure-cli]: /cli/azure/install-azure-cli
[operator-best-practices-advanced-scheduler]: operator-best-practices-advanced-scheduler.md
[quotas-skus-regions]: quotas-skus-regions.md
[supported-versions]: supported-kubernetes-versions.md
[tag-limitation]: ../azure-resource-manager/management/tag-resources.md
[taints-tolerations]: operator-best-practices-advanced-scheduler.md#provide-dedicated-nodes-using-taints-and-tolerations
[vm-sizes]: ../virtual-machines/sizes.md
[use-multiple-node-pools]: use-multiple-node-pools.md
[maximum-pods]: configure-azure-cni.md#maximum-pods-per-node
