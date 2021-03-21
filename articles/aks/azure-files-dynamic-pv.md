---
title: Динамическое создание общей папки службы Файлов Azure
titleSuffix: Azure Kubernetes Service
description: Сведения о том, как динамически создавать постоянный том с файлами Azure для использования с несколькими параллельными pod в Службе Azure Kubernetes (AKS)
services: container-service
ms.topic: article
ms.date: 07/01/2020
ms.openlocfilehash: 2ad2affee34348e8c2fc7b734c8b49d0aec8db40
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96744915"
---
# <a name="dynamically-create-and-use-a-persistent-volume-with-azure-files-in-azure-kubernetes-service-aks"></a>Динамическое создание и использование постоянного тома с файлами Azure в службе Azure Kubernetes (AKS)

Постоянный том — это часть хранилища, которая подготовлена к использованию для модулей pod Kubernetes. Постоянный том может использоваться одним или несколькими модулями и может быть подготовлен динамически или статически. Если несколько объектов pod требуют одновременный доступ в одно и то же хранилище, используйте службу Файлов Azure для подключения с помощью [протокола Server Message Block (SMB)][smb-overview]. В этой статье показано, как в кластере Службы Azure Kubernetes с помощью нескольких модулей pod предоставлять общий доступ к динамическому созданию файлов Azure.

Дополнительные сведения о томах Kubernetes см. в статье, [посвященной возможностям хранения данных приложений в AKS][concepts-storage].

## <a name="before-you-begin"></a>Перед началом

В этой статье предполагается, что у вас есть кластер AKS. Если вам нужен кластер AKS, обратитесь к краткому руководству по работе с AKS [с помощью Azure CLI][aks-quickstart-cli] или [портала Azure][aks-quickstart-portal].

Кроме того, нужно установить и настроить Azure CLI версии 2.0.59 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][install-azure-cli].

## <a name="create-a-storage-class"></a>Создание класса хранения

Класс хранения используется для определения того, как создается файловый ресурс Azure. Для хранения общих папок Azure учетная запись хранения создается автоматически в [группе ресурсов узла][node-resource-group] для использования с классом хранилища. Выберите один из следующих [вариантов избыточности хранилища Azure][storage-skus] для *skuName*:

* *Standard_LRS* — локально избыточное хранилище ценовой категории "Стандартный" (LRS);
* *Standard_GRS* — геоизбыточное хранилище ценовой категории "Стандартный" (GRS);
* *Standard_ZRS* — стандартное хранилище, избыточное в пределах зоны (ZRS);
* *Standard_RAGRS* — геоизбыточное хранилище с доступом на чтение ценовой категории "Стандартный" (RA-GRS);
* *Premium_LRS* — локально избыточное хранилище уровня "Премиум" (LRS).
* *Premium_ZRS* избыточного хранилища в зоне уровня "Премиум" (ZRS)

> [!NOTE]
> Служба Файлов Azure поддерживает хранилище уровня "Премиум" в кластерах AKS с Kubernetes версии 1.13 или более поздней (минимальный размер общей папки уровня "Премиум" — 100 ГБ)

Дополнительные сведения о классах хранения Kubernetes для файлов Azure см. в разделе о [классах хранения Kubernetes][kubernetes-storage-classes].

Создайте файл под названием `azure-file-sc.yaml` и скопируйте в него следующий пример манифеста. Дополнительные сведения о *mountOptions* см. в разделе [Параметры подключения][mount-options].

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: my-azurefile
provisioner: kubernetes.io/azure-file
mountOptions:
  - dir_mode=0777
  - file_mode=0777
  - uid=0
  - gid=0
  - mfsymlinks
  - cache=strict
  - actimeo=30
parameters:
  skuName: Standard_LRS
```

Создайте класс хранения с помощью команды [kubectl apply][kubectl-apply].

```console
kubectl apply -f azure-file-sc.yaml
```

## <a name="create-a-persistent-volume-claim"></a>Создание заявки на доступ к постоянному тому

Утверждение постоянного тома (PVC) использует объект класса хранения для динамической подготовки файлового ресурса Azure. Следующий код YAML можно использовать, чтобы создать утверждение постоянного тома в размере *5 ГБ* с доступом *ReadWriteMany*. Дополнительные сведения о режимах доступа см. в документации [по постоянным томам Kubernetes][access-modes].

Теперь создайте файл под названием`azure-file-pvc.yaml` и скопируйте в него следующий код YAML. Убедитесь, что *storageClassName* соответствует классу хранения, созданному на предыдущем шаге.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-azurefile
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: my-azurefile
  resources:
    requests:
      storage: 5Gi
```

> [!NOTE]
> Если для класса хранения используется номер SKU *Premium_LRS*, минимальное значение для параметра *storage* должно быть *100Gi*.

Создайте утверждение постоянного тома с помощью команды [kubectl apply][kubectl-apply].

