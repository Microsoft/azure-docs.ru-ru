---
title: Динамическое создание тома дисков Azure
titleSuffix: Azure Kubernetes Service
description: Узнайте, как динамически создать постоянный том с помощью дисков Azure в службе Kubernetes Azure (AKS).
services: container-service
ms.topic: article
ms.date: 09/21/2020
ms.openlocfilehash: ad51bfdf8c494e763921de880926b839cdb7be62
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96021645"
---
# <a name="dynamically-create-and-use-a-persistent-volume-with-azure-disks-in-azure-kubernetes-service-aks"></a>Динамическое создание и использование постоянного тома с дисками Azure в службе Azure Kubernetes (AKS)

Постоянный том — это часть хранилища, которая подготовлена к использованию для модулей pod Kubernetes. Постоянный том может использоваться одним или несколькими модулями и может быть подготовлен динамически или статически. В этой статье описывается динамическое создание постоянных томов с дисками Azure, используемых одним модулем pod в кластере службы Azure Kubernetes (AKS).

> [!NOTE]
> Диск Azure может быть подключен только с типом *доступа* *реадвритеонце*, что делает его доступным для одного узла в AKS. Если необходимо совместно использовать постоянный том на нескольких узлах, используйте службу [файлов Azure][azure-files-pvc].

Дополнительные сведения о томах Kubernetes см. в статье, [посвященной возможностям хранения данных приложений в AKS][concepts-storage].

## <a name="before-you-begin"></a>Перед началом

В этой статье предполагается, что у вас есть кластер AKS. Если вам нужен кластер AKS, обратитесь к краткому руководству по работе с AKS [с помощью Azure CLI][aks-quickstart-cli] или [портала Azure][aks-quickstart-portal].

Кроме того, нужно установить и настроить Azure CLI версии 2.0.59 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][install-azure-cli].

## <a name="built-in-storage-classes"></a>Встроенные классы хранения

Класс хранения используется для определения того, как единица хранения создается динамически с помощью постоянного тома. Дополнительные сведения о классах хранения Kubernetes см. в разделе [Kubernetes Storage Classes][kubernetes-storage-classes] (Классы хранения Kubernetes).

Каждый кластер AKS включает четыре предварительно созданных класса хранения, два из которых настроены для работы с дисками Azure:

* Класс хранения *по умолчанию* подготавливает диск Azure стандартного SSD.
    * Хранилище уровня "Стандартный" поддерживается стандартными твердотельными накопителями и обеспечивает экономичное хранилище, сохраняя при этом надежную производительность. 
* Класс хранения *managed-premium* подготавливает диск Azure ценовой категории "Премиум".
    * Диски уровня "Премиум" используют высокопроизводительные твердотельные накопители с низкой задержкой. Они идеально подходят для виртуальных машин, выполняющих производственную рабочую нагрузку. Если узлы AKS в кластере используют хранилище класса Premium, выберите класс *managed-premium*.
    
Если вы используете один из классов хранения по умолчанию, вы не сможете обновить размер тома после создания класса хранения. Чтобы иметь возможность обновить размер тома после создания класса хранения, добавьте строку `allowVolumeExpansion: true` в один из классов хранения по умолчанию или создайте собственный класс хранилища. Обратите внимание, что не поддерживается уменьшение размера PVC (во избежание потери данных). Существующий класс хранения можно изменить с помощью `kubectl edit sc` команды. 

