---
title: Использование удостоверений под управлением Azure Active Directory Pod в службе Kubernetes Azure (Предварительная версия)
description: Узнайте, как использовать управляемые удостоверения, управляемые модулем AAD, в службе Kubernetes Azure (AKS).
services: container-service
ms.topic: article
ms.date: 12/01/2020
ms.openlocfilehash: e7c8a96ad012afdcd724a4a242c27018563f3a10
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102176320"
---
# <a name="use-azure-active-directory-pod-managed-identities-in-azure-kubernetes-service-preview"></a>Использование удостоверений под управлением Azure Active Directory Pod в службе Kubernetes Azure (Предварительная версия)

Azure Active Directory удостоверения, управляемые модулем Pod, используют примитивы Kubernetes для связывания [управляемых удостоверений для ресурсов][az-managed-identities] и удостоверений Azure в Azure Active Directory (AAD) с Pod. Администраторы создают удостоверения и привязки в виде примитивов Kubernetes, которые позволяют модулям Pod получать доступ к ресурсам Azure, использующим AAD в качестве поставщика удостоверений.

> [!NOTE]
> При наличии существующей установки ААДПОДИДЕНТИТИ необходимо удалить существующую установку. Включение этой функции означает, что компонент MIC не требуется.

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## <a name="before-you-begin"></a>Перед началом

Необходимо установить следующий ресурс:

* Azure CLI версии 2.8.0 или более поздней.
* `azure-preview`Расширение 0.4.68 или более поздняя версия

### <a name="limitations"></a>Ограничения

* Для кластера разрешено не более 50 удостоверений Pod.
* Для кластера разрешено не более 50 исключений идентификации Pod.
* Управляемые Pod удостоверения доступны только в пулах узлов Linux.

### <a name="register-the-enablepodidentitypreview"></a>Зарегистрируйте `EnablePodIdentityPreview`.

Зарегистрируйте компонент `EnablePodIdentityPreview`:

```azurecli
az feature register --name EnablePodIdentityPreview --namespace Microsoft.ContainerService
```

### <a name="install-the-aks-preview-azure-cli"></a>Установка `aks-preview` Azure CLI

Также требуется расширение *AKS-preview* Azure CLI версии 0.4.64 или более поздней. Установите расширение Azure CLI *AKS-Preview* с помощью команды [AZ Extension Add][az-extension-add] . Или установите все доступные обновления с помощью команды [AZ Extension Update][az-extension-update] .

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

## <a name="create-an-aks-cluster-with-managed-identities"></a>Создание кластера AKS с управляемыми удостоверениями

Создайте кластер AKS с включенным управляемым удостоверением и удостоверением, управляемым на основе Pod. Следующие команды используют команду [AZ Group Create][az-group-create] для создания группы ресурсов с именем *myResourceGroup* и команды [AZ AKS Create][az-aks-create] для создания кластера AKS с именем *myAKSCluster* в группе ресурсов *myResourceGroup* .

```azurecli-interactive
az group create --name myResourceGroup --location eastus
az aks create -g myResourceGroup -n myAKSCluster --enable-managed-identity --enable-pod-identity --network-plugin azure
```

Для входа в кластер AKS используйте команду [AZ AKS Get-Credential][az-aks-get-credentials] . Эта команда также скачивает и настраивает `kubectl` сертификат клиента на компьютере разработчика.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

## <a name="create-an-identity"></a>Создание удостоверения

Создайте удостоверение с помощью команды [AZ Identity Create][az-identity-create] и задайте переменные *IDENTITY_CLIENT_ID* и *IDENTITY_RESOURCE_ID* .

```azurecli-interactive
az group create --name myIdentityResourceGroup --location eastus
export IDENTITY_RESOURCE_GROUP="myIdentityResourceGroup"
export IDENTITY_NAME="application-identity"
az identity create --resource-group ${IDENTITY_RESOURCE_GROUP} --name ${IDENTITY_NAME}
export IDENTITY_CLIENT_ID="$(az identity show -g ${IDENTITY_RESOURCE_GROUP} -n ${IDENTITY_NAME} --query clientId -otsv)"
export IDENTITY_RESOURCE_ID="$(az identity show -g ${IDENTITY_RESOURCE_GROUP} -n ${IDENTITY_NAME} --query id -otsv)"
```

## <a name="assign-permissions-for-the-managed-identity"></a>Назначение разрешений управляемому удостоверению

Управляемое удостоверение *IDENTITY_CLIENT_ID* должно иметь разрешения читателя в группе ресурсов, которая содержит масштабируемый набор виртуальных машин для кластера AKS.

```azurecli-interactive
NODE_GROUP=$(az aks show -g myResourceGroup -n myAKSCluster --query nodeResourceGroup -o tsv)
NODES_RESOURCE_ID=$(az group show -n $NODE_GROUP -o tsv --query "id")
az role assignment create --role "Reader" --assignee "$IDENTITY_CLIENT_ID" --scope $NODES_RESOURCE_ID
```

## <a name="create-a-pod-identity"></a>Создание удостоверения Pod

Создайте удостоверение Pod для кластера с помощью `az aks pod-identity add` .

> [!IMPORTANT]
> `Owner`Для создания привязки удостоверений и ролей необходимо иметь соответствующие разрешения, такие как, в подписке.

