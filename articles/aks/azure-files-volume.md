---
title: Создание общей папки службы файлов Azure вручную
titleSuffix: Azure Kubernetes Service
description: Сведения о том, как вручную создавать том с файлами Azure для использования с несколькими параллельными pod в Службе Azure Kubernetes (AKS)
services: container-service
ms.topic: article
ms.date: 03/01/2019
ms.openlocfilehash: 4e009c5de2e24c1b0bd94fb4c11b0c52a3bc378d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102609079"
---
# <a name="manually-create-and-use-a-volume-with-azure-files-share-in-azure-kubernetes-service-aks"></a>Создание вручную и совместное использование тома с файловым ресурсом Azure в службе Azure Kubernetes (AKS)

Контейнерные приложения часто требуются для обращения к данным и их хранения во внешнем томе данных. Если несколько объектов pod требуют одновременный доступ в одно и то же хранилище, используйте службу Файлов Azure для подключения с помощью [протокола Server Message Block (SMB)][smb-overview]. В этой статье показано, как вручную создать файловый ресурс Azure и присоединить его к контейнеру в AKS.

Дополнительные сведения о томах Kubernetes см. в статье, [посвященной возможностям хранения данных приложений в AKS][concepts-storage].

## <a name="before-you-begin"></a>Перед началом

В этой статье предполагается, что у вас есть кластер AKS. Если вам нужен кластер AKS, обратитесь к краткому руководству по работе с AKS [с помощью Azure CLI][aks-quickstart-cli] или [портала Azure][aks-quickstart-portal].

Кроме того, нужно установить и настроить Azure CLI версии 2.0.59 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][install-azure-cli].

## <a name="create-an-azure-file-share"></a>Создание файлового ресурса Azure

Перед использованием файлов Azure в качестве тома Kubernetes необходимо создать учетную запись хранения Azure и файловый ресурс. Следующие команды создают группу ресурсов с именем *мяксшаре*, учетную запись хранения и общую папку с именем *аксшаре*:

```azurecli-interactive
# Change these four parameters as needed for your own environment
AKS_PERS_STORAGE_ACCOUNT_NAME=mystorageaccount$RANDOM
AKS_PERS_RESOURCE_GROUP=myAKSShare
AKS_PERS_LOCATION=eastus
AKS_PERS_SHARE_NAME=aksshare

# Create a resource group
az group create --name $AKS_PERS_RESOURCE_GROUP --location $AKS_PERS_LOCATION

# Create a storage account
az storage account create -n $AKS_PERS_STORAGE_ACCOUNT_NAME -g $AKS_PERS_RESOURCE_GROUP -l $AKS_PERS_LOCATION --sku Standard_LRS

# Export the connection string as an environment variable, this is used when creating the Azure file share
export AZURE_STORAGE_CONNECTION_STRING=$(az storage account show-connection-string -n $AKS_PERS_STORAGE_ACCOUNT_NAME -g $AKS_PERS_RESOURCE_GROUP -o tsv)

# Create the file share
az storage share create -n $AKS_PERS_SHARE_NAME --connection-string $AZURE_STORAGE_CONNECTION_STRING

# Get storage account key
STORAGE_KEY=$(az storage account keys list --resource-group $AKS_PERS_RESOURCE_GROUP --account-name $AKS_PERS_STORAGE_ACCOUNT_NAME --query "[0].value" -o tsv)

# Echo storage account name and key
echo Storage account name: $AKS_PERS_STORAGE_ACCOUNT_NAME
echo Storage account key: $STORAGE_KEY
```

Запишите имя учетной записи хранилища и ключ, указанные в конце выходных данных сценария. Эти значения нужны для создания тома Kubernetes в одном из следующих действий.

## <a name="create-a-kubernetes-secret"></a>Создание секрета Kubernetes

Kubernetes требуются учетные данные для доступа к файловому ресурсу, созданные на предыдущем шаге. Эти учетные данные хранятся в [секрете Kubernetes][kubernetes-secret], который указывается при создании pod Kubernetes.

Создайте секрет с помощью команды `kubectl create secret`. Следующий пример создает ресурс с именем *azure-secret* и заполняет параметры *azurestorageaccountname* и *azurestorageaccountkey* данными, созданными на предыдущем шаге. Чтобы использовать существующую учетную запись Azure, укажите имя и ключ учетной записи.

```console
kubectl create secret generic azure-secret --from-literal=azurestorageaccountname=$AKS_PERS_STORAGE_ACCOUNT_NAME --from-literal=azurestorageaccountkey=$STORAGE_KEY
```

## <a name="mount-file-share-as-an-inline-volume"></a>Подключение файлового ресурса в качестве встроенного тома
> Примечание. Начиная с 1.18.15, 1.19.7, 1.20.2, 1.21.0, секретного пространства имен во встроенном `azureFile` томе можно задать только как `default` пространство имен, чтобы указать другое секретное пространство имен, вместо него используйте пример постоянного тома.