Например, если вы хотите использовать диск размером 4 Тиб, необходимо создать класс хранения, который определяет, `cachingmode: None` так как [кэширование дисков не поддерживается для дисков 4 Тиб и больше](../virtual-machines/premium-storage-performance.md#disk-caching).

Дополнительные сведения о классах хранения и создании собственного класса хранения см. [в статье Параметры хранилища для приложений в AKS][storage-class-concepts].

Используйте команду [kubectl get sc][kubectl-get], чтобы просмотреть предварительно созданные классы хранения. В следующем примере демонстрируются предварительно созданные классы хранения, доступные для кластера AKS.

```console
$ kubectl get sc

NAME                PROVISIONER                AGE
default (default)   kubernetes.io/azure-disk   1h
managed-premium     kubernetes.io/azure-disk   1h
```

> [!NOTE]
> Утверждения постоянных томов указаны в ГиБ, но счета за использование управляемых дисков Azure выставляются по номеру SKU для определенного размера. Эти номера SKU находятся в диапазоне от 32GiB для дисков S4 или P4 до 32TiB для дисков S80 или P80 (в предварительной версии). Число операций ввода-вывода в секунду и пропускная способность управляемого диска уровня "Премиум" зависят как от номера SKU, так и от размера экземпляров узлов в кластере AKS. Дополнительные сведения см. на странице [Цены на управляемые диски][managed-disk-pricing-performance].

## <a name="create-a-persistent-volume-claim"></a>Создание заявки на доступ к постоянному тому

Утверждение постоянного тома (PVC) используется для автоматической подготовки хранилища на основе класса хранения. В этом случае утверждение постоянного тома может использовать один из предварительно созданных классов хранения для создания управляемого диска Azure уровня "Стандартный" или "Премиум".

Создайте файл `azure-premium.yaml` и скопируйте в него следующий манифест. Утверждение запрашивает диск с именем `azure-managed-disk` размером *5 ГБ* с доступом *ReadWriteOnce*. Класс хранилища *managed-premium* указан как класс хранения.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azure-managed-disk
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: managed-premium
  resources:
    requests:
      storage: 5Gi
```

> [!TIP]
> Чтобы создать диск, который использует хранилище уровня "Стандартный", используйте класс `storageClassName: default` вместо *managed-premium*.

Создайте заявку на доступ к постоянному тому с помощью команды [kubectl apply][kubectl-apply] и укажите файл *azure-premium.yaml*.

```console
$ kubectl apply -f azure-premium.yaml

persistentvolumeclaim/azure-managed-disk created
```

## <a name="use-the-persistent-volume"></a>Использование постоянного тома

После создания утверждения постоянного тома и успешной подготовки диска можно создать группу pod с доступом к диску. Приведенный ниже манифест создает базовый модуль pod NGINX, который использует утверждение постоянного тома *azure-managed-disk* для подключения диска Azure по пути `/mnt/azure`. Для контейнеров Windows Server укажите *mountPath* в формате пути Windows, например *"D:"* .

Создайте файл `azure-pvc-disk.yaml` и скопируйте в него следующий манифест.

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
    volumeMounts:
    - mountPath: "/mnt/azure"
      name: volume
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: azure-managed-disk
```

Создайте модуль pod с помощью команды [kubectl apply][kubectl-apply], как показано ниже.

```console
$ kubectl apply -f azure-pvc-disk.yaml

pod/mypod created
```

Теперь у вас есть работающий pod с диском Azure, подключенным к каталогу `/mnt/azure`. Эту конфигурацию можно видеть при проверке модуля с помощью `kubectl describe pod mypod`, как показано в сокращенном примере ниже.

```console
$ kubectl describe pod mypod

[...]
Volumes:
  volume:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  azure-managed-disk
    ReadOnly:   false
  default-token-smm2n:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-smm2n
    Optional:    false
[...]
Events:
  Type    Reason                 Age   From                               Message
  ----    ------                 ----  ----                               -------
  Normal  Scheduled              2m    default-scheduler                  Successfully assigned mypod to aks-nodepool1-79590246-0
  Normal  SuccessfulMountVolume  2m    kubelet, aks-nodepool1-79590246-0  MountVolume.SetUp succeeded for volume "default-token-smm2n"
  Normal  SuccessfulMountVolume  1m    kubelet, aks-nodepool1-79590246-0  MountVolume.SetUp succeeded for volume "pvc-faf0f176-8b8d-11e8-923b-deb28c58d242"
[...]
```

## <a name="use-ultra-disks"></a>Использование Ultra Disks
Чтобы использовать Ultra Disk, см. раздел [Использование Ultra Disks в службе Kubernetes Azure (AKS)](use-ultra-disks.md).

## <a name="back-up-a-persistent-volume"></a>Резервное копирование постоянного тома

Для резервного копирования данных на постоянном томе создайте моментальный снимок управляемого диска для этого тома. Затем этот моментальный снимок можно использовать для создания восстановленного диска и присоединить к модулям pod в качестве средства восстановления этих данных.

В первую очередь получите имя тома с помощью команды `kubectl get pvc`, например для PVC с именем *azure-managed-disk*:

```console
$ kubectl get pvc azure-managed-disk

NAME                 STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
azure-managed-disk   Bound     pvc-faf0f176-8b8d-11e8-923b-deb28c58d242   5Gi        RWO            managed-premium   3m
```

Это имя тома формирует имя соответствующего диска Azure. Отправьте запрос идентификатора диска с помощью команды [az disk list][az-disk-list] и укажите имя тома PVC, как показано в примере ниже.

```azurecli-interactive
$ az disk list --query '[].id | [?contains(@,`pvc-faf0f176-8b8d-11e8-923b-deb28c58d242`)]' -o tsv

/subscriptions/<guid>/resourceGroups/MC_MYRESOURCEGROUP_MYAKSCLUSTER_EASTUS/providers/MicrosoftCompute/disks/kubernetes-dynamic-pvc-faf0f176-8b8d-11e8-923b-deb28c58d242
```

Используйте идентификатор диска, чтобы создать моментальный снимок диска с помощью команды [az snapshot create][az-snapshot-create]. В следующем примере создается моментальный снимок с именем *pvcSnapshot* в той же группе ресурсов, что и кластер AKS (*MC_myResourceGroup_myAKSCluster_eastus*). При создании моментальных снимков и восстановлении дисков в группах ресурсов могут возникнуть проблемы, когда кластер AKS не может получить к доступ к этим дискам.

```azurecli-interactive
$ az snapshot create \
    --resource-group MC_myResourceGroup_myAKSCluster_eastus \
    --name pvcSnapshot \
    --source /subscriptions/<guid>/resourceGroups/MC_myResourceGroup_myAKSCluster_eastus/providers/MicrosoftCompute/disks/kubernetes-dynamic-pvc-faf0f176-8b8d-11e8-923b-deb28c58d242
```

В зависимости от объема данных на диске создание моментального снимка может занять несколько минут.

## <a name="restore-and-use-a-snapshot"></a>Восстановление из моментального снимка и его использование

Чтобы восстановить диск и использовать его в модуле pod Kubernetes, используйте моментальный снимок как источник при создании диска с помощью команды [az disk create][az-disk-create]. Эта операция сохраняет исходный ресурс, если необходимо получить доступ к исходному моментальному снимку данных. В следующем примере создается диск с именем *pvcRestored* из моментального снимка с именем *pvcSnapshot*.

```azurecli-interactive
az disk create --resource-group MC_myResourceGroup_myAKSCluster_eastus --name pvcRestored --source pvcSnapshot
```

Чтобы использовать восстановленный диск с модулем pod, укажите идентификатор этого диска в манифесте. Получите идентификатор диска с помощью команды [az disk show][az-disk-show]. В следующем примере возвращается идентификатор диска *pvcRestored*, созданного на предыдущем шаге.

```azurecli-interactive
az disk show --resource-group MC_myResourceGroup_myAKSCluster_eastus --name pvcRestored --query id -o tsv
```

Создайте манифест модуля pod с именем `azure-restored.yaml` и укажите URI-код диска, полученного на предыдущем шаге. В следующем примере создается базовый веб-сервер NGINX с восстановленным диском, подключенным в качестве тома в точке */mnt/azure*.

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: mypodrestored
spec:
  containers:
  - name: mypodrestored
    image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
    volumeMounts:
    - mountPath: "/mnt/azure"
      name: volume
  volumes:
    - name: volume
      azureDisk:
        kind: Managed
        diskName: pvcRestored
        diskURI: /subscriptions/<guid>/resourceGroups/MC_myResourceGroupAKS_myAKSCluster_eastus/providers/Microsoft.Compute/disks/pvcRestored
```

Создайте модуль pod с помощью команды [kubectl apply][kubectl-apply], как показано ниже.

```console
$ kubectl apply -f azure-restored.yaml

pod/mypodrestored created
```

Можно использовать `kubectl describe pod mypodrestored` для просмотра сведений о модуле pod, как в следующем кратком примере отображения сведений о томе.

```console
$ kubectl describe pod mypodrestored

[...]
Volumes:
  volume:
    Type:         AzureDisk (an Azure Data Disk mount on the host and bind mount to the pod)
    DiskName:     pvcRestored
    DiskURI:      /subscriptions/19da35d3-9a1a-4f3b-9b9c-3c56ef409565/resourceGroups/MC_myResourceGroupAKS_myAKSCluster_eastus/providers/Microsoft.Compute/disks/pvcRestored
    Kind:         Managed
    FSType:       ext4
    CachingMode:  ReadWrite
    ReadOnly:     false
[...]
```

## <a name="next-steps"></a>Дальнейшие действия

Соответствующие рекомендации см. в разделе [Рекомендации по хранению и резервному копированию в AKS][operator-best-practices-storage].

Узнайте больше о постоянных томах Kubernetes, использующих диски Azure.

> [!div class="nextstepaction"]
> [Подключаемый модуль Kubernetes для дисков Azure][azure-disk-volume]

<!-- LINKS - external -->
[access-modes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubernetes-storage-classes]: https://kubernetes.io/docs/concepts/storage/storage-classes/
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/
[managed-disk-pricing-performance]: https://azure.microsoft.com/pricing/details/managed-disks/

<!-- LINKS - internal -->
[azure-disk-volume]: azure-disk-volume.md
[azure-files-pvc]: azure-files-dynamic-pv.md
[premium-storage]: ../virtual-machines/disks-types.md
[az-disk-list]: /cli/azure/disk#az-disk-list
[az-snapshot-create]: /cli/azure/snapshot#az-snapshot-create
[az-disk-create]: /cli/azure/disk#az-disk-create
[az-disk-show]: /cli/azure/disk#az-disk-show
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[operator-best-practices-storage]: operator-best-practices-storage.md
[concepts-storage]: concepts-storage.md
[storage-class-concepts]: concepts-storage.md#storage-classes
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