```azurecli-interactive
export POD_IDENTITY_NAME="my-pod-identity"
export POD_IDENTITY_NAMESPACE="my-app"
az aks pod-identity add --resource-group myResourceGroup --cluster-name myAKSCluster --namespace ${POD_IDENTITY_NAMESPACE}  --name ${POD_IDENTITY_NAME} --identity-resource-id ${IDENTITY_RESOURCE_ID}
```

> [!NOTE]
> При включении удостоверения, управляемого на основе Pod, в кластере AKS в пространство имен *KUBE-System* добавляется азуреподидентитексцептион с именем *AKS-added-Exception* . Азуреподидентитексцептион позволяет модулям Pod с определенными метками получить доступ к конечной точке службы метаданных экземпляра Azure (IMDS) без перехвата сервером идентификации (NMI), управляемой узлами. *AKS-Add-Exception* позволяет выполнять надстройки AKS First, такие как удостоверение, управляемое AAD, без необходимости ручной настройки азуреподидентитексцептион. При необходимости можно добавлять, удалять и обновлять азуреподидентитексцептион с помощью `az aks pod-identity exception add` , `az aks pod-identity exception delete` , `az aks pod-identity exception update` или `kubectl` .

## <a name="run-a-sample-application"></a>Запуск примера приложения

Чтобы использовать удостоверение, управляемое AAD, модулю Pod требуется *аадподидбиндинг* метка со значением, совпадающим с селектором из *азуреидентитибиндинг*. Чтобы запустить пример приложения с помощью удостоверения, управляемого AAD, создайте `demo.yaml` файл со следующим содержимым. Замените *POD_IDENTITY_NAME*, *IDENTITY_CLIENT_ID* и *IDENTITY_RESOURCE_GROUP* значениями из предыдущих шагов. Замените *SUBSCRIPTION_ID* своим идентификатором подписки.

> [!NOTE]
> На предыдущих шагах вы создали переменные *POD_IDENTITY_NAME*, *IDENTITY_CLIENT_ID* и *IDENTITY_RESOURCE_GROUP* . Например, можно использовать команду, например, `echo` чтобы отобразить значение, заданное для переменных `echo $IDENTITY_NAME` .

```yml
apiVersion: v1
kind: Pod
metadata:
  name: demo
  labels:
    aadpodidbinding: POD_IDENTITY_NAME
spec:
  containers:
  - name: demo
    image: mcr.microsoft.com/oss/azure/aad-pod-identity/demo:v1.6.3
    args:
      - --subscriptionid=SUBSCRIPTION_ID
      - --clientid=IDENTITY_CLIENT_ID
      - --resourcegroup=IDENTITY_RESOURCE_GROUP
    env:
      - name: MY_POD_NAME
        valueFrom:
          fieldRef:
            fieldPath: metadata.name
      - name: MY_POD_NAMESPACE
        valueFrom:
          fieldRef:
            fieldPath: metadata.namespace
      - name: MY_POD_IP
        valueFrom:
          fieldRef:
            fieldPath: status.podIP
  nodeSelector:
    kubernetes.io/os: linux
```

Обратите внимание, что определение Pod содержит метку *аадподидбиндинг* со значением, совпадающим с именем удостоверения Pod, которое было запущено `az aks pod-identity add` на предыдущем шаге.

Разверните `demo.yaml` приложение в том же пространстве имен, что и удостоверение Pod, используя `kubectl apply` :

```azurecli-interactive
kubectl apply -f demo.yaml --namespace $POD_IDENTITY_NAMESPACE
```

Убедитесь, что пример приложения успешно запущен с помощью `kubectl logs` .

```azurecli-interactive
kubectl logs demo --follow --namespace $POD_IDENTITY_NAMESPACE
```

Убедитесь, что в журналах показано, что маркер успешно получен, а операция *получения* прошла успешно.
 
```output
...
successfully doARMOperations vm count 0
successfully acquired a token using the MSI, msiEndpoint(http://169.254.169.254/metadata/identity/oauth2/token)
successfully acquired a token, userAssignedID MSI, msiEndpoint(http://169.254.169.254/metadata/identity/oauth2/token) clientID(xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx)
successfully made GET on instance metadata
...
```

## <a name="clean-up"></a>Очистка

Чтобы удалить удостоверение, управляемое AAD, из кластера, удалите пример приложения и удостоверение Pod из кластера. Затем удалите удостоверение.

```azurecli-interactive
kubectl delete pod demo --namespace $POD_IDENTITY_NAMESPACE
az aks pod-identity delete --name ${POD_IDENTITY_NAME} --namespace ${POD_IDENTITY_NAMESPACE} --resource-group myResourceGroup --cluster-name myAKSCluster
az identity delete -g ${IDENTITY_RESOURCE_GROUP} -n ${IDENTITY_NAME}
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об управляемых удостоверениях для ресурсов Azure см. [в этой статье][az-managed-identities].

<!-- LINKS - external -->
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-group-create]: /cli/azure/group#az-group-create
[az-identity-create]: /cli/azure/identity#az_identity_create
[az-managed-identities]: ../active-directory/managed-identities-azure-resources/overview.md
[az-role-assignment-create]: /cli/azure/role/assignment#az_role_assignment_create