Чтобы подключить файловый ресурс Azure к модулю Pod, настройте том в спецификации контейнера. Создайте новый файл с именем `azure-files-pod.yaml` и следующим содержимым. Если вы изменили имя файлового ресурса или секрета, обновите *shareName* и *secretName*. При необходимости обновите `mountPath` — путь, в котором файловый ресурс установлен в pod. Для контейнеров Windows Server укажите *mountPath* в формате пути Windows, например *"D:"* .

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
    name: mypod
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
    volumeMounts:
      - name: azure
        mountPath: /mnt/azure
  volumes:
  - name: azure
    azureFile:
      secretName: azure-secret
      shareName: aksshare
      readOnly: false
```

Создайте pod с помощью команды `kubectl`.

```console
kubectl apply -f azure-files-pod.yaml
```

Теперь у вас есть запущенная группа pod с подключенным файловым ресурсом Azure в папке */mnt/azure*. Можно использовать `kubectl describe pod mypod` для проверки успешного подключения ресурсов. Ниже приведен краткий пример выходных данных, где показано, что том подключен к контейнеру:

```
Containers:
  mypod:
    Container ID:   docker://86d244cfc7c4822401e88f55fd75217d213aa9c3c6a3df169e76e8e25ed28166
    Image:          mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
    Image ID:       docker-pullable://nginx@sha256:9ad0746d8f2ea6df3a17ba89eca40b48c47066dfab55a75e08e2b70fc80d929e
    State:          Running
      Started:      Sat, 02 Mar 2019 00:05:47 +0000
    Ready:          True
    Mounts:
      /mnt/azure from azure (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-z5sd7 (ro)
[...]
Volumes:
  azure:
    Type:        AzureFile (an Azure File Service mount on the host and bind mount to the pod)
    SecretName:  azure-secret
    ShareName:   aksshare
    ReadOnly:    false
  default-token-z5sd7:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-z5sd7
[...]
```

## <a name="mount-file-share-as-an-persistent-volume"></a>Подключение общей папки в качестве постоянного тома
 - Параметры подключения

Значение по умолчанию для *fileMode* и *дирмоде* — *0777* для Kubernetes версии 1,15 и выше. В следующем примере задается значение *0755* для объекта *персистентволуме* :

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: azurefile
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  azureFile:
    secretName: azure-secret
    secretNamespace: default
    shareName: aksshare
    readOnly: false
  mountOptions:
  - dir_mode=0755
  - file_mode=0755
  - uid=1000
  - gid=1000
  - mfsymlinks
  - nobrl
```

Чтобы обновить параметры подключения, создайте файл *азурефиле-Mount-Options-PV. YAML* с *персистентволуме*. Пример:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: azurefile
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  azureFile:
    secretName: azure-secret
    shareName: aksshare
    readOnly: false
  mountOptions:
  - dir_mode=0777
  - file_mode=0777
  - uid=1000
  - gid=1000
  - mfsymlinks
  - nobrl
```

Создайте файл *азурефиле-Mount-Options-PVC. YAML* с *Персистентволумеклаим* , использующим *персистентволуме*. Пример:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azurefile
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ""
  resources:
    requests:
      storage: 5Gi
```

Используйте `kubectl` команды для создания *Персистентволуме* и *персистентволумеклаим*.

```console
kubectl apply -f azurefile-mount-options-pv.yaml
kubectl apply -f azurefile-mount-options-pvc.yaml
```

Убедитесь, что *персистентволумеклаим* создан и привязан к *персистентволуме*.

```console
$ kubectl get pvc azurefile

NAME        STATUS   VOLUME      CAPACITY   ACCESS MODES   STORAGECLASS   AGE
azurefile   Bound    azurefile   5Gi        RWX            azurefile      5s
```

Обновите спецификацию контейнера, чтобы она ссылалась на *персистентволумеклаим* и обновить свой модуль. Пример:

```yaml
...
  volumes:
  - name: azure
    persistentVolumeClaim:
      claimName: azurefile
```

## <a name="next-steps"></a>Дальнейшие действия

Соответствующие рекомендации см. в разделе [Рекомендации по хранению и резервному копированию в AKS][operator-best-practices-storage].

Дополнительные сведения о взаимодействии кластеров AKS с файлами Azure, см. в разделе [Подключаемый модуль Kubernetes для службы файлов Azure][kubernetes-files].

Сведения о параметрах класса хранения см. [в разделе статическая инициализация (создание собственного файлового ресурса)](https://github.com/kubernetes-sigs/azurefile-csi-driver/blob/master/docs/driver-parameters.md#static-provisionbring-your-own-file-share).

<!-- LINKS - external -->
[kubectl-create]: https://kubernetes.io/docs/user-guide/kubectl/v1.8/#create
[kubernetes-files]: https://github.com/kubernetes/examples/blob/master/staging/volumes/azure_file/README.md
[kubernetes-secret]: https://kubernetes.io/docs/concepts/configuration/secret/
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/volumes/
[smb-overview]: /windows/desktop/FileIO/microsoft-smb-protocol-and-cifs-protocol-overview
[kubernetes-security-context]: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

<!-- LINKS - internal -->
[az-group-create]: /cli/azure/group#az-group-create
[az-storage-create]: /cli/azure/storage/account#az-storage-account-create
[az-storage-key-list]: /cli/azure/storage/account/keys#az-storage-account-keys-list
[az-storage-share-create]: /cli/azure/storage/share#az-storage-share-create
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[operator-best-practices-storage]: operator-best-practices-storage.md
[concepts-storage]: concepts-storage.md