```console
kubectl apply -f azure-file-pvc.yaml
```

После ее выполнения будет создан файловый ресурс. Также будет создан секрет Kubernetes, содержащий сведения о подключении и учетные данные. Вы можете использовать команду [kubectl get][kubectl-get], чтобы просмотреть состояние PVC.

```console
$ kubectl get pvc my-azurefile

NAME           STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
my-azurefile   Bound     pvc-8436e62e-a0d9-11e5-8521-5a8664dc0477   5Gi        RWX            my-azurefile      5m
```

## <a name="use-the-persistent-volume"></a>Использование постоянного тома

В следующем YAML создается модуль Pod, использующий постоянное количество томов *My-азурефиле* для подключения файлового ресурса Azure по пути */МНТ/Азуре* . Для контейнеров Windows Server укажите *mountPath* в формате пути Windows, например *"D:"* .

Создайте файл `azure-pvc-files.yaml` и скопируйте в него следующий код YAML. Убедитесь, что *claimName* соответствует утверждению постоянного тома, созданному на предыдущем шаге.

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
        claimName: my-azurefile
```

Выполните команду [kubectl apply][kubectl-apply], чтобы создать объект pod.

```console
kubectl apply -f azure-pvc-files.yaml
```

Теперь у вас есть запущенный pod с общей папкой "Файлы Azure", созданной в каталоге */mnt/azure*. Эта конфигурация может отображаться при проверке вашего модуля через `kubectl describe pod mypod`. Ниже приведен краткий пример выходных данных, где показано, что том подключен к контейнеру:

```
Containers:
  mypod:
    Container ID:   docker://053bc9c0df72232d755aa040bfba8b533fa696b123876108dec400e364d2523e
    Image:          mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
    Image ID:       docker-pullable://nginx@sha256:d85914d547a6c92faa39ce7058bd7529baacab7e0cd4255442b04577c4d1f424
    State:          Running
      Started:      Fri, 01 Mar 2019 23:56:16 +0000
    Ready:          True
    Mounts:
      /mnt/azure from volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-8rv4z (ro)
[...]
Volumes:
  volume:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  my-azurefile
    ReadOnly:   false
[...]
```

## <a name="mount-options"></a>Параметры подключения

В Kubernetes версии 1.13.0 и более поздних значение по умолчанию для *fileMode* и *dirMode* — *0777*. При динамическом создании постоянного тома с классом хранилища параметры подключения можно указать в объекте класса хранения. В следующем примере задается значение *0777*.

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: my-azurefile
provisioner: kubernetes.io/azure-file
mountOptions:
  - dir_mode=0777
  - file_mode=0777
  - uid=0
  - gid=0
  - mfsymlinks
  - cache=strict
  - actimeo=30
parameters:
  skuName: Standard_LRS
```

## <a name="next-steps"></a>Дальнейшие действия

Соответствующие рекомендации см. в разделе [Рекомендации по хранению и резервному копированию в AKS][operator-best-practices-storage].

Сведения о параметрах класса хранения см. в разделе [динамическая инициализация](https://github.com/kubernetes-sigs/azurefile-csi-driver/blob/master/docs/driver-parameters.md#dynamic-provision).

Узнайте больше о постоянных томах Kubernetes, использующих службу файлов Azure.

> [!div class="nextstepaction"]
> [Подключаемый модуль Kubernetes для службы файлов Azure][kubernetes-files]

<!-- LINKS - external -->
[access-modes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubernetes-files]: https://github.com/kubernetes/examples/blob/master/staging/volumes/azure_file/README.md
[kubernetes-secret]: https://kubernetes.io/docs/concepts/configuration/secret/
[kubernetes-security-context]: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
[kubernetes-storage-classes]: https://kubernetes.io/docs/concepts/storage/storage-classes/#azure-file
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/
[pv-static]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#static
[smb-overview]: /windows/desktop/FileIO/microsoft-smb-protocol-and-cifs-protocol-overview

<!-- LINKS - internal -->
[az-group-create]: /cli/azure/group#az-group-create
[az-group-list]: /cli/azure/group#az-group-list
[az-resource-show]: /cli/azure/aks#az-aks-show
[az-storage-account-create]: /cli/azure/storage/account#az-storage-account-create
[az-storage-create]: /cli/azure/storage/account#az-storage-account-create
[az-storage-key-list]: /cli/azure/storage/account/keys#az-storage-account-keys-list
[az-storage-share-create]: /cli/azure/storage/share#az-storage-share-create
[mount-options]: #mount-options
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[az-aks-show]: /cli/azure/aks#az-aks-show
[storage-skus]: ../storage/common/storage-redundancy.md
[kubernetes-rbac]: concepts-identity.md#role-based-access-controls-rbac
[operator-best-practices-storage]: operator-best-practices-storage.md
[concepts-storage]: concepts-storage.md
[node-resource-group]: faq.md#why-are-two-resource-groups-created-with-aks
