---
title: Использование зон доступности в Службе Azure Kubernetes (AKS)
description: Узнайте, как создать кластер, который распределяет узлы между зонами доступности в службе Kubernetes Azure (AKS)
services: container-service
ms.custom: fasttrack-edit, references_regions, devx-track-azurecli
ms.topic: article
ms.date: 09/04/2020
ms.openlocfilehash: 3eec8a6c331227d9d6298c46b272a5784080d342
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102180332"
---
# <a name="create-an-azure-kubernetes-service-aks-cluster-that-uses-availability-zones"></a>Создание кластера Службы Azure Kubernetes (AKS), который использует зоны доступности

Кластер Azure Kubernetes Service (AKS) распространяет ресурсы, такие как узлы и хранилище, по логическим разделам базовой инфраструктуры Azure. Эта модель развертывания при использовании зон доступности гарантирует, что узлы в заданной зоне доступности физически отделены от тех, которые определены в другой зоне доступности. Кластеры AKS, развернутые с несколькими зонами доступности, настроенными в кластере, обеспечивают более высокий уровень доступности для защиты от сбоев оборудования или запланированного обслуживания.

Если определить пулы узлов в кластере для нескольких зон, узлы в данном пуле узлов могут продолжать работать, даже если одна зона не работает. Приложения могут по-прежнему быть доступны даже в случае физического сбоя в одном центре обработки данных, если настроен допуск сбоя нескольких узлов.

В этой статье показано, как создать кластер AKS и распределить компоненты узлов между зонами доступности.

## <a name="before-you-begin"></a>Перед началом

Вам необходимо установить и настроить службу Azure CLI версии 2.0.76 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][install-azure-cli].

## <a name="limitations-and-region-availability"></a>Ограничения и доступность в регионах

В настоящее время кластеры AKS можно создавать с помощью зон доступности в следующих регионах.

* Восточная Австралия
* Центральная Канада
* Центральная часть США
* Восточная часть США 
* восточная часть США 2
* Центральная Франция
* Центрально-Западная Германия
* Восточная Япония
* Северная Европа
* Юго-Восточная Азия
* Центрально-южная часть США
* южная часть Соединенного Королевства
* US Gov (Вирджиния)
* Западная Европа
* западная часть США 2

При создании кластера AKS с помощью зон доступности применяются следующие ограничения.

* Зоны доступности можно определить только при создании кластера или пула узлов.
* Параметры зон доступности невозможно обновить после создания кластера. Вы также не можете обновить существующий кластер зон, не являющихся зонами доступности, для использования зон доступности.
* Выбранный размер узла (SKU виртуальной машины) должен быть доступен во всех выбранных зонах доступности.
* Для кластеров с включенными зонами доступности необходимо использовать Load Balancer (цен. категория "Стандартный") Azure для распределения между зонами. Этот тип подсистемы балансировки нагрузки может быть определен только во время создания кластера. Дополнительные сведения и ограничения стандартной подсистемы балансировки нагрузки см. в статье [Ограничения Load Balancer (цен. категория "Стандартный") Azure][standard-lb-limitations].

### <a name="azure-disks-limitations"></a>Ограничения для дисков Azure

Тома, использующие управляемые диски Azure, в настоящее время не являются ресурсами, избыточными между зонами. Тома не могут быть присоединены между зонами и должны размещаться в той же зоне, что и данный узел, на котором размещен целевой модуль.

