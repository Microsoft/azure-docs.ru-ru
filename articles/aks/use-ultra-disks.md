---
title: Включение поддержки Ultra Disk в службе Kubernetes Azure (AKS)
description: Узнайте, как включить и настроить Ultra Disks в кластере Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 07/10/2020
ms.openlocfilehash: d66b806adb7285e0ce2a21d8fe9254b3dbe89bcb
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102178853"
---
# <a name="use-azure-ultra-disks-on-azure-kubernetes-service-preview"></a>Использование Azure Ultra Disks в службе Kubernetes Azure (Предварительная версия)

[Диски Azure "Ultra](../virtual-machines/disks-enable-ultra-ssd.md) " обеспечивают высокую пропускную способность, высокую скорость операций ввода-вывода и постоянную задержку на диске для приложений с отслеживанием состояния. Одним из основных преимуществ использования Ultra Disks является возможность динамического изменения производительности SSD вместе с рабочими нагрузками без необходимости перезапуска узлов агента. Ultra Disks подходит для рабочих нагрузок, интенсивно использующих данные.

## <a name="before-you-begin"></a>Перед началом

Этот компонент можно задать только при создании кластера или во время создания пула узлов.

> [!IMPORTANT]
> Для дисков Azure Ultra требуются нодепулс, развернутые в зонах доступности и регионах, которые поддерживают эти диски, а также только определенные серии виртуальных машин. См. [**область и ограничения для Ultra дисков уровня "Глобальный**](../virtual-machines/disks-enable-ultra-ssd.md#ga-scope-and-limitations)".

### <a name="register-the-enableultrassd-preview-feature"></a>Регистрация `EnableUltraSSD` функции предварительной версии

Чтобы создать кластер AKS или пул узлов, которые могут использовать диски Ultra, необходимо включить `EnableUltraSSD` флаг компонента в подписке.

Зарегистрируйте `EnableUltraSSD` флаг функции с помощью команды [AZ Feature Register][az-feature-register] , как показано в следующем примере:

```azurecli-interactive
az feature register --namespace "Microsoft.ContainerService" --name "EnableUltraSSD"
```

Через несколько минут отобразится состояние *Registered* (Зарегистрировано). Состояние регистрации можно проверить с помощью команды [az feature list][az-feature-list].

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/EnableUltraSSD')].{Name:name,State:properties.state}"
```

Когда все будет готово, обновите регистрацию поставщика ресурсов *Microsoft. ContainerService* с помощью команды [AZ Provider Register][az-provider-register] :

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

### <a name="install-aks-preview-cli-extension"></a>Установка расширения интерфейса командной строки предварительной версии AKS

Для создания кластера AKS или пула узлов, который может использовать Ultra Disks, требуется Последнее расширение CLI *AKS-Preview* . Установите расширение Azure CLI *AKS-Preview* с помощью команды [AZ Extension Add][az-extension-add] или установите все доступные обновления с помощью команды [AZ Extension Update][az-extension-update] .

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
``` 

### <a name="limitations"></a>Ограничения
- См. [ **область и ограничения для Ultra дисков уровня "Глобальный** "](../virtual-machines/disks-enable-ultra-ssd.md#ga-scope-and-limitations)
- Поддерживаемый диапазон размера для Ultra Disks — от 100 до 1500

## <a name="create-a-new-cluster-that-can-use-ultra-disks"></a>Создание нового кластера, который может использовать Ultra Disks

Создайте кластер AKS, который может использовать Ultra Disks, с помощью следующих команд интерфейса командной строки. Используйте `--aks-custom-headers` флаг, чтобы задать `EnableUltraSSD` функцию.

Создайте группу ресурсов Azure:

```azurecli-interactive
# Create an Azure resource group
az group create --name myResourceGroup --location westus2
```

Создайте кластер AKS с поддержкой Ultra Disks.

```azurecli-interactive
# Create an AKS-managed Azure AD cluster
az aks create -g MyResourceGroup -n MyManagedCluster -l westus2 --node-vm-size Standard_L8s_v2 --zones 1 2 --node-count 2 --aks-custom-headers EnableUltraSSD=true
```

Если вы хотите создавать кластеры без поддержки Ultra Disk, это можно сделать, опустив пользовательский `--aks-custom-headers` параметр.

## <a name="enable-ultra-disks-on-an-existing-cluster"></a>Включение Ultra Disks в существующем кластере

Вы можете включить Ultra Disks в существующих кластерах, добавив новый пул узлов в кластер, поддерживающий Ultra Disks. Настройте новый пул узлов для использования шифрования на основе узла с помощью `--aks-custom-headers` флага.

```azurecli
az aks nodepool add --name ultradisk --cluster-name myAKSCluster --resource-group myResourceGroup --node-vm-size Standard_L8s_v2 --zones 1 2 --node-count 2 --aks-custom-headers EnableUltraSSD=true
```

Если вы хотите создать новые пулы узлов без поддержки Ultra Disks, это можно сделать, опустив пользовательский `--aks-custom-headers` параметр.

## <a name="use-ultra-disks-dynamically-with-a-storage-class"></a>Динамическое использование Ultra Disks с классом хранения

Для использования Ultra Disks в наших развертываниях или наборах с отслеживанием состояния можно использовать [класс хранения для динамической подготовки](azure-disks-dynamic-pv.md).

### <a name="create-the-storage-class"></a>Создание класса хранения

Класс хранения используется для определения того, как единица хранения создается динамически с помощью постоянного тома. Дополнительные сведения о классах хранения Kubernetes см. в разделе [Kubernetes Storage Classes][kubernetes-storage-classes] (Классы хранения Kubernetes).

В этом случае мы создадим класс хранения, который ссылается на Ultra Disks. Создайте файл `azure-ultra-disk-sc.yaml` и скопируйте в него следующий манифест.

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: ultra-disk-sc
provisioner: kubernetes.io/azure-disk
volumeBindingMode: WaitForFirstConsumer # optional, but recommended if you want to wait until the pod that will use this disk is created 
parameters:
  skuname: UltraSSD_LRS
  kind: managed
  cachingmode: None
  diskIopsReadWrite: "2000"  # minimum value: 2 IOPS/GiB 
  diskMbpsReadWrite: "320"   # minimum value: 0.032/GiB
```

Создайте класс хранения с помощью команды [kubectl Apply][kubectl-apply] и укажите файл *Azure-Ultra-Disk-SC. YAML* :

```console
$ kubectl apply -f azure-ultra-disk-sc.yaml


storageclass.storage.k8s.io/ultra-disk-sc created
```

## <a name="create-a-persistent-volume-claim"></a>Создание заявки на доступ к постоянному тому

Утверждение постоянного тома (PVC) используется для автоматической подготовки хранилища на основе класса хранения. В этом случае PVC может использовать ранее созданный класс хранения для создания диска Ultra.

Создайте файл `azure-ultra-disk-pvc.yaml` и скопируйте в него следующий манифест. Заявка запрашивает диск с именем `ultra-disk` размером *1000 ГБ* с доступом *реадвритеонце* . Класс хранения *Ultra-Disk-SC* указывается как класс хранения.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ultra-disk
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: ultra-disk-sc
  resources:
    requests:
      storage: 1000Gi
```

Создайте утверждение Постоянного тома с помощью команды [kubectl Apply][kubectl-apply] и укажите файл *Azure-Ultra-Disk-PVC. YAML* :

```console
$ kubectl apply -f azure-ultra-disk-pvc.yaml

persistentvolumeclaim/ultra-disk created
```

## <a name="use-the-persistent-volume"></a>Использование постоянного тома

После создания утверждения постоянного тома и успешной подготовки диска можно создать группу pod с доступом к диску. Следующий манифест создает базовый модуль NGINX, использующий утверждение Постоянного тома с именем *Ultra-Disk* для подключения диска Azure по пути `/mnt/azure` .

Создайте файл `nginx-ultra.yaml` и скопируйте в него следующий манифест.

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: nginx-ultra
spec:
  containers:
  - name: nginx-ultra
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
        claimName: ultra-disk
```

Создайте модуль pod с помощью команды [kubectl apply][kubectl-apply], как показано ниже.

```console
$ kubectl apply -f nginx-ultra.yaml

pod/nginx-ultra created
```

Теперь у вас есть работающий pod с диском Azure, подключенным к каталогу `/mnt/azure`. Эту конфигурацию можно видеть при проверке модуля с помощью `kubectl describe pod nginx-ultra`, как показано в сокращенном примере ниже.

```console
$ kubectl describe pod nginx-ultra

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


## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о Ultra Disks см. [в статье Использование Ultra дисков Azure](../virtual-machines/disks-enable-ultra-ssd.md).
- Дополнительные сведения о рекомендациях по хранению см. в статье рекомендации [по хранению и резервному копированию в службе Azure Kubernetes Service (AKS)][operator-best-practices-storage] .

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
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
