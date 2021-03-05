---
title: Использование драйверов интерфейса хранилища контейнера (CSI) для службы файлов Azure в службе Kubernetes Azure (AKS)
description: Узнайте, как использовать драйверы интерфейса хранилища контейнера (CSI) для службы файлов Azure в кластере Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 08/27/2020
author: palma21
ms.openlocfilehash: 93f7f7a3c59beca362145ac16f7cf727df773f81
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102174067"
---
# <a name="use-azure-files-container-storage-interface-csi-drivers-in-azure-kubernetes-service-aks-preview"></a>Использование драйверов службы файлов хранилища контейнера Azure (CSI) в службе Kubernetes Azure (AKS) (Предварительная версия)

Драйвер интерфейса хранилища контейнера файлов Azure (CSI) — это драйвер, совместимый с [спецификацией CSI](https://github.com/container-storage-interface/spec/blob/master/spec.md), используемый службой Kubernetes Azure (AKS) для управления жизненным циклом файловых ресурсов Azure.

CSI является стандартом для предоставления произвольных блоков и систем хранения файлов в контейнерные рабочие нагрузки на Kubernetes. Принимая и используя CSI, AKS теперь может писать, развертывать и выполнять итерацию подключаемых модулей, чтобы предоставлять новые или улучшать существующие системы хранения в Kubernetes, не прибегая к основному коду Kubernetes и дожидаться циклов выпуска.

Сведения о создании кластера AKS с поддержкой драйвера CSI см. в статье [Включение драйверов CSI для дисков Azure и файлов Azure в AKS](csi-storage-drivers.md).

>[!NOTE]
> *Драйверы в дереве* относятся к текущим драйверам хранилища, которые являются частью основного кода Kubernetes, а также новыми драйверами CSI, которые являются подключаемыми модулями.

## <a name="use-a-persistent-volume-with-azure-files"></a>Использование постоянного тома с файлами Azure

[Постоянный том (ПС)](concepts-storage.md#persistent-volumes) представляет собой часть хранилища, подготовленную для использования с модулями Kubernetes. ПС может использоваться одним или несколькими модулями Pod и может быть динамическим или статически подготовленным. Если нескольким модулям Pod требуется одновременный доступ к тому же тому хранилища, можно использовать службу файлов Azure для подключения с помощью [протокола SMB][smb-overview]. В этой статье показано, как динамически создать файловый ресурс Azure для использования несколькими модулями Pod в кластере AKS. Сведения о статической подготовке см. [в статье Создание и использование тома с общей папкой Azure вручную](azure-files-volume.md).

Дополнительные сведения о томах Kubernetes см. в статье, [посвященной возможностям хранения данных приложений в AKS][concepts-storage].

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## <a name="dynamically-create-azure-files-pvs-by-using-the-built-in-storage-classes"></a>Динамическое создание файлов Azure постоянного хранилища версий с помощью встроенных классов хранения

Класс хранения используется для определения способа создания общей папки службы файлов Azure. Учетная запись хранения автоматически создается в [группе ресурсов узла][node-resource-group] для использования с классом хранения для хранения общих папок службы файлов Azure. Выберите один из следующих [номеров SKU избыточности службы хранилища Azure][storage-skus] для *skuName*:

* **Standard_LRS**: стандартное локально избыточное хранилище
* **Standard_GRS**: стандартное геоизбыточное хранилище
* **Standard_ZRS**: хранилище, избыточное в стандартном поясе
* **Standard_RAGRS**: стандартное геоизбыточное хранилище с доступом для чтения
* **Premium_LRS**: локально избыточное хранилище уровня "Премиум"

> [!NOTE]
> Служба файлов Azure поддерживает хранилище Azure класса Premium. Минимальная общая папка Premium составляет 100 ГБ.

Если вы используете драйверы CSI хранилища в AKS, есть два дополнительных встроенных `StorageClasses` , которые используют драйверы службы хранилища файлов Azure CSI. Дополнительные классы хранения CSI создаются с кластером наряду с классами хранения по умолчанию в дереве.

- `azurefile-csi`— Использует хранилище Azure уровня "Стандартный" для создания общей папки службы файлов Azure.
- `azurefile-csi-premium`: Использует хранилище Azure класса Premium для создания общей папки службы файлов Azure.

Политика reclaimа на оба класса хранения гарантирует, что базовая папка файлов Azure будет удалена при удалении соответствующего PV. Классы хранения также настраивают файловые ресурсы, которые должны быть расширяемыми, нужно просто изменить устойчивое утверждение тома (PVC) с новым размером.

Чтобы использовать эти классы хранения, создайте [PVC](concepts-storage.md#persistent-volume-claims) и соответствующий модуль, который ссылается на них и использует их. PVC используется для автоматической инициализации хранилища на основе класса хранилища. PVC может использовать один из предварительно созданных классов хранения или определяемый пользователем класс хранения для создания общего ресурса службы файлов Azure для требуемого номера SKU и размера. При создании определения Pod указывается PVC для запроса требуемого хранилища.

Создайте [Пример PVC и Pod, который выводит текущую дату в `outfile` ](https://github.com/kubernetes-sigs/azurefile-csi-driver/blob/master/deploy/example/statefulset.yaml) с помощью команды [kubectl Apply][kubectl-apply] :

```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/pvc-azurefile-csi.yaml
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/nginx-pod-azurefile.yaml

persistentvolumeclaim/pvc-azurefile created
pod/nginx-azurefile created
```

После того как модуль Pod находится в состоянии выполняется, можно проверить, что файловый ресурс правильно подключен, выполнив следующую команду и убедившись, что выходные данные содержат `outfile` :

```console
$ kubectl exec nginx-azurefile -- ls -l /mnt/azurefile

total 29
-rwxrwxrwx 1 root root 29348 Aug 31 21:59 outfile
```

## <a name="create-a-custom-storage-class"></a>Создание пользовательского класса хранилища

Классы хранения по умолчанию соответствуют наиболее распространенным сценариям, но не всем. В некоторых случаях может потребоваться настроить собственный класс хранения с собственными параметрами. Например, используйте следующий манифест для настройки `mountOptions` общей папки.

Значение по умолчанию для *fileMode* и *дирмоде* — *0777* для подключенных файловых ресурсов Kubernetes. Можно указать различные параметры монтирования для объекта класса хранения.

Создайте файл с именем `azure-file-sc.yaml` и вставьте следующий пример манифеста:

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: my-azurefile
provisioner: file.csi.azure.com
reclaimPolicy: Delete
volumeBindingMode: Immediate
allowVolumeExpansion: true
mountOptions:
  - dir_mode=0640
  - file_mode=0640
  - uid=0
  - gid=0
  - mfsymlinks
  - cache=strict # https://linux.die.net/man/8/mount.cifs
  - nosharesock
parameters:
  skuName: Standard_LRS
```

Создайте класс хранения с помощью команды [kubectl apply][kubectl-apply].

```console
kubectl apply -f azure-file-sc.yaml

storageclass.storage.k8s.io/my-azurefile created
```

Драйвер CSI файлов Azure поддерживает создание [моментальных снимков постоянных томов](https://kubernetes-csi.github.io/docs/snapshot-restore-feature.html) и базовых файловых ресурсов.

Создайте [класс моментальных снимков томов](https://github.com/kubernetes-sigs/azurefile-csi-driver/blob/master/deploy/example/snapshot/volumesnapshotclass-azurefile.yaml) с помощью команды [kubectl Apply][kubectl-apply] :

```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/snapshot/volumesnapshotclass-azurefile.yaml

volumesnapshotclass.snapshot.storage.k8s.io/csi-azurefile-vsc created
```

Создайте [моментальный снимок тома](https://github.com/kubernetes-sigs/azurefile-csi-driver/blob/master/deploy/example/snapshot/volumesnapshot-azurefile.yaml) из постоянной виртуальной цепи, [созданной динамически в начале этого руководства](#dynamically-create-azure-files-pvs-by-using-the-built-in-storage-classes), `pvc-azurefile` .


```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/snapshot/volumesnapshot-azurefile.yaml


volumesnapshot.snapshot.storage.k8s.io/azurefile-volume-snapshot created
```

Убедитесь, что моментальный снимок создан правильно:

```bash
$ kubectl describe volumesnapshot azurefile-volume-snapshot

Name:         azurefile-volume-snapshot
Namespace:    default
Labels:       <none>
Annotations:  API Version:  snapshot.storage.k8s.io/v1beta1
Kind:         VolumeSnapshot
Metadata:
  Creation Timestamp:  2020-08-27T22:37:41Z
  Finalizers:
    snapshot.storage.kubernetes.io/volumesnapshot-as-source-protection
    snapshot.storage.kubernetes.io/volumesnapshot-bound-protection
  Generation:        1
  Resource Version:  955091
  Self Link:         /apis/snapshot.storage.k8s.io/v1beta1/namespaces/default/volumesnapshots/azurefile-volume-snapshot
  UID:               c359a38f-35c1-4fb1-9da9-2c06d35ca0f4
Spec:
  Source:
    Persistent Volume Claim Name:  pvc-azurefile
  Volume Snapshot Class Name:      csi-azurefile-vsc
Status:
  Bound Volume Snapshot Content Name:  snapcontent-c359a38f-35c1-4fb1-9da9-2c06d35ca0f4
  Ready To Use:                        false
Events:                                <none>
```

## <a name="resize-a-persistent-volume"></a>Изменение размера постоянного тома

Вы можете запросить больший объем для PVC. Измените объект PVC и укажите больший размер. Это изменение активирует расширение базового тома, который производит резервное копирование PV.

> [!NOTE]
> Новый ПС никогда не будет создан для удовлетворения утверждения. Вместо этого размер существующего тома изменяется.

В AKS встроенный `azurefile-csi` класс хранения уже поддерживает расширение, поэтому используйте [PVC, созданный ранее с этим классом хранения](#dynamically-create-azure-files-pvs-by-using-the-built-in-storage-classes). PVC запросил общую папку 100Gi. Мы можем подтвердить, выполнив:

```console 
$ kubectl exec -it nginx-azurefile -- df -h /mnt/azurefile

Filesystem                                                                                Size  Used Avail Use% Mounted on
//f149b5a219bd34caeb07de9.file.core.windows.net/pvc-5e5d9980-da38-492b-8581-17e3cad01770  100G  128K  100G   1% /mnt/azurefile
```

Разверните PVC, увеличив `spec.resources.requests.storage` поле:

```console
$ kubectl patch pvc pvc-azurefile --type merge --patch '{"spec": {"resources": {"requests": {"storage": "200Gi"}}}}'

persistentvolumeclaim/pvc-azurefile patched
```

Убедитесь, что виртуальная цепь и файловая система в модуле Pod показывают новый размер:

```console
$ kubectl get pvc pvc-azurefile
NAME            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
pvc-azurefile   Bound    pvc-5e5d9980-da38-492b-8581-17e3cad01770   200Gi      RWX            azurefile-csi   64m

$ kubectl exec -it nginx-azurefile -- df -h /mnt/azurefile
Filesystem                                                                                Size  Used Avail Use% Mounted on
//f149b5a219bd34caeb07de9.file.core.windows.net/pvc-5e5d9980-da38-492b-8581-17e3cad01770  200G  128K  200G   1% /mnt/azurefile
```


## <a name="nfs-file-shares"></a>Общие файловые ресурсы NFS
[Теперь служба файлов Azure поддерживает протокол NFS версии 4.1](../storage/files/storage-files-how-to-create-nfs-shares.md). Поддержка NFS 4,1 для службы файлов Azure предоставляет полностью управляемую файловую систему NFS как службу, основанную на высокодоступной и высокоустойчивой распределенной платформе отказоустойчивого хранилища.

 Этот параметр оптимизирован для рабочих нагрузок произвольного доступа с обновлениями данных на месте и обеспечивает полную поддержку файловой системы POSIX. В этом разделе показано, как использовать общие папки NFS с драйвером CSI файлов Azure в кластере AKS.

Обязательно проверьте [ограничения](../storage/files/storage-files-compare-protocols.md#limitations) и [доступность регионов](../storage/files/storage-files-compare-protocols.md#regional-availability) на этапе предварительной версии.

### <a name="register-the-allownfsfileshares-preview-feature"></a>Регистрация `AllowNfsFileShares` функции предварительной версии

Чтобы создать общую папку, использующую NFS 4,1, необходимо включить `AllowNfsFileShares` флаг компонента в подписке.

Зарегистрируйте `AllowNfsFileShares` флаг компонента с помощью команды [AZ Feature Register][az-feature-register] , как показано в следующем примере:

```azurecli-interactive
az feature register --namespace "Microsoft.Storage" --name "AllowNfsFileShares"
```

Через несколько минут отобразится состояние *Registered* (Зарегистрировано). Проверьте состояние регистрации с помощью команды [AZ Feature List][az-feature-list] .

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.Storage/AllowNfsFileShares')].{Name:name,State:properties.state}"
```

Когда все будет готово, обновите регистрацию поставщика ресурсов *Microsoft. Storage* с помощью команды [AZ Provider Register][az-provider-register] :

```azurecli-interactive
az provider register --namespace Microsoft.Storage
```

### <a name="create-a-storage-account-for-the-nfs-file-share"></a>Создание учетной записи хранения для файлового ресурса NFS

[Создайте `Premium_LRS` Учетная запись хранения Azure](../storage/files/storage-how-to-create-file-share.md) со следующими конфигурациями для поддержки общих папок NFS:
- Тип учетной записи: Филестораже
- требуется безопасное перемещение (только включение HTTPS-трафика): false
- Выберите виртуальную сеть узлов агента в окне брандмауэры и виртуальные сети. Поэтому вы можете создать учетную запись хранения в MC_ группе ресурсов.

### <a name="create-nfs-file-share-storage-class"></a>Создание класса хранилища файлового ресурса NFS

Сохраните `nfs-sc.yaml` файл с указанным ниже манифестом, изменив соответствующие заполнители.

```yml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azurefile-csi-nfs
provisioner: file.csi.azure.com
parameters:
  resourceGroup: EXISTING_RESOURCE_GROUP_NAME  # optional, required only when storage account is not in the same resource group as your agent nodes
  storageAccount: EXISTING_STORAGE_ACCOUNT_NAME
  protocol: nfs
```

После изменения и сохранения файла создайте класс хранения с помощью команды [kubectl Apply][kubectl-apply] :

```console
$ kubectl apply -f nfs-sc.yaml

storageclass.storage.k8s.io/azurefile-csi created
```

### <a name="create-a-deployment-with-an-nfs-backed-file-share"></a>Создание развертывания с файловым ресурсом, поддерживающим NFS
Вы можете развернуть пример [набора с отслеживанием состояния](https://github.com/kubernetes-sigs/azurefile-csi-driver/blob/master/deploy/example/statefulset.yaml) , который сохраняет метки времени в файл `data.txt` , развернув следующую команду с помощью команды [kubectl Apply][kubectl-apply] :

 ```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/statefulset.yaml

statefulset.apps/statefulset-azurefile created
```

Проверьте содержимое тома, выполнив команду:

```console
$ kubectl exec -it statefulset-azurefile-0 -- df -h

Filesystem      Size  Used Avail Use% Mounted on
...
/dev/sda1                                                                                 29G   11G   19G  37% /etc/hosts
accountname.file.core.windows.net:/accountname/pvc-fa72ec43-ae64-42e4-a8a2-556606f5da38  100G     0  100G   0% /mnt/azurefile
...
```

>[!NOTE]
> Обратите внимание, что, поскольку общая папка NFS находится в учетной записи Premium, минимальный размер общей папки равен 100 ГБ. При создании PVC с небольшим размером хранилища может возникнуть ошибка "не удалось создать общую папку... Размер (5)... ".


## <a name="windows-containers"></a>Контейнеры Windows

Драйвер CSI для файлов Azure также поддерживает узлы и контейнеры Windows. Если вы хотите использовать контейнеры Windows, следуйте указаниям в [учебнике по контейнерам Windows](windows-container-cli.md) , чтобы добавить пул узлов Windows.

После создания пула узлов Windows используйте встроенные классы хранения, например `azurefile-csi` или создайте пользовательские. Можно развернуть пример набора с [отслеживанием состояния на основе Windows](https://github.com/kubernetes-sigs/azurefile-csi-driver/blob/master/deploy/example/windows/statefulset.yaml) , который сохраняет метки времени в файл `data.txt` , развернув следующую команду с помощью команды [kubectl Apply][kubectl-apply] :

 ```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/windows/statefulset.yaml

statefulset.apps/busybox-azurefile created
```

Проверьте содержимое тома, выполнив команду:

```console
$ kubectl exec -it busybox-azurefile-0 -- cat c:\\mnt\\azurefile\\data.txt # on Linux/MacOS Bash
$ kubectl exec -it busybox-azurefile-0 -- cat c:\mnt\azurefile\data.txt # on Windows Powershell/CMD

2020-08-27 22:11:01Z
2020-08-27 22:11:02Z
2020-08-27 22:11:04Z
(...)
```

## <a name="next-steps"></a>Дальнейшие действия

- Сведения об использовании драйверов CSI для дисков Azure см. в статье [Использование дисков Azure с драйверами CSI](azure-disk-csi.md).
- Дополнительные сведения о рекомендациях по хранению см. в статье рекомендации [по хранению и резервному копированию в службе Azure Kubernetes][operator-best-practices-storage].


<!-- LINKS - external -->
[access-modes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubernetes-storage-classes]: https://kubernetes.io/docs/concepts/storage/storage-classes/
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/
[managed-disk-pricing-performance]: https://azure.microsoft.com/pricing/details/managed-disks/
[smb-overview]: /windows/desktop/FileIO/microsoft-smb-protocol-and-cifs-protocol-overview


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
[node-resource-group]: faq.md#why-are-two-resource-groups-created-with-aks
[storage-skus]: ../storage/common/storage-redundancy.md