Kubernetes учитывает зоны доступности Azure с версии 1,12. Вы можете развернуть объект Персистентволумеклаим, ссылающийся на управляемый диск Azure в кластере AKS с несколькими зонами, и [Kubernetes позаботится о планировании](https://kubernetes.io/docs/setup/best-practices/multiple-zones/#storage-access-for-zones) любого модуля, который заявляет эту PVC в правильной зоне доступности.

## <a name="overview-of-availability-zones-for-aks-clusters"></a>Общие сведения о зонах доступности для кластеров AKS

Зоны доступности являются предложением, обеспечивающим высокий уровень доступности и защищающим приложения и данные от сбоев центров обработки данных. Зоны — уникальные физические расположения в пределах одного региона Azure. Каждая зона состоит из одного или нескольких центров обработки данных, оснащенных независимыми системами электроснабжения, охлаждения и сетевого взаимодействия. Для обеспечения устойчивости всегда существует несколько зон во всех регионах с включенной зоной. Физическое разделение зон доступности в пределах региона защищает приложения и данные от сбоев центров обработки данных.

Дополнительные сведения см. в статье [Что такое зоны доступности в Azure?][az-overview].

Кластеры AKS, развернутые с помощью зон доступности, могут распределять узлы по нескольким зонам в одном регионе. Например, кластер в регионе  *Восточная часть США 2*  может создавать узлы во всех трех зонах доступности в зоне *Восточная часть США 2*. Это распределение ресурсов кластера AKS повышает доступность кластера, так как они устойчивы к сбоям определенной зоны.

![Распределение узлов AKS между зонами доступности](media/availability-zones/aks-availability-zones.png)

Когда одна зона становится недоступной, приложения продолжают работать, если кластер распределен между несколькими зонами.

## <a name="create-an-aks-cluster-across-availability-zones"></a>Создание кластера AKS в разных зонах доступности

При создании кластера с помощью команды [az aks create][az-aks-create] параметр `--zones` определяет, в какие зоны развертываются узлы агента. Компоненты плоскости управления, такие как etcd или API, распределяются по доступным зонам в регионе, если вы определяете `--zones` параметр во время создания кластера. Отдельные зоны, по которым распределяются компоненты уровня управления, не зависят от того, какие явные зоны выбраны для начального пула узлов.

Если вы не определили зоны для пула агентов по умолчанию при создании кластера AKS, компоненты уровня управления не будут распределяться между зонами доступности. Вы можете добавить дополнительные пулы узлов с помощью команды [az aks nodepool add][az-aks-nodepool-add] и указать `--zones` для новых узлов, но это не изменит способ распределения уровня управления между зонами. Параметры зоны доступности можно определить только во время создания кластера или пула узлов.

В следующем примере в группе ресурсов *myResourceGroup* создается кластер AKS с именем *myAKSCluster*. Создается всего *3* узла — один агент в зоне *1*, один в зоне *2* и один — в зоне *3*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus2

az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --generate-ssh-keys \
    --vm-set-type VirtualMachineScaleSets \
    --load-balancer-sku standard \
    --node-count 3 \
    --zones 1 2 3
```

Создание кластера AKS занимает несколько минут.

При принятии решения о зоне, к которой должен принадлежать новый узел, в заданном пуле узлов AKS будет использоваться [оптимальная балансировка по зонам, предлагаемая базовыми масштабируемыми наборами виртуальных машин Azure][vmss-zone-balancing]. Пул узлов AKS считается сбалансированным, если каждая зона имеет одинаковое количество виртуальных машин или +\- 1 VM во всех остальных зонах для масштабируемого набора.

## <a name="verify-node-distribution-across-zones"></a>Проверка распределения узлов между зонами

Когда кластер будет готов, вызовите список узлов агента в масштабируемом наборе, чтобы увидеть, в какой зоне доступности они развернуты.

Сначала получите учетные данные для кластера AKS с помощью команды [az aks get-credentials][az-aks-get-credentials].

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Затем используйте команду [kubectl Опишите][kubectl-describe] , чтобы вывести список узлов в кластере и выполнить фильтрацию по значению *Failure-domain.Beta.kubernetes.IO/Zone* . Ниже приведен пример для оболочки bash.

```console
kubectl describe nodes | grep -e "Name:" -e "failure-domain.beta.kubernetes.io/zone"
```

В следующем примере выходных данных показаны три узла, распределенные по заданному региону и зонам доступности, такие как *eastus2-1* для первой зоны доступности и *eastus2-2* для второй зоны доступности.

```console
Name:       aks-nodepool1-28993262-vmss000000
            failure-domain.beta.kubernetes.io/zone=eastus2-1
Name:       aks-nodepool1-28993262-vmss000001
            failure-domain.beta.kubernetes.io/zone=eastus2-2
Name:       aks-nodepool1-28993262-vmss000002
            failure-domain.beta.kubernetes.io/zone=eastus2-3
```

При добавлении дополнительных узлов в пул агентов платформа Azure автоматически распределяет базовые виртуальные машины в указанных зонах доступности.

Обратите внимание, что в более новых версиях Kubernetes (начиная с 1.17.0) AKS использует более новую метку `topology.kubernetes.io/zone` в дополнение к устаревшей `failure-domain.beta.kubernetes.io/zone`. Результат можно получить, выполнив следующий скрипт:

```console
kubectl get nodes -o custom-columns=NAME:'{.metadata.name}',REGION:'{.metadata.labels.topology\.kubernetes\.io/region}',ZONE:'{metadata.labels.topology\.kubernetes\.io/zone}'
```

Это обеспечит более сжатые выходные данные:

```console
NAME                                REGION   ZONE
aks-nodepool1-34917322-vmss000000   eastus   eastus-1
aks-nodepool1-34917322-vmss000001   eastus   eastus-2
aks-nodepool1-34917322-vmss000002   eastus   eastus-3
```

## <a name="verify-pod-distribution-across-zones"></a>Проверка распределения узлов между зонами

Как описано в статье [Хорошо известные метки, заметки и taint][kubectl-well_known_labels], Kubernetes использует метку `failure-domain.beta.kubernetes.io/zone` для автоматического распределения модулей pod в контроллере или службе репликации в разных доступных зонах. Чтобы протестировать это, можно увеличить масштаб кластера с 3 до 5 узлов, чтобы убедиться в правильности распределения модулей pod:

```azurecli-interactive
az aks scale \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 5
```

Когда операция масштабирования завершается через несколько минут, команда `kubectl describe nodes | grep -e "Name:" -e "failure-domain.beta.kubernetes.io/zone"` в оболочке bash должна вывести результат, аналогичный приведенному в этом примере:

```console
Name:       aks-nodepool1-28993262-vmss000000
            failure-domain.beta.kubernetes.io/zone=eastus2-1
Name:       aks-nodepool1-28993262-vmss000001
            failure-domain.beta.kubernetes.io/zone=eastus2-2
Name:       aks-nodepool1-28993262-vmss000002
            failure-domain.beta.kubernetes.io/zone=eastus2-3
Name:       aks-nodepool1-28993262-vmss000003
            failure-domain.beta.kubernetes.io/zone=eastus2-1
Name:       aks-nodepool1-28993262-vmss000004
            failure-domain.beta.kubernetes.io/zone=eastus2-2
```

Теперь у нас есть два дополнительных узла в зонах 1 и 2. Можно развернуть приложение, состоящее из трех реплик. В качестве примера мы будем использовать NGINX:

```console
kubectl create deployment nginx --image=nginx
kubectl scale deployment nginx --replicas=3
```

При просмотре узлов, на которых выполняются модули pod, вы увидите модули pod на узлах, соответствующих трем различным зонам доступности. Например, с помощью команды `kubectl describe pod | grep -e "^Name:" -e "^Node:"` в интерпретаторе bash вы получите примерно такой результат:

```console
Name:         nginx-6db489d4b7-ktdwg
Node:         aks-nodepool1-28993262-vmss000000/10.240.0.4
Name:         nginx-6db489d4b7-v7zvj
Node:         aks-nodepool1-28993262-vmss000002/10.240.0.6
Name:         nginx-6db489d4b7-xz6wj
Node:         aks-nodepool1-28993262-vmss000004/10.240.0.8
```

Как видно из предыдущих выходных данных, первый модуль pod выполняется на узле 0, который находится в зоне доступности `eastus2-1`. Второй модуль pod выполняется на узле 2, который соответствует `eastus2-3`, а третий — на узле 4 в `eastus2-2`. Без дополнительной настройки Kubernetes правильно распределяет модули pod по всем трем зонам доступности.

## <a name="next-steps"></a>Дальнейшие действия

В этой статье подробно описано, как создать кластер AKS, использующий зоны доступности. Дополнительные рекомендации о высокодоступных кластерах см. в руководстве по [обеспечению непрерывности бизнеса и аварийного восстановления в AKS][best-practices-bc-dr].

<!-- LINKS - internal -->
[install-azure-cli]: /cli/azure/install-azure-cli
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-overview]: ../availability-zones/az-overview.md
[best-practices-bc-dr]: operator-best-practices-multi-region.md
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[standard-lb-limitations]: load-balancer-standard.md#limitations
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-aks-nodepool-add]: /cli/azure/ext/aks-preview/aks/nodepool#ext-aks-preview-az-aks-nodepool-add
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[vmss-zone-balancing]: ../virtual-machine-scale-sets/virtual-machine-scale-sets-use-availability-zones.md#zone-balancing

<!-- LINKS - external -->
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-well_known_labels]: https://kubernetes.io/docs/reference/labels-annotations-taints/
